# ☁️ Desire 优选订阅生成器 (Cloudflare Worker)

这是一个基于 Cloudflare Workers + D1 数据库 + KV 存储的**全栈轻量级代理节点管理与订阅分发系统**。
它可以将你干瘪的基础节点（VLESS / VMess / Trojan），与你私人维护的“优选 IP 库”或“外部公开源”自动结合，瞬间裂变生成包含数百个优质线路的订阅链接，并附带二维码供客户端一键扫码使用。

## ✨ 核心特性

- 🌗 **双模式引擎**：支持使用**本地 D1 数据库私有节点**，或实时拉取**外部公开源 (TXT/CSV/API)**，数据隔离互不干扰。
- 📱 **完美客户端支持**：自动生成 `vless://` / `vmess://` 订阅，并在网页端实时渲染 **二维码** 供手机便捷扫码。
- 🛡️ **智能容错与防爆**：当外部链接失效或拉取被拦截时，系统不会导致客户端解析崩溃，而是会下发友好的**伪装错误节点**提示具体原因。
- 📊 **极致可视化后台**：内置现代化毛玻璃 UI 管理后台。支持：
  - **动态分页 & 防抖搜索**：轻松管理成百上千条优选 IP。
  - **弹窗编辑 & 状态控制**：一键开启/禁用单节点，随时修改备注名称。
  - **智能清洗**：一键去重、一键按地区/优先级排序。
- 🔒 **双重安全防护**：支持 `ADMIN_PASSWORD` 后台独立密码验证与 `SUB_TOKEN` 前台订阅防白嫖机制。
- ⚡ **边缘缓存保护**：自动利用 Cloudflare `caches.default` 缓存机制，极大降低 D1 数据库读取压力。

---

## 🛠️ 部署指南

### 第一步：准备 Cloudflare 资源
1. 登录 Cloudflare 控制台。
2. 创建一个 **D1 数据库**（例如命名为 `sub-db`）。
3. 创建一个 **KV 命名空间**（例如命名为 `sub-kv`）。

### 第二步：初始化数据库表结构
进入你刚刚创建的 D1 数据库，点击 **控制台 (Console)**，粘贴并执行以下 SQL 语句来创建表：
\`\`\`sql
CREATE TABLE IF NOT EXISTS ips(
    id INTEGER PRIMARY KEY, 
    ip TEXT UNIQUE NOT NULL, 
    name TEXT, 
    active INTEGER DEFAULT 1, 
    priority INTEGER DEFAULT 0
);
CREATE UNIQUE INDEX IF NOT EXISTS idx_ips_ip ON ips(ip);
CREATE INDEX IF NOT EXISTS idx_ips_active ON ips(active);
CREATE INDEX IF NOT EXISTS idx_ips_priority ON ips(priority);
\`\`\`

### 第三步：创建 Worker 并绑定
1. 创建一个新的 Cloudflare Worker。
2. 进入 Worker 的 **设置 (Settings)** -> **绑定 (Bindings)**：
   - 添加 D1 数据库绑定，变量名称 **必须** 设置为 `DB`。
   - 添加 KV 命名空间绑定，变量名称 **必须** 设置为 `TASK_KV`。
3. 进入 **设置 (Settings)** -> **变量和机密 (Variables and Secrets)**，添加以下环境变量（可选但强烈建议）：
   - `ADMIN_PASSWORD`：你的后台管理密码。
   - `SUB_TOKEN`：你的前台订阅安全验证密钥。

### 第四步：部署代码
将本仓库中的 `worker.js` 里的全部代码，复制并粘贴到你 Cloudflare Worker 的代码编辑器中，点击 **保存并部署**。

---

## 💻 使用说明

### 1. 录入与管理私有 IP (管理后台)
- 访问：`https://你的worker域名.workers.dev/admin` (需手动输入 `/admin` 访问)。
- 输入你设置的 `ADMIN_PASSWORD` 登录。
- 在“批量导入”框中，粘贴你搜集到的优选 IP（支持自动从完整的 VLESS/VMess 链接中剥离 IP）。

### 2. 生成优选订阅 (用户前台)
- 访问：`https://你的worker域名.workers.dev/`。
- 填入你的单个基础节点链接。
- **选择模式**：你可以选择使用刚建好的**本地私有优选库**，或者填入第三方 URL 使用**外部公开库**。
- 若设置了 `SUB_TOKEN`，请在安全 Token 框内填入。
- 点击“生成”，即可获得专属 Base64 订阅链接及二维码！

---
*本项目由 Desire 维护。如果你觉得好用，欢迎点个 ⭐ Star!*
