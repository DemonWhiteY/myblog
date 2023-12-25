---
title: JAVA联机五子棋（二）多用户网络通讯实例——多用户群聊
date: 2023-12-24 00:59:48
tags:
---
# 多用户网络通讯实例——多用户群聊

在实现五子棋之前我们先来熟悉一下适用于多用户的网络通讯。
![屏幕截图 2023-12-24 113957.png](https://s2.loli.net/2023/12/25/wYDpmCsSJuFt5Rf.png)
首先我们来想一下，我们需要用几个线程吧，
首先是服务端，服务端需要几个线程呢，肯定是有几个用户就有几个线程，毕竟服务端要时刻准备接受客户端传来的消息（当然这种阻塞式的线程管理方法，在用户数量大的情况下自然是行不通的，毕竟没有服务器能跑几千几万个线程，这是就要用到JAVA专门用于这类操作的类库了）

<br>
然后考虑客户端，客户端需要几个线程呢，答案是两个，客户端的网络模块主要有两个部分的功能，一个是接手消息，另一个就是发送消息，因此我们要写一个SendThread,一个ReciveThread.

<br>
这时就有同学想要问了，为什么服务端不需要像客户端那样写发消息手消息两个线程呢，是因为服务端的收发消息都是同时进行的，收到一个立马发送掉，随收随发，因此它并不需要一个消息队列，也就不需要一个专门发消息的线程。

<br>
而客户端呢，客户端的收消息和发消息并不是同时进行的，客户端时刻可能收到消息，也时刻可能发送消息，这些都是不能预测的，因此必须用两个线程来接受和发送。

# 定义传输信息类

```java
package mail;

import java.io.Serializable;

public class Message implements Serializable {
    private static final long serialVersionUID = 1L;
    public String username;
    public String content;
    public String time;

    public Message(String username,String content,String time){
        super();
        this.username=username;
        this.content=content;
        this.time=time;

    }

    public Message()
    {
        super();
    }

}

```

# 服务端实例

```java
package mail;

import mail.Message;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.LinkedList;

public class Server {
    public Server(){}
    public static void main(String[] arg) throws IOException {


        ServerSocket ss = new ServerSocket(8080);

        int num = 1;
        int userID=0;
        System.out.println("正在连接中……");

        while (true) {
            Socket s = ss.accept();
            ServerThread newServerThtear=new ServerThread(s);
            newServerThtear.start();

            System.out.println("新用户加入,当前在线用户"+num+"人");
            num++;
        }

    }



}
```

# 服务端线程

```java
class ServerThread extends Thread{

    private Socket s;
    private String username;
    private ObjectInputStream ois;
    private ObjectOutputStream oos;

    public static LinkedList<ObjectOutputStream>online=new LinkedList<ObjectOutputStream>();



    public ServerThread(Socket s){
        super();
        this.s=s;
    }
    public void run(){


        try {

            ois = new ObjectInputStream(s.getInputStream());
            oos = new ObjectOutputStream(s.getOutputStream());
            online.addLast(oos);


            while(true)
            {
                Message msg=(Message)ois.readObject();

                System.out.println(msg.time+"\n"+msg.username+":"+msg.content);
                for(ObjectOutputStream on:online)
                {
                    on.writeObject(msg);
                }
            }




        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }


    }
}
```

# 客户端实例（使用了Swing）

```java
package mail;

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.io.*;
import java.net.Socket;
import java.net.UnknownHostException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.LinkedList;


import static java.lang.System.out;


public class Client {

    private JFrame mainFrame;
    private JTextField username;
    private JScrollPane statusLabel;
    private JPanel controlPanel;
    public JTextArea show;
    private static Client client=new Client();

    public static LinkedList<Message> sendMessage=new LinkedList<Message>();

   private     Client(){
              prepareGUI();


        }
        public static  Client getClient()
        {
            return client;
        }



    private void prepareGUI(){
        mainFrame = new JFrame("Java Swing JTextArea示例");
        mainFrame.setSize(400,400);
        mainFrame.setLayout(new GridLayout(3, 1));

        mainFrame.addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent windowEvent){
                System.exit(0);
            }
        });
        username=new JTextField(20);
        show=new JTextArea(20,20);
        statusLabel=new JScrollPane(show);

        controlPanel = new JPanel();
        controlPanel.setLayout(new FlowLayout());

        mainFrame.add(username,BorderLayout.NORTH);
        mainFrame.add(statusLabel,BorderLayout.CENTER);
        mainFrame.add(controlPanel,BorderLayout.SOUTH);

        mainFrame.setVisible(true);
    }





    public static void main(String[] args) {



        Client client=Client.getClient();

        SimpleDateFormat sdf = new SimpleDateFormat();// 格式化时间
        sdf.applyPattern("HH:mm:ss a");// a为am/pm的标记
        Date date = new Date();// 获取当前时间


        try {
            Socket s = new Socket("localhost",8080);
            ObjectOutputStream os=new ObjectOutputStream(s.getOutputStream());
            ObjectInputStream is=new ObjectInputStream(s.getInputStream());






            client.username.setText("新用户");
            JLabel  commentlabel= new JLabel("发送: ", JLabel.RIGHT);

            JTextArea commentTextArea =
                    new JTextArea("你好",3,20);

            JScrollPane scrollPane = new JScrollPane(commentTextArea);
            JButton showButton = new JButton("发送");

            showButton.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    Message msg=new Message(client.username.getText(),commentTextArea.getText(),sdf.format(date));
                    out.println(client.username.getText());
                    client.sendMessage.addLast(msg);
                    System.out.println(client.sendMessage.size());

                }
            });
            client.controlPanel.add(commentlabel);
            client.controlPanel.add(scrollPane);
            client.controlPanel.add(showButton);
            client.mainFrame.setVisible(true);

            SendThread sendThread=new SendThread(os,client.username.getText());
            ReciveThread reciveThread=new ReciveThread(is);

            sendThread.start();
            reciveThread.start();


        } catch (UnknownHostException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

```

# 客户端Receive线程

```java

package mail;

import java.io.IOException;
import java.io.ObjectInputStream;

public class ReciveThread extends Thread{

    private ObjectInputStream ois;
    private volatile boolean flag = true;

    public ReciveThread(ObjectInputStream ois) {
                 super();
                 this.ois = ois;
            }
  public void run() {
        while(true)
        {

            System.out.println("111");
            Message rmsg= null;
            try {
                rmsg = (Message)ois.readObject();
            } catch (IOException e) {
                throw new RuntimeException(e);
            } catch (ClassNotFoundException e) {
                throw new RuntimeException(e);
            }

                System.out.println(rmsg.time + "\n" + rmsg.username + ":" + rmsg.content);
                Client.getClient().show.append(rmsg.time + "\n" + rmsg.username + ":" + rmsg.content + "\n");

        }

  }
}


```

# 客户端Send线程

```java
package mail;

import java.io.IOException;
import java.io.ObjectOutputStream;

public class SendThread extends Thread {
private ObjectOutputStream os;
private String username;
public SendThread(ObjectOutputStream os,String username)
{
    super();
    this.os=os;
    this.username=username;
}

public void run()
{

    try {


        while(true)
        {

            if(!Client.getClient().sendMessage.isEmpty())
            {
                System.out.println("111");

                Message msg=Client.getClient().sendMessage.getFirst();

                    os.writeObject(msg);
                    Client.getClient().sendMessage.removeFirst();

                }
            }
        }catch (IOException e) {
        throw new RuntimeException(e);

}
}


}

```
