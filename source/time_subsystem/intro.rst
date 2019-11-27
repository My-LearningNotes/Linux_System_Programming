****
简介
****

Linux内核提供各种\ **time line(时间线)**\ , real time clock, monotonic clock, monotonic raw clock等.

* ``CLOCK_REALTIME``\ 类型的系统时钟

	.. note::

		其实就是墙上时钟(wall time).

时间就像是一条直线(line), 不知道起点, 也不知道终点, 因此我们称之为time line.
time line有很多种, 和如何定义0值的时间以及用什么样的刻度来度量时间相关.
人类熟悉的墙上时间和linux kernel中定义的CLOCK_REALTIME都是用来描述time line的, 只不过时间原点和如何度量time line上两点距离的刻度不一样.
对于人类的时间, 0值是耶稣诞生的时间; 对于CLOCK_REALTIME, 0值是linux epoch, 即1970年1月1日.
对于墙上时间, 在度量的时候虽然也是基于秒的, 但是人类做了grouping, 因此使用了年月日时分秒的概念.
对于linux世界中的CLOCK_REALTIME时间, 直接使用秒以及纳秒在当前秒内的偏移来表示.

* ``CLOCK_MONOTONIC``\ 类型的系统时钟.

monotonic的字面意思是单调, 实际上它指的是系统启动以后流逝的时间.




时间的计时单位
人类常用的时间计时单位是: 年月日时分秒等.
在计算机中, clock也有自己的计时单位; 计时的数值，加上单位才有意义.