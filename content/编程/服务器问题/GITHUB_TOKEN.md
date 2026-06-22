# GitHub Token 使用简明指南

> 面向常用场景的高效实践：生成、配置、调用 API、权限说明与安全规范。

---

## 1. 生成 Token

- Web 路径：GitHub → Settings → Developer settings → Personal access tokens
- 推荐：Fine-grained tokens（细粒度令牌）
  - 选择资源拥有者（通常为你的账号），勾选需要的仓库或组织范围
  - 设置权限（如 Contents、Issues、Pull requests 等）与过期时间
  - 复制生成的 Token，妥善保存（仅显示一次）
- 备选：Classic tokens（经典令牌，范围更广，不建议在新项目使用）
  - 选择 scopes（如 `repo`、`workflow`、`gist` 等）与过期时间

---

## 2. 配置 Token

- 环境变量（推荐方式）
  - Windows PowerShell：
    ```powershell
    $env:GITHUB_TOKEN = "ghp_your_token_here"
    ```
    - `$env:GITHUB_TOKEN`：在当前会话中设置环境变量供命令读取。
  - Linux / macOS Bash：
    ```bash
    export GITHUB_TOKEN="ghp_your_token_here"
    ```
    - `export`：为当前 shell 会话及其子进程暴露变量。

- Git 使用（HTTPS）：当 Git 询问密码时，使用 Token 作为密码即可。
  - 示例：
    ```bash
    git clone https://github.com/<owner>/<repo>.git
    ```
    - `<owner>/<repo>`：仓库拥有者与名称；首次推送时输入 Token 作为密码。

- GitHub CLI（可选）：
  ```bash
  gh auth login
  ```
  - 交互式登录并配置令牌；也可读取 `GITHUB_TOKEN` 环境变量。

---

## 3. 使用 Token 调用 API

- 通用请求头（建议统一使用）：
  ```bash
  curl -s \
    -H "Authorization: Bearer $GITHUB_TOKEN" \
    -H "Accept: application/vnd.github+json" \
    -H "X-GitHub-Api-Version: 2022-11-28" \
    https://api.github.com
  ```
  - `Authorization: Bearer $GITHUB_TOKEN`：令牌认证；从环境变量读取。
  - `Accept: application/vnd.github+json`：使用 GitHub REST v3 的 JSON 媒体类型。
  - `X-GitHub-Api-Version: 2022-11-28`：锁定 API 版本，减少兼容性风险。
  - `-s`：静默模式，不输出进度。

### 示例 1：获取当前用户

```bash
curl -s \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/user
```
- 方法与路径：`GET /user`（基于令牌的用户上下文）

响应示例（节选）：
```json
{
  "login": "octocat",
  "id": 1,
  "name": "The Octocat",
  "company": null,
  "email": null
}
```

### 示例 2：列出用户仓库（含查询参数）

```bash
curl -s \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/user/repos?visibility=all&per_page=20&page=1"
```
- 方法与路径：`GET /user/repos`
- 查询参数解释：
  - `visibility=all`：返回公开与私有仓库；可选 `public`/`private`/`all`。
  - `per_page=20`：每页返回 20 项；范围 1–100。
  - `page=1`：第 1 页；用于分页遍历。

响应示例（数组节选）：
```json
[
  {
    "name": "hello-world",
    "private": false,
    "html_url": "https://github.com/octocat/hello-world",
    "description": "My first repository"
  }
]
```

### 示例 3：创建仓库（请求体参数）

```bash
curl -s -X POST \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/user/repos \
  -d '{
    "name": "demo-repo",
    "description": "Created via API",
    "private": true,
    "auto_init": true,
    "has_issues": true,
    "has_projects": false,
    "has_wiki": false
  }'
```
- 方法与路径：`POST /user/repos`
- 请求体参数解释：
  - `name`：仓库名（必填）。
  - `description`：描述（选填）。
  - `private`：是否私有；`true`/`false`。
  - `auto_init`：是否自动创建 README；便于立即推送。
  - `has_issues`、`has_projects`、`has_wiki`：是否启用对应功能。

响应示例（节选）：
```json
{
  "name": "demo-repo",
  "private": true,
  "html_url": "https://github.com/yourname/demo-repo",
  "default_branch": "main"
}
```

### 示例 4：创建 Gist（演示 `gist` 权限）

```bash
curl -s -X POST \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/gists \
  -d '{
    "description": "API-created gist",
    "public": false,
    "files": {
      "hello.txt": { "content": "Hello from API" }
    }
  }'
```
- 方法与路径：`POST /gists`
- 请求体参数解释：
  - `description`：Gist 描述。
  - `public`：是否公开；私密建议 `false`。
  - `files`：文件字典，键为文件名，值为内容对象。

响应示例（节选）：
```json
{
  "html_url": "https://gist.github.com/xxxxxxxx",
  "public": false,
  "files": { "hello.txt": { "size": 14 } }
}
```

---

## 4. 权限说明（核心项）

- 细粒度令牌（Fine-grained）仓库权限：
  - Contents（读/写）：读取与推送代码内容；对应大多数 Git 操作与文件 API。
  - Issues（读/写）：创建、修改、查询 Issue。
  - Pull requests（读/写）：创建与管理 PR。
  - Actions（读/写）：管理 GitHub Actions 工作流与运行。
  - Packages（读/写）：读取与发布 GitHub Packages。
  - Secrets（读/写）：管理仓库/环境密钥。
  - Metadata（读）：读取基础元数据（通常默认允许）。

- 经典令牌（Classic）常用 scopes：
  - `repo`：访问私有与公共仓库的完整权限（谨慎授予）。
  - `public_repo`：仅公共仓库。
  - `workflow`：管理 Actions 工作流。
  - `gist`：创建与读取 Gist。
  - `read:org` / `admin:org`：读取/管理组织信息（极度谨慎）。
  - `read:user`、`user:email`：读取用户基础信息与邮箱。
  - `read:packages`、`write:packages`：读取/发布软件包。
  - `delete_repo`：删除仓库（极度谨慎）。

---

## 5. 安全最佳实践

- 最小权限与细粒度令牌优先；按仓库限定访问范围。
- 设置过期时间并定期轮换；撤销不再使用的令牌。
- 将 Token 存储在环境变量或密钥管理器；避免写入代码仓库与日志。
- 在 CI/CD（如 GitHub Actions）使用仓库或组织级别的 Secrets，而非硬编码。
- 使用 HTTPS 与官方 API 域名；避免第三方代理泄漏。
- 避免在命令历史中泄漏（用环境变量，避免 `--header "...token..."` 直接写死）。
- 启用组织/仓库的 Secret Scanning 与 Dependabot 警报。

---

## 6. 常见问题速查

- 401 Unauthorized：检查 `Authorization` 头与 Token 是否过期、权限是否足够。
- 403 Forbidden：检查权限范围（scopes/permissions）与组织策略限制。
- 404 Not Found：确认资源是否存在、令牌是否可见该仓库（组织/私有仓库）。