# 分析分布式项目的登陆跟菜单设计
### 分布式组件的登陆一般操作是：
* 1.获取后台验证码，返回验证码图片到前台，并将验证码的值放入到session中,并将session放入到redis里面,（以session_id为key）.
* 2.前后登陆后，密码是以md5形式存储，直接比较md5,通过验证码和密码校验后，将当前用户信息存储到session里面，再把session重新存储到redis里面，同时加载菜单权限到用户信息里面，菜单权限也就是一些url路径，拿到权限后设置到用户信息里面，再次将session设置入redis里面（菜单缓存这里可以用异步操作）。
* 3.用户访问页面时，首先会经过登陆拦截器，拦截器会读取redis的用户信息，看用户是否存在，存在再允许用户进入。
* 4.此时再校验用户权限，从redis中读取菜单权限缓存和页面的url进行比较，如果权限列表包含此url，则成功进入页面。

### 下面是登陆的时序图
![](https://github.com/tsmairc/Login-MenuDesign/blob/master/image/seq.png?raw=true)

### 下面介绍一些重点的代码
#### 验证码
验证码这里采用了![patchca](https://github.com/pusuo/patchca)插件，详细代码如下：
```java
//获取验证码图片时后台操作，这里的getSession实际上都是根据前台传来的session_id去redis找出缓存的session对象
Map<String, Object> session = getSession();
//token为实际值
String token = EncoderHelper.getChallangeAndWriteImage(cs, "png", response.getOutputStream());
session.put(Consts.VERIFICATION_CODE_OPERATOR, token);

//登陆时操作
String _verification_code = getSession().get(Consts.VERIFICATION_CODE_OPERATOR);

```
