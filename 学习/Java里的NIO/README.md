# Java里的NIO

* 传统的BIO编程	
 网络编程基本模型是Client/Server模型，也就是2个进程间的相互通信，其中服务端提供位置信息（绑定Ip和监听端口），客户端通过连接操作向服务器监听的地址发起请求，通过三次握手建立连接，如果连接成功，双方就可以通过网络套接字进行通信。
 
 ![](http://47.95.12.0:3389/ftp/socket连接.png)  
 > 该模型最大的问题是可伸缩性太差，当客户端访问增加后，服务端的线程数也随之1:1的新增。
 
 * 伪异步I/O
   传统的BIO模型会导致服务端的线程数量随着客户端的连接数增加同比例的新增，我们可以利用Java的线程池，伪异步。这里也会有问题，如果客户端连接数很多，但是通信之间海阻塞的，这样很容易导致线程耗尽。
   
   
* NIO
> NIO从JDK1.4 开始引入的，我么可以称它为新I/O，但是我觉得叫他非阻塞I/O可能更体现它的本质。

	* 缓冲区Buffer
   
	   Buffer包含一些需要写入和读取的数据，在NIO类库里面加入Buffer对象，体现新库和老库的区别，在NIO库里面，所有的数据都是用缓冲区来处理的，在读取数据的时候它是从缓冲区来读取的，在写入的数据时，也是写到花冲区。缓冲区的本质是一个数组，通常是一个字节数组（ByteBuffer）
 
 * 通道Chanel	
	   打个比喻，它就像是水管，网络上的数据都是通过Channel来读取和写入。Channel相对于流	来说是全双工的。
   
* 多路复用Selector	
	   多路复用器提选择就绪的任务能力，简单来说，Selector会不断的轮训注册在Chanel，如果某个Channel上面发生读或者写事件，这个Channel 就处于就绪状态，会被Selector轮训出来，谈后通过SelectionKey可以获取就绪Channel 
	的集合，进行后续的I/O操作。
	   一个多路复用器可以轮训多个Channel  ，JDk使用的epoll()代替传统的select实现，所以它并没有连接最大句柄1024/2048的限制。这就是说一个线程负责Selector轮训，就可以实现成千上万的客户端的接入。
	   
	   
   * 服务端创建步骤
      1. 打开serverSocketChannel	
      ```java
       ServerSocketChannel	 ssc=ServerSocketCHannel.open();
       ```
      2. 绑定监听端口，设置连接为非阻塞模式	
      ```java
       ssc.socket().bind(new   InetSocketAddress(inteAddress.getByName("IP),port));
       ssc.configureBlocking(false);
       ```
      3. 创建Reactor线程，创建多苦复用器并启动线程。
      ```java
       Selector selector=Selector.open();
       New Thread(new ReactorTask()).start();
       ```
      4. 将ServerSocketChannel  注册到Selector上
      ```java
       SelectionKey key=scc.register(selector,SelectionKey.OP_ACCEPT,handle);   
       ```
      5. 多路复用器在线程run方法的无限循环体内轮询准备就绪的Key
	      ```java
	     int num=selector.select();
	     Set selectedKeys=selector.selectedkeys();
	     Iterator it=selectedKeys.iterator();
	     while(it.hasNex()){
	      SelectionKey key=(SelectionKey)it.next();
	      //...deal with I/O event...
	     }
	     ```
   	   6. 多路复用器监听到有闲的客户端接入，处理新的接入请求，完成三次TCP握手，建立物理链路
	   ```java
	   SocketChannel channel=scc.accept();
   		```
   		7. 设置客户端为非阻塞	
	  ```java 
	  channel.configureBlocking(false);
	   channel.socket.setReuseAddress(true);
	   ```
	   8. 将新接入的客户端连接注册到Reactor线程的多路复用器上，监听读操作，读取客户端发送的网络消息
	   ```java
	   SelectionKey key=ssc.register(selector,SelectionKey.OP_READ,handle);
	   ```
	   9. 异步读取客户端请求到缓冲区
	    int 	 readNumber=channel.read(receviverBuffer);
	   10. 对ByteBuffer进行编码，如果有半包消息指针reset，继续读取后续的斑纹，将解码成功的消息封装成Task，投递到业务线程池中，进行业务逻辑判断
	   ```java
	   Object message=null;
	   while(true){
	   byteBuffer.mark();
	   Object message=decode(byteBuffer);
	   if(null==message){
	    byteBuffer.reset();
	    break;
	   }
	   messagelist.add(message);
	   }
	   if(!byteBuffer.hasRemain)
	   {
	   byteBuffer.clear();
	   }
	   else{
	   byteBuffer.compact();
	   }
	   if(CollectionUtils.isNotEmpty(messageList)){
	   for(Object message:messagelist){
	   handertask(message);
	   }
	   }
	   ```
	   11. 将POJO对象encode成ByteBuffer，调用SocketChannel
	的异步wait方法将消息发送给客户端	
	```java
	socketChannel.wirte(buffer);
	```
	   
	    
  
   
	
         
	   
	   
   
   