## Global Configuration：

Get the global configuration before making any data requests:
`https://data.gj.com/data/config.json`

Response

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

## Parameter

1. Global parameters that must be included in the outgoing HTTP request

- Equipment type: device=PC|Android|Pad|TV
- Version: version=1.0.0 
- Language (optional):lang=zh_CN|en_US   By default, the browser's language is automatically detected for adaptation.
- Time zone（optional）: timeZone=480    480 means positive 8 area，The default is the server's TimeZone
- Application Id: appId=web|mobile|adroid|ios
- Timestamp：t=26133477 The unit is minutes

Example

```js
{
    lang: zh_cn
    t: 26133477 // Timestamp The unit is minutes
    appId: web
    device: pc
    version: 1.0.0
    timeZone: 480
}
```

## Response

1. Unified data format of the server: data header + data content

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
   //Data Body
  }
}
```

### CODE Description

Request normal：code =200，ok =true
Get the data content

```js
if (json.code == 200) {
  var data = json.messages.data;
}
```

Request error: code=500, error= true
Get the error content

```js
if (json.code==500){
    var errro = json.messages.error;
    {code:"Error code",message:"Error content"}
}
```

Request requires verification: code = 403, verify=true

```js
if (json.code==403){
    Need to be verified by the bullet box, with verifyCode, verifyToken and original content resend
 }
```

Request is redirected: code = 302, redirect=true

```js
if (json.code == 302) {
  url = json.messages.url; //New request URL
}
```

## Markets

URL: https://data-cache.gj.com  
API: /data/kline/price.json  
(https://data-cache.gj.com/data/kline/price.json)

Method: GET

Response

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

There is no formatted symbol name in the response data. If you need this field, you can view it in Interface Description/Global Configuration/data.symbols

## Create Socket

URL: `wss://data.gj.com/notify.do?device=pc&version=1.0.0&timeZone=480&appId=web&t=1568011900469`

1. Send init event

Parameter

```json
{
  "action": "init",
  "data": {
    "t": 1568013864196
  }
}
```

2. Send symbol event

```json
{
  "action": "symbol",
  "symbol": "btcusdt"
}
```

3. When an event_trade_xxx or event_kline_xxx event is received, the K-line interface can be called to get the latest data. 
(xxx indicates a specific symbol)

## K-line API

URL: https://data-cache.gj.com  
API：/data/kline/kline.json
(https://data-cache.gj.com/data/kline/kline.json)

Parameter

```js
{
    symbol: btcusdt 
    from: 26112050 // Starting time
    to: 26133651 // End Time
    type: 1min
}
```

Response

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

## Order book & Market trades

URL: https://data-cache.gj.com  
API：/data/kline/symbolData.json
(https://data-cache.gj.com/data/kline/symbolData.json)

Parameter

```js
{
  symbol: "btcusdt";
  dataKey: "L3GckVpMcYVQOxNWrA0l1IfzHCsjDqFb"; //  Refer to "Create Socket". After sending the symbol event, you will receive the "actions: symbol" event, which will return datakey
}
```

Response

```json
{
  "redirect": false,
  "code": 200,
  "verify": false,
  "messages": {
    "data": {
      "items": [
		"bid0": { // Order book of different precision (Default options:bid0)
			"trade": {
				"volume": 32.484,
				"side": "SELL",
				"price": 0.16645
			},
			"buys": [],
			"sells": []
		}, 
		"bid1": {}, // Order book of different precision
		"bid2": {}, // Order book of different precision
		"price": {}, // 24H
		"trade": [ 	// Market trades list
			{
				"volume": 32.484,
				"side": "SELL",
				"price": 0.16645,
				"ctime": "2019-11-12T12:52:41+0800"
			}, {
				"volume": 5.143,
				"side": "SELL",
				"price": 0.16651,
				"ctime": "2019-11-12T12:51:41+0800"
			},
			...
		] 
      ]
    },
    "serverTime": "2019-09-09T16:50:43+0800"
  },
  "ok": true,
  "error": false
}
```
