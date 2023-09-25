ADPCM播放器
======================

.. _音频媒体文件调用方法: media-file.html

.. note::
   
   前置知识： `音频媒体文件调用方法`_

简介
-------------------------

ADPCM文件的播放以辅助播放形式存在，即，当存在MP3，S1A，SILK，MIDI播放时，同时可播放ADPCM。主要用于播放简短的音源，如提示音。

API说明
-------------------------

.. c:function:: void ADPCM_decompression(uint8_t *inputADPCM,int32_t *outputWAVE,uint32_t lenth,uint32_t sacle,ADPCM_parameter_t* ADPCM_basic_parameter);
 
 ADPCM解码

 :param inputADPCM: 输入的ADCPM缓存首地址
 :param outputWAVE: 输出的ADCPM缓存首地址
 :param lenth: 音频的采样点数，对于ADCPM数据为4bit的个数
 :param sacle: adpcm转换后左移位数，范围0-15
 :param ADPCM_basic_parameter: 上一片计算保留给下一片的计算状态值
 :note: 由adpcm解码的数据，会与原outputWAVE中数据叠加输出

.. c:struct:: ADPCM_parameter_t

 ADPCM解码状态结构体

  .. c:member:: int8_t index

   上一次计算，step size表索引序号，初始值为0

  .. c:member:: int32_t new_sample

   上一次计算，采样点的值，初始值由ADPCM文件给出

使用方法
-------------------------

在MP3，S1A，SILK，MIDI的帧回调函数中调用ADPCM_decompression（）解码函数。

注意事项
-------------------------

 ADPCM解码数据不可单独播放，只能与MP3，S1A，SILK，MIDI一起播放。