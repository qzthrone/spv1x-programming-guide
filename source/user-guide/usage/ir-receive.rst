IR解码
===============================

操作原理
-------------------------------

根据红外协议的对各功能位的定义，对红外一体化接收头输出的脉冲波形进行捕获测量，
然后将测量的脉冲宽度，翻译为对应的功能位（如引导码，数据位0，数据位1）。

实现方式
-------------------------------

对于不同的红外协议，对接收信号脉宽测量的实现方式有所区别。

通常，红外帧由引导码+数据位构成。
在不同的红外协议中，引导码的高低电平宽度与数据位的高低电平宽度有着明显的差异，因此其引导码和数据位的区分较为简单。

对于数据位，数据位0和数据位1的编码方式（最终也表现为数据位的高低电平宽度）较为接近，
需要根据其具体的差异点进行针对性的区分。比较典型的红外数据位表示方式主要是3种：

 1. 数据位的低电平宽度不变，以数据位的总宽度表示0和1。

    .. image:: ../../_static/kiwi-ir-data-format-1.png
      :align: center

    NEC红外协议就属于这种类型。对于这种类型的红外协议，可以使用下降沿捕获功能，完成对其数据位的周期进行测量。

    每次下降沿捕获完成后，就得到了当前数据位的周期宽度，然后根据对应的协议，将周期数据与数据位进行匹配。

 2. 数据位的总宽度不变，以数据位高低电平的相对宽度表示0和1。

    .. image:: ../../_static/kiwi-ir-data-format-2.png
      :align: center

    对于这种类型的红外协议，可以使用Fall-Rise捕获功能，完成对其数据位的低电平宽度进行测量。

    Fall-Rise捕获功能会针对相邻下降沿和上升沿之间的低电平进行捕获，捕获完成后，就可以根据捕获到的低电平宽度，进行数据位的匹配。

 3. 以曼切斯特编码表示的数据位。

    .. image:: ../../_static/kiwi-ir-data-format-3.png
      :align: center

    对于这种类型的红外协议，可以使用双边沿（Both）捕获功能，将高电平和低电平的宽度依次进行捕获和存储，用于后续软件解码。

    为了增强捕获结果的准确性，需要在捕获完成中断里读取红外信号的电平，从而确定本次捕获的详细信息：如果读取到高电平，说明是上升沿触发捕获，捕获值代表低电平的宽度；如果读取到低电平，说明是下降沿触发捕获，捕获值代表的是高电平的宽度。这些电平信息有助于后续的软件解码。

    根据曼切斯特编码的特点，数据位阶段捕获到的高低电平宽度都只有T和2T两种情况，这个特点可以用于快速验证捕获值是否合理。

    以如下波形为例，假定高电平到低电平的跳变表示数据位0，低电平到高电平的跳变表示数据位1，整个数据位的解码过程如下：

    .. image:: ../../_static/kiwi-ir-data-format-example.png
      :align: center

    a. 对波形进行双边沿捕获，其捕获的结果如下。

       .. image:: ../../_static/kiwi-ir-table-1.png
         :align: center

    b. 根据捕获值和对应的电平，将捕获值为2T的电平进行展开。

       .. image:: ../../_static/kiwi-ir-table-2.png
         :align: center

    c. 每2个电平一组，进行电平的跳变分析，解码出对应的数据位。

       .. image:: ../../_static/kiwi-ir-table-3.png
         :align: center

注意事项
-------------------------------

 1. SPV1x定时器的计数核心为16bit，在有效功能位（引导码和数据位）捕获过程中，需要确保定时器计数不会溢出。合理使用TIMER的预分频功能，可以避免捕获过程中的溢出情况。此外，对于捕获到的两帧红外之间的空闲，捕获值的溢出不会影响软件解码，因为紧接着的引导码会让软件解码状态机跳过此处的错误。
 2. 使用TIMER的消抖（DEBOUNCE）功能，可以滤除信号的高频杂波，提高捕获的稳定性。
 3. TIMER的捕获中断优先级不能太低，中断的延时需要低于1个最小的捕获脉宽的时间宽度，以避免丢失捕获数据。
 4. 解码的软件状态机需要具有一定的健壮性，以应对可能损坏的红外帧（如其他红外干扰、红外帧信号中途遮挡）。以下几点可提升解码效果：
    
     a. 充分利用引导码。
        引导码是一帧红外开始的关键信号，每解码一个功能位脉宽之前，都尽可能对此功能位脉宽进行一次引导码判断。在解码一帧红外的数据位之前，需要确认先收到过引导码。如果在解码一帧红外帧的中途，再次识别到引导码，应放弃当前帧的解码，开始新的一帧的解码。
     b. 对每一个数据位的捕获值合理性进行判断。
        接收到的数据位的宽度与理论值通常存在一定的偏差，软件需要进行一定的兼容，但同时需要进行异常判断。比如NEC协议中，数据位的低电平宽度为560us，实际捕获的低电平宽度可能在560±100us。对于特别异常的捕获值，则判定为红外帧损坏，软件应放弃当前帧的解码，重新等待新的引导码。