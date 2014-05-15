---
layout: post
title: Openfire发布webservice接口
category: notes---

之前碰到有人问怎么把openfire的接口发布成webservice，由于我们项目中采用的是Dubbo形式，还真就没实际操作过。不过估计原理差不多，得空研究一下。

首先还是注册插件：XMPPServer类里的loadModules方法中加入

    //加载数据集成模块。该模块为外部扩展模块，可能会不存在。
    loadModule("com.dylanvivi.openfire.common.DylanModule");

新建xmpp-plugin项目，通过jetty搭建webservice服务，[参考文章](http://blog.csdn.net/robert8803/article/details/8137461)。

运行结果：

![image](/assets/post-images/2014-05-15-824aeea7-faff-4962-e9de-06644f28774a.png)

openfire启动...init plugin...

![image](/assets/post-images/2014-05-15-8a790299-942f-4635-a625-a7db9614d790.png)

webservice服务

![image](/assets/post-images/2014-05-15-35f15ed9-f2aa-442b-8fa1-0f432c4e97e3.png)

服务发布成功

**测试代码**

  public class TestXFire {
  
  	public static void main(String[] args) {
  		TestXFire tMain = new TestXFire();
  		tMain.testWebservice();
  	}
  
  	public void testWebservice() {
  		Client client;
  		try {
  			client = new Client(new URL("http://127.0.0.1:8190/OpenfireWebService?wsdl"));
  			Object[] hello = client.invoke("sayHello", new Object[] { new String("dylan") });
  			System.out.println(hello[0]);
  		} catch (MalformedURLException e) {
  			e.printStackTrace();
  		} catch (Exception e) {
  			e.printStackTrace();
  		}
  	}
  }
  
输出：

![image](/assets/post-images/2014-05-15-30d615e1-9506-437b-ffe2-c94534bf3f94.png)



