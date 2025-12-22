
#### 下面这个文件是合成数据工作流的input文件, 可以看出MCP server有15个工具
![[Pasted image 20251222091610.png]]

Foundry平台生成的合成数据包括两个文件: train and valid, 都是jsonl数据. 其中train dataset 271行, 也就是有271条训练数据! 每一个行是一个训练数据, 结构如下: 
![[Pasted image 20251222093356.png]]
#### 其中messages是system, user, assistant, tool的chat记录, 构成一个conversation

![[Pasted image 20251222093743.png]]

#### 对生成的训练数据做一个基本的统计分析, 结果如下: 
![[Pasted image 20251222095146.png]]

- 有两列: num_turns指一次对话的轮数(turn), tool_calls指一次对话中工具调用的次数(call)
可以看出, 对话的轮数平均值是16.47, 而每次对话平均进行3.93次工具调用, 最多(max)6次! 

数据可视化可以看的更清楚
![[Pasted image 20251222114500.png]]
![[Pasted image 20251222114609.png]]
![[Pasted image 20251222114634.png]]

#### 可以尝试分析一下tool_calls次数最多的一个对话, 共6个turn

SYSTEM
    "E-Commerce Order Management Agent Policy ..."
          |
          v
USER
    "Hi, I want to exchange my blue hoodie for a red one in medium size.
     Can you help me with that?"
          |
          v
ASSISTANT
    Asks to authenticate user:
    "Please provide your email address for verification."
          |
          v
USER
    "Sure, my email is emily.park88@example.com."
          |
          v
ASSISTANT  --(calls find_user_id_by_email)-->
    TOOL CALL: find_user_id_by_email(...)
          |
          v
**TOOL**
    **find_user_id_by_email**
          |
          v
ASSISTANT
    Thanks user, asks about order number or lookup:
    "Do you remember the order number, or should I look up all your orders?"
          |
          v
USER
    "I do not remember the order number, sorry. Please look it up."
          |
          v
ASSISTANT  --(calls get_user_details)-->
    TOOL CALL: get_user_details(...)
          |
          v
**TOOL**
    **get_user_details**
          |
          v
ASSISTANT  --(calls get_order_details)-->
    TOOL CALL: get_order_details(...)
          |
          v
**TOOL**
    **get_order_details**
          |
          v
ASSISTANT
    Confirms found delivered blue hoodie order:
    "To confirm, you want to exchange this for a red Classic Hoodie in medium?"
          |
          v
USER
    "Yes, that's correct. Please exchange it for the red one in the same size."
          |
          v
ASSISTANT  --(calls list_all_product_types)-->
    TOOL CALL: list_all_product_types(...)
          |
          v
**TOOL**
    **list_all_product_types**
          |
          v
ASSISTANT  --(calls get_product_details)-->
    TOOL CALL: get_product_details(...)
          |
          v
**TOOL**
    **get_product_details**
          |
          v
ASSISTANT
    Confirms availability and summarizes intended exchange:
    "Red Classic Hoodie (Medium) is available, same price.
     I will exchange the blue hoodie from order #W0005862 for a red hoodie
     in Medium, and your original payment method will be used for any
     adjustments. Should I proceed?"
          |
          v
USER
    "Yes, please go ahead and process the exchange."
          |
          v
ASSISTANT  --(calls exchange_delivered_order_items)-->
    TOOL CALL: exchange_delivered_order_items(...)
          |
          v
**TOOL**
    **exchange_delivered_order_items**
          |
          v
[END OF GIVEN TRACE]


#### 筛选与整理合成数据集

具体来说，我们筛选满足如下要求的训练数据：

- 调用一个 find_* 工具（例如 find_user_id_by_email、find_user_id_by_name_zip）； 
- 随后立即发起一次正确的 get_user_details 调用； 
- 正确传递并保持工具调用参数一致（例如 user_id 前后一致、无凭空编造/幻觉参数）。

###   工具调用（Per-Tool-Call）的评估设计

合成测试数据集最初由多轮对话构成，每个场景中包含多次工具调用。

为了评估，我们将其扩展为一个专门的“按工具调用”数据集：

- 对每段对话，我们定位其中的每一次工具调用；
- 对每一次工具调用，我们创建一个独立的评估样本，其中：
    - **messages** 包含截至该工具调用之前（但不包括该工具调用本身）的对话历史；
    - **expected_output** 只包含接下来**应该发生的那一次**助手工具调用。

从概念上讲，每条评估记录都在问：

> “基于目前为止的对话，助手接下来应该发起哪一次工具调用？”

这样做带来：

- 细粒度的、按单次工具调用的评估；
- 数以千计彼此独立的评估样本；
- 清晰、无歧义的正确性判定标准。


#### 将合成测试集扩展为评估数据集

合成数据生成器会产出一个测试数据集(valid.jsonl)，其中每条记录都是一段完整的多轮对话，并且可能包含多次工具调用。  
为了评估，我们希望在**每一个工具调用的决策点**测试模型：

> “基于目前为止的对话，助手接下来应该生成哪一次工具调用？”

为此，我们将测试数据集进行扩展，使得**每一次工具调用**都成为一条独立的评估记录。

我们为此提供了一个脚本：`convert_to_evals.py`

该脚本会：

- 加载合成测试数据集（`valid.jsonl`）
- 扫描每段对话的 `messages`，查找assistant的工具调用消息
- 对于每一次工具调用：
    - 将对话截断到该工具调用发生之前
    - 从助手消息中移除该工具调用
    - 将该工具调用放入 `expected_output.tool_calls`
- 将扩展后的数据集写出为可用于评估的文件，例如 `eval.jsonl`

我们会用这个eval.json文件, 来对模型(基座模型和微调)进行性能评估!!! 

![[Pasted image 20251222142232.png]]
(eval.json文件中的一条数据, messages是截取之后的对话内容, expected_output里预期结果, tools是mcp服务的toolset列表)

### 基于Python的评分器（确定性，不涉及LLM）

该评估使用纯Python评分器，而不是基于模型的评分器。这样可以使评估：

- **确定性** — 相同的输入总是产生相同的分数
- **快速** — 不需要额外的模型调用
- **透明** — 你可以准确检查为什么一个预测通过或失败
- **专注** — 我们只评估下一个工具调用的正确性

评分器(grader)接受两个对象：

- **item** → 来自`eval.jsonl`的评估记录，包含期望的工具调用
- **sample** → 由Microsoft Foundry捕获的模型输出（`output_tools`）

评分器执行三个步骤：

1. **提取期望的工具调用**  
    `item["expected_output"]["tool_calls"]`  
    参数以JSON字符串存储并进行标准化。
    
2. **提取模型预测的工具调用**  
    `sample["output_tools"]`  
    在比较之前进行解析和标准化。
    
3. **评分工具调用**
    
    - **1.0** → 正确的工具和正确的参数
    - **0.5** → 正确的工具，但参数不同
    - **0.0** → 错误的工具、缺失的工具或格式错误的调用

这种评分方法故意设计得轻量且精确 — 它准确测量我们在监督微调中关心的内容：模型是否选择了正确的下一个工具并提供了正确的参数。

Note: 标题里写的: 确定性, 不涉及LLM, 这句话比较有歧义. 整个evaluation的过程肯定是有LLM参与的, 因为评估的就是大模型的效果. 这里说的不涉及LLM, 是指grade操作, 是python程序里实现的. 

eval.json文件的每一条数据的messages字段的内容当作prompt发给对应的model, model会返回一个结果, 这个结果就是grader的第二个参数sample: 由Microsoft Foundry捕获的模型输出（`output_tools`）

eval.json文件的每一条数据的expected_output是预期结果, 作为评分器(grader)的第二个参数. 

![[Pasted image 20251223060056.png]]

grade_tool_calls就是按照文本内容, 直接比较实际结果和预期结果. 这个比较的过程, 没有LLM参与, 单纯的脚本. (因为是文本内容的比价, 其实可以由LLM完成, 但是这样eval的结果就没有确定性了!!!)

