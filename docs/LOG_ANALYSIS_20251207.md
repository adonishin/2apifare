# 服务器日志分析报告

> 分析时间：2025-12-07
> 日志来源：Docker 容器日志 (300MB)
> 日志时间范围：2025-12-05 15:30 ~ 2025-12-06 15:23

---

## 📊 问题概览

| 严重程度 | 问题类型 | 出现频率 | 影响 |
|----------|----------|----------|------|
| 🔴 严重 | 429 配额耗尽 | 极高 | 请求失败，用户体验差 |
| 🔴 严重 | 524 超时错误 | 中等 | Workers 代理超时 |
| 🟡 中等 | 403 权限错误 | 中等 | Antigravity 账号无权限 |
| 🟢 轻微 | socket 异常 | 低 | 日志噪音 |

---

## 🔴 严重问题

### 1. 429 配额耗尽 - 重复尝试问题

**现象**：
```log
[2025-12-05 15:30:58] [ERROR] Google API returned status 429 (STREAMING)
"message": "You have exhausted your capacity on this model. Your quota will reset after 5h5m47s."

[2025-12-05 15:30:58] [INFO] Rotated to credential index 1
[2025-12-05 15:30:58] [INFO] [RETRY] 429 error, switched to new credential, retrying immediately (1/5)

[2025-12-05 15:30:58] [ERROR] Google API returned status 429 (STREAMING)
"message": "You have exhausted your capacity on this model. Your quota will reset after 5h5m47s."

[2025-12-05 15:30:58] [INFO] Rotated to credential index 2
[2025-12-05 15:30:58] [INFO] [RETRY] 429 error, switched to new credential, retrying immediately (2/5)
...
```

**问题分析**：
- 系统在 **1 秒内** 连续尝试了 5 个账号，全部返回 429
- 429 错误包含两种类型：
  1. `QUOTA_EXHAUSTED` - 模型配额耗尽（gemini-2.5-pro 等）
  2. `RATE_LIMIT_EXCEEDED` - 每日请求限制（cloudaicompanion.googleapis.com）
- 系统没有有效记住哪些账号/模型系列已经 429

**影响**：
- 用户请求最终失败
- 浪费 API 调用配额
- 增加 Google API 的请求压力

**根本原因**：
- 429 系列封禁逻辑可能未正确触发
- 或封禁信息未正确传递给后续请求

---

### 2. Cloudflare 524 超时错误

**现象**：
```log
[2025-12-06 01:19:49] <title>cloudcode-pa.googleapis.com | 524: A timeout occurred</title>
[2025-12-06 03:21:06] <title>cloudcode-pa.googleapis.com | 524: A timeout occurred</title>
[2025-12-06 03:21:35] <title>cloudcode-pa.googleapis.com | 524: A timeout occurred</title>
[2025-12-06 07:37:34] <title>cloudcode-pa.googleapis.com | 524: A timeout occurred</title>
[2025-12-06 12:58:33] <title>cloudcode-pa.googleapis.com | 524: A timeout occurred</title>
[2025-12-06 15:23:30] <title>cloudcode-pa.googleapis.com | 524: A timeout occurred</title>
```

**问题分析**：
- Cloudflare 524 错误 = 源站（Google API）响应超时
- 主要集中在凌晨和上午时段
- 可能原因：
  1. Google API 响应慢
  2. Workers 代理超时配置不足
  3. 网络波动

**影响**：
- 长时间请求（如复杂对话）可能失败
- 用户体验差

---

### 3. Antigravity 403 权限错误

**现象**：
```log
[2025-12-05 17:53:48] [Antigravity] 主端点异常: 403 Forbidden - 该账号没有使用权限
[2025-12-05 17:53:49] [Antigravity] 备用端点异常: 403 Forbidden - 该账号没有使用权限
[2025-12-05 17:53:49] [ERROR] [Attempt 1/5] Antigravity streaming error: 403 Forbidden
```

**受影响账号**：
| 邮箱 | 状态 |
|------|------|
| 13903938819l@gmail.com | 403 无权限 |
| wzq1196256615@gmail.com | 403 无权限 |
| xizhuxibushinizhuxi@gmail.com | 403 无权限 |
| fup471231@gmail.com | 403 无权限 |

**问题分析**：
- 这些账号可能：
  1. 未开通 Antigravity 服务
  2. 账号被 Google 限制
  3. projectId 验证失败（免费账号）
- 系统应该自动禁用这些账号，但似乎仍在尝试使用

---

## 🟡 中等问题

### 4. 大量账号已禁用

**现象**：
```log
[2025-12-05 15:31:28] Account 101259842766713616076 state - disabled: True, email: qwe500189@gmail.com
[2025-12-05 15:31:28] Account 114539771884472638778 state - disabled: True, email: xuemantian123@gmail.com
[2025-12-05 15:31:28] Account 113272833957829134342 state - disabled: True, email: zzh3364139062@gmail.com
...
```

**统计**：
- 启动时加载了大量账号
- 其中相当比例已被禁用
- 禁用原因可能是之前的 403/429 错误

**建议**：
- 定期清理无效账号
- 或设置自动重新验证机制

---

## 🟢 轻微问题

### 5. socket.send() 异常

**现象**：
```log
[2025-12-05 18:44:55] socket.send() raised exception.
[2025-12-05 18:44:55] socket.send() raised exception.
[2025-12-05 18:44:55] socket.send() raised exception.
...
```

**问题分析**：
- 客户端在流式响应过程中断开连接
- 服务器仍在尝试发送数据
- 这是正常现象（用户取消请求）

**影响**：
- 仅产生日志噪音
- 不影响功能

---

## ✅ 正常工作的功能

### Token 刷新
```log
[2025-12-05 15:31:40] Token expired for account: hl7904825@gmail.com
[2025-12-05 15:31:40] Current token expired, refreshing...
[2025-12-05 15:31:40] Refreshing access token for account: hl7904825@gmail.com
[2025-12-05 15:31:41] Successfully refreshed token for account: hl7904825@gmail.com
```
✅ Token 自动刷新功能正常工作

### 凭证轮换
```log
[2025-12-05 17:23:10] Rotated to Antigravity account 3/114: jiyuning1314@gmail.com
```
✅ 凭证轮换功能正常工作

### 账号发现
```log
[2025-12-05 15:31:34] Discovered 113 enabled Antigravity accounts
[2025-12-05 15:31:34] Antigravity credential manager initialized with 113 accounts
```
✅ 账号发现和初始化正常

---

## 📋 优化建议

### 高优先级

#### 1. 增强 429 系列封禁逻辑

**问题**：429 错误后系统仍然尝试已知配额耗尽的账号

**建议修改**：
```python
# 在 mark_credential_error 中确保 429 封禁被正确处理
async def mark_credential_error(self, virtual_filename: str, error_code: int, error_message: str = ""):
    if error_code == 429:
        # 确保解析 429 错误详情并设置系列封禁
        await self._handle_429_series_ban(virtual_filename, error_message)
        
        # 同时在内存中标记，避免立即重试
        self._temporarily_banned_accounts.add(virtual_filename)
```

#### 2. 添加 524 超时重试逻辑

**问题**：Workers 代理超时导致请求失败

**建议**：
- 在 Workers 中增加超时时间
- 或在 Python 端添加 524 错误重试逻辑

#### 3. 确保 Antigravity 403 自动禁用

**问题**：403 账号仍在被尝试使用

**建议检查**：
```python
# 确认 403 错误码在自动封禁列表中
auto_ban_error_codes = [401, 403, 429, 400]
```

### 中优先级

#### 4. 减少 socket 异常日志

**建议**：
```python
try:
    await response.send(chunk)
except Exception:
    # 客户端断开连接，静默处理
    break
```

#### 5. 添加账号健康检查

**建议**：
- 定期检查禁用账号是否可以重新启用
- 自动清理长期无效的账号

---

## 📈 监控指标建议

| 指标 | 当前状态 | 建议阈值 |
|------|----------|----------|
| 429 错误率 | 高 | < 10% |
| 524 超时率 | 中 | < 5% |
| 403 错误率 | 中 | < 5% |
| 账号禁用率 | 高 | < 20% |
| Token 刷新成功率 | 100% | > 95% |

---

## 🔧 下一步行动

1. [x] ~~检查 429 系列封禁逻辑是否正确触发~~ ✅ 已修复
2. [ ] 检查 Workers 代理超时配置
3. [ ] 确认 Antigravity 403 自动禁用逻辑
4. [ ] 清理无效的禁用账号
5. [ ] 添加更详细的错误监控

---

## ✅ 已修复的问题

### 429 系列封禁不生效（2025-12-07 修复）

**问题根因**：
1. `_set_series_ban` 只更新了文件，没有更新内存中的账号数据
2. `_check_series_ban` 检查的是内存中的数据，所以封禁不生效
3. `_identify_model_series` 只识别 `gemini-3` 和 `claude`，不识别 `gemini-2.5-pro`
4. 全局配额耗尽（`Chat API requests`）时没有模型名称，导致跳过封禁

**修复内容**：

1. **`_set_series_ban` 同时更新内存数据**：
```python
# [CRITICAL FIX] 同时更新内存中的账号数据，确保封禁立即生效
for cred_entry in self._credential_accounts:
    if cred_entry.get("virtual_filename") == virtual_filename:
        cred_entry["account"][field_name] = ban_until_str
        break
```

2. **扩展 `_identify_model_series` 识别范围**：
```python
# 新增识别：
# - gemini-2.5 系列 -> gemini_2_5_series
# - gemini-2 系列 -> gemini_2_series
# - 通用 gemini -> gemini_series（兜底）
```

3. **全局配额耗尽时封禁所有系列**：
```python
# 无法识别模型系列时，封禁所有常见系列
all_series = ['gemini_series', 'gemini_2_series', 'gemini_2_5_series', 'gemini_3_series', 'claude_series']
for s in all_series:
    await self._set_series_ban(virtual_filename, s, ban_until)
```

**预期效果**：
- 429 错误后，账号的对应模型系列会被立即封禁
- 后续请求会自动跳过被封禁的账号
- 减少无效的 API 调用

---

## 附录：关键日志时间线

| 时间 | 事件 |
|------|------|
| 2025-12-05 15:30:57 | 服务启动，加载 113 个 Antigravity 账号 |
| 2025-12-05 15:30:58 | 首次 429 错误，连续 5 次重试失败 |
| 2025-12-05 17:53:48 | Antigravity 403 权限错误 |
| 2025-12-06 01:19:49 | 首次 524 超时错误 |
| 2025-12-06 03:21:06 | 524 超时错误集中出现 |
| 2025-12-06 15:23:30 | 最后一条 524 超时错误 |
