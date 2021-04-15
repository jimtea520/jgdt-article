随着微信小程序的发展，越来越多的移动端应用选择了微信产品作为媒介。无论是公众号开发还是小程序开发，微信支付永远都是绕不开的话题。由于微信支付涉及了很多场景，本文我们只学习如何在公众号、小程序中接入微信支付。



## 一、微信支付的前提条件

### 1.1 公众号

微信公众号大体上可以分为服务号和订阅号，订阅号和服务号的具体区别在以前写过的一篇文章[Spring Boot开发微信公众号](https://mp.weixin.qq.com/s?__biz=MzU5NjA3MjQ5MA==&mid=2247486150&idx=1&sn=9aa2e8e3313871c27e6d2f3ec8020afe&scene=21#wechat_redirect)中具体介绍了，这里就不再赘述，总的来说就是服务号提供了更高级的功能。

微信支付接入需要**已经完成微信认证的服务号**。如果是小程序的话，也需要完成**微信认证**。

公众号可以关联同一主体的10个小程序，不同主体的3个小程序，如果是和公众号同一主体的小程序并且公众号已经完成认证，则直接可以在公众号后台的`小程序管理`中，进行快速注册并认证，这样就无需重复支付微信认证所需的`300`RMB了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVeaxVzRKkqehALVVQTDIqCyIM6yE1DEDmOcRoBoickcHMsPolTX0viba5JTZZCqibCKhUvdHjNcibuTNQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 1.2 微信商户平台

微信认证完成后，在公众号后台的 `微信支付` 中开通微信支付功能。提交微信支付申请后，3-5个工作日内，会进行审核，审核通过后会往你填写的邮箱里发送一份包含商户号信息的邮件，同时会往你填写的对公账户中打几毛钱的汇款，需要你查看具体金额后在商户平台中验证。

商户分为普通商户和服务商商户，千万不要申请错了。

普通商户是可以进行交易，但是不能拓展商户。

服务商可以拓展商户，但是不能交易。

服务商就是提供统一的支付入口，它需要绑定具体的普通商户，微信支付时会在支付接口中携带普通商户参数，支付成功后金额会直接到具体的普通商户账户上。

![图片](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVeaxVzRKkqehALVVQTDIqCy6fNO53y6ZKlafp1TqxJn3JFrkfWtHriaB5BjKGQjGYEu8SoHo2rx8UA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

申请时直接申请**普通商户**就可以了。

### 1.3 绑定商户

微信支付发起依赖于公众号、小程序等应用与商户号的绑定关系。因此在进行开发前，需要将商户与具体应用进行绑定。

如果商户和需要绑定的AppID是同一主体，只需要以下步骤即可完成绑定。

- 在商户平台-产品中心-AppID账户管理中关联AppID，输入AppId申请绑定
- 在公众号或小程序后台微信支付-商户号管理中进行确认。

![图片](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVeaxVzRKkqehALVVQTDIqCyiaaciaXERkWudOuYESBbOyricYYVicUsYxSTJhRFgicvjWLEDg9voicRc7hQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果商户和需要绑定的AppID是不同主体，步骤和上述一样，除了输入AppId之外，还需要填入AppId的认证信息。

![图片](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVeaxVzRKkqehALVVQTDIqCyWOiaRnQibOhLehGP6fcqUI3Xd4bMibCIdJqXAdEibdjevG7yVchD9RFNxA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 二、微信支付相关配置

### 2.1 支付产品类型

**1. 付款码支付**

用户打开微信钱包-付款码的界面，商户扫码后提交完成支付。

**2. JSAPI支付**

用户通过微信扫码，关注公众号等方式进入商家H5页面，并在微信内调用JSSDK完成支付。

**3. Native支付**

用户打开微信扫一扫，扫描商户的二维码后完成支付。

**4. APP支付**

商户APP中集成微信SDK，用户点击后跳转到微信内完成支付。

**5. H5支付**

用户在微信以外的手机浏览器请求微信支付的场景唤起微信支付。

**6. 小程序支付**

用户在微信小程序中使用微信支付的场景。

**7. 刷脸支付**

无需掏出手机，刷脸完成支付，适合线下各种场景。

在商户平台-产品中心-我的产品中申请开通支付产品。

![图片](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVeaxVzRKkqehALVVQTDIqCy7v9ic7icUxricZPaAM7bWDz0BQpFUHgCvjdd5pibwvQUaLLyadBv9zD17w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2.2 支付授权目录配置

在商户平台-产品中心-开发配置中进行支付授权目录的配置（即你开发的下单接口地址），需要注意的是授权目录最多可以配置**五个**，在开发过程中请合理定义支付接口。

![图片](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVeaxVzRKkqehALVVQTDIqCynvan3qfE6y4iaFGxsZPeRIj8Azx8y9V4maWg2Ka53KzEwl7ls9iaKNjA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2.3 配置商户密钥

在商户平台-账户中心-API安全中设置API密钥。

![图片](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVeaxVzRKkqehALVVQTDIqCyXSshdI35qZ1pic99iacwSONzJvxR8RktGCF4rzX8yqicaa2c4aReTwx7Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

第一次设置时，需要安装操作证书，傻瓜式安装，按照提示一步一步操作就可以。

API密钥需要一个**32位**的随机字符串，记得**不要随意更改API密钥**。

![图片](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVeaxVzRKkqehALVVQTDIqCyViatwmoal4t9g5iaNG8FEYdRpL4PrIFjtBY7iaNhARap6Usyp9fxFCZQw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在微信API v3版本中，除了要配置API密钥外，还需要配置APIv3密钥和申请CA颁发的API证书。

- API v3密钥主要用于平台证书解密、回调信息解密。
- API证书用于调用更高级别的api接口，包含退款、红包等接口。

**如果使用开源的微信开发包，请了解是否支持v3版本**。

### 2.4 配置服务器

在公众号后台-开发-基本配置-服务器配置中启用并填写服务器信息。

![图片](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVeaxVzRKkqehALVVQTDIqCyGQNfSFPzY5eSc4fo4j5ibS7n3o4ADxuNTVSDbOKj5AnRqvq3OnVpTPA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2.5 白名单配置

在公众号后台-开发-基本配置-公众号开发信息中配置开发者密钥，同时填写IP白名单。

![图片](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVeaxVzRKkqehALVVQTDIqCywf76GwpJ6KydCvl3PGUYCibp7yZCZm1ACfee5jTaia0lrF0SMcZ6NRrQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2.6 JS接口安全域名

在公众号后台-公众号设置-功能设置中设置JS接口安全域名。

![图片](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVeaxVzRKkqehALVVQTDIqCygEzU7EE3arZFC1g4HPg6xGHONnibDUF1gzCUdM2OkA7XG9h6kk9PUbg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上面的配置是基于公众号支付配置的，小程序支付没有这么麻烦，小程序支付不用配置支付授权目录和授权域名。

|          | JSAPI      | 小程序 |
| :------- | :--------- | :----- |
| 支付协议 | HTTP/HTTPS | HTTPS  |
| 支付目录 | 有         | 无     |
| 授权域名 | 有         | 无     |

## 三、微信支付流程

由于微信升级了API接口，在API v3接口中，需要加载申请的API证书，微信已经封装了相关jar包，并且提供了加载示例，具体可参考“https://pay.weixin.qq.com/wiki/doc/apiv3/open/pay/chapter2_3.shtml”，这里就不再赘述。我们以API v2为例详细学习一下微信接入的主要流程（因为API v3的一些接口还在持续升级，v2接口相对完整）。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/lgiaG5BicLkVeaxVzRKkqehALVVQTDIqCyPwT27BRN7Cl8txYpjIibVdU22LNFgQZ14qkriaLMaAwFGBGoLmkfX1nQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上面的这张图片来自微信开发文档，我们详细分析一下支付流程。

### 3.1 微信下单接口

用户通过微信客户端发起支付，在商戶后台生成订单，然后调用微信下单接口，生成预支付订单，返回订单号！

下单接口涉及到的主要参数，只列举重要的几个参数：

| 请求参数     | 是否必传 | 类型   | 描述                      |
| :----------- | :------- | :----- | :------------------------ |
| appid        | 是       | String | 公众号appid               |
| mch_id       | 是       | String | 商户号                    |
| nonce_str    | 是       | String | 随机字符串，32位以内      |
| sign         | 是       | String | 签名，默认使用MD5进行加密 |
| out_trade_no | 是       | String | 系统内部订单号            |
| total_fee    | 是       | Int    | 订单总金额，单位是分      |
| notify_url   | 是       | String | 支付结果通知接口          |

```
sign`的签名也比较通用，涉及了一个保证签名不可预测的`nonce_str
```

- 将所有发送的非空参数使用字典排序生成键值对（key1=value1&key2=value2）;
- 将商户平台密钥拼接在上述字符串的最后（"String"+&key=密钥）；
- 将上述字符串采用MD5加密

### 3.2 支付

拉起微信支付，输入密码，完成支付。这一步需要在H5网页中执行JS调起支付。

需要以下参数，因此在预付订单返回时，需要将下列参数封装后响应给页面，由页面完成支付。

| 参数名    | 是否必传 | 类型   | 描述                            |
| :-------- | :------- | :----- | :------------------------------ |
| appId     | 是       | String | 公众号id                        |
| timeStamp | 是       | String | 当前时间戳                      |
| nonceStr  | 是       | String | 随机字符串                      |
| package   | 是       | String | 预支付订单，格式为prepay_id=*** |
| signType  | 是       | String | 签名类型，默认MD5               |
| paySign   | 是       | String | 签名                            |

签名和下单接口的签名方式一样。

JS伪代码如下：

```
function onBridgeReady(){
    WeixinJSBridge.invoke(
        'getBrandWCPayRequest', {
            // 公众号ID，由商户传入
            "appId":"wx2421b1c4370ec43b",
            // 时间戳，自1970年以来的秒数
            "timeStamp":"1395712654",  
            // 随机串
            "nonceStr":"e61463f8efa94090b1f366cccfbbb444",
            "package":"prepay_id=u802345jgfjsdfgsdg888", 
            // 微信签名方式
            "signType":"MD5",
            // 微信签名
            "paySign":"70EA570631E4BB79628FBCA90534C63FF7FADD89"
        },
        function(res){
            if(res.err_msg == "get_brand_wcpay_request:ok" ){
                // 使用以上方式判断前端返回,微信团队郑重提示：
                // res.err_msg将在用户支付成功后返回ok，但并不保证它绝对可靠。
            } 
        }); 
}
if (typeof WeixinJSBridge == "undefined"){
    if( document.addEventListener ){
        document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
    }else if (document.attachEvent){
        document.attachEvent('WeixinJSBridgeReady', onBridgeReady); 
        document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
    }
}else{
    onBridgeReady();
}
```

注意伪代码中的这句话`// res.err_msg将在用户支付成功后返回ok，但并不保证它绝对可靠。`为什么这么说呢，我举个例子应该就明白了。假如你去超市买东西，是不是你说支付成功了你就可以把东西带走呢？肯定不是，是当商家收到钱后才算你支付成功，你才可以把东西带走。也就是说，这里提示的成功并不能说一定支付成功了，具体是否成功，微信平台会以异步的方式给你进行通知。

### 3.3 异步通知

异步通知是比较重要的一步，在这里你可以根据通知结果处理你的业务逻辑。但是，可能会由于网络波动等原因通知不到，或者说微信接收到的响应不符合API的规定，微信会持续发起多次通知（请在回调通知接口中合理处理，**避免重复通知造成业务重复处理**），直到成功为止，通知频率为`15s`/`15s`/`30s`/`3m`/`10m`/`20m`/`30m`/`30m`/`30m`/`60m`/`3h`/`3h`/`3h`/`6h`/`6h` - 总计 `24h4m`）。但是微信不保证通知最终一定会成功。

异步通知**响应参数**如下：

| 参数名      | 是否必传 | 类型   | 描述                         |
| :---------- | :------- | :----- | :--------------------------- |
| return_code | 是       | String | 返回状态码，`SUCCESS`/`FAIL` |
| return_msg  | 否       | String | 返回信息                     |

如果微信一直通知不成功怎么？还是刚才那个例子，你明明支付成功了，但是商家却一直说她没收到钱，这时候你怎么办？肯定是去看一下她的手机是否真的没有收到钱！这里也一样。

### 3.4 支付状态查询

![图片](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVeaxVzRKkqehALVVQTDIqCyVV8Lqd7hWJTnoMLkLp7YjyxI0pIicBMmicSQxAfSV5KWYZWA6XbUh3GA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1. 商户APP或者前端页面收到支付返回时，商户需要调用商户查单接口确认订单状态，并把查询结果展示给用户。
2. 商户后台需要准确、高效地处理微信支付发送的异步支付结果通知，并按接口规范把处理结果返回给微信支付。
3. 商户后台未收到异步支付结果通知时，商户应该主动调用 `微信支付查单接口`，同步订单状态。
4. 商户在T+1日从微信支付侧获取T日的交易账单，并与商户系统中的订单核对。如出现订单在微信支付侧成功，但是在商户侧未成功的情况，商户需要给用户补发货或者退款处理。

## 四、总结

本文主要以公众号支付为例，总结了接入微信支付需要的相关配置和支付流程。其他支付像APP支付也是开发中比较常见的应用场景，APP支付需要在 `微信开放平台` 去创建应用来接入微信支付。除此之外，微信支付API在向v3平滑升级，有些接口也还没有升级完成，升级完的接口相较于v2发生了一些数据格式方面的变化。如果引用第三方开发包进行开发，需要注意接口对应的版本。

#### 参考

微信支付开发文档