
- SFT for structured behaviors (classification, extraction, tool-calling patterns)
- Distillation when you want to compress behavior from a bigger teacher model to a cheaper student model
- Direct Preference Optimization(DPO): trains models to prefer certain types of response over others by learning from comparative feedback, without requiring a separate reward model. 
   note: 训练集数据的output有两个值, 形成对比, 二选一的效果
- Reinforcement fine-tuning(RFT) when you care about multi-step reasoning and end-to-end outcomes.




