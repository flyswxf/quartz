## kaggle token
在setting中获取
![[kaggle token.png]]
手动注册到`~/.kaggle/access_token`(创建文件并写入即可)
```bash
mkdir -p ~/.kaggle && echo TOKEN > ~/.kaggle/access_token && chmod 600 ~/.kaggle/access_token
```
- TOKEN替换为实际token内容

配置后, 可以使用[[kaggle#kaggle hub|kaggle hub]]


## kaggle hub
```bash
pip install kagglehub
```

可以进行**数据集上传/更新**操作
>由于内部使用的`www.googleapis.com`, 必须开启vpn使用

需要提供
- handle: `f"{owner}/{dataset}"`
	- owner: kaggle用户名(上方小字)
	  ![[kaggle用户名.png]]
	- dataset: 数据集名称

### 保存Output为数据集
默认会将`/kaggle/working` 作为output结果

如果该文件夹中, 所有文件总和较大(100k+), **输出会被压缩为_output.zip**, 且运行时长也会出现异常. **但这不会影响保存为数据集后的解析**, 数据集形态下, 会自动解压缩
 