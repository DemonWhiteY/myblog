---
title: JAVA联机五子棋（四）
date: 2023-12-25 17:05:10
tags: JAVA
---

# 网络模块

## 定义Message 类

在我们开发联机五子棋的过程中最大的一个问题是，各种消息太过复杂，对话消息，下棋消息，邀请消息，悔棋消息，将这些消息标准化定义，让服务器能够识别收到的到底是什么消息就很重要。

```java
import java.io.Serializable;
import java.util.LinkedList;

public class Message implements Serializable {
    private static final long serialVersionUID = 1L;


    public String username;
    public String content;
    public String time;

    public String code;

    public boolean color;

    public int x,y;
    public int ID;

    public LinkedList<user> userlist;
    public Message(String code,int ID,String username)
    {
        this.code=code;
        this.ID=ID;
        this.username=username;

    }

    public Message(String code,int ID)
    {
        this.code=code;
        this.ID=ID;

    }

    public Message(String code, LinkedList<user> userlist)
    {
        this.code=code;
        this.userlist=userlist;
    }

    public Message(String code,String username,String content,String time){
        super();
        this.code=code;
        this.username=username;
        this.content=content;
        this.time=time;

    }


    public Message(String code,int x,int y,boolean color){
        super();
        this.code=code;
        this.x=x;
        this.y=y;
        this.color=color;
    }

    public Message(String code,String username)
    {
        this.code=code;
        this.username=username;
    }

    public Message(String code)
    {
        this.code=code;

    }


    public Message()
    {
        super();
    }

}

class   Code{
    public static final String MESSAGE="message";
    public static final String CHESS="chess";

    public static final String REQUEST="request";
    public static final String BACK="back";
    public static final String LOST="lost";
    public static final String PEACE="peace";

    public static final String LOADING="loading";

    public static final String RENAME="rename";

    public static final String GETID="getid";

    public static final String REFUSE="refuse";
    public static final String STARTGAME="startgame";

    public static final String PEACEGAME="peacegame";

    public static final String WINGAME="wingane";

    public static final String REFUSEPEACE="refusepeace";

    public static final String UPDATE="update";

    public static final String OUTOFDATE="outofdate";



}
```

### 服务端

```java
package gobang;


import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.LinkedList;

public class Server {
    public static ServerModel model=new ServerModel();
    public static LinkedList<ObjectOutputStream>online=new LinkedList<ObjectOutputStream>();
    public Server(){}
    public static void main(String[] arg) throws IOException {


           ServerSocket ss = new ServerSocket(8000);
            int num = 1;
            int ID=0;
            System.out.println("正在连接中……");

            while (true) {
                Socket s = ss.accept();
                ServerThread newServerThtear=new ServerThread(s,ID);
                newServerThtear.start();
                user newuser=new user("新用户"+ID,ID);
                model.adduser(newuser);
                System.out.println("新用户加入,当前在线用户"+num+"人");

                num++;
                ID++;
            }

    }



}


class ServerThread extends Thread{

    private Socket s;

    private ObjectInputStream ois;
   private ObjectOutputStream oos;

   private int enemyID=-1;
   private int ID;
    public ServerThread(Socket s,int ID){
        super();
            this.s=s;
            this.ID=ID;

    }
    public void run(){


        try {

            ois = new ObjectInputStream(s.getInputStream());
            oos = new ObjectOutputStream(s.getOutputStream());
            Server.online.addLast(oos);
            Message Mmsg=new Message(Code.GETID,ID,Server.model.getdata().get(ID).username);
            oos.writeObject(Mmsg);


            while(true)
            {
                Message msg=(Message)ois.readObject();
                if(msg.code.equals("message")) {
                    if (enemyID != -1) {
                        System.out.println(msg.time + "\n" + msg.username + ":" + msg.content);
                        Server.online.get(enemyID).writeObject(msg);
                    }
                }
                else if(msg.code.equals("chess"))
                {
                    if (enemyID != -1) {

                        System.out.println(msg.time + "\n" + msg.username + ":" + msg.content);
                        Server.online.get(enemyID).writeObject(msg);
                    }
                }
                else if(msg.code.equals(Code.LOADING))
                {
                   Message answermsg=new Message(Code.LOADING,Server.model.getdata());
                    oos.writeObject(answermsg);
                    System.out.println(Server.model.getdata().getFirst().username);
                } else if (msg.code.equals(Code.UPDATE)) {
                    LinkedList<user> List=new LinkedList<>();
                    for(int i=0;i<Server.model.getdata().size();i++)
                    {
                        List.addLast(Server.model.getdata().get(i));
                    }

                    Message answermsg=new Message(Code.UPDATE,List);
                    System.out.println(answermsg.userlist.size());
                    System.out.println(answermsg.userlist.getFirst().username+" "+answermsg.userlist.getLast().username);
                   oos.writeObject(answermsg);



                } else if(msg.code.equals(Code.RENAME))
                {
                    Server.model.rename(this.ID,msg.username);
                    System.out.println("更改名字成功");

                }
                else if(msg.code.equals(Code.REQUEST))
                {
                   Message requestmsg=new Message(Code.REQUEST,ID, msg.username);
                    Server.online.get(msg.ID).writeObject(requestmsg);
                    System.out.println("请求发送给"+msg.ID);
                    this.enemyID=msg.ID;


                }else if(msg.code.equals(Code.STARTGAME))
                {
                    Server.online.get(msg.ID).writeObject(msg);
                    enemyID= msg.ID;

                }else if(msg.code.equals(Code.REFUSE))
                {
                    Server.online.get(msg.ID).writeObject(msg);
                } else
                {
                    if (enemyID != -1) {
                        Server.online.get(enemyID).writeObject(msg);
                    }
                }
//
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

### 客户端

```java
package gobang;

import java.io.*;
import java.net.Socket;
import java.net.UnknownHostException;

public class NET {

    public  Socket s;

    public String host="localhost";
    public int port=8080;


    NET() {

        try {
            s = new Socket("localhost", 8080);
            ObjectOutputStream os=new ObjectOutputStream(s.getOutputStream());
            ObjectInputStream is=new ObjectInputStream(s.getInputStream());
            SendThread sendThread=new SendThread(os);
            ReceiveThread receiveThread=new ReceiveThread(is);
            sendThread.start();
            receiveThread.start();

        }catch (UnknownHostException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

    public void SETNET()
    {
        try {
            s.close();
            s = new Socket(host,port);
            ObjectOutputStream os=new ObjectOutputStream(s.getOutputStream());
            ObjectInputStream is=new ObjectInputStream(s.getInputStream());
            SendThread sendThread=new SendThread(os);
            ReceiveThread receiveThread=new ReceiveThread(is);
            sendThread.start();
            receiveThread.start();

        }catch (UnknownHostException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

    }

}

```

### 客户端发送线程

```java
package gobang;



import java.io.IOException;
import java.io.ObjectOutputStream;

public class SendThread extends Thread {
    private ObjectOutputStream os;

    public SendThread(ObjectOutputStream os) {
        super();
        this.os = os;

    }

    public void run() {

        try {


            while (true) {
                System.out.print("");
                if (!Vars.model.messageModel.isEmpty()) {


                    Message msg = Vars.model.messageModel.getFirst();
                    System.out.println(msg.time+"\n"+msg.username+":"+msg.content);
                    os.writeObject(msg);
                    Vars.model.messageModel.removeFirst();

                }
            }
        } catch (IOException e) {
            throw new RuntimeException(e);

        }
    }

}
```

### 客户端接收线程

```java
package gobang;


import javax.swing.*;
import java.awt.*;
import java.io.IOException;
import java.io.ObjectInputStream;

public class ReceiveThread extends Thread{

    private ObjectInputStream ois;
    private volatile boolean flag = true;

    public ReceiveThread(ObjectInputStream ois) {
        super();
        this.ois = ois;
    }
    public void run() {


            while(true) {

                System.out.print("");
                Message msg = null;
                try {
                    msg = (Message) ois.readObject();
                    Vars.messageManager.getMessage(msg);
                } catch (IOException e) {
                    throw new RuntimeException(e);
                } catch (ClassNotFoundException e) {
                    throw new RuntimeException(e);
                }

                

            }

    }
}
```

# 消息管理器

```java
package gobang;

import javax.swing.*;
import java.awt.*;

public class messageManager {
    public void getMessage(Message msg)
    {
        if(msg.code.equals("message")) {
            System.out.println(msg.time + "\n" + msg.username + ":" + msg.content);
            Vars.talkPanel.showarea.append(msg.time + "\n" + msg.username + ":" + msg.content + "\n");
        }
        else if(msg.code.equals("chess")){
            System.out.println();
            Vars.controler.addNetChess(msg.x, msg.y);
        }
        else if(msg.code.equals(Code.BACK)){
            Vars.controler.NETback();
        } else if (msg.code.equals(Code.LOST)) {
            Vars.dialogStyle.Gameover(Start.getStart(),"对方认输");

        } else if (msg.code.equals(Code.PEACE)) {
            Vars.dialogStyle.AnswerPeace(Start.getStart());
        }
        else if(msg.code.equals(Code.LOADING)){
            Vars.model.userList=msg.userlist;
            System.out.println();
        }
        else if (msg.code.equals(Code.REQUEST)) {
            Vars.dialogStyle.AnswerRequest(MainWindows.getMainWindows(), msg.username, 30);
            Vars.model.enemyID= msg.ID;
            Vars.model.enemyname=msg.username;
        } else if (msg.code.equals(Code.REFUSE)) {
            Vars.dialogStyle.waitDialog(MainWindows.getMainWindows(),"对方拒绝了你的邀请",30);
            Vars.model.enemyID=-1;
        }else if(msg.code.equals(Code.STARTGAME)){
            Vars.controler.turnyou=false;
            Vars.roundPanel.update();
            Start start=Start.getStart();
            Vars.timePanel.restart();
        }else if(msg.code.equals(Code.PEACEGAME)){
            Vars.dialogStyle.Gameover(Start.getStart(),"平局");
        }else if(msg.code.equals(Code.REFUSEPEACE)){
            Vars.dialogStyle.waitDialog(Start.getStart(),"对方拒绝了你的求和",10);
        }else if(msg.code.equals(Code.UPDATE)){

            System.out.println(msg.userlist.getFirst().username+" "+msg.userlist.getLast().username);
            System.out.println(msg.userlist.size());
            MainWindows.getMainWindows().Player.removeAll();

            Vars.model.userList=msg.userlist;



            if(Vars.model.userList.size()==1)
            {

                JLabel label=new JLabel("当前无其他用户");
                Font font=new Font("黑体",Font.PLAIN,50);
                label.setFont(font);
                label.setForeground(Color.decode("#0fa99a"));
                MainWindows.getMainWindows().Player.add(label);
            }
            else {
                for (int i = 0; i < Vars.model.userList.size(); i++) {
                    if(i!=Vars.model.ID) {
                        PlayerLabel player = new PlayerLabel(msg.userlist.get(i).username, msg.userlist.get(i).userID);
                        MainWindows.getMainWindows().Player.add(player);
                    }



                }
            }
            JLabel label2=new JLabel("当前在线人数"+String.valueOf(msg.userlist.size()));
            Font font=new Font("黑体",Font.PLAIN,50);
            label2.setFont(font);
            label2.setForeground(Color.decode("#0fa99a"));
            MainWindows.getMainWindows().Player.add(label2);
            MainWindows.getMainWindows().Player.updateUI();
        } else if (msg.code.equals(Code.GETID)) {
            System.out.println("用户信息加载成功");
            System.out.println(msg.ID+msg.username);
            Vars.model.ID=msg.ID;
            Vars.model.username= msg.username;

        }
    }
    
    
}

```
