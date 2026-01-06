
了解过大模型原理的同学都知道, 我们可以通过调整几个关键的参数,比如temperature, top_p之类的调节大模型的行为. 

那么把temperature设为0, 按照大模型的贪心解码策略, 输出的结果就应该是确定的了吧? 同样一个prompt跑两遍, 应该得到一毛一样的结果了吧?!

但是理论和实际是有差距的, 虽然你做了上述的设置, 但是实际上你还是会得到多多少少有一些差异的答案. 这个差异, 不是来自模型, 而是来自GPU!

https://medium.com/@sulbha.jindal/why-llms-arent-truly-deterministic-7cc7b451fdab

