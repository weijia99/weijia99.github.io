---
title: langchain使用
date: '2024-07-02 14:32:50'
updated: '2024-10-03 21:07:55'
description: 问题引入现有llm只能对话，无法进行操作，查询相关data如果要换一个llm，那就需要重新更新llm的代码不能实现根据不同任务选择不同的llm无法有记忆，不能查到最新的消息langchain介绍解耦合了，变成prompt，model，chain，agent，vector等多个模块LangCha...
---


# 问题引入


1. 现有llm只能对话，无法进行操作，查询相关data
2. 如果要换一个llm，那就需要重新更新llm的代码
3. 不能实现根据不同任务选择不同的llm
4. 无法有记忆，不能查到最新的消息



# langchain介绍


1. 解耦合了，变成prompt，model，chain，agent，vector等多个模块



<font style="color:rgb(25, 27, 31);">LangChain 提供了对向量数据库的支持，能够把超长的 txt、pdf 等通过大模型转换为 embedding 的形式，存到向量数据库中，然后利用数据库进行检索。这样就可以支持更多长度的输入，解放了 LLM 的优势。</font>

<font style="color:rgb(25, 27, 31);"></font>

<font style="color:rgb(25, 27, 31);"></font>

# <font style="color:rgb(25, 27, 31);">基本模块介绍</font>
最简单的运行模板

```python
DASHSCOPE_API_KEY=""
from langchain_community.llms import Tongyi
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate
llm=Tongyi(temperature=1,dashscope_api_key=DASHSCOPE_API_KEY)

prompt =PromptTemplate.from_template("给我一个很土但是听起来很好养活的{animal}名字")
prompt.format(animal="狗")

chain = LLMChain(llm=llm, prompt=prompt)
chain.run("老鹰")
```

1. 定义llm
2. 定义prompt
3. 定义chain，传入llm还有提示
4. 之后使用run来预测



## model 
这是标准的llm模块，方便介入各种大模型，zhipu，chatgpt，qwen等



```python
DASHSCOPE_API_KEY=""
from langchain_community.llms import Tongyi
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate
llm=Tongyi(temperature=1,dashscope_api_key=DASHSCOPE_API_KEY)
```

```python
from langchain.llms import OpenAI
llm = OpenAI(openai_api_key="...")
```

直接通过llm模块皆可以快速引入各种模型，具体参考官网的llm





### 自定义model
只需要继承llm类，实现_call方法，同时设置一个属性是静态，返回模型的名称。

```python
class   weijia(LLM):
    @property
    def _llm_type(self) -> str:
        return "weijia-ai"
    
    def _call(self,
               prompt:str,
               stop:Optional[List[str]]=None,
               run_manager:Optional[CallbackManagerForLLMRun] = None
               ) -> str:
        pd = prompt.find("吗")
        if pd >=0 :
            return prompt[:pd]+"."
        return "哦。"
```

## prompt
这是提示模块，就是输入的指令，可以分为的下面多个提示词。

> 为什么需要提示词
>

1. 指令，扮演猫娘
2. 案例学习，现在我要你计算1+2=？，我们知道1+1=2，然后。。，最后。。给定一个事例，按照这个事例来进行输出
3. 提问，询问这个文档的主题是什么





```python
prompt =PromptTemplate.from_template("给我一个很土但是听起来很好养活的{animal}名字")
prompt.format(animal="狗")
```

以模板的样子来进行设置提示词，animal就是可以传入的参数，





<font style="background-color:#FBDE28;">案例学习</font>

```python
from langchain.prompts.few_shot import FewShotPromptTemplate
from langchain.prompts.prompt import PromptTemplate

examples = [
    {
        "question":"你好吗？",
        "answer":"帅哥，我很好"
    },
    {
        "question":"今天周几？",
        "answer":"帅哥，今天是周一"
    },
    {
        "question":"今天天气怎么样？",
        "answer":"帅哥，今天天气很好"
    }
]
# 给定的案例，需要包含question和answer字段，然后使用参数的prompt
example_prompt = PromptTemplate(input_variables=["question","answer"],template="Question:{question}\n{answer}")
# prompt 是通过上面传参形成的模板
prompt = FewShotPromptTemplate(example_prompt=example_prompt,examples=examples,suffix="Question: {input}",input_variables=["input"])

print(prompt.format(input="我怎么这么丑?"))
```

使用的是少样本学习，给定example，之后使用fewshot来实现填充，最后一个是末尾加入一个suffix，相当于提问

1. 定义example，使用{}来包括，放入到一个大列表
2. 之后调用普通模板，只需要传入上面的格式
3. 最后是fewshot，传入example，传入链，最后是加入后缀，就是自己要提问的

<font style="background-color:#FBDE28;">与fintune区别：finetue是需要微调llm，fewshot不需要，直接是提示工程</font>

<font style="background-color:#FBDE28;"></font>

<font style="background-color:#FBDE28;"></font>

<font style="background-color:#FBDE28;">聊天提示模板</font>

思路就是把上轮的对话内容也加入到提示里面来

<font style="background-color:#FBDE28;"></font>

```python
from langchain.prompts import (
    ChatPromptTemplate,
    PromptTemplate,
    SystemMessagePromptTemplate,
    AIMessagePromptTemplate,
    HumanMessagePromptTemplate,
)
from langchain.schema import (
    AIMessage,
    HumanMessage,
    SystemMessage
)
template="You are a helpful assistant that translates {input_language} to {output_language}."
system_message_prompt = SystemMessagePromptTemplate.from_template(template)
human_template="{text}"
human_message_prompt = HumanMessagePromptTemplate.from_template(human_template)
chat_prompt = ChatPromptTemplate.from_messages([system_message_prompt, human_message_prompt])

# get a chat completion from the formatted messages
chat_prompt.format_prompt(input_language="English", output_language="French", text="I love programming.").to_messages()
```

<font style="background-color:#FBDE28;"></font>

## 路由提示
> 如何让llm判断是什么类型，然后进行选择哪一个提示。一个是取名字，一个是做数学题，我想让他帮忙进行取名字
>

使用路由提示，和网关作用差不多，gateway选择微服务。

那就需要两个提示，一个提示是gateway提示，还有一个是所有提示的聚合

<font style="background-color:#FBDE28;"></font>

```python
naming_template="""你是一个非常有创意的起名字专家. \
你非常擅长给新生儿们起好听的，富含寓意的名字. \
用中文给出回答 \
这里是问题： \
{input}"""


math_template ="""You are a very good mathematician.You are great at answering math questions.
You are so good because you are able to break down hard problems into their component parts, \
answer the component parts,and then put them together to answer the broader question.
Here is a question:
{input}"""

prompt_infos = [
    {
        "name": "naming",
        "description": "给新生儿起名字",
        "template": naming_template
    },
    {
        "name": "math",
        "description": "解数学问题",
        "template": math_template
    }
]

from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain, ConversationChain

DASHSCOPE_API_KEY=
from langchain_community.llms import Tongyi
llm=Tongyi(temperature=1,dashscope_api_key=DASHSCOPE_API_KEY)
destination_chains = {}
# 这是所有的提示，需要初始化
for p_info in prompt_infos:
    name = p_info["name"]
    prompt_template = p_info["template"]
    prompt = PromptTemplate(template=prompt_template,input_variable=["input"])
    chain =LLMChain(prompt=prompt,llm=llm)
    destination_chains[name] = chain
default_chain = ConversationChain(llm=llm,output_key="text")

# 构建路由器
# 加载路由器
from langchain.chains.router.llm_router import LLMRouterChain,RouterOutputParser
from langchain.chains.router.multi_prompt_prompt import MULTI_PROMPT_ROUTER_TEMPLATE
# 形成路由器
destinations =[f"{p['name']}:{p['description']}" for p in prompt_infos]
destination_str = ",".join(destinations)
# 也是需要放入llm，llm根据描述来判断选择哪一个name，之后dest根据name获取对应的提示
router_template = MULTI_PROMPT_ROUTER_TEMPLATE.format(destinations=destination_str)
router_prompt = PromptTemplate(template=router_template,input_variable=["input"],output_parser=RouterOutputParser())
# 这个只是进行选择哪一个prompt
router_chain = LLMRouterChain.from_llm(llm,router_prompt)
from langchain.chains.router import MultiPromptChain
# 定义路由器+所有的prompt
chain = MultiPromptChain(router_chain=router_chain,destination_chains=destination_chains,default_chain=default_chain,verbose=True)
print(chain.run("计算100的1/2次方"))
```



1. 定义所有的prompt，有dest，根据dest【name】来获取
2. 定义router路由，根据描述来进行选择name，之后使用dest的name来进行获取 
3. 然后放入到运行的链<font style="color:rgb(35, 41, 48);background-color:rgb(250, 250, 250);">chain</font><font style="color:rgb(225, 0, 35);background-color:rgb(250, 250, 250);">=</font><font style="color:rgb(35, 41, 48);background-color:rgb(250, 250, 250);">MultiPromptChain</font><font style="color:rgb(38, 38, 38);background-color:rgb(250, 250, 250);">(</font><font style="color:rgb(35, 41, 48);background-color:rgb(250, 250, 250);">router_chain</font><font style="color:rgb(225, 0, 35);background-color:rgb(250, 250, 250);">=</font><font style="color:rgb(35, 41, 48);background-color:rgb(250, 250, 250);">router_chain</font><font style="color:rgb(38, 38, 38);background-color:rgb(250, 250, 250);">,</font><font style="color:rgb(35, 41, 48);background-color:rgb(250, 250, 250);">destination_chains</font><font style="color:rgb(225, 0, 35);background-color:rgb(250, 250, 250);">=</font><font style="color:rgb(35, 41, 48);background-color:rgb(250, 250, 250);">destination_chains</font><font style="color:rgb(38, 38, 38);background-color:rgb(250, 250, 250);">,</font><font style="color:rgb(35, 41, 48);background-color:rgb(250, 250, 250);">default_chain</font><font style="color:rgb(225, 0, 35);background-color:rgb(250, 250, 250);">=</font><font style="color:rgb(35, 41, 48);background-color:rgb(250, 250, 250);">default_chain</font><font style="color:rgb(38, 38, 38);background-color:rgb(250, 250, 250);">,</font><font style="color:rgb(35, 41, 48);background-color:rgb(250, 250, 250);">verbose</font><font style="color:rgb(225, 0, 35);background-color:rgb(250, 250, 250);">=</font><font style="color:rgb(38, 38, 38);background-color:rgb(250, 250, 250);">True)</font>

<font style="color:rgb(38, 38, 38);background-color:rgb(250, 250, 250);"></font>

## <font style="color:rgb(38, 38, 38);background-color:rgb(250, 250, 250);">数据连接</font>
> 大模型只有语言知识，但是如果是要外接入数据，那就是需要进行读取数据，例如pdf等数据，这就是需要是用data connection来进行解码数据
>

例如pdf还有csv等数据。

最终输出的是doc模块



如果pdf页数太大，超过llm的token，这个时候还需要进去切词。这个就是<font style="background-color:#FBDE28;">分割器。可以参考MapReduce的操作</font>



整体流程

1. 输入pdf
2. 进行分割
3. 之后使用doc来进行embedding
4. 最后存放到向量数据库



```python
from langchain.document_loaders import PyPDFLoader
loader = PyPDFLoader("./se230522013.pdf")
# documents = loader.load()

# 定义分割器
from langchain.text_splitter import RecursiveCharacterTextSplitter
# 表示的分割的块是100，有20个进行重合
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=100,
    chunk_overlap=20,
    length_function=len
)
pages =loader.load_and_split(text_splitter)
# 进行切分文档
# 编码器
from langchain_community.embeddings import ModelScopeEmbeddings
model_id = "damo/nlp_corom_sentence-embedding_english-base"
embeddings = ModelScopeEmbeddings(model_id=model_id)

# 放入db
from langchain.vectorstores import Chroma
db = Chroma.from_documents(pages, embeddings)
query = "Artificial Intelligence"
docs = db.similarity_search(query, k=5)
```

## memory


## agent
之前的使用llmmath来进行解决数学题，学会调用工具

