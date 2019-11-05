## cookie 是啥？

简单的说，Cookie就是服务器暂存放在你计算机上的一笔资料，**好让服务器用来辨认你的计算机**。当你在浏览网站的时候，Web服务器会先送一小小资料放在你的计算机上，当下次你再光临同一个网站，Web服务器会先看看有没有它上次留下的Cookie资料，有的话，就会依据Cookie里的内容来判断使用者，送出特定的网页内容给你。

> 一个cookie 的表现形式为： name : value 。
>
> cookie 存放在本地计算机上，它的属性包含 所属域名，例如 `baidu.com`。路径，例如 `/`，创建时间，到期时间。

## cookie 实现原理

- 第一次请求浏览器，在浏览器的cookie存储区，没有`cookie`。

- web服务器可以通过`http`响应消息中增加`Set-Cookie`响应头，将`Cookie`信息发送给浏览器。

- 在浏览器再次访问 **该域名**的时候，请求头会带上 Cookie 信息。


## cookie 保存时间

如果不设置cookie的生存时期，默认是浏览器 会话结束（即浏览器关闭）就删除 cookie 。

例如：

```java
 Cookie cookie = new Cookie("name","wyf");
 response.addCookie(cookie);
```

如果设置了过期时间，即将`cookie` 持久化，在浏览器关闭后 cookie 并不删除，只有当到期时间到了之后，才会删除。

```java
Cookie cookie = new Cookie("name","wyf");
cookie.setMaxAge(30);
response.addCookie(cookie);
```

## 什么是Session

Session 的主要作用是`服务端` 保存用户的状态。如果该服务器的开启 `session` 后，第一次连接该服务器时，服务器会在响应头通过`Set-Cookie` 向浏览器设置一个`cookie` ,`name` 为 `JSESSIONID`。

同时在服务器端会生成 `id` 与 `Session` 的对应关系。后端程序通过 `request.getSeesion()`获取其对应的`Session`。

默认情况下，服务端返回浏览器的 `cookie`  没有设置过期时间，即浏览器关闭后，便删除该`cookie` 。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       

## 单系统登陆

用户登录

```java
@PostMapping(value = "/user/session", produces = {"application/json;charset=UTF-8"})
public Result login(String mobileNo, String password, String inputCaptcha, HttpSession session, HttpServletResponse response) {

    //判断验证码是否正确
    if (WebUtils.validateCaptcha(inputCaptcha, "captcha", session)) {

        //判断有没有该用户
        User user = userService.userLogin(mobileNo, password);
        if (user != null) {
            /*设置自动登陆，一个星期.  将token保存在数据库中*/
            String loginToken = WebUtils.md5(new Date().toString() + session.getId());
            //  数据库中 loginToken 对应着 user
            user.setLoginToken(loginToken);
            
            User user1 = userService.userUpload(user);
			// 将用户信息添加到 session 中
            session.setAttribute("user", user1);
			// 设置 cookie 中 loginToken 过期时间
            CookieUtil.addCookie(response,"loginToken",loginToken,604800);

            return ResultUtil.success(user1);

        } else {
            return ResultUtil.error(ResultEnum.LOGIN_ERROR);
        }
    } else {
        return ResultUtil.error(ResultEnum.CAPTCHA_ERROR);
    }

}
```

用户退出

```java
@DeleteMapping(value = "/session", produces = {"application/json;charset=UTF-8"})
public Result logout(HttpSession session,HttpServletRequest request,HttpServletResponse response ) {

    //删除session和cookie
    session.removeAttribute("user");
    CookieUtil.clearCookie(request, response, "loginToken");

    return ResultUtil.success();
}
```

拦截器；实现自动登陆功能

```java
public class UserInterceptor implements HandlerInterceptor {
@Autowired
private UserService userService;

public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object o) throws Exception {
    User sessionUser = (User) request.getSession().getAttribute("user");

    // 已经登陆了，放行
    if (sessionUser != null) {
        return true;
    } else {
        //得到带过来cookie是否存在
        String loginToken = CookieUtil.findCookieByName(request, "loginToken");
        if (StringUtils.isNotBlank(loginToken)) {
            //到数据库查询有没有该Cookie
            User user = userService.findUserByLoginToken(loginToken);
            if (user != null) {
                request.getSession().setAttribute("user", user);
                return true;
            } else {
                //没有该Cookie与之对应的用户(Cookie不匹配)
                CookieUtil.clearCookie(request, response, "loginToken");
                return false;
            }
        } else {

            //没有cookie、也没有登陆。是index请求获取用户信息，可以放行
            if (request.getRequestURI().contains("session")) {
                return true;
            }

            //没有cookie凭证
            response.sendRedirect("/login.html");
            return false;
        }
    }
	}
}
```

总结一下上面代码的思路：

- 用户登录时，验证用户的账户和密码
- 生成一个Token保存在数据库中，将Token写到Cookie中
- 将用户数据保存在Session中
- 请求时都会带上Cookie，检查有没有登录，如果已经登录则放行

## 多系统登陆的问题与解决

### Session 不共享问题

单系统登录功能主要是用Session保存用户信息来实现的，但我们清楚的是：多系统即可能有多个Tomcat，而Session是依赖当前系统的Tomcat，所以系统A的Session和系统B的Session是**不共享**的。

解决系统之间Session不共享问题有一下几种方案：

- Tomcat集群Session全局复制（集群内每个tomcat的session完全同步）【会影响集群的性能呢，不建议】
- 根据请求的IP进行**Hash映射**到对应的机器上（这就相当于请求的IP一直会访问同一个服务器）【如果服务器宕机了，会丢失了一大部分Session的数据，不建议】
- 把Session数据放在Redis中（使用Redis模拟Session）【**建议**】

我们可以将登陆功能单独抽取出来，做成一个子系统。

抽取出来成为子系统

SSO （登陆系统）的逻辑如下：

```java
// 登录功能(SSO单独的服务)
@Override
public TaotaoResult login(String username, String password) throws Exception {

    //根据用户名查询用户信息
    TbUserExample example = new TbUserExample();
    Criteria criteria = example.createCriteria();
    criteria.andUsernameEqualTo(username);
    List<TbUser> list = userMapper.selectByExample(example);
    if (null == list || list.isEmpty()) {
        return TaotaoResult.build(400, "用户不存在");
    }
    //核对密码
    TbUser user = list.get(0);
    if (!DigestUtils.md5DigestAsHex(password.getBytes()).equals(user.getPassword())) {
        return TaotaoResult.build(400, "密码错误");
    }
    //登录成功，把用户信息写入redis
    //生成一个用户token
    String token = UUID.randomUUID().toString();
    jedisCluster.set(USER_TOKEN_KEY + ":" + token, JsonUtils.objectToJson(user));
    //设置session过期时间
    jedisCluster.expire(USER_TOKEN_KEY + ":" + token, SESSION_EXPIRE_TIME);
    return TaotaoResult.ok(token);
}
```

其他子系统登陆时，请求SSO（登陆系统）进行登陆，将返回的token写到Cookie 中，下次访问时则把Cookie 带上：

```java
public TaotaoResult login(String username, String password, 
        HttpServletRequest request, HttpServletResponse response) {
    //请求参数
    Map<String, String> param = new HashMap<>();
    param.put("username", username);
    param.put("password", password);
    //登录处理
    String stringResult = HttpClientUtil.doPost(REGISTER_USER_URL + USER_LOGIN_URL, param);
    TaotaoResult result = TaotaoResult.format(stringResult);
    //登录出错
    if (result.getStatus() != 200) {
        return result;
    }
    //登录成功后把取token信息，并写入cookie
    String token = (String) result.getData();
    //写入cookie
    CookieUtils.setCookie(request, response, "TT_TOKEN", token);
    //返回成功
    return result;

}
```

总结：

- SSO系统生成一个token，并将用户信息存到Redis中，并设置过期时间
- 其他系统请求SSO系统进行登录，得到SSO返回的token，写到Cookie中
- 每次请求时，Cookie都会带上，拦截器得到token，判断是否已经登录

## Cookie跨域的问题

上面我们解决了Session不能共享的问题，但其实还有另一个问题。**Cookie是不能跨域的**

比如说，我们请求`<https://www.google.com/>`时，浏览器会自动把`google.com`的Cookie带过去给`google`的服务器，而不会把`<https://www.baidu.com/>`的Cookie带过去给`google`的服务器。

这就意味着，**由于域名不同**，用户向系统A登录后，系统A返回给浏览器的Cookie，用户再请求系统B的时候不会将系统A的Cookie带过去。

针对Cookie存在跨域问题，有几种解决方案：

1. 服务端将Cookie写到客户端后，客户端对Cookie进行解析，将Token解析出来，此后请求都把这个Token带上就行了
2. 多个域名共享Cookie，在写到客户端的时候设置Cookie的domain。
3. 将Token保存在SessionStroage中（不依赖Cookie就没有跨域的问题了）

## CAS 原理

说到单点登录，就肯定会见到这个名词：CAS （Central Authentication Service），下面说说CAS是怎么搞的。

**如果已经将登录单独抽取成系统出来**，我们还能这样玩。现在我们有两个系统，分别是`www.java3y.com`和`www.java4y.com`，一个SSO`www.sso.com`

首先，用户想要访问系统A`www.java3y.com`受限的资源(比如说购物车功能，购物车功能需要登录后才能访问)，系统A`www.java3y.com`发现用户并没有登录，于是**重定向到sso认证中心，并将自己的地址作为参数**。请求的地址如下：

- `www.sso.com?service=www.java3y.com`

sso认证中心发现用户未登录，将用户引导至登录页面，用户进行输入用户名和密码进行登录，用户与认证中心建立**全局会话（生成一份Token，写到Cookie中，保存在浏览器上）**

随后，认证中心**重定向回系统A**，并把Token携带过去给系统A，重定向的地址如下：

- `www.java3y.com?token=xxxxxxx`

接着，系统A去sso认证中心验证这个Token是否正确，如果正确，则系统A和用户建立局部会话（**创建Session**）。到此，系统A和用户已经是登录状态了。

此时，用户想要访问系统B`www.java4y.com`受限的资源(比如说订单功能，订单功能需要登录后才能访问)，系统B`www.java4y.com`发现用户并没有登录，于是**重定向到sso认证中心，并将自己的地址作为参数**。请求的地址如下：

- `www.sso.com?service=www.java4y.com`

注意，因为之前用户与认证中心`www.sso.com`已经建立了全局会话（当时已经把Cookie保存到浏览器上了），所以这次系统B**重定向**到认证中心`www.sso.com`是可以带上Cookie的。

认证中心**根据带过来的Cookie**发现已经与用户建立了全局会话了，认证中心**重定向回系统B**，并把Token携带过去给系统B，重定向的地址如下：

- `www.java4y.com?token=xxxxxxx`

接着，系统B去sso认证中心验证这个Token是否正确，如果正确，则系统B和用户建立局部会话（**创建Session**）。到此，系统B和用户已经是登录状态了。

看到这里，其实SSO认证中心就类似一个**中转站**。