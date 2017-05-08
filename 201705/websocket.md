# WebSocket 协议

以前当我们创建一个可以双向通信的实时web应用时，我们需要采用http轮询的方式来实现。这样的话，显然会有很多的问题，例如：每一个消息都有额外的http header增加了不必要的开销，服务器端需要维持两个tcp连接用来发送和接收消息，同样客户端JavaScript需要维持连接之间的映射关系。

在这种背景下，WebSocket协议诞生了，它是一种真正的长连接，很轻松实现服务器端的推送功能。

## OverView

协议主要包括两个部分：握手和传输数据。

## 握手，WebSocket的握手是基于HTTP来完成的。  

* Client 需要发起一个WebSocket连接时，首先先发送一个http请求，header的格式如下:

```js
    GET /chat HTTP/1.1
    Host: server.example.com
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
    Origin: http://example.com
    Sec-WebSocket-Protocol: chat, superchat
    Sec-WebSocket-Version: 13
```

其中 "Connection : Upgrade" 和"Upgrade: websocket 是客户端请求切换到Websocket协议。

"Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ=="是客户端随机生成的字符串经过base64编码之后的结果。这个key用来做客户端检查用的，具体怎么用，后面再说。

* 服务器端接收到相应的请求后，响应的http header 如下：

```js
    HTTP/1.1 101 Switching Protocols
    pgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

其中"Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=" 是服务器端拿到客户端发过来的"Sec-WebSocket-Key"的值之后经过计算得到的，具体的计算过程为先将key与字符串"258EAFA5-E914-47DA-95CA-C5AB0DC85B11"连接在一起得到新的字符串，再将新的字符串经过sha-1算法计算哈希，最后把得到的结果经过base64编码，就是Accept的值。

如果这个Accept的值生成的不正确，那么浏览器控制台会有相应的提示说，"Sec-Websocket-Accept"这个字段的值不正确。导致连接不能建立。

## 数据传输

* 在WebSocket协议中，数据的传输是通过帧序列进行的。具体数据帧的格式如下所示：

```
 0 1 2 3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-------+-+-------------+-------------------------------+
 |F|R|R|R| opcode|M| Payload len | Extended payload length       |
 |I|S|S|S| (4)   |A| (7)         | (16/64)                       |
 |N|V|V|V|       |S|             | (if payload len==126/127)     |
 | |1|2|3|       |K|             |                               |
 +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
 | Extended payload length continued, if payload len == 127      |
 + - - - - - - - - - - - - - - - +-------------------------------+
 |                               |Masking-key, if MASK set to 1  |
 +-------------------------------+-------------------------------+
 | Masking-key (continued)       | Payload Data                  |
 +-------------------------------- - - - - - - - - - - - - - - - +
 :                 Payload Data continued ...                    :
 + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
 |                 Payload Data continued ...                    |
 +---------------------------------------------------------------+

```

| 名称    | 长度  |  解释 | 
| ------  | ----- | ------ | 
| FIN | 1 bit | 表示是否是整个消息的最后一帧|
| RSV1,RSV2,RSV3 | 1 bit | 保留位，一般为0 | 
| opcode | 4 bits | 定义这一帧的类型|
| Mask  | 1 bit  | 为1时表示内容需要用掩码进行异或运算|
| Masking-key | 0 0r 4 bits | 掩码异或运算所用的值|
|Paylaod length | 7bits or 7+16 bits or 7+64 bits| payload 的长度|

* opcode 的类型定义

opcode表示帧的类型。具体的定义如下

| 值 | 类型|
|----|-----|
| 0 | 这一帧是上一帧的延续|
| 1 | 这一帧是text frame|
| 2 | 表示binary帧|
| 3-7 | 保留值|
| 8 | 表示关闭连接|
| 9 | ping | 
| A | pong |
| B-F | 保留值|

## 掩码

协议里面提出所有客户端发到服务器端的帧MASK位必须为1，也就是必须要经过掩码异或运算。为什么这样规定呢？难道是用来加密，加密的话，解密的算法是已知的，显然不是为了加密。  

网上查了下，如下

> 原因是因为有一些代理会以为websocket是个普通的http请求，在对请求数据转发的时候会解释请求。如果发现类似HTTP 请求头的时候就会转发相应的请求到服务器，如果请求数据是故意伪造的，那么有些代理就会向目标地址发起请求。这个可以用来攻击目标服务或者故意污染代理服务，导致其他用户会访问伪造的数据而被攻击

不是很明白是什么意思。

## 计算具体payload data的长度

得到上图 Payload len的值

* 如果在0-125之间的话，那么它就是Payload data length
* 如果为126的话，那么它后面2个字节表示的无符号整数为length
* 如果为127的话，那么它后面8个字节表示的无符号整数位length

## 解析与生成数据帧

对位运算一直不敏感，网上查了下，很开心，能看懂。 

具体的算法如下，直接拿的[次钴碳酸](https://www.web-tinker.com/article/20306.html)

```js
function decodeDataFrame(e) {
    var i = 0, j, s, frame = {
        //解析前两个字节的基本数据
        FIN: e[i] >> 7, Opcode: e[i++] & 15, Mask: e[i] >> 7,
        PayloadLength: e[i++] & 0x7F,
    };
    frame.length = frame.PayloadLength;
    //处理特殊长度126和127
    if (frame.PayloadLength == 126)
        frame.length = (e[i++] << 8) + e[i++];
    if (frame.PayloadLength == 127){
        var len = 0;
        for(var j  = 7; j >= 0;j--){
            len += (e[i ++ ] << (j * 8));
        }
        frame.length = len;
    }
    //判断是否使用掩码
    if (frame.Mask) {
        //获取掩码实体
        frame.MaskingKey = [e[i++], e[i++], e[i++], e[i++]];
        //对数据和掩码做异或运算
        for (j = 0, s = []; j < frame.length; j++)
            s.push(e[i + j] ^ frame.MaskingKey[j % 4]);
    } else s = e.slice(i, frame.PayloadLength); //否则直接使用数据
    //数组转换成缓冲区来使用
    s = new Buffer(s);
    //如果有必要则把缓冲区转换成字符串来使用
    if (frame.Opcode == 1) s = s.toString();
    //设置上数据部分
    frame.PayloadData = s;
    //返回数据帧
    return frame;
}

```

```js
function encodeDataFrame(e) {
    var s = [], o = new Buffer(e.PayloadData), l = o.length;
    //输入第一个字节
    s.push((e.FIN << 7) + e.Opcode);
    //输入第二个字节，判断它的长度并放入相应的后续长度消息
    //永远不使用掩码
    if (l < 126) s.push(l);
    else if (l < 0x10000) 
    s.push(126, (l & 0xFF00) >> 8, l & 0xFF);
    else s.push(
        127, 0, 0, 0, 0, //8字节数据，前4字节一般没用留空
        (l & 0xFF000000) >> 24, (l & 0xFF0000) >> 16, (l & 0xFF00) >> 8, l & 0xFF
    );
    //返回头部分和数据部分的合并缓冲区
    return Buffer.concat([new Buffer(s), o]);
}
```


## 未完待续....





