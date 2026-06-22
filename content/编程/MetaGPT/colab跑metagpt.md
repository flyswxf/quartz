
1. 下载依赖包, 并设置metagpt作为全局参数入口, 可以直接在cmd中使用'metagpt'访问
	1. !git clone https://github.com/geekan/MetaGPT && cd MetaGPT && pip install --upgrade -e .
2. 设置config文件, 位于/root/.metagpt/config2.yaml, 可以直接双击文件编辑
	1. !metagpt --init-config
```yaml
	# Full Example: https://github.com/geekan/MetaGPT/blob/main/config/config2.example.yaml
	# Reflected Code: https://github.com/geekan/MetaGPT/blob/main/metagpt/config2.py
	# Config Docs: https://docs.deepwisdom.ai/main/en/guide/get_started/configuration.html
	llm:
	  api_type: "openai"
	  model: "ecnu-max"
	  base_url: "https://chat.ecnu.edu.cn/open/api/v1"
	  api_key: "sk-5c69362740cf4cfa9444e8b7ebce657a"
	
	repair_llm_output: true
```
3. 执行命令
	1. !metagpt "Create a 2048 game"
4. 进入工作文件夹
	1. cd content/MetaGPT/workspace/2048_game_1752108883/
5. 对外设置端口, 可以在本地访问(方法来自[python - 是否有在 Google Colab 上运行 Web 应用程序的通用方法？ - SegmentFault 思否](https://segmentfault.com/q/1010000043293753))
	1. 5173为我设置的端口号, 这个端口号可以随便设置, 但是要和后续的端口号相同
```python
from google.colab.output import eval_js

print(eval_js("google.colab.kernel.proxyPort(5173)"))
#会返回类似结果: https://z4spb7cvssd-496ff2e9c6d22116-8000-colab.googleusercontent.com/
```
6. 启动服务, 访问步骤五输出的网址, 查看报错内容
	1. `!npm run dev -- --host 0.0.0.0 --port 5173`
	2. 报错内容类似
	   Blocked request. This host ("m-s-2n5fzveq69pkp.us-west4-a.c.codatalab-user-runtimes.internal") is not allowed. 
	   o allow this host, add "m-s-2n5fzveq69pkp.us-west4-a.c.codatalab-user-runtimes.internal" to \`server.allowedHosts\` in vite.config.js.
7. 按照报错指示在工作文件夹中的vite.config.js中新增条目, 注意端口号要与之前的一致
```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vite.dev/config/
export default defineConfig({
  plugins: [react()],
  // server为新增的, 原始文件内不会包含
  server: {
    // 添加错误信息中要求的主机名
    allowedHosts: [js
      "m-s-2n5fzveq69pkp.us-west4-a.c.codatalab-user-runtimes.internal"
    ],
    
    // 同时添加以下配置以确保正常工作
    host: true,          // 监听所有接口
    strictPort: true,    // 严格使用指定端口
    port: 5173           // 确保与您使用的端口一致
  }
})
```
8. 再次启动服务
	1. `!npm run dev -- --host 0.0.0.0 --port 5173`
	2. 返回类似结果
	   \> 2048_game_1752108883@0.0.0 dev 
	   \> vite --host 0.0.0.0 --port 5173 
	   
	   1;1H 
	   VITE v7.0.3 ready in 735 ms
	    ➜ Local: [http://localhost](http://localhost/):5173/ 
	    ➜ Network: [http://172.28.0.12](http://172.28.0.12/):5173/ 
	    ➜ press h + enter to show help
	3. 访问步骤五输出的结果即可