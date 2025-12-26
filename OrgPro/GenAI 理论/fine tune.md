
微调的微体现在两个方面: 

- 训练数据量的大小
- 调整参数量的大小

LoRA
- First, preserve the pre-trained weights of the model, preventing catastrophic forgetting and ensuring that valuable knowledge from pre-training is retained. 
- Second, it employs efficient rank-decomposition?? 

**SFT** 和 **LoRA** 的关系在于：它们经常一起出现在现代微调流程里，但**不是同一件事**。

- **SFT（Supervised Fine-Tuning，监督式微调）**：一种**训练目标/训练方式 + 数据形式**。通常用带标签的数据（例如“指令 → 回复”的成对样本）做监督学习，让模型更会遵循指令、学习特定风格或完成特定任务。
- **LoRA（Low-Rank Adaptation，低秩适配）**：一种**参数高效微调方法**。它规定了你**如何**更新模型权重：冻结底座模型原有权重，只训练额外加入的低秩适配矩阵（adapter）。
 
- SFT = **训练什么/用什么训练数据** _what training you do_ (supervised learning on curated pairs)
- LoRA = **怎么更新参数/用什么方式实现微调** _how you apply parameter updates_ (efficient adapter-based update)

LoRA 常被用来**更高效地做 SFT**：

- **全量 SFT（Full SFT）**：更新模型的大部分或全部参数。
- **LoRA 版 SFT（SFT with LoRA）**：训练流程仍是监督微调，但只训练 LoRA 的 adapter 参数，底座权重不动。

换句话说，**LoRA 是微调的一种实现策略**，而 **SFT 是微调的一种类型**。

你可以 **做 SFT 时用 LoRA，也可以不用 LoRA**；同时你也可以 **把 LoRA 用在 SFT 之外的训练目标上**

- **SFT + LoRA**（非常常见）：用 adapter 进行指令微调，节省显存/算力，且方便为不同任务切换不同 adapter。
- **SFT（全量微调）**：当你算力充足、希望模型能力迁移更彻底时可能更合适。
- **LoRA 用于其他目标**：例如一些偏好优化方法（类似 DPO 的训练）也可以用 LoRA 来训练 adapter。



