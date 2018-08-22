---
title: UI自动化测试之selenium超神之路
date: 2018-03-27 22:44:00
tags: 自动化测试
---
UI自动化测试之 selenium 超神之路
=========================


`作者：法狗狗自动化测试工程师 泰斯特`

## Background

随着时代车轮的转动以及科技行业的蓬勃发展 ， 越来越多的人开始选择从事 IT 行业 ， 在产品逐渐成型过程中，**测试扮演着重要的角色** 。 随着产品复杂度不断上升，迭代过程中需要做非常大量的**回归测试** ，这个时候相信很多测试人员都有着这样的想法： 如果能把**重复繁重**的回归测试任务交给机器去做该多好啊！ 作为一名有着些 _（**小**）_ 许 _（**白**）_ 开发能力的测试爱好者，经过一段时间的自学后，算是有所收获 。在这里 ， 仅给大家展示一下 UI 自 _（**操**）_ 动  _（**控**）_ 化 _（**浏**）_ 测 _（**览**）_ 试 _（**器**）_ 的**魅力**。

## Target

借助 selenium （UI 自动化测试框架），不仅可以实现 B\S 架构软件自动化测试 ， 也可以给工作、生活增添便利与色彩 **（只要是人通过使用浏览器能够做到的事情 ， 就没有不能自动化实现的！）**

##  实现第一个自动化测试用例

首先，我们来了解一下什么叫做**测试用例**：

>  测试用例（Test Case）即是为某个特殊目标而编制的一组测试输入、执行条件以及预期结果，用于测试某个程序路径或核实是否满足某个特定需求。 

然后，我们再来了解一下何谓**自动化测试**：

> 自动化测试是把以人为驱动的测试行为转化为机器执行的一种过程。通常，在设计了测试用例并通过评审之后，由测试人员根据测试用例中描述的规程一步步执行测试，得到实际结果与期望结果的比较。在此过程中，为了节省人力、时间或硬件资源，提高测试效率，便引入了自动化测试的概念。

好的 ， 在了解完基础知识以后我们来**模拟一次简单的、真实的测试场景**（验证百度首页链接的正确性） : 

![测试用例到此一游](https://fgg-admin.oss-cn-shanghai.aliyuncs.com/20180327144959402.png)

**我们通过执行脚本的方式 ， 自动化实现整个测试用例的执行过程** ，常见的UI自动化测试环境搭建步骤（selenium + java OR selenium + python）在这里就不做介绍了。

**直接上代码**

```java
public class BaiduTest {
	
	WebDriver driver = null;
	
	@BeforeSuite
	public void Setup() {
		System.setProperty("webdriver.chrome.driver",
				  "C:\\MyDownloads\\Download"
				  +"\\chrome-win32\\chromedriver.exe");
		driver = new ChromeDriver();
	}
	
	@Test(dataProvider = "data")
	 public void Baidu_Homepage_0001(String url, String expectTitle) {
		 driver.navigate().to(url);
		 Assert.assertEquals(driver.getTitle(),expectTitle);
	 }
	  
	 @DataProvider(name = "data")
	 public Object[][] data(){
		 return new Object[][]{
	           { "https://www.baidu.com", "百度一下，你就知道" },
		 };
	 }
	 
	 @AfterSuite
	 public void tearDown() {
		driver.quit();
	 }
```
这里采用了单元测试框架 **TestNG** 进行了用例和测试数据的封装 ，相信大家可以清晰的看到，在 **@Test** 中执行了整个测试用例 ， 并且可以发现 **@Test 方法名与测试用例编号一致** 。测试数据则由 **@DataProvider** 提供 。至于 **@BeforeSuite** 以及 **@AfterSuite** 则代表着整套测试的初始以及收尾工作 。以 TestNG Test 运行以后 ， 控制台输出如下：


```
PASSED: Baidu_Homepage_0001("https://www.baidu.com", "百度一下，你就知道")

===============================================
    Default test
    Tests run: 1, Failures: 0, Skips: 0
===============================================


===============================================
Default suite
Total tests run: 1, Failures: 0, Skips: 0
===============================================
```

可以看到在上面输出中看到了 **PASSED** 的字样，代表此用例通过。
然后我们可以在当前项目目录下找到TestNG自动生成的测试报告，如下所示：



![简陋的测试报告到此一游](https://fgg-admin.oss-cn-shanghai.aliyuncs.com/20180327150447178.png)


至此一个自动化测试用例就算是实现了 ， 当然 ， **这里只是最简单的展示了一个完整自动化测试流程的一部分** ， 但是我相信对于一些完全没有接触过UI自动化测试的人来说已经能够起到一定的**抛砖引玉**的作用 ， 在之后的章节中 ， 将会继续深入挖掘各种**骚操作**。

	
	

    

    

	



