# 基于大模型的文本内容智能评判

## 1. 数据准备

```python
import pandas as pd
train_df = pd.read_csv('./train.csv', encoding='gb2312', encoding_errors='ignore')
test_df = pd.read_csv('./test.csv', encoding='gb2312', encoding_errors='ignore')
res = []
for i in range(len(train_df)):
    llm_item = train_df.loc[i]
    if (llm_item['评判维度']=="选择题"):
        tmp = { "conversation": [
            {
        "system": "以下是一个大模型完成的选择题。如果大模型的回答与参考答案一致，输出1；如果不一致，输出0。",
        "input": f"选择题:{llm_item['创作要求']},评判内容:{llm_item['待评判内容']}",
        "output": str(llm_item['标注分值']) }
            ]
        }
        res.append(tmp)

    elif (llm_item['评判维度']=="流畅性"):
        tmp = { "conversation": [
            {
        "system": """以下是一个大模型完成的创作题目。按照流畅性评分标准给大模型创作打分(只取1分、2分、3分、4分、5分其一)。流畅性评分标准：
        1分: 非常不流畅，不具备可读性：语法错误明显，难以理解；大量拼写错误和错别字，影响阅读；表达不清晰，难以捉摸要表达的意思。（平均每百字错误数 > 2.5个）; 
        2分: 具有可读性，但较不流畅：常见语法错误多，需花费一定时间理解；一些拼写错误和错别字，阅读中断；表达较为模糊，需用一些猜测才能明白含义。（平均每百字错误数 (2,2.5]个）;
        3分：基本流畅，存在少量语法错误，但影响较小：稍有拼写错误，但不影响阅读；主要意思表达清楚，但部分地方表述不够准确。（平均每百字错误数(1,2]个）;
        4分：较流畅，语法错误稀少，易读性较高：几乎无拼写错误，阅读顺畅；表达清晰、准确，容易理解。（平均每百字错误数(0.5,1]个）；
        5分：非常流畅，语法、拼写完美，阅读体验优秀：表达精炼、准确、得体；文句优美，行文连贯，思维严密。（平均每百字错误数[0,0.5]个）""",
        "input": f" 大模型创作:{llm_item['待评判内容']}",
        "output": str(llm_item['标注分值']) }
            ]
        }
        res.append(tmp)

    else:
        tmp = { "conversation": [
            {
        "system": """以下是一个大模型完成的创作题目。按照规范性评分标准给大模型创作打分(只取1分、2分、3分、4分、5分其一)。规范性评分标准：
        1分: 创作内容离题，与提示语句要求不符，格式非常不规范：杂乱无章，句子结构混乱，缺乏逻辑。（平均每千字错误数 > 5个）; 
        2分: 创作内容与提示语句要求有一定契合但覆盖不全，格式较不规范：缺乏清晰的结构，但基本逻辑仍能找到。（平均每千字错误数(4,5]个）;
        3分：创作内容与提示语句要求基本契合但覆盖不全，格式一般规范：结构基本顺畅，逻辑较清晰。（平均每千字错误数 (2,4]个）;
        4分：创作内容与提示语句要求基本契合且基本覆盖，格式较规范：结构清晰，逻辑条理分明。（平均每千字错误数 (1,2]个）；
        5分：创作内容与提示语句要求完美契合，格式非常规范：结构严谨，逻辑清晰，层次分明。（平均每千字错误数 [0,1]个）""",
        "input": f"创作题目:{llm_item['创作要求']},大模型创作:{llm_item['待评判内容']}",
        "output": str(llm_item['标注分值']) }
            ]
        }
        res.append(tmp)
import json
import numpy as np

def convert_to_builtin_type(obj):
    if isinstance(obj, np.int64):  # 如果对象是 int64 类型
        return int(obj)  # 转换为内置的 int 类型
    raise TypeError(f"Object of type {type(obj)} is not JSON serializable")

with open('./data/llm_train.json', mode='w', encoding='utf-8') as f:
    json.dump(res, f, ensure_ascii=False, indent=4, default=convert_to_builtin_type)
```
## 2. 推理
见：[https://github.com/yaosenJ/NoteNest/blob/main/llm.ipynb](https://github.com/yaosenJ/NoteNest/blob/main/llm.ipynb)
