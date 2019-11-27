*************
pthread mutex
*************

``pthread_mutex_t`` - posix下表示锁类型的结构.

主要使用以下5个函数进行操作:
* ``pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr)``

	初始化锁变量\ *mutex*\ , *attr*\ 为锁属性, *NULL*\ 表示使用默认属性.

* ``pthread_mutex_lock(pthread_mutex_t *mutex)``

	加锁, 如果指定的锁已经在其它地方被加锁, 则挂起等待锁被释放.

* ``pthread_mutex_trylock(pthread_mutex_t *mutex)``

	加锁, 但是如果指定的锁已经在其它地方被加锁, 返回\ ``EBUSY``\ , 而不是挂起等待.

* ``pthread_mutex_unlock(pthread_mutex_t *mutex)``

	释放锁.

* ``pthread_mutex_destroy(pthread_mutex_t *mutex)``