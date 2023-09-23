用户程序引导和初始化
======================

.. _SPV1x SoC Memory Map说明: memory-map.html
.. _事件驱动型用户程序设计: user-fw-design.html

.. note::
   
   前置知识： `SPV1x SoC Memory Map说明`_ 和 `事件驱动型用户程序设计`_

BROM执行用户程序引导后，CPU PC跳转至L2-CACHED NOR FLASH首地址0x48000000开始执行已有的用户程序。
本章节将对用户程序正式进入事件监听前的早期引导和初始化流程进行描述：

 1. CPU PC通过0x48000000地址处定义的跳转指令，进入 ``_cpu_exec_entry`` 定义的用户程序入口开始顺序执行。
 2. 进行 ``global pointer(gp)`` 和 ``stack pointer(sp)`` 寄存器的初始化
 3. 配置同步异常处理函数入口管理 ``mtvec``
 4. 配置中断向量表入口管理 ``mtvt``
 5. 配置Non-maskable Interrupt(NMI)处理函数入口 ``mnvec``
 6. 初始化SRAM空间 .iram段
 7. 初始化SRAM空间 .data段
 8. 初始化SRAM空间 .bss段
 9. 调用函数 ``system_init()``, 包含：

  - 系统时钟配置
  - NORC时钟和工作模式配置

 10. 调用函数 ``event_kernel_init()``, 包含：

  - 消息队列 ``msg_queue()`` 初始化
  - 调用 ``app_init()`` 进行用户应用初始化
  - Time-tick服务配置和启动
  - 全局中断使能开启

 11. 调用函数 ``loop()``, 运行事件处理机制 ``event_handler()`` 和 常驻任务 ``always_run()``

``event_handler()`` 通过轮询方式查询消息队列 ``msg_queue`` 中是否存在待处理事件消息，
若存在，则按照消息进入队列的时间顺序，读取最早一条待处理事件内容。

读取事件后， ``event_handler()`` 将解析消息中的消息类别字段 ``msg.type``，并根据其内容，分支进入对应类别的处理流程。
具体的消息类别包括并不限于：按键动作、滴答定时、红外指令接收等。

在进入指定事件类别的处理流程后，event_handler()将进一步解析消息中的ID字段 ``msg.id``，并根据其内容，调用对应的处理函数。
例如根据滴答定时类别下的 *TICK_ID_10MS*、*TICK_ID_100MS* 和 *TICK_ID_1S* 等预设id，分别调用tick_handler_10ms()、
tick_handler_100ms() 和 tick_handler_1s() 进行响应。

.. warning:: 

 - 事件驱动型用户程序流程不设置传统意义下的main()

