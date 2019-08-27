
```
import Vue from 'vue'
import axios from 'axios'
import router from '@/router'
import codes from '@/config/codes'
import VueAxios from 'vue-axios'
import { Indicator, Toast } from 'mint-ui'
import api from '@/api'

function pathRewrite(params) {
  console.log('???', process.env.NODE_ENV === 'production' ? params.prefix + params.v : params.proxy + params.prefix + params.v);
  return process.env.NODE_ENV === 'production'
    ? '.....' + params.prefix + params.v
    : '.........' + params.prefix + params.v
}

Vue.use(VueAxios, axios)

// baseURL 将被放在 url 前面，除非 url 是绝对的
// 为 axios的实例设置 baseURL 来传递相对 url 是很方便的
// 若不是生产环境设置代理前缀( /server 对应 proxyTable 中的键名)
// 接口前缀 prefix 与 版本号 v 不是必须的，你可以将 server 对象里面的对应的键值置为空

// 多个不同的 request 请求配置
const server = {
  proxy: '/api',
  prefix: '',
  v: '',
}
const server2 = {
  proxy: '/ipa',
  prefix: '',
  v: '',
}
// const server3 = {
//   proxy: '/server3',
//   prefix: '/api3',
//   v: '/v1',
// }

const code = api.code
axios.defaults.baseURL = pathRewrite(server)
axios.defaults.headers['Content-Type'] = 'application/json'; 
axios.defaults.timeout = 50000

axios.interceptors.request.use(
  config => {
    // const token = window.localStorage.getItem('token');
    // 请求可以通过config { hide: true } = params 不打开 Indicator
    if (config.params && !config.params.hide) {
      Indicator.open()
    }
    return config
  }, function (error) {
    Indicator.close()
    Toast('请求错误')
    return Promise.reject(error)
  }
)

axios.interceptors.response.use(
  res => {
    Indicator.close();
    const  resCode  = res.data.code;
    // 3007发送验证码失败 3004发送验证码失败 0000014卡卷锁定中
    if ((resCode === code) || (resCode === '3007')|| (resCode === '3004') || (resCode === '0000014') ) {
      return res;
    } else if(resCode === '4000'){
      // token失效 重新走授权
      Toast(res.data.message);
      localStorage.clear();
      const path = router.history.current.fullPath || '/';
      window.localStorage.setItem("beforeLoginUrl", path); // 保存用户进入的url
      router.push('/author');
    } else {
      const path = codes.get(resCode);
      if (path) {
        router.push(path);
      } else {
        Toast(res.data.message);
        router.push('/error/system');
      }
    }
  },function (error) {
    Indicator.close();
    Toast('响应错误');
    return Promise.reject(error);
  }
)

// 以下暴露多个不同 request 服务，建议项目通过 vuex 管理数据派发事件时选择以下方法
// 反之推荐使用直接注入到 vue 实例上的 this.axios 方法

function createRequest(data, url, option) {
  return axios({
    // method: option.method || 'GET',
    baseURL: pathRewrite(data),
    url,
    ...option,
  })
}

// axios({
//   method: 'post',
//   url: '/user/12345',
//   data: {
//     firstName: 'Fred',
//     lastName: 'Flintstone'
//   }
// });
export function requestMock(url, option) {
  return axios({
    url,
    ...option,
  })
}

export function request(url, option) {
  return createRequest(server, url, option)
}

export function request2(url, option) {
  return createRequest(server2, url, option)
}

export function request3(url, option) {
  return createRequest(server3, url, option)
}

/*
 * 你还可以通过 axios.create 来创建自定义实例，使不同实例拥有不同的配置
 * const instance = axios.create({
 *    baseURL: '',
 * });
 * instance.defaults.headers.common['Authorization'] = AUTH_TOKEN;
*/

```
