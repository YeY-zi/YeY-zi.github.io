# Canvas 与香港岗位雷达使用指南

核查日期：2026-10-25。建议先把 Canvas 助手用于整理课程，再小范围试用岗位雷达。你的公开 GitHub 主页用于介绍本人和展示作品；个人课程看板、求职数据库与凭据留在本地。使用这两个开源工具不要求先 fork，也不需要把个人数据上传 GitHub。

本指南依据当前仓库 README、配置模板、实现代码与公开 CI 整理；尚未使用你的 Canvas 或招聘网站账户运行，因此实际可读取的课程、岗位和推送通道需要首次运行时核对。这里的命令供你之后操作，未替你安装软件或设置账号。

## 先分清两种“本地”

| 使用方式 | 数据实际去向 | 适合放在哪里 |
|---|---|---|
| Canvas 本地 Python 路线 | 本机直接查询学校 Canvas，在本机生成周报、HTML、ICS；不需要作者服务器或 Cloudflare | 本机使用；公开网站仅放虚构样例 |
| Radar 的本地统计与看板 | `stats` 和单纯打开 `board` 读取本机数据库；首次可能是空看板 | 本机使用 |
| Radar 的真实单次扫描 | 从外部招聘来源获取岗位；把背景画像与岗位内容发送到配置的模型服务；向本人选择的通道发日报或高分提醒 | 本机运行，但存在明确外发 |

**Radar 当前 CLI 没有“全部通知关闭、无模型 Key、只抓一轮”的现成模式。** `run.py once` 同时要求可用模型 Key 和至少一个配置完整的通知通道；它会抓取、打分、推送后退出。不能把“本机运行”理解成“数据完全不出本机”。这一点由 [CLI](https://github.com/prui0474/hk-job-radar/blob/master/src/hk_job_radar/cli.py) 和 [配置门禁](https://github.com/prui0474/hk-job-radar/blob/master/src/hk_job_radar/config.py) 确认。

## Canvas 十分钟首次使用

时间以电脑已装好 Git 和 Python 3.8+、学校可生成个人访问令牌为前提。若学校生成入口不可用，先处理权限，不用反复运行脚本。源码只依赖 Python 标准库。

### 第 0–2 分钟：下载到自己的工具文件夹

打开 PowerShell，先检查环境：

```powershell
git --version
python --version
```

首次安装执行：

```powershell
$toolRoot = Join-Path $env:USERPROFILE 'Documents\GitHub-tools'
New-Item -ItemType Directory -Force -Path $toolRoot | Out-Null
git clone https://github.com/Famalhaut04/canvas-weekly-hub.git (Join-Path $toolRoot 'canvas-weekly-hub')
Set-Location (Join-Path $toolRoot 'canvas-weekly-hub')
git log -1 --oneline
Copy-Item -LiteralPath '.\config\canvas_config.example.json' -Destination '.\canvas_config.json'
```

已有这个工具文件夹时，进入已有目录即可，不要再复制配置覆盖自己的设置。核查时上游 `main` 最新提交为 `7f2e222`（2026-10-04）。

### 第 2–5 分钟：本人在学校页面生成令牌

在自己的浏览器登录 [CityU Canvas](https://canvas.cityu.edu.hk/)，按学校当前页面进入“账户 → 设置 → 新访问令牌”。用途可以写“个人课程整理”；到期日按学校要求设置。

令牌由你自己复制到本地配置，不粘贴进聊天、README、公开仓库、截图或公共网站。学校若禁用个人令牌，就以学校提供的方式为准；这个助手不能绕过权限。

### 第 5–7 分钟：只改本地配置

```powershell
notepad .\canvas_config.json
```

在现有 JSON 模板里核对这些字段，其他字段先保持默认：

| 字段 | 首次建议 |
|---|---|
| `canvas_url` | `https://canvas.cityu.edu.hk`，以你实际登录学校的地址为准 |
| `access_token` | 本人刚生成的令牌，仅在本地填入 |
| `lookback_days` / `upcoming_days` | 都先用 `7` |
| `download_files` | 先改成 `false`，先看文件清单，避免下载大量课件 |
| `github.push_enabled` | 保持 `false` |
| `token_expires_at` | 可填写自己设置的到期日，用作提醒 |

JSON 字符串用英文双引号；最后一个字段后面不要额外加逗号。保存后关闭记事本。

### 第 7–10 分钟：检查、抓取、打开看板

在同一个项目目录顺序运行：

```powershell
python canvas_weekly_report.py --check-config
python canvas_weekly_report.py
python serve_board.py
```

第一条出现 `CONFIG_OK` 只代表配置格式正确，不证明课程权限正常。第二条根据真实请求生成周报、ICS 和离线看板；出现 `PARTIAL_OK` 时先看哪些文件或公告未完整读取。课程或作业抓取失败时，不把旧内容当作刚更新的数据。第三条运行期间打开：

[本机 Canvas 看板](http://localhost:8137/)

本地服务窗口需要保持打开，结束时按 `Ctrl+C`。也可双击生成的“我的学习网站.html”查看离线快照；离线文件不会自己查询 Canvas。

第一次核对三件事：课程名称是否与学校页面一致；下一项作业的截止时间是否一致；新上传文件的名称和日期是否可见。先用真实课程验证这些事实，再依赖看板安排学习。

### 以后怎样刷新与提醒

当前 v2.2 网页采用**手动刷新课程**，默认每三天出现的更新提醒只是提示点击，并不是后台抓取。当前版本暂停云端订阅与微信推送。Python 路线可以以后再配本机计划任务，但需要电脑、Python 和网络在线；本指南没有创建计划任务。

ICS 是一次导出的日历快照。导入个人日历后，应先确认日历应用对更新、重复导入的处理方式，不能把它当作已建立实时订阅。

上游仍有 [403/404 与部分课程读取问题 #5](https://github.com/Famalhaut04/canvas-weekly-hub/issues/5) 和 [作业未同步报告 #4](https://github.com/Famalhaut04/canvas-weekly-hub/issues/4)。权限拒绝不一定是令牌过期；先在 Canvas 原页面核对，不能承诺工具覆盖所有内容。

配置与产物默认已列入 `.gitignore`。可检查忽略规则：

```powershell
git check-ignore canvas_config.json
git status --short
```

学校资料、作业状态、课件、周报和导出的个人 HTML 都留在本机。不要用公开 GitHub Pages 托管真实课程数据。

来源：[当前 README](https://github.com/Famalhaut04/canvas-weekly-hub/blob/main/README.md) · [本地部署指南](https://github.com/Famalhaut04/canvas-weekly-hub/blob/main/docs/部署指南.md) · [配置模板](https://github.com/Famalhaut04/canvas-weekly-hub/blob/main/config/canvas_config.example.json) · [系统概述](https://github.com/Famalhaut04/canvas-weekly-hub/blob/main/docs/系统概述.md)。

## Radar 分阶段试用

建议先确认自己想找的岗位，再逐步启用来源和通知。不要一开始同时登录三个招聘网站、连接简历和开启长期监控。首轮目标只是确认“这个工具能不能帮我找到几条值得看的岗位”。

### 阶段一：准备本地环境，暂不抓取

需要 Python 3.10+。下面手动路线与上游安装器使用的依赖步骤一致，但不会自动打开配置、招聘网站登录或常驻运行。

```powershell
$toolRoot = Join-Path $env:USERPROFILE 'Documents\GitHub-tools'
New-Item -ItemType Directory -Force -Path $toolRoot | Out-Null
git clone https://github.com/prui0474/hk-job-radar.git (Join-Path $toolRoot 'hk-job-radar')
Set-Location (Join-Path $toolRoot 'hk-job-radar')
git log -1 --oneline
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m playwright install chromium
Copy-Item -LiteralPath '.\config.example.yaml' -Destination '.\config.yaml'
Copy-Item -LiteralPath '.\data\profiles\profile.example.yaml' -Destination '.\data\profiles\profile.yaml'
```

若配置文件已存在，不重复执行覆盖配置的两条 `Copy-Item`。安装依赖与 Chromium 会访问软件分发服务，但不需要你的招聘账号、模型 Key 或简历。

确认程序能启动，先运行不需要凭据的本地命令：

```powershell
.\.venv\Scripts\python.exe run.py stats
.\.venv\Scripts\python.exe run.py board --port 8765
```

`board` 会打开本地网页，以程序打开的网址为准；首次数据库为空属于正常情况。保持窗口打开，结束时按 `Ctrl+C`。此阶段不要点“简历匹配”“投递副驾”或“同步已投”等动作。

也可使用上游 `install.bat` 完成安装，但它首次会继续索要模型 Key、打开配置向导，并尝试引导 JobsDB 登录。若只想先准备环境，使用上面的手动命令更容易确认每一步做了什么。

### 阶段二：只配置小范围来源和脱敏画像

```powershell
notepad .\config.yaml
notepad .\data\profiles\profile.yaml
```

在 `config.yaml` 中先明确写出全部来源开关：

```yaml
sources:
  jobsdb: false
  linkedin: false
  indeed: false
  workday: true

keywords:
  - "data analyst"
  - "business analyst"
  - "product analyst"
  - "product manager"

graduate_keywords: []
resume_path: ""
```

这些岗位词是试用示例，按你的真实目标删改。关键词初筛是子串匹配，不要直接填 `ai`、`ml`、`python` 或 `sql`：短词和技能词容易命中大量无关描述。可先保留两三个岗位词，不追求第一次覆盖全部市场。

`career_stage` 控制岗位类型：当前实现中 `student` 主要筛实习/兼职，`graduate` 主要筛全职，`both` 覆盖两者。根据你这次搜索的目标填写。模板里的学生签证/IANG文字不能自动代表你的实际工作资格；不要把示例中的日期、签证和薪资口径直接复制成个人事实。

第一轮可以把 `daily_limits` 中的 `workday` 降到较小值，例如 `10`，并保留其他原有配置字段。小配额意味着覆盖不完整，目的是观察结果与成本。Workday 使用雇主招聘系统接口，不需要先登录 LinkedIn、JobsDB 或 Indeed；它能获取哪些岗位仍取决于当前雇主名单、接口与关键词。

在 `profile.yaml` 的 `background` 中只写打分所需的职业信息，例如实际在读专业、已验证的技能、做过的项目、目标岗位和排除项。可以省略姓名、电话、邮箱、住址和证件信息。不要填写尚未掌握的技能。`basic` 中的姓名电话等投递字段可先留空，`data/resumes/` 不放简历，也不运行 `match` 或 `apply`。

**外发内容说明：** [LLM 打分源码](https://github.com/prui0474/hk-job-radar/blob/master/src/hk_job_radar/scoring/llm_scorer.py)把 `background` 与岗位信息发送到 `llm.base_url` 指定的服务，默认是 DeepSeek。[简历匹配源码](https://github.com/prui0474/hk-job-radar/blob/master/src/hk_job_radar/matcher.py)还会发送简历原文；本阶段通过 `resume_path: ""` 和不使用匹配/副驾功能避免这个路径。评分是模型相关度建议，不是拿到面试的概率。

### 阶段三：准备本人凭据，接受外发后再扫一轮

当前 `once` 命令必须配齐以下两项才能运行，缺一项会报配置错误：

1. 本人模型 API Key。当前默认服务为 DeepSeek；从其官方平台自行创建，只在本机填写，确认 API 计费和额度后使用。可在项目根目录新建 `api_key.txt`，里面只写 Key 本身；此文件被 `.gitignore` 排除。本指南不需要真实 Key 值。
2. 一个本人接收的通知通道。初次只选一个，例如邮件，按邮箱服务商的 SMTP 设置与授权方式填写 `notify` 段；`auth_code` 使用邮箱授权码，不直接把登录密码复制进去。其他通道也可用，按上游[推送渠道对比](https://github.com/prui0474/hk-job-radar/blob/master/docs/推送渠道对比.md)设置。

如使用网页向导：

```powershell
.\.venv\Scripts\python.exe run.py setup
```

保存后重新打开 `config.yaml`，人工检查 `sources`、关键词、通知通道和 `resume_path`。核查时最新提交 `fbaf7d6`（2026-10-23）的 [Windows/macOS CI](https://github.com/prui0474/hk-job-radar/actions/runs/39917310340)均失败；Windows 日志显示来源开关保存测试未通过，结果意外带入 `workday: true`。这不代表全程序无法运行，但不应只相信向导保存成功提示。

准备好后顺序执行：

```powershell
.\.venv\Scripts\python.exe run.py doctor --no-llm
.\.venv\Scripts\python.exe run.py notify test
.\.venv\Scripts\python.exe run.py once
.\.venv\Scripts\python.exe run.py stats
.\.venv\Scripts\python.exe run.py board --port 8765
```

`doctor --no-llm` 只跳过模型连通测试，仍可能执行网站和网络诊断；它不等于完全离线。`notify test` 会向所选通道发送一条测试消息。`once` 会真实获取岗位、调用模型并推送，完成后退出，不是常驻程序。

首轮检查：抓到了多少条；关键词留下多少条；模型成功打分多少条；看板内的三五个岗位链接是否仍有效；是否真的符合你的目标。零岗位也可能来自雇主名单、类型过滤或配额，不直接等于安装失败。程序去重并不能保证所有平台岗位都被发现。

### 阶段四：确定有用后再加来源或长期运行

先按官方说明分别处理某个网站的本人登录或人机验证，再只开启该站测试。当前仍有 [JobsDB 登录问题 #2](https://github.com/prui0474/hk-job-radar/issues/2)，外站结构和访问策略也可能变化。不要照搬“肯定不漏”或“永远稳定”的宣传描述。

以后若确实需要常驻，再使用：

```powershell
.\.venv\Scripts\python.exe run.py run
```

也可双击上游 `start.bat`。程序与电脑要保持运行，日报默认香港时间 `09:00`；关机时不会凭空在云端运行。只想当天看一轮，继续用 `once` 即可。本指南没有创建开机自启或计划任务。

检查个人配置是否被忽略：

```powershell
git check-ignore config.yaml data/profiles/profile.yaml api_key.txt
git status --short
```

`.gitignore` 不能撤回过去已提交过的秘密；如果曾误提交，先撤销相关凭据，再处理 Git 历史。不要把 `data/`、浏览器登录态、数据库、真实画像、API Key 或通知凭据放上公开 GitHub。

来源：[Radar README](https://github.com/prui0474/hk-job-radar/blob/master/README.md) · [新手教程](https://github.com/prui0474/hk-job-radar/blob/master/docs/getting-started-zh.md) · [配置模板](https://github.com/prui0474/hk-job-radar/blob/master/config.example.yaml) · [画像模板](https://github.com/prui0474/hk-job-radar/blob/master/data/profiles/profile.example.yaml) · [CLI 命令](https://github.com/prui0474/hk-job-radar/blob/master/src/hk_job_radar/cli.py) · [本地看板实现](https://github.com/prui0474/hk-job-radar/blob/master/src/hk_job_radar/workbench.py)。

## 是否放进自己的 GitHub

| 你的动作 | 什么时候用 | 个人主页怎样表述 |
|---|---|---|
| Star 或收藏 | 先记住工具，关注上游 | “我在使用的工具”或资源链接 |
| Clone | 下载到本机使用或学习；不产生自己的在线仓库 | 不必当作作品展示 |
| Fork | 准备保留改动、同步上游、贡献修复 | 写清原作者、上游链接及你实际新增的部分 |
| 自己开发并验证新功能 | 已有具体需求、实现、测试或使用结果 | 可以写“基于某开源项目的改造”，用证据说明个人贡献 |

两者均采用 MIT 许可，可以 fork 和改造，保留来源及许可证：[Canvas MIT](https://github.com/Famalhaut04/canvas-weekly-hub/blob/main/LICENSE) · [Radar MIT](https://github.com/prui0474/hk-job-radar/blob/master/LICENSE)。

**不建议把未经修改的工具列为“我独立开发的项目”，也不建议为了充实主页批量复制仓库。** 对你的求职更有用的是形成一份真实记录：自己遇到了什么问题、为何选择工具、实际验证了什么、发现了哪些限制、做过什么改进。比如为课程看板补一个经过验证的状态检查，或为岗位雷达补一个可复现的来源开关修复；有真实修改后再展示。

公开个人主页可以介绍“我使用开源工具整理学习和求职流程”，链接上游，展示虚构或彻底脱敏的样例。真实课程内容和求职记录仍在本机，公开作品展示与个人使用数据各自有清楚边界。
