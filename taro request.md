
```
import Taro from '@tarojs/taro'
import { API_USER, API_LOGIN, API_ADD_ORDER, API_HOME } from '@constants/api'
import {LOGIN_INFO} from '@constants/user'

const CODE_SUCCESS = 200
const CODE_AUTH_EXPIRED = 401
const CODE_AUTH_EXPIREDS = 405
const COOE_NO_PHONE = 550
const CODE_ORDER_NOPHONE = 206

function getStorage(key) {
  return Taro.getStorage({ key }).then(res => res.data).catch(() => '')
}

function updateStorage(data = {}) {
  return Promise.all([
    Taro.setStorage({ key: 'token', data: data['token'] || '' }),
    // Taro.setStorage({ key: 'uid', data: data['uid'] || ''})
  ])
}

/**
 * 简易封装网络请求
 * // NOTE 需要注意 RN 不支持 *StorageSync，此处用 async/await 解决
 * @param {*} options
 */
export default async function fetch(options) {
  const { url, payload, method = 'GET', showToast = true, autoLogin = true, canSendToken = true } = options
  const token = await getStorage('token')
  // const header = token ? { 'auth_token': token, 'X-WX-3RD-Session': token } : {}
  // const canSendToken = (url === API_LOGIN || url === API_HOME) ? false : true
  const header = token && canSendToken ? { 'auth_token': token, 'X-WX-3RD-Session': token } : {}
  if (method === 'POST') {
    header['content-type'] = 'application/json'
  }
  if (url === API_LOGIN) {
    header['content-type'] = 'application/x-www-form-urlencoded'
  }

  const dataPl = canSendToken ? {
    ...payload,
    auth_token: token
  } : {...payload}

  return Taro.request({
    url,
    method,
    data: dataPl,
    header
  }).then(async (res) => {
    const { code, contents } = res.data
    if (code !== CODE_SUCCESS) {
      if (code === CODE_AUTH_EXPIRED) {
        await updateStorage({})
      }
      return Promise.reject(res.data)
    }
    if (url === API_LOGIN) {
      await updateStorage(contents)
    }

    // XXX 用户信息需展示 uid，但是 uid 是登录接口就返回的，暂时糅合在 fetch 中解决
    if (url === API_USER) {
      await updateStorage(contents)
      // const uid = await getStorage('uid')
      // return { ...contents, uid }
    }
    return Promise.resolve(contents)
    // return data
  }).catch((err) => {
    // const defaultMsg = err.code === CODE_AUTH_EXPIRED ? '登录失效' : '请求异常'
    const defaultMsg = err.code === CODE_AUTH_EXPIRED ? '登录中~' : err.msg === '购买数量不能为空' ? '请选择商品属性': err.msg
    if (showToast) {
      Taro.showToast({
        title: err && defaultMsg || err.msg,
        icon: 'none'
      })
    }
    if (err.code === CODE_AUTH_EXPIRED){ // 401认证失败
      Taro.login().then(res => {
        // console.log(78978978)
        // console.log(res)
        if (res.code){
          const header = {}
          header['content-type'] = 'application/x-www-form-urlencoded'
          Taro.request({
            url: API_LOGIN,
            data: {code: res.code},
            method: 'POST',
            header
          }).then(async (res1) => {
            const { code, contents } = res1.data
            if (code === CODE_SUCCESS) {
              await updateStorage(contents)
              wx.startPullDownRefresh()
              Taro.stopPullDownRefresh()
              // Taro.showToast({
              //   title: '请重试',
              //   icon: 'none'
              // })
            } else {
              Taro.navigateTo({
                url: '/pages/authorize/authorize'
              })
            }
          })
        }
      })
    }
    if (err.code === CODE_AUTH_EXPIREDS) { // 405token失效
      Taro.login().then(res => {
        if (res.code){
          const header = {}
          header['content-type'] = 'application/x-www-form-urlencoded'
          Taro.request({
            url: API_LOGIN,
            data: {code: res.code},
            method: 'POST',
            header
          }).then(async (res1) => {
            const { code, contents } = res1.data
            if (code === CODE_SUCCESS) {
              await updateStorage(contents)
              wx.startPullDownRefresh()
              Taro.stopPullDownRefresh()
              // Taro.showToast({
              //   title: '请重试',
              //   icon: 'none'
              // })
            } else {
              Taro.navigateTo({
                url: '/pages/authorize/authorize'
              })
            }
          })
        }
      })
    }
    if (err.code === CODE_ORDER_NOPHONE) {
      Taro.navigateTo({
        url: '/pages/user-login/user-login'
      })
    }
       return Promise.reject({ message: defaultMsg, ...err })
  })
}

```
