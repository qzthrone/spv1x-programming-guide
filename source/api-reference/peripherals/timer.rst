TIMER
======================

TIMER为多功能定时器，可对不同信号计数，达到特定功能

外设特性
----------------------

 1. 2个独立的16位计数器通道
 2. 每个计数器都支持计时，计数，捕获三种模式
 3. 通道独立的计数溢出中断与捕获中断服务函数入口（共四个中断服务函数入口）
 4. 每个通道都支持捕获值dma传输
 5. 支持脉冲捕获

功能描述及使用
-----------------------

 1. 定时器不同模式，是定时器的计数器对不同信号的计数
 2. 定时器满足下列条件之一，计数器的计数值将归零

  - 计数达到设定的溢出值
  - 捕获模式下，外部信号满足设定的边沿触发条件
  - 向TIMER_CTL.RESTART写1

工作时钟
^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-timer-clk.jpg
  :align: center

 - 两通道独立的时钟控制，由CMU模块寄存器TIMER0CLK和TIMER1CLK控制。
 - 当通道时钟使能，且CMU的CLKEN0寄存器的TIMER比特置位、RMU的RSTEN0寄存器的TIEMR比特置位，才可以对通道寄存器读写。
 - 注意时钟源选择时，选择的时钟是否打开，参看时钟控制模块 XX

计时模式（timer mode）
""""""""""""""""""""""""""""

 .. image:: ../../_static/kiwi-timer-mode-timer.png
  :align: center

 计时模式时计数器对TIMER通道的工作时钟经行计数，当计数值达到设定的溢出值，产生pending，由于时钟频率固定，故可达到定时的效果。通过设定计数溢出值，可计算产生pending的时间间隔。计时时间计算公式：

    .. image:: ../../_static/kiwi-timer-formula-timer.jpg
        :align: center

    - *T_ms*: 定时间隔，单位ms 
    - *Timer_clk*: TIEMR通道的工作时钟，单位赫兹（Hz），寄存器CMU_TIMERnCLK中设定
    - *peidiv*: TIMER通道计数分频值，范围0-31，寄存器TIMER_CTL的PERDIVL比特域设定
    - *Timer_len*: TIMER通道设定的计数溢出值个数，TIEMR_LEN寄存器设定，范围1-65535

 计时模式也可由外部信号控制：可只在输入信号位高时计数，或可只在输入信号为低时计数

 **使用**

 **1. 使能TIMER模块，并配置通道时钟**

    - 置位CMU_CLKEN0.TIMER，开启TIMER时钟使能；
    - 置位RMU_RSTEN0.TIMER，释放TIMER模块；
    - 配置CMU_TIMERnCLK寄存器，选择通道工作时钟源，并打开通道工作时钟。

 **2. 配置TIMER_CTL寄存器，设置通道工作参数**

    - TIMER_CTL.MODE 设定通道工作模式，值设为0，意为计时模式；
    - TIMER_CTL.DIRECTION设定计数方式，0表示通道计数从零递增计数到TIMER_LEN的值，产生计数溢出pending；1表示从TIEMR_LEN的值递减计数到0，产生计数溢出pending；
    - TIMER_CTL.EDGE 设定外部信号控制计数器计数方式，根据实际选择，值为0表示一直计数，不受外部信号影响，值为1表示当外部信号为高时计数，值为2时表示当外部信号为低时计数，值为3时表示停止计数；
    - TIMER_CTL.GPIO_SRC 设定外部信号的输入的GPIO引脚号。值为0-28表示信号从GPIO0-GPIO28输入，值为29、30表示timer0和timer1的信号；
    - TIMER_CTL.DEBOUNCE 设定输入信号是否进行消抖延时，单位为通道工作时钟个数，值为0时表示不延时，1到7为2的1到7次方的时钟个数；
    - TIMER_CTL.PREDIVL 设定计数器对时钟计数的分频值，实际分频数为设定数加一；
    - TIMER_CTL.RELOAD_EN 设定计数值溢出后，是否继续重新计数。0表示计数值溢出后，停止；1表示计数值溢出后，继续重新计数；
    - TIMER_CTL其它值在此模式设为0即可；

 **3. 设置计数溢出值**
    
    - TIMER_LEN.LENGTH 设定计数溢出值，范围1-65535，当计数值达到此值产生溢出pending；

 **4. 清除PD寄存器**
  
    - 清除TIMER_PD寄存器对应通道的pending，TIMER_PD.TIMER0表示通道0计数溢出pending，TIMER_PD.TIMER1表示通道1计数溢出pending；

 **5. 配置中断IE**

    - 根据使用情况使能TIMER_IE寄存器，TIMER_IE.TIMER0表示通道0溢出中断，TIMER_IE.TIMER1表示通道1溢出中断。中断服务函数分别为timer0_irq_entry()和timer1_irq_entry()；

 **6. 使能通道，开始计数**

    - TIMER_CTL.RUN置位，开始计数；

计数模式（counter mode）
""""""""""""""""""""""""""""

 .. image:: ../../_static/kiwi-timer-mode-counter.png
  :align: center

 - 计数模式是计数器对外部输入信号的电平边沿进行计数，当计数值达到设定值，产生pending，可对单独对上升沿，下降沿，双边沿计数。
 - 计数模式的输入信号电平变化频率大于通道的工作时钟，对输入信号的计数会出错，此时应该提高通道的工作时钟。
 
 **使用**

 **1. 使能TIMER模块，并配置通道时钟**

    - 置位CMU_CLKEN0.TIMER，使能TIMER时钟
    - 置位RMU_RSTEN0.TIMER，释放TIMER模块
    - 配置CMU_TIMERnCLK寄存器，选择通道工作时钟源，并打开通道工作时钟。
 
 **2. 配置TIMER_CTL寄存器，设置计数器工作参数**

    - TIMER_CTL.MODE 设定通道工作模式，值设为1，意为计数模式
    - TIMER_CTL.DIRECTION设定计数方式，0表示通道计数器从零递增计数到TIMER_LEN的值，1表示从TIMER_LEN的值递减计数到0
    - TIMER_CTL.EDGE 设定外部信号控制计数器计数方式，根据实际选择，值为0表示计数外部信号的下降沿，值为1表示计数外部信号的上升沿，值为2时表示计数外部信号的上升沿和下降沿。
    - TIMER_CTL.GPIO_SRC 设定外部信号的输入的GPIO引脚号。值为0-28表示信号从GPIO0-GPIO28输入，值为29、30表示timer0和timer1的信号
    - TIMER_CTL.DEBOUNCE 设定输入信号是否进行消抖延时，单位为通道工作时钟个数，值为0时表示不延时，1到7为2的1到7次方的时钟个数
    - TIMER_CTL.PREDIVL 设定计数器对电平边沿计数的分频值，实际分频数为设定数加一，即实际对电平边沿计数时，当有PREDIVL 设定值个数的电平变化时，VAL计数一次
    - TIMER_CTL.RELOAD_EN 设定计数器计数值溢出后，是否继续重新计数。0表示计数值溢出后，停止；1表示计数值溢出后，继续重新计数
    - TIMER_CTL其它值在此模式设为0即可
 
 **3. 设置计数溢出值**

    - TIMER_LEN.LENGTH 设定计数溢出值，范围1-65535，当计数值达到此值产生溢出pending

 **4. 清除PD寄存器**

    - 清除TIMER_PD寄存器对应通道的pending，TIMER_PD.TIMER0表示通道0计数溢出pending，TIMER_PD.TIMER1表示通道1计数溢出pending

 **5. 配置中断IE**

    - 根据使用情况使能TIMER_IE寄存器，TIMER_IE.TIMER0表示通道0溢出中断，TIMER_IE.TIMER1表示通道1溢出中断。中断服务函数分别为timer0_irq_entry()和timer1_irq_entry()

 **6. 使能通道，开始计数**

    - TIMER_CTL.RUN置位，开始计数

捕获模式（capture mode）
""""""""""""""""""""""""""""

 .. image:: ../../_static/kiwi-timer-mode-capture.png
  :align: center

 - 捕获模式是计数器对定时器通道工作时钟计数，计数的开始和停止由外部信号的边沿控制。
 - 当外部信号产生捕获边沿事件，将产生捕获pending，并将当前计数值放入CAP寄存器中。由于时钟频率固定，故可达到捕获电平边沿之间时间的功能
 - 捕获时间计算公式：

    .. image:: ../../_static/kiwi-timer-formula-capture.jpg
        :align: center

    - *C_ms*: 捕获的电平沿之间的时间，单位毫秒ms 
    - *Timer_clk*: TIMER通道的工作时钟，单位赫兹Hz，寄存器CMU_TIMERnCLK中设定
    - *perdiv*: TIMER通道设定分频值，范围0-31，寄存器TIMER_CTL的PERDIVL比特域设定
    - *Timer_cap*: TIMER通道捕获的计数值，TIMER_CAP寄存器获取

 **捕获模式的特殊情况**
    
    - 由于计数器只有16比特，加上32分频，也会较大可能存在电平边沿之间时长超过计数器能表示的最大时间。在这种情况下，必须同时使用捕获pending和定时pending来得到正确的捕获时间。
    - 当捕获模式极性间的计数时间，小于溢出设定值对应时间，溢出值对应的pending将不会触发，因为在捕获时间到达，会清零计数值，从而不溢出’

 **使用**

 **1. 使能TIMER模块，并配置通道时钟**

    - 置位CMU_CLKEN0.TIMER，使能TIMER时钟
    - 置位RMU_RSTEN0.TIMER，释放TIMER模块
    - 配置CMU_TIMERnCLK寄存器，选择通道工作时钟源，并打开通道工作时钟

 **2. 配置TIMER_CTL寄存器，是定计数器工作模式**

    - TIMER_CTL.MODE 设定通道工作模式，值设为2，意为捕获模式
    - TIMER_CTL.DIRECTION设定计数方式，0表示通道计数器从零递增计数到TIMER_LEN的值，1表示从TIMER_LEN的值递减计数到0。推荐使用递增计数，否者递减计数需要用TIMER_LEN的值减去捕获值才是真正的捕获计数值
    - TIMER_CTL.EDGE 设定外部信号触发计数器方式，根据实际选择，值为0表示外部信号为上升沿时将计数值放入TIMER_CAP中，产生捕获pending并归零重新计数；值为1表示外部信号为下降沿时将计数值放入TIMER_CAP中，产生捕获pending并归零重新计数；值为2表示外部信号为上升沿和下降沿沿时将计数值放入TIMER_CAP中，产生捕获pending并归零重新计数。值为3表示上升沿/下降沿开始计数，下降沿/上升沿时将值放入TIMER_CAP中，产生捕获pending并停止计数，并使TIMER_CTL.CAP_POL比特有效
    - TIMER_CTL.CAP_POL 当TIMER_CTL.EDGE为3，此比特有效，用于指定开始计数沿。当值为0，表示上升沿开始计数，下降沿停止计数，并将计数值放入TIMER_CAP中，产生捕获pending。当值为1，表示下降沿开始计数，上升沿停止计数，并将计数值放入TIMER_CAP中，产生捕获pending
    - TIMER_CTL.GPIO_SRC 设定外部信号的输入的GPIO引脚号。值为0-28表示信号从GPIO0-GPIO28输入，值为29、30表示timer0和timer1的信号
    - TIMER_CTL.PREDIVL 设定计数器对时钟计数的分频值，实际分频数为设定数加一
    - TIMER_CTL.DEBOUNCE 设定输入信号是否进行消抖延时，单位为通道工作时钟个数，值为0时表示不延时，1到7为2的1到7次方的时钟个数
    - TIMER_CTL.CAP_FMT 设定捕获寄存器最高位是否表示极性，值为0表示最高位不表示极性，值为1表示最高位表示极性
    - TIMER_CTL.CAP_START 设定第一次捕获模式的开始，值为0表示使能RUN后立刻开始第一次的捕获计数，值为1表示使能RUN后，需等到第一捕获时间到达，才开始第一次捕获计数
    - TIMER_CTL.RELOAD_EN和 TIMER_CTL.CAP_SINGLE 设定计数器计数值溢出后，是否继续重新计数。RELOAD_EN为0同时CAP_SINGLE为0表示计数值溢出后，停止；RELOAD_EN为0同时CAP_SINGLE为1表示满足捕获条件，停止，RELOAD_EN为1同时CAP_SINGLE为0表示计数值溢出后，继续重新计数
    - TIMER_CTL其它值在此模式设为0即可

 **3. 设置计数溢出值**

    - TIMER_LEN.LENGTH 设定计数溢出值，范围1-65535。当计数值达到此值产生溢出pending

 **4. 清除PD寄存器**

    - 清除TIMER_PD寄存器对应通道的pending，TIMER_PD.TIMER0表示通道0计数溢出pending，TIMER_PD.TIMER1表示通道1计数溢出pending。TIMER_PD.CAPTURE0表示通道0捕获pending，TIMER_PD.CAPTURE1表示通道1捕获pending

 **5. 配置中断IE**

    - 根据使用情况使能TIMER_IE寄存器，TIMER_IE.TIMER0表示通道0溢出中断，TIMER_IE.TIMER1表示通道1溢出中断。中断服务函数分别为timer0_irq_entry()和提timer1_irq_entry()。TIMER_IE.CAPTURE0表示通道0捕获中断，TIMER_IE.CAPTURE1表示通道1捕获中断，中断服务函数分别为capture0_irq_entry()和capture1_irq_entry()

 **6. 使能通道，开始计数**
    
    - TIMER_CTL.RUN置位，开始计数

 **7. 读取  TIMRE_CAP值获取捕获值**

    - 当捕获pending到达，读取捕获值，若在捕获pending之前通道溢出pending也置位，捕获值需要再加上TIMER_LEN乘以溢出pending产生次数

捕获模式与计时模式的异同
""""""""""""""""""""""""""""

 **同**

    - 计数器都是对通道时钟计数
    - 计数值达到LEN寄存器的设定值时都会产生pending，并计数值归零再计数

 **异**
    
    - 计时模式的计数器是在信号为某电平时才计数工作时钟。捕获模式是有电平沿才开始对时钟计数，直至电平沿又发生指定变化是，将电平沿发生变化时的计数值放入TIEMR_CAP寄存器中，同时产生捕获pending，并计数值归零再计数
    - 当信号电平发生变化时，计数模式不会产生pending，而捕获模式会产生捕获Pending

暂停功能
^^^^^^^^^^^^^^^^^^^^^^^^^^^

    - 当TIMER_CTL.PAUSE置位，计数器暂停计数。TIMER_CTL.PAUSE复位，接着暂停的计数值计数
    - 注意，当为捕获模式时，计数器会暂停计数，但外部信号发生电平变化，且变化是捕获事件，则会产生捕获pending

重计数功能
^^^^^^^^^^^^^^^^^^^^^^^^^^^

    - 当向写TIMER_CTL.RESTART比特写1，计数器会清零，重新计数。

注意
----------------------------

 - 当TIMER正在工作，即通道TIMER_CTL.RUN比特置位时，不要修改 CTL，LEN寄存器的值，否则修改不但不会生效，计数还会产生错误，只有当计数停下，即TIMER_CTL.RUN复位，才可修改他们的值

API说明
----------------------------

 定时器的控制参数并非都可以再API中配置，API不可配置将使用固定值

  - 技术方向都使用自增计数
  - 捕获寄存器的最高位表示极性

简介
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. c:enum:: timer_source_clk_t

    TIMER通道时钟源

    - *TIMER_Clock_SYSCLK*: 系统时钟
    - *TIMER_Clock_OSCPMU*: 32kHz
    - *TIMER_Clock_OSCAUDIO*: 49.152MHz
    - *TIMER_Clock_OSCCORE*: corepll时钟
    - *TIMER_Clock_EXTCLK*: 外部输入时钟

.. c:enum:: timer_mode_t

    TIMER通道工作模式

    - *TIMER_Timer_Single_Mode*: 单次计时模式
    - *TIMER_Timer_Circular_Mode*: 循环计时模式
    - *TIMER_Counter_Single_Mode*: 单次计数模式
    - *TIMER_Counter_Circular_Mode*: 循环计数模式
    - *TIMER_Capture_Single_Mode*: 单次捕获模式
    - *TIMER_Capture_Circular_Mode*: 循环捕获模式

:说明:

1. 单次模式是指，计数溢出pending，或达到捕获pending后，停止计数；
2. 循环模式是指，计数溢出pending，或达到捕获pending后，计数值归零，重新计数，如此往复，直至软件复位TIEMR_CTL.RUN，及调用函数timer_abort(chx)
3. 计时，计数，捕获模式详细看功能 **功能描述及使用** 

.. c:enum:: timer_trigger_t

    TIMER模式外部信号触发

    计数模式和捕获模式时可选择

    - *TIMER_Rise_Edge*: 上升沿
    - *TIMER_Fall_Edge*: 下降沿
    - *TIMER_Both_Edge*: 双边沿
    - *TIMER_Rise_Fall*: 双边沿之 上升沿到下降沿，只有捕获模式有效
    - *TIMER_Fall_Rise*: 双边沿之 下降沿到上升沿，只有捕获模式有效

    计时模式时选择

    - *TIMER_Free_Run*: 不受触发信号影响计时
    - *TIMER_High_Level*: 触发信号高时计时
    - *TIMER_Low_Level*: 触发信号低时计时

:说明:

1. IMER_Rise_Fall和TIMER_Fall_Rise只有捕获模式有效
2. 在计数模式，事件可选TIMER_Rise_Edge、TIMER_Fall_Edge、TIMER_Both_Edge即上升沿、下降沿、双边沿触发一次计数。
3. 在捕获模式，可选TIMER_Rise_Edge，TIMER_Fall_Edge、TIMER_Both_Edge、TIMER_Rise_Fall、TIMER_Fall_Rise即，上升沿、下降沿、双边沿触发捕获，TIMER_Rise_Fall表示捕获上升沿到下降沿、TIMER_Fall_Rise表示捕获下降沿到上升沿。
4. 在计时模式，可选TIMER_Free_Run、TIMER_High_Level、TIMER_Low_Level 即不受信号控制一直计数，高电平计数，下降沿计数。

.. c:enum:: timer_debounce_t

    输入信号消抖时间

    - *TIMER_Debounce_0CYCLE*: 
    - *TIMER_Debounce_2CYCLE*: 
    - *TIMER_Debounce_4CYCLE*: 
    - *TIMER_Debounce_8CYCLE*: 
    - *TIMER_Debounce_16CYCLE*: 
    - *TIMER_Debounce_32CYCLE*: 
    - *TIMER_Debounce_64CYCLE*: 
    - *TIMER_Debounce_128CYCLE*: 

:说明:

1. 消抖的时间单位为通道工作时钟频率
2. 当电平变化小于消抖时间，消抖时间内电平的变化将被忽略

.. c:enum:: timer_it_type_t

    中断类型

    - *TIMER_Overflow_IT*: 
    - *TIMER_Capture_IT*: 

:说明:

1. TIMER_Overflow_IT指三种模式下，计数器计数值达到设定计数值产生的中断
2. TIMER_Capture_IT 指捕获模式下，外部输入信号满足触发条件产生的中断
3. 无论中断是否使能，都不影响pending的产生，只影响中断服务函数的进入
4. 通过判断TIMER_Capture_IT的pending是否置位，判断捕获事件是否发生
5. 同一通道的TIMER_Overflow_IT中断服务函数timerX_irq_entry()，TIMER_Capture_IT为另一中断服务函数captureX_irq_entry()。不同通道的服务函数不为同一个。
6. 在捕获模式中，若捕获事件之间的时间小于设定溢出值对应的时间，将不会触发溢出pending，因为在捕获事件时就将计数值清零

.. c:enum:: timer_capture_start_t

    捕获模式第一次启动计数方式

    - *TIMER_Capture_Start_Software*: 调用timer_start()捕获计数开始
    - *TIMER_Capture_Start_Event*: 调用timer_start()后，第一个触发事件后捕获计数开始

:说明:

1. 只在捕获模式有效，指定捕获模式第一次开始的实际。

.. c:enum:: timer_init_parameter_t

    定时器通道初始化参数

    - *mode*: TIMER工作模式，参数timer_mode_t
    - *trigger_pin*: 触发信号输入脚，范围0-31，其中29 timer0_run,30 timer1_run,31 dbio_o[0]
    - *trigger_pin_debounce*: 触发信号输入脚debounce时间，参数timer_debounce_t
    - *trigger_type*: 触发信号方式，参数timer_trigger_t
    - *pridiv*: 触发信号分频，参数1-32
    - *counter*: 计数溢出值1-0xffff

:说明:

1. counter指定计数器计数到设定值，产生溢出pending，在三种模式都有效
2. pridiv指定计数器计数分频，即当计数事件次数达到分频设定值，计数值才加一

.. c:enum:: timer_state_t

    TIMER运行状态

    - *TIMER_State_Inited*: 初始状态，timer_init()或timer_abort()函数进入该状态
    - *TIMER_State_Reset*: 复位状态，timer_deinit()函数进入该函数
    - *TIMER_State_Running*: 运行状态，timer_start(),timer_restart()或timer_resume()函数进入该状态
    - *TIMER_State_Paused*: 暂停状态，dma_pause()函数进入该状态

:说明:

1. TIMER通道运行状态由四种状态组成，每个状态由特定函数进入，在指定状态只能调用指定函数，否则调用无效，甚至使搬运出错。影响状态函数共6个，各状态下状态函数调用说明

 .. image:: ../../_static/kiwi-timer-api-state.jpg
  :align: center

 .. image:: ../../_static/kiwi-timer-api-state-t.png
  :align: center



.. c:function:: void timer_clock_set(uint32_t chx,timer_source_clk_t source_clk)

    TIMER通道时钟设置

    :param chx: TIMER通道号，参数范围0-1
    :param source_clk: source_clk可选时钟源，参数选timer_source_clk_t
    :returns: 无

.. c:function:: void timer_init(uint32_t chx,timer_init_parameter_t *timer_init_param)

    根据timer_init_parameter_t 参数初始化通道

    :param chx: TIMER通道号，参数范围0-1
    :param timer_init_param: dma_init_parameter_t结构体指针
    :returns: 无
    :note: 调用该函数后，TIEMR通道状态进入初始态TIMER_State_Inited

.. c:function:: void timer_deinit(uint32_t chx)

    TIMER通道去初始化

    :param chx: TIMER通道号，参数范围0-1
    :returns: 无
    :note: 调用该函数后，TIMER通道状态进入复位态TIMER_State_Reset

.. c:function:: void timer_start(uint32_t chx)

    TIMNER通道开始运行

    :param chx: TIMER通道号，参数范围0-1
    :returns: 无
    :note: 调用该函数后，TIMER通道状态进入运行态TIMER_State_Running

.. c:function:: void timer_restart(uint32_t chx)

    TIMER通道重新开始计数

    :param chx: TIMER通道号，参数范围0-1
    :returns: 无
    :note: 调用该函数后，状态不变，计数值清零

.. c:function:: void timer_pause(uint32_t chx)

    TIMER通道停止运行

    :param chx: TIMER通道号，参数范围0-1
    :returns: 无
    :note: 调用该函数后，TIMER通道状态进入暂停态TIMER_State_Paused
    :note: 捕获模式下暂停，当捕获事件到达，仍会产生捕获pending，但捕获值为暂停时的值

.. c:function:: void timer_resume(uint32_t chx)

    TIMER通道恢复运行，恢复由timer_pause()暂停的通道

    :param chx: TIMER通道号，参数范围0-1
    :returns: 无
    :note: 调用该函数后，TIMER通道状态进入运行态TIMER_State_Running

.. c:function:: void timer_abort(uint32_t chx)

    TIMER通道运行终止

    :param chx: TIMER通道号，参数范围0-1
    :returns: 无
    :note: 调用该函数后，TIMER通道状态进入初始态TIMER_State_Inited

.. c:function:: timer_state_t timer_get_state(uint32_t chx)

    获取TIMER通道状态

    :param chx: TIMER通道号，参数范围0-1
    :returns: 通道当前状态
    :retval: timer_state_t
    :note: 详细参看 `timer_state_t` 说明

.. c:function:: void timer_set_counter(uint32_t chx,uint32_t counter)

    设置TIMER通道技术溢出值

    :param chx: TIMER通道号，参数范围0-1
    :param counter: 计数溢出值，范围0xffff
    :returns: 无

.. c:function:: uint32_t timer_get_counter(uint32_t chx)

    获取TIMER通道设置计数溢出值

    :param chx: TIMER通道号，参数范围0-1
    :returns: 计数溢出值
    :retval: uint32_t

.. c:function:: uint32_t timer_get_capture(uint32_t chx)

    获取TIMER通道当前捕获值

    :param chx: TIMER通道号，参数范围0-1
    :returns: 当前补捕获值
    :retval: uint32_t
    :note: 在通道对应的捕获pending即寄存器PD.CaptureX比特置位时更新

.. c:function:: void timer_capture_set_start_type(uint32_t chx,timer_capture_start_t start_type)

    设置TIMER通道捕获模式开始方式

    :param chx: TIMER通道号，参数范围0-1
    :param tart_type: 开始捕获方式，参数timer_capture_start_t

        - *TIMER_Capture_Start_Software*:  当timer_start调用，捕获计数开始
        - *TIMER_Capture_Start_Event*: 当timer_start调用后，等到第一个捕获事件达到开始计数

    :returns: 无
    :note: 只有捕获模式有效

.. c:function:: void timer_irq_enable(uint32_t chx,timer_it_type_t it_type)

    TIMER通道中断使能

    :param chx: TIMER通道号，参数范围0-1
    :param it_type: 中断类型

        - *TIMER_Overflow_IT*: 计数值溢出中断
        - *TIMER_Capture_IT*: 捕获模式事件发生中断
    
    :returns: 无
    :note: 中断使能只影响中断服务函数进入，不影响pending的产生

.. c:function:: void timer_irq_disable(uint32_t chx,timer_it_type_t it_type)

    TIMER通道中断失能

    :param chx: TIMER通道号，参数范围0-1
    :param it_type: 中断类型

        - *TIMER_Overflow_IT*: 计数值溢出中断
        - *TIMER_Capture_IT*: 捕获模式事件发生中断
    
    :returns: 无
    :note: 中断失能只影响中断服务函数进入，不影响pending的产生

.. c:function:: soc_set_t timer_irq_get_flag(uint32_t chx,timer_it_type_t it_type)

    获取TIMER通道中断pending状态

    :param chx: TIMER通道号，参数范围0-1
    :param it_type: 中断类型

        - *TIMER_Overflow_IT*: 计数值溢出中断
        - *TIMER_Capture_IT*: 捕获模式事件发生中断
    
    :returns: 中断状态
    :retval Set: 对应中断pending置位
    :retval Reset: 对用中断pending未置位

.. c:function:: void timer_irq_clear_flag(uint32_t chx,timer_it_type_t it_type)

    TIMER通道中断pending状态清除

    :param chx: TIMER通道号，参数范围0-1
    :param it_type: 中断类型

        - *TIMER_Overflow_IT*: 计数值溢出中断
        - *TIMER_Capture_IT*: 捕获模式事件发生中断

    :returns: 无

.. c:function:: void timer_capture_dma_enable(uint32_t chx)

    TIMER通道捕获模式DMA请求使能，指定DMA的请求源

    :param chx: TIMER通道号，参数范围0-1
    :returns: 无

使用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

中断方式
"""""""""""""""""""""""""""""""""""""""

 1. 调用函数timer_get_state(chx)确定timer通道是否在运行，若在运行，调用timer_abort(chx)停止通道或更换通道；
 2. 调用timer_clock_set(chx,source_clk)设置通道工作时钟
 3. 调用timer_deinit(chx)去初始化，主要为了清除上一次通道为清除的pending和关闭中断。再调用timer_init(chx,param_addr)初始化通道
 4. 调用函数timer_irq_enable(chx,intype)使能中断
 5. 调用函数timer_start(chx)启动计数
 6. 中断服务函数中，调用timer_irq_get_flag(chx,it_type)确定对应通道中断的产生
 7. 若对应中断置位，调用函数timer_irq_chear_flag(chx,it_type),清除pending，否则中断服务函数退出后会立刻再次进入。

  .. image:: ../../_static/kiwi-timer-api-use.jpg
   :align: center
 
寄存器定义
----------------------------------

CLk
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-timer-clk.png
   :align: center

IE
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-timer-ie.png
   :align: center

PD
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-timer-pd.png
   :align: center

CTL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-timer-ctl-1.png
   :align: center
   :width: 762px
 .. image:: ../../_static/kiwi-reg-timer-ctl-2.png
   :align: center
   :width: 762px

LEN
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-timer-len.png
   :align: center

VAL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-timer-val.png
   :align: center

CAP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-timer-cap.png
   :align: center
