
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


```python
# 下载底座模型
import torch
from modelscope import snapshot_download, AutoModel, AutoTokenizer
import os
model_dir = snapshot_download('qwen/Qwen2.5-0.5B-Instruct', cache_dir='D:/cached_models/', revision='master')
```


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


## LoRA
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
		- 在大模型主流架构中，MLP 包含了三个巨大的线性矩阵：`gate_proj`、`up_proj` 和 `down_pro`





## 参考资料

- [self-llm/models/Qwen2.5/05-Qwen2.5-7B-Instruct Lora 微调.md at master · datawhalechina/self-llm](https://github.com/datawhalechina/self-llm/blob/master/models/Qwen2.5/05-Qwen2.5-7B-Instruct%20Lora%20%E5%BE%AE%E8%B0%83.md)
- [self-llm/models/Qwen2/05-Qwen2-7B-Instruct Lora.ipynb at master · datawhalechina/self-llm](https://github.com/datawhalechina/self-llm/blob/master/models/Qwen2/05-Qwen2-7B-Instruct%20Lora.ipynb)
- [可视化并理解 PyTorch 中的 GPU 内存 --- Visualize and understand GPU memory in PyTorch](https://huggingface.co/blog/train_memory)
- 