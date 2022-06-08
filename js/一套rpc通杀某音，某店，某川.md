# 一套rpc通杀某音，某店，某川
**声明：** 

     本篇文章及代码仅供学习交流，严禁用于商业用途，否则由此产生的一切后果均与作者无关

       众所周知字节系的接口千奇百怪，有的校验\_signature（这个\_signature 有的还要带 cookie 来算）, 有的校验 X-Bogus 值，有的带 cookie 就行，往下继续接着做 js 逆向算法还原，则会遭遇 jsvmp 这种逆起来非常耗费体力的加密方式，本文将通过 rpc 来实现一个比较通用的字节系加密解决方案。

**什么是 RPC？**

远程过程调用协议 RPC, 允许像调用本地服务一样调用远程服务，当然有着更广阔的应用，但对于爬虫来说，只需要知道我们是在调用浏览器的环境来执行我们自己的代码就可以了。

**怎么进行 rpc？**  

 我们以某店的订单接口作为例子，常规操作打个 url 包含 “\_signature” 的 xhr 断点，断到了下面这个位置

![](https://mmbiz.qpic.cn/mmbiz_png/v9a0CSGWGFzgBnetvtOxW3j2aib9o4l9sTfhibibIyfnWDcp2IGxuvb0ky4yjeZe5KzJRwwjd3a7Z339cxC8Ve2vQ/640?wx_fmt=png)

n 是一团参数不用管，为什么？因为我们已经知道了我们需要的更关键的信息 this 是一个 XMLHttpRequest 对象，我们需要的信息就包含在其中的\_url 属性里（已包含\_signature 和 x-bogs 值）  

![](https://mmbiz.qpic.cn/mmbiz_png/v9a0CSGWGFzgBnetvtOxW3j2aib9o4l9sCu6LcQw1cLD3NMJFiaO6XFk3NK3HWwecrjDIIs9wwXyKI3Cr7ClpvwA/640?wx_fmt=png)

通过在控制台打印，我们知道了 o 其实就是 XMLHttpRequest 的 send 方法  

![](https://mmbiz.qpic.cn/mmbiz_png/v9a0CSGWGFzgBnetvtOxW3j2aib9o4l9sicAEHjPPkXFppMULX1HBLVQPSf2KibicDwghJA2f6xE80gP7Wbqw7XVbA/640?wx_fmt=png)

这是最关键的点，为什么这套方案适用于字节系的大部分加密，因为字节都重写了这个 send 方法，不管他中间怎么加密，最终都是通过这个 send 把加密后的结果提交给服务器。  

既然有 send 那有没有 open 呢？直接搜 XMLHttpRequest.prototype.open 就可以到下面这个地方

![](https://mmbiz.qpic.cn/mmbiz_png/v9a0CSGWGFzgBnetvtOxW3j2aib9o4l9s8QrdDLj1s1BE4fmCZYSWz2gp1tBA5le0PETSiblnLxKQ6eYyxicRbgSg/640?wx_fmt=png)

熟悉 XMLHttpRequest 流程的话就会知道 open 这个方法将初始化请求参数以供 send 方法后续使用，到这里我们的 js 分析就结束了，什么 jsvmp？那玩意是人看的吗？狗都不看

回到最开始断点处，改写 o

![](https://mmbiz.qpic.cn/mmbiz_png/v9a0CSGWGFzgBnetvtOxW3j2aib9o4l9sFZLtRI6jic8YncRf7icr0Pd2bKHPA3FZUxp68ziadW4erSCnNpA0uIYdA/640?wx_fmt=png)

我们不能让它把请求发出去，那样的话状态就变了，这个 x 其实就是 this 也就是 XMLHttpRequest 对象我们把它的\_url 返回给我们，顺便打印出来（改写完记得把断点放开，不然后面调用就卡住了）  

整个流程就是我们程序调用 open 方法来传参，中间 js 怎么加密不管，最后 send 方法把\_url 返回给我们，怎么实现浏览器和我们程序之间的通信呢？这里选用 WebSocket 协议，只需一次连接就可以一直通信。网上找了个 nodejs 实现的 WebSocket 服务，功能也很简单，就是把程序发的信息给浏览器，把浏览器发的给程序。

    var ws = require("nodejs-websocket");console.log("开始建立连接...")var cached = {}var server = ws.createServer(function(conn){    conn.on("text", function (msg) {        if (!msg) return;        // conn.sendText(str)        // console.log(str);        if (msg.length > 1000){            console.log("msg 这是js代码")        }else{            console.log("msg", msg);        }        var key = conn.key;        if ((msg === "Browser") || (msg === "Python")){            // browser或者python第一次连接            cached[msg] = key;            console.log("cached",cached);            return;        }        console.log('ccc',cached);        console.log('key',key)        if (Object.values(cached).includes(key)){            console.log('server',server.connections.forEach(conn=>conn.key));            var targetConn = server.connections.filter(function(conn){                return conn.key !== key;            })            console.log('targetConn',targetConn.key);                        targetConn.forEach(conn=>{                conn.send(msg);            })        }        // broadcast(server, str);    })    conn.on("close", function (code, reason) {        console.log("关闭连接")    });    conn.on("error", function (code, reason) {        console.log("异常关闭")    });}).listen(8014)console.log("WebSocket建立完毕")// var server = http.createServer(function(request, response){// response.end("ok");// }).listen(8000);

来编写我们的 python 脚本

    import randomimport timefrom urllib.parse import urlencodefrom ws4py.client.threadedclient import WebSocketClientclass CG_Client(WebSocketClient):    def opened(self):        print("连接成功")        self.send("Python")    def closed(self, code, reason=None):        self.close()        print("Closed down:", code, reason)    def received_message(self, resp):        url = 'https://xxxxxxxxxxx.com' + str(resp)#接收浏览器返回的数据        print(url)def get_13timestamp():    t = time.time()    return int(round(t * 1000))def create_lid():    timestamp = get_13timestamp()    random_int = random.randint(1000,9999)    timestamp_cut = str(timestamp)[5:]    lid = timestamp_cut+str(random_int)    return liddef get_data():    cookie = "填自己的cookies"    #顺便传个cookie实现多账号切换    payload = {        "tab": "all",        "order_by": "create_time",        "order": "desc",        "ttl": "1654064855476",        "page": "2",        "pageSize": "10",        "appid": "1",        "__token": "在cookies里写个正则匹配一下",        "_bid": "ffa_order",        "aid": "4272",        "_lid": create_lid(),#lld参数就是时间戳简单地变换一下        "msToken": "在cookies里写个正则匹配一下",    }    api_url = '/api/order/searchlist?'    payload = urlencode(payload)    url = api_url + payload #注意传入的url和open方法传入的保持一致    ws.send(str([url, cookie])) #发送参数到浏览器if __name__ == "__main__":    ws = CG_Client("ws://127.0.0.1:8014/")    ws.connect()    time.sleep(1) # 注意要等待一下连接完成    get_data()    ws.run_forever()

编写浏览器的 js 脚本

    (function() {    var mess = document.getElementById("mess");    if(window.WebSocket){        ws = new WebSocket('ws://127.0.0.1:8014/');        ws.onopen = function(e){            // console.log("连接服务器成功");            ws.send("Browser");        }        ws.onclose = function(e){            console.log("服务器关闭");        }        ws.onerror = function(){            console.log("连接出错");        }        ws.onmessage = function(e){            var data = e.data//接收传进来的参数            var jsobj = eval("" + data);            cookie = jsobj[1];            url = jsobj[0];            document.cookie = cookie;//重置cookie来实现账号切换            var p = new XMLHttpRequest;//new 一个新的XMLHttpRequest对象            p.open("GET",url)//传参            url = p.send(null)//是我们刚刚改写的send方法，返回加密后的结果            console.log(url)            ws.send(url)        }        }})();

先运行 nodejs 的 WebSocket 服务，执行浏览器的 js 脚本，再跑 py 脚本，拿到返回的 url  

![](https://mmbiz.qpic.cn/mmbiz_png/v9a0CSGWGFzgBnetvtOxW3j2aib9o4l9sMglaGR8SZpzuaRq3sOf6FbALQlFibVicxSPFcHiaGgYIzK5k1uaTzDDZA/640?wx_fmt=png)

可以看到是有 X-Bogus 和\_signature 值的

测试一下（注意 headers 里 user-agent 和浏览器保持一致，去掉 "accept-encoding": "gzip, deflate, br", 不然乱码）  

![](https://mmbiz.qpic.cn/mmbiz_png/v9a0CSGWGFzgBnetvtOxW3j2aib9o4l9saXDa2dktcZthriabwCwpWm4oeWibW7VY6NrERYIaTTw4peSv7ibzL6u1w/640?wx_fmt=png)

搞定，数据正常返回

其他字节系的产品可能函数名略有不同但这套思路是通用（有的接口严格点切账号不止要改 cookie 还要改 localStorage 里的 tt_scid、ttcid）  

**总结：**   

rpc 的缺点是必须得挂着一个浏览器，当并发量过高时会有浏览器崩溃的问题，优点自然就是不用去分析 jsvmp 这种高度混淆的代码，通用性也还行。  

整篇文章还是面向新手，力求细致，写得比较繁琐（不是新手都去手撕 vmp 了，谁看这个）  

思路参考自 志远大佬的编程猫 js 逆向教学 
 [https://mp.weixin.qq.com/s/3sgODvF-v-yGe5IfoyXI8Q](https://mp.weixin.qq.com/s/3sgODvF-v-yGe5IfoyXI8Q)
