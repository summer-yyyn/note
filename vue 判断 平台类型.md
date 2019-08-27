
```
const ua = window.navigator.userAgent
const isIos = !!ua.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/)
const isAndroid = ua.indexOf('Android') > -1 || ua.indexOf('Adr') > -1
const isWx = ua.indexOf('MicroMessenger') !== -1
const isAp = ua.indexOf('AlipayClient') !== -1

const authorizePlatform = (() => {
  const platForm = [{
    ua: 'MicroMessenger', // ua标识
    key: 'wechat_token', // cookie中键名
    exclusiveKey: 'code', // 可能影响授权的参数
    url: '/wechat/oauth2tokenview/modourechargethirdpart?link_param=' // modourechargethirdpart
  }, {
    ua: 'AlipayClient',
    key: 'alipay_token',
    exclusiveKey: 'auth_code',
    url: '/alipay/oauth2view/modourechargethirdpart?link_param='
  }]

  for (let i = 0, l = platForm.length; i < l; i++) {
    const item = platForm[i]
    if (ua.indexOf(item.ua) >= 0) {
      return {
        ...item,
        index: i
      }
    }
  }
  return false
})()

export default {
  v: 'v1.0.0',
  isIos,
  isAndroid,
  isWx,
  isAp,
  code: '0000000',

  terraceType: (authorizePlatform && authorizePlatform.index + 1) || 0,

  exchangeCoupon: '/api/order/getExchangeCoupons',
  couponList: '/api/order/unifyOrderShow',
  authorUrl: '/api/user/getAuthLink',
  codeToken: '/api/user/getUserInfo'
}

```
