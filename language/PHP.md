# PHP

---

## 运行环境

- LNMP = Linux + Nginx + MySQL + PHP
- LAMP = Linux + ApacheHttpd + MySQL + PHP

思考：

- LNMP 与 LAMP 的优缺点？

## 开发规范

- PSR：PHP Framework Interop Group 组织的 PHP 语言开发规范，很多知名框架和类库都遵守 PSR 规范

## 引用传递

- 值传递

  ```php
  $a = 'Hello world!';
  echo $a; // Hello world!
  ```

- 引用传递

  ```php
  $a = 'Hello';
  $b = &$a;
  $b .= ' world';
  echo($a); // Hello world!
  ```

  ```php
  function foo(&$a) {
    $a .= ' world!';
  }
  $a = 'Hello';
  foo($a);
  echo $a; // Hello world!
  ```

思考

- 以下情况属于值传递还是引用传递

  ```php
  class A {
    public $test = 'Hello dog!';
    public function display() {
      echo $this->test;
    }
  }
  
  function foo ($obj) {
    $obj->test = 'Hello world!';
  }
  
  $a = new A();
  foo($a);
  echo($a->display());
  ```

## 引用计数

- https://github.com/niklaus0823/demo-php-zval

## 魔术方法

- http://php.net/manual/zh/language.oop5.magic.php

## 加载方式

- `require*`

- `include*`

- `spl_autoload_register`

  **foo1.php**

  ```php
  class Foo1
  {
      static public function display()
      {
          echo "I am foo1!\n";
      }
  }
  ```

  **foo2.class.php**

  ```php
  class Foo2
  {
      static public function display()
      {
          echo "I am foo2!\n";
      }
  }
  ```

  **run.php**

  ```php
  $my_autoload1 = function ($classname)
  {
      echo "entry my_autoload1 \n";
      $filename = "./". lcfirst($classname) .".php";
      include_once($filename);
  };
  
  $my_autoload2 = function ($classname)
  {
      echo "entry my_autoload2 \n";
      $filename = "./". lcfirst($classname) .".class.php";
      include_once($filename);
  };
  
  spl_autoload_register($my_autoload1);
  spl_autoload_register($my_autoload2);
  
  Foo1::display();
  Foo2::display();
  ```

  **ext/spl/php_spl.c**

  > 源码分析，通过 `spl_autoload_register` 注册自动加载函数，会在 `Zend` 引擎中维护一个 `autoload` 队列，即可添加多个 `autoload` 函数，并在 `PHP` 调用当前文件未知的类时，触发 `autoload_func` 的调用（执行顺序按注册顺序）。

  

  ```c
  PHP_FUNCTION(spl_autoload_register)
  {
      char *func_name, *error = NULL;
      int  func_name_len;
      char *lc_name = NULL;
      zval *zcallable = NULL;
      zend_bool do_throw = 1;
      zend_bool prepend  = 0;
      zend_function *spl_func_ptr;
      autoload_func_info alfi;
      zval *obj_ptr;
      zend_fcall_info_cache fcc;
  
      if (zend_parse_parameters_ex(ZEND_PARSE_PARAMS_QUIET, ZEND_NUM_ARGS() TSRMLS_CC, "|zbb", &zcallable, &do_throw, &prepend) == FAILURE) {
          return;
      }
  
      if (ZEND_NUM_ARGS()) {
          if (Z_TYPE_P(zcallable) == IS_STRING) {
              if (Z_STRLEN_P(zcallable) == sizeof("spl_autoload_call") - 1) {
                  if (!zend_binary_strcasecmp(Z_STRVAL_P(zcallable), sizeof("spl_autoload_call"), "spl_autoload_call", sizeof("spl_autoload_call"))) {
                      if (do_throw) {
                          zend_throw_exception_ex(spl_ce_LogicException, 0 TSRMLS_CC, "Function spl_autoload_call() cannot be registered");
                      }
                      RETURN_FALSE;
                  }
              }
          }
      
          if (!zend_is_callable_ex(zcallable, NULL, IS_CALLABLE_STRICT, &func_name, &func_name_len, &fcc, &error TSRMLS_CC)) {
              alfi.ce = fcc.calling_scope;
              alfi.func_ptr = fcc.function_handler;
              obj_ptr = fcc.object_ptr;
              if (Z_TYPE_P(zcallable) == IS_ARRAY) {
                  if (!obj_ptr && alfi.func_ptr && !(alfi.func_ptr->common.fn_flags & ZEND_ACC_STATIC)) {
                      if (do_throw) {
                          zend_throw_exception_ex(spl_ce_LogicException, 0 TSRMLS_CC, "Passed array specifies a non static method but no object (%s)", error);
                      }
                      if (error) {
                          efree(error);
                      }
                      efree(func_name);
                      RETURN_FALSE;
                  }
                  else if (do_throw) {
                      zend_throw_exception_ex(spl_ce_LogicException, 0 TSRMLS_CC, "Passed array does not specify %s %smethod (%s)", alfi.func_ptr ? "a callable" : "an existing", !obj_ptr ? "static " : "", error);
                  }
                  if (error) {
                      efree(error);
                  }
                  efree(func_name);
                  RETURN_FALSE;
              } else if (Z_TYPE_P(zcallable) == IS_STRING) {
                  if (do_throw) {
                      zend_throw_exception_ex(spl_ce_LogicException, 0 TSRMLS_CC, "Function '%s' not %s (%s)", func_name, alfi.func_ptr ? "callable" : "found", error);
                  }
                  if (error) {
                      efree(error);
                  }
                  efree(func_name);
                  RETURN_FALSE;
              } else {
                  if (do_throw) {
                      zend_throw_exception_ex(spl_ce_LogicException, 0 TSRMLS_CC, "Illegal value passed (%s)", error);
                  }
                  if (error) {
                      efree(error);
                  }
                  efree(func_name);
                  RETURN_FALSE;
              }
          }
          alfi.closure = NULL;
          alfi.ce = fcc.calling_scope;
          alfi.func_ptr = fcc.function_handler;
          obj_ptr = fcc.object_ptr;
          if (error) {
              efree(error);
          }
      
          lc_name = safe_emalloc(func_name_len, 1, sizeof(long) + 1);
          zend_str_tolower_copy(lc_name, func_name, func_name_len);
          efree(func_name);
  
          if (Z_TYPE_P(zcallable) == IS_OBJECT) {
              alfi.closure = zcallable;
              Z_ADDREF_P(zcallable);
  
              lc_name = erealloc(lc_name, func_name_len + 2 + sizeof(zend_object_handle));
              memcpy(lc_name + func_name_len, &Z_OBJ_HANDLE_P(zcallable),
                  sizeof(zend_object_handle));
              func_name_len += sizeof(zend_object_handle);
              lc_name[func_name_len] = '\0';
          }
  
          if (SPL_G(autoload_functions) && zend_hash_exists(SPL_G(autoload_functions), (char*)lc_name, func_name_len+1)) {
              if (alfi.closure) {
                  Z_DELREF_P(zcallable);
              }
              goto skip;
          }
  
          if (obj_ptr && !(alfi.func_ptr->common.fn_flags & ZEND_ACC_STATIC)) {
              /* add object id to the hash to ensure uniqueness, for more reference look at bug #40091 */
              lc_name = erealloc(lc_name, func_name_len + 2 + sizeof(zend_object_handle));
              memcpy(lc_name + func_name_len, &Z_OBJ_HANDLE_P(obj_ptr), sizeof(zend_object_handle));
              func_name_len += sizeof(zend_object_handle);
              lc_name[func_name_len] = '\0';
              alfi.obj = obj_ptr;
              Z_ADDREF_P(alfi.obj);
          } else {
              alfi.obj = NULL;
          }
  
          if (!SPL_G(autoload_functions)) {
              ALLOC_HASHTABLE(SPL_G(autoload_functions));
              zend_hash_init(SPL_G(autoload_functions), 1, NULL, (dtor_func_t) autoload_func_info_dtor, 0);
          }
  
          zend_hash_find(EG(function_table), "spl_autoload", sizeof("spl_autoload"), (void **) &spl_func_ptr);
  
          if (EG(autoload_func) == spl_func_ptr) { /* registered already, so we insert that first */
              autoload_func_info spl_alfi;
  
              spl_alfi.func_ptr = spl_func_ptr;
              spl_alfi.obj = NULL;
              spl_alfi.ce = NULL;
              spl_alfi.closure = NULL;
              zend_hash_add(SPL_G(autoload_functions), "spl_autoload", sizeof("spl_autoload"), &spl_alfi, sizeof(autoload_func_info), NULL);
              if (prepend && SPL_G(autoload_functions)->nNumOfElements > 1) {
                  /* Move the newly created element to the head of the hashtable */
                  HT_MOVE_TAIL_TO_HEAD(SPL_G(autoload_functions));
              }
          }
  
          if (zend_hash_add(SPL_G(autoload_functions), lc_name, func_name_len+1, &alfi.func_ptr, sizeof(autoload_func_info), NULL) == FAILURE) {
              if (obj_ptr && !(alfi.func_ptr->common.fn_flags & ZEND_ACC_STATIC)) {
                  Z_DELREF_P(alfi.obj);
              }                
              if (alfi.closure) {
                  Z_DELREF_P(alfi.closure);
              }
          }
          if (prepend && SPL_G(autoload_functions)->nNumOfElements > 1) {
              /* Move the newly created element to the head of the hashtable */
              HT_MOVE_TAIL_TO_HEAD(SPL_G(autoload_functions));
          }
  skip:
          efree(lc_name);
      }
  
      if (SPL_G(autoload_functions)) {
          zend_hash_find(EG(function_table), "spl_autoload_call", sizeof("spl_autoload_call"), (void **) &EG(autoload_func)); /* 注意此处 */
      } else {
          zend_hash_find(EG(function_table), "spl_autoload", sizeof("spl_autoload"), (void **) &EG(autoload_func));
      }
      RETURN_TRUE;
  } /* }}} */
  ```

- `__autoload` : PHP 7.2 弃用`

思考

- require 和 include 区别

## 数据库连接

- mysql_connect 短连接
- mysql_pconnect 持久连接

思考

- 两种连接方式的使用时机是什么？各有什么优缺点？

## 关于 Swoole

> 以下是韩天峰的文章里摘抄的，没有用过这个框架，仅做过了解。
>
> 个人理解：
>
> 1. 解决了 PHP 一直以来的被诟病的软肋，但对大部分 PHP 项目而言，可能并太用不上。
> 2. 内置的 Server 让 PHP 能以常驻内存的方式运行，这才是让 PHP 有本质上改变的地方。
> 3. 至于性能提升方面，需要进行压力测试后才可以评估。
> 4. 即使如韩天峰所说，不需要改动代码即可支持，但实际上真要改造一个项目，很多观念是需要扭转，甚至一个只熟悉 PHP 的程序员是无法完全胜任的。

Swoole 是 PHP 的扩展框架，作者韩天峰（Rango）。

PHP 的同步阻塞模式存在很多缺陷，并发能力弱，慢速请求会导致服务不可用，而 Swoole 框架内置了协程模式，并且兼容其他同步阻塞 PHP 框架，例如 Laravel，Yii，实现不需要改动代码，即可无缝切换协程模式，从而提高系统的性能和兵并发能力。

目前这套框架在虎牙直播的长连接推送服务中表现不俗，仅用 2 台服务器就支持了十万人同事在线，峰值每秒推送 10 万条消息

Swoft Farmework：https://www.swoft.org/

## SQL注入，XSS 劫持