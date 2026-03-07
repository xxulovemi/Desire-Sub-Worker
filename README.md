# ☁️ Desire 优选订阅生成器 (Cloudflare Worker)

这是一个基于 Cloudflare Workers + D1 数据库 + KV 存储的轻量级代理节点优选订阅生成器。
它可以将你干瘪的基础节点（VLESS / VMess / Trojan），与你私人维护的“优选 IP 库”自动结合，裂变生成包含数百个优质线路的订阅链接，供客户端一键使用。

## ✨ 核心特性

- **🚀 多协议支持**：支持解析和生成 VLESS、VMess、Trojan 协议节点。
- **📊 独立管理后台**：内置现代化、带有毛玻璃特效的 Web UI 管理后台。
- **🔍 智能管理**：支持批量导入/删除、精准防抖搜索、一键去重、地区智能排序。
- **🛡️ 安全防护**：支持 `ADMIN_PASSWORD` 后台密码验证 与 `SUB_TOKEN` 前台订阅防白嫖机制。
- **⚡ 极致性能**：利用 Cloudflare 边缘缓存 (`caches.default`)，保护 D1 数据库免受客户端频繁刷新消耗。

---

## 🛠️ 部署指南

### 第一步：准备 Cloudflare 资源
1. 登录 Cloudflare 控制台。
2. 创建一个 **D1 数据库**（例如命名为 `sub-db`）。
3. 创建一个 **KV 命名空间**（例如命名为 `sub-kv`）。

### 第二步：初始化数据库表结构
进入你刚刚创建的 D1 数据库，点击 **控制台 (Console)**，粘贴并执行以下 SQL 语句来创建必要的表：
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
   - `SUB_TOKEN`：你的前台订阅防白嫖密钥。

### 第四步：部署代码
将本仓库中的 `worker.js` 里的全部代码，复制并粘贴到你 Cloudflare Worker 的代码编辑器中，点击 **保存并部署**。

---

## 💻 使用说明

### 1. 录入优选 IP (管理后台)
- 访问：`https://你的worker域名.workers.dev/admin` (自动隐藏入口，需手动输入 `/admin` 访问)。
- 输入你设置的 `ADMIN_PASSWORD` 登录。
- 在“批量导入”框中，粘贴你搜集到的优选 IP（格式支持：`纯IP:端口` 或 `带有IP的完整VLESS/VMess链接`）。

### 2. 生成订阅 (用户前台)
- 访问：`https://你的worker域名.workers.dev/`。
- 填入你搭建的单个基础节点链接。
- 若设置了 `SUB_TOKEN`，请在安全 Token 框内填入。
- 点击“生成优选订阅”，即可获得专属于你的 Base64 订阅链接，可直接导入 v2rayN、Clash、Shadowrocket 等客户端使用！

---
*本项目由 Desire 维护。如果你觉得好用，欢迎点个 ⭐ Star!*
