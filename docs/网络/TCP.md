### 三次握手和四次挥手
#### TCP 的三次握手过程如下：
第一次握手：客户端向服务器发送一个带有 <font style="color:#DF2A3F;">SYN </font>标志的数据包，会携带一个<font style="color:#DF2A3F;">初始序列号</font>假设为x，这个序列号是<font style="color:#DF2A3F;">随机生成</font>的，用于标识客户端后续发送的<font style="color:#DF2A3F;">数据字节流的</font>**<font style="color:#DF2A3F;">起始位置</font>**。然后发起请求建立连接。此时客户端进入<font style="color:#DF2A3F;"> SYN_SENT</font> 状态。

第二次握手：服务器收到客户端的 SYN 数据包后，为了确认客户端的请求，会发送一个<font style="color:#DF2A3F;"> SYN + ACK</font> 包。这个包中，服务器也会生成一个<font style="color:#DF2A3F;">自己的初始序列号</font>，假设为 y。同时，服务器会把客户端发送的<font style="color:#DF2A3F;">序列号 x + 1 作为确认应答号</font>放在这个包中，表示已经收到了客户端的序列号为 x 的 SYN 包，期望下次收到客户端从序列号 x + 1 开始的数据。此时表示同意建立连接。此时服务器进入<font style="color:#DF2A3F;"> SYN_RCVD</font> 状态。

第三次握手：客户端收到服务器的<font style="color:#DF2A3F;"> SYN + ACK</font> 数据包后，会向服务器发送一个<font style="color:#DF2A3F;"> ACK 包</font>进行确认，这个 ACK 包中的确认应答号为服务器的序列号<font style="color:#DF2A3F;"> y + 1</font>，表示已经收到了服务器的序列号为<font style="color:#DF2A3F;"> y </font>的 SYN 包，期望下次收到服务器从序列号 y + 1 开始的数据。确认连接建立。此时客户端和服务器都进入 <font style="color:#DF2A3F;">ESTABLISHED</font> （/ <font style="color:rgba(0, 0, 0, 0.85);">ɪˈstæblɪʃt /</font>）状态，连接建立成功。

##### 不能两次
TCP 不能使用两次握手来建立连接，主要原因是：<font style="color:rgb(62, 62, 62);">两次握手</font><font style="color:#DF2A3F;">只能保证单向连接是畅通</font><font style="color:rgb(62, 62, 62);">的。三次主要是为了</font><font style="color:#DF2A3F;">建立可靠的通信信道</font><font style="color:rgb(62, 62, 62);">，</font>两次握手无法保证客户端和服务器双方的**<font style="color:#DF2A3F;">初始序列号都能被对方正确接收和确认</font>**，无法<font style="color:rgb(62, 62, 62);">保证客户端与服务端</font><font style="color:#DF2A3F;">同时具备发送、接收数据</font><font style="color:rgb(62, 62, 62);">的能力。</font>可能会导致**<font style="color:#DF2A3F;">已失效的连接请求报文段</font>**突然又传送到了服务器，从而产生错误。

#### TCP 的四次挥手过程如下：
第一次挥手：主动关闭方（通常是客户端）发送一个带有 <font style="color:#DF2A3F;">FIN </font>标志的数据包，其中会携带一个<font style="color:#DF2A3F;">序列号</font>，假设为 **u**。这个序列号用于标<font style="color:#DF2A3F;">识主动关闭方之前已发送的数据</font>。此时主动关闭方进入<font style="color:#DF2A3F;"> FIN_WAIT_1 </font>状态，表示它<font style="color:#DF2A3F;">不再发送数据，但仍可以接收数据</font>。

第二次挥手：被动关闭方收到主动关闭方的 FIN 数据包后，会发送一个带有<font style="color:#DF2A3F;"> ACK </font>标志的数据包，ACK 号为主动关闭方发送的 <font style="color:#DF2A3F;">FIN </font>包中的序列号** u + 1**，表示已经<font style="color:#DF2A3F;">收到了主动关闭方的关闭请求</font>。此时被动关闭方进入 <font style="color:#DF2A3F;">CLOSE_WAIT </font>状态，表示它收到了关闭请求，但<font style="color:#DF2A3F;">还可能有数据需要继续发送给主动关闭</font>方。主动关闭方收到这个 ACK 包后，进入 <font style="color:#DF2A3F;">FIN_WAIT_2</font> 状态，等待被动关闭方<font style="color:#DF2A3F;">发送完剩余数据</font>。

第三次挥手：被动关闭方处理完剩余数据后，发送一个带有 <font style="color:#DF2A3F;">FIN </font>标志的数据包，其中也会携带一个<font style="color:#DF2A3F;">序列号</font>，假设为** w**。此时被动关闭方进入 <font style="color:#DF2A3F;">LAST_ACK</font> 状态，表示它也<font style="color:#DF2A3F;">不再发送数据</font>，等待主动关闭方的确认。

第四次挥手：主动关闭方收到被动关闭方的 <font style="color:#DF2A3F;">FIN</font> 数据包后，发送一个带有<font style="color:#DF2A3F;"> ACK </font>标志的数据包，ACK 号为被动关闭方发送的 FIN 包中的序列号** w + 1**，表示确认收到被动关闭方的关闭请求。此时连接<font style="color:#DF2A3F;">彻底关闭</font>，双方都进入<font style="color:#DF2A3F;"> TIME_WAIT</font> 状态，经过一段时间后才真正释放资源。

##### <font style="color:rgb(62, 62, 62);">为什么四次：</font>
<font style="color:rgb(62, 62, 62);">因为需要</font><font style="color:#DF2A3F;">确保客户端与服务端的数据能够完成传输</font><font style="color:rgb(62, 62, 62);">。</font>

## **超时重传**
   <font style="color:#DF2A3F;">  在发送数据的时候,设置一个</font>**<font style="color:#DF2A3F;">定时器</font>**<font style="color:#DF2A3F;">,当超过指定的时间后,没有收到 ACK应答,就会重发数据,分为 两种情况 1.数据包丢失 2.确认应答丢失.</font>

RTT   数据发送时间到收到确认的时间差

RTO  超时重传时间   linux不断采集当前RTT ,还有当前RTT的波动范围  进行加权平均,        算出一个平滑的RTT值

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725258784500-fcebe546-1f3d-4a03-b910-e0395efe738c.png)

<font style="color:#DF2A3F;">每一次超时重传时会将下一次的超时时间间隔(RTO)设置为先前值的两倍 </font>**<font style="color:#DF2A3F;">,两次超时</font>**<font style="color:#DF2A3F;">,就说明</font>**<font style="color:#DF2A3F;">网络环境差</font>**<font style="color:#DF2A3F;">,不宜频繁反复发送</font>

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725259142814-3272653b-e1b8-4818-911f-8393097ac1ec.png)![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725259102548-df5fe727-e2eb-41bf-8688-17aa7f00f41d.png)





## 快速重传
<font style="color:#DF2A3F;">不以时间为驱动,而是以数据驱动重传,工作方式是当</font>**<font style="color:#DF2A3F;">收到三个相同的 ACK 报文时</font>**<font style="color:#DF2A3F;">，会在定时器</font>**<font style="color:#DF2A3F;">过期之前</font>**<font style="color:#DF2A3F;">，重传丢失的报文段。  </font>

例如发6个包 除了第二个包其他的全收到了ack就会回复 ack2 ack2 ack2  发送方收到三个ack2后就知道第二个包丢了就会重发第二个包



<font style="color:#DF2A3F;">快速重传只解决了一个问题,超时间的问题,但是仍面临另一个问题,就是重传的时候,是重传一个,还是重传所有的问题.</font>

所以引入SACK方法解决

### SACK-选择性确认 方法
<font style="color:#DF2A3F;">在TCP 头部选项字段里加一个SACK 的东西他可以将</font>**<font style="color:#DF2A3F;">已收到的数据</font>**<font style="color:#DF2A3F;">信息发送给[发送方]这样就知道发送方那些数据收到了,那些数据没收到,从而只</font>**<font style="color:#DF2A3F;">重传丢失</font>**<font style="color:#DF2A3F;">的数据</font>

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725259932978-dc74dd05-7a5d-4e3b-844c-e84e4cca5e84.png)

### <font style="color:rgb(44, 62, 80);">Duplicate SACK (D-SACK)</font>
    <font style="color:#DF2A3F;">使用SACK来告诉发送方有哪些数据被重复接受  一般两种情况 1.ACK 丢包2.网络延迟</font>

<font style="color:#000000;">判</font><font style="color:#DF2A3F;">断的时候看如果ACK >SACK   则sack就是D-sack里面的部分就是重复接收的部分</font>

<font style="color:#DF2A3F;">好处:</font>![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725260934875-0f1a8a0f-e22f-4f7a-96c6-0e936e05761e.png)

<font style="color:#DF2A3F;"></font>

<font style="color:#DF2A3F;"></font>

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725260218611-a63ab6fd-088f-418b-99da-0bd424db29d0.png)

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725260385677-f1879913-d3a4-48e4-ab29-539394bab6ea.png)

情况2 网络延迟



![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725260427957-7f7acfd9-e55e-44d1-b770-64b07ba5f781.png)

## 滑动窗口
<font style="color:#DF2A3F;">因为tcp如果每一次都要等到收到应答ack才发下一个包,效率太慢,所有就引入滑动窗口</font>,假如**窗口数是3**就连着发三个包,只要收到最后一个包的应答就算完成

<font style="color:#DF2A3F;">实现:就是操作系统开辟的一个缓存空间,发送方主机在得到应答之前必须在缓存区中保留已发送的数据,如果按期收到确认应答,此时数据就可以从</font>**<font style="color:#DF2A3F;">缓存区中清除</font>**

<font style="color:#DF2A3F;">窗口大小: 无需等待确认应答,而可以继续发送数据的最大值</font>

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725263100195-c3121c25-1f5b-4eee-8b2e-6eef5c2d69ee.png)

### **窗口大小由哪一方决定**
<font style="color:#DF2A3F;">tcp头部有个字段 </font>**<font style="color:#DF2A3F;">Window</font>**<font style="color:#DF2A3F;"> ,也就是窗口大小这个字段是</font>**<font style="color:#DF2A3F;">接收端告诉发送方</font>**<font style="color:#DF2A3F;">自己有多少缓冲区可以接受数据,</font>于是发送端就可以根据这个接收端的处理能力来发送数据,而不会导致接收端处理不过来<font style="color:#DF2A3F;">,所以窗口大小是由接收方的窗口大小来决定的,</font>**<font style="color:#DF2A3F;">发送方窗口大小不能超过接收方</font>**

### 发送方的滑动窗口(swnd)
滑动窗口分为四部分

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725263756599-37571e2b-ef93-4cbf-b753-9c06295573d6.png)



![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725263796201-e10bf80b-1c8d-44ff-ab59-4a09b437c77a.png)

程序用三个指针表示发送方滑动窗口的四部分

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725264867901-6cc2cb42-4ed5-42cc-861c-d14400d88d0e.png)

### 接受方的滑动窗口(rwnd)
![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725265004764-de575a1b-af7b-4063-9f04-10be3b76dc84.png)



### 接收方窗口和发送窗口大小是相等的么?
![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725265158118-3566937c-409e-4fde-a225-22b09dc47160.png)

## 流量控制
<font style="color:#DF2A3F;">tcp提供的一种机制可以让发送方</font>**<font style="color:#DF2A3F;">根据接收方的实际接受</font>**<font style="color:#DF2A3F;">能力控制发送的数据量,就是所谓的流量控制   目的是为了</font>**<font style="color:#DF2A3F;">避免发送方发送的数据填满接收方的缓存</font>**

### 操作系统缓存区与滑动窗口的关系
[4.2 TCP 重传、滑动窗口、流量控制、拥塞控制 | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/network/3_tcp/tcp_feature.html#%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E7%BC%93%E5%86%B2%E5%8C%BA%E4%B8%8E%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E7%9A%84%E5%85%B3%E7%B3%BB)

     当接受方的缓存区只接受了数据,但是应用并没有读取数据的时候,接收方在回ack的时候就会将当前窗口大小-以接受但未读大小 发给发送方,发送方接受后会重新设置窗口大小

tcp不允许同时减少缓存和窗口       要先收缩窗口过段数据再减少缓存,要不会造成丢包现象

### 窗口关闭的潜在危险
当接受方向发送方发一个ack window 0的时候就是告诉他我关窗口了,然后发送方就啥也不干,等着接收方处理完后发给他一个ack告诉他我完事了你可以继续发了,但是假如这个ack丢失了,那发送方会一直等着,接收方也会一直等着,造成死锁的现象





### tcp是如何解决窗口关闭时潜在的死锁现象呢
为了解决这个问题tcp设有一个计时器当窗口为0时发送方启动计时器,当计时器超时后发送窗口探针给接收方,接收方收到后回复当前窗口大小,如果为0 继续启动计时器,否则就打破死锁,有点tcp当窗口探针三次为0 后就会发rst报文中断连接



### 糊涂窗口综合征
<font style="color:#DF2A3F;">指的是接收方只剩下很小的窗口大小了,发送方还接着发很小的数据甚至数据都比tcp+ip头部40字节还小了,造成了资源的浪费</font>

这个情况会发生在接收方和发送方

 	接收方可以通告一个小窗口

 发送方可以发送小数据

要解决糊涂窗口综合征就要同时解决上面两个问题

让接收方不通过小窗口给发送方

让发送方避免发送小数据

**问题1:如何让接收方不通过小窗口给发送方**

      当窗口大小小于min(MSS,缓存空间/2),也就是小于其中最小值的时候就会告诉发送方窗口为0,组织了对方发数据过来

等到接收方处理了一些数据之后,窗口大小>=MSS,或者有一半缓存空间可以用的时候就可以把窗口打开,让对方继续发

**问题2:怎样让接收方不通过小窗口给发送方?**

使用Nagle算法 ,思路是延迟处理,只有满足下面两个条件的任意一个才能发数据

<font style="color:#DF2A3F;">条件1:等到窗口大小>=mss 并且数据大小>=mss;</font>

<font style="color:#DF2A3F;">条件2:收到之前发送数据的ack回包</font>

****

## 拥塞控制
<font style="color:#DF2A3F;">流量控制是为了避免发送方填满接收方的缓存,流量控制是为了避免发送方填满,网络假如网络拥塞了,数据包发生时延丢失等  发过去迟迟收不到ack 那么发送方就会一直发,更加造成网络拥塞所以就有了拥塞控制</font>

**拥塞窗口(cwnd)**

   发送窗口(swnd)=min(   cwnd(拥塞窗口)  ,  rwnd(接收窗口)    );

网络没有拥塞 cwnd增大

网络发送拥塞  cwnd减小

**如何知道发生了网络拥塞呢?**

只要发出数据没有接受到ack应答报文,也就是发生了超时重传就可以认为发生了网络拥塞



**拥塞控制有哪些算法?**

四个:** **<font style="color:#1DC0C9;">慢启动 拥塞避免 拥塞发生 快速恢复</font>

****

### **慢启动**
<font style="color:#DF2A3F;">指的是tcp刚建立连接,发送方慢慢发数据   规则,每接受到一个ack,拥塞窗口(cwnd)就+1,</font>

每次cwnd变化是*2  1-2-4-8-16

第一次发1个       收到1个ack    cwnd+1=2

第二次发2个	   收到2个ack   cwnd+2=4

第二次发4个	   收到4个ack   cwnd+4=8

第二次发8个	   收到8个ack   cwnd+8=16

<font style="color:#DF2A3F;">2的n次方  指数倍增加</font>

**慢启动涨到什么时候才是个头?**

有一个叫 <font style="color:#DF2A3F;">慢启动门限ssthresh  </font>的状态变量

当  cwnd<ssthresh时,使用慢启动算法

当cwnd>=ssthresh时,就会使用拥塞避免算法

### 拥塞避免算法
一般来说 ssthresh大小为 65535字节

  拥塞避免算法 规则   :<font style="color:#DF2A3F;">每当收到1个ack时,cwnd增加1/cwnd ,就从慢启动的指数增长变为了线性增长</font>

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725281074846-71668a08-df78-44ee-b449-944edf30985c.png)

<font style="color:#DF2A3F;"></font>

但是这样还是会一直增长当增长到网络发生拥塞了就会进入拥塞发生算法

### 拥塞发生
当网络出现拥塞的时候,发生数据包重传,重传的机制有两种

超时重传  快重传



**发生超时重传时的拥塞发生算法**

<font style="color:#DF2A3F;">ssthresh  设为 cwnd/2</font>

<font style="color:#DF2A3F;">cwnd 重置为1  (恢复为初始化值,假设为1)    然后重新开始慢启动</font>

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725281329826-27716bfa-d258-40b3-b7fe-b172bdb8cb5b.png)

但是这种方式太激进了,反应太强烈了,突然减少数据流 会造成网络卡顿



### 发生快速重传的拥塞发生算法
发生快速重传的时候 也就是几个包只丢了一个所以tcp认为这种情况问题不大

因为大部分包没丢

cwnd=cwnd/2

ssthresh =cwnd





快速重传的触发条件是当接收方收到失序的报文段时，就立即发出重复确认，而不必等到自己发送数据时才进行捎带确认。如果发送方连续收到 3 个重复确认，就会立即重传对方尚未收到的报文段，而不必等待重传计时器超时。

快速恢复的触发条件是当发送方收到 3 个重复确认时，就执行快速恢复算法。





### 快速恢复
  快速重传和快速恢复算法一般同时使用,快速恢复认为还能收到几个重复的ack说明网络还不那么糟糕所以不想超时重传那么强烈



先快速重传

cwnd=cwnd/2

ssthresh =cwnd

然后快速恢复  cwnd =ssthresh +3  (+几就是几个包收到了  )

如果再收到重复的ack 那么cwnd+1

如果收到新数据的ack后  把cwnd设置为第一步中的ssthresh 的值,原因是该ack确认了新的数据说明 <font style="color:rgb(44, 62, 80);">duplicated ACK (重复ack)时的数据已经全部收到该恢复过程已经结束,可以恢复之前的状态了,再次进入拥塞避免后续继续进行线性增长</font>

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725282200461-474e2e58-3496-4601-aecb-3054ed0fa97a.png)

**<font style="color:rgb(44, 62, 80);background-color:rgb(243, 245, 247);">快速恢复算法过程中，为什么收到新的数据后，cwnd 设置回了 ssthresh ？</font>**

<font style="color:rgb(44, 62, 80);background-color:rgb(243, 245, 247);">首先根本目的是 发生快速重传拥塞后 降低cwnd来减缓拥塞 所以要设回ssthresh</font>

<font style="color:rgb(44, 62, 80);background-color:rgb(243, 245, 247);">其次 刚开始cwnd每次加1是为了快速把丢失的数据发给目标,从而解决快速重传的根本问题(三次相同ack导致的快速重传)所以cwnd先增大后减小</font>

## tcp半连接和全连接队列
在tcp三次握手的时候,linux内核会维护两个队列,

半连接队列, 也称SYN队列

全连接队列,也称accept队列

<font style="color:#DF2A3F;">服务端收到客户端发起的SYN请求后,</font><font style="color:#1DC0C9;">内核会把该链接存储到半连接队列</font><font style="color:#DF2A3F;">,并向客户端响应SYN+ACK,接着客户端会返回ACK请求,服务端收到第三次握手的ACK后,</font><font style="color:#1DC0C9;">内核会把连接从半连接队列移除,然后创建新的完全连接,并将其添加到accept队列,</font><font style="color:#DF2A3F;">等待进程调用accept函数时把连接取出来</font>

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725352180879-bb3b95e6-ffa9-41bf-a48e-0ff49c033acb.png)

#### 如果全连接队列溢出会怎么办?
当超过了tcp的最大全连接队列,服务器则会丢掉后续进来的tcp连接,丢掉的个数会被统计起来   ,会吹那服务的请求数量上不去的现象

linux有个参数可以指定当tcp全连接队列满了会使用什么策略来回应客户端

tcp_abort_on_overflow 两个值 0 1

0:server会扔掉client发过来的ack

1:满了server会发一个reset包给client,废掉这个握手过程和连接

#### 如何增大tcp全连接队列
tcp全连接取 min(somaxconn(linux内核默认参数128)    backlog (nginx默认511) )

backlog 是listen(int sockfd ,int backlog)

#### 半连接队列溢出
客户端一直发syn并不回应第三次握手的ack造成服务器有大量处于SYN_RECV状态的tcp连接  这就是所谓的SYN洪范,SYN攻击,DDOS攻击

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725356689606-eea5cbeb-10c0-4e9e-b818-1112137927b2.png)

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725356717229-637e9402-b62c-4efe-a513-d46cf3b080f5.png)

![](https://cdn.nlark.com/yuque/0/2024/png/40598760/1725356771001-6a2ed331-ed12-42ca-94fe-8e2586818f88.png)

