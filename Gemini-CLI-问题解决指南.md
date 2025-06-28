# Gemini CLI 问题解决指南

本文档整理了使用 Gemini CLI 过程中常见的问题及其解决方案。

## 目录

- [安装相关](#安装相关)
- [认证问题](#认证问题)
- [编码问题](#编码问题)
- [API权限问题](#api权限问题)

---

## 安装相关

### Q1: 两种安装方法有什么区别？

**问题描述：** `npx https://github.com/google-gemini/gemini-cli` 和 `npm install -g @google/gemini-cli` 有什么不同？

**解答：**

| 特性 | npx 方式 | 全局安装方式 |
|------|----------|-------------|
| **安装方式** | 临时运行，每次从GitHub下载 | 永久安装到系统全局环境 |
| **版本管理** | 始终最新版本 | 版本固定，需手动更新 |
| **网络依赖** | 每次使用都需要网络连接 | 安装后可离线使用 |
| **磁盘占用** | 不占用本地存储 | 占用磁盘空间 |
| **启动速度** | 较慢（需下载） | 较快（本地运行） |

**推荐使用场景：**
- **偶尔使用**：选择 npx 方式
- **频繁使用**：选择全局安装方式

### Q2: 如何更新全局安装的 Gemini CLI？

**问题描述：** 使用 `npm install -g @google/gemini-cli` 安装后，如何更新到最新版本？

**解决方案：**

```powershell
# 检查当前版本
gemini --version

# 检查是否有新版本
npm outdated -g @google/gemini-cli

# 更新到最新版本
npm update -g @google/gemini-cli

# 或强制重新安装最新版本
npm install -g @google/gemini-cli@latest

# 验证更新成功
gemini --version
```

---

## 认证问题

### Q3: 国内用户如何配置代理使用 Gemini CLI？

**问题描述：** 国内用户启动 Gemini CLI 后，在认证步骤一直卡在"登录中"或超时，但浏览器可以正常访问 Google 服务。

**原因分析：** 
1. 国内网络环境无法直接访问 Google 服务
2. 浏览器使用了代理设置，但 Gemini CLI 没有使用代理
3. OAuth2 认证需要访问 Google 服务器，受网络限制影响

**解决方案：**

#### 配置 Node.js 使用代理（已验证可行）
```powershell
# 获取浏览器的代理设置后，配置环境变量
# 替换为您的实际代理地址和端口
$env:HTTP_PROXY="http://proxy-server:port"
$env:HTTPS_PROXY="http://proxy-server:port"

# 如果代理需要认证
$env:HTTP_PROXY="http://username:password@proxy-server:port"
$env:HTTPS_PROXY="http://username:password@proxy-server:port"

# 启动 gemini
gemini
```

**操作步骤：**
1. **获取代理信息**：从浏览器设置中查看代理服务器地址和端口
2. **配置环境变量**：在 PowerShell 中设置 HTTP_PROXY 和 HTTPS_PROXY
3. **启动 Gemini**：配置完成后启动 gemini 进行 OAuth2 认证
4. **验证连接**：确保认证过程能够正常完成

**注意事项：**
- 如果之前认证失败，可以清除缓存重试：`Remove-Item -Path "$env:USERPROFILE\.gemini\oauth_creds.json" -ErrorAction SilentlyContinue`
- 确保代理服务器地址和端口正确
- 如果代理需要用户名密码，请正确配置认证信息

---

## 编码问题

### Q4: Shell 命令输出中文乱码怎么解决？

**问题描述：** 执行包含中文输出的命令时显示乱码。

**原因分析：** Gemini CLI 强制使用 UTF-8 解码器处理输出，但 Windows cmd.exe 默认使用 GBK 编码输出中文。

**解决方案：**

#### 正确的方法（在 Gemini 内设置）
```bash
# 1. 进入 Shell 模式
!

# 2. 设置编码为 UTF-8
chcp 65001

# 3. 现在可以正常显示中文
dir
echo 你好世界
date /t

# 4. 退出 Shell 模式
!
```

#### 单次命令设置编码
```bash
# 在命令前加上编码设置
!chcp 65001 && dir
!chcp 65001 && echo 你好世界
```

**注意：** 不要在启动 Gemini 前设置 `chcp 65001`，这样无效。必须在 Gemini 的 Shell 模式中设置。

---

## API权限问题

### Q5: 出现 403 权限错误怎么办？

**问题描述：** 认证成功后使用时出现类似错误：
```
[API Error: {
    "error": {
      "code": 403,
      "message": "Gemini for Google Cloud API has not been used in project xxx before or it is disabled..."
    }
}]
```

**原因分析：** 
认证成功后使用的 Google Cloud Project 中没有启用 Gemini for Google Cloud API。

**解决方案：**

#### 激活 Google Project 下的 Gemini API
1. **访问 Google Cloud Console**：
   - 打开错误信息中提供的链接，或访问 [Google Cloud Console](https://console.cloud.google.com/)
   - 选择对应的项目（错误信息中显示的 project ID）

2. **启用 Gemini for Google Cloud API**：
   - 在项目中搜索并选择 "Gemini for Google Cloud API"
   - 点击 "启用" 按钮

3. **等待 API 激活**：
   - 等待 5-10 分钟让 API 激活生效

4. **重新启动 Gemini**：
   ```powershell
   gemini
   ```

**注意事项：**
- 确保在正确的 Google Cloud Project 中启用 API
- API 启用后需要等待几分钟才能生效
- 如果仍有问题，可以清除认证缓存重试：`Remove-Item -Path "$env:USERPROFILE\.gemini\oauth_creds.json" -ErrorAction SilentlyContinue`

---

### 常用命令速查

```bash
# 系统信息
!ver                    # Windows 版本
!whoami                 # 当前用户
!systeminfo             # 系统详细信息

# 文件操作
!dir                    # 列出文件
!type filename          # 查看文件内容
!copy source dest       # 复制文件

# 网络测试
!ping google.com        # 网络连通性
!nslookup domain.com    # DNS 解析
```

---

## 故障排除检查清单

遇到问题时，按以下顺序检查：

- [ ] 确认使用正确的安装和启动方式
- [ ] 检查网络连接和代理设置
- [ ] 验证认证方式（推荐 API Key）
- [ ] 确认使用 Windows 命令语法
- [ ] 在 Shell 模式中设置正确的字符编码
- [ ] 查看具体错误信息并对照本指南

---

**最后更新：** 2025年6月
**适用版本：** Gemini CLI 0.1.7