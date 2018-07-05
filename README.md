# 万趣SDK接入文档

---
#SDK导入
* 导入tofu.framework和tofuBundle.bundle

#工程配置
* bundle id 格式 com.wanquyouxi.g1(末尾的1为game_id)
* info.plist 配置 GAME_ID 和 CHANNEL_ID(如不知请询问对接人员）
* build Setting ->other linker flags 加 -ObjC 和-all_load

* 添加白名单，下面提供白名单代码，source code 打开info.plist 粘贴就可以

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
<string>alipay</string>
<string>wechat</string>
<string>weixin</string>
<string>sinaweibohd</string>
<string>sinaweibo</string>
<string>sinaweibosso</string>
<string>weibosdk</string>
<string>weibosdk2.5</string>
<string>mqqapi</string>
<string>mqq</string>
<string>mqqOpensdkSSoLogin</string>
<string>mqqconnect</string>
<string>mqqopensdkdataline</string>
<string>mqqopensdkgrouptribeshare</string>
<string>mqqopensdkfriend</string>
<string>mqqopensdkapi</string>
<string>mqqopensdkapiV2</string>
<string>mqqopensdkapiV3</string>
<string>mqzoneopensdk</string>
<string>wtloginmqq</string>
<string>wtloginmqq2</string>
<string>mqqwpa</string>
<string>mqzone</string>
<string>mqzonev2</string>
<string>mqzoneshare</string>
<string>wtloginqzone</string>
<string>mqzonewx</string>
<string>mqzoneopensdkapiV2</string>
<string>mqzoneopensdkapi19</string>
<string>mqzoneopensdkapi</string>
<string>mqzoneopensdk</string>
</array>
```
![白名单.png](http://upload-images.jianshu.io/upload_images/1899979-4fe604e8958cddfa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 接口调用 
* 登录接口 导入#import "tofu.framework/Headers/TofuGame.h"

```objc
//登录接口调用
[[TofuGame defaultInstance] login:^(NSString *userName) {
//登录成功 userName为用户唯一标识
} failure:^(NSString *info) {
//登录失败
}];
```

* 退出登录
```objc
//设置退出登录回调
[TofuGame defaultInstance].logoutBlock = ^{
//退出登录回调
};
//调用退出登录
[[TofuGame defaultInstance] logout];
```

* 支付接口

```objc
TGOrderInfo *order = [[TGOrderInfo alloc] init];
order.goodsID = @"com.bobo.g6.6";
order.productName = @"元宝";
order.productDesc = @"元宝";

order.cpOrderID = [NSString stringWithFormat:@"%.0f", [NSDate date].timeIntervalSince1970];
order.callbackUrl = @"wanqu.com";
order.extrasParams = @"asdf";
order.amount = 6;


TGRoleInfo *role = [[TGRoleInfo alloc] init];
role.serverName = @"大陆一区";
role.serverId = @"2001";
role.gameRoleName = @"你好啊";
role.gameRoleID = @"1002";
role.gameUserLevel = @"11";

[[TofuGame defaultInstance] payOrderInfo:order roleInfo:role success:^{

} failure:^(NSInteger code) {

}];
```

* 角色等级上传

```objc
TGRoleInfo *role = [[TGRoleInfo alloc] init];
role.serverName = @"大陆一区";
role.serverId = @"2001";
role.gameRoleName = @"你好啊";
role.gameRoleID = @"1002";
role.gameUserLevel = @"11";
[[TofuGame defaultInstance] updateRoleInfoWith:role isCreate:NO];
```


# 服务端充值结果异步通知

请在收到充值回调结果后，获取相应的充值提示语，返回给充值玩家。
使用get请求通知游戏商玩家的充值结果


参数名    类型    说明
game_order_sn    string    游戏订单号
money    int    充值金额(单位：分)
status    int    充值状态：0 充值失败；1：充值成功
msg    string    充值备注
createtime    int    充值时间
ext    string    透传字段
username    string    充值用户名或用户id
order_sn    string    平台订单号
sign    string    签名字符串 md5(game_order_sn+money+status+createtime+key) 注：key由我方提供，md5之后字符串转小写。+号为连接符号，不参与签名

* 成功输出小写字符串 ： success 失败输出小写字符串 fail

# 登录验证


[游戏登录服务端]携带相关的参数请求此接口； [万趣SDK服务器]接收并判断用户是否是真实登录，并返回相应的信息。


* 请求地址举例（get请求）

http://jk.wanquyx.com/api.php?m=public&a=checklogin&username=abc123&sign=abc123wews123 &time=142345612&game_id=2


* 请求参数

|参数名        |  类型    |说明             |
|:----------- |:-------- |:---------------:|
|  username   |  string  |  用户的唯一标识（用户登录成功，sdk会返回给游戏方这个参数）    |    
|  sign       |  string  |  md5(username + time + KEY)这里传递的md5字串为小写字母, KEY是表示平台和游戏方双方提前协商约定好的登录密钥;加号不参与签名   |  
|  time       |  int     |  Unix时间戳 (当前请求时间)  |
|  game_id       |  int     |  游戏id(由万趣提供) | 

### 返回内容（json）

```
{
“err”: int,            // 错误码：0-登录成功, 其他失败

“errMsg”: string,           // 错误信息描述
}
```

