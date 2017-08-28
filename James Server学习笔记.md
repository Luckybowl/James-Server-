# 1. 简介
  Apache James（Java Apache Mail Enterprise Server）是Apache组织的子项目之一，完全采用纯Java技术开发，实现了SMTP、POP3与NNTP等多种邮件相关协议。

James也是一个邮件应用平台，可以通过Match与Mailet扩充其功能，如Mail2SMS、Mail2Fax等。James提供了比较完善的配置方案，尤其是关于邮件内容存储和用户信息存储部分，可以选择在文件、数据库或其他介质中保存。

James性能稳定、可配置性强，还是开源项目，所有源代码不存在版权问题，因此，James在项目中的应用日益广泛，现在已经有3.0版本，不过下载后发现，里面内容太过庞大，因此还是选择精简且稳定的2.3.2版本做介绍。

# 2. 安装与配置
## 2.1 Windows环境下
第一步：安装JDK，此处略。
第二步：下载James，并解压。
[点击下载](http://archive.apache.org/dist/james/server/apache-james-2.3.2.zip)（推荐此链接，其余版本可能出错。）
第三步：直接运行或需要配置JAVA_HOME。
 这时，可以尝试直接双击/james/bin/run.bat，若启动无误，将提示如下：
```
Using PHOENIX_HOME:   C:/james
Using PHOENIX_TMPDIR: C:/james/temp
Using JAVA_HOME:

Phoenix 4.0.1

James 2.1
Remote Manager Service started plain:4555
POP3 Service started plain:110
SMTP Service started plain:25
NNTP Service Disabled
Fetch POP Disabled

```
要关闭 James 服务，请使用 Ctrl + C
启动前请确保您的JDK环境变量如JAVA_HOME等已经设置好；James 启动时，其SMTP 服务默认在 25 端口启动，POP3 服务默认在 110 端口启动， NNTP 服务默认在 119 端口启动, 请确保这些端口未被占用。
如果这几个端口已经占用的话，打开run.bat是会一闪而过的，请在james的文件路径apps/james/SAR-INF下打开config.xml文件，这个文件是服务器的配置文件，用notepad++或editplus等编辑器打开。CTRL+F找到pop3server这个标签：把110改成其他端口
```
<pop3server enabled="true">
   <!-- POP3协议端口 -->  
   <port>110</port>
   <handler>
    helloName autodetect="true">myMailServer</helloName>
     <connectiontimeout>120000</connectiontimeout>
   </handler>  
</pop3server>

```
同理，把下面smtpserver和nntpserver的端口也改掉。
我们修改完这个几个端口后，应该就可以顺利启动James服务了。

PS:若使用其他版本的james，可能会报
```
”caused by: java.io.IOException: 文件名、目录名或卷标语法不正确。”
```
## 2.2 linux环境下
第一步：安装jdk，配置环境变量，此处略。
第二步：下载，并解压。
```
1. wget http://mirror.bjtu.edu.cn/apache//james/server/apache-james-2.3.2.tar.gz
2. tar -zxvf apache-james-2.3.2.tar.gz
3. ln -s james-2.3.2 mailserver
```
第三步：运行。必须先运行一下，才能配置。
```
1. cd james-2.3.2/
2. chmod +x bin/*.sh
3. vi bin/run.sh #在第一行加入export JAVA_HOME=/opt/java(此处为你的JAVA_HOME)
4. bin/run.sh
```
运行成功后如下：
```
[root@dev6 james-2.3.2]# sh bin/run.sh
Using PHOENIX_HOME: /opt/james-2.3.2
Using PHOENIX_TMPDIR: /opt/james-2.3.2/temp
Using JAVA_HOME: /opt/java
Running Phoenix:
Phoenix 4.2
James Mail Server 2.3.2
Remote Manager Service started plain:4555
POP3 Service started plain:110
SMTP Service started plain:25
NNTP Service started plain:119
FetchMail Disabled
```

# 3. 配置james
默认配置启动James服务，只能给内网发送邮件，我们的要求是可以给外网的其他邮箱发邮件，比如163，qq,sina等邮箱发送邮件，那么我们必须修改James默认配置，接下来我们就来看看如何修改
还是打开config.xml文件，找到postmaster标签：
```
<postmaster>Postmaster@localhost</postmaster>   
……   
<servernames autodetect="true" autodetectIP="true">   
    <servername>localhost</servername>   
</servernames>   

```
把localhost该成你自己想要的邮箱域名， autodetect和autodetectIP设置为“false”，这里localhost假设改成 cp.com 如果开了一个帐号 cp ,那么他的邮件地址就是 cp@cp.com(注意两个localhost都要改)，改完如下：
```
<postmaster>Postmaster@cp.com</postmaster>
    <servernames autodetect="false" autodetectIP="false">
       <servername>cp.com</servername>
    </servernames>

```
修改理由：
1.autodetect设为true的话会自动侦测你的主机名，设成false会用你指定的servername
2.autodetectIP设为true会为你的servername加上IP，然而并不需要
3.servername改为你的server名字，如clararun.com
4.在C:\WINDOWS\System32\drivers\etc\host文件中添加127.0.0.1 cp.com
(此处就相当于注册了一个伪域名，之后可以用此邮箱向本地和外面的其他邮箱发邮件，但是接收不到外面邮箱发来的邮件，因为没有注册。)

实际上我把这个配置文件中所有的localhost都改成了我的域名；把所有的autodetect属性，修改为false，autodetectIP也设为false；查找所有myMailServer,替换为域名
找到
```
<mailet match="RemoteAddrNotInNetwork=127.0.0.1" class="ToProcessor">   
    <processor> relay-denied </processor>   
    <notice>550 - Requested action not taken: relaying denied</notice>   
</mailet>

```
把这一整段都注释掉。

找到下面这一段
```
<!-- <authRequired>true</authRequired>  -->
```
去掉它的注释。

找到dnsserver标签：
```
<dnsserver>
   <servers>
      <!--Enter ip address of your DNS server, one IP address per server -->
      <!-- element. -->
      <!--
       <server>127.0.0.1</server>
      -->
   </servers>
   <!-- Change autodiscover to false if you would like to turn off autodiscovery -->
   <!-- and set the DNS servers manually in the <servers> section -->
   <autodiscover>true</autodiscover>
   <authoritative>false</authoritative>
   <!-- Maximum number of entries to maintain in the DNS cache -->
   <maxcachesize>50000</maxcachesize>
</dnsserver>
```
在标签下加入：
```
<server>49.123.92.128</server> <!--你的IP地址-->
<server>202.197.96.16</server> <!--第一个DNS地址-->
<server>202.197.96.1</server> <!--第二个DNS地址-->
```
上面的三个IP要根据你的电脑情况来填写，第一个是你电脑的IP地址，也就是服务器地址，第二和第三个都是DNS地址，这三个地址都可以通过在cmd输入命令ipconfig中查看得到。
这样就算配置完成了，重新启动一下服务器。

# 4. 创建邮件账号
打开cmd，输入telnet localhost 4555,会提示你输入login id和password,这个id和password可以在config.xml中修改，CTRL+F查找password，把login和password的值换掉，默认是root和root。
```
<account login="root" password="!changeme!"/>
```
创建新用户的命令是adduser username password

例如

    adduser cp cp
    adduser jack jack
输入命令listusers可以查看所有用户。

下面是一些命令的含义：

![asd](http://upload-images.jianshu.io/upload_images/7628226-607d09ffad52ef2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 5. 使用邮件客户端收发邮件
这里介绍用Foxmail客户端来进行邮件的收发。

![image.png](http://upload-images.jianshu.io/upload_images/7628226-9451c17e6a9c19af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
填写E-mail地址和密码，地址是cp@cp.com，密码就是刚才用命令添加的cp。

![image.png](http://upload-images.jianshu.io/upload_images/7628226-c3a5aa795f6913a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
收件服务器和发件服务器都是你自己的ip地址，其他均为默认。
同理，我们再添加rose账户，就可以愉快地背着jack和rose通信了~

![image.png](http://upload-images.jianshu.io/upload_images/7628226-b92f4d3597bb638f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
至此，邮件服务器的搭建和客户端的测试已经完成了。
[文章参考](http://clararun.me/2017/05/14/%E7%94%A8Apache-James%E6%90%AD%E5%BB%BA%E9%82%AE%E4%BB%B6%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%94%B6%E5%8F%91%E9%82%AE%E4%BB%B6/)

# 6. 用JavaMail测试James
为了确保我们的设置正常运行，我们将编写一组快速发送消息和列出收件箱内容的类，模拟典型电子邮件客户端的基本功能。
```
import java.io.IOException;
import java.util.*;
import javax.mail.*;
import javax.mail.internet.*;

public class MailClient extends Authenticator{
	
  public static final int SHOW_MESSAGES = 1;
  public static final int CLEAR_MESSAGES = 2;
  public static final int SHOW_AND_CLEAR =
    SHOW_MESSAGES + CLEAR_MESSAGES;
  
  protected String from;
  protected Session session;
  protected PasswordAuthentication authentication;
  
  public MailClient(String user, String host)
  {
    this(user, host, false);
  }
  
  public MailClient(String user, String host, boolean debug)
  {
    from = user + '@' + host;
    authentication = new PasswordAuthentication(user, user);  //由于创建的user和password一样，所以传两个user就OK
    Properties props = new Properties();
    props.put("mail.user", user);
    props.put("mail.host", host);
    props.put("mail.debug", debug ? "true" : "false");
    props.put("mail.store.protocol", "pop3");
    props.put("mail.transport.protocol", "smtp");
    session = Session.getInstance(props, this);
  }
  
  public PasswordAuthentication getPasswordAuthentication()
  {
    return authentication;
  }
  
  public void sendMessage(
    String to, String cc, String subject, String content)
      throws MessagingException
  {
    System.out.println("SENDING message from " + from + " to " + to);
    System.out.println();
    MimeMessage msg = new MimeMessage(session);
    msg.addRecipients(Message.RecipientType.TO, to);  //添加收件人
    msg.addRecipients(Message.RecipientType.CC, cc);  //添加转发人
    msg.setSubject(subject);
    msg.setText(content);
    Transport.send(msg);
  }
  
  public void checkInbox(int mode)
    throws MessagingException, IOException
  {
    if (mode == 0) return;
    boolean show = (mode & SHOW_MESSAGES) > 0;
    boolean clear = (mode & CLEAR_MESSAGES) > 0;
    String action =
      (show ? "Show" : "") +
      (show && clear ? " and " : "") +
      (clear ? "Clear" : "");
    System.out.println(action + " INBOX for " + from);
    Store store = session.getStore();
    store.connect();
    Folder root = store.getDefaultFolder();
    Folder inbox = root.getFolder("inbox");
    inbox.open(Folder.READ_WRITE);
    Message[] msgs = inbox.getMessages();
    if (msgs.length == 0 && show)
    {
      System.out.println("No messages in inbox");
    }
    for (int i = 0; i < msgs.length; i++)
    {
      MimeMessage msg = (MimeMessage)msgs[i];
      if (show)
      {
        System.out.println("    From: " + msg.getFrom()[0]);
        System.out.println(" Subject: " + msg.getSubject());
        System.out.println(" Content: " + msg.getContent());
      }
      if (clear)
      {
        msg.setFlag(Flags.Flag.DELETED, true);
      }
    }
    inbox.close(true);
    store.close();
    System.out.println();
  }
}
```
这个mail client主要是为了让我们发送消息和显示或删除给定用户的服务器上可用的邮件列表。
此类还实现了Authenticator接口，可以很容易地检索电子邮件时管理登录过程。

我创建了两个构造函数，其中之一显式设置JavaMail调试标志。这将打印客户端/服务器协议交互到控制台，以便您可以看到发生了什么。较短的构造函数将此标志关闭。另外两个参数是用户名和主机。通过暗示，电子邮件地址可以从用户和主机派生。我们创建一个PasswordAuthentication可以通过接口getPasswordAuthentication()指定的方法返回的对象Authenticator。

该checkInbox()方法执行更多的工作，因为它需要列出消息，并且可选地擦除它们。也可以根据您用于模式参数的标志来清除消息而不查看它们。要实际获取消息，我们需要Store通过我们的会话对象获取引用，连接到服务器，然后打开收件箱文件夹。在我们引用该文件夹之后，我们可以遍历消息并显示或删除它们。

下面是mail client的测试类：
```
public class MailClientTest {
	
	public static void main(String[] args)
    throws Exception
  {
    // CREATE CLIENT INSTANCES
    MailClient cp = new MailClient("cp", "cp.com");
    MailClient rose = new MailClient("rose", "cp.com");
    MailClient jack = new MailClient("jack", "cp.com");
    String c1 = ContentModes.content1;
    String c2 = ContentModes.content2;
    
    // CLEAR EVERYBODY'S INBOX
    cp.checkInbox(MailClient.CLEAR_MESSAGES);
    rose.checkInbox(MailClient.CLEAR_MESSAGES);
    jack.checkInbox(MailClient.CLEAR_MESSAGES);
    Thread.sleep(500); // Let the server catch up
    
    // SEND A COUPLE OF MESSAGES TO BLUE (FROM RED AND GREEN)
    cp.sendMessage(
      "rose@cp.com,502966686@qq.com",
      "",
      "Testing rose from cp",
      "Dear" + "\n" +
      "This is a test message." +"\n"+
      "Good day! "); 
    
    rose.sendMessage(
      "cp@cp.com",
      "502966686@qq.com",
      "Testing cp from rose",
      "Dear" + "\n" +
      "This is a test message." +"\n"+
      "Good day! ");
    
    rose.sendMessage(
    	      "cp@cp.com",
    	      "502966686@qq.com",
    	      "New Situation!",
    	      c1);
    
    Thread.sleep(500); // Let the server catch up
    
    // LIST MESSAGES FOR BLUE (EXPECT MESSAGES FROM RED AND GREEN)
//    jack.checkInbox(MailClient.SHOW_AND_CLEAR);
    cp.checkInbox(MailClient.SHOW_MESSAGES);
    rose.checkInbox(MailClient.SHOW_AND_CLEAR);

  }

}
```

# 7. Matchers
james带来了许多标准匹配器即matcher。每个都实现了Matcher API，如下所示，并提供了现有MTA以及其他有用的扩展通用的功能。界面相当简单; 它包括一对生命周期方法，init()和destroy()，和一对记账方法，getMatcherInfo()和getMatcherConfig()，还有main方法--match()，它用来操作Mail对象。该Mail引用提供对容器状态，邮件消息和元数据的访问以进行处理。
```
public interface Matcher
{
  void init(MatcherConfig config);
  void destroy();
  String getMatcherInfo();
  MatcherConfig getMatcherConfig();
  Collection match(Mail mail);
}
```



以下是写好的Matcher接口。


![5~ABJ219E1V38RQCO1PGI}U.png] (http://upload-images.jianshu.io/upload_images/7628226-fd2bfc330b65aa8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![VU] (http://upload-images.jianshu.io/upload_images/7628226-0fda35a5d327bb6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 8. Mailet
许多james的功能是通过Mailet API实现的，习惯于用servlet的人会对此很熟悉。
与Matcher API一样，Mailet接口提供了init()和destroy()两个生命周期的方法。两种有返回值的方法。第一个， getMailetInfo()，返回一个String对象，其中包含了与mailet关联的作者，版本和版权等信息。第二个，getMailetConfig()，用来返回最近的mailet配置信息。
```
public interface Mailet
{
  void init(MailetConfig config);
  void destroy();
  String getMailetInfo();
  MailetConfig getMailetConfig(); 
  void service(Mail mail);
}
```
主进程在services()方法中进行，带有一个Mail对象参数。此对象提供对容器状态，邮件消息和要进行处理的元数据的附加访问。
以下是已经实现的mailet接口。

![1.png] (http://upload-images.jianshu.io/upload_images/7628226-a17dd0f5ef576862.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![%H15_BJOCH15(72DPA(U%D5.png] (http://upload-images.jianshu.io/upload_images/7628226-2c2bfc13d5efeb52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 9. Mailet和Matcher的简单尝试
Mailet API是一个用来创建邮件处理程序的简单的API,它被配置在邮件服务器端执行,分匹配器Matcher和Mailet的接口两种,匹配器根据特定的条件匹配邮件消息,并触发相应的Mailet。
Mailet的简单可编程接口可以用来做一些邮件处理,比如反垃圾邮件,检查邮件病毒以及邮件博客等等,利用移动设备可发送email的功能,可以做到手机通过mail发送信息到邮件服务器交给Mailet处理,形成移动博客的模型。

```
import javax.mail.MessagingException;

import org.apache.mailet.GenericRecipientMatcher;
import org.apache.mailet.MailAddress;
import org.apache.mailet.MatcherConfig;

public class MatcherTest1 extends GenericRecipientMatcher {
	 public boolean matchRecipient(MailAddress recipient)
	 throws MessagingException {
	 if(recipient.getUser().indexOf("rose")!= -1){
		
		 System.out.println(recipient.getHost());
		 System.out.println(recipient.getUser());
	 return true ;
	 }
	 return false;
	 }
 }
```

```
import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;

import org.apache.mailet.GenericMailet;
import org.apache.mailet.Mail;

public class MailetTest1 extends GenericMailet  {
	public void service(Mail mail) throws MessagingException {
		MimeMessage mmp ;
		mmp = (MimeMessage)mail.getMessage();
		mmp.setSubject("This is emergency!"+mmp.getSubject());
		mmp.setFileName(getMailetName()+"aha!");
		System.out.println("Received a piece of Email."); 
		
	}
}
```
接下来是配置部署。
　　Mailet跟Servlet一样，是服务器端程序，是不能直接在客户端运行的，必须要部署到服务器端方可生效。部署具体步骤如下： 
1、 将我们编写的Matcher和Mailet打包成jar文件； 
2、 在\james-2.3.1\apps\james\SAR-INF目录下新建一个lib文件夹； 
3、 将打包好的jar文件复制到刚刚新建的lib文件夹下； 
4、 打开config.xml配置文件，找到以下这段代码： 
```
<mailetpackages>      
<mailetpackage>org.apache.james.transport.mailets</mailetpackage>      
<mailetpackage>org.apache.james.transport.mailets.smime</mailetpackage></mailetpackages>      
<matcherpackages>      
<matcherpackage>org.apache.james.transport.matchers</matcherpackage>      
<matcherpackage>org.apache.james.transport.matchers.smime</matcherpackage></matcherpackages>    
```
前半部分是用于配置Mailet包所在位置，后半部分是用于配置Matcher包所在位置，我们把我们刚编写的Mailet和Matcher所在位置配置进去就可以了。配置后的结果如下： 
```
<mailetpackages>
      <mailetpackage>com.primeton.mailet.test</mailetpackage>
      <mailetpackage>org.apache.james.transport.mailets</mailetpackage>
      <mailetpackage>org.apache.james.transport.mailets.smime</mailetpackage>
   </mailetpackages>
   <matcherpackages>
      <matcherpackage>com.primeton.mailet.test</matcherpackage>
      <matcherpackage>org.apache.james.transport.matchers</matcherpackage>
      <matcherpackage>org.apache.james.transport.matchers.smime</matcherpackage>
   </matcherpackages>
```
这样就完成了包的配置。我们都知道，Mailet的工作过程是：首先由Matcher来匹配所接收到的邮件，然后提交给相应的Mailet处理，但是哪个匹配器对应哪个Mailet呢？我们还需要配置Mailet的对应关系。同样在config.xml中找到下面的代码：
```
<mailet match="All" class="PostmasterAlias"/>
```
在这段代码下面加入我们自己的Mailet：
```
<mailet match="All" class="PostmasterAlias"/>
         <mailet match="MatcherTest1" class="MailetTest1"/> 
```
这样就完成了我们自定义的Mailet的配置部署工作了。重启James服务器，则此Mailet即可生效。

# 10.总结
至此，james server的笔记结束了，它是一个很不错的开源邮箱项目，希望能帮到大家，谢谢。
