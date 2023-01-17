
# 开发文档

## 目录

- [开发前必读](#开发前必读)
    - [简介](#简介)
    - [通用说明](#通用说明)
        - [接口使用前提](#接口使用前提)
        - [接口授权](#接口授权)
        - [签名规则](#签名规则)
- [订单管理](#订单管理)

    - [统一下单](#统一下单)    
    - [订单查询](#订单查询)
    - [余额查询](#余额查询)
    - [统一下单【签名】](#统一下单签名)    
    - [订单查询【签名】](#订单查询签名)
    - [余额查询【签名】](#余额查询签名)
- [异步通知](#异步通知)

- [更新日志](#更新日志)
- [联系我们](#联系我们)

# 开发前必读
## 简介
## 通用说明
### 接口使用前提
本平台支持access_token与sign两种模式，请开发之前先获取适合自己团队的请求方式
### 接口授权
```
该授权方式于团队开发者读写团队下的数据。
```
#### 第一步： 获取团队授权 access_token
1. 这里获取的 access_token 与团队资源相关，安全等级非常高，必须只保存在服务器，不允许传给客户端。
2. 后续通过 access_token 获取团队数据等步骤，也必须从服务器发起。
3. 建议开发者使用中控服务器统一获取和刷新 access_token，其他业务逻辑服务器所使用的 access_token 均来自于该中控服务器，不应该各自去刷新，否则容易造成冲突，导致 access_token 覆盖而影响业务（多次刷新 access_token ，仅有最新一个有效）。
4. access_token 有效期为2小时，建议中控服务器不仅需要内部定时主动刷新，还需要提供被动刷新 access_token 的接口，这样便于业务服务器在API调用获知 access_token 已超时的情况下，可以触发 access_token 的刷新流程。

##### 接口地址
- 功能: 统一下单
- 请求方式: GET / POST
- 请求地址: v1/api/pay/accesstoken

```bash
GET https://api.supay01.com/v1/api/pay/accesstoken?mch_id=10086&secret=4ZWQBPN5SFP7MW6EQHXSCGLMAZVJZT4I
```
##### 参数说明

| 参数名 | 类型   | 是否必须 | 描述                 |
| ------ | ------ | :------: | -------------------- |
| mch_id | int |    是    | 唯一性商户编号       |
| secret | string |    是    | 颁发给商户的接口密钥 |


##### 返回说明
| 参数名     | 类型   | 描述                                        |
| ---------- | ------ | ------------------------------------------- |
| token      | string | 授权码                                      |
| expires_in | int    | 有效时间，默认为2小时，过期后须重新发起授权 |


```json
{
    "code": 0,
    "msg": "OK",
    "data": {
        "token": "ACCESS_TOKEN",
        "expires_in": 7200
    }
}
```
### 签名规则
```
该方式用于直接采用MD5签名的方式
```

#### 签名算法

##### 第一步
设所有发送或者接收到的数据为集合M，将集合M内非空参数值的参数按照参数名ASCII码从小到大排序（字典序），使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串stringA。

特别注意以下重要规则：
- 参数名ASCII码从小到大排序（字典序）；
- 参数名区分大小写；
- 商户传送的sign参数不参与签名，平台主动通知传送的sign参数不参与签名
- 接口可能增加字段，验证签名时必须支持增加的扩展字段

##### 第二步
在stringA最后拼接上key得到stringSignTemp字符串，并对stringSignTemp进行MD5运算，再将得到的字符串所有字符转换为小写，得到sign值。

举例：
假设传送的参数如下：
```
appid： wxd930ea5d5a258f4f
mch_id： 888888
device_info： 876543
body： body
nonce_str： ABCDEFG
```

对参数按照key=value的格式，并按照参数名ASCII字典序排序如下：
```go
stringA="appid=wxd930ea5d5a258f4f&body=body&device_info=876543&mch_id=888888&nonce_str=ABCDEFG";
```

拼接API密钥：
```go
stringSignTemp=stringA + "&key=api_key" //注：api_key为商户平台设置的密钥key
```


## 使用规则
欢迎您使用速支付（下称 “产品”）。在您正式开通接口服务之前，请您务必审慎阅读、充分理解在使用速支付接口时应当遵循的规则。您使用速支付接口服务即视为您已阅读并同意本规则全部内容的约束。

1. 速支付 API 接口服务为商户提供订单 webhook 接口配置等技术服务，具体服务内容请见 接口文档（下称 “服务”）。

2. 使用限制：每个商户账号的 mer_id 调用开放接口的频率不可超过 20 次 / 秒，否则可能会因高频调用而返回错误。


3. 速支付特别提醒您应妥善保管您的应用凭证（包括 mer_id 和 secret）。速支付与您共同负有维护应用凭证安全的责任。速支付会采取并不断更新技术措施，努力保护您的应用凭证在服务器端的安全。您需要采取特定措施保护您的应用凭证安全。您不得将速支付的应用凭证以任何方式与第三方共享。因您保管不善可能导致应用凭证被他人使用或共享秘钥引起的损失，责任由您自行承担。

4. 在您怀疑他人在使用您的应用凭证时，您同意立即通知速支付。如果您当前使用的应用凭证并不是您初始申请开通的或者通过速支付提供的其他途径获得的，但您却知悉该应用凭证，您不得用该应用凭证进行任何操作，并请您在第一时间通知速支付。



# 订单管理
## 统一下单
### 接口概述
- 功能: 统一下单
- 请求方式: POST
- 请求地址: /v1/api/pay/unifiedorder
### 请求参数
| 参数名       | 类型   | 是否必须 | 描述                                                           | 示例值                                  |
| ------------ | ------ | :------: | -------------------------------------------------------------- | --------------------------------------- |
| token        | string |    是    | 授权码                                                         | BA1B637A5C8D4B28ACB0889E559C5803        |
| channel_no   | int |    是    | 通道编码                                                       | 1                                       |
| subject      | string |    否    | 标题                                                           | subject                                 |
| out_order_no | string |    是    | 商户订单号                                                     | 20150320010101001                       |
| out_username | string |    否    | 商户会员用户名，用于后台展示用                                    | kehu1                               |
| money        | string |    是    | 金额，单位为元，精确到小数点后两位                               | 1000                                    |
| client_ip    | string |    是    | 客户IP                                                         | 0.0.0.0                                 |
| notify_url   | string |    是    | 异步通知地址，支付成功后将支付成功消息以POST请求发送给这个网址 | http://www.demo.com/recieve_notice.html |
| return_url   | string |    否    | 支付成功后跳转地址                                             | http://www.demo.com/paysucc.html        |
| param        | string |    否    | 透传参数                                                       | xxxxxxxxxxxxxxxxx                       |
| payer_name   | string |    否    | 付款人姓名，传入以后，付款人必须匹配                             | 马化腾                                   |
| timestamp    | int64  |    是    | 发送请求的时间戳,13位带毫秒                                    | 1626863144831                           |

### 响应参数
| 参数名            | 类型   | 描述               |
| ----------------- | ------ | ------------------ |
| mch_id            | int    | 商户编号           |
| order_no          | string | 平台订单号         |
| out_order_no      | string | 商户订单号         |
| money             | string | 订单金额           |
| real_money        | string | 真实的订单金额     |
| pay_url           | string | 支付链接           |
| expired_time      | string | 过期时间           |
| expired_timestamp | int64  | 过期时间时间戳毫秒 |

### 响应实例
#### 请求成功
```json
{
    "code":0,
    "msg":"ok",
    "data":{
        "mch_id":"1000123",
        "order_no":"202204157885214563228",
        "out_order_no":"20150320010101001",
        "money":"1000.00",
        "pay_url":"https://www.demo.com/#/?order_no=202204157885214563228",
        "expired_time":"2022-07-25 20:41:01",
        "expired_timestamp":1658752861000
    },
    "request_id":"ddec96d2165e4f3e8a642057db116983"
}
```

#### 请求失败

```json
{
    "code":2,
    "msg":"InvalidArgument",
    "data":{},
    "request_id":"ddec96d2165e4f3e8a642057db116983"
}
```

### 错误码
| 错误代码 | 错误描述                                   |
| -------- | ------------------------------------------ |
| 1        | 找不到数据                                 |
| 2        | token校验不通过                            |
| 4        | 账号状态异常                               |
| 9        | 插入数据库失败                             |
| 10       | 插入数据库失败                             |
| -1       | money 参数有误                             |
| -2       | channel_no 参数有误                        |
| -3       | out_order_no 参数有误                      |
| -4       | 找不到合适的银行卡，没有配置或者限额已超过 |
| -5       | 未开通对应通道的频道信息                   |


## 订单查询
### 接口概述
- 功能: 订单查询
- 请求方式: POST
- 请求地址: /v1/api/pay/query
### 请求参数
| 参数名    | 类型   | 是否必须 | 描述              | 示例值                           |
| --------- | ------ | :------: | ----------------- | -------------------------------- |
| token     | string |    是    | 授权码            | BA1B637A5C8D4B28ACB0889E559C5803 |
| channel_no | int   |    是    | 通道号            | 2  |
| mch_id    | int    |    是    | 商户号            | 100123                           |
| order_no  | string |    是    | 订单号,与out_order_no二选一            | 2015042321001004720              |
| out_order_no  | string |    是    |商户订单号, 与order_no二选一         | YY2015042321001004720              |
| timestamp | int    |    是    | 时间戳,13位带毫秒 | 1626863144831                    |

### 响应参数
| 参数名            | 类型   | 描述                                                           |
| ----------------- | ------ | -------------------------------------------------------------- |
| mch_id            | int | 商户号                                                         |
| order_no          | string | 平台订单号                                                     |
| out_order_no      | string | 商户订单号                                                     |
| state             | int    | 状态	0=未出码,1=交易失败,2=待支付,3=交易成功                   |
| money             | string | 订单金额                                                       |
| notify_succ       | int    | 1=通知成功，0=通知失败                                         |
| expired_time      | string | 过期时间                                                       |
| expired_timestamp | string | 过期时间时间戳毫秒                                             |
| notify_url        | string | 异步通知地址，支付成功后将支付成功消息以POST请求发送给这个网址 |
| pay_url           | string | 支付地址                                                       |

### 响应实例
#### 请求成功
```json
{
    "code":0,
    "msg":"ok",
    "data":{
        "mch_id":"1000122",
        "order_no":"2015042321001004720",
        "out_order_no":"20150320010101001",
        "state":0,
        "money":"1000.00",
        "notify_succ":"1",
        "expired_time":"2022-07-25 20:41:01",
        "expired_timestamp":"1658752861000",
        "notify_url":"https://your.domain.com/",
        "pay_url":"https://www.demo.com/#/?order_no=202204157885214563228",
    },
    "request_id":"ddec96d2165e4f3e8a642057db116983"
}
```

#### 请求失败

```json
{
    "code":1,
    "msg":"InvalidArgument",
    "data":{},
    "request_id":"ddec96d2165e4f3e8a642057db116983"
}
```

### 错误码
| 错误代码 | 错误描述          |
| -------- | ----------------- |
| 1        | 找不到数据        |
| 2        | 参数校验不通过    |
| -1       | order_no 参数有误 |


## 余额查询
### 接口概述
- 功能: 余额查询
- 请求方式: POST
- 请求地址: /v1/api/pay/balance
### 请求参数
| 参数名    | 类型   | 是否必须 | 描述              | 示例值         |
| --------- | ------ | :------: | ----------------- | -------------- |
| token     | string |    是    | 描述              | 12345223412342 |
| mch_id    | int    |    是    | 商户号            | 100123         |
| timestamp | int    |    是    | 时间戳,13位带毫秒 | 1626863144831  |
### 响应参数
| 参数名  | 类型   | 描述 |
| ------- | ------ | ---- |
| balance | string | 余额 |

### 响应实例
#### 请求成功
```json
{
    "code":0,
    "msg":"ok",
    "data":{
        "balance":"10000.00"
    },
    "request_id":"ddec96d2165e4f3e8a642057db116983"
}
```

#### 请求失败
```json
{
    "code":1,
    "msg":"InvalidArgument",
    "data":{},
    "request_id":"ddec96d2165e4f3e8a642057db116983"
}
```

### 错误码
| 错误代码 | 错误描述        |
| -------- | --------------- |
| 1        | 找不到数据      |
| 2        | token校验不通过 |
| 3        | 参数校验不通过  |


## 统一下单【签名】
### 接口概述
- 功能: 统一下单
- 请求方式: POST
- 请求地址: /v1/api/pay/sign/unifiedorder
### 签名字符串
```
channel_no=%d&client_ip=%s&mch_id=%d&money=%s&notify_url=%s&out_order_no=%s&out_username=%s&param=%s&payer_name=%s&return_url=%s&subject=%s&timestamp=%d&key=%s
```
### 请求参数
| 参数名       | 类型   | 是否必须 | 描述                                                           | 示例值                                  |
| ------------ | ------ | :------: | -------------------------------------------------------------- | --------------------------------------- |
| sign         | string |    是    | 签名                                                           | aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa        |
| mch_id       | int    |    是    | 商户编号                                                       | 1000010                                 |
| channel_no   | int |      是    | 通道编码                                                       | 1                                       |
| subject      | string |    否    | 标题                                                           | subject                                 |
| out_order_no | string |    是    | 商户订单号                                                     | 20150320010101001                       |
| out_username | string |    否    | 商户会员用户名，用于后台展示用                                   | kehu1                               |
| money        | string |    是    | 金额，单位为元，精确到小数点后两位                             | 1000                                    |
| client_ip    | string |    是    | 客户IP                                                         | 0.0.0.0                                 |
| notify_url   | string |    是    | 异步通知地址，支付成功后将支付成功消息以POST请求发送给这个网址 | http://www.demo.com/recieve_notice.html |
| return_url   | string |    否    | 支付成功后跳转地址                                             | http://www.demo.com/paysucc.html        |
| param        | string |    否    | 透传参数                                                       | xxxxxxxxxxxxxxxxx                       |
| payer_name   | string |    否    | 付款人姓名，传入以后，付款人必须匹配                             | 马化腾                                   |
| timestamp    | int64  |    是    | 发送请求的时间戳,13位带毫秒                                    | 1626863144831                           |

### 响应参数
| 参数名            | 类型   | 描述               |
| ----------------- | ------ | ------------------ |
| mch_id            | int    | 商户编号           |
| order_no          | string | 平台订单号         |
| out_order_no      | string | 商户订单号         |
| money             | string | 订单金额           |
| real_money        | string | 真实的订单金额     |
| pay_url           | string | 支付链接           |
| expired_time      | string | 过期时间           |
| expired_timestamp | int64  | 过期时间时间戳毫秒 |

### 响应实例
#### 请求成功
```json
{
    "code":0,
    "msg":"ok",
    "data":{
        "mch_id":"1000123",
        "order_no":"202204157885214563228",
        "out_order_no":"20150320010101001",
        "money":"1000.00",
        "pay_url":"https://www.demo.com/#/?order_no=202204157885214563228",
        "expired_time":"2022-07-25 20:41:01",
        "expired_timestamp":1658752861000
    },
    "request_id":"ddec96d2165e4f3e8a642057db116983"
}
```

#### 请求失败

```json
{
    "code":2,
    "msg":"InvalidArgument",
    "data":{},
    "request_id":"ddec96d2165e4f3e8a642057db116983"
}
```

### 错误码
| 错误代码 | 错误描述                                   |
| -------- | ------------------------------------------ |
| 1        | 找不到数据                                 |
| 2        | token校验不通过                            |
| 4        | 账号状态异常                               |
| 9        | 插入数据库失败                             |
| 10       | 插入数据库失败                             |
| -1       | money 参数有误                             |
| -2       | channel_no 参数有误                        |
| -3       | out_order_no 参数有误                      |
| -4       | 找不到合适的银行卡，没有配置或者限额已超过 |
| -5       | 未开通对应通道的频道信息                   |


## 订单查询【签名】
### 接口概述
- 功能: 订单查询
- 请求方式: POST
- 请求地址: /v1/api/pay/sign/query
### 签名字符串
```
mch_id=%d&order_no=%s&timestamp=%d&key=%s
``` 
### 请求参数
| 参数名    | 类型   | 是否必须 | 描述              | 示例值                           |
| --------- | ------ | :------: | ----------------- | -------------------------------- |
| sign      | string |    是    | 签名              | aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa |
| channel_no | int   |    是    | 通道号            | 2  |
| mch_id    | int    |    是    | 商户编号          | 1000010                          |
| order_no  | string |    是    | 订单号,与out_order_no二选一            | 2015042321001004720              |
| out_order_no  | string |    是    |商户订单号, 与order_no二选一         | YY2015042321001004720              |
| timestamp | int    |    是    | 时间戳,13位带毫秒 | 1626863144831                    |

### 响应参数
| 参数名            | 类型   | 描述                                                           |
| ----------------- | ------ | -------------------------------------------------------------- |
| mch_id            | int | 商户号                                                         |
| order_no          | string | 平台订单号                                                     |
| out_order_no      | string | 商户订单号                                                     |
| state             | int    | 状态	0=未出码,1=交易失败,2=待支付,3=交易成功                   |
| money             | string | 订单金额                                                       |
| notify_succ       | int    | 1=通知成功，0=通知失败                                         |
| expired_time      | string | 过期时间                                                       |
| expired_timestamp | string | 过期时间时间戳毫秒                                             |
| notify_url        | string | 异步通知地址，支付成功后将支付成功消息以POST请求发送给这个网址 |
| pay_url           | string | 支付地址                                                       |

### 响应实例
#### 请求成功
```json
{
    "code":0,
    "msg":"ok",
    "data":{
        "mch_id":"1000122",
        "order_no":"2015042321001004720",
        "out_order_no":"20150320010101001",
        "state":0,
        "money":"1000.00",
        "notify_succ":"1",
        "expired_time":"2022-07-25 20:41:01",
        "expired_timestamp":"1658752861000",
        "notify_url":"https://your.domain.com/",
        "pay_url":"https://www.demo.com/#/?order_no=202204157885214563228",
    },
    "request_id":"ddec96d2165e4f3e8a642057db116983"
}
```

#### 请求失败

```json
{
    "code":1,
    "msg":"InvalidArgument",
    "data":{},
    "request_id":"ddec96d2165e4f3e8a642057db116983"
}
```

### 错误码
| 错误代码 | 错误描述          |
| -------- | ----------------- |
| 1        | 找不到数据        |
| 2        | 参数校验不通过    |
| -1       | order_no 参数有误 |


## 余额查询【签名】
### 接口概述
- 功能: 余额查询
- 请求方式: POST
- 请求地址: /v1/api/pay/sign/balance
### 签名字符串
```
mch_id=%d&timestamp=%d&key=%s
``` 
### 请求参数
| 参数名    | 类型   | 是否必须 | 描述              | 示例值                           |
| --------- | ------ | :------: | ----------------- | -------------------------------- |
| sign      | string |    是    | 签名              | aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa |
| mch_id    | int    |    是    | 商户编号          | 1000010                          |
| timestamp | int    |    是    | 时间戳,13位带毫秒 | 1626863144831                    |
### 响应参数
| 参数名  | 类型   | 描述 |
| ------- | ------ | ---- |
| balance | string | 余额 |

### 响应实例
#### 请求成功
```json
{
    "code":0,
    "msg":"ok",
    "data":{
        "balance":"10000.00"
    },
    "request_id":"ddec96d2165e4f3e8a642057db116983"
}
```

#### 请求失败
```json
{
    "code":1,
    "msg":"InvalidArgument",
    "data":{},
    "request_id":"ddec96d2165e4f3e8a642057db116983"
}
```

### 错误码
| 错误代码 | 错误描述        |
| -------- | --------------- |
| 1        | 找不到数据      |
| 2        | token校验不通过 |
| 3        | 参数校验不通过  |


# 异步通知
```
该链接是通过【统一下单API】中提交的参数notify_url设置，如果链接无法访问，商户将无法接收到通知。

通知url必须为直接可访问的url，不能携带参数。示例：notify_url：“https://pay.demo.com/pay.html”

该通知以POST方式请求，编码：UTF8，JSON格式数据流

商户接收到通知后，返回“SUCCESS”字符串，平台将不再通知(SUCCESS 使用大写字母)；如果没有反馈，平台将在10分钟内，通知6次，之后将不再主动发起通知
```
## 参数说明
| 参数名       | 类型   | 描述                                | 示例值              |
| ------------ | ------ | ----------------------------------- | ------------------- |
| mch_id       | int    | 商户编号                            | 1000010             |
| money        | string | 订单金额                            | 1000.00             |
| notify_time  | string | 通知时间                            | 2006-01-02 15:04:05 |
| order_no     | string | 平台订单号                          | 2015042321001004720 |
| out_order_no | string | 商户订单号                          | CZX00901239888173   |
| param        | string | 透传参数                            | xxxxxxxxxxxxxxx     |
| state        | int    | 1未出码,2待支付,3交易成功,4交易失败 | 1                   |
| sign         | string | 签名，详情见签名规则                |

## 签名字符串
```
mch_id=%d&money=%s&notify_time=%s&order_no=%s&out_order_no=%s&param=%s&state=%d&key=%s
```



# 联系我们
任何建议和问题随时 吐个槽

官方tg: https://t.me/supay

官方网站: 
