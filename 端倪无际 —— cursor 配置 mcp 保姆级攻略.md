---
# themes: juejin, github, smartblue, cyanosis, channing-cyan, fancy, hydrogen, condensed-night-purple, greenwillow, v-green, vue-pro, healer-readable, mk-cute, jzman, geek-black, awesome-green, orange, scrolls-light, simplicity-green, arknights, vuepress, Chinese-red, nico, devui-blue
theme: arknights
---

| 计算机发展史阶段    | 技术跃迁本质 |
| ------------------- | ------------ |
| 大型机 → 个人电脑   | 资源民主化   |
| 单机软件 → 云计算   | 算力泛在化   |
| 专有协议 → 开放标准 | 生态指数增长 |
| 功能机 → 智能终端   | 交互范式颠覆 |

## MCP 简介

**MCP（Model Context Protocol，模型上下文协议）** 是由 Anthropic 推出的开放协议，旨在解决大型语言模型（LLM）与外部数据源、工具之间的通信标准化问题。

*   **类比**：如同 USB-C 接口统一了电子设备的连接方式，MCP 为 AI 应用提供了标准化接口，支持灵活连接文件系统、数据库、API 等资源
*   **核心目标**：通过协议标准化，让 LLM 能够安全、高效地访问本地和远程数据源，解决传统 AI 工具开发中的“数据孤岛”和“平台碎片化”问题

MCP 基于 **客户端-服务器架构**，包含三大核心模块：

1.  **资源（Resources）**提供数据读取接口，例如文件系统、数据库查询结果或 API 响应数据
2.  **工具（Tools）**定义可执行函数（如文件写入、数据检索），供 LLM 调用以完成操作任务
3.  **提示（Prompts）**预设任务模板（如“生成周报”），指导 AI 完成复杂工作流

**典型架构流程**：

**Host**（如 Cursor IDE）发起请求 → **Client** 连接 Server → **Server**（如文件系统服务）执行操作 → 返回结果至 LLM 生成响应

![image-20250524103538365](https://github.com/lizy-coding/blog_documents/blob/main/image/image-20250524103538365.png)


## Windows 下 Cursor MCP Server 配置全指南（2025.03.29 更新）

***

#### 一、核心环境配置

**1. Node.js 与 npm 基础环境**

*   **安装要求**：Node.js ≥18.x（推荐 22.14.0 LTS）
*   **验证安装**：
    ```powershell
    node -v  # 预期输出 v22.14.0  
    npm -v   # 预期输出 10.9.2+
    ```
*   **镜像加速**（国内用户必选）：
    ```powershell
    npm config set registry https://registry.npmmirror.com
    ```

**2. 全局安装 MCP 服务包**

*   文件系统服务：
    ```powershell
    npm install -g @modelcontextprotocol/server-filesystem
    ```
*   GitHub 服务：
    ```powershell
    npm install -g @modelcontextprotocol/server-github
    ```

> *提示：全局安装路径默认为 `C:\Users\<用户名>\AppData\Roaming\npm\node_modules`，需确保无读写权限限制*

***

#### 二、MCP 服务查找与选择

**1. 官方推荐服务**

*   **文件系统**：`@modelcontextprotocol/server-filesystem`（支持本地目录读写）
*   **GitHub**：`@modelcontextprotocol/server-github`（需配置 PAT 令牌）
*   **其他服务**：通过 [Awesome-MCP-Servers](https://github.com/punkpeye/awesome-mcp-servers) 查找社区服务

**2. 服务兼容性验证**

*   检查服务是否支持 **Windows 命令行调用模式**
*   优先选择明确标注 `windows-compatible` 的包

***

#### 三、Cursor 配置流程

**创建 MCP 配置文件**

手动路径创建：项目根目录或全局配置路径（`C:\Users\<用户名>\.cursor\mcp.json`）

**或者在软件内创建**

`Ctrl + Shift + P` 进入编译器搜索 setting 进入配置

![image-20250524103856554](https://github.com/lizy-coding/blog_documents/blob/main/image/image-20250524103856554.png)

笔者界面是已经创建好的，没创建 json 的话右上角有创建全局的选项

![image-20250524103944071](https://github.com/lizy-coding/blog_documents/blob/main/image/image-20250524103944071.png)

举例子：使用 filesystem 及 github mcp 的配置如下

**windows 用户**
//"D:\desktop\test" 替换为你的本地文件夹路径
//your\_token\_here 替换为在 [Personal Access Tokens (Classic)](https://github.com/settings/tokens) 生成的个人 token

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "cmd",
      "args": [
        "/c", 
        "npx", 
        "-y", 
        "@modelcontextprotocol/server-filesystem",
        "D:\\desktop\\test"
      ]
    },
    "github": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "your_token_here"
      }
    }
  }
}
```

**Mac 或 linux**

```json
{
  "mcpServers": {
    "filesystem": {
      "command":  "npx",
      "args": [
        "-y", 
        "@modelcontextprotocol/server-filesystem",
        "D:\\desktop\\test"
      ]
    },
    "github": {
      "command":  "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "your_token_here"
      }
    }
  }
}
```

**Windows 特有配置要点**

*   **强制使用 cmd 解释器**：避免 PowerShell 执行策略限制（如 `Restricted` 模式）
*   **路径转义处理**：
    *   使用双反斜杠 `\\` 或正斜杠 `/`（如 `D:/desk/code`）
    *   含空格路径需双引号包裹：`"C:/Program Files"`
*   **环境变量注入**：敏感数据（如 GitHub Token）通过 `env` 字段传递

***

#### 四、服务启动与验证

**1. 手动启动调试**
或者直接步骤 2 在 cursor 启动也可以

```powershell
# 文件系统服务验证
cmd /c npx -y @modelcontextprotocol/server-filesystem D:\\desk\\code --log-level=debug

# GitHub 服务验证（需替换真实 Token）
cmd /c npx -y @modelcontextprotocol/server-github --env GITHUB_PERSONAL_ACCESS_TOKEN=your_token_here
```

> *预期输出：服务启动日志及端口监听信息（如 `Listening on port 3000`）*

**2. Cursor 界面验证**

1.  打开 `Settings → Features → MCP` ，点击服务的刷新角标会启动 cmd 窗口，不要关闭
2.  检查服务状态指示灯：
    *   **绿色**：运行正常
    *   **红色**：查看日志（点击服务右侧的 `View Logs`）

![image-20250524104015103](https://github.com/lizy-coding/blog_documents/blob/main/image/image-20250524104015103.png)

（可选）开启 Enable  auto-run model，后续使用 mcp 自动授权

![image-20250524104143768](https://github.com/lizy-coding/blog_documents/blob/main/image/image-20250524104143768.png)

3.  功能测试指令示例：

使用命令模式

```plaintext
/mcp filesystem  D:\desk\code\flutter\flutter_study 下有什么文件
```

使用 agent 模式

![image-20250524104333210](https://github.com/lizy-coding/blog_documents/blob/main/image/image-20250524104333210.png)

![image-20250524104354141](https://github.com/lizy-coding/blog_documents/blob/main/image/image-20250524104354141.png)

***

#### 五、常见问题与解决策略

| **问题现象**           | **原因分析**         | **解决方案**                                                 |
| ---------------------- | -------------------- | ------------------------------------------------------------ |
| **Client closed** 错误 | 服务进程未启动或崩溃 | 1. 检查全局包安装路径权限<br>2. 以管理员运行 Cursor          |
| **路径访问被拒绝**     | Windows 权限限制     | 1. 右键文件夹 → 安全 → 添加用户完全控制权限<br>2. 使用非系统盘路径 |
| **GitHub 401 错误**    | Token 失效或权限不足 | 1. [重新生成 PAT](https://github.com/settings/tokens) 并更新配置<br>2. 检查 `repo` 权限是否勾选 |
| **npx 下载超时**       | 网络连接问题         | 1. 换用 `npm exec` 替代 `npx`<br>2. 配置代理：`npm config set proxy http://127.0.0.1:1080` |
| **服务间歇性断开**     | 防火墙拦截           | 1. 允许 Node.js 通过防火墙<br>2. 指定固定端口：`--port 3000` |

***

#### 六、高阶优化建议

1.  **容器化部署**：
    
    ```powershell
    docker run -v D:\desk\code:/data modelcontextprotocol/server-filesystem
    ```
    *避免环境依赖冲突，适合生产环境*
    
2.  **多服务负载均衡**：\
    在 `mcp.json` 中为同一服务配置多个实例，通过不同端口分流请求

3.  **日志监控**：\
    使用 `--log-file mcp.log` 参数记录运行日志，配合 [Wireshark](https://www.wireshark.org/) 分析网络流量

***

**扩展阅读**：

*   [MCP 协议官方文档](https://modelcontextprotocol.io)
*   [Windows 下 MCP 最佳实践（火山引擎）](https://developer.volcengine.com/articles/725234543732871)
*   [Cursor MCP 异常诊断手册](https://docs.roocode.com/mcp-troubleshooting)
*   [Introduction - Model Context Protocol](https://modelcontextprotocol.io/introduction)
