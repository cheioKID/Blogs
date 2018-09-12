---
date: 2018-09-11
title: "Java FTP timeout Solution"
tags:
    - Java
    - Exception
categories:
    - Java
comment: true
---

因为API响应太久，FTP连接已经断开，会抛出异常

```java
org.apache.commons.net.ftp.FTPConnectionClosedException: FTP response 421 received.  Server closed connection.
2018-09-11 11:38:09,082 [main] ERROR com.example.sms.FtpUtil - fetch 1st waiting file error
	at org.apache.commons.net.ftp.FTP.__getReply(FTP.java:388)
	at org.apache.commons.net.ftp.FTP.__getReply(FTP.java:300)
	at org.apache.commons.net.ftp.FTP.sendCommand(FTP.java:523)
	at org.apache.commons.net.ftp.FTP.sendCommand(FTP.java:648)
	at org.apache.commons.net.ftp.FTP.sendCommand(FTP.java:622)
	at org.apache.commons.net.ftp.FTP.pasv(FTP.java:1045)
	at org.apache.commons.net.ftp.FTPClient._openDataConnection_(FTPClient.java:895)
	at org.apache.commons.net.ftp.FTPClient._openDataConnection_(FTPClient.java:785)
	at org.apache.commons.net.ftp.FTPClient.initiateListParsing(FTPClient.java:3409)
	at org.apache.commons.net.ftp.FTPClient.initiateListParsing(FTPClient.java:3339)
	at org.apache.commons.net.ftp.FTPClient.listFiles(FTPClient.java:3016)
	at org.apache.commons.net.ftp.FTPClient.listFiles(FTPClient.java:3069)
	at com.example.sms.test.main(test.java:126)
org.apache.commons.net.ftp.FTPConnectionClosedException: FTP response 421 received.  Server closed connection.
	at org.apache.commons.net.ftp.FTP.__getReply(FTP.java:388)
	at org.apache.commons.net.ftp.FTP.__getReply(FTP.java:300)
	at org.apache.commons.net.ftp.FTP.sendCommand(FTP.java:523)
	at org.apache.commons.net.ftp.FTP.sendCommand(FTP.java:648)
	at org.apache.commons.net.ftp.FTP.sendCommand(FTP.java:622)
	at org.apache.commons.net.ftp.FTP.pasv(FTP.java:1045)
	at org.apache.commons.net.ftp.FTPClient._openDataConnection_(FTPClient.java:895)
	at org.apache.commons.net.ftp.FTPClient._openDataConnection_(FTPClient.java:785)
	at org.apache.commons.net.ftp.FTPClient.initiateListParsing(FTPClient.java:3409)
	at org.apache.commons.net.ftp.FTPClient.initiateListParsing(FTPClient.java:3339)
	at org.apache.commons.net.ftp.FTPClient.listFiles(FTPClient.java:3016)
	at org.apache.commons.net.ftp.FTPClient.listFiles(FTPClient.java:3069)
	at com.example.sms.FtpUtil.fetchOneWaitingFile(FtpUtil.java:166)
	at com.example.sms.test.main(test.java:102)
org.apache.commons.net.ftp.FTPConnectionClosedException: Connection closed without indication.
	at org.apache.commons.net.ftp.FTP.__getReply(FTP.java:324)
	at org.apache.commons.net.ftp.FTP.__getReply(FTP.java:300)
	at org.apache.commons.net.ftp.FTP.sendCommand(FTP.java:523)
	at org.apache.commons.net.ftp.FTP.sendCommand(FTP.java:648)
	at org.apache.commons.net.ftp.FTP.sendCommand(FTP.java:622)
	at org.apache.commons.net.ftp.FTP.quit(FTP.java:904)
	at org.apache.commons.net.ftp.FTPClient.logout(FTPClient.java:1148)
	at com.example.sms.test.main(test.java:138)
```

试了用try catch包裹起来catch不到`FTPConnectionClosedException`

不管是`ftpClient.isConnected()`还是`ftpClient.isAvailable()`或者`ftpClient.isRemoteVerificationEnabled()`的值都为`True`，不能判断是否连接超时。

最后采用了一种拙劣的方法catch到了`FTPConnectionClosedException`

```java
try {
    //used for check connection
    ftpClient.listFiles();
} catch (Exception e) {

    Log.info(e.getMessage());

    if (!ftp.login(ftp_url,ftp_port,ftp_username,ftp_password)) {
    	Log.error("FTP reconnect and relogin failed with " + ftp_url + ":" + ftp_port + ",username: " + ftp_username + ",password: " + ftp_password);
    } else {
    	Log.info("successfully relogin for time out exception");
    }
}
```

