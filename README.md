# Brnn

## 安装说明

### 项目依赖
1. MySql数据库5.7以上

### 数据库初始化
初始化脚本文件 `dbinit.sql`

## API 文档

### 约定
1. 方括号[]内的参数表示可选参数；
2. 尖括号<>内的参数表示必选参数；
3. API返回一般格式为
```
{
  "error": "error_code",
  "errorMessage": "错误信息",
  "traceActivity": "错误日志跟踪ID",
  "result": <<具体API返回的结果对象>>
}
```
4. error_code列表
* > **success** API调用成功，对应response code 200-299
* > **invalid_data** 无效的请求数据，对应response code 400
* > **object_not_found** 所请求的对象不存在，对应response code 404
* > **dependency_failed** API所依赖的第三方服务调用失败，对应response code 500
* > **object_conflict** 所请求的对象与现有的对象冲突，对应response code 409
* > **more_data** 需要更多的数据完成请求对象（具体API会具体说明需要什么数据），对应response code 200-299
* > **service_not_ready** 系统服务还没有就绪，可以重试，对应response code 503
* > **invalid_credentials** 无效的用户令牌或者非法的用户信息，对应response code 401或者400
* > **password_change_required** 需要修改密码（发生在登录API上），对应response code 200-299
* > **not_supported** API不支持所请求的操作，请联系客服
* > **unknwn** 未知系统错误，对应response code 500
* > **challenge_failed** 用户密保问题答案中有无效的答案
* > **bank_account_required** 需要完善银行卡信息，该错误信息一般由申请兑奖时发生，对应response code 400

5. 如果客户端启用Cookie则登录以后调用所有的API的时候不需要传入用户令牌，同时所有的可选参数都可以不必传入

### 获取验证码图片
```
GET /Home/CaptchaImage?token=[token]
```
API返回PNG二进制图片数据。
> **token**是可选参数，值由客户端传入，必须是唯一标识，用来关联验证码图片和验证码，登录和注册的时候需要将*token*传入。
>  如果*token*为空，那么服务器将会把验证码和用户会话信息通过cookie关联。

### 用户注册
```
POST /api/User?token=[验证码token]&captcha=<验证码>&referrer=[推广码]

Content-Type: application/json
{
  "accountName": "手机号码",
  "password": "密码"
}
```
API对象
```
Content-Type: application/json
{
  "id": 用户ID,
  "accountName": "手机号码"
}
```
如果返回的error_code为more_data则需要完善用户的密码保护问题和答案，具体参考相应的API说明。

### 用户名检测
```
HEAD /api/User?account=<手机号码>&token=[token]&captcha=<captcha>
```
返回
* > 400 无效请求
* > 404 用户名不存在
* > 200 用户已经存在

### 用户登录
```
POST /api/User/Login?token=[token]&captcha=<captcha>

Content-Type: application/json
{
  "accountName": "手机号",
  "password": "密码"
}
```
返回对象
```
{
  "userId": UserID,
  "token": "用户令牌，用于访问用户其他信息的令牌"
}
```

### 获取用户账户信息
```
GET /api/User/<id>

x-rainier-ticket: <用户令牌>
```
返回对象
```
{
    "error": "success",
    "errorMessage": null,
    "traceActivity": null,
    "result": {
        "id": UserID,
        "role": {
            "id": RoleID
        },
        "accountName": "13345670894",
        "avatar": null,
        "nickName": null,
        "balance": 999998,
        "savingBalance": 0,
        "email": null,
        "phone": null,
        "promotionCode": "7AMAAAAAAAA",
        "status": 1,
        "hasSubAccounts": false,
        "createTimestamp": 0,
        "passwordLastSet": 0,
        "createTime": "1970-01-01T00:00:00",
        "passwordLastSetTime": "1970-01-01T00:00:00"
    }
}
```

### 获取当前登录用户信息
```
GET /api/User/Me?[promotionUrl=true]

x-rainier-ticket: <用户令牌>
```
返回对象
```
{
    "error": "success",
    "errorMessage": null,
    "traceActivity": null,
    "result": {
        "id": UserID,
        "role": {
            "id": RoleID
        },
        "accountName": "13345670894",
        "avatar": null,
        "nickName": null,
        "balance": 999998,
        "savingBalance": 0,
        "email": null,
        "phone": null,
        "promotionCode": "7AMAAAAAAAA",
        "promotionUrl": "http://t.cn/EMgONSD",
        "status": 1,
        "hasSubAccounts": false,
        "createTimestamp": 0,
        "passwordLastSet": 0,
        "createTime": "1970-01-01T00:00:00",
        "passwordLastSetTime": "1970-01-01T00:00:00"
    }
}
```

* > 要获取用户推广URL的时候带参数promotionUrl=true，返回的推广URL在promotionUrl属性里面。

### 获取用户密保问题列表
```
GET /api/SecurityQuestion
```
返回对象
```
[
  {
    "id": ID1,
    "question": "问题1"
  },
  {
    "id": ID2,
    "question": "问题2"
  },
  ...
]
```

### 保存用户密保答案
```
POST /api/User/<id>/SecurityAnswers

x-rainier-ticket: <用户令牌>
Content-Type: application/json
[
  {
    "question": {
      "id": 问题ID1
    },
    "answer": "答案1"
  },
  {
    "question": {
      "id": 问题ID2
    },
    "answer": "答案2"
  },
  ...
]
```
返回对象：
```
{
  "accountName":"手机号"
}
```

### 更新用户信息
```
PUT /api/User/<id>

x-rainier-ticket: <用户令牌>
Content-Type: application/json
{
  "nickName": "昵称",
  "email": "Email",
  "phone": "电话号码"
}
```
返回对象：
```
{
  "accountName":"手机号"
}
```

### 更改用户密码
```
POST /api/User/<id>/UpdatePassword

x-rainier-ticket: <用户令牌>
Content-Type: application/json
{
  "oldPassword": "现有密码",
  "newPassword": "新密码"
}
```
返回对象：
```
{
  "accountName":"手机号"
}
```

### 更改用户取款密码
```
POST /api/User/<id>/UpdateWithdrawPassword

x-rainier-ticket: <用户令牌>
Content-Type: application/json
{
  "oldPassword": "现有密码",
  "newPassword": "新密码"
}
```
返回对象：
```
{
  "accountName":"手机号"
}
```
可能的错误
* > 404 错误代码 object_not_found 现有密码不对

### 转账到保险柜
```
POST /api/User/<id>/TransferToSaving

x-rainier-ticket: <用户令牌>
Content-Type: application/json
{
  "amount": 金额
}
```
返回对象:
```
{
    "id": UserID,
    "role": {
        "id": RoleID
    },
    "accountName": "13345670894",
    "avatar": null,
    "nickName": null,
    "balance": 999998,
    "savingBalance": 0,
    "email": null,
    "phone": null,
    "promotionCode": "7AMAAAAAAAA",
    "status": 1,
    "hasSubAccounts": false,
    "createTimestamp": 0,
    "passwordLastSet": 0,
    "createTime": "1970-01-01T00:00:00",
    "passwordLastSetTime": "1970-01-01T00:00:00"
}
```
可能的错误
* > 400 错误代码 invalid_data 取款密码不对或者金额无效

### 保险柜转出
```
POST /api/User/<id>/TransferFromSaving

x-rainier-ticket: <用户令牌>
Content-Type: application/json
{
  "amount": 金额,
  "withdrawPassword": "取款密码"
}
```
返回对象:
```
{
    "id": UserID,
    "role": {
        "id": RoleID
    },
    "accountName": "13345670894",
    "avatar": null,
    "nickName": null,
    "balance": 999998,
    "savingBalance": 0,
    "email": null,
    "phone": null,
    "promotionCode": "7AMAAAAAAAA",
    "status": 1,
    "hasSubAccounts": false,
    "createTimestamp": 0,
    "passwordLastSet": 0,
    "createTime": "1970-01-01T00:00:00",
    "passwordLastSetTime": "1970-01-01T00:00:00"
}
```
可能的错误
* > 400 错误代码 invalid_data 取款密码不对或者金额无效

### 赠送金币
```
POST /api/User/<用户ID>/TransferTo

x-rainier-ticket:<用户令牌>
Content-Type: application/json

{
    "accountName": "13345670892",
    "amount": 1000
}
```
返回对象
```
{
  id: 用户ID
  balance: 用户新余额
}
```
可能的错误
* > 400 错误代码 invalid_data 金额不得小于等于0; object_not_found 目标用户不存在
* > 404 用户不存在

### 赠送金币记录
```
GET /api/User/<用户ID>/TransferLogs

x-rainier-ticket:<用户令牌>
```
返回对象
```
[
  {
    "id": 20,
    "amount": -1000.0,
    "balanceBefore": 10000.0, //开始余额
    "balanceAfter": 9000.0, //结束余额
    "timestamp": 1551680937135, //赠送时间
    "destination": "13345670892" //目标用户
  },
  {
    "id": 22,
    "amount": -1000.0,
    "balanceBefore": 9000.0,
    "balanceAfter": 8000.0,
    "timestamp": 1551681316293,
    "destination": "13345670892",
  }
]
```

### 获取银行卡信息
```
GET /api/User/<id>/BankAccount

x-rainier-ticket: <用户令牌>
```
返回对象
```
{
    "userId": 1002,
    "accountName": "张三",
    "bankName": "农行",
    "accountNumber": "6228480000000000001",
    "province": "上海",
    "city": "黄埔区"
}
```
* > 如果用户没有银行卡信息则返回 HTTP 404

### 更新银行卡信息
```
POST /api/User/<id>/BankAccount

x-rainier-ticket: <用户令牌>
Content-Type: application/json
{
    "accountName": "张三",
    "bankName": "农行",
    "accountNumber": "6228480000000000001",
    "province": "上海",
    "city": "黄埔区"
}
```
返回对象
```
{
    "userId": 1002,
    "accountName": "张三",
    "bankName": "农行",
    "accountNumber": "6228480000000000001",
    "province": "上海",
    "city": "黄埔区"
}
```

### 获取直属会员
```
GET /api/User/<id>/Members?page=[第几页]&pageToken=[上一次请求拿到的pageToken]

x-rainier-ticket: <用户令牌>
```
返回对象
```
{
    "totalPages": 1,
    "page": 0,
    "records": [
        {
            "id": 1003,
            "tenantId": 0,
            "accountName": "13345670892",
            "balance": 0,
            "savingBalance": 0,
            "promotionCode": "6wMAAAAAAAA",
            "gameId": 0,
            "status": 0,
            "hasSubAccounts": false,
            "createTimestamp": 0,
            "passwordLastSet": 0,
            "lastLoginTimestamp": 0,
            "createTime": "1970-01-01T00:00:00Z",
            "passwordLastSetTime": "1970-01-01T00:00:00Z",
            "lastLoginTime": "1970-01-01T00:00:00Z",
            "userName": "1003",
            "name": "13345670892",
            "roles": []
        }
    ],
    "pageToken": "0,1003"
}
```

### 获取用户充值记录
```
GET /api/User/<id>/Payments

x-rainier-ticket: <用户令牌>
```
返回对象
```
[
  {
      "id": 10021,
      "userId": 1032,
      "tenantId": 0,
      "type": 1,
      "channelId": 1,
      "amount": 500.0,
      "status": 3,
      "createTime": 1551577119355,
      "lastUpdatedTime": 1551577187000,
      "orderId": "0E789D525B56A69040636297398095B5"
  },
  {
      "id": 10022,
      "userId": 1032,
      "tenantId": 0,
      "type": 1,
      "channelId": 1,
      "amount": 500.0,
      "status": 3,
      "createTime": 1551577119355,
      "lastUpdatedTime": 1551577187000,
      "orderId": "0E789D525B56A69040636297398095B5"
  },
  ...
]
```

### 获取用户兑奖记录
```
GET /api/User/<id>/Payments?type=2

x-rainier-ticket: <用户令牌>
```
返回对象
```
[
  {
      "id": 10021,
      "userId": 1032,
      "tenantId": 0,
      "type": 1,
      "channelId": 1,
      "amount": 500.0,
      "status": 3,
      "createTime": 1551577119355,
      "lastUpdatedTime": 1551577187000,
      "orderId": "0E789D525B56A69040636297398095B5"
  },
  {
      "id": 10022,
      "userId": 1032,
      "tenantId": 0,
      "type": 1,
      "channelId": 1,
      "amount": 500.0,
      "status": 3,
      "createTime": 1551577119355,
      "lastUpdatedTime": 1551577187000,
      "orderId": "0E789D525B56A69040636297398095B5"
  },
  ...
]
```

### 获取用户登录记录
```
GET /api/User/<id>/LoginLogs

x-rainier-ticket: <用户令牌>
```
返回对象
```
[
  {
    "id": 435,
    "user": {
      "id": 1032
    },
    "browser": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) LayaAirIDE/1.12.2 Chrome/56.0.2924.87 Electron/1.6.6 Safari/537.36",
    "ip": "123.xxx.xxx.xxx",
    "timestamp": 1551521567892,
    "time": "2019-03-02T10:12:47.892Z"
  },
  {
    "id": 434,
    "user": {
      "id": 1032
    },
    "browser": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) LayaAirIDE/1.12.2 Chrome/56.0.2924.87 Electron/1.6.6 Safari/537.36",
    "ip": "123.xxx.xxx.xxx",
    "timestamp": 1551519618582,
    "time": "2019-03-02T09:40:18.582Z"
  },
  ...
]
```

### 获取直属代理
```
GET /api/User/<id>/Agents?page=[第几页]&pageToken=[上一次请求拿到的pageToken]

x-rainier-ticket: <用户令牌>
```
返回对象
```
{
    "totalPages": 1,
    "page": 0,
    "records": [
        {
            "id": 1003,
            "tenantId": 0,
            "accountName": "13345670892",
            "balance": 0,
            "savingBalance": 0,
            "promotionCode": "6wMAAAAAAAA",
            "gameId": 0,
            "status": 0,
            "hasSubAccounts": false,
            "createTimestamp": 0,
            "passwordLastSet": 0,
            "lastLoginTimestamp": 0,
            "createTime": "1970-01-01T00:00:00Z",
            "passwordLastSetTime": "1970-01-01T00:00:00Z",
            "lastLoginTime": "1970-01-01T00:00:00Z",
            "userName": "1003",
            "name": "13345670892",
            "roles": []
        }
    ],
    "pageToken": "0,1003"
}
```

### 获取团队信息
```
GET /api/User/<id>/Team

x-rainier-ticket: <用户令牌>
```
返回对象
```
{
    "total": 1,
    "directMembers": {
        "total": 1,
        "thisWeekIncrement": 1,
        "thisMonthIncrement": 1
    },
    "directAgents": {
        "total": 0,
        "thisWeekIncrement": 0,
        "thisMonthIncrement": 0
    }
}
```

### 获取业绩信息
```
GET /api/User/<id>/Revenue

x-rainier-ticket: <用户令牌>
```
返回对象
```
{
    "totalRevenue": 0,
    "memberRevenue": 0,
    "agentRevenue": 0,
    "myCommission": 0,
    "totalComission": 0,
    "agencyCommission": 0
}
```

### 获取可提现佣金
```
GET /api/User/<id>/CommissionBalance

x-rainier-ticket: <用户令牌>
```
返回对象
```
{
    "userId": <userId>,
    "commission": 佣金
}
```
* > 如果返回 HTTP 202则说明上一周的佣金正在结算当中，应该给个按钮让用户过10秒钟后继续调用该API获取结算出来的数据

### 获取佣金提现记录
```
GET /api/User/<id>/CommissionCashRecords

x-rainier-ticket: <用户令牌>
```
返回对象
```
[
  {
    "userId": <userId>,
    "week": 20190128,
    "commission": 1000.00,
    "cashed": true, (已经提现)
    "cash_time": 提现时间1970/1/1至今的毫秒数
  }
]
```

### 佣金提现
```
POST /api/User/<id>/CashCommission

x-rainier-ticket: <用户令牌>
```
返回对象
{
  "available": 用户新的可用余额
}
可能的错误
* > 404 用户不存在
* > 400 错误代码: invalid_data 没有可提现的佣金

## 用户找回密码

### 开始找回密码流程
```
POST /api/SecuritySession

Content-Type: application/json
{
  "accountName": "用户手机号码",
  "captcha": "验证码",
  "captchaToken": "验证码Token(可选)"
}
```
返回对象
{
  "id": 流程ID,
  "securityQuestions": [
    {
      "id": id1,
      "question": "密保问题1"
    },
    {
      "id": id2,
      "question": "密保问题2"
    },
    {
      "id": id3,
      "question": "密保问题3"
    }
  ]
}
可能返回的错误
* > 400 invalid_data 用户名找不到
* > 400 not_supported 没有设置密保，请联系客服

## 提交密保问题答案
> 当前面一步成功(HTTP 200)以后才可以进行该步操作
```
POST /api/SecuritySession/<流程id>/Resolve

Content-Type: application/json
[
  {
    "question": {
      "id": 密保问题ID1
    },
    "answer": "密保问题答案1"
  },
  {
    "question": {
      "id": 密保问题ID2
    },
    "answer": "密保问题答案2"
  },
  {
    "question": {
      "id": 密保问题ID3
    },
    "answer": "密保问题答案3"
  }
]
```
返回对象
```
{
  {
    "id": userId
  }
}
```
可能返回的错误
* > 400 invalid_data 数据无效
* > 404 object_not_found 无效的流程ID
* > 400 challenge_failed 有密保问题答案错了

### 修改密码
> 当前面一步成功(HTTP 200)以后才可以进行该步操作
```
POST /api/SecuritySession/<流程id>/Complete

Content-Type: application/json
{
  "newPassword": "新密码"
}
```
返回对象
```
{
  {
    "id": userId
  }
}
```
可能返回的错误
* > 400 invalid_data 数据无效
* > 404 object_not_found 无效的流程ID

## 支付

### 获取支付通道类型
```
GET /api/Payment/ChannelTypes

x-rainier-ticket: <用户令牌>
```
返回对象
```
[
    {
        "id": 1,
        "name": "支付宝支付",
        "icon": "upload/alipay.png"
    },
    {
        "id": 2,
        "name": "微信支付",
        "icon": "upload/wechat.png"
    }
]
```

### 获取支付通道
```
GET /api/Payment/ChannelTypes/<id>/Channels

x-rainier-ticket: <用户令牌>
```
返回对象
```
[
    {
        "id": 1,
        "name": "支付宝扫码",
        "value": "ALIPAY"
    },
    {
        "id": 2,
        "name": "支付宝H5",
        "value": "ALIH5"
    },
    {
        "id": 4,
        "name": "支付宝扫码",
        "value": "ALIPAY"
    }
]
```

### 提交支付请求
```
POST /api/Payment

x-rainier-ticket: <用户令牌>
Content-Type: application/json
{
  "amount": 金额,
  "channelId": 支付通道ID
}
```
返回对象
{
  redirectUrl:"跳转地址,需要访问该地址来完成支付"
}

## 兑奖

### 提交兑奖请求
```
POST /api/Payment/Payout

x-rainier-ticket: <用户令牌>
Content-Type: application/json
{
  "amount": 金额
}
```
返回对象
```
{
  id: <订单ID>
}
```

可能返回的错误
* > 400 invalid_data 金额超出用户余额
* > 400 bank_account_required 需要先填写银行卡信息
* > 400 user_in_game 用户还在游戏当中，请断开游戏连接

### 获取兑奖订单列表
```
GET /api/User/<userId>/Payments?type=2

x-rainier-ticket: <用户令牌>
```
返回对象
```
[
  {
    id: 编号,
    amount: 金额,
    createTime: 提交时间,
    status: 当前状态, 2 - 等待, 3 - 已批准, 4 - 已拒绝
    lastUpdatedTime: 最后更新时间
  }
]
```
请注意：所有的时间都是epoch时间，1970/1/1至今的毫秒数


### 获取系统公告信息
```
GET /api/Notice

x-rainier-ticket: <用户令牌>
```
返回对象
```
{
  "id": 编号,
  "title": 标题,
  "content": 内容
}
```

## 游戏通道规范

### Web Socket地址
```
/ws/game?_ticket=<用户令牌>
```

### 约定
* 参考brnn.proto
