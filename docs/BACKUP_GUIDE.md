# 凭证自动备份系统使用指南

## 📋 功能概述

自动备份系统会在服务运行期间：
- ✅ 每小时自动备份凭证文件到 GitHub 私有仓库
- ✅ 使用东八区（UTC+8）时间
- ✅ 保留最近 96 小时（4天）的备份
- ✅ 每天凌晨0点自动清理超过4天的旧备份
- ✅ 文件夹按年/月/日/时结构组织（简洁高效）

## 🔧 配置说明

### 1. 修改 `creds/config.toml`

```toml
[backup]
enabled = true  # 启用备份功能
github_token = "your_github_personal_access_token"  # GitHub Personal Access Token
github_repo = "https://github.com/username/repo-name.git"  # GitHub 仓库地址
backup_files = ["creds.toml", "accounts.toml"]  # 要备份的文件列表
auto_commit_message = "🔒 自动备份凭证文件"  # 提交信息
```

### 2. 创建 GitHub 私有仓库

1. 在 GitHub 上创建一个**私有仓库**（如 `creds-backup`）
2. 创建 Fine-grained Personal Access Token：
   - 访问：Settings → Developer settings → Personal access tokens → Fine-grained tokens
   - 权限设置：
     - **Repository access**: 选择你的备份仓库
     - **Repository permissions** → **Contents**: Read and write
   - 复制生成的 token

### 3. 配置完成

将 token 和仓库地址填入 `config.toml` 即可。

## 🚀 使用方式

### 自动模式（推荐）

服务启动时自动开始备份：

```bash
python web.py
```

备份调度器会在服务启动时自动运行，每小时执行一次备份。

### 手动执行备份

如果需要手动执行一次备份：

```bash
python backup_creds.py
```

## 📁 备份文件结构

```
creds-backup/  (GitHub 仓库)
├── 2025/
│   ├── 11/
│   │   ├── 24/  (11月24日)
│   │   │   ├── 00/  (凌晨0点)
│   │   │   │   ├── creds_400_1124_000012.toml.bak
│   │   │   │   └── accounts_15_1124_000012.toml.bak
│   │   │   ├── 01/  (凌晨1点)
│   │   │   ├── 02/  (凌晨2点)
│   │   │   ├── ...
│   │   │   └── 23/  (晚上11点)
│   │   ├── 25/  (11月25日 - 24个小时文件夹)
│   │   ├── 26/  (11月26日 - 24个小时文件夹)
│   │   └── 27/  (11月27日 - 24个小时文件夹)
│   │       ├── 10/
│   │       │   ├── creds_386_1127_100523.toml.bak
│   │       │   └── accounts_15_1127_100523.toml.bak
│   │       └── 11/
│   │           ├── creds_386_1127_110738.toml.bak
│   │           └── accounts_15_1127_110738.toml.bak
```

**结构说明**：
- 最多保留 **4天** 的备份（96小时）
- 每天有 **24个小时文件夹**（00-23）
- 每小时备份一次到对应的小时文件夹
- 每天凌晨0点自动删除最早的一天

### 文件命名规则

格式：`{filename}_{count}_{MMDD}_{HHmmss}.toml.bak`

- `filename`: 原文件名（如 creds、accounts）
- `count`: 凭证数量（如 400 表示有 400 个秘钥）
- `MMDD`: 月日（如 1127 表示 11月27日）
- `HHmmss`: 时分秒（如 120236 表示 12:02:36）

示例：
- `creds_400_1127_100828.toml.bak`
  - 400 个秘钥
  - 11月27日 10:08:28
  - 完整路径：`2025/11/27/10/creds_400_1127_100828.toml.bak`（2025年11月27日10点）

## 🔍 监控和管理

### 查看备份日志

服务运行时会输出备份日志：

```
INFO - 凭证备份调度器已启动（每小时自动备份）
INFO - 开始执行定时备份 - 2025-01-27 10:08:28 (UTC+8)
INFO - 备份: creds_400_20250127100828.toml.bak (凭证数: 400)
INFO - 备份: accounts_150_20250127100828.toml.bak (凭证数: 150)
INFO - 下次备份将在 59 分钟后执行
```

### GitHub 仓库查看

访问你的 GitHub 私有仓库，可以看到：
- 所有备份文件按时间组织
- 每次备份的提交记录
- 可以下载任意历史版本

## ⚙️ 高级配置

### 修改备份间隔

默认每小时备份一次，如需修改，编辑 `web.py`：

```python
# 修改为每2小时备份一次
start_backup_scheduler(interval_hours=2)
```

### 修改最大备份天数

默认保留 4 天的备份，如需修改，编辑 `backup_creds.py`：

```python
self.max_backup_days = 7  # 保留 7 天的备份
```

**性能优化说明**：
- 系统每天凌晨0点执行一次清理
- 直接删除整天的文件夹，不需要逐个文件检查
- 相比逐个文件清理，性能提升显著

### 添加更多备份文件

在 `config.toml` 中添加：

```toml
backup_files = ["creds.toml", "accounts.toml", "config.toml"]
```

## 🛡️ 安全建议

1. ✅ **必须使用私有仓库** - 凭证文件包含敏感信息
2. ✅ **Token 权限最小化** - 只给 Contents 的 Read/Write 权限
3. ✅ **定期轮换 Token** - 建议每 3-6 个月更换一次
4. ✅ **不要将 config.toml 提交到公开仓库** - 添加到 `.gitignore`

## 📊 恢复备份

### 方法 1：直接从 GitHub 下载

1. 访问 GitHub 仓库
2. 找到需要恢复的备份文件
3. 下载并重命名为原文件名
4. 复制到 `creds/` 目录

### 方法 2：使用 Git 命令

```bash
# 克隆备份仓库
git clone https://github.com/username/creds-backup.git

# 找到需要的备份文件
cd creds-backup/2025/01/27/10/08/28/

# 复制到项目目录
cp creds_400_20250127100828.toml.bak /path/to/project/creds/creds.toml
```

## ❓ 故障排查

### 备份失败

检查日志中的错误信息：
- GitHub Token 是否正确
- 仓库地址是否正确
- Token 权限是否足够
- 网络连接是否正常

### 手动测试

```bash
# 测试备份脚本
python backup_creds.py

# 如果成功，会在 .backup_repo 目录看到备份文件
ls .backup_repo/
```

### 清理本地备份缓存

如果遇到问题，可以删除本地缓存重新开始：

```bash
rm -rf .backup_repo
```

下次备份时会重新克隆仓库。

## 📞 技术支持

如有问题，请检查：
1. 日志文件中的错误信息
2. GitHub 仓库的提交历史
3. config.toml 配置是否正确

---

**备份系统版本**: 1.0.0
**最后更新**: 2025-01-27
