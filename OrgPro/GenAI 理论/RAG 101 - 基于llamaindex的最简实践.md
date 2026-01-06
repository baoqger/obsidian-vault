


![[Pasted image 20260104161113.png]]

RAG核心分为: **建立索引**和**检索生成**两个部分! 



![[Pasted image 20260104162757.png]]


建立索引包括四个步骤: 

- 文档解析 => 读取加载数据
- 文本分段 => 分割段落
- 文本向量化 => 嵌入计算进行数字化表示(文本向量化), 相似度比较
- 存储索引 => 对向量化的文本进行持久化存储(借助矢量数据库), 可以避免对上述步骤的重复, 提升响应速度

![[Pasted image 20260104163145.png]]

检索生成包括两个步骤: 
- 检索 => 把query进行文本向量化, 通过矢量计算(consine或者点积)评估相似性, 找出相关段落. **检索是RAG应用中最重要的环节.** 除了采用更强大的嵌入模型, 也有很多优化方法, 比如: rerank重排, 句子窗口检索等方法.   
- 生成: 问题 + 检索出的文本段落 = 最终提示词, 利用大模型的总结能力, 回答问题. **此处的提示词模板很重要**

基于LlamaIndex的具体实现: 


```
# 导入依赖
from llama_index.embeddings.dashscope import DashScopeEmbedding,DashScopeTextEmbeddingModels
from llama_index.core import SimpleDirectoryReader,VectorStoreIndex
from llama_index.llms.openai_like import OpenAILike



#1 print("解析文件")
# LlamaIndex提供了SimpleDirectoryReader方法，可以直接将指定文件夹中的文件加载为document对象，对应着解析过程
documents = SimpleDirectoryReader('./docs').load_data()

#2 print("切片, 矢量化, 创建索引")
# from_documents方法包含切片与建立索引步骤
index = VectorStoreIndex.from_documents(
    documents,
    # 指定embedding 模型
    embed_model=DashScopeEmbedding(
        # 你也可以使用阿里云提供的其它embedding模型：https://help.aliyun.com/zh/model-studio/getting-started/models#3383780daf8hw
        model_name=DashScopeTextEmbeddingModels.TEXT_EMBEDDING_V2
    ))
    
#3 print("创建提问引擎...")
query_engine = index.as_query_engine(
    # 设置为流式输出
    streaming=True,
    # 此处使用qwen-plus模型，你也可以使用阿里云提供的其它qwen的文本生成模型：https://help.aliyun.com/zh/model-studio/getting-started/models#9f8890ce29g5u
    llm=OpenAILike(
        model="qwen-plus",
        api_base="https://dashscope.aliyuncs.com/compatible-mode/v1",
        api_key=os.getenv("DASHSCOPE_API_KEY"),
        is_chat_model=True
        ))
        
#4 print("正在生成回复...")
streaming_response = query_engine.query('我们公司项目管理应该用什么工具')
print("回答是：")
# 采用流式输出
streaming_response.print_response_stream()
```

在没有向量数据库的时候, 可以简单地把索引存储到本地的文件系统

```
index.storage_context.persist("knowledge_base/test")
print("索引文件保存到了knowledge_base/test")
```

在需要的时候, 可以再加载回来, 需要指定当时创建索引的时候所使用的嵌入模型, 这样后续检索的步骤可以保证矢量化的行为一致! 

```
from llama_index.core import StorageContext,load_index_from_storage
storage_context = StorageContext.from_defaults(persist_dir="knowledge_base/test")
index = load_index_from_storage(storage_context,embed_model=DashScopeEmbedding(
        model_name=DashScopeTextEmbeddingModels.TEXT_EMBEDDING_V2
    ))
print("成功从knowledge_base/test路径加载索引")
```

之后可以照旧使用这个index 

```
print("正在创建提问引擎...")
query_engine = index.as_query_engine(
    # 设置为流式输出
    streaming=True,
    # 此处使用qwen-plus模型，你也可以使用阿里云提供的其它qwen的文本生成模型：https://help.aliyun.com/zh/model-studio/getting-started/models#9f8890ce29g5u
    llm=OpenAILike(
        model="qwen-plus",
        api_base="https://dashscope.aliyuncs.com/compatible-mode/v1",
        api_key=os.getenv("DASHSCOPE_API_KEY"),
        is_chat_model=True
        ))
print("正在生成回复...")
streaming_response = query_engine.query('我们公司项目管理应该用什么工具')
print("回答是：")
streaming_response.print_response_stream()
```


