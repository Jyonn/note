# 易被攻击的API接口
#### 工刀 最后一次更新 Nov. 20, 2017

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
