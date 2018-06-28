> 前几天看到了一篇文章关于防抖和节流的实现, 可是没有this, that, closure的细节. 我在翻阅了一些文档之后做了一点自己的总结. 原文: https://mp.weixin.qq.com/s/xMCna_VtoOev0K5uK1J0aQ

Debounce 防抖
=========================
本质上就是在用户停止输入一段时间后再触发某些操作的功能. 比如用户打字, 你得确保用户输入完了他想输入的内容后再开始搜索, 否则意义不大.

```js
function debounce(func, delay) {
  var timeout;
  return function(e) {
    console.log('clear', timeout, e.target.value);
    clearTimeout(timeout);
    var that = this, args = arguments;
    
    timeout = setTimeout(function() {
      func.apply(that, args);
      console.log(this, that)
    }, delay);
    console.log('new timer', timeout, e.target.value);
  };
}

var validation = debounce(function(e) {
  console.log('---', e, this);
}, 1000);

document.querySelector('input').addEventListener('input', validation);
```

代码的解释请参考原文, 这里只解释以下几点:

**1. 为什么clearTimeout可以清除上一个timeout?**                       
首先要搞明白一个东西, 那就是这个validation方法究竟是啥.                     
错误的看法: 是一个包装了debounce的方法, 每一次输入都会触发这个方法, 所以每次都会声明一个新的timeout然后传给里面的闭包用. 所以每次clear的都是自己.             
正确的看法: 是一个运行完debounce返回(生成)的方法, 所以他本质上就是debounce里面的那个方法, 只不过能够访问到某些外部资源(这里是timeout, func, delay)              

看到这里就明白了, 在第一次声明并赋值timeout之后, 你之后再input调用的listener总会引用到同一个timeout而不是再去声明他, 因为这时你已经没有权限去声明这个变量了. 所以在你clear时, 清除的就是这个timeout.

**2. 为什么要用闭包?**              
经过上面的解释, 你知道了我们需要引用一个存在的timeout然后后续能让我们的方法一直对其操作, 这就是为什么要用闭包. 如果这里直接把cleartimeout等代码直接写在debounce里面, 那么就会变成上面第一种错误的情况.              

**3. 为什么要用that, 几个位置的this究竟是啥?**               
首先, 这段代码的本意是把正确的上下文context应用到原方法上, 所以原方法的上下文不会变调. 那什么是正确的context呢?是这个被绑定的input. 那为什么要在setTimeout外面去保存this呢?因为setTimeout会改变上下文.改成什么呢?改成widnow. 看到这里你肯定又会问为什么要改? 好吧其实这里不叫改, 而是你应该自己去找谁是当前的上下文, 通俗的讲, 是谁在使用这个方法.             

我们从头来看: addEventListener方法是会给方法绑定上下文的(真正的改上下文), 绑定到具体的对象上使之更有意义. 以下MDN原话: addEventListener() works by adding a function or an object that implements EventListener to the list of event listeners for the specified event type **on the EventTarget on which it's called**. 所以这时validation方法具有了正确的context, 他的this指向input. 然而谁是setTimeout的回调函数的context, 他处在一个方法中而不是一个对象中,也没有人用apply/call/bind给他具体的context, 一直往上找最后找到了window. 这this就是为什么要用that保存一下外面的context.                   

**4. 为什么不用胖箭头?**                 
看完上面一题, 你能立即想到把上面改成setTimeout(()=>{console.log(this)}, 1000)去解决这个问题, 他会把外面的context正确的传进来. 你只需要看胖箭头函数在哪里创建就可以.                


