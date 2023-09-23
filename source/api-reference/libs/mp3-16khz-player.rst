16KHz单声道MP3播放器
======================

.. _音频媒体文件调用方法: media-file.html

.. note::
   
   前置知识： `音频媒体文件调用方法`_

简介
-------------------------

该场景库实现MP3文件解码并通过DSM单元播放，仅限16KHz单声道PCM编码的MP3文件使用。


API说明
-------------------------




.. c:function:: int32_t player_mp3_16k_init(player_init_parameter_t *init_parameter)

  初始化16KHz单声道MP3播放器。

  :param init_parameter: player_init_parameter_t类型指针。详细信息参看 player_init_parameter_t
  :returns: 播放器初始化后的SRAM空间以使用的总量(字节数)。
  :retval >0: 播放器使用到sram的等高线
  :retval -1: SRAM不足，播放器初始化失败。
  :retval <-1: 绝对值为播放器初始化后使用到sram的等高线,负号表示tsps初始化失败

.. c:struct:: player_init_parameter_t

  初始化参数

   .. c:member:: void* file_info

    播放文件信息，资源文件放在flash上时，应为resource_t的指针

   .. c:member:: uint32_t fade_in_samples

    开始播放时，淡入音频点数

   .. c:member:: uint32_t fade_out_samples

    停止播放时，淡出音频点数

   .. c:member:: player_type_bit_t type

    播放的特性，详细参看  player_type_bit_t

   .. c:member:: uint32_t speed 

    播放速度，Q29 范围(0.5,2)

   .. c:member:: uint32_t pitch

    播放变调，Q29 范围 (0.5,2)

.. c:struct:: player_type_bit_t

  播放的特性
   
   .. c:member:: uint32_t loop_en :1

    循环播放，置位有效

   .. c:member:: uint32_t drc_en :1

    drc后置处理，置位有效

   .. c:member:: uint32_t file_type :1

    文件存放位置，0表示存放Flash




.. c:function:: int32_t player_mp3_16k_deinit()

  16KHz单声道MP3播放器去初始化。

  :retval  >0: 播放器释放sram后的地址
  :retval -1: 播放器不存在。

.. c:function:: int32_t player_mp3_16k_cmd(player_music_cmd_t cmd)

  MP3控制命令。

  :param cmd: 支持预设枚举定义，见 player_music_cmd_t
  :retval  0: 播放器命令执行成功。
  :retval -1: 播放器不存在。
  :retval -2: 并未解码过，即刚初始化就调用stop。
  :retval -3: 不支持的命令。

.. c:enum:: player_music_cmd_t

  解码命令，函数 player_xxx_xxx_cmd()输入参数

  - *Player_CMD_Stop*: 值为0，解码停止，若正在播放，会为其添加 fade_out_samples 长度的淡出
  - *Player_CMD_Start*: 值为1，解码开始，会为其添加 fade_in_sample 长度的淡入
  - *Player_CMD_Pause*: 值为2，解码暂停
  - *Player_CMD_Resume*: 值为3，恢复解码暂停


.. c:function:: int32_t player_mp3_16k_get_status(dec_info_t *dec_info)

  获取播放器内部解码器状态信息。

  :param dec_info: 结构体 dec_info_t 指针，结构体定义见后文。 
  :retval  0: 内部解码器状态获取成功。
  :retval -1: 播放器不存在。

.. c:struct:: dec_info_t
  
  解码器运行状态结构体

   .. c:member:: player_dec_sequence_t dec_state

     解码器运行状态

   .. c:member:: uint32_t processed_frames

     已处理的数据帧数

   .. c:member:: int32_t total_size

     音频文件总大小（字节数）
    
   .. c:member:: uint32_t sampling_rate

     音频文件原生采样率

   .. c:member:: uint32_t scale

     当前音量缩放因子

.. c:enum:: player_dec_sequence_t

  解码器状态枚举，状态参数 dec_state 值选择范围

  - *Sequence_End*: 值为0，解码完成且播放完毕
  - *Sequence_Start*: 值为1，
  - *Sequence_Paused*: 值为2，解码暂停，此状态后不会再进解码和dma搬运播放
  - *Sequence_Stopped*: 值为3，解码停止
  - *Sequence_Initialised*: 值为4，解码初始化完成
  - *Sequence_Frame_Processing*: 值为5，正在解码帧
  - *Sequence_Frame_Processed*: 值为6，解码一帧完成，暂处空闲


.. c:function:: int32_t player_mp3_16k_set_volume_scale(uint32_t scale)

  设置播放器的音量缩放。

  :param scale: Q16.16格式无符号定点数，65536对应无缩放。
  :retval  0: 缩放因子设置成功。
  :retval -1: 播放器不存在。

.. c:function:: int32_t player_mp3_16k_set_frame_hook( void (*func)(void* buffer_addr,int32_t *len) )

  可选配钩子函数，配置每帧解码完成钩子函数

  :param func: 钩子函数，包含两个参数，一个为解码音频地址，第二个为解码音频长度，宽度默认32bit。
  :retval  0: 钩子函数设置成功。
  :retval -1: 播放器不存在。

.. c:function:: int32_t player_mp3_16k_set_end_hook(void (*func)())

  可选配钩子函数，配置后在曲目播放自然结束后触发调用。

  :param func: 钩子函数，要求无参无返回值。
  :retval  0: 钩子函数设置成功。
  :retval -1: 播放器不存在。

.. c:function:: int32_t player_mp3_16k_set_stop_hook(void (*func)())

  可选配钩子函数，设置播放一首音乐主动停止钩子函数

  :param func: 钩子函数，要求无参无返回值。
  :retval  0: 钩子函数设置成功。
  :retval -1: 播放器不存在。
  :note: 当正常播，会在中断中调用;当已经播放完毕调用stop会在，stop命名后立马回调

.. c:function:: int32_t player_mp3_16k_append_upon_stop(player_init_parameter_t *preplay_info)

  调用stop命令之前，指定stop命令之后播放的文件信息

  :param preplay_info: 文件信息，与初始化播放器参数一致。
  :retval  0: 设置成功。
  :retval -1: 播放器不存在。
  :note: 调用stop命令之后，需要立马播放指定文件时，需要在stop命令前调用此函数。

.. c:enum:: player_file_attribute_t

  播放文件属性枚举，资源文件参数 attribute 值选择范围

  - *FileAttribute_None*: 值为0，资源文件，结束属性
  - *FileAttribute_MP3*: 值为1，资源文件为MP3，不区分采样率
  - *FileAttribute_S1A*: 值为2，资源文件为S1A，不区分采样率
  - *FileAttribute_MIDI*: 置为3，资源文件为midi文件
  - *FileAttribute_ADPCM*: 置为14，资源文件为ADPCM文件
  - *FileAttribute_SILK*: 值为15，资源文件为silk，不区分采样率


使用方法
-------------------------

 .. image:: ../../_static/kiwi-mp3-16k-fsm.png
  :align: center
 
 1. 调用 player_mp3_16k_init() 进行播放器初始化，播放器进入 Ready 状态。
 2. 调用 player_mp3_16k_cmd(`Player_CMD_Start`)，开始播放，播放器进入 Playing 状态。
 3. 播放过程中可以随时调用 player_mp3_16k_cmd(`Player_CMD_Pause`)/player_mp3_16k_cmd(`Player_CMD_Resume`) 在 Playing 和 Paused 之间切换播放器状态。
 4. 播放过程自然结束或调用 player_mp3_16k_cmd(`Player_CMD_Stop`) 都会使得播放器进入 Stopped 状态。
 5. 通过调用 player_mp3_16k_init() 可以将播放器重新置于 Ready 状态。
 6. 否则，调用 player_mp3_16k_deinit() 即可释放播放器资源占用(Cleared 状态)。

注意事项
-------------------------

 1. 源码中需要先定义 player_dec_sequence_t ,player_file_attribute_t ,player_music_cmd_t 枚举和 dec_info_t 结构体，否者编译错误
 2. 播放器运行过程占用DSM单元，指定的一路DMA通道，以及软件中断(MSIP)。播放器去初始化后，资源占用将被释放。
 