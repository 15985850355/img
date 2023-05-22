# xhs xs _webmsxyw 纯算法还原盗用代码请注明出处搬来搬去真的很下头！_孤独的三毛的博客-CSDN博客
本文以教学为基准、本文提供的可操作性不得用于任何商业用途和违法违规场景。  
本人对任何原因在使用本人中提供的代码和策略时可能对用户自己或他人造成的任何形式的损失和伤害不承担责任。  
**最新版 x-s**

没露任何版权请审核员认真对待谢谢。

**【2023.05.22】 更新全站接口通用  
【2023.05.22】 更新全站接口通用  
【2023.05.22】 更新全站接口通用  
【2023.05.22】 更新全站接口通用  
【2023.05.22】 更新全站接口通用  
【2023.05.22】 更新全站接口通用  
【2023.05.22】 更新全站接口通用  
【2023.05.22】 更新全站接口通用  
【2023.05.22】 更新全站接口通用  
【2023.05.22】 更新全站接口通用  
【2023.05.22】 更新全站接口通用  
【2023.05.22】 更新全站接口通用  
【2023.05.22】 更新全站接口通用  
【2023.05.22】 更新全站接口通用  
【2023.05.22】 更新全站接口通用**

行业内那么多卷王，割韭菜卖书，是某人自行体会，当然喜欢卷，我也加入你们吧。  
外面已经出了很多卷王断点js文件位置那就省略此处，我上一篇5.16号订阅文章也有提到。  
**如下在下列下断点：**   
xs 是XYW_+base64串解开之后列如下，我们要解决的就是payload，看起来像是某种加密hex值也有可能算法被改动，但无所谓最原始的还原它管他是shape F5还是什么加密全都是浮云：  
{“signSvn”:“50”,“signType”:“x1”,“appId”:“xhs-pc-web”,“signVersion”:“1”,“payload”:“e9423b92a875c431a21f8fc2cd67ec8bd8d3674299ca1e896a26df2c28ca150165036dfc754135538351f196afad9e7016e2e3bfb89e2da1dad61c5041d6ac2bdad61c5041d6ac2bba1c4fcc5520a3e3f9f6b953ff819f7c4d3964d610405efa91ae217323940b20e19535f9336a7ea0dfaf19bed5594393e0c2a4378e693534832b5e08bfc964352c7015c6dd9d1a5d8a61966753da79a54ba7d2180c677dc8ce36caaac351ee4885f461badf9c6ae3454a1421cd035e614eb25aa99d40d80b”}

第一步：  
![](https://img-blog.csdnimg.cn/57766c7078c24c7cb16e47500ca2f54a.png)

第二步：  
最终控制台跟踪上面的window._websxyw方法可以定位到源js截图如下：  
![](https://img-blog.csdnimg.cn/231b6f52bb9441ab9210c8addeaddd23.png)

第三步：在所有计算出打印logo日志，下断点如下，如右键494位置，addlogpoint：  
![](https://img-blog.csdnimg.cn/dcaedf79e8ae45db9c0d6417971f0081.png)

第四步：  
刷新页面日志会显示在控制台，我的日志已保存到本地日志：  
![](https://img-blog.csdnimg.cn/7a13a06ed8634e1aa7086015c6401524.png)
  
从这里我们能看到加密字符串，分析过程就暂不讲解了 日志可以追溯算法。  
x1=md5(url除去域名的字符串) 如果是post方法就是拼接一下就行了。  
x2是环境校验值，固定写死。  
x3=其实是cookie里面的可有可无，如有最后计算的hex会长一点但没有实际校验它。  
x4=时间戳  
![](https://img-blog.csdnimg.cn/8528cf185ef34c51af95fb137e5b7602.png)
  
第五步：  
是第二次对上面的串进行加密，也是可以根据日志还原算法，但其实当我还原后才发现这他喵的是就是base64此处浪费时间可避免你们踩坑：  
![](https://img-blog.csdnimg.cn/43035bcc602d49128b20434e967b94a3.png)

![](https://img-blog.csdnimg.cn/3a765bade25b4bec88f2d2f63b746a03.png)

![](https://img-blog.csdnimg.cn/b0d12fb29071432fb1a1f0cf6152422d.png)

第六步：  
将上面的base64加密  
![](https://img-blog.csdnimg.cn/beb842588cc0408aae7c611978bef4f8.png)
  
那我们就从上到下全文搜索日志第一个字符找到第一个生成规律：  
![](https://img-blog.csdnimg.cn/752b93782d2243c1b85b919387539b37.png)
  
很奇怪我们这里只定位到了一个长串而且位数貌似是8位，这里也要记录一下，控制台打印一下第一个值的charCodeAt()可以看到是233。  
![](https://img-blog.csdnimg.cn/67f85cae33e746c1b803e415b98b6087.png)

当前位置继续往上找233很快就能看见他是由一个数>>>24而来：  
![](https://img-blog.csdnimg.cn/8d3670a5adce4ee69a3adbd560d9f3ba.png)
  
依次往上找可以看到数-552697436的由来，一直往上即可，每次记录一下每个值的关联，再由第一个数开始向下看写出算法即可。这仅仅是第一个数  
![](https://img-blog.csdnimg.cn/5b87820609b74cd7bc4e6f6b0264bcec.png)
  
第二个数是B ：66  
![](https://img-blog.csdnimg.cn/ffc5d5869abb4b95b9d88e1647436f3e.png)
  
从日志不难看出是计算的大数 >>>16 &255  
![](https://img-blog.csdnimg.cn/fc947405ff6542749df943a99738e934.png)

第三个其实也能发现是 >>>8 &255，第四第五。。。第八自己找 日志我会放下面。  
依此往下8个一组。

最终代码实现如下：  
![](https://img-blog.csdnimg.cn/b59a1465d1b44788aab5b977521892e7.png)

![](https://img-blog.csdnimg.cn/b2144f53139b474989bc15456d1f7dd8.png)
  
此文章仅仅提供学习使用，请勿使用非法途径，否则一切法律后果自负。