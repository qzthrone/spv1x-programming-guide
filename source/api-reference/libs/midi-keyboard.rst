MIDI键盘
======================

简介
----------------------



场景库实现midi琴功能：综合琴键消息并通过DSM单元播放，该库还包含琴键的拓展功能

    - 一键修改琴键音色
    - 支持打击乐
    - 支持琴键移调
    - 支持琴键颤音
    - 支持midi文件播放、伴奏和变速播放
    - 支持琴键录音、回放
    - 支持假弹功能
    - 支持音量控制

基本琴键
----------------------

向midi综合器发送midi消息，以弹奏不同频率音调

**函数使用**

 - 调用player_midi_keyboard_init()函数后，在调用player_midi_keyboard_deinit()函数之前，可根据情况调用其它任意函数

.. c:function:: int32_t player_midi_keyboard_init()

    midi琴键初始化

    :returns: 错误码
    :retval >0: 初始化成功，返回目前剩余SRAM字节数
    :retval -1: SRAM空间不足，初始化失败 
    :note: 开始midi琴键功能，同时包含使能软件中断，注册软件中断和dma中断服务函数，搬运对应bss、data段数据，获取midi资源信息，并初始化midi综合器
    :note: 所有功能的使用都建立在琴键功能基础上，故此函数必须第一个调用，否则其它调用都无意义

.. c:function:: int32_t player_midi_keyboard_deinit()

    midi琴键去初始化

    :returns: 错误码
    :retval 0: 去初始化成功
    :retval -1: midi综合器未初始化
    :note: 当不再需要琴键功能，通过此函数释放琴键功能占有的资源，包括DMA、DSM和SRAM。
    :note: 会将dsm关闭，dma中断和软件中断服务函数会修改原默认函数入口

.. c:function:: int32_t player_send_midi_msg_note(int32_t note_val,int note_type)

    琴键消息，发送琴键消息给综合器

    :param note_val: 琴键号，参数范围 0-127
    :param note_type: 琴键状态， 1表示按下， 0表示松开
    :returns: 错误码
    :retval 0: 琴键消息发送成功
    :retval -1: midi综合器未初始化
    :retval -2: 琴键消息缓存溢出
    :note: 按照midi标准，琴键消息分为按下和抬起，琴键号支持0-127。另外琴键速度固定127，琴键消息通道固定通道15。

.. c:function:: int32_t player_send_midi_msg_timbre(int32_t timbre_val)

    琴键音色，发送修改琴键音色消息

    :param timber_val: 标准midi音色号
    :returns: 错误码
    :retval 0: 音色修改成功
    :retval -1: midi综合器未初始化
    :retval -2: 琴键消息缓存溢出
    :retval -3: 资源文件没有对应音色
    :note: 音色的修改，需要资源库中包含的音色才可修改，否则修改失败，音色号使用标准midi音色号

.. c:function:: int32_t player_send_midi_msg_modulation(uint8_t modulation_value)

    琴键颤音，给琴键的声音添加颤音

    :param modulation_value: 颤音的程度，值越大程度越大，0表示不添加，参数范围 0-255
    :returns: 错误码
    :retval 0: 颤音添加成功
    :retval -1: midi综合器未初始化
    :retval -2: 琴键消息缓存溢出

.. c:function:: int32_t player_send_midi_msg_pitch(uint8_t pitch_value_L,uint8_t pitch_value_H)

    琴键移调，调整琴键整体声音频率偏移

    :param pitch_value_L: 频移的底七位
    :param pitch_value_H: 频移的高七位
    :returns: 错误码
    :retval 0: 移调成功
    :retval -1: midi综合器未初始化
    :retval -2: 琴键消息缓存溢出
    :note: pitch_value_H改变0x20即可实现相邻键变调

.. c:function:: int32_t player_send_midi_msg_percussion(int32_t percussion_value ,int32_t percussion_type)

    打击乐，向midi发送打击乐消息

    :param percussion_value: 打击乐序号，从0开始
    :param percussion_type: 按下或松开 按下为1，松开为0
    :returns: 错误码
    :retval 0: 打击乐信息发送成功
    :retval -1: midi综合器未初始化
    :retval -2: 琴键消息缓存溢出
    :note: 不同打击乐对应不同音色号，区分按下和松开。
    :note: 打击乐的音色号不是标准midi中的音色号，而是资源文件中的已有音色号，从0开始递增；另外打击乐固定使用通道9

.. c:function:: void music_set_soundvolume(int8_t voice)

    设置指定音量

    :param voice: 播放音量 ，范围0-16，值越大音量越大
    :returns: 无

.. c:function:: int32_t player_get_player_state(Midikeyboard_Player_Info_Struct *player_info)

    获取琴键功能状态信息

    :returns: 错误码
    :retval 0: 获取状态成功
    :retval -1: midi综合器未初始化
    :note: 详情结构体 Midikeyboard_Player_Info


使用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. 调用函数 player_midi_keyboard_init() 初始化
2. 根据按键功能划分调用特定消息发送函数
3. 根据情况调用 int32_t player_midi_keyboard_deinit() 去初始化


扩展功能
----------------------

MIDI文件播放
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

将midi文件按功能分为伴奏midi和普通midi，伴奏midi一般无琴键演奏音色的声音，否则伴奏过程中，按键的琴键音很容易被掩盖。


**函数使用**

 - 调用player_midi_keyboard_init()函数后，在调用player_midi_keyboard_deinit()函数之前，可根据情况调用其它任意函数
 - 函数player_common_midi()、player_accompany_midi()、player_sham_midi()用于播放midi文件，player_comeoff_midifile()用于中止midi文件的播放

.. c:function:: int32_t player_common_midi(int32_t file_id)

    播放普通midi文件

    :param file_id: midi文件在普通midi文件的序号
    :returns: 错误码
    :retval 0: 播放文件成功
    :retval -1: midi综合器未初始化
    :retval -2: 没有该midi文件
    :note: 调用此函数后，即可播放指定midi文件，琴键仍可使用
    :note: 一首普通midi文件播放完毕后停止

.. c:function:: int32_t player_accompany_midi(int32_t file_id)

    播放伴奏midi文件

    :param file_id: midi文件在普通midi文件的序号
    :returns: 错误码
    :retval 0: 播放文件成功
    :retval -1: midi综合器未初始化
    :retval -2: 没有该midi文件
    :note: 调用此函数后，即可播放指定伴奏midi文件。琴键仍可用
    :note: 一首伴奏midi文件会重复播放，

.. c:function:: int32_t player_sham_midi(int32_t file_id)

    假弹奏，虚假按键演奏midi文件

    :param file_id: midi文件在普通midi文件的序号
    :returns: 错误码
    :retval 0: 播放文件成功
    :retval -1: midi综合器未初始化
    :retval -2: 没有该midi文件
    :note: midi文件的播放，需要琴键有动作，midi文件的消息才会被综合器处理，而琴键对应的消息综合器不以理睬

.. c:function:: int32_t player_comeoff_midifile()

    中止播放，中止midi文件的播放

    :returns: 错误码
    :retval 0: 播放文件暂停
    :retval -1: midi综合器未初始化


.. c:function:: int32_t player_midifile_playspeed_increase()

    改变midi文件播放速度

    :returns: 错误码
    :retval 0: 改变文件播放速度成功
    :retval -1: midi综合器未初始化
    :note: 减小midi消息间的时间差，每次调用减少10ms

.. c:function:: int32_t player_midifile_playspeed_decrease()

    减慢midi文件播放速度

    :returns: 错误码
    :retval 0: 改变文件播放速度成功
    :retval -1: midi综合器未初始化
    :note: 增加midi消息间的时间差，每次调用增加10ms

.. c:function:: int32_t player_midifile_playspeed_reset();

    midi文件的播放恢复到正常速度

    :returns: 错误码
    :retval 0: 改变文件播放速度成功
    :retval -1: midi综合器未初始化


琴键录音
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**函数使用**

 - 调用player_midi_keyboard_init()函数后，在调用player_midi_keyboard_deinit()函数之前，可根据情况调用其它任意函数
 - 录音文件的中止播放，也由 player_comeoff_midifile() 函数控制

.. c:function:: int32_t player_midi_keyrecord_start(flash_init_area_functype flash_init_area_funcaddr,flash_save_data_functype flash_save_data_funcaddr,uint32_t flash_addr,uint32_t size_kbyte)

    开始录音

    :param flash_init_area_funcaddr: flash区域擦除函数，函数类型flash_init_area_functype
    :param flash_save_data_funcaddr: flash写函数，函数类型flash_save_data_functype
    :param flash_addr: 使用的flash区域首地址
    :param size_kbyte: 录按键消息的flash大小 in kbyte，只会取size_kbyte值中4kbyte的倍数
    :returns: 错误码
    :retval 0: 成功
    :retval -1: midi综合器未初始化
    :retval -2: flash大小不足4k
    :retval -3: 未注册flash操作函数
    :retval -4: flash操作失败

.. c:type:: int32_t (*flash_init_area_functype)(uint32_t addr,uint32_t size)

    flash区域擦除函数

    :param addr: 写入flash首地址
    :param size: 使用flash区域大小 in byte
    :returns: 错误码
    :retval 0: 擦除成功
    :retval 其它值: 擦除失败
    
.. c:type:: int (*flash_save_data_functype)(uint32_t addr,uint8_t *pbuf,uint32_t len)

    flash区域数据写入函数

    :param addr: 写入flash首地址
    :param pbuf: 待写入数据的首地址
    :param len: 待写入数据的长度 in byte
    :returns: 错误码
    :retval 0: 写入成功
    :retval 其它值: 写入失败

.. c:function:: int32_t player_midi_keyrecord_done()

    录音结束

    :returns: 错误码
    :retval 0: 停止成功
    :retvla -1: midi综合器未初始化
    :note: 按键消息记录完成，停止录音，并将缓存消息全部导入flash

.. c:function:: int32_t player_midi_keyrecord_replay(uint32_t flash_addr)

    录音回放

    :param flash_addr: 记录在flash的按键消息首地址
    :returns: 错误码
    :retval 0: 回放成功
    :retvla -1: midi综合器未初始化
    :note: 将录好的按键信息，回放





