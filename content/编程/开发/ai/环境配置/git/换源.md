
服务器没有vpn时, github太慢, 可以通过代理来连接
### kkgithub（2026年首选） 
```bash
git clone https://kkgithub.com/flyswxf/Physical-Prompt-Injection-Attack.git
```

在服务器端 #疑似 git push必须得是从原始github才能登录, 进而push(pull不受影响), 因此需要临时换remote

```bash
git remote set-url origin https://github.com/flyswxf/Physical-Prompt-Injection-Attack.git
```

```bash
git remote set-url origin https://kkgithub.com/flyswxf/Physical-Prompt-Injection-Attack.git
```
通过
```bash
git remote -v
```
查看当前远程仓库地址 