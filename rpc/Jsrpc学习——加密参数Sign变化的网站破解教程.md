# Jsrpc学习——加密参数Sign变化的网站破解教程
点击上方 “**Python 共享之家**”，进行关注

回复 “**资源**” 即可获赠 Python 学习资料

今

日

鸡

汤

商女不知亡国恨，隔江犹唱后庭花。

大家好，我是皮皮。前几天给大家分享 jsrpc 的介绍篇，[Python 网络爬虫之 js 逆向之远程调用 (rpc) 免去抠代码补环境简介](http://mp.weixin.qq.com/s?__biz=MzU5MjY2ODEwMQ==&mid=2247489430&idx=1&sn=708286eaa0174d39b22eb7f94d0eba50&chksm=fe1d633cc96aea2aa1e97476a90038b006883b89898d69c47c6e9ac42c386746c90c07aa6eb4&scene=21#wechat_redirect)，还有实战篇，[Jsrpc 学习——网易云热评加密函数逆向](http://mp.weixin.qq.com/s?__biz=MzU5MjY2ODEwMQ==&mid=2247489716&idx=1&sn=dae3e78c0534dfb2bce8124bc7496d27&chksm=fe1d6c1ec96ae50897fc55872acd77bbaaadedfa73938a06511b75d0bbcf935e6faae0cde357&scene=21#wechat_redirect)，[Jsrpc 学习——Cookie 变化的网站破解教程](http://mp.weixin.qq.com/s?__biz=MzU5MjY2ODEwMQ==&mid=2247489706&idx=1&sn=2bd4d54ae99d66cbd958a503bda9e17d&chksm=fe1d6c00c96ae5167f22cdb782a78d5cbad69ffe50ade8c85e78849885a9a82d3446e7d7e8bc&scene=21#wechat_redirect)感兴趣的小伙伴可以戳此文前往。  

今天给大家来个 jsrpc 实战教程，让大家加深对 jsrpc 的理解和认识。下面是具体操作过程，不懂的小伙伴可以私我。

今天我们介绍的这个网站是 cookie 参数不变，但是加密参数 Sign 变化的一个网站，一起来使用 jsrpc 来攻破它吧！

![](https://mmbiz.qpic.cn/mmbiz_png/PwoXOzvn9Y7GOuAhGUib1Y4ko2zh0ITR1WEEKUdrQ9Zm2kXtdWYPFuAicBcGmicJe7ySwtDL8qKjtX4icsSv2QxiaXw/640?wx_fmt=png)

1、这里使用的网站是`87aed0b6bc8cb687d63dd7eee0f64d38`，MD5 加密处理过的。

2、需要提取 100 个网页中的数字，然后求和。

![](https://mmbiz.qpic.cn/mmbiz_png/PwoXOzvn9Y7GOuAhGUib1Y4ko2zh0ITR1Lj6Zf8dkXDHCboMiaCyiarMjnvlcWnrFziabRtR7pJO3C76t26Bbhd60w/640?wx_fmt=png)

3、打开浏览器抓包，然后打断点调试，依次点击右边的 Call Stack 内的东西，直到找到加密函数，里边的值对应请求参数即可判定。

![](https://mmbiz.qpic.cn/mmbiz_png/PwoXOzvn9Y7GOuAhGUib1Y4ko2zh0ITR1aLIJvjEboTtWbBQkhludydECYxw4iadyNuhG8WA79O7soJicdY9UJJAw/640?wx_fmt=png)

4、最终在这里找到了一堆人看不懂的东西。

![](https://mmbiz.qpic.cn/mmbiz_png/PwoXOzvn9Y7GOuAhGUib1Y4ko2zh0ITR1qARtRuGABaXNxGU4gdqnZw17wHfkx49fzcrBJbRcE5Au3U5BHGWGmA/640?wx_fmt=png)

5、仔细寻找，发现加密的函数在这里了。

![](https://mmbiz.qpic.cn/mmbiz_png/PwoXOzvn9Y7GOuAhGUib1Y4ko2zh0ITR1s6Ce0picice5hP55foTPWtZfNgzj6LicfZC5mrq3MYvmcxiaUWzDAJeTjw/640?wx_fmt=png)

6、之后可以在控制台输入指令`window.dcpeng = window.get_sign`，其中`window.get_sign`为加密函数。**注意**：这个地方挺重要的，很多时候我们会写成 ct.update()，这样会有问题！加了括号就是赋值结果，没加就是赋值整个函数！千差万别。

![](https://mmbiz.qpic.cn/mmbiz_png/PwoXOzvn9Y7GOuAhGUib1Y4ko2zh0ITR1o1QXdrCzAZG9lrPouQzSic0EMt81z3g8qpCoKRqx4pmTtAxy3jLibTrQ/640?wx_fmt=png)

7、关闭网页 debug 模式。**注意**：这个地方挺重要的，很多时候如果不关闭，ws 无法注入！

8、此时在本地双击编译好的文件`win64-localhost.exe`，启动服务。

![](https://mmbiz.qpic.cn/mmbiz_png/PwoXOzvn9Y7GOuAhGUib1Y4ko2zh0ITR1Sa4jHvIjusMJRf1FP2e8aKbSQDY4YuXicYN8ojl5anPXLNTpFSaxL9g/640?wx_fmt=png)

9、之后在控制台注入 ws，即将`JsEnv.js`文件中的内容全部复制粘贴到控制台即可 (注意有时要放开断点)。

![](https://mmbiz.qpic.cn/mmbiz_png/PwoXOzvn9Y7GOuAhGUib1Y4ko2zh0ITR1ak0iakFoYNII0yrmN4C8B78EXJkMhTpflVh9VUqbciaiaJOyWvoolf8FA/640?wx_fmt=png)

10、连接通信，在控制台输入命令`var demo = new Hlclient("ws://127.0.0.1:12080/ws?group=para&name=test");`

11、随后继续输入命令：

`// 注册一个方法 第一个参数 get_v 为方法名，  
// 第二个参数为函数，resolve 里面的值是想要的值 (发送到服务器的)  
// param 是可传参参数，可以忽略  
demo.regAction("get_para", function (resolve) {  
    dcpeng();  
 var res = window.sign  
    resolve(res);  
})  
`

也许有小伙伴会觉得奇怪，window.sign 明明是在 list 这个变量中，为啥我们通过 window.get_sign() 可以获取到，莫非 window.get_sign() 和 window.sign 返回的值是一样的？其实 window 是整个全局，它只是声名一个 list 对象里面有 signature 等于全局的 sign，这个全局的 sign 的值通过 window.get_sing() 得到。

dcpeng() 就是一个函数，里面写的最后结果就是 window.sign=window.get_sign()，并没有 return 东西。

12、之后就可以在浏览器中访问数据了，打开网址 `http://127.0.0.1:12080/go?group={}&name={}&action={}&param={}` ，这是调用的接口 group 和 name 填写上面注入时候的，action 是注册的方法名，param 是可选的参数，这里续用上面的例子，网页就是：`http://127.0.0.1:12080/go?group=para&name=test&action=get_para`

![](https://mmbiz.qpic.cn/mmbiz_png/PwoXOzvn9Y7GOuAhGUib1Y4ko2zh0ITR12oicTvnGjj7Osx0tu93l2GN5oJaMia5rlibrXXWdGpuHs6o4gsficFQQJQ/640?wx_fmt=png)

13、如上图所示，我们看到了那个变化的参数 v 的值，直接通过 requests 库可以发起 get 请求。

14、现在我们就可以模拟数据，进行请求发送了。

15、将拷贝的内容可以丢到这里进行粘贴：`http://tool.yuanrenxue.com/curl`

![](https://mmbiz.qpic.cn/mmbiz_png/PwoXOzvn9Y7GOuAhGUib1Y4ko2zh0ITR1Gng6PLngp4DlNtAbgJZNtQpcaFaMhK1L4eVNce2zaEHFlibBVgpaBow/640?wx_fmt=png)

16、之后将右侧的代码复制到 Pycharm 中即可用，非常便利。

17、之后就可以构造请求了，加一个整体循环，然后即可获取翻页的内容，整体代码如下所示。

\`import requests  
import json

cookies = {  
    'session': '6c78df1c-37aa-4574-bb50-99784ffb3697.Qcl0XN6livMeZ-7tbiNe-Ogn8L4',  
    'v': 'A7s8gqX6XgjWtmKFwCNKPNdQSpQgEM9-ySWTzq14lzDRLtVKNeBfYtn0IxW-',  
}

headers = {  
    'Connection': 'keep-alive',  
    'Accept': 'application/json, text/javascript, _/_; q=0.01',  
    'X-Requested-With': 'XMLHttpRequest',  
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36 Edg/97.0.1072.69',  
    'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',  
    'Origin': '[http://spider.wangluozhe.com'](http://spider.wangluozhe.com'),  
    'Referer': '[http://spider.wangluozhe.com/challenge/2'](http://spider.wangluozhe.com/challenge/2'),  
    'Accept-Language': 'zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6',  
}  
all_data = \[]  
for page_num in range(1, 101):  
    sign_url = '[http://127.0.0.1:12080/go?group=para&name=test&action=get_para](http://127.0.0.1:12080/go?group=para&name=test&action=get_para)'  
    sign = requests.get(url=sign_url).json()["get_para"]  
    # print(sign)  
    data = {  
      'page': f'{page_num}',  
      'count': '10',  
      '\_signature': sign  
    }  
    print(f'Crawlering page {page_num}')

    response = requests.post('87aed0b6bc8cb687d63dd7eee0f64d38', headers=headers, cookies=cookies, data=data, verify=False).json()  
    for item in response["data"]&#x3A;  
        all_data.append(item["value"])  
        # print(item["value"])

print(sum(all_data))

\`

![](https://mmbiz.qpic.cn/mmbiz_png/PwoXOzvn9Y7GOuAhGUib1Y4ko2zh0ITR19b6UKvs5D81ZyTRibUKZHDusNErzH44f2vsPIWhjn4o8SEj6Q3Qkawg/640?wx_fmt=png)

运行结果如上图所示，和网页上呈现的数据一模一样。

18、至此，请求就已经完美的完成了，如果想获取全部网页，构造一个 range 循环翻页即可实现。

17、也欢迎大家挑战该题目，我已经挑战成功了，等你来战！

![](https://mmbiz.qpic.cn/mmbiz_png/PwoXOzvn9Y7GOuAhGUib1Y4ko2zh0ITR16vVEdH5NWoUUAAYOiciaZibq443zdlsm8RUznXaFicFvYqvFKwneseNrzA/640?wx_fmt=png)

### 总结

大家好，我是皮皮。这篇文章主要给大家介绍了 jsrpc 的实战教程，使用 jsrpc 工具可以在网络爬虫过程中事半功倍，无需仔细的去扣环境，去一步步逆向，只一个黑盒的模式，我们就拿到了想要的结果，屡试不爽。

初次接触 jsrpc 的小伙伴可能看不懂，这里还有黑哥录制的一个视频，大家可以对照着视频进行学习，地址：`https://www.bilibili.com/video/BV1EQ4y1z7GS`，黑哥全程无声演示，视频的 BGM 很大，建议大家可以静音播放，领会其中奥义。

关于 jsrpc 工具，可以点击原文前往获取。

小伙伴们，快快用实践一下吧！如果在学习过程中，有遇到任何问题，欢迎加我好友，我拉你进 Python 学习交流群共同探讨学习。

![](https://mmbiz.qpic.cn/mmbiz_png/PwoXOzvn9Y4HHpEYUuX6JPxnGySltZ0f2DMn3YuFaT1HwAYoicEoJ0c3DCmDypoCak84StmUt0Lmj4AQPqraFyg/640?wx_fmt=png)

****\*\*****---**--****\*\*******---\***\*---**\*\*\*\*****---**--****\*\*******---****************\*\*\*\***************** End ****\*\*****---**--****\*\*******---**--**---****\*\*****---**--****\*\*******-****************\*\*\*\*****************  

往期精彩文章推荐：

-   [Python 网络爬虫之 js 逆向之远程调用 (rpc) 免去抠代码补环境简介](http://mp.weixin.qq.com/s?__biz=MzU5MjY2ODEwMQ==&mid=2247489430&idx=1&sn=708286eaa0174d39b22eb7f94d0eba50&chksm=fe1d633cc96aea2aa1e97476a90038b006883b89898d69c47c6e9ac42c386746c90c07aa6eb4&scene=21#wechat_redirect)
-   [一文带你了解 Python Socket 编程](http://mp.weixin.qq.com/s?__biz=MzU5MjY2ODEwMQ==&mid=2247489440&idx=1&sn=0b72a71d0243833d7b4ea95806cc9802&chksm=fe1d630ac96aea1c44a9025dfdd27111c516a50301f9b1ce6782f1485e3ef3892fdef9170f26&scene=21#wechat_redirect)
-   [盘点一道 Python 列表合并的基础题目 (列表推导式)](http://mp.weixin.qq.com/s?__biz=MzU5MjY2ODEwMQ==&mid=2247489410&idx=1&sn=89c912f81a7e32a880d93f3d7c7b7286&chksm=fe1d6328c96aea3eba488e60169dbd3ba94841131550a4adbe3d1e881be591834b63f7c22660&scene=21#wechat_redirect)
-   [手把手教你使用 Pandas 实现 DataFrame 分组条件查找值](http://mp.weixin.qq.com/s?__biz=MzU5MjY2ODEwMQ==&mid=2247489397&idx=1&sn=2dc274f7553e108abfa47b7dda535048&chksm=fe1d63dfc96aeac99e6ed46f8b882fe88b8400730c79fbd8aafc43a171b56af73012a27fc1c8&scene=21#wechat_redirect)  

![](https://mmbiz.qpic.cn/mmbiz_png/peDq5Y9hmZaHvs19XSuQEGbkast75g4uziam64mHRseaJibQEIZGUgwWthoqHAiakAXcicszKuT0OgAbXM2k2hiaSyA/640?wx_fmt=png)

欢迎大家**点赞，\*\***留言，\***\* 转发，**转载，\*\*\*\* 感谢大家的相伴与支持

想加入 Python 学习群请在后台回复【**入 \*\***群 \*\*】

万水千山总是情，点个【**在看**】行不行 
 [https://mp.weixin.qq.com/s/1ndHDUx-nxtqW3KFpm-GHg](https://mp.weixin.qq.com/s/1ndHDUx-nxtqW3KFpm-GHg)
