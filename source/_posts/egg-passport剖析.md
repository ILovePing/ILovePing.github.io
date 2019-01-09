---
title: egg的passport实现探究
date: 2018-06-13 11:08:11
categories: 技术
tags: [node,passport，authentication]
---

之前梳理了目前常见的几种web验证机制，重新回到正题，谈谈egg-passport如何实现验证的。

先看官网对于egg-passport的解释。
> passport plugin for egg, base on passportjs.
>
> Egg 在它之上提供了 egg-passport 插件，把初始化、鉴权成功后的回调处理等通用逻辑封装掉，使得开发者仅需调用几个 API 即可方便的使用 Passport 。

可以知道egg-passport的核心就是passport，不了解passport，是搞不定egg在其上的封装逻辑的。

### passport使用
我们要把矛头先转向最重要的passport。
先了解passport一波：它是node的中间件，可以嵌到express请求前后，用来验证用户身份，可以选择不同的验证方式（strategies），可以定制或选择第三方的验证逻辑，常用的有：Local/Session/Basic/Digest/OpenId/Bearer Strategy几种。local就是使用username和password登录;basic就是之前提到过的常见的base64加密后放到请求头Authentication传输;digest就是弥补basic的安全缺陷设计的密码摘要认证;bear就是基于OAuth2.0的一种验证。

passport最基础的用法如下（以下例子涉及到node服务的都是基于express）：

1.passport初始化。

```javascript
var passport = require('passport')
app.use(passport.initialize());
app.use(passport.session());
```
2.通过passport.use配置本地验证策略，实例化该策略需要传入一个自定义的验证用户信息的方法。

```javascript
var LocalStrategy = require('passport-local').Strategy;

passport.use(new LocalStrategy(
  function(username, password, done) {
    User.findOne({ username: username }, function (err, user) {
      if (err) { return done(err); }
      if (!user) {
        return done(null, false, { message: 'Incorrect username.' });
      }
      if (!user.validPassword(password)) {
        return done(null, false, { message: 'Incorrect password.' });
      }
      return done(null, user);
    });
  }
));
```
3.在express实例路由中使用配置好的passport中间件，/login的post请求进行身份验证，成功失败都指定了跳转地址，如果开启了步骤二中的session配置，那么登录成功之后req.user就保留了登录信息，下次请求就会自动带上sessionId（减少服务端内存占用，一般就是使用一个唯一的sessionId映射用户信息，当服务端拿到req.user时，会通过passport.deserializeUser()去查询用户信息），当然这基本就属于session实现了，这里就不赘述。

```javascript
app.post('/login',
  passport.authenticate('local', {
  successRedirect: '/',
  failureRedirect: '/login'
  })
);

```
这样一个基本的passport验证中间件就应用进去了，那么我们可以看出这个passport关键的就是上面使用到的几个方法：use、initialize、session、authenticate以及strategy部分。好的，那我们就可以带着目标去看看源码里这几个方法都实现了什么能达到这样的效果。

### passport探究
翻看passport包，代码基本都在lib中，我们从index.js入口关键代码开始看。

```javascript
exports = module.exports = new Passport();
```
可以看到index文件主要导出了一个实例化的authenticator对象，我们require("passport")的就是这个instance对象。
我们看下authenticator.js的部分代码：

```javascript
var SessionStrategy = require('./strategies/session');
function Authenticator() {
  this._key = 'passport';
  this._strategies = {};
  this._serializers = [];
  this._deserializers = [];
  this._infoTransformers = [];
  this._framework = null;
  this._userProperty = 'user';

  this.init();
}
Authenticator.prototype.init = function() {
  this.framework(require('./framework/connect')());
  this.use(new SessionStrategy());
};
Authenticator.prototype.framework = function(fw) {
  this._framework = fw;
  return this;
};
// 1.初始化执行的两个方法
Authenticator.prototype.initialize = function(options) {
  options = options || {};
  this._userProperty = options.userProperty || 'user';

  return this._framework.initialize(this, options);
};
Authenticator.prototype.session = function(options) {
  return this.authenticate('session', options);
};
// 2.使用localStrategy的use方法
Authenticator.prototype.use = function(name, strategy) {
  if (!strategy) {
    strategy = name;
    name = strategy.name;
  }
  if (!name) { throw new Error('Authentication strategies must have a name'); }

  this._strategies[name] = strategy;
  return this;
};
// 3.路由中配置中间件使用的验证方法
Authenticator.prototype.authenticate = function(strategy, options, callback) {
  return this._framework.authenticate(this, strategy, options, callback);
};
module.exports = Authenticator;
```

这是部分代码，还有一些原型链上的扩展方法没有贴出来，我们按照我们使用passport步骤看上面使用到的方法。

1. 实例化```Authenticator```。首先我们实例化类的时候定义了一些内置属性，然后执行了init方法。在init方法中执行了两个方法，一是require了connect.js执行结果（是个对象，下文会讲）赋值给_framework变量，二是默认use了sessionStrategy，这里的策略代码如何我们先不管，use方法可以看到实际上就是将传入的strategy存到了_strategies对象中，以上是```require("passport")```拿到passport对象做的，小结一下，初始化就是初始化变量，给_framework和_strategies赋值了一下。
2. 使用```initialize()```和```session()```初始化。通过代码不难看出这两个方法最后实际上是调用了一开始赋值给```_framework```的initialize方法和authenticate方法。不难猜出上一点实例化给```_framework```赋的值应该就是一个拥有这两个key的对象。注：initialize方法通过传入```options.userProperty```指定session存放user的key，默认为```user```。
3. 使用```passport.use```配置localStrategy。同1中所述就是再往_strategies对象中存入实例化Strategy，key名为实例的属性name（查看passport-local代码可知此处name为"local"）。
4. 路由接入authenticate验证。ez，还是调用了```_framework```变量的authenticate方法，此处传入的是local的中间件。

上面过完，更加好奇这个关键的_framework到底存放了啥，也就是connect.js到底返回了啥，接着去看```./framework/connect.js```代码：

```javascript
var initialize = require('../middleware/initialize')
  , authenticate = require('../middleware/authenticate');
exports = module.exports = function() {

  // HTTP extensions.
  exports.__monkeypatchNode();

  return {
    initialize: initialize,
    authenticate: authenticate
  };
};

exports.__monkeypatchNode = function() {
  var http = require('http');
  var IncomingMessageExt = require('../http/request');

  http.IncomingMessage.prototype.login =
  http.IncomingMessage.prototype.logIn = IncomingMessageExt.logIn;
  http.IncomingMessage.prototype.logout =
  http.IncomingMessage.prototype.logOut = IncomingMessageExt.logOut;
  http.IncomingMessage.prototype.isAuthenticated = IncomingMessageExt.isAuthenticated;
  http.IncomingMessage.prototype.isUnauthenticated = IncomingMessageExt.isUnauthenticated;
};
```

ok，30行左右的代码，主要做了三点，对应require了的三个模块：

+ ```../http/request.js``` 扩展http可读流req接口，增加logIn，logOut，isAuthenticated，isUnauthenticated接口，望名生义, 配置了passport的express对象，req对象里的这几个方法来源于此，此处代码比较长不赘述，可以自行去看实现。
+ ```../middleware/initialize``` 一个每次请求到来后都执行passport初始化的中间件。

```javascript
module.exports = function initialize(passport) {
  return function initialize(req, res, next) {
  	 // 1.将Authenticator实例赋值给 req._passport.instance
    req._passport = {};
    req._passport.instance = passport;
	 // 2.将缓存的session赋值给req._passport.session
    if (req.session && req.session[passport._key]) {
      req._passport.session = req.session[passport._key];
    }

    next();
  };
};
```
+ ```../middleware/authenticate ```  一个每次请求调用都会执行authentic身份认证的中间件，通过闭包的形式返回。

```javascript
module.exports = function authenticate(passport, name, options, callback) {
  if (typeof options == 'function') {
    callback = options;
    options = {};
  }
  options = options || {};

  var multi = true;
  if (!Array.isArray(name)) {
    name = [ name ];
    multi = false;
  }

  return function authenticate(req, res, next) {
  // 确保http的req添加了logIn，logOut等方法扩展
    if (http.IncomingMessage.prototype.logIn
        && http.IncomingMessage.prototype.logIn !== IncomingMessageExt.logIn) {
      require('../framework/connect').__monkeypatchNode();
    }
    //... 下面一堆认证策略的模版代码
    var failures = [];
    function allFailed() {...}
     (function attempt(i) {
      var layer = name[i];
      if (!layer) { return allFailed(); }
      var prototype = passport._strategy(layer);
      if (!prototype) { return next(new Error('Unknown authentication strategy "' + layer + '"')); }
      var strategy = Object.create(prototype);
      strategy.success = function(user, info) {...}
      strategy.fail = function(challenge, status) {...}
      strategy.redirect = function(url, status) {...}
      strategy.pass = function() {next();};
      strategy.error = function(err) {...}
      strategy.authenticate(req, options);
     })(0);
  };
};
```
可以看到模版代码里是遍历name（策略name可以为数组多个），iife的闭包形式自执行，通过```Object.create(passport._strategy(name[i]))```创建一个基于配置的strategy作为原型的实例，并在此实例上定义了一些额外的方法，那么这些方法都是在哪里用到呢，看到这里是疑惑的，继续看下去，最后一行执行了```stragey.authenticate(req, options);```，也就是说这个```authenticate```是策略类自身已经定义好的，ok那我们去看下开头使用的passport-local代码是如何的吧。

```
var passport = require('passport-strategy')
  , util = require('util')
  , lookup = require('./utils').lookup;
function Strategy(options, verify) {
  if (typeof options == 'function') {
    verify = options;
    options = {};
  }
  if (!verify) { throw new TypeError('LocalStrategy requires a verify callback'); }

  this._usernameField = options.usernameField || 'username';
  this._passwordField = options.passwordField || 'password';

  passport.Strategy.call(this);
  this.name = 'local';
  this._verify = verify;
  this._passReqToCallback = options.passReqToCallback;
}
util.inherits(Strategy, passport.Strategy);
Strategy.prototype.authenticate = function(req, options) {
  options = options || {};
  var username = lookup(req.body, this._usernameField) || lookup(req.query, this._usernameField);
  var password = lookup(req.body, this._passwordField) || lookup(req.query, this._passwordField);

  if (!username || !password) {
    return this.fail({ message: options.badRequestMessage || 'Missing credentials' }, 400);
  }

  var self = this;

  function verified(err, user, info) {
    if (err) { return self.error(err); }
    if (!user) { return self.fail(info); }
    self.success(user, info);
  }

  try {
    if (self._passReqToCallback) {
      this._verify(req, username, password, verified);
    } else {
      this._verify(username, password, verified);
    }
  } catch (ex) {
    return self.error(ex);
  }
};
module.exports = Strategy;
```
看关键代码，一个Strategy函数，原形扩展一个authenticate函数就是上文提到的实例会执行的方法。查看该方法可以看到verified方法里面调用了之前在实例上扩展的success，fail，error模版方法，这个verified方法传给了内置的```_verify```变量，而该变量就是我们一开始如何使用passport+localStrategy时传入的方法：

```javascript
passport.use(new LocalStrategy(
  function(username, password, done) {
    User.findOne({ username: username }, function (err, user) {
      if (err) { return done(err); }
      if (!user) {
        return done(null, false, { message: 'Incorrect username.' });
      }
      if (!user.validPassword(password)) {
        return done(null, false, { message: 'Incorrect password.' });
      }
      return done(null, user);
    });
  }
));
```
**这里的done就是verified方法。**

可以串联起来，理一下，在new的时候传入一个签名为username，password，done的函数，里面自定义登录的验证逻辑，然后调用done即verified方法，执行对应的定义好的模版代码strategy.sucess/fail/redirect等方法。
