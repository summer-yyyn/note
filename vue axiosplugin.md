
```
import axios from 'axios';
const AxiosInstance = axios.create({
    // baseURL:'http://yanfa5.51shaoxi.cn/'
    baseURL:'http://smqpy-api.51shaoxi.com/'
});
AxiosInstance.defaults.headers.common['Access-Control-Allow-Origin'] = '*';
AxiosInstance.defaults.headers.common['Access-Control-Allow-Headers'] = 'Origin, X-Requested-With, Content-Type, Accept';

AxiosInstance.interceptors.request.use(
    (config) => {
        return config
    },
    error => Promise.reject(error)
)

AxiosInstance.interceptors.response.use(
    (response) => {
        const { data } = response;
        return data;
    },
    error => Promise.reject(error)
)

const AxiosPlugin = {
    install(_vue){
        _vue.mixin({
            created () {
                const self = this;
                self.request = AxiosInstance;
            }
        })
    }
}
export default AxiosPlugin;
```
