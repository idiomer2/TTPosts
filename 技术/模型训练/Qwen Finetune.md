
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


## 参考资料

- [self-llm/models/Qwen2.5/05-Qwen2.5-7B-Instruct Lora 微调.md at master · datawhalechina/self-llm](https://github.com/datawhalechina/self-llm/blob/master/models/Qwen2.5/05-Qwen2.5-7B-Instruct%20Lora%20%E5%BE%AE%E8%B0%83.md)