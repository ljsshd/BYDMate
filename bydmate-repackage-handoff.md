# Handoff — BYDMate 精简重打包

- 日期: 2026-10-13
- 语言: 中文
- 会话主题: 把 BYDMate（BYD DiLink 车机 Android 应用）重打包成个人精简版，删掉国内用不上、以及与用户已装"迪友桌面"重叠的功能。

## 工程位置与环境

- 工作区: `e:\code\BYDMate-main`（外层目录）
- 实际工程根: `e:\code\BYDMate-main\BYDMate-main`
- 包名 `com.bydmate.app`；Kotlin 2.1 + Jetpack Compose；源码 `app/src/main/kotlin/com/bydmate/app`
- 许可证: PolyForm Noncommercial 1.0.0（个人自用可改，**不可商用**）
- 重要: **该目录不是 git 仓库**（无 `.git`），删文件不可回滚；动手前必须先 `git init` 建基线
- 本机环境: 只有 Java 1.8；无 `ANDROID_HOME`/`ANDROID_SDK_ROOT`、无 `local.properties`；项目要求 **JDK 17 + Android SDK 34**，首次构建需联网 → **当前无法打包验证**

## 当前状态

- 仅完成**分析与方案**，**未改动任何代码**。
- 用户明确选择交付方式 = **plan-only（先出方案，暂不动代码）**。
- 一份完整的"逐文件改动清单"已在**对话中**产出（分 P0–P5 阶段：语言 → 删重叠功能 → 删语音/Yandex/联网 → 删杂项 → 收尾），**尚未落成文件**。用户曾被问"是否要落成 `REPACKAGE_PLAN.md`"，未答复。

## 已确认范围（权威来源见记忆文件）

记忆文件（直接读）：
- `C:\Users\Administrator\.comate\projects\code-BYDMate-main\memory\MEMORY.md`（索引）
- `C:\Users\Administrator\.comate\projects\code-BYDMate-main\memory\project_bydmate_fork.md`（范围与决策）
- `C:\Users\Administrator\.comate\projects\code-BYDMate-main\memory\user_profile.md`（用户画像）

摘要：
- **删除**: 俄语语音子系统（ASR GigaAM、离线俄语语音包、LLM 智能体 `agent/`、NLU、语音日志、麦克风/方向盘语音键）——**仅保留在线 TTS**；Yandex 系（`navdata/`、`media/` 导航镜像、`hud/`）；联网遥测（ABRP、Webhook、Telegram、云端 LLM、天气/搜索/充电站、SmartHome）；多语言（**只留中文**）；与迪友桌面重叠（`ui/widget/` 悬浮窗、`cluster/` 仪表投射、`split/` 分屏、`hud/`、`camera/` 盲区摄像头）；杂项（GitHub 自动更新检查、捐赠弹窗/收款码、BYD 云/VPN 设置、俄语文档）。
- **保留**: **在线 TTS**（MiniMax/Gemini/OpenRouter，用于播报）；**自动化控车**（`data/automation/`+`ui/automation/`，迪友桌面没有条件触发，是差异化价值；但需裁掉依赖已删功能的动作/触发器）；本地 AI 洞察；行程/充电/能耗/电池健康/技术面板/电价费用/本地备份恢复；TrackingService/autoservice/ADB；日志录制；fid 目录导出；纯中文界面。

## 待用户拍板（阻塞下一步）

1. 方向盘按键无障碍服务 `cluster/SteeringWheelKeyService`：**保留**（移出 `cluster/` 单独留，自动化保留"方向盘按键"触发 + 旋钮播放/暂停）还是**随 cluster 一起删**？
2. 本地 AI 洞察确认保留？（建议保留，本地计算不联网）
3. 英文资源 `values-en` 也删、只留中文？（默认按"删"处理）

## 关键技术坑（删之前必须处理）

1. `data/remote/` 是**混装目录**：遥测（IternioTelemetryClient/WebhookTelemetryClient）与本地 AI 洞察（`LocalInsightEngine`/`InsightsManager`/`InsightStats*`/`OpenRouterClient`/`HttpPrewarm`）同处一地 → **只删遥测客户端**；先核实 `OpenRouterClient` 是否被洞察复用。
2. `media/MediaSessionListenerService.kt` 是**双职责**（导航镜像 + 音乐会话）→ 保留服务本体，只删导航解析部分；Manifest 条目保留。
3. `voice/AudioCapture.kt` 可能被 TTS 播放/音频闪避共用 → 删 ASR 前核实（TrackingService 有 `audioCapture.restoreStuckDuck`）。
4. **默认语言是俄语**（`res/values/` 即俄语，不存在 `values-ru`）→ 只留中文需把 `values-zh/strings.xml` 提升为默认 `values/strings.xml`，并合并 zh 中缺失的键。
5. `ui/settings/SettingsScreen.kt` 约 4594 行，是改动重灾区；`SettingsSection` 需去掉 `WIDGET`/`DISPLAY`/`SPLIT`。
6. 自动化裁剪点：`ActionDispatcher.kt`、`AutomationEngine.kt`、`RuleShare.kt`、`RuleText.kt`、`RuleImportSummary.kt`、`ui/automation/AutomationViewModel.kt`、`ui/automation/AutomationScreen.kt` —— 去 `navigate`/`yandex_music`/`youtube`/`cluster_projection`/`split_screen*` 动作与 `button_press`/`steering_key` 触发器。
7. `app/build.gradle.kts` 可去依赖：`sherpa-onnx` AAR、`commons-compress`（仅离线语音包解压用）。

## 下一步建议

1. 等用户确认"待拍板 3 点"。
2. `git init` + 全量提交做基线（可逆、安全）。
3. 可选：把对话里的逐文件方案落成 `REPACKAGE_PLAN.md`（用户已问过是否要落文件，未答）。
4. 按 P0→P5 执行；每阶段 `./gradlew :app:assembleDebug` 编译通过后再 commit，出错即回滚。
5. 若要能编译验证：安装 JDK 17 + Android SDK 34（网络受限，下载可能需用户批准）。

## Suggested skills

- 本任务**没有专门匹配的 skill**；技能列表中的多数（`yc-hssp-dev`、`prod-fault-locate`、`schema-query-skill`、`kb-capture`、`review-report` 等）属其他项目，**不适用**。
- 建议下一个 agent：
  - `code-review` — 真正动手改完后对 diff 过一遍（重点：自动化裁剪、Manifest/DI/导航图改动）。
  - `handoff` — 若需再次交接。
- 注意：用户**未开启 ultracode / workflow**，不要擅自启动多代理工作流。

## 敏感信息

- 对话中**未出现**任何 API key / 密码 / PII。BYDMate 的密钥全部由用户在应用内填写，代码库不含密钥。
