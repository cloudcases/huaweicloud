# 五分钟理解VRF

[五分钟理解VRF（Virtual Routing and Forwarding，虚拟路由转发）](https://www.cnblogs.com/grhack/p/13981378.html)

![img](https://img-blog.csdnimg.cn/20190301110350669.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0thbmd5dWNoZW5n,size_16,color_FFFFFF,t_70)

 

![img](https://img-blog.csdnimg.cn/20190301110855304.gif)

假设PC1与R2这一侧的网络属于一个独立的业务；PC2与R3这一侧的网络属于另一个独立的业务，由于设备资源有限或者其他方面的原因，这两个独立的业务的相关节点连接在R1上，也就是同一台设备上。那么在完成相关配置后，R1的路由表如上图所示。
现在如果PC1要发一个数据包到2.2.2.2，那么这个数据包在到达R1后，R1就会去查看自己的路由表，发现有一条2.2.2.0/24的路由匹配，因此将这个IP包从GE0/0/2口转发给192.168.100.2。这是没有问题的，然而如果PC1要访问3.3.3.0/24网络呢？也是无压力的，因为数据包到达R1后，她照样查找路由表结果发现有匹配的路由，因此将数据包转给R3。但是实际上，从业务的角度考虑，我们禁止PC1访问3.3.3.0/24网络。
那么怎么办？

![img](https://img-blog.csdnimg.cn/20190301110351539.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0thbmd5dWNoZW5n,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/2019030111115184.gif)

现在，我们在R1上创建两个VRF：VRF1及VRF2，创建完成后，我们可以理解为，拥有了两台虚拟路由器。当然，现在这两台虚拟路由器上啥也没有。
接下去我们将GE0/0/1口及GE0/0/2口绑定到VRF1；将GE0/0/3及GE0/0/4口绑定到VRF2。如此一来这两台虚拟路由器就各自拥有了两个物理接口。值得注意的是，这两台虚拟路由器是虽然都在同一台物理设备上，但是却是隔离的，他们将有自己的接口，自己的路由表，自己的ARP表等等相关的内容。我们的环境就变成有点像这样：

 

 ![img](https://img-blog.csdnimg.cn/20190301111226761.gif)

我们看到，VRF1及VRF2有了自己的接口，也有了自己的路由表。并且相互之间是隔离的。
现在PC1要发送一个数据包到2.2.2.2，R1从接口GE0/0/1收到了这个数据包，由于此时GE0/0/1已经绑定到了VRF1，因此在执行目的IP的路由查找的时候，查的是VRF1的路由表，查找到匹配的路由条目后，间个数据包从其指示的GE0/0/1口转发给下一跳192.168.100.2。

那么如果PC1要访问3.3.3.3呢？数据包发到了R1，R1从接口GE0/0/1收到了这个数据包，于是它在做路由查找的时候，查的仍然是VRF1的路由表。经过查表后，它发现并无匹配的条目，因此将数据包丢弃。

 ![img](https://img-blog.csdnimg.cn/20190301111315435.gif)

分类: [网络基础知识](https://www.cnblogs.com/grhack/category/1576983.html), [交换机部分](https://www.cnblogs.com/grhack/category/1863975.html), [路由部分](https://www.cnblogs.com/grhack/category/1863976.html)