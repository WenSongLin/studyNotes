1.实现多线程的方式?

```java
1.Thread类
    创建线程子类对象，start
2.实现Runnable接口  public void run()
    结合Thread的有参构造。 new Thread(Runnable r,)
3.实现Callable接口   call  vs run     public T call() throws Exception
    FutureTask task = new FutureTask();
    Thread threa = new Thread(task);
  或者
    ThreadPoolExecuter 
```



2. 线程安全?

   ```java
   线程之间有独立的本地内存。 内存不可见。----> 计算机里面 MESI  缓存一致性
   JMM:
      java内存模型----> 内存不可见   原子性  有序性 (计算机的处理器优化)
          限制处理器优化和解决内存屏障。
    1.synchornized/Lock
    2.volatile      
    3.并发包里面的类  JUC    CAS
          
    synchornized  vs     volatile?
          
     volatile:  (一写多读)
          可以保证并发里面的可见性  有序性  不能保证原子性  可以避免指令重排
              -- ++  int a=10
     synchornized:  可见性   有序性  原子性   效率低    
   ```

```java
加监视器对象:
  synchornized:
     1.同步代码块
     2.同步方法
         
  Lock.RentrantLock :  
   try{
      
   }finally{
      
   }       
```



3. wait  vs sleep？

```java
1.sleep 不会释放锁   wait: 自动释放锁
2.wait  一般用户线程通信(生产者+消费者)  sleep: 线程之间的调度
3.唤醒方式:
    wait() 只能notify/notifyAll唤醒  sleep() 超时自己醒过来
4.方法调用
    sleep()    在任意方法中都可以调用
    wait()/notify 必须结合   synchornized 
5.所属
  sleep--->Thread    wait--->Object        
```



4.  A  B  C三个线程

```java
控制线程的执行顺序:
    start--->join()  等待线程终止
```



# Socket

# 1. 计算机网络

```java
是指将地理位置不同的具有独立功能的多台计算机及其外部设备，通过通信线路连接起来，在网络操作系统，网络管理软件及网络通信协议的管理和协调下，实现资源共享和信息传递的计算机系统。 
```

```java
使用远程资源 
共享信息、程序和数据 
分布处理 
```

```java
计算机网络信息交互方式:
   架构设计: 
      B/S： browser/server  浏览器/服务器   淘宝  京东 
      c/s:  client/server    客户端/服务器  基于pc端   
```



# 2. 组成部分

## 1. 协议

```java
规则。
整个计算机网络通信，基于七层模型进行设计的。 每一层都有他自己的协议。
  通过程序实现网络数据交互，传输层。 
TCP/IP: 传输控制协议/互联网协议  
    http: 超文本传输协议  默认端口80       http://www.baidu.com 
    https: 加密规则(安全证书) 默认端口443
    ftp: 文件传输协议  21
    sftp:
    pop3: 邮箱协议  

UDP: 用户数据报协议、      
```



```java
TCP/IP:  
  1.安全可靠的连接  无状态  生活中的打电话。
  2.传输数据的方式 IO 读写数据
  3.内容大小  没有限定
  4.效率 慢
  5.场景
      B/S c/s mysql 
      
       
   http:
      三次握手/四次握手   客户端请求和服务端的成功的响应。
 http://www.baidu.com  百度首页  百度服务没有开启 
 
UDP:
   1. 不安全的协议。 生活中的发短信。
   2. 传输数据的方式 数据包
   3. 数据包大小64kb  
   4. 效率较快  
   5. 场景
        pc端的软件。
  QQ:
   A: UDP+TCP        
```



## 2. 端口

```java
访问百度的资源:
   1.百度服务器---> 一台计算机--->计算机里面的一个软件(应用程序)
   2.http://www.baidu.com  --->首页  http://ip:端口
     www.baidu.com域名   0-65535    0-1023 系统保留使用的端口号。
         8080  tomcat
         3306  mysql
         1521  oracle

计算机:
  安装了很多软件。 应用软件。 ip:端口
```





## 3. IP地址

```java
每台计算机都有ip地址   32位或者128位的无符号的数字组成的
    ipconfig
本地链接 IPv6 地址. . . . . . . . : fe80::a811:5ea3:f814:61dd%6
    
IPv4 地址 . . . . . . . . . . . . : 192.168.14.113    
ipv4:
  255.255.255.255
  ip分了4个段:
   A  B  C  D     
```



# 3. 常用类

## 1. InetAddress

```java
public class InetAddress
extends Object
implements Serializable
此类表示Internet协议（IP）地址。 
IP地址是由IP使用的32位或128位无符号数字
    
 Inet4Address ， Inet6Address 
    
// C:\Windows\System32\drivers\etc\hosts    
```

```java
 private static void demo3() {
        //获得ipv6的信息
//        try {
//            System.out.println(Inet6Address.getLocalHost());//默认获得的还是ipv4
//        } catch (UnknownHostException e) {
//            e.printStackTrace();
//        }

        try {
            //的IP地址列表
            Enumeration<NetworkInterface> interfaces = NetworkInterface.getNetworkInterfaces();
            //与Iterator
            while (interfaces.hasMoreElements()) {
                NetworkInterface anInterface = interfaces.nextElement();
                Enumeration<InetAddress> addresses = anInterface.getInetAddresses();
                while (addresses.hasMoreElements()) {
                    InetAddress inetAddress = addresses.nextElement();
                    if (inetAddress instanceof Inet6Address) {
                        System.out.println("ipv6:" + inetAddress);
                    } else {
                        System.out.println("ipv4:" + inetAddress);
                    }
                }
                System.out.println("----------------------------");
            }

        } catch (SocketException e) {
            e.printStackTrace();
        }


    }

    private static void demo2() {
        //锁定局域网范围之内的任意一台的计算机
        try {
            InetAddress name = InetAddress.getByName("");
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
    }

    private static void demo1() {
        //获得计算机的ip地址
        try {
            InetAddress localHost = InetAddress.getLocalHost();//本机的ip地址

            System.out.println(localHost);
            //Lisa/192.168.14.113
            // 127.0.0.1 所有的计算机里面都具备的ip  一般测试
            //127.0.0.1  localhost

            System.out.println(localHost.getHostAddress());
            System.out.println(localHost.getHostName());
            System.out.println(Arrays.toString(localHost.getAddress()));

        } catch (UnknownHostException e) {
            e.printStackTrace();
        }

    }
```





## 2. URL

```java
数据交互路径。指向万维网上的“资源”的指针
统一定位符。  
 协议://ip地址:端口名称/资源/资源/.../index.html   

URL(String spec) 
```

```java
 private static void demo1() {

        String path = "http://www.baidu.com";
        try {
            URL url = new URL(path);
            System.out.println(url.getProtocol());//协议
            System.out.println(url.getPort());//端口 -1
            System.out.println(url.getDefaultPort());//看协议  80

            System.out.println(url.getHost());
            System.out.println(url.getAuthority());

            System.out.println(url.getQuery());// ?后面的内容  参数

            System.out.println(url.getFile());

            System.out.println(url.getRef());//#后面的内容 锚记  <a>
            System.out.println(url.getContent());//sun.net.www.protocol.http.HttpURLConnection$HttpInputStream@3d075dc0

            System.out.println(url.openStream());

            URLConnection connection = url.openConnection();
            connection.connect();
            System.out.println(connection.getInputStream());

        } catch (IOException e) {
            e.printStackTrace();
        }

    }
```



> 调用服务接口: 获得特定省份的城市的天气数据。

```java
 private static void demo2() {
        int cityId = 101010100;
        getWeatherInfo(cityId);
    }

    //获得指定的天气的数据
    private static void getWeatherInfo(int cityId) {
        String path = "http://t.weather.itboy.net/api/weather/city/" + cityId;
        BufferedReader reader = null;
        try {
            URL url = new URL(path);
            //读到网络数据--->字节输入流
            URLConnection connection = url.openConnection();
            connection.connect();
            //read()  read(byte[] bytes)
            //网络数据一般只有1行--->读取一行就ok 高效字符流
            reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));

            String info = reader.readLine();
            System.out.println(info);
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                if(reader!=null) reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```



> 有2种结果:  成功   失败

```json
{"message":"CityId101050600不在返回之内,加QQ群：608222884，反馈问题。","status":403}
```

```json
{
    "message": "success感谢又拍云(upyun.com)提供CDN赞助",
    "status": 200,
    "date": "20210412",
    "time": "2021-04-12 14:09:05",
    "cityInfo": {
        "city": "朝阳区",
        "citykey": "101010300",
        "parent": "北京市",
        "updateTime": "12:01"
    },
    "data": {
        "shidu": "66%",
        "pm25": 84.0,
        "pm10": 122.0,
        "quality": "轻度",
        "wendu": "13",
        "ganmao": "儿童、老年人及心脏、呼吸系统疾病患者人群应减少长时间或高强度户外锻炼",
        "forecast": [
            {
                "date": "12",
                "high": "高温 20℃",
                "low": "低温 8℃",
                "ymd": "2021-04-12",
                "week": "星期一",
                ......
```



# 4. Socket

> 一般充当服务器体现。  文件服务器     web服务器   邮件服务器

```java
两个进程(应用程序)间可以通过一个双向的网络通信连接实现数据交换，这种通信链路的端点被称为“套接字”（Socket）
```

## ==1. 基于TCP/IP==

```java
1. 服务器开启  等待着客户端连接
2. 客户端主动发起连接
3. 交互  ---> 数据读写 IO  
```

```java
创建服务端并监听指定的端口号。  ServerSocket
ServerSocket(int port)     
    
public static void main(String[] args) {

        try {
            //1.创建服务端程序  监听某个端口
            ServerSocket serverSocket = new ServerSocket(8888);
            System.out.println("---------------服务器开启---------------------");
            //2.等待着客户端连接
            while (true) {
                Socket client = serverSocket.accept();//阻塞  BIO
                System.out.println("成功连接的客户端:" + client.getInetAddress());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

    }    
```



```java
Socket:
  客户端。(双向的端点  既可以是客户端 又可以是服务端)
 Socket(InetAddress address, int port) 
 Socket(String host, int port)  
      
public static void main(String[] args) {
        try {
            //创建客户端  主动发起连接  连接特定的服务端程序(特定的计算机里面的应用程序)
            Socket socket = new Socket("192.168.14.113",8888);
            System.out.println("server:"+socket);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }      
```



> 模拟图片服务器

```java
//服务端--->接收文件内容 写入新的文件中
public class ImageServer {

    private static LongAdder adder = new LongAdder();

    static {
        adder.add(1000);
    }

    //保证文件名称是唯一的
    public static int getId() {
        adder.increment();
        return adder.intValue();
    }

    public static void main(String[] args) {

        try {
            ServerSocket imageServer = new ServerSocket(6666);

            System.out.println("-----------图片服务器开启----------------");
            while (true) {
                //等待客户端连接
                Socket clientSocket = imageServer.accept();
                final String targetFilePath = "day21/upload/" + getId() + ".jpg";
                //读取客户端提交的文件数据  read InputStream
                InputStream inputStream = clientSocket.getInputStream();
                //将内容写到目标文件中 write
                OutputStream outputStream = new FileOutputStream(targetFilePath);

                byte[] bytes = new byte[1024];
                int len;
                while ((len = inputStream.read(bytes)) != -1) {
                    outputStream.write(bytes, 0, len);
                }
                //服务端可以告诉客户端: 图片上传成功
              DataOutput dataOutput = new DataOutputStream(clientSocket.getOutputStream());
              dataOutput.writeUTF("图片上传成功");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```



```java
//客户端----->上传文件内容
public class UserClient {

    public static void main(String[] args) {

        String sourceFilePath = "C:\\Users\\DELL\\Pictures\\Saved Pictures\\c.jpg";

        try {
            Socket socket = new Socket("192.168.14.113", 6666);

            //将文件内容提交给服务器---> 读取到源文件数据   写给服务器 write
            byte[] bytes = new byte[1024];
            int len;
            InputStream inputStream = new FileInputStream(sourceFilePath);
            OutputStream outputStream = socket.getOutputStream();
            while ((len = inputStream.read(bytes)) != -1) {
                outputStream.write(bytes, 0, len);
            }     

            //内容上传完毕了
            socket.shutdownOutput();

            //读取服务器返回的数据
            DataInput dataInput = new DataInputStream(socket.getInputStream());
            System.out.println(dataInput.readUTF());

        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```





> 案例: 一对一

```java
public static void main(String[] args) throws IOException {

        final String serverName = "server:";
        //Exception in thread "main" java.net.BindException: Address already in use: JVM_Bind
        //端口号被占用了  服务端程序启动2次
        ServerSocket serverSocket = new ServerSocket(8888);
        System.out.println("-------------聊天开始--------------");
        Scanner input = new Scanner(System.in);
        while (true) {
            Socket client = serverSocket.accept();

            //读写数据
            while (true) {
                //服务端写给客户端
                DataOutput output = new DataOutputStream(client.getOutputStream());
                System.out.print(serverName);
                String str = input.nextLine();
                output.writeUTF(serverName + str);

                //服务端读取客户端数据
                DataInput dataInput = new DataInputStream(client.getInputStream());
                System.out.println(dataInput.readUTF());
            }

        }

    }
```



```java
 public static void main(String[] args) throws IOException {

        final String clientName = "client:";
        Scanner input = new Scanner(System.in);
        Socket socket = new Socket("localhost", 8888);

        while (true) {
            //客户端读写数据
            DataInput dataInput = new DataInputStream(socket.getInputStream());
            System.out.println(dataInput.readUTF());

            DataOutput dataOutput = new DataOutputStream(socket.getOutputStream());
            System.out.print(clientName);
            String s = input.nextLine();
            dataOutput.writeUTF(clientName +s );
        }
    }
```



> 一对一: 线程

```java
public class ReadWriteMsg {

    private static Scanner input;
    @NonNull
    private Socket socket;

    static {
        input = new Scanner(System.in);
    }

    public void readMsg() {
        try {
            DataInput dataInput = new DataInputStream(socket.getInputStream());
            System.out.println(dataInput.readUTF());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    public void writeMsg() {
        try {
            DataOutput dataOutput = new DataOutputStream(socket.getOutputStream());
            String threadName = Thread.currentThread().getName();
            System.out.print(threadName);
            String msg = input.nextLine();
            if("bye".equals(msg)){
                System.exit(-1);
            }
            dataOutput.writeUTF(threadName + msg);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public ReadWriteMsg(Socket socket) {
        this.socket = socket;
    }
}

```



## 2. 基于UDP

```java
public class DatagramSocket
extends Object
implements Closeable
    
 DatagramSocket(int port)  
    
```

```java
DatagramPacket(byte[] buf, int length) 
DatagramPacket(byte[] buf, int offset, int length)  接收数据包
    
DatagramPacket(byte[] buf, int length, InetAddress address, int port) 
DatagramPacket(byte[] buf, int offset, int length, InetAddress address, int port)     
DatagramPacket(byte[] buf, int offset, int length, SocketAddress address) 发送
DatagramPacket(byte[] buf, int length, SocketAddress address)     
```

```java
public class UDPClient {

    public static void main(String[] args) {
        //客户端 将 数据发送给服务端
        try {
            DatagramSocket client = new DatagramSocket();

            //DatagramPacket(byte[] buf, int length, InetAddress address, int port)
            byte[] bytes = "hello".getBytes();
            DatagramPacket packet = new DatagramPacket(bytes, bytes.length, InetAddress.getLocalHost(), 6666);
            client.send(packet);

            System.out.println("客户端数据发送成功");

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
 public static void main(String[] args) {

        try {
            DatagramSocket server = new DatagramSocket(6666);

            //接收数据
            //DatagramPacket(byte[] buf, int length)
            byte[] bytes = new byte[1024];
            DatagramPacket packet = new DatagramPacket(bytes, bytes.length);
            server.receive(packet);

            int len = packet.getLength();//获得有效的字节个数
            System.out.println(new String(bytes, 0, len));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

