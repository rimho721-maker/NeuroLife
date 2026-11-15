# NeuroLife

NeuroLife 是一个基于 [Britneycode/dsh-fish-soul](https://github.com/Britneycode/dsh-fish-soul) 扩展的 DeepSeek Harness（DSH）插件。它保留了上游的人格、记忆和日常机制，并增加了自我模型、后台记忆审阅和 RamKo 提案审核等功能。

> **测试版。所有功能均处于测试阶段。** 当前提供 Windows 新环境测试包。建议在独立 DSH profile 中创建新人格，用合成数据验收。换机安装和不同 DSH 版本的兼容性仍待验证。

## 功能

| 模块 | 功能 |
| --- | --- |
| 人格与日常 | 人格模板、角色卡、心境、日记、主动开口、目标与承诺 |
| 长期记忆 | 记忆记录与纠正、关键词召回、可选的 Embedding 向量召回 |
| 思考与自我模型 | 分析本地 Thought，展示计划、情绪、偏好及判断变化 |
| 记忆审阅 | 在 RamKo 手动提交和审核提案。开启定期记忆审阅后，后台可根据符合条件的记忆生成待审提案 |
| 外部集成 | 可选的 QQ、绘图、语音等功能，需分别配置对应服务 |

审核只记录 Owner 的决定。修改记忆、创建 Goal 或执行外部动作需要另行操作。当前 DSH 桌面工具无法取得提交所需的可信 Owner 身份，手动提交请使用 RamKo 页面表单。

## 安装

测试包的目标环境是 **Windows x64、DSH Desktop 0.2.1-rc.5、Node 24.21.0**。注册依赖时使用 pnpm，原环境版本为 11.28.5。测试包包含 SQLite 初始化代码、脱敏配置模板和合成测试数据。DSH 安装程序、Embedding 服务和模型权重需另备。

1. 下载并解压 `NeuroLife_Windows_NewMachine_20261116.zip`，打开解压目录里的 `docs/INSTALL.md`。
2. 安装 DSH Desktop。设置独立的 `DSH_HOME`，例如 `C:\NeuroLife\dsh-home`，用它启动测试实例。后续启动时沿用这个目录。
3. 将解压得到的 `plugin/` 放到固定位置。按安装说明用 `profile/package.json.template` 配置独立 profile，将 `{{PLUGIN_PATH}}` 替换为插件的实际路径。确认 `dsh.profile.bundles` 中有 `dsh-fish-soul`，再按说明链接插件并运行 `pnpm install`。包内的参考锁文件包含占位符，用新机生成的锁文件安装。
4. 启动 DSH，确认「插件」列表有 `dsh-fish-soul`，再打开 **RamKo → 人格 / 角色**，创建新人格。记忆库、聊天归档库和自我模型库会在各模块首次使用时初始化。
5. 按测试包内的 `docs/ACCEPTANCE.md` 验收。先测试聊天收发、合成记忆写入与召回，再测试提案提交与审核。

## 使用

- **手动审阅**：进入 **RamKo → 认知 → 自主活动 → 自我模型 → 提交记忆审阅草稿**。填写测试提案，在「待审阅提案与历史」中确认显示，再采纳或拒绝。
- **定期记忆审阅**：在自我模型页开启独立开关，等待约一个调度周期（约 10 分钟），再刷新待审列表。没有符合条件的记忆时可以零提案。此功能读取本地 SQLite，可在未配置 Embedding 时测试。
- **向量召回与外部服务**：按测试包内的 `docs/EXTERNAL_DEPENDENCIES.md` 配置。新机首次验收可先使用本地文本记忆。

## 测试与反馈

提交问题时请附 Windows、DSH 和 Node 版本、复现步骤、预期结果以及脱敏后的日志。测试包使用空库和合成数据。请将真实人格、聊天记录、数据库、Cookie、Token 和 API Key 保存在本机。

## 上游与许可

上游项目为 [Britneycode/dsh-fish-soul](https://github.com/Britneycode/dsh-fish-soul)，上游插件声明采用 MIT 许可。分发源码时请随附上游原始许可和版权声明，并保留第三方组件各自的许可文件。
