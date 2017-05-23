一.闭包的定义：闭包是指有权访问另一个函数作用域中的变量的含糊。创建闭包的常见方式，就是在函数内部创建另一个函数。（引自高级程序设计）

二.闭包的用途：

1.匿名自执行函数 
  我们知道所有的变量，如果不加上var关键字，则默认的会添加到全局对象的属性上去，这样的临时变量加入全局对象有很多坏处，
  
  比如：别的函数可能误用这些变量；造成全局对象过于庞大，影响访问速度(因为变量的取值是需要从原型链上遍历的)。
  
  除了每次使用变量都是用var关键字外，我们在实际情况下经常遇到这样一种情况，即有的函数只需要执行一次，其内部变量无需维护，
  
  比如UI的初始化，那么我们可以使用闭包：
  
  var datamodel = {    
    table : [],    
    tree : {}    
  };    
     
 (function(dm){    
    for(var i = 0; i < dm.table.rows; i++){    
       var row = dm.table.rows[i];    
       for(var j = 0; j < row.cells; i++){    
           drawCell(i, j);    
       }    
    }    
       
    //build dm.tree      
  })(datamodel);   
  
  我们创建了一个匿名的函数，并立即执行它，由于外部无法引用它内部的变量，因此在执行完后很快就会被释放，关键是这种机制不会污染全局对象。
  
  2.缓存
  
  再来看一个例子，设想我们有一个处理过程很耗时的函数对象，每次调用都会花费很长时间，那么我们就需要将计算出来的值存储起来，当调用这个函数的时候，首先在缓
  
  存中查找，如果找不到，则进行计算，然后更新缓存并返回值，如果找到了，直接返回查找到的值即可。闭包正是可以做到这一点，因为它不会释放外部的引用，从而函数
  
  内部的值可以得以保留。
  
  var CachedSearchBox = (function(){    
    var cache = {},    
       count = [];    
    return {    
       attachSearchBox : function(dsid){    
           if(dsid in cache){//如果结果在缓存中    
              return cache[dsid];//直接返回缓存中的对象    
           }    
           var fsb = new uikit.webctrl.SearchBox(dsid);//新建    
           cache[dsid] = fsb;//更新缓存    
           if(count.length > 100){//保正缓存的大小<=100    
              delete cache[count.shift()];    
           }    
           return fsb;          
       },    
     
       clearSearchBox : function(dsid){    
           if(dsid in cache){    
              cache[dsid].clearSelection();      
           }    
       }    
    };    
})();    
     
CachedSearchBox.attachSearchBox("input1");    

这样，当我们第二次调用CachedSearchBox.attachSerachBox(“input1”)的时候，我们就可以从缓存中取道该对象，而不用再去创建一个新的searchbox对象。

3.实现封装

可以先来看一个关于封装的例子，在person之外的地方无法访问其内部的变量，而通过提供闭包的形式来访问：

var person = function(){    
    //变量作用域为函数内部，外部无法访问    
    var name = "default";       
       
    return {    
       getName : function(){    
           return name;    
       },    
       setName : function(newName){    
           name = newName;    
       }    
    }    
}();    
     
print(person.name);//直接访问，结果为undefined    
print(person.getName());    
person.setName("abruzzi");    
print(person.getName());    
   
得到结果如下：  
   
undefined  
default  
abruzzi  

4 闭包的另一个重要用途是实现面向对象中的对象，传统的对象语言都提供类的模板机制，这样不同的对象(类的实例)拥有独立的成员及状态，互不干涉。虽然

JavaScript中没有类这样的机制，但是通过使用闭包，我们可以模拟出这样的机制。还是以上边的例子来讲：

function Person(){    
    var name = "default";       
       
    return {    
       getName : function(){    
           return name;    
       },    
       setName : function(newName){    
           name = newName;    
       }    
    }    
};    
     
     
var john = Person();    
print(john.getName());    
john.setName("john");    
print(john.getName());    
     
var jack = Person();    
print(jack.getName());    
jack.setName("jack");    
print(jack.getName());    
   
运行结果如下：  
   
default  
john  
default  
jack  

由此代码可知，john和jack都可以称为是Person这个类的实例，因为这两个实例对name这个成员的访问是独立的，互不影响的。
  
三.闭包的坏处

1.内存消耗
 通常来说，函数的活动对象会随着执行期上下文一起销毁，但是，由于闭包引用另外一个函数的活动对象，因此这个活动对象无法被销毁，这意味着，闭包比一般的函数
 
 需要更多的内存消耗。尤其在IE浏览器中需要关注。由于IE使用非原生javascript对象实现DOM对象，因此闭包会导致内存泄露问题，例如：
 
 function A(){  
      var a=document.createElement("div")，//  
            msg="Hello";  
       a.onclick=function(){  
          alert(msg);  
          }  
   }  
 A(); 
 
 以上的闭包会在IE下导致内存泄露，假设A()执行时创建的作用域对象ScopeA，ScopeA引用了DOM对象a,DOM对象a引用了function(aleert(msg))，函数
 
 function(alert(msg))引用了ScopeA，这是一个循环引用，在IE会导致内存泄露。
 
 2.性能问题

使用闭包时，会涉及到跨作用域访问，每次访问都会导致性能损失。因此在脚本中，最好小心使用闭包，它同时会涉及到内存和速度问题。不过我们可以通过把跨作用域

变量存储在局部变量中，然后直接访问局部变量，来减轻对执行速度的影响。
 
