# ☁️ Desire 优选订阅生成器 (Cloudflare Worker)

这是一个基于 Cloudflare Workers + D1 数据库 + KV 存储的**全栈轻量级代理节点管理与订阅分发系统**。
它可以将你的基础节点（VLESS / VMess / Trojan），与你私人维护的“优选 IP 库”或“外部公开源”自动结合，瞬间裂变生成包含数百个优质线路的订阅链接，并附带二维码供客户端扫码使用。

## ✨ 核心特性

- 🌗 **双模式引擎**：支持使用**本地 D1 数据库私有节点**，或实时拉取**外部公开源 (API/TXT/CSV)**，数据隔离互不干扰。
- 📱 **完美扫码支持**：前端页面实时渲染 **二维码**，方便手机端用户一键扫码添加订阅。
- 🛡️ **智能容错防爆**：当外部库拉取失败时，系统会下发友好的**错误提示节点**，防止客户端因解析失败而崩溃。
- 📊 **极致可视化后台**：内置现代化毛玻璃 UI 管理面板，包含以下高级功能：
  - **动态分页 & 智能页码跳转**：轻松管理成百上千条 IP 数据。
  - **防抖 (Debounce) 搜索**：无需回车，输入内容即刻自动过滤 IP 或备注。
  - **单节点精细管理**：支持弹窗修改 IP 信息、一键切换“启用/禁用”状态。
  - **智能清洗**：一键去重、一键按地区排序、批量导入/精准删除。
- 🔒 **多重安全防护**：
  - **后台验证**：使用 `ADMIN_PASSWORD` 进行 Basic Auth 强力拦截。
  - **订阅防盗**：支持 `SUB_TOKEN` 验证，防止接口被他人滥用生成订阅。
  - **入口隐藏**：前台无后台入口，需手动访问 `/admin` 路径。
- ⚡ **边缘缓存保护**：集成 Cloudflare `caches.default` 缓存机制，极大降低 D1 数据库读取次数，节省额度。

---

## 🛠️ 部署指南

### 第一步：准备 Cloudflare 资源
1. 登录 Cloudflare 控制台。
2. 创建一个 **D1 数据库**（建议命名为 `sub-db`）。
3. 创建一个 **KV 命名空间**（建议命名为 `TASK_KV`）。

### 第二步：初始化数据库
进入 D1 数据库管理页面，点击 **控制台 (Console)**，执行以下 SQL 语句：
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

### 第三步：Worker 配置与部署
1. 创建 Worker 并在 **设置 (Settings)** -> **绑定 (Bindings)** 中：
   - 添加 D1 数据库绑定，变量名为 `DB`。
   - 添加 KV 空间绑定，变量名为 `TASK_KV`。
2. 在 **变量和机密 (Variables and Secrets)** 中添加：
   - `ADMIN_PASSWORD`：设置你的后台管理密码。
   - `SUB_TOKEN`：设置你的订阅验证密钥（可选）。
3. 将仓库中的 `worker.js` 代码全部粘贴到编辑器中并 **保存部署**。

---

## 💻 使用说明

### 1. 管理优选 IP (后台)
- 访问：`https://你的域名/admin`。
- 输入密码登录后，你可以：
  - 批量导入 IP 列表或完整的节点链接。
  - 通过**搜索框**快速定位某个失效的 IP。
  - 点击**编辑**或**启用/禁用**按钮灵活调整节点。

### 2. 生成订阅链接 (前台)
- 访问：`https://你的域名/`。
- **本地模式**：默认使用你在后台维护的精品 IP 库。
- **外部模式**：选择“外部公开优选库”，可一键切换电信、联通、移动等三网专用优选 API。
- 生成后可直接点击链接复制或手机扫码导入。

---
*本项目由 Desire 维护。如果你觉得好用，请点个 ⭐ Star 支持一下！*
