# 京东h5st 4.8逆向-CSDN博客
最开始扒的时候版本号还是4.1，现在已经到了4.8了，原来的AES也没有了，现在都是魔改的加密方法，还整成了vmp。跟了一下整个流程，和原来的流程大差不差，h5st变长了，一共分9部分。简单走一下流程。

第一个加密参数 body  
![](https://i-blog.csdnimg.cn/direct/c754a0fcab1341a4bf3afc7de39eb877.png)
  
这里就是原生的SHA256，直接调`crypto-js`生成。  
接着往下走跟到 `sign`方法里面

第二个参数也就是`h5st`第八段

![](https://i-blog.csdnimg.cn/direct/ec4297f68a7a4397aafdc193c859bc38.png)
  
跟进去就开始走vmp控制流了，对所有`call`这种调用方法的地方下断点。  
先是通过`_$y8`方法生成一系列的环境值

![](https://i-blog.csdnimg.cn/direct/a33009086ad5419fa7551e3e206050d5.png)

![](https://i-blog.csdnimg.cn/direct/1553fda2b60b4537a863f1a95ac7d892.png)
  
对于这个参数可以写死，里面随机字符串生成方法直接用的4.1版本的代码

```
`function bw() {
        var t, r = arguments.length > 0 && void 0 !== arguments[0] ? arguments[0] : {},
            n = r.size,
            e = void 0 === n ? 10 : n,
            o = r.dictType,
            i = void 0 === o ? "number" : o,
            u = r.customDict,
            a = "";
        if (u && "string" == typeof u) t = u;
        else switch (i) {
            case "alphabet":
                t = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
                break;
            case "max":
                t = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ_-";
                break;
            default:
                t = "0123456789"
        }
        for (; e--;) a += t[Math.random() * t.length | 0];
        return a
    }
    
console.log(bw({size: 11,dictType: "max",customDict: null}))` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24


```

然后用上面生成的对象转`json`，先调用的`Utf8.parse`，再调用`Base64.encode`生成第八段。  
接着走到下一个方法，它直接生成`h5st`，跳进方法里面下断点。

第三个参数是`h5st`第一段，日期格式化，直接用原来的代码

```
`_$vF = function(e) {return e.concat};

function xb() {
        var e = arguments.length > 0 && void 0 !== arguments[0] ? arguments[0] : Date.now()
          , t = arguments.length > 1 && void 0 !== arguments[1] ? arguments[1] : "yyyy-MM-dd"
          , r = new Date(e)
          , n = t
          , a = {
            "M+": r.getMonth() + 1,
            "d+": r.getDate(),
            "D+": r.getDate(),
            "h+": r.getHours(),
            "H+": r.getHours(),
            "m+": r.getMinutes(),
            "s+": r.getSeconds(),
            "w+": r.getDay(),
            "q+": Math.floor((r.getMonth() + 3) / 3),
            "S+": r.getMilliseconds()
        };
        return /(y+)/i.test(n) && (n = n.replace(RegExp.$1, "".concat(r.getFullYear()).substr(4 - RegExp.$1.length))),
        Object.keys(a).forEach((function(e) {
            if (new RegExp("(".concat(e, ")")).test(n)) {
                var t, r = "S+" === e ? "000" : "00";
                n = n.replace(RegExp.$1, 1 == RegExp.$1.length ? a[e] : _$vF(t = "".concat(r)).call(t, a[e]).substr("".concat(a[e]).length))
            }
        }
        )),
        n
      }

var b = Date.now();
var s = xb(b, "yyyyMMddhhmmssSSS");` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32


```

继续往下走就到了`test`方法里面，生成一串字符串`key`

![](https://i-blog.csdnimg.cn/direct/214a5109647f48cdbd9f9e5df6f6dc72.png)

下一个加密参数是`h5st`的第五段`Sign`，调用魔改的SHA256，传参和4.1版本的`__gensign`一样，前后为`test`生成的`key`，中间夹着查询信息拼接成的键值对。

最后一个参数是`h5st`的第九段`SignDefault`，和上面一样调用魔改的SHA256，加密参数也是两边是`key`中间夹着键值对。

整个过程调用的加密算法都是魔改的，可以先把加密算法扣下来。全抠下来，加上一些小方法不到2000行，最后导出直接调用就好。

![](https://i-blog.csdnimg.cn/direct/5ad8c3524cef407597d7d7b0906e99a9.png)

注意一点就是里面用到的那些大数组不要在定义的地方去复制，要在调用的时候去复制，有些中间做了变动。

最后放上访问成功的截图

![](https://i-blog.csdnimg.cn/direct/6c2a967076b2483b8ffa1106e1e7a997.jpeg)