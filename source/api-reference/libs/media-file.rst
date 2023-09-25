.. _media-file-format:

音频媒体文件调用方法
======================

.. _音频转换工具: ../../get-started/audio-converter.html

.. note::
   
   前置知识： `音频转换工具`_

音频转换工具 AudioConvertTool 生成唯一的 ``音频资源文件 media.dat`` ，置于用户工程资源文件夹 *./img* 下，
经过工具链后(Build)与CPU代码组合成单一SoC固件文件(.bin)，用于上位机向NOR Flash烧录。

由于音频转换工具提供不同格式音乐媒体的混合打包能力，资源文件 `media.dat` 也设计了针对性的机制用于区分内部音频。

.. image:: ../../_static/kiwi-media-file.png
  :align: center

上图以一个打包有3段 ``音频内容(Content Block)`` 的资源文件结构为例，解释资源文件的解析方法：

 - 资源文件从起始位置(`offset = 0`)起，放置若干连续排列的 ``信息头(Content Header)``，数目和顺序与音频内容（ `Content Block` ）的数目和排列顺序保持一致。
 - `Content Header` 区域和 `Content Block` 区域之间为一段全0分隔区，其大小与一个 `Content Header` 的大小一致。
 - 每个 `Content Header` 内含一个预定义的结构体实体，用于携带对应 `Content Block` 的信息：
  
  .. c:struct:: resource_t
  
  音频内容信息结构体

   .. c:member:: player_file_attribute_t attribute

     音频内容格式，如MP3、S1A、SILK、MIDI等，通过预设的枚举定义赋值和区分

   .. c:member:: uint8_t name[20]

     音频内容名称，如曲名

   .. c:member:: uint32_t offset

     音频内容相对于资源文件起始 `offset = 0` 位置的偏移
    
   .. c:member:: uint32_t size

     音频内容的字节数大小

  .. c:enum:: player_file_attribute_t

  播放文件属性枚举，资源文件参数 attribute 值选择范围

  - *FileAttribute_None*: 值为0，资源文件，结束属性
  - *FileAttribute_MP3*: 值为1，资源文件为MP3，不区分采样率
  - *FileAttribute_S1A*: 值为2，资源文件为S1A，不区分采样率
  - *FileAttribute_MIDI*: 值为3，资源文件为midi文件
  - *FileAttribute_ADPCM*: 值为14，资源文件为ADPCM文件
  - *FileAttribute_SILK*: 值为15，资源文件为silk，不区分采样率



 - 用户程序通过全局符号 ``__media_file_addr`` 获得音频资源文件 `media.dat` 在SoC寻址空间的绝对起始地址，
   该地址一般被映射至L2-CACHED NOR FLASH空间，用户可以根据需要，套用SDK的宏定义，将其重映射至其他NOR FLASH空间进行访问。
 - 用户程序从音频资源文件的起始地址开始，以轮询的方式遍历 `Content Header` 区域，直至全0的分隔区。
   用以获得 `Content Header` 的总数。
 - 用户程序从特定 `Content Header` 获得 offset 信息，并以 __media_file_addr + offset 的方式计算出特定音频内容在SoC寻址空间的绝对位置，
   供播放器场景的初始化流程(init)调用。

.. note::
   
   音频资源文件 media.dat 内部的音频内容排列顺序，与音频转换工具 AudioConvertTool 的打包顺序保持一致。


示例代码
------------------------------------
 .. code-block:: c

  获取资源文件总音频数量

  typedef struct
  {
    uint32_t attribute;	//1:MP3;2:S1A;3:MIDI
    uint8_t name[20];
    uint32_t offset;
    uint32_t size;
  }resource_t;

  extern char __media_file_addr;
  extern char __media_file_size;
  
  /*********************获取资源文件总音频数量*************************/
  uint32_t addr,size;
  addr = NORC_DATA_UNCACHE_SINGLE((uint32_t)&__media_file_addr);
  size = (uint32_t)&__media_file_size;
  resource_t * res;
  int cnt=0; //记录音频文件数量
  int size_total=0; //记录所有音频文件总大小
  while(1)
  {
    // 根据id获取文件的头的memory地址
    res = (resource_t*)(addr + sizeof(resource_t)*cnt);
    if(res->attribute==0||(sizeof(resource_t)*cnt)>=size)
    {
      break;
    }
    size_total+=res->size;
    debug("%s:%d\n",res->name,res->size);
    cnt++;
  }
  debug("MusicIDNumber:%d\n",cnt);
  debug("size_total:%d byte\n",size_total);

 .. code-block:: c

  获取某一音频文件信息

  /**********************获取某一音频文件信息************************/
  uint32_t addr,size_all,len;
  addr = NORC_DATA_UNCACHE_SINGLE((uint32_t)&__media_file_addr);
  size_all =  (uint32_t)&__media_file_size;
  resource_t * res;
  if(sizeof(resource_t)*id >=size_all)//XXX !!!!id:表示音频文件在资源文件的序号
  {
    return;
  }
  debug("%s:offset:%x size:%d\n",res->name,res->offset,res->size);