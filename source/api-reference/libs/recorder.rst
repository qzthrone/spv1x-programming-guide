录音机
=========================

简介
-------------------------

SPV1x的录音库，提供音频录制和压缩存储功能。该录音库有如下特性：

 1. 音频采样率为16KHz。
 2. 使用MP3编码器压缩音频，压缩后的音频码率为16kbps。
 3. 向应用提供回调函数接口：录音开始回调、录音结束回调、帧采集完成回调、VAD计算完成回调。
 4. 提供音频回溯功能，最长支持1秒的音频回溯。

API说明
-------------------------

.. c:function:: uint32_t recorder_init(uint32_t flash_addr,uint32_t flash_size)

  录音库初始化。

  :param flash_addr: 用于保存录音数据的flash区域的起始地址。
  :param flash_size: 用于保存录音数据的flash区域的大小（单位：字节）。
  :returns: 录音库占用的SRAM大小(字节数)，该值用于评估项目内存使用情况。

.. c:function:: void recorder_set_start_hook(void (*start)(void))

   设置录音开始回调函数。

   :param start: 录音开始回调函数指针。
   :returns: 无
   :note: 用户调用 `recorder_start()` 后，录音并不是立即开始。因此录音开始回调函数的执行会相对 `recorder_start()` 有一定的延迟。

.. c:function:: void recorder_set_stop_hook(void (*stop)(void))

   设置录音结束回调函数。

   :param stop: 录音结束回调函数指针。
   :returns: 无
   :note: 用户调用 `recorder_stop()` 后，录音并不是立即结束。因此录音结束回调函数的执行会相对 `recorder_stop()` 有一定的延迟。

.. c:function:: void recorder_set_vad_hook(void (*vad)(uint32_t energy))

   设置VAD计算完成回调函数。

   :param vad: VAD计算完成回调函数指针。该回调函数带一个形参，用于传递当前音频帧采样点的平均能量信息。
   :returns: 无

.. c:function:: void recorder_set_frame_hook(void (*frame)(int16_t * pcm,uint32_t len))

   设置帧采集完成回调函数。

   :param frame: 帧采集完成回调函数指针。该回调函数第一个参数为当前帧PCM数据的内存地址，第二个参数为PCM数据的长度（采样点数目）。
   :returns: 无

.. c:function:: void recorder_deinit(void)

  录音库去初始化。

  :returns: 无

.. c:function:: void recorder_start(uint32_t pre_time_ms)

   请求开始录音。

   :param pre_time_ms: 录音需要回溯的时间，0ms ≤ pre_time_ms ≤ 1000ms。
   :returns: 无
   :note: 录音库在初始化后就开始缓存音频数据，最多缓存1秒的音频。如果在初始化录音库后立即开始录音，则没有音频数据可以回溯。

.. c:function:: void recorder_stop(void)

   请求停止录音。

   :returns: 无


录音数据在flash中的存放
-------------------------

 .. image:: ../../_static/recorder_data_area.png
  :align: center

 1. 录用功能使用的flash区域由调用 `recorder_init()` 时的参数 `flash_addr` 和 `flash_size` 决定。
 2. flash区域的第一个word（0~4字节，小端格式）保存着实际录音数据的长度信息。
 3. flash区域的第二个word（4~8字节，小端格式）保存着 `RECORDER_MAGIC` 常数，其值为0x00444352。该值用于确定flash区域是否存在有效的录音数据。
 4. 从第9字节开始，用于存放MP3编码后的录音数据。

录音库使用方法
-------------------------

 1. 调用 `recorder_init()` 初始化录音功能需要用到的资源，并传入用于保存录音数据的flash区域参数。
 2. 根据需求，设置对应的回调函数。未设置的回调函数默认为NULL，录音库会跳过对NULL回调函数的调用。
 3. 调用 `recorder_start()` 开始录音：SPV1x对ADC采集的音频数据进行编码压缩，然后写入flash。
    当指定长度的flash空间用尽后，SPV1x自动停止录音。
 4. 用户也可以根据场景需求，主动调用 `recorder_stop()` 手动停止录音。
 5. 如果需要进行VAD录音，用户可以在VAD回调函数中对音频帧的能量参数进行判断，并决定是否调用 `recorder_start()` 启动录音或者调用 `recorder_stop()` 停止录音。
 6. 调用 `recorder_deinit()` 去初始化：释放录音功能需要用到的资源，结束录音机场景。

注意事项
-------------------------

 1. 录音功能需要配合SPV1x的 `事件驱动型用户程序框架`_ 使用。
 2. 录音功能用会用到一些外设资源，用户应避免这些资源的使用，以免因为资源冲突导致录音功能无法正常工作。录音用到外设资源如下：

   a. ADC：采集音频数据。
   b. DMA：通道3，录音过程中音频数据的搬运。
   c. 软件中断（MSIP）：音频数据的编码和其他处理。
   d. SPI0：操作flash进行录音数据的保存。
   e. GPIO10~13：作为SPI0的功能引脚。

 3. 录音开始回调、录音结束回调、帧采集完成回调、VAD计算完成回调都在中断函数中调用，因此回调函数的内容需要简短。
 4. flash的擦写时间对录音功能影响较大，普冉flash的擦除时间比较能满足录音功能的需求。

 .. _事件驱动型用户程序框架: ../../user-guide/user-fw-design.html
