## Java网络编程

[TOC]



### Network



**什么是计算机网络**

- 两台或更多的计算机组成的网络
- 同一网络内的任意两台计算机都可以直接通信
- 所有计算机必须遵循同一种网络协议

**Internet**

- 互联网是网络中的网络
- 互联网采用TCP/IP协议
  - TCP/IP协议泛指互联网协议
  - 其中最重要的两个协议就是TCP协议和IP协议



#### OSI模型 & TCP/IP模型

- Open System Interconnect

|   应用层   |
| :--------: |
| **表示层** |
| **会话层** |
| **传输层** |
| **网络层** |
| **链路层** |
| **物理层** |

- TCP/IP

|     应用层     |
| :------------: |
|   **传输层**   |
|    **IP层**    |
| **网络接口层** |



**IP协议**

- 分组交换
- 不保证可靠传输

**TCP协议**

- 传输控制协议
- 面向连接
- 可靠传输
- 双向通信

**UDP协议**

- 数据报文协议
- 无连接
- 不保证可靠传输
- 传输效率高

------



### TCP编程



**Socket编程**

- IP地址

- 端口号（0~65535）

  

**服务器端：监听端口**

```java
ServerSocket ss = new ServerSocket(port); // 指定监听的端口
Socket sock = ss.accept();
InputStream in = sock.getInputStream();
OutputStream out = sock.getOutputStream();
//读写字节流
```



**客户端：主动连接**

```java
Socket sock = new Socket(InetAddress, port); //连接到远程服务器的指定端口
InputStream in = sock.getInputStream();
OutputStream out = sock.getOutputStream();
//读写字节流
```



#### Code Demo

TCPServer.java

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.charset.StandardCharsets;
import java.time.LocalDateTime;

public class TCPServer {

    /**
     * @ serverSocket -- 创建服务器端监听端口
     * @ socket -- 用于向客户端返回一个已成功建立的连接（等待客户端连接请求）
     * @ reader -- 用new BufferedReader(new InputStreamReader(socket.getInputStream())) 包装socket输入流（读取），用于读取客户端发来的数据
     * @ writer -- 用new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()))包装socket输出流（写入），用于写入响应客户端的数据
     */
    public static void main(String[] args) throws Exception{
        ServerSocket serverSocket = new ServerSocket(9090);
        System.out.println("TCP server ready");
        Socket socket = serverSocket.accept();
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream(), StandardCharsets.UTF_8))){
            try (BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream(), StandardCharsets.UTF_8))){
                String cmd = reader.readLine();
                if ("time".equals(cmd)) {
                    writer.write(LocalDateTime.now().toString() + "\n");
                    writer.flush();
                } else {
                    writer.write("Pardon?\n");
                    writer.flush();
                }
            }
        }

        socket.close();
        serverSocket.close();
    }
}
```

TCPClient.java

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.net.InetAddress;
import java.net.Socket;
import java.nio.charset.StandardCharsets;

public class TCPClient {
	/**
	 * @ address -- 服务器端的IP地址
	 * @ socket -- 连接指定的IP地址端口（向服务器端发起连接请求）
	 * @ reader -- BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream())) 包装socket输入流（读取），用于读取服务器端响应的数据
	 * @ writer -- 用new BufferedWriter(new OutputStreamWriter(socket.getOutputStream())) 包装socket输出流（写入），用于写入向服务器发送的数据
	 */
    public static void main(String[] args) throws Exception{

        InetAddress address = InetAddress.getLoopbackAddress();
        try (Socket socket = new Socket(address, 9090)){
            try (BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream(), StandardCharsets.UTF_8))){
                try (BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream(), StandardCharsets.UTF_8))){
                    writer.write("time\n");
                    writer.flush();
                    String resp = reader.readLine();
                    System.out.println("Response: " + resp);
                }
            }
        }
    }
}
```





**TCP编程模型：**

- 客户端使用`Socket(InetAddress, port)`打开Socket
- 服务器断用`ServerSocket`监听端口
- 服务器端用`accept`接收链接并返回Socket
- 双方通过Socket打开`InputStream / OutputStream`读写数据
- `flush()`用于强制输出缓冲区

------



### TCP多线程编程



#### TCP多线程编程模型

- 服务器端使用无限循环
- 每次accept返回后，创建新的线程来处理客户端请求
- 每隔客户端求对应一个服务线程
- 使用线程池可以提高运行效率



#### Code Demo (Unsolved)

Server

```java
import com.sun.xml.internal.ws.policy.privateutil.PolicyUtils;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.charset.StandardCharsets;
import java.time.LocalDateTime;

public class ServerNeverStop {

    public static void main(String[] args) throws Exception{
        ServerSocket serverSocket = new ServerSocket(9090);
        System.out.println("Server is ready");
		/**
		 * 服务器无限等待连接请求
		 */
        for (;;) {
            Socket socket = serverSocket.accept();
            System.out.println("Accept from: " + socket.getRemoteSocketAddress());
            TimeHolder timeHolder = new TimeHolder(socket);
            timeHolder.start();
        }
    }
}

class TimeHolder extends Thread{

    Socket socket;

    public TimeHolder(Socket socket) {
        this.socket = socket;
    }
    /**
     * 通过读取客户端发来的数据做出对应的响应
     */
    @Override
    public void run() {
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(this.socket.getInputStream(), StandardCharsets.UTF_8))){
            try (BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(this.socket.getOutputStream(), StandardCharsets.UTF_8))){
                for (;;) {
                    String cmd = reader.readLine();
                    if("q".equals(cmd)) {
                        writer.write("Byebye!");
                        writer.flush();
                        break;
                    } else if ("time".equals(cmd)) {
                        writer.write(LocalDateTime.now().toString());
                        writer.flush();
                    } else {
                        writer.write("Pardon\n");
                        writer.flush();
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                this.socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

Client

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.net.InetAddress;
import java.net.Socket;
import java.nio.charset.StandardCharsets;

public class TCPClient2VMS {

    public static void main(String[] args) throws Exception{
        // getByName(hostname) & getByAddress(new byte[] {IP_address})
//        InetAddress address = InetAddress.getByName("Lab");
        InetAddress address = InetAddress.getByAddress(new byte[] {(byte)192, (byte)168, (byte)247, (byte) 134});
        
        /**
         * 连接服务器，发送指定数据获取回应
         */
        try (Socket socket = new Socket(address, 9090)){
            try (BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream(), StandardCharsets.UTF_8))){
                try (BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream(), StandardCharsets.UTF_8))){
                    writer.write("time\n");
                    writer.flush();
                    String resp = reader.readLine();
                    System.out.println("Response: " + resp);

                    Thread.sleep(2000);

//                    writer.write("sdfe\n");
//                    writer.flush();
//                    resp = reader.readLine();
//                    System.out.println("Response: " + resp);
//
//                    Thread.sleep(2000);

                            writer.write("q\n");
                    writer.flush();
                    resp = reader.readLine();
                    System.out.println("Response: " + resp);
                }
            }
        }
    }
}
```





**ServerSocket**

```java
// 允许所有IP的指定端口创建连接
new ServerSocket(int port)
    
// 指定IP和端口创建连接
new ServerSocket(int port, InetAddress address)
```

------



### UDP编程



- 不需要建立连接
- 可以直接发送和接收数据



#### UDP Client

```java
DatagramSocket sock = new DatagramSocket();
sock.connect(addr, port);
    
// Send
byte[] data = ...
DatagramPacket sendPacket = new DatagramPacket(data, data.length);
sock.send(sendPacket);

// Receive
byte[] buffer = new byte[1024];
DatagramPacket receivePacket = new DatagramPacket(buffer, buffer.length);
sock.receive(receivePacket);
```



#### UDP Server

```java
DatagramSocket ds = new DatagramSocket(port);
for(;;) {
    // Receive
    byte[] buffer = new byte[1024];
    DatagramPacket packet = new DatagramPacket(buffer, buffer.lenght);
    ds.receive(packet);
    
    // Send
    byte[] data = ...;
    packet.setData(date);
    ds.send(packet);
}
```



**UDP编程模型**

- 客户端使用DatagramSocket.connect()指定远程地址和端口
- 服务器端使用DatagramSocket(port)监听端口
- 双方通过receive / send读写数据
- 没有IO流接口



#### Code Demo

UDP Client

```java
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.nio.charset.StandardCharsets;

public class UDPClient {

    public static void main(String[] args) throws Exception{
		/**
		 * @ address -- 获取远程地址
		 * 连接地址address和端口为9090的主机
		 *
		 * @ data -- 待发送的数据
		 * @ packet -- 包装待发送的数据
		 * .send(packet) -- 发送数据
		 * @ buffer -- 创建缓冲区存储接收到的数据
		 * @ resp -- 包装接收数据包
		 * @ receiveData -- 获取byte类型的接收数据
		 */
        InetAddress address = InetAddress.getLoopbackAddress();
        try (DatagramSocket socket = new DatagramSocket()) {
            socket.connect(address, 9090);
            byte[] data = "time".getBytes(StandardCharsets.UTF_8);
            DatagramPacket packet = new DatagramPacket(data, data.length);

            socket.send(packet);
            System.out.println("Data sent");
            Thread.sleep(1000);

            byte[] buffer = new byte[1024];
            DatagramPacket resp = new DatagramPacket(buffer, buffer.length);
            socket.receive(resp);
            byte[] receiveData = resp.getData();
            String respTxt = new String(receiveData, 0, resp.getLength(), StandardCharsets.UTF_8);
            System.out.println("Response: " + respTxt);
        }
    }
}
```

UDP Sever

```java
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.nio.charset.StandardCharsets;
import java.time.LocalDateTime;
import java.util.Arrays;

public class UDPServer {
	/**
	 * @ buffer -- 创建缓冲区存储接收到的数据
	 * @ packet -- 包装接收数据包
	 * receive() -- 接收数据
	 * data -- 获取byte类型的接收数据
	 * setData -- 重置数据包里的数据
	 * send() -- 发送数据
	 */
    public static void main(String[] args) throws Exception{
        DatagramSocket datagramSocket = new DatagramSocket(9090);
        System.out.println("Server is ready");

        for (;;) {
            byte[] buffer = new byte[1024];
            DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
            datagramSocket.receive(packet);
            byte[] data = packet.getData();
            String s = new String(data, StandardCharsets.UTF_8);
            System.out.println("Packet received from: " + Arrays.toString(packet.getData()));

            String resp = LocalDateTime.now().toString();
            packet.setData(resp.getBytes(StandardCharsets.UTF_8));
            datagramSocket.send(packet);
        }
    }
}
```

------



### Send ing  Email

**SMTP协议**

- Simple Mail Transport Protocol
- 标准端口25
- 加密端口465 / 587



**JavaMail API**

```java
// Create session
Session session = Session.getInstance(props, new Authenticator() {...});

// Create Message Object
MimeMessage message = new MimeMessage(session);
message.setFrom(new InternetAddress("from@email.com"));
message.setRecipient(Message.RecipientType.TO, new InternetAddress("to@email.com"));
message.setSubject("RE: how to use JavaMail", "UTF-8");
message.setText("emailcontent...", "UTF-8");

//Send mail
Transport.send(message);
```





**Java发送Email**

- 用Maven引入javamail依赖
- 确定SMTP服务器信息：域名 / 端口 / 使用明文 / SSL / TLS
- 调用相关API发送Email（无需关心底层TCP Socket连接）
- 设置debug模式可以查看通信详细内容（便于排查错误）



#### Code Demo (Unresolved)

```java
package org.smtp.javamail;

import javax.mail.*;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;
import java.util.Properties;

/**
 * Hello world!
 *
 */
public class SendMail
{
    final String smtpHost;
    final String username;
    final String passwd;
    final boolean debug;

    public SendMail(String smtpHost, String username, String passwd) {
        this.smtpHost = smtpHost;
        this.username = username;
        this.passwd = passwd;
        this.debug = true;
    }
    
    /**
     * @ senMail -- Create a new object with an account to send email, include SMTP host, username and password 
     */
    public static void main ( String[] args ) throws Exception
    {
        SendMail sendMail = new SendMail("smtp.office365.com", "mjhisoka24@outlook.com", "Lmy***^");
        Session session = sendMail.createSSLSession();
        Message message = createTextMessage(session, sendMail.username, "44**95@qq.com", "JavaMail Testing", "You got it, hahahah");
        Transport.send(message);
    }
	
    /**
     * Create a session with SSL encryption 
     * Properties need to put SMTP host, port, authenticate
     */
    Session createSSLSession() {
        Properties properties = new Properties();
        properties.put("mail.smtp.host", this.smtpHost);
        properties.put("mail.smtp.port", "587");
        properties.put("mail.smtp.auth", "true");

        properties.put("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSession");
        properties.put("mail.smtp.socketFactory.port", "587");
		// Account authenticate 
        Session session = Session.getInstance(properties, new Authenticator() {
            @Override
            protected PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication(SendMail.this.username, SendMail.this.passwd);
            }
        });
        session.setDebug(this.debug);
        return session;
    }
	/**
	 * createTextMessage -- Create detail of a mail, session, who send, who receive, mail subject and mail body
	 */
    static Message createTextMessage(Session session, String from, String to, String subject, String body) throws Exception{
        MimeMessage message = new MimeMessage(session);
        message.setFrom(new InternetAddress(from));
        message.setRecipients(Message.RecipientType.TO, String.valueOf(new InternetAddress(to)));
        message.setSubject(subject,"UTF-8");
        message.setText(body, "UTF-8");
        return message;
    }
}
```

------



### Receive Email

**POP3协议**

- Post Office Protocol Version 3
- 标准端口110
- 加密端口995

**IMAP协议**

- Internet Mail Access Protocol
- 标准端口143
- 加密端口993

```java
// Create session, props contain the user info
Session session = Session.getInstance(props, null);

// Create Store
Store store = new POP3SSLStore(session, url);

// Get Folder
Folder folder = store.getFolder("INBOX");

// Get Message
Message[] messages = folder.getMessage();
for (Message message : messages) {...}
```



**Java接收Email**

- 用Maven引入javamail依赖
- 确定POP3服务器信息：域名 / 端口  / 使用明文 / SSL
- 调用相关API接收Email（无需关心底层TCP Socket连接）
- 设置debug模式可以查看通信详细内容（便于排查错误）



#### Code Demo (Unresolved)

```java
package org.pop3demo;

import com.sun.mail.pop3.POP3SSLStore;

import javax.mail.*;
import javax.mail.internet.MimeMessage;
import javax.mail.internet.MimeUtility;
import java.io.IOException;
import java.util.Properties;

public class Pop3 {
    
    final String popHost;
    final String username;
    final String password;
    final boolean debug;
    
    public Pop3(String popHost, String username, String password) {
        
        this.debug = true;
        this.popHost = popHost;
        this.username = username;
        this.password = password;
    }

    public static void main(String[] args) throws Exception{
        Pop3 pop3 = new Pop3("pop.163.com", "email@163.com", "xxxxxx");
        Folder folder = null;
        Store store = null;
        
        try {
            store = pop3.createSSLSession();
            folder = store.getFolder("INBOX");
            folder.open(Folder.READ_WRITE);
            System.out.println("Total Messages: " + folder.getMessageCount());
            System.out.println("New Messages: " + folder.getNewMessageCount());
            System.out.println("Unread Messages: " + folder.getUnreadMessageCount());
            System.out.println("Delete Messages: " + folder.getDeletedMessageCount());
            
            Message[] messages = folder.getMessages();
            for (Message message : messages) {
                printMessage((MimeMessage) message);
            }
        } finally {
            
        }
    }
    
    
    public Store createSSLSession() throws MessagingException {
        Properties properties = new Properties();
        properties.setProperty("mail.store.protocol", "pop3");
        properties.setProperty("mail.pop3.port", "995");
        properties.setProperty("mail.pop3.host", this.popHost);
        
        properties.put("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSocketFactory");
        properties.put("mail.smtp.socketFactory.port", "995");
        URLName urlName = new URLName("pop3", this.popHost, 995, "", this.username, this.password);
        Session session = Session.getInstance(properties, null);
        session.setDebug(this.debug);
        Store store = new POP3SSLStore(session, urlName);
        store.connect();
        return store;
    }
    
    static void printMessage(MimeMessage message) throws Exception{
        System.out.println("-----------------------");
        System.out.println("Subject: " + MimeUtility.decodeText(message.getSubject()));
        System.out.println("From: " + getFrom(message));
        System.out.println("To: " + getTo(message));
        System.out.println("sent: " + message.getSentDate().toString());
        System.out.println("seen: " + message.getFlags().contains(Flags.Flag.SEEN));
        System.out.println("Priority: " + getPriority(message));
        System.out.println("Size: " + message.getSize() / 1024 + "kb");
        System.out.println("Body: " + getBody(message));
        System.out.println("-----------------------");
        System.out.println();
    }
    
    static String getBody(Part part) throws MessagingException, IOException {
        if(part.isMimeType("text/*")) {
            return part.getContent().toString();
        }
        if(part.isMimeType("multipart/*")) {
            Multipart multipart = (Multipart) part.getContent();
            for (int i = 0; i < multipart.getCount(); i++) {
                BodyPart bodyPart = multipart.getBodyPart(i);
                String body = getBody(bodyPart);
                if (!body.isEmpty()) {
                    return body;
                }
            }
        }
        return "";
    }
}
```

------



### HTTP Programming

**HTTP**

- HyperText Transfer Protocol
- 基于TCP协议之上的请求 / 响应协议
- 目前使用最广泛的高级协议



**HTTP Server**

- 处理HTTP请求，发送HTTP响应
- Java EE Servlet API

**HTTP Client**

- 发送HTTP请求， 接收HTTP响应
- java.net.HttpURLConnection



**GET请求**

```java
URL url = new URL("http://xxxxx.xxx.xx");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
int code = conn.getResponseCode();
try (InputStream input = conn.getInputStream()) {
    // 读取响应数据
}
conn.disconnect();
```

**POST请求**

```java
URL url = new URL("http://xxxxx.xxx.xx");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("POST");
conn.setDoOutput(true);
byte[] postData = "loginName=test&password=123456".getBytes("UTF-8");
conn.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
conn.setRequestProperty("Content-Length", "String.valueOf(postDate.length)");
try (OutputStream output = conn.getOutputStream()) {
    output.write(postData);
}
int code = conn.getResponseCode();
try (InputStream input = conn.getInputStream()) {
    //读取响应数据
}
conn.disconnect();
```



- HTTP协议是一个基于TCO的请求 / 响应协议
- 广泛用于浏览器、手机App于服务器的数据交互
- Java提供了HttpURLConnection实现HTTP客户端

------



### RMI

**RMI远程调用**

- Remote Method Invocation
- 目的：把一个接口的方法暴露给远程



#### Code Demo

Interface

```java
import java.rmi.Remote;
import java.rmi.RemoteException;
import java.time.LocalDateTime;

public interface Clock extends Remote {

    LocalDateTime currentTime() throws RemoteException;
}
```

Implement

```java
import java.rmi.RemoteException;
import java.time.LocalDateTime;

public class ClockImple implements Clock{

    @Override
    public LocalDateTime currentTime() throws RemoteException {
        return LocalDateTime.now();
    }
}
```

Server

```java
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.server.UnicastRemoteObject;
import java.time.LocalDateTime;

public class RMIServer {
	/**
	 * Bind the method implemented with "Clock" 
	 */
    public static void main(String[] args) throws Exception{
        Clock impl = new ClockImple();
        Clock stub = (Clock) UnicastRemoteObject.exportObject(impl, 1099);
        LocateRegistry.createRegistry(1099);
        Registry registry = LocateRegistry.getRegistry();
        registry.bind("Clock", stub);
        System.out.println("Clock server is ready");

    }
}
```

Client

```java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.time.LocalDateTime;

public class RMIClient {
	/**
	 * Get the "Clock"
	 * Recall the method implemented
	 */
    public static void main(String[] args) throws Exception{
        Registry registry = LocateRegistry.getRegistry(null);
        Clock clock = (Clock) registry.lookup("Clock");
        LocalDateTime dt = clock.currentTime();
        System.out.println("RMI reslut: " + dt);
    }
}
```



- RMI远程调用是针对Java语言的一种远程调用（进程对进程）
- 远程接口必须继承自Remote
- 远程方法必须抛出RemoteException
- 客户端调用RMI方法和调用本地方法类似
- RMI方法调用被自动通过网络传输到服务器端
- 服务器端通过自动生成的stub类接收远程调用请求