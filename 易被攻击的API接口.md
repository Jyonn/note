# 易被攻击的API接口
#### 工刀 最后一次更新 Jan. 13, 2020

### 验证码发送
> POST `https://wx.51bushou.com/ygg-hqbs/user/sendBoundCode`

#### 举例
```
{
    "mobileNumber": "13567089898",
    "addressCode": "86",
    "channelType": "1"
}
```

#### 参数
|参数         |描述|
|:----------:|:---------:
| mobileNumber | 手机号
| addressCode | 国家／地区
| channelType | 未知


### 有师英语
- 讨论区的WSS协议没有加密措施，可伪装任何人