#netty入门之Hello World

###安装 IntelliJ 

	具体安装办法，这里不赘述
	
###创建 maven 工程

	根据向导创建 maven 工程
	groupId 填写 com
	artifactId 填写	rabbit.netty.example.echoServer
	点击完成后，工程创建成功，当前目录下没有java源文件，只有一个 pom.xml 文件，具体内容如下:
	
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com</groupId>
    <artifactId>rabbit.netty.example.echoServer</artifactId>
    <version>1.0-SNAPSHOT</version>

</project>
```

###添加 netty 依赖

添加完 netty 依赖后，pom.xml 文件如下:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com</groupId>
    <artifactId>rabbit.netty.example.echoServer</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.0.36.Final</version>
        </dependency>
    </dependencies>

</project>
```

>注意，不需要去查找 netty 使用的版本，由于 IntellJ 的自动补全功能，先输入 dependencies，然后输入 dependency，groupId 和 artifactId 会自动填写，然后在 groupId 中输入 io.netty（这个也有提示），然后 version 会自动提示选择需要使用的版本

###创建一个Handler

```
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelHandler;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.util.CharsetUtil;


/**
 * Created by liuzebo on 16/8/23.
 */
@ChannelHandler.Sharable
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //super.channelRead(ctx, msg);
        ByteBuf in = (ByteBuf)msg;
        System.out.println("Server received : " + in.toString(CharsetUtil.UTF_8));
        ctx.write(in);
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("Server closed : " + ctx.channel().remoteAddress());
        super.channelInactive(ctx);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        //super.channelReadComplete(ctx);
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER).addListener(ChannelFutureListener.CLOSE);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //super.exceptionCaught(ctx, cause);
        cause.printStackTrace();
        ctx.close();
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("Server received connection : " + ctx.channel().remoteAddress());
        super.channelActive(ctx);
    }
}

```

>注意，当 extends 继承 ChannelInboundHandlerAdapter 后，输入 @Override 会自动提示需要重载哪个成员函数，上下键选择对应的函数即可


###创建主类

```
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;

import java.net.Inet4Address;
import java.net.InetSocketAddress;

/**
 * Created by liuzebo on 16/8/23.
 */
public class echoServer {
    private final int port;

    public echoServer(int port) {
        this.port = port;
    }

    public static void main(String[] args) throws Exception {
        int port = 5058;
        if (args.length != 1) {
            System.out.println("Usage : " + echoServer.class.getSimpleName() + " <port>");
            //return;
        } else {
            port = Integer.parseInt(args[0]);
        }

        new echoServer(port).start();
    }

    public void start() throws Exception {
        NioEventLoopGroup group = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(group).channel(NioServerSocketChannel.class)
                    .localAddress(new InetSocketAddress(port) /*new InetSocketAddress(port)*/)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new EchoServerHandler());
                        }
                    });
            ChannelFuture f = b.bind().sync();
            System.out.println(echoServer.class.getName() + " start listen on " + f.channel().localAddress());
            f.channel().closeFuture().sync();
        } finally {
            System.out.println("error!");
            group.shutdownGracefully();
        }

    }
}

```

###运行主类