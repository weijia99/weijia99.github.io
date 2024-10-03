---
title: nlp任务
date: '2024-07-03 15:15:24'
updated: '2024-07-03 17:02:50'
cover: 'https://cdn.nlark.com/yuque/0/2024/png/26023389/1719992183415-03f15df5-3638-49ca-9cfc-3618eae82902.png'
description: 情感类别分类整体的思路就是，进行数据清洗，然后设置编码的时候，把fp函数首先经过tokenlization，之后设置label是数据集上的label，然后进行返回。使用分类模型。之后的评估函数是f1还有acc，然后设置trainer读取ds分割ds对数据集使用map进行fp变换设置model之...
---
# 情感类别分类


整体的思路就是，进行数据清洗，然后设置编码的时候，把fp函数首先经过tokenlization，之后设置label是数据集上的label，然后进行返回。使用分类模型。之后的评估函数是f1还有acc，然后设置trainer



1. 读取ds
2. 分割ds
3. 对数据集使用map进行fp变换
4. 设置model
5. 之后设置评估acc
6. 然后设置trainer

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification, Trainer, TrainingArguments
from datasets import load_dataset
dataset = load_dataset("csv", data_files="./ChnSentiCorp_htl_all.csv", split="train")
dataset = dataset.filter(lambda x: x["review"] is not None)
dataset


datasets = dataset.train_test_split(test_size=0.1)
datasets
import torch

tokenizer = AutoTokenizer.from_pretrained("hfl/chinese-macbert-large")

def process_function(examples):
    tokenized_examples = tokenizer(examples["review"], max_length=32, truncation=True, padding="max_length")
    tokenized_examples["labels"] = examples["label"]
    return tokenized_examples

tokenized_datasets = datasets.map(process_function, batched=True, remove_columns=datasets["train"].column_names)
tokenized_datasets


model = AutoModelForSequenceClassification.from_pretrained("hfl/chinese-macbert-large")


def eval_metric(eval_predict):
    predictions, labels = eval_predict
    predictions = predictions.argmax(axis=-1)
    acc = acc_metric.compute(predictions=predictions, references=labels)
    f1 = f1_metirc.compute(predictions=predictions, references=labels)
    acc.update(f1)
    return acc
    
train_args = TrainingArguments(output_dir="./checkpoints",      # 输出文件夹
                               per_device_train_batch_size=1,   # 训练时的batch_size
                               gradient_accumulation_steps=32,  # *** 梯度累加 ***
                               gradient_checkpointing=True,     # *** 梯度检查点 ***
                               optim="adafactor",               # *** adafactor优化器 *** 
                               per_device_eval_batch_size=1,    # 验证时的batch_size
                               num_train_epochs=1,              # 训练轮数
                               logging_steps=10,                # log 打印的频率
                               evaluation_strategy="epoch",     # 评估策略
                               save_strategy="epoch",           # 保存策略
                               save_total_limit=3,              # 最大保存数
                               learning_rate=2e-5,              # 学习率
                               weight_decay=0.01,               # weight_decay
                               metric_for_best_model="f1",      # 设定评估指标
                               load_best_model_at_end=True)     # 训练完成后加载最优模型
from transformers import DataCollatorWithPadding

# *** 参数冻结 *** 
for name, param in model.bert.named_parameters():
    param.requires_grad = False

trainer = Trainer(model=model, 
                  args=train_args, 
                  train_dataset=tokenized_datasets["train"], 
                  eval_dataset=tokenized_datasets["test"], 
                  data_collator=DataCollatorWithPadding(tokenizer=tokenizer),
               
                  compute_metrics=eval_metric)

trainer.train()
trainer.predict(tokenized_datasets["test"])
sen = "我觉得这家酒店不错，饭很好吃！"
id2_label = {0: "差评！", 1: "好评！"}
model.eval()
with torch.inference_mode():
    inputs = tokenizer(sen, return_tensors="pt")
    inputs = {k: v.cuda() for k, v in inputs.items()}
    logits = model(**inputs).logits
    pred = torch.argmax(logits, dim=-1)
    print(f"输入：{sen}\n模型预测结果:{id2_label.get(pred.item())}")
```

# 命名实体识别
![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/d55810984c2541061aed1f4bb128b60a.png?token=AGECE2J4QN62WYKECX4XABDG72WSY)

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/5ac847a86e1181eed2f3619df8cd53be.png?token=AGECE2NK2ZMF452C2WY4I73G72WS4)

<font style="color:rgb(0, 0, 0);">先列出来BIOES分别代表什么意思：</font>

+ <font style="color:rgb(0, 0, 0);">B，即Begin，表示开始</font>
+ <font style="color:rgb(0, 0, 0);">I，即Intermediate，表示中间</font>
+ <font style="color:rgb(0, 0, 0);">E，即End，表示结尾</font>
+ <font style="color:rgb(0, 0, 0);">S，即Single，表示单个字符</font>
+ <font style="color:rgb(0, 0, 0);">O，即Other，表示其他，用于标记无关字符</font>

<font style="color:rgb(0, 0, 0);">将“小明在北京大学的燕园看了中国男篮的一场比赛”这句话，进行标注，结果就是：</font>

```python
[B-PER，E-PER，O, B-ORG，I-ORG，I-ORG，E-ORG，O，B-LOC，E-LOC，O，O，B-ORG，I-ORG，I-ORG，E-ORG，O，O，O，O]
```

<font style="color:rgb(0, 0, 0);">通过NER模型，将“小明 ”以PER，“北京大学”以ORG，“燕园”以LOC，“中国男篮”以ORG为类别分别挑了出来。</font>

:::info
这个含义就是说b是开始b-per（人名的开始）e-per（人名的结束），o表示没用的相当于介词

:::



:::color4
这个ner就是开始识别每一个字是什么位置，起什么作用的

:::



## dataset模块
ner_datasets["train"][0]

```python
{'id': '0',
 'tokens': ['海',
  '钓',
  '比',
  '赛',
  '地',
  '点',
  '在',
  '厦',
  '门',
  '与',
  '金',
  '门',
  '之',
  '间',
  '的',
  '海',
  '域',
  '。'],
 'ner_tags': [0, 0, 0, 0, 0, 0, 0, 5, 6, 0, 5, 6, 0, 0, 0, 0, 0, 0]}
```

```plain
label_list = ner_datasets["train"].features["ner_tags"].feature.names
label_list


['O', 'B-PER', 'I-PER', 'B-ORG', 'I-ORG', 'B-LOC', 'I-LOC']
```

可以看到是每一个word都有一个属性0-6，



## fp函数
重点的就是实现fp函数，如何把label进行标注

1. 首先通过tokenlization进行编码
2. 之后对每一个句子开始进行遍历，放入labels
3. 然后设置返回的labels

```python
# 借助word_ids 实现标签映射
def process_function(examples):
    tokenized_exmaples = tokenizer(examples["tokens"], max_length=128, truncation=True, is_split_into_words=True)
    labels = []
    for i, label in enumerate(examples["ner_tags"]):
        word_ids = tokenized_exmaples.word_ids(batch_index=i)
        label_ids = []
        for word_id in word_ids:
            if word_id is None:
                label_ids.append(-100)
            else:
                label_ids.append(label[word_id])
        labels.append(label_ids)
    tokenized_exmaples["labels"] = labels
    return tokenized_exmaples

tokenized_datasets = ner_datasets.map(process_function, batched=True)
tokenized_datasets
```



```python

print(tokenized_datasets["train"][0])


{'id': '0', 'tokens': ['海', '钓', '比', '赛', '地', '点', '在', '厦', '门', '与', '金', '门', '之', '间', '的', '海', '域', '。'], 'ner_tags': [0, 0, 0, 0, 0, 0, 0, 5, 6, 0, 5, 6, 0, 0, 0, 0, 0, 0], 'input_ids': [101, 3862, 7157, 3683, 6612, 1765, 4157, 1762, 1336, 7305, 680, 7032, 7305, 722, 7313, 4638, 3862, 1818, 511, 102], 'token_type_ids': [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0], 'attention_mask': [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1], 'labels': [-100, 0, 0, 0, 0, 0, 0, 0, 5, 6, 0, 5, 6, 0, 0, 0, 0, 0, 0, -100]}
```

## 模型创建
<font style="background-color:#FBDE28;">使用的不是二分类任务，这个时候就要制定特定的num</font>

```python
# 对于所有的非二分类任务，切记要指定num_labels，否则就会device错误
model = AutoModelForTokenClassification.from_pretrained("hfl/chinese-macbert-base", num_labels=len(label_list))
```



## 评估函数
```python
import numpy as np

def eval_metric(pred):
    predictions, labels = pred
    predictions = np.argmax(predictions, axis=-1)

    # 将id转换为原始的字符串类型的标签
    true_predictions = [
        [label_list[p] for p, l in zip(prediction, label) if l != -100]
        for prediction, label in zip(predictions, labels) 
    ]

    true_labels = [
        [label_list[l] for p, l in zip(prediction, label) if l != -100]
        for prediction, label in zip(predictions, labels) 
    ]

    result = seqeval.compute(predictions=true_predictions, references=true_labels, mode="strict", scheme="IOB2")

    return {
        "f1": result["overall_f1"]
    }
```

## trainer设置
直接跳过，参考上一节



## 完整的代码
```python
import evaluate
from datasets import load_dataset
from transformers import AutoTokenizer, AutoModelForTokenClassification, TrainingArguments, Trainer, DataCollatorForTokenClassification1
# 如果可以联网，直接使用load_dataset进行加载
#ner_datasets = load_dataset("peoples_daily_ner", cache_dir="./data")
# 如果无法联网，则使用下面的方式加载数据集
from datasets import DatasetDict
ner_datasets = DatasetDict.load_from_disk("ner_data")
ner_datasets
tokenizer = AutoTokenizer.from_pretrained("hfl/chinese-macbert-base")
def process_function(examples):
    tokenized_exmaples = tokenizer(examples["tokens"], max_length=128, truncation=True, is_split_into_words=True)
    labels = []
    for i, label in enumerate(examples["ner_tags"]):
        word_ids = tokenized_exmaples.word_ids(batch_index=i)
        label_ids = []
        for word_id in word_ids:
            if word_id is None:
                label_ids.append(-100)
            else:
                label_ids.append(label[word_id])
        labels.append(label_ids)
    tokenized_exmaples["labels"] = labels
    return tokenized_exmaples
tokenized_datasets = ner_datasets.map(process_function, batched=True)
tokenized_datasets
model = AutoModelForTokenClassification.from_pretrained("hfl/chinese-macbert-base", num_labels=len(label_list))


import numpy as np

def eval_metric(pred):
    predictions, labels = pred
    predictions = np.argmax(predictions, axis=-1)

    # 将id转换为原始的字符串类型的标签
    true_predictions = [
        [label_list[p] for p, l in zip(prediction, label) if l != -100]
        for prediction, label in zip(predictions, labels) 
    ]

    true_labels = [
        [label_list[l] for p, l in zip(prediction, label) if l != -100]
        for prediction, label in zip(predictions, labels) 
    ]

    result = seqeval.compute(predictions=true_predictions, references=true_labels, mode="strict", scheme="IOB2")

    return {
        "f1": result["overall_f1"]
    }

  args = TrainingArguments(
    output_dir="models_for_ner",
    per_device_train_batch_size=64,
    per_device_eval_batch_size=128,
    evaluation_strategy="epoch",
    save_strategy="epoch",
    metric_for_best_model="f1",
    load_best_model_at_end=True,
    logging_steps=50,
    num_train_epochs=1
)  
trainer = Trainer(
    model=model,
    args=args,
    train_dataset=tokenized_datasets["train"],
    eval_dataset=tokenized_datasets["validation"],
    compute_metrics=eval_metric,
    data_collator=DataCollatorForTokenClassification(tokenizer=tokenizer)
)
trainer.train()


# 根据start和end取实际的结果
ner_result = {}
x = "小明在北京上班"
for r in res:
    if r["entity_group"] not in ner_result:
        ner_result[r["entity_group"]] = []
    ner_result[r["entity_group"]].append(x[r["start"]: r["end"]])

ner_result
```



# 机器阅读
相当于是做阅读理解，这个时候就是下面这种格式

```python
{'id': 'TRAIN_186_QUERY_0',
 'context': '范廷颂枢机（，），圣名保禄·若瑟（），是越南罗马天主教枢机。1963年被任为主教；1990年被擢升为天主教河内总教区宗座署理；1994年被擢升为总主教，同年年底被擢升为枢机；2009年2月离世。范廷颂于1919年6月15日在越南宁平省天主教发艳教区出生；童年时接受良好教育后，被一位越南神父带到河内继续其学业。范廷颂于1940年在河内大修道院完成神学学业。范廷颂于1949年6月6日在河内的主教座堂晋铎；及后被派到圣女小德兰孤儿院服务。1950年代，范廷颂在河内堂区创建移民接待中心以收容到河内避战的难民。1954年，法越战争结束，越南民主共和国建都河内，当时很多天主教神职人员逃至越南的南方，但范廷颂仍然留在河内。翌年管理圣若望小修院；惟在1960年因捍卫修院的自由、自治及拒绝政府在修院设政治课的要求而被捕。1963年4月5日，教宗任命范廷颂为天主教北宁教区主教，同年8月15日就任；其牧铭为「我信天主的爱」。由于范廷颂被越南政府软禁差不多30年，因此他无法到所属堂区进行牧灵工作而专注研读等工作。范廷颂除了面对战争、贫困、被当局迫害天主教会等问题外，也秘密恢复修院、创建女修会团体等。1990年，教宗若望保禄二世在同年6月18日擢升范廷颂为天主教河内总教区宗座署理以填补该教区总主教的空缺。1994年3月23日，范廷颂被教宗若望保禄二世擢升为天主教河内总教区总主教并兼天主教谅山教区宗座署理；同年11月26日，若望保禄二世擢升范廷颂为枢机。范廷颂在1995年至2001年期间出任天主教越南主教团主席。2003年4月26日，教宗若望保禄二世任命天主教谅山教区兼天主教高平教区吴光杰主教为天主教河内总教区署理主教；及至2005年2月19日，范廷颂因获批辞去总主教职务而荣休；吴光杰同日真除天主教河内总教区总主教职务。范廷颂于2009年2月22日清晨在河内离世，享年89岁；其葬礼于同月26日上午在天主教河内总教区总主教座堂举行。',
 'question': '范廷颂是什么时候被任为主教的？',
 'answers': {'text': ['1963年'], 'answer_start': [30]}}
```

:::color4
内容+q+a

:::

