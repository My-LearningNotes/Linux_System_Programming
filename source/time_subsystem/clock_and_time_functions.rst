************************
Clock and time functions
************************


``clockid_t``
=============

``clockid_t``\ 表示kernel的时钟的基本类型, 常用的有: ``CLOCK_REALTIME``, ``CLOCK_MONOTONIC``\ 等.

完整的time类型可以在\ ``/usr/include/linux/time.h``\ 中查看:

.. code-block:: c
	:emphasize-lines: 4, 5, 6, 7, 8, 9, 10, 11, 12, 13

	/*
 	 * The IDs of the various system clocks (for POSIX.1b interval timers):
 	 */
	#define CLOCK_REALTIME                  0
	#define CLOCK_MONOTONIC                 1
	#define CLOCK_PROCESS_CPUTIME_ID        2
	#define CLOCK_THREAD_CPUTIME_ID         3
	#define CLOCK_MONOTONIC_RAW             4
	#define CLOCK_REALTIME_COARSE           5
	#define CLOCK_MONOTONIC_COARSE          6
	#define CLOCK_BOOTTIME                  7
	#define CLOCK_REALTIME_ALARM            8
	#define CLOCK_BOOTTIME_ALARM            9


``struct timespec``
===================

用来定义一个time.

.. code-block:: c

	struct timespec {
		long tv_sec;	/* Seconds */
		long tv_nsec;	/* Nanoseconds */
	}

``struct timespec``\ 有两个成员, 一个是秒, 一个是纳秒, 所以最高精度是纳秒.

一般使用函数\ ``int clock_gettime(clockid_t, struct timespec*)``\ 来获取特定时钟的时间, 常用如下四种时钟:

	* ``CLOCK_REALTIME``
		系统实时时间, 即从UTC1970-1-1 0:0:0(Linux Epoch)开始计时的时间;
		该时间可以被用户修改.

	* ``CLOCK_MONOTONIC``
		表示从系统启动时刻开始计时的单调递增时间, 不受系统时间被用户修改的影响.

	* ``CLOCK_PROCESS_CPUTIME_ID``
		本进程到当前代码系统CPU花费的时间.

	* ``CLOCK_THREAD_CPUTIME_ID``
		本线程到当前代码系统CPU花费的时间.


``struct timeval``
==================

``struct timeval``\ 和\ ``struct timvespec``\ 一样, 也是用来表示一个time.

.. code-block:: c

	struct timeval {
		long tv_sec;	/* Seconds */
		long tv_usec;	/* Microseconds */
	}

``strcut timeval``\ 有两个成员, 一个秒和一个微秒.


.. note::

	``struct timespec``\ 和\ ``struct timeval``\ 都是用来表示一个时间, 它们的区别: **时间精度不一样**\ .
	``struct timespec``\ 有两个成员, 一个秒和一个纳秒; ``struct timeval``\ 也有两个成员, 一个秒和一个微秒.


Clock and time functions
========================

* **秒级精度的时间函数**

``time()``\ 提供了秒级的精确度.

.. code-block:: c

	#include <time.h>

	// 返回从UTC1970-1-1 0:0:0开始到现在的秒数
	time_t time(time_t * timer);

* **纳秒精度的时间函数**

.. code-block:: c

	#include <time.h>

	/* 这三个函数对应的时间结构体为struct timepsec */

	int clock_getres(clockid_t clk_id, struct timespec *res);
	int clock_gettime(clockid_t clk_id, struct timespec *tp);
	int clock_settime(clockid_t clk_id, const struct timespec *tp);

The function ``clock_getres()`` finds the resolution(precision) of the specified clock *clk_id*, and, if *res* is non-NULL, stores it in the struct timespec pointed to by *res*. 
The resolution of clocks depends on the implementation and cannot be configured by a particular process. 
If the time value pointed to by the argument *tp* of ``clock_settime()`` is not a multiple of res, then it is truncated to a multiple of *res*.

The function ``clock_gettime()`` and ``clock_settime`` retrieve and set the time of the specified clock *clk_id*.

* **微秒精度的时间函数**

.. code-block:: c

	#include <sys/time.h>

	/* 这两个函数对应的时间结构体为struct timeval */

	int gettimeofday(struct timeval *tv, struct timezone *tz);
	int settimeofday(const struct timeval *tv, const struct timezone *tz);

The functions ``gettimeofday()`` and ``settimeofday()`` can get and set the time as well as a timezone.

If either *tv* or *tz* is NULL, the corresponding structure is not set or returned.

The use of the ``timezone`` structure is obsolete; the *tz* argument should normally be specified as NULL.


**Example:**

.. code-block:: c

	#include <stdio.h>
	#include <time.h>
	#include <sys/time.h>

	void currenttime_ns()
	{
		printf("--------------------struct timespec--------------------\n");
		printf("[time(NULL)] : %ld\n", time(NULL));

		struct timespec ts;
		clock_gettime(CLOCK_REALTIME, &ts);
		printf("clock_gettime: tv_sec=%ld, tv_nsec=%ld\n", ts.tv_sec, ts.tv_nsec);
	}

	void currenttime_us()
	{
		printf("--------------------struct timeval--------------------\n");
		printf("[time(NULL)] : %ld\n", time(NULL));

		struct timeval tv;
		gettimeofday(&tv, NULL);
		printf("gettimeofday: tv_sec=%ld, tv_usec=%ld\n", tv.tv_sec, tv.tv_usec);
	}

	int main()
	{
		currenttime_ns();
		printf("\n");
		currenttime_us();
		printf("\n");

		return 0;
	}
