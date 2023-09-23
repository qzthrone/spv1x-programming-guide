SWM
======================

SWM模块为PWM的子模块，由软件模式和PDM模式组成

.. note::
  
  SWM模块与PWM模块的通道PWM0为同一通道

  当SWM模式选择差分或classD输出方式，将占用PWM1通道

简介
----------------------

.. c:enum:: swm_source_clk_t

    SWM通道工作时钟

    - *SWM_Clock_OSCPMU*: 32kHz时钟
    - *SWM_Clock_OSCAUDIO*: 49.152MHz时钟
    - *SWM_Clock_OSCCORE*: corepll时钟，大小由corepll配置决定，参看时钟配置模块 XX
    - *SWM_Clock_EXTCLK*: 外部输入时钟

:说明:
 
 1. SWM时钟与PWM0时钟为同一个
 2. SWM的时钟配置可以配置时钟的占空比，即时钟不以方波形式输出。具体参看时钟配置函数

.. c:enum:: swm_it_type_t

    SWM中断

    - *SWM_Fifo_Threshold_IT*: FIFO阈值中断
    - *SWM_Fifo_Overflow_IT*: FIFO上溢中断
    - *SWM_Fifo_Underflow_IT*: FIFO下溢中断
    - *SWM_Fifo_End_IT*: FIFO最后一数据输出中断

:说明:

 1. SWM_Fifo_Threshold_IT FIFO可写数据大于等于阈值pending，阈值由SMW初始参数data_threshold指定；SWM_Fifo_Overflow_IT FIFO数据上溢pending，即FIFO被填爆；SWM_Fifo_Underflow_IT FIFO下溢，即FIFO被读爆；SWM_Fifo_End_IT FIFO最后一个数据被发送pending
 2. pending的产生与中断使能无关，四个中断的服务函数入口相同，需要在中断服务函数中判断具体触发中断的pending

.. c:enum:: swm_data_width_t

    SWM的FIFO中有效数据的bit数，指定SWM模式FIFO中有效数据的bit数

    - *SWM_Data_Width_5Bit* 至 *SWM_Data_Width_16Bit*

.. c:enum:: swm_data_format_t

    FIFO的每个word中有效数据的个数

    - *SWM_Format_1Data*: 
    - *SWM_Format_2Data*: 
    - *SWM_Format_3Data*: 
    - *SWM_Format_4Data*: 

:说明:

    - 当设定每个word中有效数据为1个，数据存放在word的低16bit；
    - 设定每个word中有效数据为2个，数据分别存放在低16bit和高16bit；
    - 当设定word中有效数据为3个，数据分别存放在[9:0]、[19:10]、[29:20]，最后两bit无效；
    - 当设定word中由4个数据，分别存放在[7：0]、[15：8]、[23：16]、[31：24]。

.. c:enum:: swm_data_type_t

    与 *swm_data_width_t* 共同指定输出脉冲的周期

    - *SWM_Data_Hex*: 
    - *SWM_Data_Dec*: 

:说明:

    - 参数与swm_data_width_t指定输出脉冲的周期。当选择SWM_Data_Hex，有效数据bit5-16分别指定周期为32，64，128，256，512，1024，2048，4096，8192，16384，32768，65536；当选择SWM_Data_Dec，有效数据bit5-16分别指定周期为30，60，120，250，500，1000，2000，4000，8000，16000，32000，64000。单位为通道工作时钟。

.. c:enum:: swm_soft_mode_t

    swm输出模式

    - *SWM_Soft_Single_Mode*: 
    - *SWM_Soft_Overturn_Mode*: 
    - *SWM_Soft_Difference_Mode*: 
    - *SWM_Soft_ClassD_Mode*: 

:说明:

    - 参看功能及描述软件模式

.. c:enum:: swm_signal_format_t

    指定有效数据符号位形式

    - *SWM_Data_Signal_Magnitude*: 最高位符号位
    - *SWM_Data_Signal_Complement*:补码形式

:说明:

 1. SWM_Data_Signal_Magnitude表示最高位表示符号位。最高位不是有效数据的最高位，而是能表示最大数据的最高位即下图中各有效数据的最高位
    
    .. image:: ../../_static/kiwi-pwm-format.png
        :align: center
 
 2. SWM_Data_Signal_Complement表示补码形式表示符号

.. c:enum:: swm_idle_polarity_t

    指定空闲时，通道输出的极性

    - *SWM_Idle_Polarity_Low*: 空闲时极性低
    - *SWM_Idle_Polarity_High*: 空闲时极性高 

.. c:enum:: swm_parameter_t

    SWM初始化参数

    - *mode*: 输出模式，swm_soft_mode_t中选择
    - *data_width*: fifo数据宽度，swm_data_width_t中选择
    - *data_format*: fifo中数据格式，swm_data_format_t中选择
    - *data_repeat*: fifo中每个数据连续输出次数，范围1-16
    - *data_type*:  与data_width共同指定脉冲周期，swm_data_type_t中选择
    - *data_signal_format*: fifo数据符号位表示方式，swm_signal_format_t中选择
    - *continue_en*: fifo空时， 输出行为，参数选择Enable Disable
    - *modulate_en*: 脉冲输出是否加入时钟调制，参数选择Enable Disable 只影响底边，即PWM0通道对应的通道
    - *idle_polarity*: 空闲极性，swm_idle_polarity_t中选择
    - *data_threshold*: 产生SWM_Fifo_Threshold_IT阈值,fifo中还剩多少可写中断 1-8

:说明:

    - mode指定输出模式；
    - data_width指定fifo中有效数据的宽度；
    - data_format指定fifo中每个32bit中有效数据的个数；
    - data_repeat指定每个有效数据输出次数；
    - data_type指定输出脉冲的周期，与data_width共同决定；
    - data_signal_format指定数据符号格式；
    - continue_en指定fifo空时，输出行为，当值为Enable表示fifo空，继续输出最后一个32bit数据，当值为Disable表示fifo空输出空闲极性；
    - idle_polarity指定空闲时，通道输出的极性；modulate_en指定输出脉冲是否加上时钟调制输出；
    - data_threshold指定fifo可填数据的阈值，用于触发SWM_Fifo_Threshold_IT pending

.. c:function:: void swm_clock_set(swm_source_clk_t source_clk,uint32_t div,uint32_t low_duty)

    SWM通道时钟设置

    :param source_clk: source_clk可选时钟源，参数选swm_source_clk_t
    :param div: 时钟分频系数，范围1-8192
    :param low_duty: 时钟低电平占比数量，参数范围1-div
    :returns: 无

.. c:function:: void swm_init(swm_parameter_t *soft_parameter)

    SWM模式初始化

    :param soft_parameter: swm参数，参数范围swm_parameter_t
    :returns: 无

.. c:function:: void swm_pdm_init(soc_set_t idle_polarity)

    SWM通道PDM初始化

    :param idle_polarity: 初始化后，空闲极性
    :returns: 无

.. c:function:: void swm_write_data(uint32_t data)

    向SWM的fifo中写数据

    :param data: 需要写入的数据
    :returns: 无

.. c:function:: uint32_t swm_get_write_state()

    获取SWM的fifo状态

    :returns: FIFO状态
    :retval 0: 非空非满
    :retval 1: 满
    :retval 2: 空

.. c:function:: void swm_deinit()

    SWM的去初始化

    :returns: 无

.. c:function:: void swm_start()

    SWM的开始输出pwm

    :returns: 无
    
.. c:function:: void swm_abort()

    SWM停止输出pwm

    :returns: 无

.. c:function:: void swm_irq_enable(swm_it_type_t it_type)

    SWM中断使能

    :param it_type: 中断类型

        - *SWM_Fifo_Threshold_IT*: fifo阈值中断，触发阈值data_threshold控制
        - *SWM_Fifo_Overflow_IT*: fifo上溢中断
        - *SWM_Fifo_Underflow_IT*: fifo下溢中断
        - *SWM_Fifo_End_IT*: 出书一个PWM后中断

    :returns: 无

.. c:function:: void swm_irq_disable(swm_it_type_t it_type)

    SWM中断失能

    :param it_type: 中断类型

        - *SWM_Fifo_Threshold_IT*: fifo阈值中断，触发阈值data_threshold控制
        - *SWM_Fifo_Overflow_IT*: fifo上溢中断
        - *SWM_Fifo_Underflow_IT*: fifo下溢中断
        - *SWM_Fifo_End_IT*: 出书一个PWM后中断

    :returns: 无    

.. c:function:: soc_set_t swm_irq_get_flag(swm_it_type_t it_type)

    获取SWM中断pending

    :param it_type: 中断类型

        - *SWM_Fifo_Threshold_IT*: fifo阈值中断，触发阈值data_threshold控制
        - *SWM_Fifo_Overflow_IT*: fifo上溢中断
        - *SWM_Fifo_Underflow_IT*: fifo下溢中断
        - *SWM_Fifo_End_IT*: 出书一个PWM后中断

    :returns: 中断状态
    :retval Reset: 对应pending未置位
    :retval Set: 对应pending置位   

.. c:function:: void swm_irq_clear_flag(swm_it_type_t it_type)

    清除SWM中断pending

    :param it_type: 中断类型

        - *SWM_Fifo_Threshold_IT*: fifo阈值中断，触发阈值data_threshold控制
        - *SWM_Fifo_Overflow_IT*: fifo上溢中断
        - *SWM_Fifo_Underflow_IT*: fifo下溢中断
        - *SWM_Fifo_End_IT*: 出书一个PWM后中断

    :returns: 无

.. c:function::  void swm_irq_handler()

    SWM中断处理函数

    :retuans: 无
    :note: 需要在pwm_irq_entry中调用
    :note: 弱函数，用户可再定义同名函数

.. c:function:: void swm_tx_dma_enable()

    SWM通道DMA请求使能,指定DMA的请求源

    :retuans: 无



API使用
----------------------

CPU查询方式
^^^^^^^^^^^^^^^^^^^^^^

 1. 确定通道未被使用
 2. 调用swm_clock_set(source_clk, div,low_duty)通道工作时钟
 3. 调用swm_init(soft_parameter_addr)或swm_pdm_init()初始通道
 4. 调用swm_tx_dma_enable()使能dma搬运
 5. 调用DMA通道初始化
 6. 调用DMA通道使能，开搬运
 7. 调用swm_start()开始输出脉冲

 .. image:: ../../_static/kiwi-pwm-api-swm-cpu.jpg
  :align: center

dma搬运方式
^^^^^^^^^^^^^^^^^^^^^^

 1. 确定通道未被使用
 2. 调用swm_clock_set(source_clk, div,low_duty)通道工作时钟
 3. 调用swm_init(soft_parameter_addr)或swm_pdm_init()初始通道
 4. 调用swm_tx_dma_enable()使能dma搬运
 5. 调用DMA通道初始化
 6. 调用DMA通道使能，开搬运
 7. 调用swm_start()开始输出脉冲

 .. image:: ../../_static/kiwi-pwm-api-swm-dma.jpg
  :align: center