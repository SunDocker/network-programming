# 网络编程概述

## 1 Java与Internet

Java是Internet上的语言，它从语言级上提供了对网络应用程序的支持，程序员能够很容易开发常见的网络应用程序。
Java提供的网络类库，可以实现无痛的网络连接，**联网的底层细节被隐藏在Java的本机安装系统里**，由JVM进行控制。并且Java实现了一个**跨平台**的网络库，**程序员面对的是一个统一的网络编程环境**。

## 2 网络基础

计算机网络：把分布在不同地理区域的计算机与专门的外部设备用通信线路互连成一个规模大、功能强的网络系统，从而使众多的计算机可以方便地互相传递信息、共享硬件、软件、数据信息等资源。

网络编程的目的：直接或间接地通过**网络协议**与其它计算机实现**数据交换**，进行通讯。

网络编程中有两个主要的问题：

- 如何准确地**<u>定位</u>**网络上一台或多台**主机**；定位主机上的特定的**应用**
- 找到主机后如何可靠高效地进行**<u>数据传输</u>**

# 网络通信要素

[网络通信要素](网络通信要素/README.md)

# TCP网络编程

## 1 客户端

1. 通过**`InetAddress`和端口号构造`Socket`**
   - 直接用`Socket`的构造器就行
   - 注意这个`InetAddress`是指明通信对方的地址
2. 通过`Socket`实例的`getOutputStream()`方法获得流对象，输出数据
3. 关闭流和`Socket`资源

## 2 服务端

1. 指定**端口号**，**构造`ServerSocket`**

   > 不需要指明IP地址了，因为就是本地

   > 其实我个人更倾向于叫它`SocketServer`，用于监听端口并接收`Socket`的`Server`

2. 调用`ServerSocket`的实例的`accept()`方法接收来自服务端的`Socket`实例

3. 通过`Socket`实例的`getInputStream()`方法获得流对象，接收数据

   > 关于缓冲区的问题：如果包含中文字符的话，使用字节流和单个`byte[]`的方式读取，再用`byte[]`数组生成字符串，有可能出现字符编码被扩散的情况
   > 解决方法：使用`ByteArrayOutputStream`，先将`byte[]`中读取到的数据写入进去，然后再用其`toString()`方法整体转化为字符串
   >
   > > 自己用`byte[]`数组来拼接也可以，但这不是有现成的API嘛；如果是接收到`byte[]`数组之后要立刻输出到其他地方，一段一段输出拼接，那也可以不用`ByteArrayOutputStream`

4. 关闭流和`Socket`资源

5. 关闭服务器`ServerSocket`资源

> :star:`Socket`的补充：
>
> - `Socket`只有真实的IP地址和端口上有服务时正常运行，不然会抛出异常
>
>   > 因为需要**三次握手**，一段时间内握手失败了就是异常。
>   >
>   > 所以如果是UDP协议的`DatagramSocket`，就没有这种问题
>
> - 可以调用`Socket`实例的`getInetAddress()`方法获得套接字中的`InetAddress`对象
>
>   > 可用于接收方查看发送方的IP
>
> - 当然也有`getPort()`方法，不过一般自己都知道在哪个端口

> :star::star:实现有响应的服务端
>
> 在上面那个简单的服务端基础上：
>
> - 服务端拿到`accept`的`socket`，直接调用其`getOutputStream`方法返回数据即可
> - 客户端这边，调用`socket`的`getInputStream()`方法，用同样的方法接收响应数据即可
> - :star:解决一个问题`InputStream`实例的`read`方法是一个**阻塞**式的方法，需要**明确的结束标志**才会结束，所以，客户端和服务端这两边都需应当调用`socket`的`shutdownOutputStream`方法表明输出结束，这样才会停止阻塞

# UDP网络编程

## 1 概述

类`DatagramSocket`和`DatagramPacket`实现了**基于UDP协议网络程序**。

UDP数据报通过数据报套接字`DatagramSocket`发送和接收，系统不保证UDP数据报一定能够安全送到目的地，也不能确定什么时候可以抵达。

`DatagramPacket`对象封装了**UDP数据报**，在数据报中包含了**发送端的IP地址和端口号**以及**接收端的IP地址和端口号**。

UDP协议中每个数据报都给出了完整的地址信息，因此无须建立发送方和接收方的连接。如同发快递包裹一样。

## 2 发送方

1. 用构造器创建`DatagramSocket`

   > 可以不传入参数，因为IP地址和端口号都要放在**UDP数据报**中

2. 用构造器创建用于发送数据的`DatagramPacket`，传入数据、IP地址、端口号

3. 调用`DatagramSocket`实例的`send(DatagramPacket)`方法发送数据

> 另外一种发送方式：
>
> - 在创建`DatagramSocket`后，调用其`connect`方法传入IP地址和端口号
>
>   > 调用这个方法后就意味着，此`socket`只能给固定的IP地址和端口号发送数据，而不能随着`DatagramPacket`中指明的IP地址和端口号而变化，这也称为***bound***
>
> - 之后再调用`DatagramSocket`实例的`send(DatagramPacket)`方法发送数据时，就可以不指明地址和端口了
>
>   > 如果又在`DatagramSocket`显示指明了地址和端口，那必须与`connect`方法中传入的一致，否则异常
>
> - 在调用过`connect`方法后，`getInnetAddress`和`getPort`方法就不再返回`null`和`-1`了，而就是`connect`方法中传入的值

## 3 接收方

1. 用构造器创建`DatagramSocket`，传入**端口号**
2. 用构造器创建用于接收数据的`DatagramPacket`，传入用于接收数据的`byte[]`即可
3. 调用`DatagramSocket`实例的`receive(DatagramPacket)`方法接收数据
4. 调用`DatagramSocket`实例的`getData()`方法获取其中的数据，调用`DatagramSocket`实例的`getLength()`方法获取其中数据的长度

# URL编程

## 1 概述

URL(Uniform Resource Locator)：统一资源定位符，它表示Internet上某一资源的地址。

它是一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何locate这个资源。

通过URL我们可以访问Internet上的各种网络资源，比如最常见的www, ftp站点。浏览器通过解析给定的URL可以在网络上查找相应的文件或其他资源。

URL的基本结构由5部分组成：

- `<传输协议>://<主机名>:<端口号>/<文件路径>#片段名?参数列表`

  > 例如:
  > http://192.168.1.100:8080/helloworld/index.jsp#a?username=shkstart&password=123
  >
  > - #片段名：即锚点，例如看小说，直接定位到章节
  > - 参数列表格式:参数名=参数值&参数名=参数值.....

## 2 URL对象

创建：`new URL("url")`

常用方法：

- `url.getProtocol()`：协议名
- `url.getHost()`：主机名
- `url.getPort()`：端口号
- `url.getPath()`：文件路径
- `url.getFile()`：文件名（端口号后面的都算）
- `url.getQuery()`：参数列表

## 3 获取URL静态资源

1. 创建`URL`对象
2. 通过`URL`对象的`openConnection()`方法获取`URLConnection`实例
   - 如果使用的是HTTP协议，可以直接把这个对象强转为`HTTPURLConnection`
3. 调用`URLConnection`实例的`connect()`方法连接至服务器
4. 调用`URLConnection`实例的`getInputStream()`方法获取连接的输入流
5. 调用`URLConnection`实例的`disconnect()`方法断开连接

## 4 发送HTTP请求

> 主要讲的是GET请求和POST请求

1. 创建`URL`对象

2. 通过`URL`对象的`openConnection()`方法获取`URLConnection`实例，并直接把这个对象强转为`HTTPURLConnection`

3. 通过`URL`对象的`setRequestMethod("GET/POST")`方法设置请求方式

   - 默认是`GET`请求类型

4. 通过`URL`对象的方法设置读取和连接超时时长

   - `setReadTimeout(timeout)`
   - `setConnectTimeout(timeout)`

5. （POST请求）通过`URL`对象的方法设置允许对外输出

   - `setDoOutput(true)`
   - `setInstanceFollowRedirects(true)`

6. （POST请求）通过`URL`对象的`getOutputStream()`方法得到输出流，输出数据

7. 通过`URL`对象的`getResponseCode()`方法获取响应码，并判断请求是否成功

   > 在这个方法开始执行时，才在物理层面向服务器发起了请求

8. 调用`URLConnection`实例的`getInputStream()`方法获取连接的输入流

9. 调用`URLConnection`实例的`disconnect()`方法断开连接







