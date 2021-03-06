# 读图解TCP/IP—IP部分

1. IP属于OSI的那一层

    在OSI网络分层概念中，IP协议划分在网络层。

2. IP和数据链路层之间的关系，及为什么要分为两层进行处理？

    数据链路负责两个同在一个链路层的设备通信，与之相比，作为网络层的IP负责两个不同网络之间进行通信。

    为什么分为两层进行处理，对于复杂的网络环境，不可能所有的电脑都在同一个链路中。所有需要从某一个网络主机A找到另一个网络环境中的主机B，此时则需要跨过过个路由器，此时IP具有层级关系的优势则非常明显，非常类似现在的身份证，从某省到某市到某镇，到最终找到对应的主机。

3. IP寻址，路由控制

    如何根据IP地址找到对方主机呢，这个过程可以称之为IP寻址。而IP寻址主要依靠的是路由控制，每一个路由器都会维护一个路由控制表，表中主要的数据分别为原IP地址，目标地址IP。所有根据路由控制表，则能找到目标主机。

4. IP的组成
	
	IP的由两部分组成，分别是网络号和主机号。在寻找一个IP地址，首先会根据网络号找到对应的网络，之后根据主机号在该网络内找到主机。

5. 网络IP的分类
		
	在原来的规划中，IP分类为四类，分别为
	
	A类地址：首位为0，1-8位为网络号。
	
	B类地址：首位为10，1-16位为网络号。
	
	C类地址：首位为110，1-24位为网络号。
	
	D类地址：首位为1110，1-32位为网络号。
	
	所以D类地址没有主机号。常用于多播地址。
	
6. 子网掩码的作用

	在IP仅进行4类地址分类是，极大的浪费了IP资源，所以则出现了子网掩码，可以动态的通过子网掩码确定网络号的长度。例如192.168.23.45/24
	
7. 路由控制表

	路由控制表主要应用在IP寻址的过程中，主要作用就是寻找下一个路由器。如下图
	
	![路由控制表](http://pengshuang.space/img/juhe.png)

8. 分片和重组

	在网络中，会碰到报文过大，而无法直接发送出去。此时需要对报文进行分片处理。而其中会有一个问题，则是在众多的路由中传递报文，如何确定分片报文的大小。此时则有一项技术——"路径MTU发现"，是通过发送方主机向外发送探测，确定最小的分片大小。之后则通过该大小进行分片，并进行在网络中传输。
	
	值得提到的是，路由器只进行报文分片，目的主机进行重组。主要目的是为了路由器的效率。

9. IP周边协议

	DNS：域名转换系统，主要是由于IP是一串数字，不利于人的记忆，则出现了使用英文代替IP地址，而DNS协议，用于将英文地址，转换为IP地址。这里面涉及到域名系统。
	
	ARP：地址解析协议，主要用于发送端数据到达目标主机时，数据无法直接使用，需要根据物理的MAC地址进行通信，则此时通过ARP协议，利用IP地址找到MAC地址。仅能用于与IPV4
	
	RARP：反向地址转换协议，主要作用和ARP相反，基于MAC地址和IP的映射，反向从通过MAC地址找到IP地址
	
	ICMP：Internet控制报文协议，主要作用，是辅助IP协议，判断报文是否到达目的主机，诊断在发送过程是否出现中断，网络通不通，主机是否可到达，路由是否可用。所有对用用户数据在传输过程中起到重要作用
	
	DHCP：动态主机配置协议，首先需要了解到，在以前，每一台主机都需要网络管理员为其设置IP地址，所以在同一个网络中IP地址不能重复，但仅靠人去维护，是一件比较麻烦的事情。而DHCP而为此而生，当一台主机接入网络中时，会自动向DCHP服务器，请求IP地址，而地址一旦被请求，而被标记为已使用。这用大大的减少了网络管理员的任务，以及犯错的概率。
	
	IP隧道：主要是为了解决IPV4和IPV6进行通信。通过在报文的首部中加上IPV4首部和IPV6首部实现的。
