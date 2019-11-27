线程标识
=========

就像每个进程都有一个进程ID一样, 每个线程也有一个线程ID.
进程ID在整个系统中是唯一的, 但线程ID不同, 线程ID只在它所属的进程环境中有效.


``pthread_t``
-------------

线程ID用\ ``pthread_t``\ 数据类型来表示.

``pthread_equal``
-----------------

使用\ ``pthread_equal``\ 函数来判断两个线程ID是否相等.

函数原型:

    .. code-block:: c
        :emphasize-lines: 1

        int pthread_equal(pthread_t tid1, pthread_t tid2);

        # 若相等, 返回非零(True)
        # 若不等, 返回零(False)

``pthread_self``
----------------

在一个线程中使用\ ``pthread_self``\ 函数来获取线程ID.

函数原型:

    .. code-block:: c
        :emphasize-lines: 1

        pthread_t pthread_self(void)

    
当线程需要识别以线程ID作为标识的数据结构时, ``pthread_self``\ 函数可以和\ ``pthread_equal``\ 函数一起使用.
