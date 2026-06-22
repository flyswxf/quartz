> 由于原有 Clash 版本过老，无法解析新的配置信息，因此选择使用新版 Clash Verge Rev。

---

## 1. 下载

- [最新版下载 | Clash Verge Rev](https://clash-verge-rev.cc/download.html)
- 由于阿里云服务器系统较老，无法满足最新版依赖，推荐使用 [clash-verge-1.7.7-1.x86_64.rpm](https://github.com/clash-verge-rev/clash-verge-rev/releases/download/v1.7.7/clash-verge-1.7.7-1.x86_64.rpm)
  - 可在服务器上直接 wget 下载，若被 github 卡住，可本地下载后通过 IDE 拖动上传
- 下载完成后，clash-verge 会自动将路径加入服务器 Path，可直接使用 `clash-verge` 命令

---

## 2. 运行

- Clash Verge 是图形化 Linux 软件，仅命令行无法直接使用，需借助虚拟可视化软件：

  ```bash
  Xvfb :1 -screen 0 1024x768x16 &
  export DISPLAY=:1
  clash-verge &
  ```
  - `Xvfb :1 -screen 0 1024x768x16 &`：启动一个虚拟显示器，:1 表示显示编号，screen 0 指定分辨率和色深，& 表示后台运行。
  - `export DISPLAY=:1`：设置 DISPLAY 环境变量，指定后续图形程序输出到虚拟显示器。
  - `clash-verge &`：启动 clash-verge 程序，并在后台运行。

- 若需在命令行窗口翻墙，需先设置代理：

  ```bash
  export http_proxy="http://127.0.0.1:7897"
  export https_proxy="http://127.0.0.1:7897"
  ```
  - `export http_proxy=...` 和 `export https_proxy=...`：设置终端的 HTTP/HTTPS 代理，使命令行流量通过 clash-verge 代理端口转发。

- 若要在本机（windows环境）使用代理，且不使用clash verge自带的全局代理功能：

```bash
$env:http_proxy='http://127.0.0.1:7897'
$env:https_proxy='http://127.0.0.1:7897'
```

- 至此，设置完成，可正常使用。

---

## 3. 配置

- 在服务器命令行中无法直接解析大航海 VPN 的订阅地址，可先用一键导入功能在本地导入，再在本地打开配置文件，复制内容。
![[assets/配置编辑界面.png]]
- 在服务器中创建 `config.yaml` 并粘贴内容，然后用 `mv` 命令将 config.yaml 移动到 clash-verge 的配置目录。

  ```bash
  mv config.yaml /root/.local/share/io.github.clash-verge-rev.clash-verge-rev/config.yaml
  ```

- 可通过如下命令查找 clash-verge 的配置文件位置：

  ```bash
  find ~ -type f -name "config.yaml"
  ```
  - `find`：Linux 下用于查找文件的命令。
  - `~`：表示当前用户的主目录（如 /root 或 /home/用户名）。
  - `-type f`：只查找普通文件（不包括目录、链接等）。
  - `-name "config.yaml"`：查找文件名为 config.yaml 的文件。

- 常见的配置文件路径有：

  - `/root/.local/share/io.github.clash-verge-rev.clash-verge-rev/config.yaml`（实际使用的配置文件）
  - `/root/.config/clash/config.yaml`


