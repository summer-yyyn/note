### 目前使用Taro做了两个微信小程序项目，分享一下遇到的问题
#### 1、Taro.reLaunch兼容性问题
##### A页面relaunch到B页面，再从B页面relaunch到C页面，iOS系统支持，安卓系统不支持。
==解决方法==
##### 从A页面relaunch到B页面，再从B页面redirectTo到C页面，可实现同样功能。
> ##### Taro.reLaunch同wx.reLaunch关闭所有页面，打开到应用内的某个页面。
> ##### Taro.redirectTo同wx.redirectTo关闭当前页面，打开应用内某页面。
#### 2、时间转换问题
##### 拿到的时间数据格式'2019-6-14 11:11:11'，进行时间转换，获取毫秒数

```
new Date('2019-6-14 11:11:11').getTime()
```
##### 在安卓和pc端转换正常，非常完美。但是在iOS上面返回 invalid Date
##### 测试后发现

```
new Date('2019-6-14 11:11:11')
```
在iOS上面返回  invalid Date

==解决方法==
##### 将时间字符串中的'-'改为'/'即可
```
new Date('2019-6-14 11:11:11'.replace(/-/g, "/"))
```
> iOS端支持转换的时间类型有

```
new Date(2019/6/14)
new Date("2019","6","14","11","11")
new Date(148888888888888) // 注意毫秒应该是数字而非字符串
```
#### 3事件绑定传参规则

##### 事件绑定传参一定要用bind方式
```
onClick={this.handleClick.bind(this, val)}
```
#### 4、组件间传值
##### 方法前要加on子组件蔡调的通

```
···
<List
    num={this.state.num} // 传数据
    onSumit={this.handleSubmit} // 方法
></List>
···
```
#### 5、条件渲染
##### Taro不支持在render函数以外写jsx，所以不能用函数返回的方式实现
##### jsx不支持if运算，可使用三元运算符或者表达式实现

```
{this.state.type === 1 ? <View className='report'>类型一</View> : <View className='report'>类型二</View>}
{this.state.isReport && <View className='report'>一个报告</View>}
{this.state.isReport && this.state.type === 1 && <View className='report'>一个类型一报告</View>}
{this.state.isReport || this.state.type === 1 && <View className='report'>一个类型一报告</View>}
```
#### 6、路由传参

```
Taro.navigateTo({
  url: `../sport/advice?code=${this.reportCode}`
})
```
#### 7、路由取参

```
this.reportCode = this.$router.params.code
```
#### 8、丢包问题
##### 经常npm run dev:wepp后发生丢包问题
==解决方法==
1. 重新编译项目；
2. 删除dist和node_modules目录后重新npm install 解决；
3. 关闭编辑器重新打开；

#### 9、npm run dev:wepp后有的文件没有被编译
##### 打开没有被编译的文件，保存后自动编译就有了

#### 10、小程序的onShow（）生命周期等于Taro中的componentDidShow（）生命周期

---

### 目前想起来这么多，以后再进行补充  ---    谢谢各位看官～ 啦啦啦
