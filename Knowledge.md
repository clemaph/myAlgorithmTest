#### ES6中的新语法规范

- let / const
- class 创建类
- import / export ：ES6 Module 模块的导入导出规范（JS中的模块化规范 AMD -> CMD -> CommonJS -> ES6 Module）
- Arrow Function 箭头函数
- 模板字符串
- 解构赋值
- “...” 拓展、展开、剩余运算符
- Promise / async / await
- for of循环
- Set / Map
- Array / Object ... 提供的新方法
- ......

#### 数组去重

```javascript
/*===第一种===*/
~function(){
    function unique(){
        let obj={},
            _this=this;
        for(let i=0;i<_this.length;i++){
        	let item=_this[i];
            //in hasOwnProperty typeof ...
            if(obj.hasOwnProperty(item)){
                _this[i]=_this[_this.length-1];
                _this.length--;
                i--;
                continue;
            }
            obj[item]=item;
        }
        obj=null;
        return _this;
    }
    Array.prototype.myUnique=unique;
}();
let ary = [12,23,12,13,13,12,23,14,8];
ary.myUnique().sort((a,b)=>a-b);
```

```javascript
/*===第二种===*/
~function(){
    function unique(){
        let _this=this;
       	//=>首先基于new Set实现数组去重（结果是Set的实例）
        _this=new Set(_this);
        //=>再基于Array.from把类数组等变为数组
        _this=Array.from(_this);
        return _this;
    }
    Array.prototype.myUnique=unique;
}();
let ary = [12,23,12,13,13,12,23,14,8];
ary.myUnique().sort((a,b)=>a-b);
```

```javascript
/*===第三种（不推荐）===*/
~function(){
    function unique(){
        let _this=this;
       	for(let i=0;i<_this.length-1;i++){
            let item=_this[i],
                next=_this.slice(i+1);
            if(next.includes(item)){
                _this.splice(i,1);
                i--;
            }
        }
        return _this;
    }
    Array.prototype.myUnique=unique;
}();
let ary = [12,23,12,13,13,12,23,14,8];
ary.myUnique().sort((a,b)=>a-b);
```

回去拓展更多的实现数组去重的办法

#### QUERY-URL-PARAMS

> 用来解析URL地址中问号传参的信息

```javascript
~function(){
    /*
     * getParam:获取URL问号传参中某一个参数对应的值
     *   @params
     *      key:要获取值的属性名
     *   @return
     *      当前传递KEY对应的属性值
     * by zhufengpeixun on 2019/08/12
     */
    function getParam(key){
        //1.先获取URL字符串中所有问号参数信息
        //验证是否存在问号,不存在无需处理
        let obj={},
            _this=this,
            askIn=_this.indexOf('?');
        if(askIn===-1) return;
        //获取问号后面的参数信息
        let link=document.createElement('a'),
            askText=null;
        link.href=_this;
        askText=link.search.substring(1);
        //把参数信息解析成为键值对的方式
        askText.split('&').forEach(item=>{
            let [key,value] = item.split('=');
            obj[key] = value;
        });
        //2.获取对应的属性值并返回
        return obj[key];
    }
    String.prototype.getParam=getParam;
}();
var url="locallhost?key1=val1&key2=val2&key3=val3";
console.log(url.getParam("key3")); //=>'val3'
```

#### 柯理化函数

```javascript
/*
function fn(x,y){
    return function(z){
        return x+y+z;
    }
}
*/
let fn=(x,y)=>z=>x+y+z;
let res = fn(1,2)(3);
console.log(res); //=>6  1+2+3
```

#### CALL和APPLY的重写

```javascript
~function(){
    function changeThis(context){
    	context=context||window;
        //let arg=[].slice.call(arguments,1);
        let arg=[],
            _this=this,
            result=null;
        for(let i=1;i<arguments.length;i++){
            arg.push(arguments[i]);
        }
        context.$fn=_this;
        result=context.$fn(...arg);
        delete context.$fn;
        return result;
	};
    Function.prototype.changeThis=changeThis;
}();

let obj = {name:'Alibaba',$fn:100};
function fn(x,y){
    this.total=x+y;
    return this;
}
let res = fn.changeThis(obj,100,200);
//res => {name:'Alibaba',total:300}
```

基于ES6语法重构

```javascript
~function(){
    /*生成随机函数名：时间戳的方式*/
    function queryRandomName(){
        let time=new Date().getTime();
        return '$zhufeng'+time;
    }
    /*模拟CALL方法改变函数中的THIS*/
    function changeThis(context=window,...arg){
        let _this=this,
            result=null,
            ran=queryRandomName();
        context[ran]=_this;
        result=context[ran](...arg);
        delete context[ran];
        return result;
	};
    Function.prototype.changeThis=changeThis;
}();
```
把APPLY也重写一下
```javascript
~function(){
    /*生成随机函数名：时间戳的方式*/
    function queryRandomName(){
        let time=new Date().getTime();
        return '$zhufeng'+time;
    }
    /*模拟CALL方法改变函数中的THIS*/
    function changeThis(context=window,arg=[]){
        let _this=this,
            result=null,
            ran=queryRandomName();
        context[ran]=_this;
        result=context[ran](...arg);
        delete context[ran];
        return result;
	};
    Function.prototype.changeThis=changeThis;
}();
let res = fn.changeThis(obj,[100,200]);
```

#### TO-ARRAY

```javascript
let utils = (function(){
    /*
     * toArray：转换为数组的方法
     *   @params
     *      不固定数量，不固定类型
     *   @return
     *      [Array] 返回的处理后的新数组
     * by zhufengpeixun on 2019/08/12
     */
    /*
    function toArray(){
        /!*
        let arg=[];
        for(let i=0;i<arguments.length;i++){
            arg.push(arguments[i]);
        }
        return arg;
        *!/
        return Array.prototype.slice.call(arguments,0);
    }
    */
    //function toArray(...arg){
        //return arg;
    //}
    let toArray=(...arg)=>arg;
    return {
        toArray
    };
})();

let ary = utils.toArray(10,20,30);
//ary=[10,20,30]
ary = utils.toArray('A',10,20,30);
//ary=['A',10,20,30]
```

#### hasPubProperty

```javascript
~function(){
    /*
     * hasPubProperty:检测某个属性是否为对象的公有属性
     *   @params
     *     attr:要检测的属性名（字符串或者数字）
     *   @return 
     *     [Boolean] 检测的结果TRUE/FALSE
     * by zhufengpeixun on 2019/08/12
     */
    function hasPubProperty(attr){
        if(typeof attr!=="string" && typeof attr!=="number") return false;
        return (attr in this) && !(this.hasOwnProperty(attr));
    }
   	Object.prototype.hasPubProperty=hasPubProperty;
}();

obj.hasPubProperty('name')
```

#### class重构

```javascript
class Modal{
    constructor(element=document){
        this.element=element;
    }
    show(){
    	this.element.style.display='block';
	}
    hide(){
    	this.element.style.display='none';
	}
    static setPosition(x,y){
        this.position={x:x,y:y};
    }
}
Modal.position={x:100,y:200};
```
