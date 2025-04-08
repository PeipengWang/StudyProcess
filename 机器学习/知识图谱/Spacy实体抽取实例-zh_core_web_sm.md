环境

python==3.7.6

spacy==3.6.1  

zh_core_web_sm==3.6.0

spacy-pkuseg==0.0.33

代码

```

import spacy
from spacy import displacy

# 加载英语预训练模型
nlp = spacy.load("zh_core_web_sm")

# 待分析的文本
text = """
苹果公司计划于2023年9月推出iPhone 14。这款新设备将在美国、欧洲和亚洲的商店上市。
"""

# 处理文本并抽取实体
doc = nlp(text)

# 打印所有识别的实体
for entity in doc.ents:
    print(entity.text, entity.label_)

# 可视化实体
displacy.render(doc, style="ent")


```

