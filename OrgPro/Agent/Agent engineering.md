https://blog.langchain.com/agent-engineering-a-new-discipline/

### Agent工程
一个Agent的表现, 在"本地能跑"和"线上稳定"直接有着巨大的差距. 

有一些公司确实交付了能在线上稳定运行的Agent, 比如
Clay, Vanta, LinkedIn 和 Cloudflare, 这些公司都没有采用传统软件工程的经验, 而是摸索了一个新方案: Agent Engineering (Agent工程). 

(是不是又是在造概念呢? )

什么是Agent工程呢? 就是能够有效解决大模型系统天然的不确定(non-deterministic)属性的一套方法论. 它是一个循环过程, 包括下面几个步骤: Build(构建), Test(测试), Ship(交付), Observe(观测), Refine(修正), Repeat(重复).

(看起来, 跟传统软件的生命周期有类似的地方)

![[Pasted image 20251213182939.png]]

### Agent工程的三个支柱

一套有效的Agent工程系统, 需要三个方面的支撑:

- Product Thinking(产品思考)
	- 定义提示词: 很多时候你需要写上百甚至上千行的长提示词. 这需要优秀的沟通和文字表达能力! 
	- 定义Agent的任务目标
	- 定义评估和测试指标
- Engineering(工程实现)
	- 工具集开发
	- UI/UX设计优化体验, 比如响应结果的流式输出(streaming), 类似Human In the Loop的中断处理(interrupt handling)
	- Agent运行时开发: 长任务执行场景, Human In the Loop中断场景, 记忆管理
- Data Science(数据科学)
	- 构建数据系统评估Agent的性能和稳定性
	- 对用户的使用习惯和异常报警进行数据分析(因为相比, 传统软件系统, Agent系统上用户的行为更加多样和不可预测)

成功实践了Agent工程的公司, 是在原有的团队人员的基础上, 扩充了原有各个角色的职能, 来控制Agent系统的不确定性. 作为软件工程师, 新实践包括:  要开始写提示词, 开发工具集, 为Agent添加可观测性, 优化底层模型.

### Agent工程的必要性!

- 不再有Corner Case的概念, 因为每一个输入都是Corner Case. 
- 传统的debug方法失灵了. 除了问题, 对于bug的定位会更麻烦! 因为很多逻辑发生在大模型内部, 你需要从上下游整体分析, 才能定位问题. 
- By Design(符合预期)的含义发生了变化, Agent的可用性可以是99.99%, 但是它的回答可能是错的! 

### Agent工程的实践

与传统的软件不同, 发布到生成环境, 并不代表合格, 而是刚刚开始, 需要在生成环境监控行为, 获取数据, 进行优化迭代. 







