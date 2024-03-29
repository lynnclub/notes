---
title: "Kafka"
type: "docs"
weight: 2
---

## 架构

![架构](kafka.webp)

1. 生产者（Producer）：负责将消息发布到 Kafka 集群。
2. 消费者（Consumer）：从 Kafka 集群订阅并处理消息。
3. 主题（Topic）：消息的类别，Kafka 消息被发布到主题。
4. 分区（Partition）：每个主题可以分为多个分区，以提高并行处理能力。
5. 副本（Replica）：分区的备份，用于提供容错和高可用性，当主分区（Leader）故障时会选择一个从分区（Follower）上位。
6. ZooKeeper：用于协调和管理 Kafka 集群的分布式系统。
7. Broker：Kafka 集群中的每个服务器节点称为 Broker，负责存储和处理消息。

## 性能优势

Kafka 之所以具有良好的性能，有以下几个原因：

1. 分布式架构：Kafka 采用分布式架构，可以将数据分散存储在多个节点上，实现数据的并行处理和高吞吐量。

2. 高效的存储机制：Kafka 使用磁盘存储消息，而不是内存，这使得它可以处理大量的消息并保持持久性。此外，Kafka 使用了批量写入和顺序写入的方式，提高了磁盘写入的效率。

3. 零拷贝技术：Kafka 使用零拷贝技术来避免数据在内存和磁盘之间的多次拷贝，减少了数据传输的开销，提高了性能。

4. 分区和并行处理：Kafka 将数据分为多个分区，并允许多个消费者并行地读取和处理这些分区，从而实现了高并发和高吞吐量。

5. 可伸缩性：Kafka 的分布式架构和分区机制使得它可以轻松地扩展到大规模的集群，以满足不断增长的数据处理需求。

总的来说，Kafka 通过分布式架构、高效的存储机制、零拷贝技术、分区和并行处理以及可伸缩性等特性，实现了高性能的消息传输和处理能力。

### 零拷贝

零拷贝（Zero-copy）是一种优化技术，旨在减少数据在内存和磁盘之间的多次拷贝。传统的数据传输方式涉及多次拷贝，例如从磁盘读取数据到内核缓冲区，然后再从内核缓冲区拷贝到用户空间缓冲区。这些拷贝操作会增加 CPU 和内存的开销。

零拷贝技术通过避免这些额外的拷贝操作来提高性能。它的核心思想是将数据直接从源位置传输到目标位置，而不需要在中间进行多次拷贝。这可以通过使用内存映射（mmap）或直接 I/O（Direct I/O）等技术来实现。

### 二进制存储

Kafka 在磁盘上存储的消息是以二进制格式进行存储的，而不是明文。Kafka 使用了自定义的二进制格式来表示消息，这样可以有效地压缩和序列化消息，以减少存储空间和网络传输的开销。这种二进制格式包含了消息的元数据（如主题、分区、偏移量等）以及消息的实际内容。这种结构化的二进制格式使得 Kafka 能够高效地读写和处理消息，并支持各种数据类型的消息。

批量写入+顺序读写+零拷贝+二进制存储=最佳读写性能

## 主题与分区

Kafka 不建议创建太多 topic 和 partition，单节点一般不超过 1000。topic 太多对应的 partition 也多，会导致性能下降。

1. 管理开销增加：更多的分区需要更多的管理开销，包括负载均衡、leader 选举与重现选举、消费者组，可能导致性能下降。
2. 文件数增加：更多的分区对应更多文件、占用更多文件句柄，大量小文件会降低读写性能，而且可能导致数据在磁盘上分散存储，顺序读写退化为随机读写。

## 生产数据

kafka 每次都是向 Leader 分区发送数据，并顺序写入到磁盘，然后 Leader 分区会将数据同步到各个从分区 Follower，即使主分区挂了，也不会影响服务的正常运行。

写入原则：

1. 数据写入时可以指定分区；
2. 如果没有指定分区，但是设置了数据的 key，则会根据 key 的值 hash 出一个分区；
3. 如果既没指定分区，又没有设置 key，则会轮询选出一个分区；

异步发送示例：

```go
import "github.com/segmentio/kafka-go"

write := kafka.NewWriter(kafka.WriterConfig{
		Brokers:          config.Viper.GetStringSlice("kafka"),
		Topic:            topic,
		Balancer:         &kafka.LeastBytes{},
		MaxAttempts:      5,
		Async:            true,
		RequiredAcks:     1, //-1（等待所有分区的确认），0（不等待确认），1（等待至少一个分区的确认）
		BatchSize:        200,
		CompressionCodec: kafka.Lz4.Codec(),
})
write.AllowAutoTopicCreation = true
write.Completion = func(messages []kafka.Message, err error) {
		// 处理错误逻辑
		if err != nil {
			logger.Error("Failed to deliver messages:", err.Error())
			return
		}

		// 处理成功逻辑
		for _, msg := range messages {
			fmt.Printf("Message delivered to topic: %s, partition: %d, offset: %d\n",
				msg.Topic, msg.Partition, msg.Offset)
		}
}
```

异步发送其实是加入缓冲区，批量发送，放弃同步处理错误，需要在程序退出时调用 writer.Close() 以免丢失消息。

## 确认机制

Kafka 的确认机制主要针对消息发送，确保消息被正确地写入到 Kafka 集群中，提供了三种消息确认机制。

1. 不等待确认（acks=0）：生产者发送消息后不会等待任何确认，直接将消息发送出去，这种方式可能会导致消息丢失。
2. 等待 leader 确认（acks=1）：生产者发送消息后会等待 leader 分区确认，一旦 leader 确认接收到消息，生产者就会认为消息发送成功，这种方式可以提供一定程度的可靠性。
3. 等待所有副本确认（acks=all）：生产者发送消息后会等待所有副本都确认接收到消息，只有在所有副本都确认后，生产者才会认为消息发送成功，这种方式提供最高的可靠性，但会增加延迟和开销。

## 消费数据

消费者主动去 kafka 集群拉取消息时，也是从 Leader 分区拉取。

多个消费者可以组成一个消费组，以提升并行消息能力。同一消费组内的消费者可以消费同一 topic 下不同分区的数据，同一个分区只会被一个消费组内的某一个消费者所消费，防止出现重复消费的问题。不同的组，可以消费同一个分区的数据。

如果组内消费者数量大于分区数量，就会出现消费者闲置，如果分区数量大于组内消费者数量，就会出现一个消费者负责多个分区的情况、性能不均衡，建议消费者组 consumer 数量与 partition 数量保持一致！

消费者不能同时指定消费者组 id 和分区 id，指定消费者组 Kafka 会自动将消息分配给组中的不同消费者，以实现负载均衡和高可用性，指定分区意味着放弃负载均衡。

消费者消费消息后，消费的偏移量（offset）会随之移动到已消费的消息的下一个位置。这意味着消费者已经成功消费了这些消息，并且 Kafka 不会再向消费者发送这些已经被消费的消息。

然而，即使消费者的偏移量移动了，这些已经被消费的消息仍然会在 Kafka 中保留一段时间，这个时间段由 Kafka 的配置参数 retention.ms 或 retention.bytes 决定。这个时间段内，消费者可以通过偏移量重新消费这些消息，或者进行偏移量提交（offset commit）。

## 消费位移与重试

对于消费者来说，消费者可以选择手动提交消费位移（offset）或者使用自动提交位移的方式来确认消息已经被消费。手动提交消费位移可以确保消息被成功处理，而自动提交位移则可能会导致消息重复消费或者丢失。

Kafka 还支持消息重试，当消费者处理消息失败时，可以通过以下方式进行重试：

1. 重新提交位移：消费者可以重新提交处理失败的消息的位移，然后重新拉取该消息进行处理。
2. 设置重试策略：消费者可以实现自定义的重试策略，例如指数退避（exponential backoff）或者固定延迟重试，以避免对 Kafka 产生过大的压力。
3. 使用死信队列：对于处理多次失败的消息，可以将其发送到死信队列，以便后续进行分析或者人工处理。

另外，消费者可以使用事务来确保消息的原子性处理、提高消息的可靠性，还可以结合消息重试和错误记录来进行重试。对于无法通过重试解决的异常情况，可以考虑自动告警通知相关人员处理。综合利用这些方法可以提高消费者处理失败时的重试能力和容错性。

## kafka ui

[https://github.com/provectus/kafka-ui](https://github.com/provectus/kafka-ui)

```shell
docker run -it -p 8080:8080 -e DYNAMIC_CONFIG_ENABLED=true provectuslabs/kafka-ui
```

```yaml
services:
  kafka-ui:
    container_name: kafka-ui
    image: provectuslabs/kafka-ui:latest
    restart: always
    network_mode: host
    ports:
      - 8080:8080
    environment:
      DYNAMIC_CONFIG_ENABLED: "true"
    volumes:
      - ./config.yml:/etc/kafkaui/dynamic_config.yaml
```
