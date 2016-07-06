# co-wechat-payment
微信支付 for node.js

[![npm version](https://badge.fury.io/js/co-wechat-payment.svg)](http://badge.fury.io/js/co-wechat-payment)

## Installation
```
npm install co-wechat-payment
```

## Usage

创建统一支付订单
```js
var WXPay = require('co-wechat-payment');

var wxpay = WXPay({
	appId: 'xxxxxxxx',
	mchId: '1234567890',
	partnerKey: 'xxxxxxxxxxxxxxxxx', //微信商户平台API密钥
	pfx: fs.readFileSync('./wxpay_cert.p12'), //微信商户平台证书
});

var result = yield wxpay.unifiedOrder({
	body: 'js H5支付',
	out_trade_no: '20160701'+Math.random().toString().substr(2, 10),
	total_fee: 1, // 1分钱
	spbill_create_ip: '10.10.10.10',
	notify_url: 'http://xx.xx.xx/wxpay/notify/',
	trade_type: 'JSAPI',
	product_id: '1234567890'
});
```

查询订单
https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_2
```js
// 通过微信订单号查
var result = yield wxpay.queryOrder({ transaction_id:"xxxxxx" });

// 通过商户订单号查
var result = yield wxpay.queryOrder({ out_trade_no:"xxxxxx" });
```

关闭订单
https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_3
```js
var result = yield wxpay.closeOrder({ out_trade_no:"xxxxxx"});
```
退款接口
```js
var params = {
	appid: 'xxxxxxxx',
	mch_id: '1234567890',
    op_user_id: '商户号即可',
    out_refund_no: '20140703'+Math.random().toString().substr(2, 10),
    total_fee: '1', //原支付金额
    refund_fee: '1', //退款金额
    transaction_id: '微信订单号'
};

var result = yield wxpay.refund(params);
```

### 原生支付 (NATIVE)


### 公众号支付 (JS API)

生成JS API支付参数，发给页面
```js
var result = yield wxpay.getBrandWCPayRequestParams({
	openid: '微信用户 openid',
	body: '公众号支付测试',
    detail: '公众号支付测试',
	out_trade_no: '20150331'+Math.random().toString().substr(2, 10),
	total_fee: 1,
	spbill_create_ip: '192.168.2.210',
	notify_url: 'http://wxpay_notify_url'
});

yield this.render('/wechat/pay',{payArgs:result});
```

网页调用参数（以ejs为例）
```js
WeixinJSBridge.invoke(
	"getBrandWCPayRequest", <%-JSON.stringify(payArgs)%>, function(res){
		if(res.err_msg == "get_brand_wcpay_request:ok" ) {
    		// success
    	}
});
```

根据之前预创建订单接口返回的prepare_id参数，生成JS API支付参数，发给页面
```js
var result = yield wxpay.getJsPayRequestParams('xxxxx');

yield this.render('/wechat/pay',{payArgs:result});
```

### 中间件

商户服务端处理微信的回调（koa为例）
```js
var wechatBodyParser = require('co-wechat-body');
app.use(wechatBodyParser(options));

// 支付结果异步通知
router.post('/wechat/payment/notify', wechatPayment.middleware(), function* (){
  var message = this.request.body;
  // 处理你的订单状态更新逻辑. [warn] 注意，分布式并发情况下需要加锁处理.
  // do something...

  // 向微信返回处理成功信息
  this.body = 'OK';

  // 如果业务逻辑处理异常, 向微信返回错误信息，微信服务器会继续通知.
  // this.body =  new Error('server error');
});
```