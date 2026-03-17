
## 一、LoRA实战
### ① 环境配置

```bash
conda create -y -n py312finetine python=3.12
conda activate py312finetine

pip install modelscope==1.18.0
pip install transformers==4.44.2
pip install streamlit==1.24.0
pip install sentencepiece==0.2.0
pip install accelerate==0.34.2
pip install datasets==2.20.0
pip install peft==0.11.1

```

### ② 底模下载
```python
# 下载底座模型
import torch
from modelscope import snapshot_download, AutoModel, AutoTokenizer
import os
model_dir = snapshot_download('qwen/Qwen2.5-0.5B-Instruct', cache_dir='D:/cached_models/', revision='master')
```

### ③ 微调模型
```python
# 微调模型  %cd F:/code/private/learn-finetune/

import os
os.environ['PYTORCH_CUDA_ALLOC_CONF'] = "expandable_segments:True"  # 减小峰值显存占用，随借随还

# 导入环境
from datasets import Dataset
import pandas as pd
from transformers import AutoTokenizer, AutoModelForCausalLM, DataCollatorForSeq2Seq, TrainingArguments, Trainer, GenerationConfig
df = pd.read_json('/mnt/c/Users/Administrator/Downloads/huanhuan.json')
ds = Dataset.from_pandas(df)
print(ds[:3])


# 处理数据集
tokenizer = AutoTokenizer.from_pretrained('/mnt/d/cached_models/qwen/Qwen2.5-0.5B-Instruct/', use_fast=False, trust_remote_code=True)
print(tokenizer)

def process_func(example):
    MAX_LENGTH = 384    # Llama分词器会将一个中文字切分为多个token，因此需要放开一些最大长度，保证数据的完整性
    input_ids, attention_mask, labels = [], [], []
    instruction = tokenizer(f"<|im_start|>system\n现在你要扮演皇帝身边的女人--甄嬛<|im_end|>\n<|im_start|>user\n{example['instruction'] + example['input']}<|im_end|>\n<|im_start|>assistant\n", add_special_tokens=False)  # add_special_tokens 不在开头加 special_tokens
    response = tokenizer(f"{example['output']}", add_special_tokens=False)
    input_ids = instruction["input_ids"] + response["input_ids"] + [tokenizer.pad_token_id]
    attention_mask = instruction["attention_mask"] + response["attention_mask"] + [1]  # 因为eos token咱们也是要关注的所以 补充为1
    labels = [-100] * len(instruction["input_ids"]) + response["input_ids"] + [tokenizer.pad_token_id]
    if len(input_ids) > MAX_LENGTH:  # 做一个截断
        input_ids = input_ids[:MAX_LENGTH]
        attention_mask = attention_mask[:MAX_LENGTH]
        labels = labels[:MAX_LENGTH]
    return {
        "input_ids": input_ids,
        "attention_mask": attention_mask,
        "labels": labels
    }

tokenized_id = ds.map(process_func, remove_columns=ds.column_names)
print(tokenized_id)
print(tokenizer.decode(tokenized_id[0]['input_ids']))
print(tokenizer.decode(list(filter(lambda x: x != -100, tokenized_id[1]["labels"]))))


# 创建模型
import torch
model = AutoModelForCausalLM.from_pretrained('/mnt/d/cached_models/qwen/Qwen2.5-0.5B-Instruct/', device_map="cuda",torch_dtype=torch.bfloat16)
print(model)
print(model.dtype)


# lora配置
from peft import LoraConfig, TaskType, get_peft_model
config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj", "gate_proj", "up_proj", "down_proj"],
    inference_mode=False, # 训练模式
    r=8, # Lora 秩
    lora_alpha=32, # Lora alaph，具体作用参见 Lora 原理
    lora_dropout=0.1# Dropout 比例
)
print(config)


# 配置训练参数
args = TrainingArguments(
    output_dir="./output/Qwen2_instruct_lora",
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,
    logging_steps=10,
    num_train_epochs=3,
    save_steps=100, # 为了快速演示，这里可以设置10，建议设置成100
    learning_rate=1e-4,
    save_on_each_node=True,
    gradient_checkpointing=True,
    #bf16=True,  # ！！！新增这一行，强制 Trainer 使用 bfloat16
)
if args.gradient_checkpointing:
    model.enable_input_require_grads() # 开启梯度检查点时(gradient_checkpointing=True)，要执行该方法

# lora训练
model = get_peft_model(model, config)
model.print_trainable_parameters()
trainer = Trainer(
    model=model,
    args=args,
    train_dataset=tokenized_id,
    data_collator=DataCollatorForSeq2Seq(tokenizer=tokenizer, padding=True),
)
trainer.train()
```

### ④ 测试模型

```python
# 合并加载模型
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch
from peft import PeftModel

mode_path = '/mnt/d/cached_models/qwen/Qwen2.5-0.5B-Instruct/'
lora_path = './output/Qwen2_instruct_lora/checkpoint-699' # 这里改称你的 lora 输出对应 checkpoint 地址


tokenizer = AutoTokenizer.from_pretrained(mode_path, trust_remote_code=True) # 加载tokenizer
lmodel = AutoModelForCausalLM.from_pretrained(mode_path, device_map="cuda",torch_dtype=torch.bfloat16, trust_remote_code=True).eval() # 加载模型
lmodel = PeftModel.from_pretrained(lmodel, model_id=lora_path) # 加载lora权重

prompt = "你是谁？"
inputs = tokenizer.apply_chat_template(
    [{"role": "system", "content": "假设你是皇帝身边的女人--甄嬛。"},{"role": "user", "content": prompt}],
    add_generation_prompt=True,
    tokenize=True,
    return_tensors="pt",
    return_dict=True
).to('cuda')

gen_kwargs = {"max_length": 2500, "do_sample": True, "top_k": 1}
with torch.no_grad():
    outputs = lmodel.generate(**inputs, **gen_kwargs)
    outputs = outputs[:, inputs['input_ids'].shape[1]:]
    print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```


## 二、LoRA数学原理

LoRA通常只作用于线性层（全连接层 / Dense Layer），其数学原理是将：

$$ W \cdot x $$
变成
$$ W \cdot x + (\frac{\alpha}{r}) \cdot (B \cdot A) \cdot x$$

其中：
- A是随机高斯初始化的小矩阵，如shape=(10000, 8)
- B是全零初始化的小矩阵，如shape=(8, 10000)
- r是是AB相乘后矩阵的秩（自由度），如r=8
- alpha是一个缩放因子，决定了lora对原模型的话语权有多重
	- 如果 Alpha 设得很高，模型输出就会极度偏向你训练的 LoRA 风格；如果设得很低，LoRA 的影响就若隐若现
	- 常规法则：通常把alpha设置为r的1倍或2倍（例如r=8，则alpha=8或16）
	- **为什么要设计成 $\frac{\alpha}{r}$ 这样的比值？为什么 Alpha 总是要和 r 捆绑在一起？**
		- 因为r越大，AB乘积结果的元素平均值就容易越大，因此lora权重就应该降低
		- 所以，除以r的设计，使得不同r的情况下，可以使用相同的alpha，而不必调整训练的学习率
		- 换而言之，我们可以把AB/r当成整理，alpha自己是lora的融合权重。在已经探索出了一套好的训练参数后，随意调整r的同时不必调整训练学习率
- LoRA作用域
	- 最经典的作用域：注意力机制（Attention）：
		- 在 Attention 机制中，输入特征会被分别乘以四个关键的权重矩阵：$W_q$，$W_k$，$W_v$ 和 $W_o$
		- 原版 LoRA 的结论：微软发现，如果只选一部分贴，同时作用在 $W_q$ 和 $W_v$ 上性价比最高。直觉理解： $W_q$ 改变了模型“看问题的角度”，$W_v$ 改变了模型“提取内容的方式”。改变这两者，足以让模型学会新的语气或处理特定任务的逻辑
		- 随着开源大模型越来越大，大家发现：**如果只贴 Attention 机制，模型学会新语气的速度很快，但似乎很难记住“硬核的新知识（比如垂直领域的医学知识、复杂的代码逻辑）”。** 这时候，大家把目光转向了 Transformer 模块的下半部分——多层感知机（MLP）
		- 在大模型主流架构中，MLP 包含了三个巨大的线性矩阵：`gate_proj`、`up_proj` 和 `down_proj`。学术界的新共识：如果把 Transformer 比作人类的大脑：
			- Attention 决定了“信息如何在大脑中流通和连接”（工作记忆、注意力分布）
			- MLP 则被认为是模型的**“知识库/长期记忆”（Knowledge Factual Base）
		- 因此，**如果你希望微调不仅改变风格，还要注入大量新知识，就必须把 LoRA 作用在 MLP 的权重上。**  这也催生了社区里著名的 **"All-Linear"（全线性层）** 策略：即把 LoRA 贴在 Attention 以及 MLP 的所有层上



## 三、LoRA训练经验

### ① 经验一：数据质量 > 数据数量（“100条精标胜过1万条垃圾”）

很多人以为微调模型需要海量数据，其实对于 LoRA 来说，**数据质量是绝对的生命线**。
*   **格式对齐：** 如果微调 LLM，你的训练数据格式（Prompt Format）必须和原模型**完全一致**。如果基座是 Llama-3，就必须用 Llama-3 的 `<|start_header_id|>` 格式组织你的数据。格式一旦错位，LoRA 学到的全是“如何纠正格式错误”，而不是你的业务知识。
*   **多样性陷阱：** 如果你的 1000 条训练数据全都是“请帮我写一首关于XX的诗”，模型训练完后会发生**模式坍塌（Mode Collapse）**——你问它“1+1等于几”，它也会吟一首诗。你必须在数据集中混入一些常规对话数据，以保持模型的通用能力。
*   **AI 绘画中的打标：** 训练人物 LoRA 时，如果所有的图片都是白背景，你不把“白背景 (white background)”写进提示词里，模型就会把白背景当成这个人物的“固有特征”。**把你不想要模型绑定的特征，写进打标文本里剥离出来。**

### ② 经验二：学习率（Learning Rate）要比全量微调大 10 倍

很多新手照搬全量微调（Full Fine-tuning）的经验，把学习率设成 `1e-5`（0.00001），结果跑了几天发现模型毫无变化（Loss 根本不降）。
*   **LoRA 专属区间：** 因为 LoRA 的参数量极小，你需要更大的学习率来推动它更新。通常，LLM 的 LoRA 学习率设置在 **`1e-4` 到 `3e-4`** 之间（对于 AdamW 优化器）。
*   **预热机制（Warmup）：** 千万别上来就给满学习率！一定要设置 Warmup Ratio（通常是 0.05 到 0.1）。让学习率在前 5%~10% 的步数里慢慢爬升到 `1e-4`，然后再缓慢下降（Cosine Decay）。这能极大避免训练初期模型直接崩溃。

### ③ 经验三：千万别迷信 Loss 曲线（用眼睛去评估）

炼丹师最大的错觉就是：“Loss 一直在降，模型肯定越来越好”。
*   **过拟合（Overfitting）的温水煮青蛙：** LoRA 非常容易过拟合。当 Loss 降到极低时，模型其实已经变成了“复读机”，只会死记硬背你的训练集，完全丧失了举一反三的能力（灾难性遗忘）。
*   **正确的做法（Checkpoint 盲测）：**
    1. 哪怕只训练 3 个 Epoch，也要每隔半个 Epoch 保存一个权重（Checkpoint）。
    2. 准备一个**不在训练集中**的测试问题列表。
    3. 把保存的几个 Checkpoint 拿出来，挨个输入测试问题，**用肉眼看它的回答/生成的图片**。
    4. 往往你会发现，Loss 最低的最后一个 Checkpoint 效果很僵硬，反而是中间的某个 Checkpoint 兼顾了“原模型的聪明”和“新注入的知识”。

### ④ 经验四：基座模型（Base Model）决定了你的天花板

**LoRA 只是方向盘，基座模型才是发动机。**
*   不要指望用 LoRA 把一个 1.5B 的小模型“教”成 GPT-4 的智商。LoRA 最擅长的是**“改变输出风格（Style）”、“规范输出格式（Format）”和“注入少量垂直领域知识（Knowledge）”**。
*   如果你要训练一个“暴躁老哥问答机器人”，你应该选一个本身就足够聪明、逻辑推理好的 Chat 模型作为基座，然后用 LoRA 去扭转它的语气；而不是选一个根本没经过指令微调的纯预训练（Base）模型，那样 LoRA 既要教它怎么对话，又要教它怎么暴躁，负担太重。

### ⑤ 经验五：参数组合的黄金法则（太长不看版）

在面对几十个超参数不知所措时，直接套用这套目前开源社区的**黄金默认值**，可以在 80% 的场景下拿到 85 分的成绩：
*   **Rank ($r$)**：设为 `16`。（8 有点少，32 以上提升不明显且容易过拟合）。
*   **Alpha ($\alpha$)**：设为 `32`（即 $\alpha = 2r$）。
*   **Target Modules**：无脑选 `all-linear`（把 Q, K, V, O, MLP 全加上）。虽然显存多占一点，但效果最扎实。
*   **Epoch**：大语言模型通常 `3` 到 `5` 个 Epoch 足矣。
*   **Batch Size**：在你显卡撑得住的范围内尽可能大（比如 `4` 或 `8`），配合梯度累加（Gradient Accumulation）让等效 Batch Size 达到 `16` 或 `32`，这能让梯度下降的方向更稳定。

### ⑥ 经验六：2026 年的前沿替代方案（进阶）

既然你已经深入了解了 LoRA，一定要知道在当下的技术栈中，原生 LoRA 已经有了一些非常强悍的升级变体，在实战中经常能直接替换标准 LoRA 取得更好效果：
*   **DoRA (Weight-Decomposed LoRA)：** 它把权重的“方向”和“大小（幅度）”拆解开来训练。经验上，**DoRA 能在更低的 Rank 下达到甚至超越全量微调的效果**，而且没那么容易过拟合，现在很多微调框架（如 LLaMA-Factory）只需勾选一下 `use_dora` 即可。
*   **PiSSA / O-LoRA：** 传统的 LoRA 初始矩阵 B 是全 0 的。而这些新技术通过对原模型的权重进行奇异值分解（SVD），把最核心的参数抽出来作为 LoRA 初始化，剩下的作为冻结基座。这相当于**“赢在起跑线上”**，收敛速度极快。

**总结一句话：**
成功的 LoRA 微调 = **极其干净的领域数据** + **选对聪明的基座** + **all-linear 目标层** + **10倍于平时的学习率** + **及时提前停止训练以防过拟合**。


## 参考资料

- [self-llm/models/Qwen2.5/05-Qwen2.5-7B-Instruct Lora 微调.md at master · datawhalechina/self-llm](https://github.com/datawhalechina/self-llm/blob/master/models/Qwen2.5/05-Qwen2.5-7B-Instruct%20Lora%20%E5%BE%AE%E8%B0%83.md)
- [self-llm/models/Qwen2/05-Qwen2-7B-Instruct Lora.ipynb at master · datawhalechina/self-llm](https://github.com/datawhalechina/self-llm/blob/master/models/Qwen2/05-Qwen2-7B-Instruct%20Lora.ipynb)
- [可视化并理解 PyTorch 中的 GPU 内存 --- Visualize and understand GPU memory in PyTorch](https://huggingface.co/blog/train_memory)
- 