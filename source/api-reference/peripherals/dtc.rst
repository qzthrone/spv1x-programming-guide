DTC
======================

DTC模块为PWM子模块，设计用于电机控制 

.. note::
  
  SWM模块寄存器参看PWM寄存器 PD - DTC_CTL 


简介
----------------------

 - 模式为3对即共6路PWM输出通道构成，3对输出命名分别为U、V、W，由一个参数控制输出周期，由三个不同参数控制占空比，每对输出通道由H、L两路输出通道构成
 - DTC详情：PWM模块中对DTC的描述



.. c:enum:: dtc_source_clk_t

    DTC工作时钟

    - *DTC_Clock_OSCPMU*: 32kHz时钟
    - *DTC_Clock_OSCAUDIO*: 49.152MHz时钟
    - *DTC_Clock_OSCCORE*: corepll时钟，大小由corepll配置决定，参看时钟配置模块 XX
    - *DTC_Clock_EXTCLK*: 外部输入时钟

.. c:enum:: dtc_it_type_t

    DTC中断

    - *DTC_Half_IT*: 单个脉冲输出一半时中断
    - *DTC_End_IT*: 单个脉冲输出完成中断

:说明:

 1. DTC_End_IT 一个脉冲周期输出完毕pending
 2. DTC_Half_IT 一个脉冲输出一半pending，只有当输出为中心对称是有效，次参数在通道初始化参数指定

.. c:enum:: dct_align_type_t

    DTC上下边输出对齐方式

    - *DTC_Align_Type_Up_Right*: 右边上升沿对齐
    - *DTC_Align_Type_Center*: 中心对齐
    - *DTC_Align_Type_Down_Left*: 左边下降沿对齐

.. c:enum:: dct_channel_t

    DTC模式通路，由UVW三通道构成，每个通道由两路互补输出构成。即DTC模式总共有6路脉冲输出

    - *DTCW*: W通路
    - *DTCV*: V通路
    - *DTCU*: U通路

.. c:struct:: dtc_parameter_t

    DTC初始化参数

    - *period*: 周期，范围1-256
    - *w_duty*: w通道高电平数量，范围0-256
    - *v_duty*: v通道高电平数量，范围0-256
    - *u_duty*: u通道高电平数量，范围0-256
    - *low_idle_polarity*: 空闲时底边的极性,参数选择soc_set_t
    - *high_idle_pilarity*: 空闲时高边的极性,参数选择soc_set_t
    - *low_out_polarity*: 决定输出是否反向，也会影响空闲的极性,参数选择soc_set_t
    - *high_out_pilarity*: 决定输出是否反向，也会影响空闲的极性,参数选择soc_set_t
    - *aligned_type*: 高低边对齐方式，参数选择dct_align_type_t
    - *dead_time*: 高低边死区时间，cycles .参数范围0-255

:说明:

 1. 三对通道的占空比值不能大于周期之，即w_duty、v_duty、u_duty都不能大于period
 2. 空闲极性也受输出翻转的影响，如，当空闲极性为高，但输出反正也使能，空闲时会输出低。
 3. 死区时间指每对通道的高低边输出占空比的时间差。当占空比不足死区时间，则输出占空比为0的脉冲

.. c:function:: void dtc_clock_set(dtc_source_clk_t source_clk,uint32_t div)

    DRC模式时钟设置

    :param source_clk: source_clk可选时钟源，参数选dtc_source_clk_t
    :param div: 时钟分频系数，范围1-8192
    :returns: 无

.. c:function:: void dtc_init(dtc_parameter_t *dtc_parameter)

    DTC模式初始化

    :param dtc_parameter: dct参数，参数范围dtc_parameter_t
    :returns: 无

.. c:function:: void dtc_set(uint32_t u_duty,uint32_t v_duty,uint32_t w_duty)

    DTC模式占空比参数设置

    :param u_duty: u通道占空数 0-256
    :param v_duty: v通道占空数 0-256
    :param w_duty: w通道占空数 0-256
    :returns: 无

.. c:function:: void dtc_deinit()

    DTC模式去初始化

    :returns: 无

.. c:function:: void dtc_start(dct_channel_t DTCx)

    DTC开始输出pwm

    :param DTCx: dtc通道，参数范围dct_channel_t
    :returns: 无

.. c:function:: void dtc_abort(dct_channel_t DTCx)

    DTC停止输出pwm

    :param DTCx: dtc通道，参数范围dct_channel_t
    :returns: 无

.. c:function:: void dtc_brake_enable(soc_set_t pause_polarity,soc_set_t low_polarity,soc_set_t high_polarity)

    使能GPIO控制DTC暂停输出pwm功能,通过DTC_EMB_PIN控制暂停输出

    :param pause_polarity: 表示暂停的控制脚电平极性
    :param low_polarity: 低边暂停后输出
    :param high_polarity: 高边暂停收输出
    :returns: 无

.. c:function:: void dtc_brake_disable()

    失能GPIO控制DTC输出pwm功能

    :returns: 无    

.. c:function:: void dtc_irq_enable(dtc_it_type_t it_type)

    使能DTC中断

    :param it_type: 中断类型，参数选择dtc_it_type_t
     - *DTC_End_IT*: 一个PWM输出完毕时产生
     - *DTC_Half_IT*: 一个PWM输出的中间时产生，只有中心对称时有效，否则和DTC_End_IT一起到来

    :returns: 无    

.. c:function:: void dtc_irq_disable(dtc_it_type_t it_type)

    失能DTC中断

    :param it_type: 中断类型，参数选择dtc_it_type_t
     - *DTC_End_IT*: 一个PWM输出完毕时产生
     - *DTC_Half_IT*: 一个PWM输出的中间时产生，只有中心对称时有效，否则和DTC_End_IT一起到来

    :returns: 无    

.. c:function:: soc_set_t dtc_irq_get_flag(dtc_it_type_t it_type)

    获取DTC中断pending

    :param it_type: 中断类型，参数选择dtc_it_type_t
     - *DTC_End_IT*: 一个PWM输出完毕时产生
     - *DTC_Half_IT*: 一个PWM输出的中间时产生，只有中心对称时有效，否则和DTC_End_IT一起到来

    :returns: pending状态
    :retval Reset: pending未置位
    :retval Set: pending置位    


.. c:function:: void dtc_irq_clear_flag(dtc_it_type_t it_type)

    清除DTC中断pending

    :param it_type: 中断类型，参数选择dtc_it_type_t
     - *DTC_End_IT*: 一个PWM输出完毕时产生
     - *DTC_Half_IT*: 一个PWM输出的中间时产生，只有中心对称时有效，否则和DTC_End_IT一起到来

    :returns: 无   

.. c:function:: void dtc_irq_handler()

    DTC中断处理函数

    :returns: 无 
    :note: 需要在pwm_irq_entry中调用
    :note: 弱函数，用户可再定义同名函数  


API使用
----------------------

 1. 确认通道未使用
 2. 调用dtc_clock_set(source_clk,div)设定工作时钟
 3. 调用dtc_init(dtc_parameter_addr)参数初始化
 4. 调用dtc_start(DTCx)使能通道输出
 5. 根据情况调用dtc_brake_enable(pause_polarity,low_polarity,high_polarity)使能刹车功能
 6. 调用dtc_set(u_duty,v_duty,w_duty)修改占空比

 .. image:: ../../_static/kiwi-pwm-api-dtc.jpg
  :align: center
