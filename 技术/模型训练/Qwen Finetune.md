
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
%cd F:/code/private/learn-finetune/

import torch
from modelscope import snapshot_download, AutoModel, AutoTokenizer
import os
model_dir = snapshot_download('qwen/Qwen2.5-0.5B-Instruct', cache_dir='D:/cached_models/', revision='master')


# 导入环境
from datasets import Dataset
import pandas as pd
from transformers import AutoTokenizer, AutoModelForCausalLM, DataCollatorForSeq2Seq, TrainingArguments, Trainer, GenerationConfig
df = pd.read_json('C:/Users/Administrator/Downloads/huanhuan.json')
ds = Dataset.from_pandas(df)
ds[:3]


# 处理数据集
tokenizer = AutoTokenizer.from_pretrained('D:/cached_models/qwen/Qwen2___5-0___5B-Instruct/', use_fast=False, trust_remote_code=True)
tokenizer


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
tokenized_id


tokenizer.decode(tokenized_id[0]['input_ids'])

tokenizer.decode(list(filter(lambda x: x != -100, tokenized_id[1]["labels"])))


# 创建模型
import torch
model = AutoModelForCausalLM.from_pretrained('D:/cached_models/qwen/Qwen2___5-0___5B-Instruct/', device_map="auto",torch_dtype=torch.bfloat16)
model
model.enable_input_require_grads() # 开启梯度检查点时，要执行该方法
model.dtype


# lora
from peft import LoraConfig, TaskType, get_peft_model
config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj", "gate_proj", "up_proj", "down_proj"],
    inference_mode=False, # 训练模式
    r=8, # Lora 秩
    lora_alpha=32, # Lora alaph，具体作用参见 Lora 原理
    lora_dropout=0.1# Dropout 比例
)
config
model = get_peft_model(model, config)
model.print_trainable_parameters()


# 配置训练参数
args = TrainingArguments(
    output_dir="./output/Qwen2_instruct_lora",
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,
    logging_steps=10,
    num_train_epochs=3,
    save_steps=10, # 为了快速演示，这里设置10，建议你设置成100
    learning_rate=1e-4,
    save_on_each_node=True,
    gradient_checkpointing=True
)
trainer = Trainer(
    model=model,
    args=args,
    train_dataset=tokenized_id,
    data_collator=DataCollatorForSeq2Seq(tokenizer=tokenizer, padding=True),
)
trainer.train()


# 合并加载模型
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch
from peft import PeftModel

mode_path = 'D:/cached_models/qwen/Qwen2___5-0___5B-Instruct/'
lora_path = './output/Qwen2_instruct_lora/checkpoint-10' # 这里改称你的 lora 输出对应 checkpoint 地址


tokenizer = AutoTokenizer.from_pretrained(mode_path, trust_remote_code=True) # 加载tokenizer
model = AutoModelForCausalLM.from_pretrained(mode_path, device_map="auto",torch_dtype=torch.bfloat16, trust_remote_code=True).eval() # 加载模型
model = PeftModel.from_pretrained(model, model_id=lora_path) # 加载lora权重

prompt = "你是谁？"
inputs = tokenizer.apply_chat_template([{"role": "user", "content": "假设你是皇帝身边的女人--甄嬛。"},{"role": "user", "content": prompt}],
                                       add_generation_prompt=True,
                                       tokenize=True,
                                       return_tensors="pt",
                                       return_dict=True
                                       ).to('cuda')

gen_kwargs = {"max_length": 2500, "do_sample": True, "top_k": 1}
with torch.no_grad():
    outputs = model.generate(**inputs, **gen_kwargs)
    outputs = outputs[:, inputs['input_ids'].shape[1]:]
    print(tokenizer.decode(outputs[0], skip_special_tokens=True))

```


## 参考资料

- [self-llm/models/Qwen2.5/05-Qwen2.5-7B-Instruct Lora 微调.md at master · datawhalechina/self-llm](https://github.com/datawhalechina/self-llm/blob/master/models/Qwen2.5/05-Qwen2.5-7B-Instruct%20Lora%20%E5%BE%AE%E8%B0%83.md)
- [self-llm/models/Qwen2/05-Qwen2-7B-Instruct Lora.ipynb at master · datawhalechina/self-llm](https://github.com/datawhalechina/self-llm/blob/master/models/Qwen2/05-Qwen2-7B-Instruct%20Lora.ipynb)
- 