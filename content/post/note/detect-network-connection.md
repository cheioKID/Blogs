---
date: 2019-01-21
title: "Detect Network Connection (Dull Way)"
tags:
    - note
categories:
    - Note
comment: true
---

今天搞错了，盲目的查了好久怎么写，又测试了老半天，结果不是让我写的

我写的时候就一直奇怪为什么要让我写这么难的东西，根本不好实现，弄的我午饭都没好好吃

因为要删掉了，所以记下来纪念我空耗的两小时

`Constant.java`

```java
public static boolean isConnectedToNetwork(){
		try {
			int timeout = 2000;
			InetAddress[] addresses = InetAddress.getAllByName("www.baidu.com");
			for (InetAddress address : addresses) {
				if (address.isReachable(timeout)) {
					return true;
				}
			}
		} catch (UnknownHostException e) {
			return false;
		} catch (IOException e) {
			e.printStackTrace();
		}
		return true;
	}
```

`CaseTreeView.java`

```java
exec2.scheduleWithFixedDelay(new Runnable(){
            public void run() {

                if (null != reqParameter.getStrategy() && !((LongSimplexLinkServerStrategyImpl) reqParameter.getStrategy()).isClientConnected()) {
                    isNetworkAvailable = false;
                    System.out.println("......................连接已断开");
                }

                if (!isNetworkAvailable) {
                    if (Constant.isConnectedToNetwork()) {
                        Thread thread = new Thread(new MyClientThread(reqParameter));
                        thread.start();
                        opPanel.unlink.setEnabled(true);
                        getLatestUsedValueFromDB();
                        isNetworkAvailable = true;
                        System.out.println("......................已重新连接");
                    }
                }

            }
        }, 10, 10, TimeUnit.SECONDS);
```

