******
内存锁
******

``mlock``, ``mlock2``, ``munlock``, ``mlockall``, ``munlockall`` - lock and unlock memory.


Synopsis
========

.. code-block:: c

	#include <sys/mman.h>

	int mlockall(int flags);
	int munlockall(void);


Description
===========

``mlockall()`` locks all pages mapped into the address space of the calling process.
This includes the pages of the code, data and stack segment, as well as shared libraries, user space kernel data, shared memory, and memory mapped files. 
All mapped pages are guaranteed to be resident in RAM when the call returns successfully; the pages are guaranteed to stay in RAM until later unlocked.

The *flags* argument is constructed as the bitwise OR of one or more of the following constants:

* ``MCL_CURRENT``
	表示对所有已经映射到进程地址空间的页上锁.

* ``MCL_FUTURE``
	表示对所有将来映射到进程地址空间的页都上锁.

``munlockall()`` unlocks all pages mapped into the address space of the calling process.

.. note::

	* 此函数有两个重要应用:

		* real-time algorithms(实时算法) - 对时间要求非常高
		* high-security data processing(机密数据的处理) - 如果数据被交换到外存上, 可能会泄密

	* 如果进程执行了一个\ ``execve``\ 类函数, 所有的锁都会被删除
	* 内存锁不会被子进程继承


Example
=======

.. code-block:: c
	:emphasize-lines: 5

	// main.cpp
	int main(int argc, char *argv[])
	{
  		// Keep the kernel from swapping us out
  		if (mlockall(MCL_CURRENT | MCL_FUTURE) < 0)
  		{
    			perror("Failed to lock memory.");
    			exit(EXIT_FAILURE);
  		}
  	}
