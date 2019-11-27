*************
线程sleep函数
*************

``sleep()``
===========

Sleep for the specified number of seconds.

* **Synopsis**

.. code-block:: c

	#include <unistd.h>

	unsigned int sleep(unsigned int seconds);

* **Description**

``sleep()`` makes the calling thread sleep until *seconds* seconds have elapsed or a signal arrives which is not ignored.


``usleep()``
============

Suspend execution for microseconds intervals.

* **Synopsis**

.. code-block:: c

	#include <unistd.h>

	int usleep(useconds_t used);

* **Description**

The ``usleep()`` function suspends execution of the calling thread for (at least) *usec* microseconds.

* **Notes**

The type *useconds_t* is an unsigned integer type capable of holding integers in the range [0, 1000000].
Programs will be more portable if they never mention this type explicitly. Use

.. code-block:: c

	...
	unsigned int usecs;
	...
	usleep(usecs);


``nanosleep()``
===============

High-resolution sleep.

* **Synopsis**

.. code-block:: c

	#include <time.h>

	int nanosleep(const struct timespec *req, struct timespec *rem);

* **Description**

``nanosleep`` suspends the execution of the calling thread until either at least the time specified in *req* has elapsed, 
or the delivery of a signal that triggers the invocation of a handler in the calling thread or that terminates the process.

If the call is interrupted by a signal handler, ``nanosleep()`` returns -1, set *errno* to ``EINTR``, 
and writes the remaining time into the structure pointed to by *rem* unless *rem* is NULL. 
The value of *rem* can then be used to call ``nanosleep()`` again and complete the specified pause.

The structure *timespec* is used to specify intervals of time with nanosecond precision. It is defined as follows:

.. code-block:: c

	struct timespec {
		time_t 	tv_sec;		/* Seconds */
		long 	tv_nsec:	/* Nanoseconds */
	};

The vlaue of nanoseconds field must be in the range 0 to 999999999.

Compared to ``sleep()`` and ``usleep``, ``nanosleep()`` has the following advantages: 

	* It provides a higher resolution for specifying the sleep interval;
	* POSIX.1 explicitly specifies that it does not interact with signals;
	* It makes the task of resuming a sleep that has been interrupted by a signal handler easier.

* **Notes**

If the interval specified in *req* is not an exact multiple of the granularity underlying clock, 
then the interval will be rounded up to the next multiple(如果指定的睡眠时间不是时钟单位的整数倍, 则睡眠时间会被圆整为时钟单位的整数倍).
Furthermore, after the sleep completes, there may stil be a delay before the CPU becomes free to once again execute the calling thread.

.. note::

	我们使用睡眠函数时, 以秒, 毫秒, 微秒或纳秒为单位指定睡眠的时间, 这些时间单位是便于人类使用的时间单位;
	但计算机的clock有自己的时间单位, 以秒/毫秒/微秒/纳秒为单位的时间, 最终要转换为计算机的clock的时间单位.

The fact that ``nanosleep()`` sleeps for a relative interval can be problematic if the call is repeatedly restarted after being interrupted by signals, 
since the time between the interruptions and restarts of the call will lead to drift in the time when the sleep finally completes. 
This problem can be avoided by using ``clock_nanosleep()`` with an absolute time value.

.. note::

	使用\ ``sleep()``\ ,\ ``usleep()``\ 或\ ``nanosleep()``\ 使线程睡眠时, 如果线程睡眠时被信号中断, 那么当该线程重新睡眠时就会产生时间漂移的问题.
	因为它们的睡眠时间使用\ **relative time**\ 计时, 如果睡眠中被信号中断了, 当信号处理完再次进入睡眠状态时, 被信号中断和处理的这段时间不会被计入睡眠时间, 
	所以最终线程被唤醒的时间会比预期的睡眠时间要长.

	对于sleep的精度有严格要求的线程, 可以使用\ ``clock_nanosleep()``\ 函数, 指定使用\ **absolute time**\ 计时, 可以避免睡眠时间漂移的问题.

.. note::

	``sleep()``\ 和\ ``nanosleep()``\ 都是使线程睡眠一段时间后被唤醒, 但是两者的实现完全不同.

	Linux中没有提供系统调用\ ``sleep()``\ , ``sleep()``\ 是在库函数中实现的, 它是通过调用\ ``alarm()``\ 来设定报警时间, 
	调用\ ``sigsuspend()``\ 将线程挂起在信号\ ``SIGALARM``\ 上, ``sleep()``\ 只能精确到秒级上.

	``nanosleep()``\ 则是Linux中的系统调用, 它是使用定时器来实现的, 该调用使调用线程睡眠, 并往定时器队列上加入一个\ ``timer_list``\ 型定时器, 
	``timer_list``\ 结构里包括唤醒时间以及唤醒后执行的函数, 通过\ ``nanosleep()``\ 加入的定时器的执行函数仅仅完成唤醒当前线程的功能.
	系统通过一定的机制定时检查这些队列(比如通过系统调用陷入核心后, 从核心返回用户态之前, 要检查当前线程的时间片是否已经耗尽, 如果是则调用\ ``schedule()``\ 函数重新调度, 
	该函数中就会检查定时器队列, 另外慢中断返回前也会做此检查), 如果定时时间已超过, 则执行定时器指定的函数唤醒调用线程. 当然, 由于系统时间片可能丢失, 所以\ ``nanosleep()``\ 精度也不是很高.

	``alarm()``\ 也是通过定时器实现的, 但是精度只能精确到秒级, 另外, 它设置的定时器执行函数是在指定时间向当前线程发送\ ``SIGALARM``\ 信号.



.. tip::

	对于不同时间的睡眠使用不同的函数, 秒级别的使用\ ``sleep()``\ , 毫秒/微秒级别的使用\ ``usleep()``\ , 纳秒级别的使用\ ``nanosleep()``\ .


``clock_nanosleep()``
=====================

**High-resolution sleep with specifiable clock.**

* **Synopsis**

.. code-block:: c

	#include <time.h>

	int clock_nanosleep(clockid_t clock_id, int flags,
					const struct timespec *request,
					struct timespec *remain);

* **Description**

Like ``nanosleep``, ``clock_nanosleep`` allows the calling thread to sleep for an interval specified with nanosecond precision.
It differs in allowing the caller to select the clock against which the sleep interval is to be measured, 
and in allowing the sleep interval to be specified as either an absolute or a relative value.

The time values passed to an returned by this call are specified using ``timespec`` structures, defined as follows:

.. code-block:: c

	struct timespec {
		time_t tv_sec;		/* Seconds */
		long   tv_nsec;		/* Nanoseconds [0 .. 999999999] */
	};

The ``clock_id`` argument specifies the clock against which the sleep interval is to be measured.
This argument can have one of the following values:

	* ``CLOCK_REALTIME``
		A settable system-wide real-time clock.

	* ``CLOCK_MONOTONIC``
		A nonsettable, monotonically increasing clock that measures time since some unspecified point in the past that does not change after system startup.

	* ``CLOCK_PROCESS_CPUTIME_ID``
		A settable per-process clock that measures CPU time consumed by all threads in the process.

If *flags* is 0, then the value specified in *request* is interpreted as an interval relative to the current value of the clock specified by *clock_id*.

If *flags* is **TIMER_ABSTIME**, then *request* is interpreted as an absolute time as measured by the clock, *clock_id*.
If *request* is less than or equal to the current value of the clock, the ``clock_nanosleep()`` returns immediately without suspending the calling thread.

``Clock_nanosleep()`` suspends the execution of the calling thread until either at least the time specified by *request* has elapsed, 
or a signal is delivered that causes a signal handler to be called or that terminates the process.

If the call is interrupted by a signal handler, ``clock_nanosleep()`` fails with the error ``EINTER``.
In addition, if *remian* is not NULL, and *flags* was not **TIMER_ABSTIME**, it returns the remaining unslept time in *remain*.
This value can then be used to call ``clock_nanosleep()`` again and complete a (relative) sleep.

* **Notes**

If the interval specified in *request* is not an exact multiple of the granularity underlying clock, then the interval 
will be rounded up to the next multiple. Furthermore, after the sleep completes, there may still be a delay before the 
CPU becomes free to once again execute the calling thread.

Using an absolute timer is useful for preventing timer drift problems.
To perform a relative sleep that avoids these problems, call ``clock_gettime`` for the desired clock, 
add the desired interval to the returned tiem value, and then call ``clock_nanosleep()`` with the ``TIMER_ABSTIME`` flag.

``clock_nanosleep()`` is never restarted after being interrupted by a signal handler.

The *remain* argument is unused, and unnecessary, when *flag* is ``TIMER_ABSTIME``\ .
