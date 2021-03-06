---
title: Ddv4 和 Ddsv4 系列
description: Dv4、Ddv4、Dsv4 和 Ddsv4 系列 VM 的规格。
author: rockboyfor
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
origin.date: 06/01/2020
ms.date: 07/06/2020
ms.author: v-yeche
ms.openlocfilehash: 93b3346d55a02086efab3b7346f33bb8c3c8a895
ms.sourcegitcommit: 89118b7c897e2d731b87e25641dc0c1bf32acbde
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/03/2020
ms.locfileid: "85946213"
---
<!--Pending GA on Q3 2020, only be suitable for China East 2 site-->
<!--RELEASE BEFORE CONFIRME AND BE CAREFULLY-->
# <a name="ddv4-and-ddsv4-series"></a>Ddv4 和 Ddsv4 系列

Ddv4 和 Ddsv4 系列采用英特尔&reg; 至强&reg; 铂金 8272CL (Cascade Lake) 处理器，具有超线程配置，为大多数通用工作负载提供了更好的价值主张。 它的持续全核睿频时钟速度为 3.4 GHz，采用[英特尔&reg; 睿频加速技术 2.0](https://www.intel.com/content/www/us/en/architecture-and-technology/turbo-boost/turbo-boost-technology.html)、[英特尔&reg; 超线程技术](https://www.intel.com/content/www/us/en/architecture-and-technology/hyper-threading/hyper-threading-technology.html)和[英特尔&reg; 高级矢量扩展 512（英特尔&reg; AVX-512）](https://www.intel.com/content/www/us/en/architecture-and-technology/avx-512-overview.html)。 它们还支持 [Intel&reg; Deep Learning Boost](https://software.intel.com/content/www/us/en/develop/topics/ai/deep-learning-boost.html)。 与 [Gen2 VM](/virtual-machines/linux/generation-2) 的 [Dv3/Dsv3](/virtual-machines/dv3-dsv3-series) 大小相比，这些新的 VM 大小的本地存储将增加 50%，而且本地磁盘的读写 IOPS 更佳。

D 系列用例包括企业级应用程序、关系数据库、内存中缓存和分析。

## <a name="ddv4-series"></a>Ddv4 系列

Ddv4 系列大小采用英特尔&reg; 至强&reg; 铂金 8272CL (Cascade Lake) 处理器。 Ddv4 系列提供 vCPU、内存和临时磁盘的组合，适用于大多数生产工作负载。

新的 Ddv4 VM 大小包括一个快速、大型本地 SSD 存储（多达 2,400 GiB），适用于需要低延迟、高速本地存储的应用程序，例如，适用于需要快速读取/写入临时存储或需要临时存储才能存储缓存或临时文件的应用程序。 可以将标准 SSD 和标准 HDD 存储附加到 Ddv4 VM。 远程数据磁盘存储与虚拟机分开计费。

ACU：195-210

高级存储：不支持

高级存储缓存：不支持

实时迁移：支持

内存保留更新：支持

| 大小 | vCPU | 内存:GiB | 临时存储 (SSD) GiB | 最大数据磁盘数 | 最大缓存吞吐量和临时存储吞吐量：IOPS/MBps | 最大 NIC 数/预期网络带宽 (Mbps) |
|---|---|---|---|---|---|---|
| Standard_D2d_v4 | 2 | 8 | 75 | 4 | 19000/120 | 2/1000 |
| Standard_D4d_v4 | 4 | 16 | 150 | 8 | 38500/242 | 2/2000 |
| Standard_D8d_v4 | 8 | 32 | 300 | 16 | 77000/485 | 4/4000 |
| Standard_D16d_v4 | 16 | 64 | 600 | 32 | 154000/968 | 8/8000 |
| Standard_D32d_v4 | 32 | 128 | 1200 | 32 | 308000/1936 | 8/16000 |
| Standard_D48d_v4 | 48 | 192 | 1800 | 32 | 462000/2904 | 8/24000 |
| Standard_D64d_v4 | 64 | 256 | 2400 | 32 | 615000/3872 | 8/30000 |

## <a name="ddsv4-series"></a>Ddsv4 系列

Ddsv4 系列采用英特尔&reg; 至强&reg; 铂金 8272CL (Cascade Lake) 处理器。 Ddsv4 系列提供 vCPU、内存和临时磁盘的组合，适用于大多数生产工作负载。

新的 Ddsv4 VM 大小包括一个快速、大型本地 SSD 存储（多达 2,400 GiB），适用于需要低延迟、高速本地存储的应用程序，例如，适用于需要快速读取/写入临时存储或需要临时存储才能存储缓存或临时文件的应用程序。 

 > [!NOTE]
 >Ddsv4 大小的定价和计费标准与 Ddv4 系列相同。

ACU：195-210

高级存储：支持

高级存储缓存：支持

实时迁移：支持

内存保留更新：支持

| 大小 | vCPU | 内存:GiB | 临时存储 (SSD) GiB | 最大数据磁盘数 | 最大缓存吞吐量和临时存储吞吐量：IOPS/MBps（以 GiB 为单位的缓存大小） | 最大非缓存磁盘吞吐量：IOPS/MBps | 最大 NIC 数/预期网络带宽 (Mbps) |
|---|---|---|---|---|---|---|---|
| Standard_D2ds_v4 | 2 | 8 | 75 | 4 | 19000/120(50) | 3000/48 | 2/1000 |
| Standard_D4ds_v4 | 4 | 16 | 150 | 8 | 38500/242(100) | 6400/96 | 2/2000 |
| Standard_D8ds_v4 | 8 | 32 | 300 | 16 | 77000/485(200) | 12800/192 | 4/4000 |
| Standard_D16ds_v4 | 16 | 64 | 600 | 32 | 154000/968(400) | 25600/384 | 8/8000 |
| Standard_D32ds_v4 | 32 | 128 | 1200 | 32 | 308000/1936(800) | 51200/768 | 8/16000 |
| Standard_D48ds_v4 | 48 | 192 | 1800 | 32 | 462000/2904(1200) | 76800/1152 | 8/24000 |
| Standard_D64ds_v4 | 64 | 256 | 2400 | 32 | 615000/3872(1600) | 80000/1200 | 8/30000 |

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes"></a>其他大小

- [常规用途](sizes-general.md)
- [内存优化](sizes-memory.md)
    <!--Not Avaialble on - [Storage optimized](sizes-storage.md)-->
- [GPU 优化](sizes-gpu.md)
    
    <!--Not Avaialble on - [High performance compute](sizes-hpc.md)-->
    
- [前几代](sizes-previous-gen.md)

## <a name="next-steps"></a>后续步骤

了解有关 [Azure 计算单元 (ACU)](acu.md) 如何帮助跨 Azure SKU 比较计算性能的详细信息。

<!-- Update_Description: new article about ddv4 ddsv4 series -->
<!--NEW.date: 07/06/2020-->