
```
/* global AlipayJSBridge */
export function wechatConfiguration (shareInfo, wx) {
    return new Promise((resolve) => {
      wx.ready(function () {
        wx.hideAllNonBaseMenuItem()
        wx.showMenuItems({
          menuList: [
            'menuItem:share:appMessage',
            'menuItem:share:timeline'
          ]
        })
        // 分享朋友圈
        wx.onMenuShareTimeline({
          title: shareInfo.circle_content, // 分享标题
          link: shareInfo.circle_clickurl, // 分享链接，该链接域名必须与当前企业的可信域名一致
          imgUrl: shareInfo.circle_img, // 分享图标
          success: function () {
            // 用户确认分享后执行的回调函数
          },
          cancel: function () {
            // 用户取消分享后执行的回调函数
          }
        })
        // 分享给朋友
        wx.onMenuShareAppMessage({
          title: shareInfo.title, // 分享标题
          desc: shareInfo.content, // 分享描述
          link: shareInfo.clickurl, // 分享链接，该链接域名必须与当前企业的可信域名一致
          imgUrl: shareInfo.img, // 分享图标
          type: 'link', // 分享类型,music、video或link，不填默认为link
          success: function () {
            // 用户确认分享后执行的回调函数
          },
          cancel: function () {
            // 用户取消分享后执行的回调函数
          }
        })
        resolve()
      })
    })
  }
  
  export function ready (callback) {
    // 如果jsbridge已经注入则直接调用
    if (window.AlipayJSBridge) {
      callback && callback()
    } else {
      // 如果没有注入则监听注入的事件
      document.addEventListener('AlipayJSBridgeReady', callback, false)
    }
  }
  
  export function alipayConfiguration (shareInfo, ap) {
    ready(() => {
      // AlipayJSBridge.call('setOptionMenu', {
      //   title: '分享',
      //   redDot: '-1'
      // })
      // AlipayJSBridge.call('setTitle', {
      //   title: '分享'
      // })
      AlipayJSBridge.call('hideOptionMenu')
    }, false)
  }
  
```
