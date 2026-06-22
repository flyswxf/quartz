## 使用终端命令行工具`hf`
```bash
pip install huggingface_hub   
```

##  换国内镜像
```powershell
$env:HF_ENDPOINT = "https://hf-mirror.com" 
```

```bash
export HF_ENDPOINT="https://hf-mirror.com" 
```
- 不能有空格
## 下载模型
```powershell
hf download REPO_ID --local-dir Physical_Attention_Attack\model_weights
```
- `hf download`: 下载模型
- `REPO_ID`: 目标仓库地址. 格式: `作者/模型名`
- `--local-dir`: 指定文件保存的本地目录. 不启用则会默认缓存到C盘cache
	- 虽然也可以自定义缓存目录, 但麻烦

```bash
python -c "from huggingface_hub import hf_hub_download

hf_hub_download(
    repo_id='xdecoder/SEEM',
    filename='seem_focall_v1.pt',
    local_dir='.'  # 下载到当前文件夹
)"
```

