
# https://medium.com/@sulbha.jindal/why-do-llms-have-latency-296867583fd2

大模型的响应速度直接影响了用户体验, 正义可以"虽迟但到", 但是给付费用户的服务和响应确不行, 商业和技术竞技就是这么严酷!

**Latency是什么?**

中文里一般翻译成延迟, 也有叫时延的, 都是一个意思, 描述一个请求得到响应的时间. 做过分布式应用或者网络编程的都知道低时延的重要性. 

**LLM Latency**

影响大模型时延的主要原因包括下面几个: 

- 模型大小: 参数越大的模型, 一般都需要更多的时间来计算结果. 
- Tokenization: 分词的过程是必须的, 输入的query越复杂, 分词过程也越费时. 现在讲究的Context Engineering, 力求提供更丰富的上下文, 让这个部分更为复杂了. 
- 顺序处理: 生成式模型采用了自回归(autoregressive)技术, 新token的生成必须依赖前序的计算结果, 这也是一个瓶颈
- 硬件: 这个不用解释了, 算力层面的限制
- 输出: 除了输入数据之外, 输出的token也需要对用的处理. 

如果我们按照大模型的运行原理来分析Latency, 可以分为两个阶段: Prompt Processing(输入token处理)和Autoregressive Generation(输出token生成). 而生成阶段的Latency要高出上百倍! 

**Prompt Processing**

- 处理方式(Processing Style): 由于Transformer架构采用了注意力机制(self-attention),  所有input token可以并行处理 
- 时延模式(Latency Pattern): 所有token统一被模型处理
- 单token成本(Per-Token Cost): 非常低 (0.01 ~ 0.05ms)
- 可扩展性(Scalability): 和token数量呈线性增长关系

这个过程就像一次性读完一整段文字，而不是逐字阅读, 无论是100个字还是1000个字, 模型可以一次性处理你的整个提示词. 

**Autoregressive Generation**

- 处理方式(Processing Style): 顺序执行, 一个一个token处理.
- 时延模式(Latency Pattern):  前一个toke处理完了, 会当作下一个的input, 所谓自回归(Autoregressive)
- 单token成本(Per-Token Cost): 非常高(5~20ms)
- 可扩展性(Scalability): 和token数量呈线性增长关系, 但是高出几个数量级

这个过程就像逐字写回应，每个新词都依赖于你之前写的所有内容。这个过程无法并行化——你必须等待每个词元生成后，才能生成下一个。

**降低Latency的方案** 

- Key-Value(KV) Caching: 主要针对计算注意力的过程, 如果没有Cache, 那么每次生成一个新token, 都需要把之前所有的token的关系按照注意力机制重算一遍. token挨个比对, 所以是O(n^2)的计算量. 有了Caching, 就可以把计算过的key-value缓存起来, 避免重复计算. 
- Mixture of Experts(MoE): 把整个模型分区, 对于单个token, 只激活相关部分, 避免全局计算, 保证质量的同时提升响应速度.
- 



