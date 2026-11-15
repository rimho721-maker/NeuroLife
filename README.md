[README.md](https://github.com/user-attachments/files/34880972/README.md)
# NeuroLife

NeuroLife 是一个基于 [Britneycode/dsh-fish-soul](https://github.com/Britneycode/dsh-fish-soul) 扩展的 DeepSeek Harness（DSH）插件。它保留了上游的人格、记忆和日常机制，并增加了自我模型、后台记忆审阅和 RamKo 提案审核等功能。

> **测试版。所有功能均处于测试阶段。** 当前提供 Windows 新环境测试包。建议在独立 DSH profile 中创建新人格，用合成数据验收。换机安装和不同 DSH 版本的兼容性仍待验证。

## NeuroLife 在 Fish Soul 上实现了什么

**它把聊天中积累的记忆、后台产生的思考和 RamKo 审核接了起来。** 聊天时，角色能带着相关记忆继续对话；后台运行时，插件能整理近期思考，并把值得确认的事项列到待审区。

| 功能 | 你实际能看到什么 |
| --- | --- |
| 自我模型 | 在 RamKo 查看近期思考里识别出的计划、短时情绪和偏好，查看支持证据数、不确定性和判断变化。证据不足的记录保留为无法判断。 |
| 定期记忆审阅 | Owner 开启后，后台约每 10 分钟扫描近 30 天符合条件的长期记忆，寻找金额记录不一致、同一对象的偏好方向变化、反复提及但缺少完成记录的约定。符合条件的事项生成带脱敏摘要的待审提案，同一事项重复扫描会去重。 |
| RamKo 提案审核 | Owner 可以提交草稿，也可以对后台生成的提案选择采纳或拒绝。提案和审核结果保存在本地，重启后可继续查看。 |

例如，两条长期记忆把同一笔欠款分别记为 **160 文**和 **175 文**。符合规则时，RamKo 会显示一条金额不一致提案，供 Owner 确认。审核会保存决定；实际修正记忆需要另行操作。

页面手动提交和审核已在原环境实测。定期审阅的生成与去重已通过合成测试，新机运行仍待验收。

## 其他能力

| 模块 | 功能 |
| --- | --- |
| 人格与日常 | 人格模板、角色卡、心境、日记、主动开口、目标与承诺 |
| 长期记忆 | 记忆记录与纠正、关键词召回、Embedding 向量召回 |
| 外部集成 | 可选的 QQ、绘图、语音等功能，需分别配置对应服务 |

审核只记录 Owner 的决定。修改记忆、创建 Goal 或执行外部动作需要另行操作。当前 DSH 桌面工具无法取得提交所需的可信 Owner 身份，手动提交请使用 RamKo 页面表单。

## 运行原理

1. **会话与回复。** DSH 接收消息并运行 Agent。Fish Soul 根据当前会话和说话人，从本地召回相关记忆，再把人格、心境和相关记忆组合进模型上下文。模型生成回复。符合条件的会话可以在之后提取事实、事件和感受，写入长期记忆。
2. **记忆召回。** 文本检索使用 BM25。启用 Embedding 时，也可做语义向量检索，再用 RRF 融合两路排序。Fish Soul 根据人物、会话和可见性筛选可进入上下文的记忆。
3. **后台分析与人工审核。** 自我模型分析已有 Thought，定期记忆审阅则按独立开关扫描符合范围的长期记忆。符合条件的事项生成待审提案，由 RamKo 展示。Owner 审核只记录决定；原始记忆和审核状态保存在各自的本地 SQLite 数据库中。

DSH 通过 profile 加载 `dsh-fish-soul`。RamKo 是 Fish Soul 在 DSH 内的管理界面。Embedding、QQ 的 OneBot/SnowLuma、绘图和语音通过各自的配置接入。

## 安装

### 运行依赖

| 依赖 | 用途 | 获取方式 |
| --- | --- | --- |
| Windows x64、DSH Desktop 0.2.1-rc.5 | 当前测试目标环境 | 新机安装 DSH Desktop |
| Node 24.21.0 | 运行插件和内置 SQLite | 可使用 DSH 自带的 Node 运行时 |
| pnpm 11.28.5 | 安装 profile 依赖 | 新机准备，版本与原测试环境对齐 |
| `@deepseek-ai/dsh-tools`、`@deepseek-ai/schemastery` | 插件声明的宿主依赖 | 通常由 DSH 环境提供 |
| 聊天模型及其凭证 | 验证聊天收发 | 在新机 DSH 中单独配置，测试包不含凭证 |
| Ollama + `bge-m3` | 新机验收必需，用于记忆向量化和语义召回 | 按下文安装并下载模型 |
| QQ 接入、图像生成、STT/TTS | 可选功能 | 按需配置，见测试包中的 `docs/EXTERNAL_DEPENDENCIES.md` |

### 安装步骤

1. 下载并解压 `NeuroLife_Windows_NewMachine_20261116.zip`，打开解压目录里的 `docs/INSTALL.md`。
2. 安装 DSH Desktop。设置独立的 `DSH_HOME`，例如 `C:\NeuroLife\dsh-home`，用它启动测试实例。后续启动时沿用这个目录。
3. 将解压得到的 `plugin/` 放到固定位置。按安装说明用 `profile/package.json.template` 配置独立 profile，将 `{{PLUGIN_PATH}}` 替换为插件的实际路径。确认 `dsh.profile.bundles` 中有 `dsh-fish-soul`，再按说明链接插件并运行 `pnpm install`。包内的参考锁文件包含占位符，用新机生成的锁文件安装。
4. 启动 DSH，确认「插件」列表有 `dsh-fish-soul`，再打开 **RamKo → 人格 / 角色**，创建新人格。
5. 按下文配置 Embedding，再按测试包内的 `docs/ACCEPTANCE.md` 验收聊天收发、合成记忆写入、文本和向量召回，以及提案提交和审核。

## 配置与数据

将测试包内的 `config/fish-soul-config.template.json` 复制为 `<DSH_HOME>/fish-soul-config.json`，按需填写新环境的配置。

所有数据库都位于 `<DSH_HOME>/fish-soul/`。插件使用 Node 内置的 SQLite，在相关模块首次打开数据库时自动建库并执行迁移。**无需安装数据库服务器、手动导入 SQL 或从旧机复制数据库。**

| 文件 | 用途 |
| --- | --- |
| `soul.db` | 长期记忆、关系、日程、向量索引以及部分运行状态 |
| `chat-archive.db` | 本地聊天文本归档 |
| `neurolife-self-model.sqlite` | 自我模型判断、记忆提案和审核记录 |

测试包附建库与迁移代码、脱敏配置模板及可选的合成数据。新机从空库开始，测试时自行创建新人格。原机人格、真实聊天、记忆、索引和凭证留在原机。定期记忆审阅直接读取本地 SQLite。Embedding 用于记忆向量化和语义召回。

### 安装并配置 Embedding（必需）

新机完整验收需要启用 Embedding。下面用本地 Ollama 和支持中文的 `bge-m3`。请先按前面的安装步骤准备独立的 `DSH_HOME`，并用测试包模板生成该目录下的 `fish-soul-config.json`。

**1. 安装 Ollama**

```powershell
winget install -e --id Ollama.Ollama --accept-source-agreements --accept-package-agreements
```

**2. 确认服务在运行，下载模型**

安装后重新打开 PowerShell。如果 Ollama 已在后台运行，直接下载模型。如果连接失败，打开 Ollama，或在另一个 PowerShell 窗口运行 `ollama serve`，保持该窗口打开。

```powershell
ollama pull bge-m3
```

**3. 验证 OpenAI 兼容接口**

```powershell
$body = @{
  model = 'bge-m3'
  input = @('Embedding 安装测试')
} | ConvertTo-Json -Depth 5

$result = Invoke-RestMethod -Uri 'http://127.0.0.1:11434/v1/embeddings' `
  -Method Post -ContentType 'application/json; charset=utf-8' `
  -Body ([System.Text.Encoding]::UTF8.GetBytes($body))

$result.data[0].embedding.Count
```

返回大于 0 的向量维度后再继续。参考 [Ollama Windows 安装说明](https://docs.ollama.com/windows)、[bge-m3 模型页](https://ollama.com/library/bge-m3)和 [OpenAI 兼容接口](https://docs.ollama.com/api/openai-compatibility)。

**4. 写入 Fish Soul 配置**

先正常退出 DSH，使用启动测试实例时的同一个 `DSH_HOME`。下面用示例路径，请改成你自己的测试目录。

```powershell
$env:DSH_HOME = 'C:\NeuroLife\dsh-home'
$cfgPath = Join-Path $env:DSH_HOME 'fish-soul-config.json'

if (-not (Test-Path -LiteralPath $cfgPath -PathType Leaf)) {
  throw "找不到配置文件，请先按安装步骤复制配置模板：$cfgPath"
}

$cfg = Get-Content -LiteralPath $cfgPath -Raw -Encoding UTF8 | ConvertFrom-Json
if ($null -eq $cfg.embedding) {
  $cfg | Add-Member -NotePropertyName embedding -NotePropertyValue ([pscustomobject]@{}) -Force
}

$values = @{
  enabled = $true
  baseURL = 'http://127.0.0.1:11434/v1'
  apiKey = 'local'
  model = 'bge-m3'
  timeoutMs = 800
}
foreach ($key in $values.Keys) {
  $cfg.embedding | Add-Member -NotePropertyName $key -NotePropertyValue $values[$key] -Force
}

$backupPath = "$cfgPath.$(Get-Date -Format 'yyyyMMdd-HHmmss').bak"
Copy-Item -LiteralPath $cfgPath -Destination $backupPath -ErrorAction Stop
[System.IO.File]::WriteAllText(
  $cfgPath,
  ($cfg | ConvertTo-Json -Depth 30),
  [System.Text.UTF8Encoding]::new($false)
)
```

**5. 重开 DSH 并验证向量召回**

保持 Ollama 运行，从设置了同一 `DSH_HOME` 的 PowerShell 重新启动 DSH。用新机的合成记忆测试写入和语义召回，并在 RamKo 核对向量状态。接口返回向量只证明 Ollama 可用，插件召回还要实测。如果本地模型首次加载超过 800 毫秒，可适当提高 `embedding.timeoutMs` 后重试。

## 使用

- **手动审阅**：进入 **RamKo → 认知 → 自主活动 → 自我模型 → 提交记忆审阅草稿**。填写测试提案，在「待审阅提案与历史」中确认显示，再采纳或拒绝。
- **定期记忆审阅**：在自我模型页开启独立开关，等待约一个调度周期（约 10 分钟），再刷新待审列表。没有符合条件的记忆时可以零提案。此功能读取本地 SQLite。
- **向量召回**：完成上面的 Ollama 安装和插件配置后，再在 RamKo 验证语义检索。其他外部服务按需配置。

## 目录

| 路径 | 内容 |
| --- | --- |
| `plugin/` | Fish Soul 插件源码 |
| `profile/` | DSH profile 模板、工作区配置和参考锁文件 |
| `config/` | 脱敏配置模板 |
| `synthetic/` | 合成记忆和 Thought 测试数据 |
| `docs/` | 安装步骤、外部依赖说明和验收清单 |

## 测试与反馈

提交问题时请附 Windows、DSH 和 Node 版本、复现步骤、预期结果以及脱敏后的日志。测试包使用空库和合成数据。请将真实人格、聊天记录、数据库、Cookie、Token 和 API Key 保存在本机。

## 上游与许可

上游项目为 [Britneycode/dsh-fish-soul](https://github.com/Britneycode/dsh-fish-soul)，上游插件声明采用 MIT 许可。分发源码时请随附上游原始许可和版权声明，并保留第三方组件各自的许可文件。
