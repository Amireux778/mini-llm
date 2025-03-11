<div align="center">

# 中文对话0.02B小模型 mini-llm-0.02B  

中文  | [English](./README_en.md)  

</div>


# 一、👋介绍 
本项目的目标是从0开始训练一个生成式语言模型，包括tokenizer训练、模型预训练、SFT指令微调、RLHF优化(DPO)等。 

mini-llm-0.02B 为中文对话小模型，模型参数只有0.02B（算共享权重约25.83M）。 


- 公开所有预训练、SFT指令微调、DPO偏好优化数据集来源。
- 训练Tokenizer
- 预训练：整合为端到端的预训练。
- SFT微调。
- RLHF偏好优化：使用DPO进行全量偏好优化。

# 二、🛠️mini-ChatGPT-Chinese-0.1B模型训练过程 

## 2.1 预训练数据集
所有数据集均来自互联网公开的**单轮对话**数据集。主要数据集包括： 

1. 社区问答json版webtext2019zh-大规模高质量数据集，见：[nlp_chinese_corpus](https://github.com/brightmart/nlp_chinese_corpus)。共410万，选取大约200万条用于tokenizer训练。
2. baike_qa2019百科类问答，见：<https://aistudio.baidu.com/datasetdetail/107726>，共140万，选取大约20万条。
3. 中国医药领域问答数据集，见：[Chinese-medical-dialogue-data](https://github.com/Toyhom/Chinese-medical-dialogue-data)，共79万条左右。
5. 知乎问答数据，见：[Zhihu-KOL](https://huggingface.co/datasets/wangrui6/Zhihu-KOL)，共100万条左右。
6. belle开源的指令训练数据，见：[train_0.5M_CN](https://huggingface.co/datasets/BelleGroup/train_0.5M_CN)，共50万条左右。

数据集总数量250万左右：预训练集：200万左右，评估集：还未设置。 SFT微调数据大概50万和DPO优化数据集待更新。

## 2.2 模型
（待补充）

模型参数：0.02B。词表大小：6400，仅包含中文和少量英文。

## 2.3 训练过程
硬件：
```bash
# 预训练及sft阶段：
CPU: AMD EPYC 7763 64-Core Processor
内存：512 GB
显卡：NVIDIA GeForce RTX 4090(24G) * 8
```
1. **tokenizer 训练**： 训练时间40分钟左右

2. **预训练**：（待补充）

3. **prompt监督微调（SFT）**：（待补充）

4. **dpo直接偏好优化（RLHF）**：待更新

存在问题：预训练数据集只有200万左右，模型参数也仅0.02B，不能涵盖所有领域，会有答非所问、废话生成器的情况。


# 三、📑使用说明


## 3.1 Tokenizer训练  

**1.准备txt语料  **

本项目以wiki中文百科为主。获取中文wiki语料方法：中文Wiki下载地址：[zhwiki](https://dumps.wikimedia.org/zhwiki/)，下载`zhwiki-[存档日期]-pages-articles-multistream.xml.bz2`文件，大概3GB， 将下载的bz2文件转换为wiki.txt参考：[WikiExtractor](https://github.com/apertium/WikiExtractor)，再利用python的`OpenCC`库转换为简体中文，最后将得到的`wiki.simple.txt`放到项目根目录的`data`目录下即可。

训练tokenizer非常耗内存，如果你的语料非常大（合并后的`txt`文件超过2G），建议对语料按照类别、比例进行采样，以减少训练时间和内存消耗。

**2.训练tokenizer**

```
# 确保你的训练语料`txt`文件已经data目录下
cd Tokenizer
python train_tokenizer.py
```
**3.测试训练好的tokenizer**
训练得到6400词表的tokenizer的效果：
```python
from transformers import AutoTokenizer

# 加载预训练的tokenizer
# tokenizer = AutoTokenizer.from_pretrained("./minimind_tokenizer")      #pretrain_hq.jsonl 训练的
tokenizer = AutoTokenizer.from_pretrained("./wiki_tokenizer")            # wiki_simple.txt 训练的
# tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")         # hf上的

text = "Hello, y'all! How are you 😁 ? 这句话的中文是什么？"

# 获取实际词汇表长度（包括特殊符号）
actual_vocab_size = len(tokenizer)
print('tokenizer实际词表长度：', actual_vocab_size)

text_token=tokenizer.tokenize(text)
print("编码后的token：",text_token)

ids=tokenizer.encode(text, add_special_tokens=True)
print("编码后token的ids：",ids)

de=tokenizer.decode(ids, skip_special_tokens=True)
print("ids解码后的文本：",de)
```

```
tokenizer实际词表长度： 6400
编码后的token： ['H', 'ell', 'o', ',', 'Ġ', 'y', "'", 'all', '!', 'ĠH', 'ow', 'Ġ', 'are', 'Ġ', 'y', 'ou', 'Ġ', 'ð', 'Ł', 'ĺ', 'ģ', 'Ġ', '?', 'Ġ', 'è¿Ļ', 'åı¥', 'è¯Ŀ', 'çļĦ', 'ä¸Ńæĸĩ', 'æĺ¯', 'ä»Ģä¹Ī', 'ï¼Ł']
编码后token的ids： [42, 4188, 81, 14, 223, 91, 9, 4777, 3, 2231, 2349, 223, 6271, 223, 91, 1738, 223, 175, 256, 249, 226, 223, 33, 223, 458, 3253, 1408, 265, 3030, 305, 4140, 3287]
ids解码后的文本： Hello, y'all! How are you 😁 ? 这句话的中文是什么？
```

## 3.2 预训练 
1.预训练数据集为jsonl文件，每行是一个独立的有效 JSON 对象。
文件地址：（待补充）

2.预训练数据集示例
```
{"text": "<s>鉴别一组中文文章的风格和特点，例如官方、口语、文言等。需要提供样例文章才能准确鉴别不同的风格和特点。</s> <s>好的，现在帮我查一下今天的天气怎么样?今天的天气依据地区而异。请问你需要我帮你查询哪个地区的天气呢？</s> <s>打开闹钟功能，定一个明天早上七点的闹钟。好的，我已经帮您打开闹钟功能，闹钟将在明天早上七点准时响起。</s> <s>为以下场景写一句话描述：一个孤独的老人坐在公园长椅上看着远处。一位孤独的老人坐在公园长椅上凝视远方。</s> <s>非常感谢你的回答。请告诉我，这些数据是关于什么主题的？这些数据是关于不同年龄段的男女人口比例分布的。</s> <s>帮我想一个有趣的标题。这个挺有趣的：\"如何成为一名成功的魔术师\" 调皮的标题往往会吸引读者的注意力。</s> <s>回答一个问题，地球的半径是多少？地球的平均半径约为6371公里，这是地球自赤道到两极的距离的平均值。</s> <s>识别文本中的语气，并将其分类为喜悦、悲伤、惊异等。\n文本：“今天是我的生日！”这个文本的语气是喜悦。</s>"}
```
3.预训练实现：（待补充）
    目前已实现完整训练流程
    后续准备实现混合精度训练和torchrun分布式训练
4.运行预训练：
```
python train_pretrain.py
```
5.预训练结果展示：（待补充）
```
***故事续写***
👶输入: 在一个遥远的星球上，有一种神奇的生物
🤖️输出: 叫做“神龙”。神龙有一个巨大的龙身，闪闪发光，但又有一个神奇的东西——“神龙”。神龙之所以能够发挥最神奇的能力，是能够让人类能够看到、发现、感知和探索世界的力量。神龙之所以能够发挥最神奇的能力，是能够让人类看到、发现、感知、探索世界的力量。
***对话生成***
👶输入: 用户：今天的天气怎么样？助手：
🤖️输出: 是的，天气很好，记得带伞。
***事实问答***
👶输入: 问题：太阳系中最大的行星是哪个？答案：
🤖️输出: 木星。太阳系中最大的行星是木星。木星是太阳系中最大的行星，它的直径是地球的11倍。木星是太阳系中最大的行星，其质量是其他行星的总和的318倍。
```



## 3.3 SFT微调 
SFT数据集`sft_mini_512.jsonl`同样为jsonl文件，每行是一个独立的有效 JSON 对象， 下载地址：（待补充）。

sft指令微调数据集格式：
```json
{
    "conversations": [
        {"role": "user", "content": "你好"},
        {"role": "assistant", "content": "你好！"}
    ]
}
```
sft指令微调数据集示例：
```
{"conversations": [{"role": "user", "content": "请告诉我在中国古代的“四大发明”是什么？"}, {"role": "assistant", "content": "中国古代的“四大发明”是指造纸术、印刷术、火药和指南针。这四项发明对世界文明的发展产生了深远的影响：\n\n1. **造纸术**：据史书记载，东汉时期的蔡伦改进了造纸工艺，使得纸张的生产更加便捷、成本更低，质量也更加优良。这一发明极大地促进了文化的传播和保存。\n\n2. **印刷术**：中国古代的印刷术最早可以追溯到唐代的雕版印刷，到了宋代发展出了活字印刷技术。印刷术的发明极大地促进了知识的传播，降低了书籍的成本，对教育和文化的发展起到了重要的推动作用。\n\n3. **火药**：火药最初是在唐代被发现的，最初可能用于医疗或炼金术。到了宋代，人们开始将火药用于军事目的，发明了各种火器。火药的发明改变了战争的面貌，对世界军事技术的发展产生了重大影响。\n\n4. **指南针**：指南针最初在中国被用于风水测量，后来逐渐发展成为航海导航的重要工具。这一发明极大地促进了海上航行技术的发展，对于新航路的开辟和世界地理大发现起到了关键作用。\n\n这四项发明不仅在中国历史上占有重要地位，而且对全世界的科技进步和文明发展都产生了深远的影响。"}]}
```

运行SFT微调：
``` bash
python train_full_sft.py
```
## 3.4 LoRA微调 
原始模型为进行过full_sft后的模型

1.身份LoRA微调
微调前：（由于full_sft微调数据是从网络上获取，其中包含通义千问这样的身份信息。故有如下输出）
```
👶: 你是谁  
🤖️: 我是通义千问，由阿里云开发的AI助手。我被设计用来回答各种问题、提供信息和与用户进行对话。有什么我可以帮助你的吗？
```

微调后：
```
👶: 你是谁
🤖️: 我是 mini-llm，由 amireux778 开发的人工智能助手。我被设计用来回答各种问题和提供信息，帮助用户解决问题和提供解决方案。
```
2.医学LoRA微调
微调前：
```
👶: 我最近经常头痛，尤其是后脑勺部位，伴有恶心，这是什么问题？
🤖️: 脑勺部位疼痛通常是由多种原因引起的，包括但不限于神经损伤、肌肉或骨骼损伤、神经系统疾病、内分泌问题、内耳问题、眼睛疼痛、眼部问题、神经系统疾病、内耳问题、脑部疾病等。这些问题可能导致头痛、恶心、呕吐、血管问题、关节炎、肌肉疼痛等症状。
如果你怀疑自己可能患有头痛，建议尽快就医，进行详细的检查和评估，以确定疼痛的确切原因。医生可能会建议进行头痛测试，以确定疼痛的原因，并提供相应的治疗方案。同时，保持良好的生活习惯，如规律作息、适量运动、健康饮食和定期体检，有助于改善症状。
```
微调后：
```
👶: 我最近经常头痛，尤其是后脑勺部位，伴有恶心，这是什么问题？
🤖️: 头痛和恶心可能是由于脑勺打开引起的并发症，导致的疼痛。以下是一些可能的治疗方法：
1. 手术：如果头痛严重，可能会有严重的并发症，如肿胀、淤血、心脏病等，这些症状可能需要手术治疗。手术方法取决于疼痛的原因。
2. 药物治疗：对于严重的头痛，可能需要使用药物来缓解疼痛。例如，在疼痛的部位，使用非处方药物如布洛芬或注射布洛芬。如果疼痛持续不减或者影响到日常生活，可能需要药物治疗。
3. 生活方式改变：保持健康的生活方式，如戒烟、健康饮食、定期运动、避免过度的压力和焦虑等，都有助于缓解头痛和恶心。
4. 心理治疗：包括认知行为疗法、行为疗法等，可以帮助缓解头痛和恶心。
5. 药物治疗：如果头痛严重或持续不减，可能需要药物治疗。药物包括碳酸饮料、抗生素、抗焦虑药、抗抑郁药等。
6. 定期复查：定期进行头痛检查和复查，以便及时发现并处理可能的头痛。
```

## 3.5 RLHF（强化学习人类反馈优化方法）——待更新~

偏好方法主要有两种，分别是PPO和DPO，具体实现请自行搜索论文及博客，**本项目采用DPO微调方法，比较节省显存**。 

**DPO（直接偏好优化，Direct Preference Optimization）微调**
在获得SFT模型的基础上，无需训练奖励模型，取得正向回答（chosen）和负向回答（rejected）即可开始微调。

DPO偏好优化数据集示例：
```json
{
    "prompt": "请介绍一下浙江大学",
    "chosen": "浙江大学是一所历史悠久、声誉卓著的高等学府，坐落于中国历史文化名城、风景旅游胜地杭州。",
    "rejected": "浙江大学是一所野鸡大学。"
}
```

运行偏好优化：待更新~

## 3.6 推理 
确保`model_save`和`Tokenizer`目录下有以下文件：
```bash
mini-ChatGPT-Chinese
├─model_save
|  ├─SFT_GPT.pth
├─Tokenizer
|  └─tokenizer.bin
```

控制台运行：

```bash
python client.py
```

# 四、🎓引用
如果你觉得本项目对你有所帮助，欢迎引用。
```conf
@misc{mini-ChatGPT-Chinese,
    author={Jiaxing Song},
    title={mini-ChatGPT-Chinese-0.1B},
    year={2024},
    publisher = {GitHub},
    journal = {GitHub repository},
    howpublished = {\url{https://github.com/JiaxingSong718/mini-ChatGPT-Chinese}},
}
```

# 五、🤔其他事项
本项目不承担开源模型和代码导致的数据安全、舆情风险或发生任何模型被误导、滥用、传播、不当利用而产生的风险和责任。
