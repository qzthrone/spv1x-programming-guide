midi文件播放器播放器
======================

.. _音频媒体文件调用方法: media-file.html

.. note::
   
   前置知识： :ref:`media-file-format` 和 :ref:`player-parameter-structure`

简介
-------------------------

该场景库实现midi文件的综合，并通过DSM单元播放。

特点
-------------------------

 - 32khz采样播放，单声道；
 - 支持drc

API说明
-------------------------

.. c:function:: int32_t player_midi_init(uint32_t midi_addr,uint32_t sf_addr,uint32_t dsm_dma_ch);

    midi文件播放初始化

    :param midi_addr: midi文件的存储地址
    :param sf_addr: soundfont文件的存储地址
    :retval >0: 播放器使用到sram的等高线
    :retval -1: SRAM不足，播放器初始化失败
    :note: midi文件的播放不仅需要midi文件还需要soudfont文件，所有打包的midi文件共用一个sondfont文件。

.. c:function:: int32_t player_midi_deinit();

    播放器去初始化

  :retval  >0: 播放器释放sram后的地址
  :retval -1: 播放器不存在。

.. c:function:: int32_t player_midi_cmd(player_music_cmd_t cmd)

  播放器控制命令。

  :param cmd: 支持预设枚举定义，见 player_music_cmd_t
  :retval  0: 播放器命令执行成功。
  :retval -1: 播放器不存在。


.. c:function:: int32_t player_midi_get_state(dec_info_t *dec_info)

  获取播放器内部解码器状态信息。

  :param dec_info_t: 结构体 dec_info_t 指针，结构体定义见后文。 
  :retval  0: 内部解码器状态获取成功。
  :retval -1: 播放器不存在。


.. c:function:: int32_t player_midi_set_volume_scale(uint32_t scale)

  设置播放器的音量缩放。

  :param scale: Q16.16格式无符号定点数，65536对应无缩放。
  :retval  0: 缩放因子设置成功。
  :retval -1: 播放器不存在。

.. c:function:: int32_t player_midi_set_frame_hook( void (*func)(void* buffer_addr,int32_t *len) )

  可选配钩子函数，配置每帧解码完成钩子函数

  :param func: 钩子函数，包含两个参数，一个为解码音频地址，第二个为解码音频长度，宽度默认32bit。
  :retval  0: 钩子函数设置成功。
  :retval -1: 播放器不存在。

.. c:function:: int32_t player_midi_set_finish_hook(void (*func)())

  可选配钩子函数，配置后在曲目播放自然结束后触发调用。

  :param func: 钩子函数，要求无参无返回值。
  :retval  0: 钩子函数设置成功。
  :retval -1: 播放器不存在。

使用方法
-------------------------

 .. image:: ../../_static/kiwi-mp3-16k-fsm.png
  :align: center
 
 1. 调用 player_midi_init() 进行播放器初始化，播放器进入 Ready 状态。
 2. 调用 player_midi_cmd(`Player_CMD_Start`)，开始播放，播放器进入 Playing 状态。
 3. 播放过程中可以随时调用 player_midi_cmd(`Player_CMD_Pause`)/player_midi_cmd(`Player_CMD_Resume`) 在 Playing 和 Paused 之间切换播放器状态。
 4. 播放过程自然结束或调用 player_midi_cmd(`Player_CMD_Stop`) 都会使得播放器进入 Stopped 状态。
 5. 通过调用 player_midi_init() 可以将播放器重新置于 Ready 状态。
 6. 否则，调用 player_midi_deinit() 即可释放播放器资源占用(Cleared 状态)。

注意事项
-------------------------

 1. 源码中需要先定义 音频播放器的必须品中 player_dec_sequence_t ,player_file_attribute_t ,player_music_cmd_t 枚举和 dec_info_t 结构体，否者编译错误
 2. 播放器运行过程占用DSM单元，已经指定的两路DMA2、3通道，以及软件中断(MSIP)。播放器去初始化后，资源占用将被释放。
