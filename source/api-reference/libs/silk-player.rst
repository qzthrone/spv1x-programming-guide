SILK单声道8KHz播放器
======================

.. _音频媒体文件调用方法: media-file.html

.. note::
   
   前置知识： :ref:`media-file-format` 和 :ref:`player-parameter-structure`


简介
-------------------------

该场景库实现SLK文件解码并通过DSM单元播放，仅限8KHz单声道PCM编码的SILK文件使用。

特点
-------------------------
 - 支持8khz采样，6kbps、8kbsp码率，单声道mp3文件解码；
 - 支持上采样播放；
 - 支持DRC ；
 - 支持淡入淡出；
 - 提供带有虚拟文件系统和cpu直取两版本静态库，但接口一致。

API说明
-------------------------

.. c:function:: int32_t player_silk_8k_init(player_init_parameter_t *init_parameter)

  初始化16KHz单声道MP3播放器。

  :param init_parameter: player_init_parameter_t类型指针。详细信息参看 player_init_parameter_t
  :returns: 播放器初始化后的SRAM空间已使用的总量(字节数)。
  :retval >0: 播放器使用到sram的等高线
  :retval -1: SRAM不足，播放器初始化失败。

  
.. c:function:: int32_t player_silk_8k_deinit()

  单声道S1A播放器去初始化。

  :retval  >0: 播放器释放sram后的地址
  :retval -1: 播放器不存在。


.. c:function:: int32_t player_silk_8k_cmd(player_init_parameter_t *init_parameter)
  
  SILK控制命令。

  :param cmd: 支持预设枚举定义，见 player_music_cmd_t
  :retval  0: 播放器命令执行成功。
  :retval -1: 播放器不存在。
  :retval -2: 并未解码过，即刚初始化就调用stop。
  :retval -3: 不支持的命令。
  

.. c:function:: int32_t player_silk_8k_get_state(dec_info_t *dec_info)

  获取播放器内部解码器状态信息。

  :param dec_info_t: 结构体 dec_info_t 指针，结构体定义见后文。 
  :retval  0: 内部解码器状态获取成功。
  :retval -1: 播放器不存在。

.. c:function:: int32_t player_silk_8k_set_volume_scale(uint32_t scale)

  设置播放器的音量缩放。

  :param scale: Q16.16格式无符号定点数，65536对应无缩放。
  :retval  0: 缩放因子设置成功。
  :retval -1: 播放器不存在。

.. c:function:: int32_t player_silk_8k_set_frame_hook( void (*func)(void* buffer_addr,int32_t *len) )

  可选配钩子函数，配置每帧解码完成钩子函数

  :param func: 钩子函数，包含两个参数，一个为解码音频地址，第二个为解码音频长度，宽度默认32bit。
  :retval  0: 钩子函数设置成功。
  :retval -1: 播放器不存在。

.. c:function:: int32_t player_silk_8k_set_end_hook(void (*func)())

  可选配钩子函数，配置后在曲目播放自然结束后触发调用。

  :param func: 钩子函数，要求无参无返回值。
  :retval  0: 钩子函数设置成功。
  :retval -1: 播放器不存在。

.. c:function:: int32_t player_silk_8k_set_stop_hook(void (*func)())

  可选配钩子函数，设置播放一首音乐主动停止钩子函数

  :param func: 钩子函数，要求无参无返回值。
  :retval  0: 钩子函数设置成功。
  :retval -1: 播放器不存在。
  :note: 当正常播，会在中断中调用;当已经播放完毕调用stop会在，stop命名后立马回调

.. c:function:: int32_t player_silk_8k_append_upon_stop(player_init_parameter_t *preplay_info)

  调用stop命令之前，指定stop命令之后播放的文件信息。因为stop命令会经行fade out，并不是立马停止播放。

  :param preplay_info: 文件信息，与初始化播放器参数一致。
  :retval  0: 设置成功。
  :retval -1: 播放器不存在。
  :note: 调用stop命令之后，需要立马播放指定文件时，需要在stop命令前调用此函数。

使用方法
-------------------------

 .. image:: ../../_static/kiwi-mp3-16k-fsm.png
  :align: center
 
 1. 调用 player_silk_8k_init() 进行播放器初始化，播放器进入 Ready 状态。
 2. 调用 player_silk_8k_cmd(`Player_CMD_Start`)，开始播放，播放器进入 Playing 状态。
 3. 播放过程中可以随时调用 player_silk_8k_cmd(`Player_CMD_Pause`)/player_silk_8k_cmd(`Player_CMD_Resume`) 在 Playing 和 Paused 之间切换播放器状态。
 4. 播放过程自然结束或调用 player_silk_8k_cmd(`Player_CMD_Stop`) 都会使得播放器进入 Stopped 状态。
 5. 通过调用 player_silk_8k_init() 可以将播放器重新置于 Ready 状态。
 6. 否则，调用 player_silk_8k_deinit() 即可释放播放器资源占用(Cleared 状态)。

注意事项
-------------------------

 1. 源码中需要先定义,音频播放器的必须品中 player_dec_sequence_t ,player_file_attribute_t ,player_music_cmd_t 枚举和 dec_info_t 结构体，否者编译错误
 2. 播放器运行过程占用DSM单元，已经指定的一路DMA3通道，以及软件中断(MSIP)。播放器去初始化后，资源占用将被释放。
 3. 提供两个SILK解码播放库，其中名字中不带-vfs为flash播放库，带有-vfs为同时支持sd卡和flash播放库。
 
