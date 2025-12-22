
#### 下面这个文件是合成数据工作流的input文件, 可以看出MCP server有15个工具
![[Pasted image 20251222091610.png]]

#### Foundry平台生成的合成数据包括两个文件: train and valid, 都是jsonl数据. 其中train dataset 271行, 也就是有271条训练数据! 每一个行是一个训练数据, 结构如下: 
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

