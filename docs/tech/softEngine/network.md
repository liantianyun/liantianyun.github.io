# 一些高频面试题
## 从浏览器输入URL到响应的全过程。

* DNS转换，先查本地的host文件，查不到就向本地域名服务器查询，这里为递归查询，如果查不到，本地域名服务器将代替主机成为新的客户端，向根服务器->顶级域名服务器->权限域名服务器依次发起迭代查询，直到获取到IP地址，DNS查询采用的是UDP协议。

* 查到ip地址后会通过子网掩码判断目标地址与本机是否属于同一个子网，如果属于同一个子网，则先通过arp广播获取目标的MAC地址，再转发至目的主机，若不同，则先将该包转发给网关，网关通过路由协议找到目标主机所在的网络地址后通过路由转发，到达目的网络后通过arp协议获取目的主机mac地址后转发至目的主机。以上讲的是一个报文的寻址方式，在本题中应该是先通过三报文握手建立TCP连接，连接建立后发送数据报文（HTTP报文），数据报文传输完成后四次挥手断开TCP连接。

## TCP是如何进行拥塞控制的？

网络拥塞产生的根本原因是流入网络的分组超过了网络中的物理设备所能处理的分组数量的上限，网络中处理报文需要各种资源，如缓存设备、链路的带宽、处理器等，单一的提高某个设备的性能是没有意义的（木桶效应），这么多的设备要使他们的性能达到一个平衡是非常的困难的一件事情，因此，必须从软件的层面着手，通过一定机制监控拥塞的产生，在产生拥塞的时候减少进入网络的分组的数量，这些机制就是TCP的拥塞控制。

TCP的拥塞控制包含下面四个算法：

慢开始、拥塞避免、快重传、快恢复

这几个算法是有顺序的，基本就概括了TCP拥塞控制的流程：

先慢开始（不要一次性把所有要传输的报文全部投入网络，一开始先放1个、没问题就2个、然后4个、8个...以此类推，以指数的速度增长），指数的增长无疑是很快的，这样下去网络极有可能产生拥塞，因此，在指数增长到大于一个定值（这个定值就称为慢开始门限）后，我们就改变算法，改为一次多投一个报文进入网络，这样增长速率大大变缓，网络便不易产生拥塞，因此，这个算法就叫“拥塞避免”。这两个算法下进入网络的报文总是在增长的，在这个过程中，一发生了拥塞，就调整慢开始门限，重新开始慢开始算法。提了很多次拥塞这个词，怎么样认定网络发生了拥塞呢？由于物理设备损坏的可能性很小，因此，凡是发生了超时，TCP就认为网络发生了拥塞。

在TCP的滑动窗口机制下，接收方只能对所接受报文的最后一个序号进行确认（这个序号前的报文必须全部到齐），在这种机制下，如果一串报文丢了中间那一个，那么后面的报文即使按时到达也是无法被确认的，那么，就必然会产生超时，这时，TCP就会进行拥塞控制（网络正常，仅仅是个别报文丢失），这样就降低了网络的传输速率，因此，必须设计一个算法解决这个问题，这个算法就是快重传算法，假如1-10序号的报文中5号丢失，在不采用快重传算法时发送端一定会等到超时计时器过了才会重传5号报文，采用快重传后，接收方必须对每一个报文发送及时的确认，譬如说6、7号报文到达后继续发送4号报文的确认，当发送端连续收到三个同一报文的确认后，就知道该报文下一个报文发生了丢失，这时仅仅是丢失了个别报文，不需要重新启动慢开始，而是重新选择一个点后，直接开始拥塞避免算法，快速恢复网络中的报文数量，因此这个算法就称为“快恢复”。

## XSS和CSRF攻击的原理和防范

XSS即脚本注入攻击，通过让浏览器执行攻击者注入浏览页面的脚本代码达到攻击者的目的。

防范：
* 使cookie带有Httponly属性，可以防止JS访问，这种方式可以防止XSS劫持cookie
* 输入检查，对用户的输入进行检查，对特殊字符进行处理

CSRF即利用用户已登录的特性，伪造用户请求来达到攻击者的目的。
方法：
* 加入验证码的机制
* Referer Check 通过http头部的referer字段检查该请求的来源验证请求的合法性

## TCP和UDP的差别？

* 从功能上讲，TCP比UDP多了流量控制，拥塞控制等功能
* 性能上讲，TCP是面向连接的可靠传输，UDP不可靠，TCP传输时延以较长，UDP较短，这也造成了两者适用场景的不同，一般来说要求“实时”的服务（如网络电话，视频直播）等采用UDP较为合适，要求可靠传输的采用TCP。
* 造成两者的差异的原因是两者的实现机制不同，UDP不可靠的原因是网络本身是不可靠的，UDP并未采用什么机制加以控制，TCP可靠的原因是对于每一个发出去的包TCP协议都要求对方发回一个确认报文（传输时延高的原因？），在这个规则下制定了超时重传机制和滑动窗口机制（流量控制的基础就是滑动窗口机制），正是由于机制的不同，所以UDP报文的头部只有八位，TCP报文头部有20位。

## TCP连接的三报文握手的过程

![](../images/three-hands-up.png)
首先，服务器先创建传输控制块TCB，并处于LISTEN（监听）状态，客户端发起请求时，也要先创建传输控制块TCB，接着发送一个请求报文，选定一个初始序号x，将同步位SYN置为1，表示这是一个请求报文，当SYN为1 时，报文不允许携带数据，但仍要消耗一个序号，因此，服务端发来的确认报文的ack位为x+1，同时确认报文中ACK位为1，表示这是一个确认报文，只有ACK为1的时候ack位的数字才有意义，在客户端发起请求到收到确认报文之间，客户端处于SYN-SENT（同步已发送）状态，收到确认报文之后，客户端将再次发送一个确认报文，理论上来说，收到确认报文的客户端已经拿到了发送数据的必要信息（对方的确认号，窗口大小），已经可以遵循TCP协议正常发送数据了，但是为什么要加入第三次报文确认呢？这是为了解决网络中延时的请求报文重新到达的问题。在服务器收到客户端传来的确认报文前，处于SYN-RCVD（同步已收到）状态，客户端发送确认报文号后就进入ESTABLISHED（已建立连接）状态，服务器端在收到确认报文后也进入ESTABLISHED状态，这两者时间是不对称的，设想一下，客户端发送的ACK报文丢失怎么办？此时在客户端看来，连接已建立，而服务端仍然处于SYN-RCVD状态，这种情况下，客户端如果尝试发送报文到服务端，服务端将会用RST包回应（用于强制关闭TCP连接），这时客户端就能感知到发生了错误，服务端在发送SYN-ACK包一段时间后如果仍然未收到确认报文，将会重发该包，重发包的次数可通过命令进行配置，超过这个次数后如果仍未收到，将会关闭这个连接。

## 四次握手的过程，有什么状态？

![](../images/four-hands-break.png)

在关闭连接前两端都处于ESTABLISHED状态。

数据传输结束后，双方都可以主动断开连接，但这个断开不是对称的，也就是说，不是一方断开另一方就跟着断开。先传输完数据的一方（设为客户端）在准备断开时，先发送一个表示结束的报文（FIN位置为1），假设在建立连接时已发送的包序号为u-1，此时的seq位便应置为u，发送完后，就进入了FIN-WAIT-1（终止等待1）状态，服务端收到这个报文后，发送一个确认报文，ack位应为u+1，因为FIN报文与SYN一样，即使不携带数据也要占据一个序号，seq位假设为v（与u同理，在正常连接时已发送的包序号+1）。这时服务端进入CLOSE-WAIT（关闭等待）状态，服务端在收到这个报文后进入FIN-WAIT-2（终止等待2）状态，此时客户端的连接已经释放，也就是说客户端已经传输完数据也不可以再继续传输数据了，但服务端的数据却不一定传完了，所以FIN-WAIT-2状态的意义就是等待对方传完数据，这个状态结束的标志就是收到了对方传来的FIN-ACK报文，在这个报文中，seq位不应是之前的v，因为在CLOSE-WAIT状态中可能服务端还发送了一些数据，假设此时的seq为为w，ack由于在此期间客户端已经没有发送任何报文了，所以应该重复上次的u+1，发送完这个报文后，就进入了LASK-ACK（最后确认）状态，客户端收到这个报文后，了解到服务端也已经没有数据可以发送了，但它仍然不会直接关闭进入CLOSED状态，它会再发送一个确认报文，并进入TIME-WAIT状态，这个状态的设置是为了确认对方正常收到这个报文，如果在TIME-WAIT状态内未收到服务端重传的FIN-ACK报文，就意味着服务端已经正常关闭了，这时在TIME-WAIT结束后客户端也可以正常关闭。