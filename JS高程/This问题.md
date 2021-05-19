# This

### 什么是This

通常来讲，在ECMAScript规范中，**this关键字执行为当前执行环境的ThisBinding**。MDN上解释为，**在绝大多数情况下，函数的调用方式决定了this的值**。所以，下面将从执行上下文和调用方式进行总结。

- 全局上下文

  由下面的代码可知，在浏览器环境中，全局执行上下文中 this指代window。

  ```javascript
  console.log(window===this)  //true
  
  ```

- 函数上下文

  1. 作为对象调用，this指向该对象。

     ```javascript
     function f(){
         console.log(this.a)
     }
     var obj={
         a:1,
         f:f
     }
     var a=2
     f()      //2
     obj.f()  //1
     ```

  2. 作为函数直接调用，this指向全局window

  3. 作为构造函数调用,this指向的是当前构造的新对象，(*new会创建一个空对象，将this指向当前空对象，最后赋值返回*)

     ```javascript
     function Create(){
         this.name='jack'
     }
     var o=new Create()
     console.log(o.name) //'jack'
     
     ```

箭头函数是没有自己的this，他的this一般指向最近的外层。

### 如何改变This

- .call(thisArg,arg1,arg2,....)

  ```javascript
  var foo={
      value:1
  }
  function bar(a,b){
      console.log(this.value)
      console.log(a,b)
  }
  bar.call(foo,2,3)    //1,2,3
  //此时的this指向fOO对象，ab则是后面的指定参数
  ```

- apply(thisArg,[argsArrr])

  ```javascript
  var foo={
      value:1
  }
  function bar(a,b){
      console.log(this.value)
      console.log([a,b])
  }
  bar.call(foo,[2,3])   //1  [2,3]
  //相较于call，apply可以接受一个数组作为参数
  ```

- .bind(thisArg,arg1,arg2,...)

   bind传参与call类似，但是他不会立即执行，而是返回一个新的函数引用。

   ```javascript
     var obj={
         value:10,
         say:function(){
           setTimeout(function(){
               console.log(this.value)
           },1000)
         }
           }
          obj.say() //此时this的指向为window
   -----------------------
       var obj={
         value:10,
         say:function(){
           setTimeout(function(){
               console.log(this.value)
           }.bind(this),1000)
         }
           }
          obj.say() //10
   ```


  总结，三个方法均可以改变this的值。接受的第一个值永远是this指向的目标对象，其中call，apply会立即执行。bind会返回一个绑定函数可稍后执行。call接受全部参数，apply接受一个数组集合。

------
### call/apply/bind模拟实现

- call
  ```javascript
  //ES5
  Function.prototype.mycall=function(context){
       context=Object(context)||window
       context.fn=this
       var args=[]
       for(var i=0;i<arguments.length;i++){
           args.push('arguments['+i+']')
       }
       var res=eval('context.fn('+args+')')
       delete context.fn
       return res
  }
  //ES6
  //可以将循环换成 args=[...arguments].slice(1)
  //  conetext(...args)
  ```

- apply

  ```javascript
  Function.prototype.myapply=function(context,arr){
      contex=Object(context)||window
      context.fn=this
      var res
      if(!arr){
        res= context.fn() 
      }else{
          var args=[]
          for(var i=0;i<arr.length;i++){
               args.push('arr['+i+']');
          }
          res=eval('context.fn('+args+')')
      }
      delete context.fn
      return res
  }
  ```

- bind

  ```javascript
  Function.prototype.mybind=function(context){
      if(typeof this!=='function'){
         throw new Error("Function.prototype.bind - what is trying to be bound                         is not callable");  
      }
      var self=this
      var args=Array.prototype.slice.call(arguments,1)
      var fbound= function(){
          var bindArgs=Array.prototype.slice.call(arguments)
           // 当作为构造函数时，this 指向实例，此时结果为 true，将绑定函数的 this 指向该实例，可以让实例获得来自绑定函数的值
          // 当作为普通函数时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
          return self.apply(this instanceOf fn?this:self,context,args.concat(bindArgs))
      }
       var fn=function(){}
       fn.prototype=self.prototype
       fbound.prototype=new fn()
       return fbound
  }
  ```

  如果是作为构造函数，此时的this指向为当前对象，新对象根本没有属性值。

### 如何实现new

```JavaScript
function newObj(){
    var obj=new Object()
    Constructor=[].shift.call(arguments)
    obj.__proto__=Constuctor.prototype          //获取构造函数原型上的方法
    var ret= Constructor.apply(obj,arguments)   //获取构造函数中的属性
    return typeOf ret==='Object'?ret:obj        //确保返回值是Object类型
}
```

