## 全局配置：

在发出任何数据请求之前，先拿到全局配置:
`https://data.gj.com/data/config.json`

返回

```json
{
    "redirect":false,
    "code":200,
    "verify":false,
    "messages":{
        "data":{
            "passportServer":"https://webapi.gj.com/",
            "klineTypes":{
                "1":"1min",
                "5":"5min",
                "...": "..."
            },
            "tradeServer":"https://www.gj.com/",
            "klineServer":"https://data-cache.gj.com/",
            "notifyServer":"wss://data.gj.com/notify.do",
            "rate":{
                "DUSD":7.11375496612,
                "XEQ":0.23984432,
                "...": "..."
            },
            "dataServer":"https://data.gj.com/",
            "language":{
                "langs":[
                    {
                        "showName":"English",
                        "id":"en_US"
                    },
                    {
                        "showName":"简体中文",
                        "id":"zh_CN"
                    }
                ]
            },
            "device":{
                "timeZone":480,
                "lang":"zh_cn",
                "device":"phone",
                "version":"all",
                "platform":"android"
            },
            "symbols":{
                "symbols":[
                    {
                        "name":"USDT",
                        "symbols":[
                            {
                                "limitVolumeMin":0.0001,
                                "symbol":"btcusdt",
                                "priceUnit":"0.01",
                                "marketBuyMin":5,
                                "volumeUnit":"0.0001",
                                "name":"BTC/USDT",
                                "depths":[
                                    "0.01",
                                    "0.1",
                                    "1"
                                ],
                                "timeZone":"UTC",
                                "marketSellMin":0.001,
                                "limitPriceMin":0.01
                            },
                            "...": "..."
                        ]
                    },
                    {
                        "name":"BTC",
                        "symbols":[
                            "...": "..."
                        ]
                    },
                    "...": "..."
                ]
            }
        },
        "serverTime":"2019-09-09T14:51:40+0800"
    },
    "ok":true,
    "error":false
}
```

## 参数

1. 前端发出的 HTTP 请求中必须包含的全局参数

- 设备类型 device=PC|Android|Pad|TV
- 版本:version=1.0.0 每次编译都写在代码里的
- 语言（可选）:lang=zh_CN 如果不带会自动检测浏览器的语言做适配，默认为服务器的 Locale
- 时区（可选）: timeZone=480 数值，480 表示正 8 区，默认为服务器的 TimeZone
- 应用的 Id: appId 这个值根据后台协商定义:web, mobile, adroid,ios
- t 时间戳 聚合到分钟

示例

```js
{
    lang: zh_cn
    t: 26133477 // 时间戳 聚合到分钟
    appId: web
    device: pc
    version: 1.0.0
    timeZone: 480
}
```

## 响应

1. 服务器的统一数据格式：数据头+数据内容

```js
{
  error: false,
  verify: false,
  redirect: false,
  ok: true,
  code: 200,
  messages: {
  serverTime: "2018-07-16T08:09:56+0800",
  data: {
   //数据Body
  }
}
```

### CODE 说明

请求正常：code =200，ok =true
拿到数据内容

```js
if (json.code == 200) {
  var data = json.messages.data;
}
```

请求错误: code=500, error= true
拿到错误内容

```js
if (json.code==500){
    var errro = json.messages.error;
    {code:"错误编码",message:"错误内容"}
}
```

请求需要验证: code = 403, verify=true

```js
if (json.code==403){
    需要弹框验证，并带上verifyCode和verifyToken加上原有内容重发
 }
```

请求被重定向: code = 302, redirect=true

```js
if (json.code == 302) {
  url = json.messages.url; //新的请求地址
}
```

## 行情

URL: https://data-cache.gj.com  
API: /data/kline/price.json  
Method: GET

返回

```json
{
    "redirect":false,
    "code":200,
    "verify":false,
    "messages":{
        "data":{
            "batusdt":{
                "lastClose":0.1734,
                "high":0.1773,
                "amount":1271.4843,
                "vol":7300.6,
                "low":0.1695,
                "price":0.1719,
                "change":-0.87
            },
            "enjeth":{
                "lastClose":0.00045066,
                "high":0.00045927,
                "amount":1.26921933,
                "vol":2799.7,
                "low":0.00044373,
                "price":0.00044627,
                "change":-0.97
            },
            ...
        },
        "serverTime":"2019-09-09T13:57:43+0800"
    },
    "ok":true,
    "error":false
}
```

返回数据里没有格式化的交易对名称，如需该字段，可在 接口说明/全局配置/data.symbols 查看获取

## 建立 Socket 通信

URL: `wss://data.gj.com/notify.do?device=pc&version=1.0.0&timeZone=480&appId=web&t=1568011900469`

1. 发送 init 事件

参数

```json
{
  "action": "init",
  "data": {
    "t": 1568013864196
  }
}
```

2. 发送 symbol 事件

```json
{
  "action": "symbol",
  "symbol": "btcusdt"
}
```

3. 当接收到 event_trade_xxx 或 event_kline_xxx 事件时，可以调用 K 线接口获取最新数据。（xxx 表示具体的交易对）

## K 线接口

URL: https://data-cache.gj.com  
API：/data/kline/kline.json

参数

```js
{
    symbol: btcusdt // 交易对
    from: 26112050 // 开始时间
    to: 26133651 // 结束时间
    type: 1min // 分辨率
}
```

响应

```json
{
  "redirect": false,
  "code": 200,
  "verify": false,
  "messages": {
    "data": {
      "items": [
        {
          "volume": 6.6129,
          "high": 10158.83,
          "low": 10154.24,
          "timeDesc": "2019-09-09 16:50:00",
          "time": 1568019000000,
          "close": 10154.24,
          "open": 10158.83
        }
      ]
    },
    "serverTime": "2019-09-09T16:50:43+0800"
  },
  "ok": true,
  "error": false
}
```
