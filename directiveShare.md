An Angular Custom Directive Example: showMessage
============================================================================
> 把controllers, html, 对dom的操作全部集成在一个directive主要是为了实现component style, 实现向angular2的迁移。我也注意到1.5增加了angular.component这样一个功能, 不过不管怎么说自定义directive仍然是angular1核心

> 本文的例子基于angular官网例子, 但因为他们是出于解释directive而写的例子, 所以有些东西并没有完全'封装'在directive里面, 所以实际上并不能直接使用, 对比参考 https://docs.angularjs.org/guide/directive

> 本文完整代码已host在[**plnkr**](http://plnkr.co/edit/yUo4aGckD1ktxl4ngt6p?p=preview)上, 供看官们把玩。本例是自己的一点心得，欢迎分享，还有很多不足望指正。作者github:zzs1020

## 动机: Refactoring Angular Apps to Component Style
and make Angular great again -- [原文戳这里](http://teropa.info/blog/2015/10/18/refactoring-angular-apps-to-components.html)

简而言之，你面试的时候说你用如下目录结构估计没人要你了：

- app/
  - controllers/
    - xxCtrl.js
  - directives/
    - xxDirective.js
  - services/
    - xxService.js
  ....

而都做成component的形式易于维护＋重用（DRY）
- app/
  - components/
    - messageComponent/
      - short-message.html
      - showMessageDirective.js
  ....

都是为了顺应时代的变化，建议看看那篇原文写得不错，虽不能直接像react或angular2那么方便地component，但我们也不能自暴自弃啊

## 功能解释 ```<show-message></show-message>``` 
support|yes/no
-------|------
码农指定样式success, warning, danger| y 
用户关闭气泡| y
气泡自动消失| y
根据外部controller提供的信息动态显示名字| y
有fade效果 | y

## 效果预览
![show-message-screenshot](https://github.com/zzs1020/sharedInfo/blob/master/show-message-screenshot.png)

## 开始编码
### step 1:
首先你需要一个app啥的并且引入到html中就不解释了, 大部分plnkr自己生成, 上代码:app.js和index.html
```javascript
var app = angular.module('plunker', []);
app.controller('MainCtrl', function() {
  var self = this;
  self.name = 'Steve';
});
```
``` html
<html ng-app="plunker">
  <head>
    <!--已省略其他meta/title/script标签-->
    <script src="app.js"></script>
  </head>

  <body ng-controller="MainCtrl">
      <show-message type='success'> 
            successfully logged in! Welcome, {{name}}
      </show-message>
  </body>
  </html>
```
你需要关注的只有show-message这个标签而已, 类型由程序员自己选定来制定合适的样式(比如登陆成功用success, 失败用danger, 命名跟bootstrap学的), 用来显示的信息放在show-message标签里,并且显示对应的用户名,这里用到transclude的知识,后面会讲。另外你可能注意到我没用$scope,这是为了避免**scope soup**, 简单来说就是当你的代码很多, 在不同ctrl里面都用$scope的话, 以后你查代码的时候, 在html里面你就很难分清哪个属性是哪个ctrl的,难维护。

### step 2:
首先我们来实现主要功能 app.js和short-message.html
```javascript
app.directive('showMessage', [ function() {
  return {
    restrict: 'E', 
    transclude: true, //can visit outside scope
    scope: {},
    templateUrl: 'short-message.html',
    controller: [function() {
      var self = this;
      self.isHidden = false;
      self.close = function() {
        self.isHidden = true;//when hide, only hide <div> inside <show-message>
      };
    }],
    controllerAs: 'messageCtrl'
  }
}]);
```
```html
<div ng-hide='messageCtrl.isHidden'>
  <a href="" ng-click='messageCtrl.close()'>&times;</a>
  <div ng-transclude></div>
</div>
```
我希望把这个directive做成一个标签而不是一个属性或class, 毕竟用```<show-message>文字</show-message>```看起来更合理些。所以设置restrict:'E'。如果用E, 在设置style的时候会有个小问题需要解决, 导致我之前认为只能用'A'来取代，后面会说

为什么function用[]包围? 原因是如果直接在()内注入服务的话,最后minify文件的时候**一定会出问题**,所以一概写成诸如['$scope', function($scope){}]更稳健, 当然在没有注入其他服务时不需要加[],可是保不准你什么时候就想加一个服务呢,到时候再在一堆括号里面添加方括号不是很蛋疼?

directive名字写成camel风格的,在html用时写成dash风格的, angular自动转换识别, 我们凡人不用管。

模版是写在外面文件的, 除非模版内容超级短, 建议写在外面并用templateUrl引用。这个文件就是用div包裹的一个链接+用户自己写的内容。这个a标签显示x的样子,用户点击后调用ctrl的close方法, close方法简单的设置isHidden=false, 得益于angular的2-way data binding, 界面上的ng-hide会自动做出反应。因为我们想这变成一个整体,所以应该把ctrl直接集成在这个directive里面,并且命名为messageCtrl。

至此, 已完成-粗糙地。

### step 3:
加样式 在controllerAs:'messageCtrl'后面加逗号和link
```javascript
link: function(scope, elem, attrs, messageCtrl) {
      var bgColor = '#DFF0D8';
      var borderColor = '#D6E9C6';
      var textColor = '#3C763D';

      if (attrs.type === 'success') { //green style
        bgColor = '#DFF0D8';
        borderColor = '#D6E9C6';
        textColor = '#3C763D';
      } else if (attrs.type === 'danger') { //red style
        bgColor = '#F2DEDE';
        borderColor = '#FFB8B1';
        textColor = '#D9534F';
      } else if (attrs.type === 'warning') { //yellow style
        bgColor = '#FFDDA7';
        borderColor = '#FFC378';
        textColor = '#FF6600';
      }

      elem.css({
        display: 'block', //must be block, coz show-message will show inline
        border: 'solid 1px ' + borderColor,
        borderRadius: '3px',
        color: textColor,
        backgroundColor: bgColor,
        opacity: 1,
        transition: 'opacity 2s'  //if use all, then will have other style first(black border)
      });
      
      $timeout(function(){
        elem.css({
          opacity:0
        });
      }, 5000);
    }
```
通常推荐如果你要直接操作dom的话,不是在ctrl里面,而是在link里面。传递的参数中scope就是你这个directive里面的上下文,elem是调用这个directive的tag,这里就是show-message,attrs是你在tag上写的属性,还记得前面我们加了个type='success'吗?attrs.type就可以得到值了, 最后传进去的是你的ctrl。

首先,程序员指定什么type就给什么样式,做个判断。再用css写进去样式的时候注意得加display:block, 这就是前面说的'E'的问题, 因为你自己做的tag默认不是个块级元素, 所以你的样式显示不出来, 同样如果你设置成'A'的话不加在其他块级元素上也不会正确显示样式,比如span

你可能希望就算用户不点击泡泡也自动消失,就加一个opacity的transition即可, 然后用$timeout让5秒后设置css-display-none完成。当然你需要在directive里注入$timeout服务。 

可是目前有2个小问题, 1.那就是, show-message自己变成块级元素后, 前面hide的都是show-message '里面' 的div,  自己的边框啥的还是没有被隐藏。所以最后加上一个$watch, 判断如果里面div被隐藏了,就设置show-message本身也消失。你可能想问为什么$watch不用注入,因为他是$rootscope的方法,而你在link传进来的scope参数就是继承自rootscope,所以可以用$watch。
```js
scope.$watch('messageCtrl.isHidden', function(newValue, oldValue) {
        if(newValue != oldValue) {
          elem.css({
            display: 'none'     //if don't none here, show-message still have border style
          });
        }
      })
```
2是你发现元素只是隐藏了，木有消失，还占着地方呢！这里我用了个笨办法再加一个timeout 7秒后等fade效果结束后再设置display:none
```js
$timeout(function(){
        elem.css({
          display: 'none'
        });
      }, 7000); 
```
个人觉得很可能还有更好的办法在这，如果知道请点击上方github issue告诉我！

说了这么多，你一定想吐槽我前面为毛不解释scope，写了个空{}跟不用isolate scope有啥区别？正常情况下如果你只显示一个气泡，没有任何区别。可是如果你像我plnkr上例子一样弄三个出来一起显示就有区别了。现在他们三个分别用了独立的scope，你点击一个气泡的x才不会影响到其他的泡泡。不用isolate scope那他们就是一根绳子上的蚂蚱... =o=

最后，按照angular官话，directives should clean up themselves by using scope.$on/elem.on('$destroy', function(){..}),如果你用了$interval一类的服务。避免内存泄漏。~~当然我们这里是不需要的。~~ 划掉的这句话，我觉得既对又错。因为这个directive不会重复执行，最多只会7秒后也结束了，谈不上leak，所以可以不加。但如果说你有一个按钮让(恶意软件)用户不停的点击触发这个泡泡就会有问题了。所以咱们还是加上：
```js
scope.$on('$destroy', function(){
        $timeout.cancel(t1Promise);
        $timeout.cancel(t2Promise);
      });
```
[戳这看详细解释](http://www.bennadel.com/blog/2548-don-t-forget-to-cancel-timeout-timers-in-your-destroy-events-in-angularjs.htm), 另外一个知识点是angular的$timeout返回的是promise哦，不是js里的id啦，所以你可以拿到这个promise做各种操作，再一次感受promise的强大吧

### 结尾
完成。现在你就可以把这个directive单独放在一个文件里面，带上short-message.html，一起放在某个文件夹下并命名messageComponent，而这文件夹就是所谓的能提供单独功能性的模块，到处reuse了。科科
