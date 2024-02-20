---
title: "Virtualization"
type: "docs"
weight: 1
---

## 硬件虚拟化

硬件虚拟化需要硬件支持，以提供更高的性能和效率，比如 Intel VT-x 和 AMD-V。

**KVM (Kernel-based Virtual Machine):** KVM 是一个基于 Linux 内核的虚拟化模块，它允许将 Linux 作为宿主机来创建和管理虚拟机。

**QEMU (Quick Emulator):** QEMU 是一个开源的虚拟化和模拟器工具，支持多种硬件平台，可以与 KVM 结合。它可以用于虚拟化，同时也可用于模拟不同的架构，如 ARM、MIPS 等。

**Firecracker:** 基于 KVM，aws 开源，启动速度快，主要用于 serverless。

## 裸金属虚拟化

用于在裸机上安装的虚拟化技术，这些技术通常被称为裸金属虚拟化（Bare Metal Hypervisors）。它们直接在硬件上运行，而无需依赖主机操作系统。

1. **VMware ESXi:** VMware 内核，轻量、高性能、高可靠，性能损耗趋近于 0%，不支持 docker。商用气息浓重，界面清晰易用，但硬件兼容性较差，没什么扩展性。

2. **Microsoft Hyper-V Server:** 基于 Microsoft Hyper-V 技术。它允许在 Windows 环境中运行多个虚拟机。

3. **XenServer:** 基于 Xen 虚拟化技术的裸金属虚拟化平台，由 Citrix 提供。它支持创建和管理虚拟机，并提供一些高级功能。

4. **Proxmox Virtual Environment (Proxmox VE):** Proxmox VE 是一个基于 Debian 的开源裸金属虚拟化平台，集成了 KVM 虚拟化和容器虚拟化，硬件兼容性优秀，性能损失 10%左右。默认无图形界面，web 访问操作，连不上无线网络，更适合服务器。界面功能不强，很多操作要靠命令行，但扩展能力几乎是无限的。

[https://pve.proxmox.com/](https://pve.proxmox.com/)

5. **OpenStack Ironic:** OpenStack Ironic 是 OpenStack 项目的一个组件，用于在裸机上提供虚拟化服务。它允许将物理机直接用作虚拟机宿主机。

6. **Nutanix AHV:** Nutanix Acropolis Hypervisor（AHV）是 Nutanix 公司的裸金属虚拟化平台，通常与 Nutanix 超融合基础设施一起使用。

AWS 早期使用 Xen，目前是自研的 Nitro。阿里云是自研的 飞天平台 ApsaraStack。Xen 基本被 KVM 取代。

## 云计算平台

1. **OpenStack:**

   - 美国国家航空航天局(NASA)和 Rackspace 发起，腾讯云、华为云基于 OpenStack 二次开发。
   - 天坑，架构乱、代码质量差，每一块都有厂商挖坑又不埋，裸机安装很困难。

2. **OpenNebula:**

   - 类似于 OpenStack 的开源云平台，提供虚拟化和管理工具。
   - 它支持多种虚拟化技术，包括 KVM 和 VMware。

3. **Apache CloudStack:**

   - 诞生时间较早，场景是企业内中小型虚拟化环境的管理平台，社区现在很冷清，但是还在持续更新。
   - 由 Apache 软件基金会支持的开源云计算平台。
   - 具有弹性伸缩、虚拟网络等功能。

4. **Eucalyptus:**

   - 开源的云计算平台，兼容 Amazon Web Services (AWS) API。
   - 可以通过 Eucalyptus 创建 AWS 兼容的私有云。

5. **kubernetes**
   - 容器编排平台，没有对硬件资源抽象，不支持裸机虚拟化，不属于云计算概念。

## 虚拟机

VMware

virtualbox
