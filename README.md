# PromiseC
解析Promise原理

```js
var p = new PromiseB(function(resolve, reject) {
  setTimeout(
    function() {
      //A动画
      console.log('xue');
      resolve();
    },
    300
  );
});
```
>创建一个Promise对象，然后执行function,为什么加上setTimeout？通过setTimeout机制，将resolve中执行回调的逻辑放置到JS任务队列末尾, 为了防止then方法注册回调之前，resolve函数就执行了。接着执行then方法，将想要在Promise异步操作成功时执行的回调放入callback队列（注册回调函数）。
```js
p.then(function() {
  setTimeout(
    function() {
      //B动画
      console.log('chao');
    },
    300
  );
});
```
执行then的时候会根据当前state状态值做进一步处理。
```js
if (this._status === this.STATUS.RESOLVE) {
   fun.apply(fun, this.succArg);
} else {
   this.succCbs.push(fun);
}
```
创建Promise实例时传入的函数会被赋予一个函数类型的参数，即resolve，它接收一个参数value，代表异步操作返回的结果：

 ```js
PromiseB.prototype._execFun = function (fun) {
   var that = this;
   if (this._isFunction(fun)) {
      fun(function () {
               that.succArg = Array.prototype.slice.apply(arguments);
               that._status = that.STATUS.RESOLVE;
               that.resolve.apply(that, arguments);
           }, function () {
               that.failArg = Array.prototype.slice.apply(arguments);
               that._status = that.STATUS.REJECT;
               that.reject.apply(that, arguments);
      });
   } else {
       this.resolve(fun);
   }
};
```

当异步操作执行成功后，用户会调用resolve方法，这时候其实真正执行的操作是执行callbacks队列中的回调:

```js
PromiseB.prototype.resolve = function () {
    var arg = arguments,
    ret,
    callback = this.succCbs.shift();
    if (this._status === this.STATUS.RESOLVE && callback) {
       ret = callback.apply(callback, arg);
       if (!(ret instanceof PromiseB)) {
          var _ret = ret;
          ret = new PromiseB(function (resolve) {
          	setTimeout(function () {
	            resolve(_ret);
	       });
          });
          ret.succCbs = this.succCbs.slice();
       }
    }
};
```
