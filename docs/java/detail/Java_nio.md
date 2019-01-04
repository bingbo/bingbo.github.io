

## Java NIO几个核心部分

* Channel
* Buffer
* Selector

## IO和NIO的区别

* IO 基于流(Stream oriented), 而 NIO 基于 Buffer (Buffer oriented)

	在一般的 Java IO 操作中, 我们以流式的方式顺序地从一个 Stream 中读取一个或多个字节, 因此我们也就不能随意改变读取指针的位置

	而 基于 Buffer 就显得有点不同了. 我们首先需要从 Channel 中读取数据到 Buffer 中, 当 Buffer 中有数据后, 我们就可以对这些数据进行操作了. 不像 IO 那样是顺序操作, NIO 中我们可以随意地读取任意位置的数据

* IO 操作是阻塞的, 而 NIO 

	Java 提供的各种 Stream 操作都是阻塞的, 例如我们调用一个 read 方法读取一个文件的内容, 那么调用 read 的线程会被阻塞住, 直到 read 操作完成.而 NIO 的非阻塞模式允许我们非阻塞地进行 IO 操作. 例如我们需要从网络中读取数据, 在 NIO 的非阻塞模式中, 当我们调用 read 方法时, 如果此时有数据, 则 read 读取并返回; 如果此时没有数据, 则 read 直接返回, 而不会阻塞当前线程

* IO 没有 selector 概念, 而 NIO 有 selector 概念.

	它是 Java NIO 之所以可以非阻塞地进行 IO 操作的关键.通过 Selector, 一个线程可以监听多个 Channel 的 IO 事件, 当我们向一个 Selector 中注册了 Channel 后, Selector 内部的机制就可以自动地为我们不断地查询(select) 这些注册的 Channel 是否有已就绪的 IO 事件(例如可读, 可写, 网络连接完成等). 通过这样的 Selector 机制, 我们就可以很简单地使用一个线程高效地管理多个 Channel了

## Buffer

NIO Buffer 其实是这样的内存块的一个封装, 并提供了一些操作方法让我们能够方便地进行数据的读写.

Buffer 类型有:

* ByteBuffer
* CharBuffer
* DoubleBuffer
* FloatBuffer
* IntBuffer
* LongBuffer
* ShortBuffer这些 Buffer 覆盖了能从 IO 中传输的所有的 Java 基本数据类型.

### 使用 NIO Buffer 的步骤如下:

* 将数据写入到 Buffer 中.
* 调用 Buffer.flip()方法, 将 NIO Buffer 转换为读模式.
* 从 Buffer 中读取数据
* 调用 Buffer.clear() 或 Buffer.compact()方法, 将 Buffer 转换为写模式.

### Buffer 属性

#### Capacity

一个内存块会有一个固定的大小, 即容量(capacity), 我们最多写入capacity 个单位的数据到 Buffer 中, 例如一个 DoubleBuffer, 其 Capacity 是100, 那么我们最多可以写入100个 double 数据.

#### Position

当从一个 Buffer 中写入数据时, 我们是从 Buffer 的一个确定的位置(position)开始写入的. 在最初的状态时, position 的值是0. 每当我们写入了一个单位的数据后, position 就会递增一.
当我们从 Buffer 中读取数据时, 我们也是从某个特定的位置开始读取的. 当我们调用了 filp()方法将 Buffer 从写模式转换到读模式时, position 的值会自动被设置为0, 每当我们读取一个单位的数据, position 的值递增1.
position 表示了读写操作的位置指针.

#### limit

limit - position 表示此时还可以写入/读取多少单位的数据.
例如在写模式, 如果此时 limit 是10, position 是2, 则表示已经写入了2个单位的数据, 还可以写入 10 - 2 = 8 个单位的数据.

### Direct Buffer 和 Non-Direct Buffer 的区别

#### Direct Buffer:

所分配的内存不在 JVM 堆上, 不受 GC 的管理.(但是 Direct Buffer 的 Java 对象是由 GC 管理的, 因此当发生 GC, 对象被回收时, Direct Buffer 也会被释放)
因为 Direct Buffer 不在 JVM 堆上分配, 因此 Direct Buffer 对应用程序的内存占用的影响就不那么明显(实际上还是占用了这么多内存, 但是 JVM 不好统计到非 JVM 管理的内存.)
申请和释放 Direct Buffer 的开销比较大. 因此正确的使用 Direct Buffer 的方式是在初始化时申请一个 Buffer, 然后不断复用此 buffer, 在程序结束后才释放此 buffer.
使用 Direct Buffer 时, 当进行一些底层的系统 IO 操作时, 效率会比较高, 因为此时 JVM 不需要拷贝 buffer 中的内存到中间临时缓冲区中.

#### Non-Direct Buffer:

直接在 JVM 堆上进行内存的分配, 本质上是 byte[] 数组的封装.
因为 Non-Direct Buffer 在 JVM 堆中, 因此当进行操作系统底层 IO 操作中时, 会将此 buffer 的内存复制到中间临时缓冲区中. 因此 Non-Direct Buffer 的效率就较低.

### 示例

_参考：_ http://www.ibm.com/developerworks/cn/education/java/j-nio/index.html

```java
@Test
public void testBuffer() {
    //分配指定大小的缓冲区
    IntBuffer buffer = IntBuffer.allocate(10);
    //包装一个现有的数组
    int[] arr = new int[10];
    IntBuffer buffer1 = IntBuffer.wrap(arr);

    this.displayBuffer(buffer);
    for (int i = 0; i < 10; i++) {
        buffer.put(i);
    }
    this.displayBuffer(buffer);
    //把limit设为当前位置position，position设为0
    this.displayBuffer(buffer.flip());
    while (buffer.hasRemaining()) {
        System.out.print(buffer.get());
    }
    this.displayBuffer(buffer);
    //limit不变，position设为0
    this.displayBuffer(buffer.rewind());
    this.displayBuffer(buffer.limit(5));
    //从头开始，把limit设为capacity,position设为0
    this.displayBuffer(buffer.clear());
    this.displayBufferData(buffer);
    this.displayBuffer(buffer);

    //创建子缓冲区，子缓冲区的数据与原缓冲区数据是共享的，修改子缓冲区数据，原缓冲区的部分数据也会发生变化
    buffer.position(3);
    buffer.limit(7);
    IntBuffer sliceBuffer = buffer.slice();
    displayBufferData(sliceBuffer);

    //创建只读缓冲区,与原缓冲区一样数据共享，但只能读
    IntBuffer readonlyBuffer = buffer.asReadOnlyBuffer();

}


@Test
public void testDirectBuffer() throws IOException {
    FileInputStream fis = new FileInputStream(this.infile);
    FileChannel inChannel = fis.getChannel();

    FileOutputStream fos = new FileOutputStream(this.outfile);
    FileChannel outChannel = fos.getChannel();


    /**
     * 分配一个直接缓冲区
     * 给定一个直接字节缓冲区，Java虚拟机将尽最大努 力直接对它执行本机I/O操作。
     * 也就是说，它会在每一次调用底层操作系统的本机I/O操作之前(或之后)，尝试避免将缓冲区的内容拷贝到一个中间缓冲区中 或者从一个中间缓冲区中拷贝数据
     */
    ByteBuffer buffer = ByteBuffer.allocateDirect(1024);

    while (true) {
        //先清空缓冲区
        buffer.clear();
        int r = inChannel.read(buffer);
        if (r == -1) {
            break;
        }
        //从头开始
        buffer.flip();
        outChannel.write(buffer);
    }

}

@Test
public void testMappedBuffer() throws IOException {
    RandomAccessFile accessFile = new RandomAccessFile(this.infile,"rw");
    FileChannel fileChannel = accessFile.getChannel();

    /**
     * 可以锁定文件的一部分,
     * 要获取文件的一部分上的锁，您要调用一个打开的 FileChannel 上的 lock() 方法。
     * 注意，如果要获取一个排它锁，您必须以写方式打开文件。
     */
    FileLock lock=fileChannel.lock();
    lock.release();

    /**
     * 内存映射文件 I/O 是通过使文件中的数据神奇般地出现为内存数组的内容来完成的。
     * 这其初听起来似乎不过就是将整个文件读到内存中，但是事实上并不是这样。
     * 一般来说，只有文件中实际读取或者写入的部分才会送入（或者 映射 ）到内存中
     */
    MappedByteBuffer mappedByteBuffer = fileChannel.map(FileChannel.MapMode.READ_WRITE, 0, 1024);
    byte[] ss = new byte[1024];
    mappedByteBuffer.get(ss);
    System.out.println(new String(ss));
    fileChannel.close();

}

private void displayBuffer(Buffer buffer) {
    System.out.println("capacity: " + buffer.capacity());
    System.out.println("position: " + buffer.position());
    System.out.println("limit: " + buffer.limit());
    System.out.println("-------------------------");

}

private void displayBufferData(IntBuffer buffer) {
    while (buffer.hasRemaining()) {
        System.out.print(buffer.get());
    }
    System.out.println();
}
```

## Channel

Channel 类型有:

* FileChannel, 文件操作
* DatagramChannel, UDP 操作
* SocketChannel, TCP 操作
* ServerSocketChannel, TCP 操作, 使用在服务器端.这些通道涵盖了 UDP 和 TCP网络 IO以及文件 IO.

### FileChannel

FileChannel 是操作文件的Channel, 我们可以通过 FileChannel 从一个文件中读取数据, 也可以将数据写入到文件中.

> 注意, FileChannel 不能设置为非阻塞模式.

```java
//打开 FileChannel
RandomAccessFile aFile     = new RandomAccessFile("test.txt", "rw");
FileChannel      inChannel = aFile.getChannel();

//从 FileChannel 中读取数据
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);

//写入数据
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}

//当我们对 FileChannel 的操作完成后, 必须将其关闭
channel.close();

//设置 position
long pos channel.position();
channel.position(pos +123);

//获取文件大小,注意, 这里返回的是文件的大小, 而不是 Channel 中剩余的元素个数.
channel.size();

//截断文件
channel.truncate(1024);

//可以强制将缓存的未写入的数据写入到文件中:
channel.force(true);

### SocketChannel

SocketChannel 是一个客户端用来进行 TCP 连接的 Channel.创建一个 SocketChannel 的方法有两种:

* 打开一个 SocketChannel, 然后将其连接到某个服务器中
* 当一个 ServerSocketChannel 接受到连接请求时, 会返回一个 SocketChannel 对象.

```java
//打开 SocketChannel
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://example.com", 80));

//关闭
socketChannel.close();

//读取数据,如果 read()返回 -1, 那么表示连接中断了.
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = socketChannel.read(buf);

//写入数据
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}

//设置 SocketChannel 为异步模式, 这样我们的 connect, read, write 都是异步的了
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("http://example.com", 80));
//在异步模式中, 或许连接还没有建立, connect 方法就返回了, 因此我们需要检查当前是否是连接到了主机, 因此通过一个 while 循环来判断
while(! socketChannel.finishConnect() ){
    //wait, or do something else...    
}

### ServerSocketChannel

```java
打开 关闭
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.socket().bind(new InetSocketAddress(9999));
while(true){
//使用ServerSocketChannel.accept()方法来监听客户端的 TCP 连接请求, accept()方法会阻塞, 直到有连接到来, 当有连接时, 这个方法会返回一个 SocketChannel 对象
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}
```

```java
//非阻塞模式下, accept()是非阻塞的, 因此如果此时没有连接到来, 那么 accept()方法会返回null
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));
serverSocketChannel.configureBlocking(false);

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    if(socketChannel != null){
        //do something with socketChannel...
        }
}
```


### DatagramChannel

DatagramChannel 是用来处理 UDP 连接的.

```java
//打开
DatagramChannel channel = DatagramChannel.open();
channel.socket().bind(new InetSocketAddress(9999))

//读取数据
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();

channel.receive(buf);

//发送数据
String newData = "New String to write to file..."
                    + System.currentTimeMillis();
    
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();

int bytesSent = channel.send(buf, new InetSocketAddress("example.com", 80));

/*
*连接到指定地址
*因为 UDP 是非连接的, 因此这个的 connect 并不是向 TCP 一样真正意义上的连接, 而是它会讲 DatagramChannel 锁住, 因此我们仅仅可以从指定的地址中读取或写入数据.
*/

channel.connect(new InetSocketAddress("example.com", 80));
```

## Selector

Selector 允许一个单一的线程来操作多个 Channel. 如果我们的应用程序中使用了多个 Channel, 那么使用 Selector 很方便的实现这样的目的, 但是因为在一个线程中使用了多个 Channel, 因此也会造成了每个 Channel 传输效率的降低.

为了使用 Selector, 我们首先需要将 Channel 注册到 Selector 中, 随后调用 Selector 的 select()方法, 这个方法会阻塞, 直到注册在 Selector 中的 Channel 发送可读写事件. 当这个方法返回后, 当前的这个线程就可以处理 Channel 的事件了

### 使用步骤

#### 创建选择器

```java
//通过 Selector.open()方法, 我们可以创建一个选择器:
Selector selector = Selector.open();
```

#### 将Channel注册到选择器中

为了使用选择器管理 Channel, 我们需要将 Channel 注册到选择器中

```java
channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

> 注意, 如果一个 Channel 要注册到 Selector 中, 那么这个 Channel 必须是非阻塞的, 即channel.configureBlocking(false);
因为 Channel 必须要是非阻塞的, 因此 FileChannel 是不能够使用选择器的, 因为 FileChannel 都是阻塞的

注意到, 在使用 Channel.register()方法时, 第二个参数指定了我们对 Channel 的什么类型的事件感兴趣, 这些事件有:

* Connect, 即连接事件(TCP 连接), 对应于SelectionKey.OP_CONNECT
* Accept, 即确认事件, 对应于SelectionKey.OP_ACCEPT
* Read, 即读事件, 对应于SelectionKey.OP_READ, 表示 buffer 可读.
* Write, 即写事件, 对应于SelectionKey.OP_WRITE, 表示 buffer 可写.

一个 Channel发出一个事件也可以称为 对于某个事件, Channel 准备好了. 因此一个 Channel 成功连接到了另一个服务器也可以被称为 connect ready.
我们可以使用或运算|来组合多个事件, 例如:

```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

> 注意, 一个 Channel 仅仅可以被注册到一个 Selector 一次, 如果将 Channel 注册到 Selector 多次, 那么其实就是相当于更新 SelectionKey 的 interest set. 例如:

```java
channel.register(selector, SelectionKey.OP_READ);
channel.register(selector, SelectionKey.OP_READ | SelectionKey.OP_WRITE);
```

上面的 channel 注册到同一个 Selector 两次了, 那么第二次的注册其实就是相当于更新这个 Channel 的 interest set 为 SelectionKey.OP_READ | SelectionKey.OP_WRITE

### SelectionKey

当我们使用 register 注册一个 Channel 时, 会返回一个 SelectionKey 对象, 这个对象包含了如下内容:

* interest set, 即我们感兴趣的事件集, 即在调用 register 注册 channel 时所设置的 interest set.
* ready set
* channel
* selector
* attached object, 可选的附加对象

#### interest set

我们可以通过如下方式获取 interest set:

```java
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept  = interestSet & SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;  
```

#### ready set

代表了 Channel 所准备好了的操作.我们可以像判断 interest set 一样操作 Ready set, 但是我们还可以使用如下方法进行判断:

```java
int readySet = selectionKey.readyOps();

selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```

#### Channel 和 Selector

我们可以通过 SelectionKey 获取相对应的 Channel 和 Selector:

```java
Channel  channel  = selectionKey.channel();
Selector selector = selectionKey.selector();
```

#### Attaching Object

我们可以在selectionKey中附加一个对象:

```java
selectionKey.attach(theObject);
Object attachedObj = selectionKey.attachment();
```

或者在注册时直接附加:

```java
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
```

### 通过 Selector 选择 Channel

我们可以通过 Selector.select()方法获取对某件事件准备好了的 Channel, 即如果我们在注册 Channel 时, 对其的可写事件感兴趣, 那么当 select()返回时, 我们就可以获取 Channel 了.

> 注意, select()方法返回的值表示有多少个 Channel 可操作.

### 获取可操作的 Channel

如果 select()方法返回值表示有多个 Channel 准备好了, 那么我们可以通过 Selected key set 访问这个 Channel:

```java
Set<SelectionKey> selectedKeys = selector.selectedKeys();

Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

while(keyIterator.hasNext()) {
    
    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.

    } else if (key.isConnectable()) {
        // a connection was established with a remote server.

    } else if (key.isReadable()) {
        // a channel is ready for reading

    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
}
```

> 注意, 在每次迭代时, 我们都调用 "keyIterator.remove()" 将这个 key 从迭代器中删除, 因为 select() 方法仅仅是简单地将就绪的 IO 操作放到 selectedKeys 集合中, 因此如果我们从 selectedKeys 获取到一个 key, 但是没有将它删除, 那么下一次 select 时, 这个 key 所对应的 IO 事件还在 selectedKeys 中.
例如此时我们收到 OP_ACCEPT 通知, 然后我们进行相关处理, 但是并没有将这个 Key 从 SelectedKeys 中删除, 那么下一次 select() 返回时 我们还可以在 SelectedKeys 中获取到 OP_ACCEPT 的 key.

> 注意, 我们可以动态更改 SekectedKeys 中的 key 的 interest set. 例如在 OP_ACCEPT 中, 我们可以将 interest set 更新为 OP_READ, 这样 Selector 就会将这个 Channel 的 读 IO 就绪事件包含进来了

### Selector 的基本使用流程

1. 通过 Selector.open() 打开一个 Selector.
1. 将 Channel 注册到 Selector 中, 并设置需要监听的事件(interest set)
1. 不断重复:

	* 调用 select() 方法
	* 调用 selector.selectedKeys() 获取 selected keys
	* 迭代每个 selected key:

		* 从 selected key 中获取 对应的 Channel 和附加信息(如果有的话)
		* 判断是哪些 IO 事件已经就绪了, 然后处理它们. 如果是 OP_ACCEPT 事件, 则调用 "SocketChannel clientChannel = ((ServerSocketChannel) key.channel()).accept()" 获取 SocketChannel, 并将它设置为 非阻塞的, 然后将这个 Channel 注册到 Selector 中.
		* 根据需要更改 selected key 的监听事件.
		* 将已经处理过的 key 从 selected keys 集合中删除.

### 关闭 Selector

当调用了 Selector.close()方法时, 我们其实是关闭了 Selector 本身并且将所有的 SelectionKey 失效, 但是并不会关闭 Channel.

### Selector 例子

```java
package com.ibingbo.nio;

import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;

import static java.nio.channels.SelectionKey.OP_READ;

public class NioServer {
    private static final int BUF_SIZE = 256;
    private static final int TIMEOUT = 3000;

    public static void main(String args[]) throws Exception {
        // 打开服务端 Socket
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

        // 打开 Selector
        Selector selector = Selector.open();

        // 服务端 Socket 监听8080端口, 并配置为非阻塞模式
        serverSocketChannel.socket().bind(new InetSocketAddress(8088));
        serverSocketChannel.configureBlocking(false);

        // 将 channel 注册到 selector 中.
        // 通常我们都是先注册一个 OP_ACCEPT 事件, 然后在 OP_ACCEPT 到来时, 再将这个 Channel 的 OP_READ
        // 注册到 Selector 中.
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        while (true) {
            // 通过调用 select 方法, 阻塞地等待 channel I/O 可操作
            if (selector.select(TIMEOUT) == 0) {
                System.out.print(".");
                continue;
            }

            // 获取 I/O 操作就绪的 SelectionKey, 通过 SelectionKey 可以知道哪些 Channel 的哪类 I/O 操作已经就绪.
            Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();

            while (keyIterator.hasNext()) {

                SelectionKey key = keyIterator.next();

                // 当获取一个 SelectionKey 后, 就要将它删除, 表示我们已经对这个 IO 事件进行了处理.
                keyIterator.remove();

                if (key.isAcceptable()) {
                    // 当 OP_ACCEPT 事件到来时, 我们就有从 ServerSocketChannel 中获取一个 SocketChannel,
                    // 代表客户端的连接
                    // 注意, 在 OP_ACCEPT 事件中, 从 key.channel() 返回的 Channel 是 ServerSocketChannel.
                    // 而在 OP_WRITE 和 OP_READ 中, 从 key.channel() 返回的是 SocketChannel.
                    SocketChannel clientChannel = ((ServerSocketChannel) key.channel()).accept();
                    clientChannel.configureBlocking(false);
                    //在 OP_ACCEPT 到来时, 再将这个 Channel 的 OP_READ 注册到 Selector 中.
                    // 注意, 这里我们如果没有设置 OP_READ 的话, 即 interest set 仍然是 OP_CONNECT 的话, 那么 select 方法会一直直接返回.
                    clientChannel.register(key.selector(), OP_READ, ByteBuffer.allocate(BUF_SIZE));
                }

                if (key.isReadable()) {
                    SocketChannel clientChannel = (SocketChannel) key.channel();
                    ByteBuffer buf = (ByteBuffer) key.attachment();
                    long bytesRead = clientChannel.read(buf);
                    if (bytesRead == -1) {
                        clientChannel.close();
                    } else if (bytesRead > 0) {
                        key.interestOps(OP_READ | SelectionKey.OP_WRITE);
                        System.out.println("Get data length: " + bytesRead);
                    }
                }

                if (key.isValid() && key.isWritable()) {
                    ByteBuffer buf = (ByteBuffer) key.attachment();
                    buf.flip();
                    SocketChannel clientChannel = (SocketChannel) key.channel();

                    clientChannel.write(buf);

                    if (!buf.hasRemaining()) {
                        key.interestOps(OP_READ);
                    }
                    buf.compact();
                }
            }
        }
    }
}
```


