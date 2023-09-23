DMA
======================

DMA（Direct Memory Access）即存储器直接访问，是指一种高速的数据传输操作机制，允许在外部设备和存储器之间直接读写数据。

.. note::
  
  SPV1x SDK 目前没有开放DMA链表模式的使用指南。

外设特性
----------------------

 .. image:: ../../_static/kiwi-dma-frame-struct.jpg
  :align: center
  :width: 500px

 1. 具有串行的4个通道
 2. 可实现内存到内存，内存到外设，外设到内存，外设到外设传输方式
 3. 单次最大传输1MB
 4. 单次传输数据宽度可按8bit，16bit，32bit选择
 5. 支持正常模式(Linear Mode)和链表模式(Pseudo Linked-list Mode)两种传输方式
 6. 具有暂停(Pause)功能
 7. 支持通道burst优先级配置
 8. 支持8个外设设备

工作时钟
----------------------

  .. image:: ../../_static/kiwi-dma-clk.jpg
    :align: center

  - DMA与系统时钟SYS_CLK同频
  - 系统时钟配置，由CMU模块SYS_CLK寄存器配置
  - 修改系统时钟需要注意，注意系统时钟不仅DMA使用，其它模块也会使用
  - 系统时钟的配置可以使用时钟控制API，见时钟控制模块 XX

工作模式
----------------------

正常模式
^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-dma-normal-mode.png
  :align: center

 - DMA可以设置byte、half-word、word为 ``搬运单位(element)`` ，最终搬运数据数量为搬运单位的整数倍。
   搬运单位的选择由CFG.WIDTH的配置决定。
 - element紧凑排列形成一个 ``搬运槽(slot)`` ，slot自身大小由SRC.SLOT_OFFSET和DST.SLOT_OFFSET分别决定，最大可设置为64kBytes。
 - 一个或多个slot紧凑排列，形成一个完整的 ``搬运区域(region)`` 。
 - DMA从单一slot中搬运的数据总量(字节数)由LEN.BYTE_NUM进行配置。
 - DMA从region中搬运的slot总数由LEN.SLOT_NUM进行配置，最大为16。

外设使用
-----------------------

正常模式配置
^^^^^^^^^^^^^^^^^^^^^^^

 **1. 使能DMA模块并使能时钟**
   
   - 置位CMU_CLKEN0.DMA，开启DMA时钟信号。
   - 置位RMU_RSTEN0.DMA，将DMA模块从复位状态释放。

 **2. 配置源数据格式**

   - 配置SSA寄存器，将源数据空间地址赋予SSA。
   - 配置SOA寄存器，将源数据slot大小(字节数)赋予SRC_SLOT_OFFSET:
  
   .. math::  SOA\ slot\ size = 64 * (SRC\_SLOT\_OFFSET + 1)  

   - 配置CFG.SRC_SLV 选择SSA地址空间属于内存还是外设，置1表示外设，置0表示内存。
   - 配置CFG.SRC_INC 选择地址空间是否自增，置1表示自增，自增距离为CFG.WIDTH，置0表示固定，当地址固定不增长，槽设定无效。
   - 当SRC_SLV选择外设，CFG.SRC_IFS需要指定对应外设，对应见IFS描述，SRC_SLV选择内存时，CFG.SRC_IFS值无效。
  
 **3. 配置目的数据格式**

   - 配置DSA寄存器，将目的数据空间地址赋予DSA。
   - 配置DOA寄存器，将目的数据slot大小(字节数)赋予DST_SLOT_OFFSET:

   .. math::  DOA\ slot\ size = 64 * (DST\_SLOT\_OFFSET + 1)  

   - 配置CFG.DST_SLV 选择DSA地址空间属于内存还是外设，置1表示外设，置0表示内存。
   - 配置CFG.DST_INC 选择地址空间是否自增，置1表示自增，自增距离为CFG.WIDTH，置0表示固定，当地址固定不增长，槽设定无效。
   - 当DST_SLV选择外设，CFG.DST_IFS需要指定对应外设，对应见IFS描述，DST_SLV选择内存时，CFG.DST_IFS值无效。

 **4. 配置element尺寸**

   配置CFG.WIDTH, 指定element的尺寸。

 **5. 配置实际搬运数据量**

   - 配置LEN.SLOT_NUM，将slot数量减一，写入LEN.SLOT_NUM。
   - 配置LEN.BYTE_NUM，将每个slot实际搬运数据量（字节数）减1，写入LEN.BYTE_NUM。

 **6. 选择优先级**

   CFG.PRIO配置优先级，0-3，值越大，优先级越高，当DMA通道同时被发起搬数请求，优先级高的先搬运。

 **7. 配置重载**

   - 若CFG.RELOAD置1，DMA从源到目的搬运完LEN寄存器配置长度后，将往复，继续从源到目的搬运数据。
   - 若CFG.RELOAD置0，DMA搬运完LEN寄存器配置的长度后，DMA将停止。

 **8. 配置中断**

   DMA中断类型分为三种，见下表。请根据需要情况使能中断，面向用户程序的中断服务函数为 `dma_irq_handler()` 。

  .. image:: ../../_static/kiwi-dma-irq-type.png
   :align: center

 **9. 使能DMA**

   置位DMA_EN寄存器，除了DMA_EN.CHN_EN需要置位，同时也需要置位DMA_EN.CHN_EN_WE。

PAUSE功能
^^^^^^^^^^^^^^^^^^^^^^^

当DMA_EN.CHN_EN被置位后，可以通过CFG.PAUSE置1随时暂停DMA，暂停后，可将CFG.PAUSE置0使DMA继续搬运。

注意事项
----------------------------------

 1. 使能DMA通道时，必须同时置位DMA_EN.CHN_EN和DMA_EN.CHN_EN_WE。
 2. 不要随便将CMU_CLKEN0.DMA和RMU_RSTEN0.DMA清0 ，因为四个串行通道共用一个复位和时钟开关。
 3. 为了最大程度确保用户流程和场景库的运行兼容性，SPV1x SDK提供 `dma_irq_handler()` 函数作为面向用户程序
    的DMA中断处理流程入口，请勿使用底层的 `dma_irq_entry()` 。
 4. 当使用DMA正常模式以NOR Flash空间为源地址空间进行传输时，请使用预设宏定义 `NORC_UNCACHE_ADDR(addr)` 将源地址进行转换后使用。
 5. 当DMA正在搬运，即EN寄存器对应通道置位时，不要修改通道配置寄存器SSA，SOA，DSA，DOA，LEN，CFG寄存器的值，
    否者修改不但不会生效，搬运也会产生错误，只有当EN对应通道为复位值时才可修改配置寄存器。

API说明
----------------------------------

API搬运特性
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

  为了简化DMA使用，同时仍然满足大多情况的搬运场景
    1. API中屏蔽了槽（slot）的概念，保留搬运完成和搬运半完成。但当搬运总字节数不为128的倍数，半完成pending会和完成pending一并到来。
    2. 搬运只支持连续空间数据搬运。
    3. 通道优先级固定，通道号越大，优先据越高。

简介
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. c:enum:: dma_it_type_t

  DMA中断类型枚举

   - *DMA_Half_Transfer_IT*:半完成中断
   - *DMA_Transfer_IT*:完成中断

:说明:
 1. 完成中断与半完成中断分别是指，已从源空间搬运到目的空间，设置搬运总字节数的全部或一半。与搬运模式无关，当搬运模式为循环搬运，pending也会循环触发。
 2. 无论是否使能中断，搬运完成pending和半完成pending都分别会在搬运完成和搬运半完成时置位。
 3. 当设置的搬运完成总字节数为非128字节的倍数，半完成会和完成pending一起触发，即不会在搬运总字节的一半时产生半完成pending，此时半完成pending应无视。
 4. 无论在何种搬运模式，都可以通过函数 `dma_irq_get_flag()` 获取DMA通道pending的是否置位来判断数据是否搬运完成。
 5. DMA中断使能函数 `dma_irq_enable()` 只影响中断服务函数是否被调用，不影响pending的产生。
 6. 中断服务函数为 `dma_irq_handler()` ,不要使用dma_irq_entry()，因为场景库中会调用dma_irq_handler()来确保用户DMA中断的正确响应。
 7. DMA所有通道公用一个DMA函数入口，需要在 `dma_irq_handle()` 函数中调用 `dma_irq_get_flag()` 函数来判断触发中断的通道和中断类型。

.. c:enum:: dma_data_width_t

  DMA搬运单位数据宽度

   - *DMA_Data_Width_8Bit*：搬运单位为byte 
   - *DMA_Data_Width_16Bit*：搬运单位为short
   - *DMA_Data_Width_32Bit*：搬运单位为word

:说明:
 1. 源和目的搬运单位宽度相同，由一个参数 `data_width` 共同指定。故在使用外设为搬运源或目的空间时，需要注意外设FIFO的数据宽度，不匹配的外设数据宽度会导致数据搬运错误。
 2. 源空间首地址和目的空间首地址必须按照搬运单位数据宽度对齐，即：

    - 当数据宽度为8bit(byte)时，空间首地址可任意；
    - 当数据宽度为16bit(short)，空间首地址必须对齐到2，即被2整除；
    - 当数据宽度为32bit(word)时，空间首地址必须对齐到4，即被4整除；
    - 否者导致错误数据搬运。

.. c:enum:: dma_mode_t

  DMA搬运模式

   - *DMA_Mode_Single*：单次搬运
   - *DMA_Mode_Circular*：循环搬运

:说明:
 1. 不同搬运模式是指定在DMA完成设置搬运总量后的行为。
 
   - 单次搬运是指DMA完成指定的搬运总量后，立即停止搬运，此模式可通过DMA状态查询函数 `dma_get_state()` 获取DMA通道状态，以判断数据是否搬运完成。也可以通过获取DMA通道搬运的完成pending来判断数据是否搬运完成。
   - 循环搬运是指DMA完成指定搬运总量后，又从头重新开始搬运，相当于重启DMA通道搬运，但不会清除上次搬运的pending，同时也会逐渐覆盖上次搬运的数据，循环搬运直至用户调用函数 `dma_abort()` 才会停止搬运。

.. c:struct:: dma_init_parameter_t

  DMA搬运通道初始化参数

   - *src_addr*：源空间首地址
   - *dst_addr*：目的空间首地址
   - *data_width*：搬运单位数据宽度，指选择 `dma_data_width_t` 或用sizeof()指定
   - *data_length*：总搬运长度，单位 `data_width` ,参数范围 1-10240
   - *transfer_mode*：搬运模式，值选择 `dam_mode_t` 。

:说明:
 1. 源空间和目的空间首地址必须按照搬运单位的数据宽度对齐，当首地址为外设地址时，地址在搬运过程中不会改变；当首地址为内存地址时，地址在搬运过程会自加，在循环模式中，自加至搬运总量对应地址后，地址会自动回到首地址，再自加。
 2. 成员 `data_width` ，指定的四搬运单位的数据宽度，参数选至枚举 `dma_data_width_t` ，或使用宏函数sizeof();
 3. 成员 `data_length` ,指定的时搬运单位的疏朗，参数范围1-10240，实际搬运总字节数为 `data_length` 乘以 `data_width` ;
 4. 成员 `transfer_mode` ,指定搬运模式，值选至 `dam_mode_t` 。搬运模式的含义参看 `dma_mode_t` 的说明

.. c:enum:: dma_state_t

  DMA通道运行状态

   - *DMA_State_Inited*：初始状态，dma_init()或dma_abort()函数进入该状态
   - *DMA_State_Reset*：复位状态，dma_deinit()函数进入该状态
   - *DMA_State_Running*：搬运状态，dma_start()或dma_resume()函数进入该状态
   - *DMA_State_Running*：搬运暂停状态，dma_pause()函数进入该转台

:说明:
 1. DMA通道运行状态由四种状态组成，每个状态由特定函数进入，在指定状态只能调用指定函数，否者调用无效，甚至使搬运出错。影响状态函数共6个，各状态下状态函数调用说明：
  
    .. image:: ../../_static/kiwi-dma-api-state.jpg
      :align: center
    .. image:: ../../_static/kiwi-dma-api-state-t.png
      :align: center

.. c:function:: void dma_init(uint32_t chx,dma_init_parameter_t *dma_init_param)

  DMA通道初始化，根据 `dma_init_paremeter_t` 参数初始化通道

  :param chx: DMA通道号，参数范围0-3
  :param dma_init_param:  `dam_init_parameter_t` 结构体指针
  :returns: 无
  :note: 当搬运总量（byte）不是128byte的倍数，不存在半完成中断，强行使能半完成中断，会和完成中断一起到达。
  :note: 调用该函数后，DMA通道状态进入初始态DMA_State_Inited


.. c:function:: void dma_deinit(uint32_t chx)

  DMA通道去初始化

  :param chx: DMA通道号，参数范围0-3
  :returns: 无
  :note: 调用该函数后，DAM通道状态进入复位态，DMA_State_Reset

.. c:function:: void dma_start(uint32_t chx)

  DMA通道开始搬运
  
  :param chx: DMA通道号，参数范围0-3
  :returns: 无
  :note: dma_init()后调用该函数，开始搬运
  :note: 调用该函数后，DMA通道状态进入搬运状态DMA_State_Running
  :note: 若搬运模式是DMA_Mode_Single，在数据搬运完成后，状态会自动回到初始态DMA_State_Inited


.. c:function:: void dma_abort(uint32_t chx)

  中止DMA通道搬运

  :param chx: DMA通道号，参数范围0-3
  :returns: 无
  :note: dma_start()后调用此函数，中止搬运
  :note: 调用该函数后，DMA通道状态进入初始状态DMA_State_Inited


.. c:function:: void dma_pause(uint32_t chx)

  暂停DMA通道搬运

  :param chx: DMA通道号，参数范围0-3
  :returns: 无
  :note: dma_start()后调用该函数，暂停搬运
  :note: 调用该函数后，DMA通道状态进入暂停状态DMA_State_Paused


.. c:function:: void dma_resume(uint32_t chx)

  恢复暂停DMA通道搬运

  :param chx: DMA通道号，参数范围0-3
  :returns: 无
  :note: dma_pause()后调用此函数，恢复搬运
  :note: 调用该函数后，DMA通道状态进入搬运状态DMA_State_Running


.. c:function:: dma_state_t dma_get_state(uint32_t chx)

  获取DMA通道状态

  :param chx: DMA通道号，参数范围0-3
  :returns: DMA通道当前状态
  :retval: dma_state_t
  :note: 详细参看 `dma_state_t` 说明

.. c:function:: uint32_t dma_get_transfer_residual(uint32_t chx)

  获取DMA通道搬运剩余量

  :param chx: DMA通道号，参数范围0-3
  :returns: DMA通道当前搬运剩余量，单位byte 
  :retval: uint32_t
  :note: 返回值不是任何时候都有效，只有当DMA通道处于DMA_State_Running或DMA_State_Pause状态，返回值有效

.. c:function:: void dma_irq_enable(uint32_t chx,dma_it_type_t it_type)

  使能DMA通道中断，使能DMA中断服务函数能够进入，不影响DMA中断pending产生

  :param chx: DMA通道号，参数范围0-3
  :param it_type: 中断类型，值选择dma_it_type_t

    - *DMA_Half_Transfer_IT*: 半完成中断
    - *DMA_Half_Transfer_IT*: 完成中断
  :retruns: 无
  :note: 当搬运总量（byte）不是128byte的倍数，不存在半完成中断，强行使能半完成中断，会和完成中断一起到达。

.. c:function:: void dma_irq_disbale(uint32_t chx,dma_it_type_t it_type)

  失能DMA通道中断，失能DMA中断服务函数不能够进入，不影响DMA中断pending产生

  :param chx: DMA通道号，参数范围0-3
  :param it_type: 中断类型，值选择dma_it_type_t

    - *DMA_Half_Transfer_IT*: 半完成中断
    - *DMA_Half_Transfer_IT*: 完成中断
  :retruns: 无
  :note: 当搬运总量（byte）不是128byte的倍数，不存在半完成中断，强行使能半完成中断，会和完成中断一起到达。

.. c:function:: soc_set_t dma_irq_get_flag(uint32_t chx,dma_it_type_t it_type)

  获取DMA通道中断pending标志，当搬运量达到设定值，即产生中断pending，pending的产生不受中断使能或失能的影响

  :param chx: DMA通道号，参数范围0-3
  :param it_type: 中断类型，值选择dma_it_type_t

    - *DMA_Half_Transfer_IT*: 半完成中断
    - *DMA_Half_Transfer_IT*: 完成中断
  :returns: 中断pending状态
  :retval Set: 对应中断pending置位
  :retval Reset: 对应中断pending复位

.. c::function:: void dma_irq_clear_flag(uint32_t chx,dma_it_type_t it_type)

  清除DMA通道中断Pending

  :param chx: DMA通道号，参数范围0-3
  :param it_type: 中断类型，值选择dma_it_type_t

    - *DMA_Half_Transfer_IT*: 半完成中断
    - *DMA_Half_Transfer_IT*: 完成中断
  :returns: 无
  :note: 当搬运总量（byte）不是128byte的倍数，不存在半完成中断，强行使能半完成中断，会和完成中断一起到达

外设搬运
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

  外设可作为DMA搬运起始或目的，支持DMA搬运的外设由SPI TX/RX，UART0 TX/RX,UART1 TX/RX，DSM TX/CAP0 RX，CAP1 RX，PWM TX/ADC RX。使用API搬运数据，只需要将外设地址的值填入初始化参数的首地址即可。

API使用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

CPU查询状态方式
"""""""""""""""""""""""""""""""""""""""

 1. 调用dma_get_state(chx)确定DMA是否在数据搬运，若在搬运数据，调用dma_abort(chx)停止通道或更换通道；
 2. 先调用dma_deint(chx)，主要为了清除上一次通道搬运未清除的pending和关闭中断，在调用dma_init(chx,param_addr)函数初始化通道，即指定搬运的地址，长度，以及搬运模式。
 3. 调用dma_start(chx)启动DMA通道搬运
 4. 调用dma_irq_get_flat(chx,DMA_Transfer_IT)查询DMA搬运是否完成。若为单次搬运，可以调用函数dma_get_state(chx)查询搬运是否完成。
 5. 当查询DMA搬运完成，调用dma_irq_chear_flag(chx,DMA_Transfer_IT)清除完成pending，为为此搬运查询准备。

 .. image:: ../../_static/kiwi-dma-api-cpu.png
   :align: center

中断方式
"""""""""""""""""""""""""""""""""""""""

 1. 调用dma_get_state(chx)确定dma是否在运行数据搬运，若在搬运数据，调用dma_abort(ch)停止通道或更换通道；
 2. 先调用dma_deinit( chx)，主要为了清除上一次通道搬运未清除的pending和关闭中断，再调用dma_init(chx,param_addr)函数初始化通道，即指定搬运的地址，长度，以及搬运模式。
 3. 调用dma_irq_enable(chx, it_type)函数使能中断。
 4. 调用dma_start(chx)启动DMA通道搬运。
 5. 中断服务函数中调用dma_irq_get_flag( chx, it_type)确定对应通道的对用中断pending产生。
 6. 若对应通道中断置位，调用dma_irq_clear_flag(chx,DMA_Transfer_IT)清除完成pending，否则中断服务函数退出后会立即再进入。

  注意：上述流程建立在，中断服务函数中只有使能的通道中断pending处理。若含有未使能的中断pending处理，需要添加中断是否使能判断（该函数API未提供，即不推荐在中断服务函数中含有未使能的通道中断pending处理）。

 .. image:: ../../_static/kiwi-dma-api-interrupt.jpg
   :align: center

寄存器定义
----------------------------------

SSA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-dma-ssa.png
   :align: center

SOA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-dma-soa.png
   :align: center

DSA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-dma-dsa.png
   :align: center

DOA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-dma-doa.png
   :align: center

LEN
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-dma-len.png
   :align: center

CFG
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-dma-cfg.png
   :align: center

STA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-dma-sta.png
   :align: center

EN
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-dma-en.png
   :align: center

IE
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-dma-ie.png
   :align: center

PD
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. image:: ../../_static/kiwi-reg-dma-pd.png
   :align: center