.. _prng-api-ref:

伪随机数生成
======================

.. c:function:: void srand(uint32_t seed)

  设置随机数种子，初始种子值为2。

  :param seed: 自然数池大小，比如range=10表示从0-9的连续自然数池中选择。range不超过256。
  :returns: 无
  :note: 为保证随机性，推荐读取CPU内核cycle寄存器(CSR_MCYCLE)的当前值作为seed输入。

.. c:function:: int32_t rand(void)

  返回一个随机数，范围[0-0x7ffffffe]。

  :returns: 随机32位自然数。
  :note: 后续（按需）可以通过取模操作(mod)将随机数范围进一步缩小。

.. c:function:: void random_ascending_subset(int32_t range, int32_t n, uint8_t * output)

  在从0开始的N个连续自然数中选择M个不重复的数字，按照从小到大的顺序排列。

  :param range: 自然数池大小，比如range=10表示从0-9的连续自然数池中选择。range不超过256。
  :param n: 需要选择的不重复数字的总数。
  :param output: 被选中的数字存放于output数组中，有效元素个数由n决定。output的数组的大小应不小于min{range, max(n)}。
  :returns: 无

.. c:function:: void random_partial_shuffle(uint8_t * pool, int32_t size, int32_t n)

  从元素池pool中随机抽取n个元素, 交换至元素池头部位置。

  :param pool: 元素池，元素为0-255的8bit无符号数，无初始顺序强制要求。
  :param size: 元素池大小。
  :param n: 需要抽取的元素数目。
  :returns: 无
  :note: 该函数执行后，元素池pool中起始n个元素为抽取结果。



   


