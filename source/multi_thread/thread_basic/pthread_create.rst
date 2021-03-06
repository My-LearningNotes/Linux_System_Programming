线程创建
=========

在POSIX线程(``pthread``)的情况下, 程序开始运行时, 它是以单进程中的单个控制线程(主线程)启动的, 新增的线程可以通过调用\ ``pthread_create``\ 函数创建.


``pthread_create``
------------------

``pthread_create``\ 函数用来创建一个新的线程.

函数原型:

    .. code-block:: c

        int pthread_create(pthread_t *restrict thread, 
                           const pthread_attr_t *restrict attr, 
                           void *(* start_routine)(void *), 
                           void *restrict arg);

        # 如果成功, 返回0;
        # 如果不成功, 返回一个非零的错误码.

    + 参数\ ``thread``\ 指向保存线程ID的\ ``pthread_t``\ 结构;
    + 参数\ ``attr``\ 表示一个封装了线程的各种属性的属性对象, 用来配置线程的运行, 如果为NULL, 则使新线程具有默认的属性;
    + 参数\ ``start_routine``\ 是线程的入口函数;
    + \ ``arg``\ 是传递给\ ``start_routine``\ 函数的参数.


.. note::

    关键字\ ``restrict``\ 仅对指针有用, 修饰指针, 告诉编译器被\ ``restrict``\ 修饰的指针指向的对象, 只能通过这个指针或基于这个指针的其他指针进行操作.

.. note::

    作为新创建的线程的入口函数, 原型必须为: ``void * start_routine(void *)``

     * 函数的参数是一个\ ``void``\ 类型的指针, 即\ ``void *p``;
       如果需要向入口函数传递参数, 使用\ ``pthread_create``\ 函数的第四个参数;
     * 函数的返回类型也是一个\ ``void``\ 类型的指针，即\ ``void *``;
       入口函数的返回值，表示线程的退出码;


+--------------------------------------------------------------------------------------+
| pthread_create的错误形式及相应的错误码                                               |
+--------+-----------------------------------------------------------------------------+
| 错误   | 原因                                                                        |
+========+=============================================================================+
| EAGAIN | 系统没有创建线程所需的资源，或者创建线程会超出系统对一个进行中线程总数的限制|
+--------+-----------------------------------------------------------------------------+
| EINVAL | attr参数是无效的                                                            |
+--------+-----------------------------------------------------------------------------+
| EPERM  | 调用程序没有适当的权限来设定调度策略或attr指定的参数                        |
+--------+-----------------------------------------------------------------------------+


``pthread_create``\ 函数会自动启动新创建的线程, 而不需要一个单独的启动操作.
线程创建时并不能保证哪个线程会先运行: 是新创建的线程还是调用线程.
新创建的线程可以访问进程的地址空间, 并且继承调用线程线程的浮点环境和信号屏蔽字, 但是该线程的未决信号集被清除.

注意pthread函数在调用失败时通常会返回错误码, 它们并不像其他的POSIX函数一样设置\ ``errno``\ .
每个线程都提供\ ``errno``\ 的副本, 这只是为了与使用\ ``errno``\ 的现有函数兼容.
在线程中, 从函数中返回错误码更为清晰整洁, 不需要依赖那些随着函数执行不断变化的全局状态, 因而可以把错误的范围限制在引起出错的函数中.

