
基于Ragas搭建RAG 自动化评测体系

为了系统化地评测RAG系统, 业界推出了一些非常实用的开源自动化框架, 会对以下几个维度进行评估:
- 召回质量(retrieval quality): RAG系统是否检索到了正确且相关的文档片段? 
- 答案忠诚度(faithfulness):  生成的答案是否完全基于检索到的上下文? 
- 答案相关性(answer relevance): 生成的答案是否准确地回答了用户地问题?
- 上下文利用率(context utilization/efficiency): 大模型是否有效地利用了所有提供给它地上下文信息? 

### Ragas

这是领域里非常出名的一个框架，它的核心思想是“让大模型来帮助你评估RAG系统”。在评测时，Ragas 会调用一个大模型作为评测专家来阅读你的问题、RAG检索到的上下文和生成的答案，然后根据预设的指标给出分数。Ragas的评估指标高度契合RAG系统的痛点，主要包括：

- 整体回答质量的评估：
    - Answer Correctness，用于评估 RAG 应用生成答案的准确度。
- 生成环节的评估：
    - Answer Relevancy，用于评估 RAG 应用生成的答案是否与问题相关。
    - Faithfulness，用于评估 RAG 应用生成的答案和检索到的参考资料的事实一致性。
- 召回阶段的评估：
    - Context Precision，用于评估 contexts 中与准确答案相关的条目是否排名靠前、占比高（信噪比）。
    - Context Recall，用于评估有多少相关参考资料被检索到，越高的得分意味着更少的相关参考资料被遗漏。