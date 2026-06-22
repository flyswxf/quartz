```bash
pip install git+https://github.com/facebookresearch/segment-anything.git
```

会用一个子进程去
```bash
git clone -quiet https://github.com/facebookresearch/segment-anything.git /tmp/pip-req-build-xxxxx
```
- 静默执行
- 并不是--depth=1

如果出错, 可参考[[代理#或者通过以下命令观察git是否通过了代理路径|git调试命令]]检查是否是代理问题
