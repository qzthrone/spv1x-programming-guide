.. _audio-effect-lib:

音效
======================
噪声门限 (Noise Gate)
-------------------------
    .. image:: ../../_static/kiwi-ae-drc-noisegate.png
        :align: center

API说明
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. c:function:: void ae_drc_noisegate_init( ae_drc_noisegate_init_t *drc_ng_init_param)

    DRC音效之noise gate初始化

    :param drc_ng_init_param: ae_drc_noisegate_init_t 类型指针
    :returns: 无

.. c:struct:: ae_drc_noisegate_init_t

    noise gate初始化参数

        .. c:member:: uint32_t holdtime

            保持时间是指当输入电平低于阈值时，增益在开始向稳态值下降之前保持的时间。参数范围[0,2)，单位秒，定点数q30
    
        .. c:member:: uint32_t ltrhold
    
            开始作用输入信号的最低等级。采样值，参数范围[0,32767]
    
        .. c:member:: uint32_t utrhold

            作用输入信号的最高等级。采样值，参数范围[0,32767]

        .. c:member:: uint32_t release

            当输入超过阈值范围时，应用增益从其最终值的90%下降到10%所需的时间。参数范围[0,2)，单位秒，定点数q30

        .. c:member:: uint32_t attack

            当输入低于阈值范围时，应用增益从其最终值的10%上升到90%所需的时间。参数范围[0,2)，单位秒，定点数q30

        .. c:member:: uint32_t filter_freq

            用于包络计算低通滤波器的截止频率，单位hz，范围 (0,fs/2)

        .. c:member:: uint32_t fs

            音频采样频率，单位hz

.. c:function:: void ae_drc_noisegate(int16_t *in,int16_t *out,int32_t datalen)

    DRC音效之noise gate处理

    :param in: 输入数据地址
    :param out: 输出数据地址
    :param datalen: 输入输出数据音频点数
    :returns: 无
    :note: 输入输出的数据地址必不能是同一块内存空间
    :note: 输入输出数据点数必须为4的倍数

使用方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    1. 调用ae_drc_noisegate_init()函数初始化noise gate
    2. 调用ae_drc_noisegate()函数处理音频数据

参量均衡器 (Parametric Equalizer)
-------------------------------------
    .. image:: ../../_static/kiwi-ae-eq.png
        :align: center

API说明
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. c:function:: void ae_eq_init(const int32_t coeff[AE_EQ_SERIES][5])

    音效之eq初始化

    :param coeffs: 多段eq的iir系数，每段iir系数为b0,b1,b2,-a1,-a2，定点数q30
    :returns: 无
    :note: 默认八段iir级联

 .. c:function:: void ae_eq(int32_t *in,int32_t *out,int32_t datalen)

    音效eq处理

    :param in: 输入音频地址
    :param out: 输出音频地址
    :param datalen: 输入输出数据长度
    :return: 无
    :note: 输入输出地址可以是同一块内存空间

使用方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    1. 调用ae_eq_init()函数初始化EQ
    2. 调用ae_eq()函数处理音频数据