---
title: "远程文件"
type: "docs"
weight: 3
---

## FTP/FTPS/SFTP

FTP 是一种广泛应用的标准网络协议，用于在互联网上的客户端与服务器之间传输文件。它不提供加密，所有通信（包括用户名和密码）以明文形式在网络上传输。虽然简单方便、容易部署，但是传输效率低、安全性不佳，不推荐使用。

FTPS 是 FTP 协议的安全版本，通过 SSL/TLS 协议对 FTP 会话进行加密，以保护数据安全。

SFTP 不是 FTP 的一个单独扩展，而是基于 SSH 协议的文件传输服务，提供了安全的数据传输，包括加密身份验证和数据完整性检查。

FTP/FTPS/SFTP 服务端、客户端应用成熟，但是不支持挂载为远程硬盘。

## SCP/rsync/Syncthing

SCP 是基于 SSH 协议的一个简单文件传输工具，用于在计算机之间安全复制文件和目录，确保所有数据在传输过程中被加密。

rsync 是一个开源的、高效的文件和目录同步工具，可以在本地或通过网络在两个计算机之间进行快速、增量的文件同步操作。

Syncthing 是一个开源的 P2P 文件同步工具，它可以让多台设备之间的文件保持同步，无需中心服务器，并且具有自动检测和同步文件变化的能力。

## FastDFS/GlusterFS/Ceph

FastDFS 是轻量级的开源分布式文件系统，专为互联网服务设计，提供文件存储、文件同步及文件访问等功能。

GlusterFS 是一个开源的分布式文件系统，可将多个物理存储设备组织成一个大的、统一的、虚拟的存储池，支持数据冗余、高可用性和横向扩展。

Ceph 是一个高度可扩展、统一的开源存储系统，设计用于提供对象存储、块存储和文件系统接口。它是一个分布式的、自我修复的存储平台，旨在提供大规模、高性能和可靠性的存储解决方案。

## NFS/SSHFS/DLNA/SMB/WebDAV

NFS 是一种分布式的网络文件系统协议，最初由 Sun Microsystems 开发，提供近乎本地文件系统的性能和响应速度，Unix/Linux/Mac 操作系统原生支持，从 Windows 7 开始，微软在其操作系统中内置了 NFS 客户端功能，Windows Server 还支持设置 NFS 服务器。NFS v4 提供了一些基本的权限和安全控制，但早期版本的安全性相对较低，一般在受信任的内部网络中使用。

SSHFS（Secure Shell FileSystem）是一种网络文件系统，通过 SSH（Secure Shell）协议将远程服务器上的文件系统挂载到本地计算机上。SSHFS 最大特点是安全性，适合于需要安全文件访问的场景，但是受限于 SSH 协议的速度和特性，主要用于个人和小规模文件共享。

DLNA 不是一个专门的文件传输协议，而是一个标准联盟，定义了一套设备之间媒体内容共享的规范，让数字媒体设备能够互相发现并交换多媒体内容。2017 年 DLNA 在其官网宣布：本组织的使命已经完成，已于 2017 年 1 月 5 日正式解散，未来不会再更新 DLNA 标准。

SMB 是微软开发的一种网络文件共享协议，也称为 CIFS（Common Internet File System），最初用于在 Windows 网络环境的局域网内共享文件、打印服务以及其他资源，现在跨平台支持良好，包括 Linux、Mac 等，但是因为遭受 永恒之蓝、WannaCry 等网络蠕虫攻击，其 139/445 端口在广域网被运营商封禁。SMB 缺点是传输效率稍低，速度不太稳定，大文件一定不要使用。

WebDAV 是一种基于 HTTP/HTTPS 协议的扩展，允许用户编辑和管理存储在远程 Web 服务器上的文件，就像操作本地文件系统一样。Linux、Mac、Windows 原生支持 WebDAV 服务端。由于基于 Web 协议，容易穿越防火墙，但是受限于 HTTP/HTTPS 的性能开销，频繁操作小文件不是最优选择。

NFS、SMB、WebDAV 都支持在 Linux、Mac、Windows 挂载为远程硬盘，SMB 端口在广域网被封禁、手机端需要 Wi-Fi 才能使用，SSHFS 在 Windows 下需要通过第三方工具实现类似功能。NFS 在 Android 挂载需要 root 权限，在 IOS 没有应用可以实现挂载。

WebDAV 是目前（2024.03.16）的最佳选择。

## WebDAV 挂载硬盘

Linux：Linux 系统可以通过 davfs2（一个用于挂载 WebDAV 资源的 FUSE 模块）来挂载 WebDAV 服务器，使其表现为本地文件系统的一部分。

Mac：macOS 自带对 WebDAV 的支持，用户可以直接在 Finder 中选择“前往” -> “连接服务器...”，输入 WebDAV 服务器地址并连接，即可将远程 WebDAV 目录挂载为本地卷。

Windows：Windows 操作系统也原生支持 WebDAV，用户可以通过“映射网络驱动器”功能将 WebDAV URL 映射为本地驱动器，从而访问远程 WebDAV 服务器上的文件。

Android：在 Android 系统中，虽然不直接支持挂载为文件系统，但有多种第三方应用（如 WebDAV Navigator、Solid Explorer 等）可以访问和管理 WebDAV 服务器上的文件，部分应用可通过插件或内建功能实现类似挂载的功能。

iOS：iOS 系统同样可以通过一些第三方应用来访问 WebDAV 资源，例如 Documents by Readdle，用户可以在这个应用内访问和管理 WebDAV 服务器上的文件，但同样不具备直接挂载为文件系统的功能，而是通过应用内的文件管理器进行交互。

## 私有云存储解决方案

NextCloud

[https://nextcloud.com/](https://nextcloud.com/)

OwnCloud

[https://owncloud.com/](https://owncloud.com/)

Seafile

[https://www.seafile.com/home/](https://www.seafile.com/home/)

NextCloud、OwnCloud 基于 PHP 开发，Seafile 基于 Python 开发。Seafile 对大文件分块，社区版免费。
