# CSD (Computational Storage Drive)

<p align = "justify">
Data are now accumulating exponentially in the Internet era, and how to process massive data efficiently has drawn much attention in recent years. Although distributed systems and hardware accelerators can alleviate this problem to a certain extent, more and more researches demonstrate that the main bottleneck of performance degradation is often data transmission. Under the conventional architecture, data are usually transferred from the storage to the host. However, moving vast amounts of data may lead to not only response latency but also higher energy consumption. In-storage computing has been considered as one of the directions to relieve the data transfer bottleneck. We fully exploit the intrinsic computing power of the real ARM-based NVMe SSD without additional processors to build a Lightweight CSD (Computational Storage Drive). By redesigning the firmware of the customized SSD, the inside ARM cores of the SSD will also take charge of computational tasks and only need to return the results with a much smaller data size. Equipped with this lightweight computational storage, the system performance will achieve great improvement by eliminating redundant data transmission to the host and offloading partial computations to the storage. Besides, the energy consumption of the heterogeneous system will also be significantly reduced.
</p>

## Motivation

1. Redundant data movement (page migrations of the block-based interface induce tremendous I/O traffic)
2. High energy consumption (I/O traffic and computations)
3. Long I/O stack (obvious for high performance SSD)

## Framework

<p align = "justify">
We spare partial computing power for the computing module and extend the NVMe interface for task assignment and return results. We carefully and fully utilize the computing power of each processor and adopt a parallelizable strategy to improve overall system performance. Moreover, as a role of storage, the original function of the SSD should not be affected, such as FTL and normal I/O. To leverage the computational modules in the SSD, the device driver and the interface between the device and host should also be modified correspondingly. At this stage, the lightweight computational SSD can work with basic functions, and support end-to-end workflow, as shown in the following:
</p>

```markdown
                                 ++=================================================++
                                 ||   +-----------------------------------------+   ||
                                 ||   |           CSD Host Application          |   ||
                                 ||   +-----------------------------------------+   ||
                                 ||           ⇅                1 ↓         ↑ 5      ||
                                 ||   +---------------+ +-----------------------+   ||
                                 ||   |  Host Memory  |⇆|  CSD Runtime Library  |   ||
                                 ||   +---------------+ +-----------------------+   ||              
                                 ++==================================↓====↑=========++
                                                                   2 ↓    ↑ 4            Host
                        <=============================PCIe(NVMe)=====↓====↑===================>
                                                                   3 ↓    ↑              CSD
                                 ++==================================↓====↑=========++
                                 ||   +-----------------+   +-------------------+   ||
                                 ||   |  Device Memory  |   |   CSD Framework   |   ||
                                 ||   +-----------------+   +-------------------+   ||
                                 ||            ⇅                  ↓     ↑ d         ||
                                 ||            ⇅            +-----------------+     ||
                                 ||            ⇅ ⇆ ⇆ ⇆ ⇆ ⇆  |     CSD App     |  c  ||
                                 ||                         +-----------------+     ||
                                 ||                                  ↑ b            ||
                                 ||   +-----------------------------------------+   ||
                                 ||   |               SSD Firmware              |   ||
                                 ||   +-----------------------------------------+   ||
                                 ||                         ⇅ a                     ||
                                 ||   +-----------------------------------------+   ||
                                 ||   |               Flash Array               |   ||
                                 ||   +-----------------------------------------+   ||
                                 ++=================================================++
```

### Host-side: &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; CSD-side:

1. Allocate app memory for input/output;
2. Send app parameters;
3. Send LBA list to device;
&emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp;  &emsp; &emsp; a. Send PBA address;
