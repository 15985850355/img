#  哔哩哔哩websocket协议逆向--制作一个属于自己的弹幕姬_Asadz__的博客-CSDN博客_python弹幕机
在一般的[爬虫](https://so.csdn.net/so/search?q=%E7%88%AC%E8%99%AB&spm=1001.2101.3001.7020)中都是用到Get Post来请求，这些请求方式都是无状态的服务端无法主动向客户端发送数据。

而websocket的出现就解决了这个问题 它是有状态的请求，这就意味着它允许服务端主动向客户端推送数据，使得客户端和服务端之间的数据交换变得更加简单。

方法

| 

名称

 | 

作用

 |
| 

close\[code\[, reason\]\]

 | 

关闭当前连接

 |
| 

send(data)

 | 

发送数据(排队)

 |

属性

| 

名称

 | 

作用

 |
| 

binaryType

 | 

使用二进制的数据类型连接

 |
| 

wbufferedAmount

 | 

没发送到服务器的字节数

 |
| 

extensions

 | 

服务器选择的扩展

 |
| 

onclose

 | 

指定关闭后的回调函数

 |
| 

onerror

 | 

指定失败后的回调函数

 |
| 

onmessage

 | 

指定从服务器接收到信息时的回调函数

 |
| 

onopen

 | 

指定连接成功后的回调函数

 |
| 

protocol

 | 

服务器选择的下属协议

 |
| 

readyState

 | 

当前服务器的链接状态

 |
| 

url

 | 

Websocket的绝对路径

 |

其中爬虫主要关注 send 方法的调用 和 onmessage 所绑定的回调函数。

因为服务器推送的数据第一时间会调用onmessage绑定的函数 可能进行 解码 或者 解密 等操作；

而 send 方法则是我们要发送的[心跳包](https://so.csdn.net/so/search?q=%E5%BF%83%E8%B7%B3%E5%8C%85&spm=1001.2101.3001.7020) 或者 进房数据等 与 服务端进行连接 和 保持连接 的时候会用到。

进房数据
----

随便找一个直播间点进去，打开开发者工具 (基操)

![](https://img-blog.csdnimg.cn/img_convert/f1b03896bd39909e2aa43199c4382aac.png)

可以在过滤一栏选择ws然后刷新网页获得ws请求的数据

![](https://img-blog.csdnimg.cn/img_convert/36eda7b2f311a4a086ca4f3b8e6d27c7.png)

这里可以看到 请求的数据有两个 其中sub是二进制数据 看不懂 但是在另一个包里 虽然是明文数据但是没有我们要的弹幕数据 进房数据等 所以铁定是sub这个包

然后从启动器中 直接定位到创建Websocket对象的位置并打下断点刷新

![](https://img-blog.csdnimg.cn/img_convert/90954ce1186cd311414b8bf251b9ef34.png)

网站停在了创建Websocket对象的位置

这个时候可以写一段js代码对send方法进行hook

```null
this.ws.send_ = this.ws.sendthis.ws.send = function(t) { 
```

![](https://img-blog.csdnimg.cn/img_convert/86d14d98cef5c978f6fc4fbb75f0813a.png)

然后继续执行send方法被调用的时候会被 hook断下来 断下来以后往上面一个栈看

![](https://img-blog.csdnimg.cn/img_convert/65c540ce435638ffd7c46694797ac928.png)

断在了30167行 但是很明显 t是在30163行的时候被赋值的

在 30163 行下断点然后刷新网页

![](https://img-blog.csdnimg.cn/img_convert/7cc60927014421c5da94a8cccda01642.png)

停下来以后 不难发现 i 就是进房数据

o.a.WS\_OP\_USER_AUTHENTICATION 根据名字来推断 可能是拿来确认我们发的是什么包

进入函数中细看

![](https://img-blog.csdnimg.cn/img_convert/828c65653ef50a4875fff931a4c882c4.png)

其中 r.a.getEncoder() 是 TextEncoder Object 的UTF8

o.a.WS\_PACKAGE\_HEADER\_TOTAL\_LENGTH 是一个常量 数值为 16 根据名字推断是头包长度

o.a.WS\_PACKAGE\_OFFSET 为 0 是用来偏移的 0 就是不偏移

随后 定义了一个 i 变量 是个视窗用来放编码后的数据

定义一个 s 变量用来存放 t(进房数据) UTF8编码后的数据

随后的 this.wsBinaryHeaderList 是一个列表

```null
"name": "Protocol Version",
```

使用了forEach对里头的每个元素都进行了处理

根据列表中的内容 还有处理的函数 可以知道包的样子

![](https://img-blog.csdnimg.cn/img_convert/e96e8ba24e6fcc0b586a5f2a9b205951.png)

头包长度也就固定为int32+int16+int16+int32+int32 也就是 16字节 这也说明了为什么 o.a.WS\_PACKAGE\_HEADER\_TOTAL\_LENGTH 的值是 16

然后进房数据 头包长度 协议版本 操作类型 都是固定的 所以需要更改的 也只有数据包长度

```null
    a = '{"roomid":' + str(roomid) + '}'return "000000{}001000010000000700000001".format(hex(16 + len(data))[2:]) + "".join(map(lambda x: x[2:], map(hex, data)))
```

弹幕、礼物等消息
--------

这里刷新网页回到Websocket对象被new的地方 因为send关注完了 就要关注onmessage了

因为 onmessage所指定回调函数会在message事件被触发的时候调用

这次直接将断点下在指定回调函数的地方

![](https://img-blog.csdnimg.cn/img_convert/83b78289342d250e8e301e20a8fe8772.png)

再次刷新调试后 发现data只在一个地方用 调试后也确实得到了能看懂的json数据

![](https://img-blog.csdnimg.cn/img_convert/e3ae42c31dc568ccfd5f96519f91cce7.png)

点进去查看

![](https://img-blog.csdnimg.cn/img_convert/3c87c3f0c0269f2b559e8b8bd1c33ac6.png)

初步分析来看 函数中用到了很多的 getInt32 等操作提取指定的数据 可以初步判定为TLV

TLV:

T 可以理解为 tag type 是用于标识编码格式信息

L 就是Length 表示数值长度

V 就是实际数值

一些常量:

*   o.a.WS\_PACKAGE\_OFFSET = 0 // 偏移量 0就是无偏移
    

*   o.a.WS\_OP\_MESSAGE = 5 // Message数据的op 其中这个op就表示这个数据包什么类型的
    

*   o.a.WS\_OP\_CONNECT_SUCCESS = 8 // 连接成功的 op
    

*   o.a.WS\_OP\_HEARTBEAT_REPLY = 3 // 心跳回复 op
    

*   o.a.WS\_PACKAGE\_HEADER\_TOTAL\_LENGTH = 16 // 包头种长度 其中包头会包含关于这个包的各种数据 比如这个包的长度 这个包有哪些类型
    

*   o.a.WS\_BODY\_PROTOCOL\_VERSION\_NORMAL = 0 // 普通消息
    

*   o.a.WS\_BODY\_PROTOCOL\_VERSION\_BROTLI = 3 // 压缩消息
    

```null
t.prototype.convertToObject = function(t) {if (n.packetLen = e.getInt32(o.a.WS_PACKAGE_OFFSET), this.wsBinaryHeaderList.forEach(function(t) {4 === t.bytes ? n[t.key] = e.getInt32(t.offset) : 2 === t.bytes && (n[t.key] = e.getInt16(t.offset))    n.packetLen < t.byteLength && this.convertToObject(t.slice(0, n.packetLen)), this.decoder || (this.decoder = r.a.getDecoder()),    !n.op || o.a.WS_OP_MESSAGE !== n.op && n.op !== o.a.WS_OP_CONNECT_SUCCESS)         n.op && o.a.WS_OP_HEARTBEAT_REPLY === n.op && (n.body = {count: e.getInt32(o.a.WS_PACKAGE_HEADER_TOTAL_LENGTH)for (var i = o.a.WS_PACKAGE_OFFSET, s = n.packetLen, a = "", u = ""; i < t.byteLength; i += s) {            a = e.getInt16(i + o.a.WS_HEADER_OFFSET);if (n.ver === o.a.WS_BODY_PROTOCOL_VERSION_NORMAL) {var c = this.decoder.decode(t.slice(i + a, i + s));                    u = 0 !== c.length ? JSON.parse(c) : null                } else if (n.ver === o.a.WS_BODY_PROTOCOL_VERSION_BROTLI) {var l = t.slice(i + a, i + s)                      , h = window.BrotliDecode(new Uint8Array(l));                     u = this.convertToObject(h.buffer).bodythis.options.onLogger("decode body error:", new Uint8Array(t), n, e)
```

else里的语句翻译成python大概就是

```null
    packetLen = int(data[:4].hex(), 16)    ver = int(data[6:8].hex(), 16)    op = int(data[8:12].hex(), 16)if len(data) > packetLen:        data = zlib.decompress(data[16:])            jd = json.loads(data[16:].decode('utf-8', errors='ignore'))
```

至此逆向完成

```null
        greeting = await ws.recv()await ws.send(bytes.fromhex(hb))    packetLen = int(data[:4].hex(), 16)    ver = int(data[6:8].hex(), 16)    op = int(data[8:12].hex(), 16)if len(data) > packetLen:         data = zlib.decompress(data[16:])            jd = json.loads(data[16:].decode('utf-8', errors='ignore'))if jd['cmd'] == 'DANMU_MSG': print(jd['info'][2][1], ': ', jd['info'][1])elif jd['cmd'] == 'SUPER_CHAT_MESSAGE_JPN': print(jd["data"]["user_info"]["uname"], ":", jd["data"]["message"])    a = '{"roomid":' + str(roomid) + '}'return "000000{}001000010000000700000001".format(hex(16 + len(data))[2:]) + "".join(map(lambda x: x[2:], map(hex, data)))    uri = "wss://broadcastlv.chat.bilibili.com/sub"    data_raw = bytes.fromhex(encode(roomid))async with websockets.connect(uri) as websocket:await websocket.send(data_raw)        tasks = [asyncio.create_task(sendHB(websocket)), asyncio.create_task(onmessage(websocket))]await asyncio.gather(*tasks)if __name__ == '__main__':    hb = "00000010001000010000000200000001"
```

| 

cmd

 | 

说明

 |
| 

DANMU_MSG

 | 

弹幕消息

 |
| 

WELCOME_GUARD

 | 

欢迎...老爷

 |
| 

ENTRY_EFFECT

 | 

舰长欢迎消息

 |
| 

WELCOME

 | 

欢迎...进入房间

 |
| 

SUPER\_CHAT\_MESSAGE

 | 

醒目留言

 |
| 

SEND_GIFT

 | 

投喂礼物

 |
| 

COMBO_SEND

 | 

连击礼物

 |
| 

ANCHOR\_LOT\_START

 | 

天选之人开始完整信息

 |
| 

ANCHOR\_LOT\_END

 | 

天选之人获奖id

 |
| 

ANCHOR\_LOT\_AWARD

 | 

天选之人获奖完整信息

 |
| 

GUARD_BUY

 | 

上舰长

 |
| 

USER\_TOAST\_MSG

 | 

续费了舰长

 |
| 

NOTICE_MSG

 | 

在本房间续费了舰长

 |
| 

ACTIVITY\_BANNER\_UPDATE_V2

 | 

小时榜变动

 |
| 

ROOM\_REAL\_TIME\_MESSAGE\_UPDATE

 | 

粉丝关注变动

 |