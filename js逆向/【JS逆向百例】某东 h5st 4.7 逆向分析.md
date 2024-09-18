# 【JS逆向百例】某东 h5st 4.7 逆向分析
![](https://mmbiz.qpic.cn/mmbiz_jpg/n5gIG4eThrZXM8Qg8VlEWFQJibNrNfib4MZJar33ubYhfLFdBPmxgAPQ0qd2jXpG0K7ia44JkBNPXa7lb374P3AnA/640?wx_fmt=jpeg)

声明
--

**本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关！**

**本文章未经许可禁止转载，禁止任何修改后二次传播，擅自使用本文讲解的技术而导致的任何意外，作者均不负责，若有侵权，请在公众号【K哥爬虫】联系作者立即删除！**

前言
--

最近某东也是在不断的维护升级 h5st 参数， 原因就是逐渐 `VMP 化`，现在已经到了 `4.7` 版本了，也相对稳定下来了，那我们就来分析分析。

逆向目标
----

目标：某东 h5st 4.7 参数逆向分析

网站：aHR0cHM6Ly93d3cuamQuY29tLw==

流程分析
----

我们先抓包分析一下，随便找个有 h5st 参数的接口，我们可以直接看到它的版本：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nzIHPNX6Q1pfX8MLuelju6aBYFRTCZacpGC1kjAbia1ibHsU3c6gLkpWg/640?wx_fmt=png&from=appmsg)

还有个 x-api-eid-token 这是一个风控参数，生成位置如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5naxaMJMwHgEGKdtsia4MfJyjMauEbcysQ3A3UX0g60BobwuribCUsWOLg/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5n1qnNFams4xI1JqcpSeLbqpdefITPOS6kJkx2MI84qrOXDBiaKTDywaA/640?wx_fmt=png&from=appmsg)

也是接口返回，a，d 参数由大量的浏览器环境以及指纹信息生成，这边就不具体分析了，可以先写死，我们具体来看看 h5st 参数。

逆向分析
----

定位 h5st 参数的生成，可以直接通过搜索大法，或者找堆栈都能快速定位到生成的位置：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5n3JrSDxriaaD8wXoetEFfdvnLyhEvicThsZliaxlYfeQjwj7TUcH3fUVSA/640?wx_fmt=png&from=appmsg)

跟栈进去：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5naqSJ8yARx0xCQLC8rLrI8YzHxibh5DIFFPhMpnCqV9fNo8CDccVKCyA/640?wx_fmt=png&from=appmsg)

可以发现他将 colorParamSign 传入 window.PSign.sign 这个异步函数，返回的结果就有了 h5st 参数：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nsIbXzLDEiaaoWQiaMickLkPn1UUxqxCjxGGjSgBw0bxHbvaeIwoSFNCxg/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nFxl7Yo6bZfpATMzdWPu2goDfkXiaXhQEbCovrVrmGahEFZ3ryVyS8dw/640?wx_fmt=png&from=appmsg)

取了前面的查询参数，然后 colorParamSign 的 body 就是将前面查询参数的 body 经过 SHA256 加密，测试没有魔改，我们直接套库就行。

点击进入 window.PSign.sign 函数中就到了 VMP 文件了， 也是主要的加密文件：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nCBPiam2hP6icMgyk4jag8Zph9YElbuhmxMGribxnQvQVCeVyduYelDiaKg/640?wx_fmt=png&from=appmsg)

仔细观察一下，发现他将加密的主要流程，以及加密的函数大部分 VMP 化了，不过却留下不少的特征点：

*   方法一：
    

我们可以在所有的 vmp 操作的 call 的位置打上日志断点，进行分析，不过日志点比较多，不太推荐。

*   方法二：
    

根据留下的特征点，在关键的加密函数的位置上，断点分析，既能看到传入的明文，也能看到加密后的密文，能节省不少时间。我们搜索一下一些常见的加密函数 CryptoJS， AES，SHA， Base64 等等。

能找到差不多有 10 处左右进行加密，编码操作的函数，都写在一起，而且格式，特征都基本一致：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5ntEMicQUVqEdMsrAX56XZdmrN9enPSXpbbF82ZzLsECHibaU9Kv1neB5g/640?wx_fmt=png&from=appmsg)

传入一个对象，来接收导出的加密函数：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nGwzjg3jUsNYlxDJ4GxeUsO6leZJwEGLJs6NLMXay53yoJEMpj25MFg/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nFibwYx8236HYdwoUreGZwXn8LHccAZFOJEyM9r6NFKtgxm07qs6oRhQ/640?wx_fmt=png&from=appmsg)

可以发现里面还有标准的 ob 模式的混淆，主要导出的加密函数就是 o 函数，初始直接将 o 函数的返回值，赋值给 HS 对象，后续就传入 HS 对象到 o 函数再进行导出，所以 HS 对象应该包含所有的加密函数。

我们可以输出看看，找到如下位置断下，这里就是最后拼接生成 h5st 的位置：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nxfbebXO1zLhtKbDVlZRM1e02KH5bPyUSqnd6QfUghP01yPK7hnbbRg/640?wx_fmt=png&from=appmsg)

然后 输出 HS 对象：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nMflBPOIVAeHyNewduOULahBlX2SWNqQKib0kJNxibSY0uXYteiaGjib5Ow/640?wx_fmt=png&from=appmsg)

点击对应的加密函数，下断点，这样就能精准的知道是如何进行操作的（其实可以都打上断点）我这边求方便就只打下面这几个关键断点了：

*   **AES**
    
    ![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5ntzPIF7F7qJibK064FDI9HsibKdRuYXVrpcCP7hJho6nfcXvNice4zUa1Q/640?wx_fmt=png&from=appmsg)
      
    
*   **哈希**：
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5niadbMV8QYNDE6EKeedE8NKYCnicfsiamWXxfDVPiaPeTLBjJOHkobmqBBQ/640?wx_fmt=png&from=appmsg)

*   **Base64：** 
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nZrgPnblCVA25xibYb8AKUWJRGPtyZInuSYlRXeVeKSaq7Uuh63Yze8Q/640?wx_fmt=png&from=appmsg)

我们重新请求后，就在 AES 这断下了：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nZx93ccnPlibfqQqoCTmnbnk2uvWou9KmYC7gMQPdmsj14VCExzZQvvw/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nMe8QcrTyic33mySial4HWOfdumhpyPbbPaql8wiaRey5yDjf6ibVA40KGw/640?wx_fmt=png&from=appmsg)

这里的 n, a 应该就是 key，iv ，可能有人看不懂，没事我们可以转换一下：

`const CryptoJS = require('crypto-js');  
var Bytes ={  
    words: [1598895705, 1063548518, 1312043094, 1296456536],  
    sigBytes: 16  
};  
var key = CryptoJS.enc.Utf8.stringify(Bytes);  
console.log(key);  
var Bytes = CryptoJS.enc.Utf8.parse(key);  
console.log(Bytes);  
`

测试一下，发现  AES  跟标准的  AES  结果不一致，问题不大，我们继续往下走；

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nXSrbp4Y386X73OxLoWzib3AO222BJWpPddySFJDoFYklOpRqibpcPl0w/640?wx_fmt=png&from=appmsg)

将 AES 加密的结果进行 Base64 编码（魔改了的），我们可以在 return 的地方断下来：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5npJcxiaP18427v7Dkfa8IV1bg2mBdFicTvSiaJA06lR86BHe9LoQTWHgQg/640?wx_fmt=png&from=appmsg)

继续往下走：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nbibdIJomeicNAhCAxs4daGZfou29qxAGXxQMYKFX4gxIndCprU8U8ic2w/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nrl13roStgkp7DPe1ib5ISFetF7IrSicsJu2bnB0sRHHjdcM0yFicPFfWw/640?wx_fmt=png&from=appmsg)

我们向上找一下堆栈，进到 test 可以很明显的发现是如何进行操作的：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nbAvw76O6fSPaMWmg9l0bxnXrTRAiaiaERzA1ibmqt3Hr2NeXGCxnicKwPg/640?wx_fmt=png&from=appmsg)

然后找了一下这个 test 是怎么来的，发现是 `request_algo 接口` 返回，注意 `rd 和加密的函数都是动态的`：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5n3FsNWiay5GrAzict9aqKL4jyAJTkE2w1cJsC0SbsJlL4sfKFZibVJDtog/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nRHCLzHJREHl29DZKiajFJ4ghB710QBDL0nRer26rB5qibdWqysnMuTEQ/640?wx_fmt=png&from=appmsg)

唯一要具体分析的就是 `expandParams`，这个也是走的上面魔改的 AES 加密生成，`key 值不一样`，明文主要就是一些环境值，可以先固定，这里就不具体分析了。

我们再来输出一下加密后的值，以防后续调用：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5ngISyN2b5N7Ml1DCuPzA7RZuWXXOibeQZicW1vlJsWO8qyCicZicRJuEIHQ/640?wx_fmt=png&from=appmsg)

然后继续走，就断到这里了：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nCC6hPIOX2XDqjAKTzUCUElGls5KtBLcOnGhAowaQ2bxWgKF8iarkbng/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5n4vy3tlByD9MC4275L5LiaeCzCJLj0m5JjrIm0OApF5jdVWYhd0ZqG4w/640?wx_fmt=png&from=appmsg)

`'5571a8dda12c510ad1922b90d6b048aba648d9a8c2fc679927fc1e7102520489appid:search-pc-java&body:64ef977e15f853295527f3ce1f15fc980963c43f27c40066d26dd19b542d0bd4&client:pc&clientVersion:1.0.0&functionId:pc_search_s_new&t:17152462710115571a8dda12c510ad1922b90d6b048aba648d9a8c2fc679927fc1e7102520489'  
`

*   加密结果为：
    

`626bad7f7ebe126bb799ac619409778c8808864ad364bcb3d6a42bfb14af47b7  
`

仔细观察明文其实就是 上面 test 函数生成的加密结果 + colorParamSign 的拼接 + test 函数生成的加密结果。

不过我们不知道他走的是哪个加密函数，我们可以通过 HS 对象下的加密函数来一一确认，也可以在其他的加密函数上打上断点，可能有的大佬可以直接秒，哈哈，已经确认走的是 SHA256 加密函数：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5n2Wib3tYpI9LrwcAS9LibAbsaT5etkpBicxxrZlOB7ia4CE8wqhgHJPvW8A/640?wx_fmt=png&from=appmsg)

继续往下走就来到了最后拼接生成 h5st 参数的位置：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nibAUjkm8g86x4CUC5PZycSx6nAF1rVGSsibS4P50el6tiadkovDKEBricw/640?wx_fmt=png&from=appmsg)

*   e：就是上面 SHA256 加密函数生成的；
    
*   t ：是时间戳；
    
*   r ：由 t 转换生成。
    

`function format() {  
    var e = arguments.length > 0 && void 0 !== arguments[0] ? arguments[0] : Date.now()  
      , t = arguments.length > 1 && void 0 !== arguments[1] ? arguments[1] : "yyyy-MM-dd"  
      , r = new Date(e)  
      , n = t  
      , a = {  
        "M+": r.getMonth() + 1,  
        "d+": r.getDate(),  
        "D+": r.getDate(),  
        "h+": r.getHours(),  
        "H+": r.getHours(),  
        "m+": r.getMinutes(),  
        "s+": r.getSeconds(),  
        "w+": r.getDay(),  
        "q+": Math.floor((r.getMonth() + 3) / 3),  
        "S+": r.getMilliseconds()  
    };  
    return /(y+)/i.test(n) && (n = n.replace(RegExp.$1, "".concat(r.getFullYear()).substr(4 - RegExp.$1.length))),  
    w_(a).forEach((function(e) {  
        if (new RegExp("(".concat(e, ")")).test(n)) {  
            var t, r = "S+" === e ? "000" : "00";  
            n = n.replace(RegExp.$1, 1 == RegExp.$1.length ? a[e] : j_(t = "".concat(r)).call(t, a[e]).substr("".concat(a[e]).length))  
        }  
    }  
    )),  
    n  
};

console.log(format(1715246309293, "yyyyMMddhhmmssSSS"))  // 20240509171829293

`

*   n：就是上面第一次 AES 加密进行魔改 Base64 编码后的结果；
    
*   \_fingerprint：可以先写死，算法生成；
    
*   \_appid：也可以先写死，页面里获取；
    
*   \_token：`request_algo 接口`返回。
    

h5st 参数的生成都分析完，现在就差 HS 对象下的加密函数如何获取：

看似复杂，还有 ob 混淆等等，其实直接正常扣就行了，缺啥补啥，ob 混淆其实都没必要还原，手动扣就行，混淆的地方不多，大概有 10 处这样的函数，结构都一致：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nZ3khz459tUVbwZiaG7SAlLVhiaYXdesVPcp7H2jU1ctgicm3ELqtzzd4A/640?wx_fmt=png&from=appmsg)

扣完后，输出 HS 对象，就可以直接调用里面的加密函数，注意要校验和浏览器加密的结果一致：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nYsurcrdiay4UEWm00oIg5NKRsQBfwaictHM3GwcXFYibQSZTia7M9uHYPg/640?wx_fmt=png&from=appmsg)

结果验证
----

**注意**：正确的 h5st 不会出现 <Response \[403\]> 的情况，100% 可以拿到数据：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4LhHS9UKf9IvICa0sroyd5nxLlJgKWWgNekL6hYU69Cyytuicb1aJGYkEtAqDbyicynTKEpVicCviaLJw/640?wx_fmt=png&from=appmsg)

 ![](https://mmbiz.qpic.cn/mmbiz_gif/c6gqmhWiafypu8xMx0VGdz69F9YhVguXu6TVQKDQa8J5lNFRia8Hbf7H7whbxfuAf2SKclBR306xqWibSiaRbtYQaA/640)


