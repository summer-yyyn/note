
```
<template>
    <div class='report-item'>
        <!-- <h1>授权中。。。?</h1> -->
    </div>
</template>

<script type='text/ecmascript-6'>
    import {query} from '../../utils/util'
    import api from '@/api'
    import { mapGetters, mapActions, mapState, mapMutations } from 'vuex'

    export default {
      data() {
         return {
            token: '',
         };
      },
    computed:  {
        ...mapState({
          terraceType: state => state.terraceType
        }),
        ...mapGetters([
            'authorUrl'
        ])
      },
    created() {
        this.token =  window.localStorage.getItem('user_token');
        const { terraceType } = this;
        this._getAuthorUrl(terraceType);
    },
    mounted(){
    },
    methods: {
      ...mapActions([
        'getAuthorUrl',
        'getToken'
      ]),
      ...mapMutations({
        setChoosedCoupon: 'CHOOSEED_COUPON',
        setToken: 'setToken'
      }),
      async _getAuthorUrl(type) {
        if (query('code') || query('auth_code')) {
          window.localStorage.removeItem('token');
          const  { token }  = await this.getToken({
            params: {
              auth_code: query('code') || query('auth_code'),
              device_id: query('state').split('|')[0] === 'deviceId' ? query('state').split('|')[1]: '',
              table_id: query('state').split('|')[0] === 'tableId' ? query('state').split('|')[1]: '',
              terrace_type: type
            },
            method: 'POST'
          }).catch(err => {
              return false;
          })
          // this.$store.commit('setToken',token);
          this.setToken(token)
          let url =  window.localStorage.getItem("beforeLoginUrl");
          this.$router.push(url);
          return false;
        } else {
          this.getAuthorUrl({
            params: {
              device_id: query('machineId'),
              table_id: query('tableId'),
              terrace_type: type
            },
            method: 'POST'
          })
        }
      },
      async ReturnGetCodeUrl() {
        let {
            data
        } = await getWxAuth({});
        if (data.status == 200) {
            window.location.href = data.url;
        }
      },

    },
    watch: {},

    components: {},

    mounted: function () {}
  };
</script>

<style>

</style>

```


# router部分

```
router.beforeEach((to, from, next) => {
    // window.localStorage.setItem('token', 'a39d26635049449c8cd6678e1186f835')
    // 第一次进入项目
    if (!(window.localStorage.getItem('token')) && to.path != '/author') {
      window.localStorage.setItem("beforeLoginUrl", to.fullPath); // 保存用户进入的url
      next('/author');
      return false;
    }
    next();
})
```
