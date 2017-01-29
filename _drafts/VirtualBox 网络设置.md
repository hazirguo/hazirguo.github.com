# VirtualBox 网络设置

## 硬件

VirtaulBox 可以为每个虚拟机（VM）设置八个网卡，对每个网卡可以独立选择硬件的种类，VB 可以虚拟出六种网络硬件类型：

* AMD PCNet PCI II (Am79C970A);
* AMD PCNet FAST III (Am79C973, the **default**);
* Intel PRO/1000 MT Desktop (82540EM);
* Intel PRO/1000 T Server (82543GC);
* Intel PRO/1000 MT Server (82545EM);
* Paravirtualized network adapter (virtio-net).


## 网络模式

八个网络适配器可以独立配置使用下面任何一种模式：

* **Not attached**：这种模式下VB向客户机报告网卡存在，但是没有连接，就像没有插上网线一样。
* **Network Address Translation(NAT)**：