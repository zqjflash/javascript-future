# javascript-future
下一代javascript

## 同步。异步！协作程？

  * 同步与异步（`同步编码代码量小（简单），符合人类理解方式（好写）`）

    * 同步XHR示例代码：
    
    ```js

	    var xhr = new XMLHttpRequest();
	
	    xhr.open('get', '/', false);
	
	    try {
	
	        xhr.send();
	
	    } catch (e) {
	
	        console.error(e);
	
	    }

    ```

    * 异步XHR示例代码：

    ```js
    
        var xhr = new XMLHttpRequest();

        xhr.open('get', '/');

        xhr.onerror = function(e) {

            console.error(e);

        };

        xhr.onreadystatechange = function() {

            if (xhr.readyState === 4) {

                console.log(xhr.responseText);

            }

        };

        xhr.send();
   
    ```

  * 同步有啥不好？

    * Javascript是单线程*的，必须等待前面的代码执行完毕之后才能继续执行后面的代码。

    * 在浏览器中它将导致UI阻塞（白屏等），如果连接服务器响应很慢，那么用户浏览器将冻结，不能进行其他操作。

        ![Alt text](https://raw.githubusercontent.com/zqjflash/javascript-future/master/js-synchronous.png)

    * 在后台服务中它将导致QPS下降，如果服务请求量大，响应时间会升高，处理不好还会导致雪崩。

## 回调地狱

  * 某特工九死一生偷到了美国火箭发射程序的Node.js源代码的最后一页......，`异步嵌套层数多，不易读`

    ```js

		                  });
		                });
		              });
		            });
		          });
		        });
		      });
		    });
		  });
		});

    ```

## Promise

  * Promise在非线性编码中有助于代码流程的控制，其表示一个异步操作的最终结果。`解决了回调无限嵌套的问题`

    ```js

        ajax(url, function(err, results) {
            if (e) {
                console.error(e);
                return;
            }
            ajax(url2, function(err, results) {
                if (e) {
                    console.error(e);
                } else {
                    console.log(results);
                }
            });
        });

    ```

    //--------------------------------------------------------

    ```js

        ajax(url).then(function(results) {
            return ajax(url2);
        }, function(e) {
            console.error(e);
        }).then(function(results) {
            console.log(results);
        }, function(e) {
            console.error(e);
        });

    ```

## then

  * 每个then为一个处理单元，用于获取最终的值或拒绝的原因，并可返回处理结果供下级使用。（`虽然可以解决嵌套，但仍然存在大量回调`）

    * 当前处理单元的返回值，将在下个单元作为resolve传入；
    * 当前处理单元抛出异常，将在下个单元为reject传入；
    * 当前处理单元返回Promise的对象，会替换链式调用的Promise对象

    ```js

	    .then(function () {
	        return Promise.resolve(1);
	    }, function (e) {
	        return 0;
	    });

    ```

    ```js

	    .then(function (r) {
	        throw (r + 2);
	    }, function (e) {
	        return 0;
	    });

    ```

    ```js

	    .then(function (r) {
	        console.log(r);
	    }, function (e) {
	        console.error(e);
	    });

    ```

## 能否`使用同步的写法，写出异步的代码？`

  * 同步：代码易于编写（线性编码）符合人类理解方式；
  * 异步：性能高，符合计算机运行原理。

## 生成器函数（Generator）`没有了回调，一切都显得那么自然！`

  * 生成器函数的执行可被中断，在中断的期间与其控制代码进行协作，完成后再恢复函数的执行。

  ```js

	  ajax(url, function(err, results) {
	      if (err) {
	          console.error(e);
	          return;
	      }
	      ajax(url2, function(err, results) {
	          if (err) {
	              console.error(e);
	          } else {
	              console.log(results);
	          }
	      });
	  });

  ```

  ```js

	  function *send() {
	      try {
	          yield ajax(url);
	          console.log(yield ajax(url2));
	      } catch (e) {
	          console.error(e);
	      }
	  }

  ```

## yield

  * 通过yield表达式在内部来中断函数的执行，当外部返回信号时再恢复函数的执行。
  * 调用生成器函数返回其对应迭代器（ES6 iterator）

  ```js

      function *foo(x) {
          var y = 2 * (yield (x + 1));
          var z = yield (y / 3);
          return (x + y + z);
      }

  ```

  ```js

      var it = foo(5);
      it.next() => { value: 6, done: false }
      it.next(12) => { value: 8, done: false }
      it.next(13) => { value: 42, done: true }

  ```

  * 执行过程示意图如下：

    ![Alt text](https://raw.githubusercontent.com/zqjflash/javascript-future/master/future_yield.png)

## 委托

  * 可以将一个生成器的迭代器控制权交给另一个生成器；
  * 在yield中，返回值在next()中传入；
  * 在yield *中，返回值是在return中传入。

  ```js

      function *foo() {
          yield 2;
          yield 3;
          return "foo";
      }

      function *bar() {
          yield 1;
          var v = yield *foo();
          console.log(v);
          yield 4;
      }

      var it = bar();
      it.next() => { value: 1, done: false }
      it.next() => { value: 2, done: false }
      it.next() => { value: 3, done: false }
      it.next() => { value: 4, done: false }
      it.next() => { value: undefined, done: true }

  ```

  * 执行过程示意图如下：

    ![Alt text](https://raw.githubusercontent.com/zqjflash/javascript-future/master/future_entrust.png)

## 异常

  * 可以"同步"，也可以"反方向"的捕获异常；
  * throw()方法产生一个异常传入，如果没有对应的try...catch进行捕获，这个错误将会被传出去

  ```js

      function *foo() {
          var x;
          try {
              x = yield 3;
              console.log(x);
          } catch (e) {
              x = 0;
              console.error(e);
          }
          x = yield x;
          return x.toUpperCase();
      }

      var it = foo();
      it.next() => { value: 3, done: false }
      it.throw("Oops!"); => { value: 0, done: false }
      try {
          it.next(1);
      } catch (e) {
          console.error(e);
      }

  ```

  * 执行过程示意图如下：

    ![Alt text](https://raw.githubusercontent.com/zqjflash/javascript-future/master/future_exception.png)

  * 为每个生成器单独写一个对应的迭代协作方法，不能减少代码量且不可重用。
  * 但我们通过上面的例子，可以抽象出非常通用的迭代协作处理方法：
    * 异步结果作为yield的返回值；
    * 异步异常通过throw()抛出。
  * yield操作（传入值）如何处理？
    * `由于yield只支持一个操作数，也就是说只能有一个传入值`
    * 所以，唯一的传入值必须包含"产生"异步调用所需的全部信息（Promise、thunked function）

## Promise + Generator

  * 结合Promise提供的控制与错误处理机制，可以实现通用的协作方法（生成器运行`协作`函数）。

  ```js

      function *foo(url) {
          var result;
          try {
              result = yield ajax(url);
              result = yield ajax(result.url);
          } catch (e) {
              return { error: e };
          }
          return { data: result };
      }

      function run(it, callback) {
          (function iterate(val, succ) {
              var ret = succ ? it.next(val) : it.throw(val);
              if (!ret.done) {
                  ret.value.then(function(r) {
                      iterate(r, true);
                  }, function(e) {
                      iterate(e, false);
                  });
              } else {
                  callback(ret.value);
              }
          }());
      }

      run(foo("url"), function(result) {
          console.log(result);
      });

  ```

## Thunkify + Generator

  * 常见异步函数的最后一个参数均为回调函数（Error-first callback），可将此回调分离（Thunkify）出函数体，以实现通用的协作方法（生成器运行函数）。

  ```js

      fs.readFile(path, function(err, data) {
          console.log(data);
      });
      // thunkify
      var readFile = thunkify(fs.readFile);
      // invoke
      readFile(path)(function(err, data) {
          console.log(data);
      });
      // generator
      console.log(yield readFile(path));

      function run(it, callback) {
          (function iterate(val, succ) {
              var ret = succ ? it.next(val) : it.throw(val);
              if (!ret.done) {
                  ret.value(function(err, data) {
                      if (err) {
                          iterate(err, false);
                      } else {
                          iterate(data, true);
                      }
                  });
              } else {
                  callback(ret.value);
              }
          }());
      }

  ```

## co

  * 通用的生成器协作（运行）函数库。
  * 同一个生成器中yield传入值类型可不同
  * yield传入值之间可相互嵌套

  * yield传入值类型如下：
    * promises
    * thunks (functions)
    * array (parallel execution)
    * objects (parallel execution)
    * generators (delegation)
    * generator functions (delegation)

  * co函数调用流程图：

    ![Alt text](https://raw.githubusercontent.com/zqjflash/javascript-future/master/future_co.png)

## koa

  * 由Express原班人马打造，基于生成器的下一代Node.js web框架；
  * 用户请求通过中间件，遇到yield next关键字时，会被传递到下游中间件；
  * 在yield next捕获不到下一个中间件时，逆序返回继续执行代码；
  * koa中间件（生成器函数）比起Express中间件（回调函数）代码更加直观。

  * koa框架示例代码如图：

    ![Alt text](https://raw.githubusercontent.com/zqjflash/javascript-future/master/future_koa.png)

## CSP-Style

  * Communicating Sequential Processes，通讯顺序进程
  * Sequential - 在生成器内，异步行为以类同步的方式编写，置顶而下顺序运行。
  * Processes - 高内聚的多个生成器配对在一起，合作完成一个更大的任务。

  * Q：为什么用多个生成器而不只用一个？
  * A：能力与专注分离，易于理解与维护。

  * Communicating - 生成器之间存在协调的机制：
    * 数据（接收与发送数据）
    * 控制权（适时挂起与唤醒）
  * Koa的use(...)方法就是CSP的一个例子。