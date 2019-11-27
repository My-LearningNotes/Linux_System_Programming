***************************************
类成员函数作为pthread_create函数的参数
***************************************

.. code-block:: cpp

	#include <pthread.h>

	int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
	                   void *(*start_routine) (void *), void *arg);

``pthread_create``\ 需要的\ ``start_routine``\ 参数类型为\ ``void* (*)(void*)``, 
**如果将类的成员函数作为其参数, 会出现类型不匹配的问题;
这是因为类的成员函数在经过编译器处理之后, 会变成带有\ ``this``\ 指针参数的全局函数, 所以类型是不匹配的.**

方案1: 使用类的静态成员函数
===========================

如果将\ ``start_routine``\ 定义为\ ``static``\ 类型的，那么编译器会将\ ``static``\ 形式的函数，转换成不带\ ``this``\ 指针的全局函数，
所以其类型可以与\ ``pthread_create``\ 需要的类型相匹配.
但是类的静态成员函数无法访问类的非静态成员, 不过这可以通过传递\ ``this``\ 指针解决这个问题.

.. code-block:: cpp
	:emphasize-lines: 12, 32, 33, 34, 35, 36, 42

	// myclass.hpp
	#include <pthread.h>

	class MyClass
	{
	public:
		// 在该成员函数中定义要执行的任务
		void run();

		// 在该静态函数中，通过指针调用成员函数run
		// 将该静态函数作为参数传递给pthread_create函数
		static void* start(void* ptr);

		void start_thread();

	public:
		pthread_t thread;
	};

	// myclass.cpp
	#include "myclass.h"

	#include <iostream>

	// 定义要执行的任务
	void MyClass::run()
	{
		while(1)
			std::cout << "hello, world" << std::endl;
	}

	void* MyClass::start(void* ptr)
	{
		MyClass* _ptr = (MyClass*)ptr;
		_ptr->run();  // 通过传入的指针，调用成员函数run
	}

	void MyClass::start_thread()
	{
		// 将类的静态函数作为参数传入给pthread_create函数
		// 并传入this指针
		pthread_create(&thread, NULL, &start, this);
	}

方案2: 将启动线程函数声明为模板函数
====================================

该方案的思路是: 
	定义一个模板函数, 在使用时将\ **类类型**\ 和\ **类成员函数类型**\ 作为参数传入给模板函数, 生成一个符合\ ``pthread_create``\ 原型要求的线程启动函数.

Example:

.. code-block:: cpp
	:emphasize-lines: 28, 29, 30, 31, 32, 33, 34, 44

	// myclass.hpp
	#include <pthread.h>

	class MyClass
	{
	public:
		void _RunThread();
	};

	// myclass.cpp
	#include "myclass.h"

	#include <iostream>

	void MyClass::_RunThread()
	{
		while(1)
			std::cout << "hello, world" << std::endl;
	}

	// main.cpp
	#include "myclass.h"

	#include <pthread.h>

	// 定义一个模板函数
	// 模板类型为类类型和类成员函数的类型
	template<typename TYPE, void (TYPE::*_RunThread)()>
	void* _thread_t(void* param)
	{
		TYPE* This = (TYPE*)param;
		This->_RunThread();
		return NULL;
	}

	int main()
	{
		pthread_t thread;

		MyClass my_class;

		// 向模板函数传入参数
		// 根据模板函数生成一个符合pthread_create原型要求的线程启动函数
		pthread_create(&thread, NULL, _thread_t<MyClass, &MyClass::_RunThread>, &my_class);
		pthread_join(thread, NULL);

		return 0;
	}