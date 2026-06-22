对于需要频繁更新子仓库的情况, 不要用submodule, 而是在git clone他人仓库后, 直接删了该仓库的.git文件夹. 


### Submodule 的好处

**1. 轻量级，不占空间**
- Submodule 在你的仓库里只存储一个 **commit hash**（约 40 字节的引用），而不是整个仓库的所有文件
- 比如 `Physical-Prompt-Injection-Attack` 有约 20MB 的文件，用 submodule 你的仓库几乎不增加体积

**2. 自动保持与上游同步**
- 当原作者更新了 `2023cghacker/Physical-Prompt-Injection-Attack`，你可以用一条命令拉取最新代码：
  ```bash
  cd Physical-Prompt-Injection-Attack
  git pull origin main
  ```
- 如果是直接复制文件，你就得手动去下载、覆盖，容易遗漏

**3. 来源清晰，尊重原作者**
- 在 GitHub 上显示为一个链接，任何人都能直接跳转到原仓库
- 明确了代码归属，学术复现场景下这一点很重要

**4. 可以锁定版本**
- Submodule 记录的是某个**特定 commit**，你可以选择锁定在某个稳定版本，不受上游后续改动影响
- 如果上游出了 bug，你的项目不受影响，因为你锁定的是旧版本

**5. 版本控制干净**
- 你的提交历史里不会混入别人的大量文件改动
- `git log` 只会显示你自己的修改

### 直接复制文件的好处

- **修改自由**：你可以随意修改原仓库的代码，改动会被记录在你的提交历史里
- **不依赖网络**：clone 时不需要再去拉取子模块
- **没有 submodule 的复杂性**：submodule 的操作（更新、切换版本）对新手来说有一定学习成本

### 总结

| | Submodule | 直接复制 |
|---|---|---|
| 仓库体积 | ✅ 非常小 | ❌ 很大 |
| 同步上游更新 | ✅ 一行命令 | ❌ 手动操作 |
| 自由修改 | ❌ 需要 fork 或手动处理 | ✅ 直接改 |
| 知识产权归属 | ✅ 清晰 | ⚠️ 不够清晰 |
| 学习成本 | ⚠️ 需要了解 submodule 操作 | ✅ 简单 |


## 常用操作

- 在已有文件夹中, 第一次添加一个仓库作为submodule
```
git submodule add https://github.com/2023cghacker/Physical-Prompt-Injection-Attack.git Physical-Prompt-Injection-Attack
```

- 克隆一个带有submodule的仓库
```
git clone --recurse-submodules https://github.com/flyswxf/PPIA.git
```
- `recurse`: 表示递归遍历子仓库的子仓库
