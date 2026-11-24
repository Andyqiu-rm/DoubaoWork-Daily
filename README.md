<div align="center">

<sub><strong>L0NE-6 · DAILY SIGNAL</strong></sub>

# 🫘 Doubao Work Daily

**豆包工作每日签到单文件脚本：发现活动、定位唯一 daily_checkin、领取奖励，并通过活动详情回读确认 checked_in_today。**

> 当前版本：**v1.3.4**（2026-11-14） · 核心能力：query/claim 双模式、活动组识别、固定锁、pending 台账、脱敏诊断

[简体中文](README.md) · [English](docs/README.en.md) · [繁體中文](docs/README.zh-Hant.md) · [日本語](docs/README.ja.md) · [한국어](docs/README.ko.md) · [Português (BR)](docs/README.pt-BR.md)

<a href="https://github.com/L0NE-6/DoubaoWork-Daily/releases"><img src="https://img.shields.io/github/v/release/L0NE-6/DoubaoWork-Daily?style=for-the-badge&label=Release&color=2ea44f&logo=github" alt="Release" /></a>
<img src="https://img.shields.io/github/stars/L0NE-6/DoubaoWork-Daily?style=for-the-badge&label=Stars&color=FFC75F&logo=github" alt="Stars" />
<img src="https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python" />
<img src="https://img.shields.io/badge/Platform-Windows%20%7C%20Linux%20%7C%20Qinglong-4EAA25?style=for-the-badge&logo=linux&logoColor=white" alt="Platform" />
<img src="https://img.shields.io/github/license/L0NE-6/DoubaoWork-Daily?style=for-the-badge&label=License&color=F472B6" alt="License" />
<a href="https://github.com/L0NE-6/DoubaoWork-Daily/releases"><img src="https://img.shields.io/badge/Download-Releases-2ea44f?style=for-the-badge&logo=github" alt="Download" /></a>

[🚫 使用边界](#usage-policy) · [✨ 30 秒速览](#overview) · [🧭 这是什么](#what-is-this) · [🔁 工作原理](#protocol) · [🔑 凭据](#credentials) · [🚀 本地](#local) · [🐧 青龙](#qinglong) · [☁️ Actions](#actions) · [⌨️ CLI](#cli-output) · [🧾 结果码](#result-codes) · [✅ 成功判据](#success) · [🧪 测试](#tests) · [🧱 设计原则](#design) · [🔒 隐私](#privacy) · [📁 目录](#layout) · [📦 版本](#release) · [⚠️ 免责声明](#disclaimer) · [☕ 赞赏](#donate) · [💬 反馈与 Star](#feedback)

</div>

---

<a id="usage-policy"></a>
## 🚫 使用边界与转载声明

> [!IMPORTANT]
> **官方版本永久免费。** 未经作者书面许可，禁止付费包装、搬运引流、冒充作者，或使用作者身份与二维码制造官方授权假象。

本项目仓库、脚本、部署说明和 Release 下载始终免费公开。未经作者明确书面许可，禁止将其包装为付费资源、内部版、独家版、会员权益、收费课程、付费部署或包更新服务；禁止搬运到其他账号、群组、频道、网盘、电商或付费平台进行涨粉、截流、倒卖和引流。

如需商业合作、品牌授权、联合分发或使用作者名义提供付费服务，请先取得作者明确书面授权。完整规则见 [NOTICE.md](NOTICE.md)。

---

<a id="overview"></a>
## ✨ 30 秒速览

| 维度 | 说明 |
| :--- | :--- |
| **做什么** | 读取豆包工作活动，定位唯一 `daily_checkin`，领取每日奖励并回查状态。 |
| **输入** | 豆包网页 Cookie，必须包含 `sessionid` 或 `sessionid_ss`；支持多账号。 |
| **输出** | 已领取、可领取、活动未投放、条件未满足、确认中、奖励提示和连续天数。 |
| **部署** | Windows、Linux、青龙面板、GitHub Actions。 |
| **依赖** | Python 3 + `requests`；Actions 固定 Python 3.11。 |
| **成功判据** | landing 回执 `checked_in=true` 后，活动详情仍须返回 `checked_in_today=true`。 |

---

<a id="what-is-this"></a>
## 🧭 这是什么

脚本只处理 `https://www.doubao.com` 国内站。它先通过 `/alice/user/launch` 识别活动组，再读取 v3 详情或普通活动列表，只在候选记录中存在唯一 `daily_checkin` 任务时继续。

领取前，脚本在固定锁内读取并占用 pending 台账；领取后按 0、2、4 秒回查同一 `group/item`。确认预算为 18 秒，未确认任务按 10 分钟间隔、30 分钟窗口和最多 4 轮安排后续只读回查。

---

<a id="protocol"></a>
## 🔁 协议、工作原理与回读

```text
POST /alice/user/launch
POST /alice/ug/campaign/v3/detail/
POST /alice/ug/campaign/list/
POST /alice/ug/campaign/landing/checkin  {"scene":"<campaign_group_code>"}
```

完整链路：

```text
Cookie
  → launch
  → campaign detail / list
  → 唯一 daily_checkin
  → 固定锁内读取 pending
  → 原子占用并保存台账
  → landing/checkin
  → 回执 checked_in=true
  → 详情回读 checked_in_today=true
```

如果 landing 返回 `710010702`，当前页面需要官方活动页 SDK 生成 `biz_params` 和 `AgwCommonParam`。脚本不会伪造这两个风控字段，也不会把动作回执写成完成；pending 会保留，后续只读回查。

---

<a id="credentials"></a>
## 🔑 凭据与环境变量

### 账号

| 变量 | 必填 | 说明 |
| :--- | :---: | :--- |
| `DOUBAOWORK_COOKIES` | 二选一 | 多账号，每行 `备注\|sessionid=...; sessionid_ss=...` |
| `DOUBAOWORK_COOKIE` | 二选一 | 单账号 Cookie |
| `DOUBAOWORK_ACCOUNTS_FILE` | 否 | 账号 JSON 文件 |

### 请求与通知

| 变量 | 说明 |
| :--- | :--- |
| `DOUBAOWORK_DEVICE_ID` | 可选设备 ID |
| `DOUBAOWORK_WEB_ID` | 可选 Web ID |
| `DOUBAOWORK_FINGERPRINT` | 可选指纹；缺省读取 Cookie |
| `DOUBAOWORK_USER_AGENT` | 可选 User-Agent |
| `DOUBAOWORK_CLIENT_VERSION` | 默认 `2.33.5` |
| `DOUBAOWORK_PROXY` | HTTP(S) / SOCKS5 代理 |
| `DOUBAOWORK_CAMPAIGN_GROUP` | 只用于 query 的缺组详情探测 |
| `DOUBAOWORK_NO_NOTIFY` | `1` 表示关闭通知 |
| `DOUBAOWORK_BASE_URL` | 默认 `https://www.doubao.com` |
| `DOUBAOWORK_STATE_FILE` | pending 台账路径 |
| `DOUBAOWORK_LOCK_FILE` | 固定锁文件路径 |
| `PUSHPLUS_TOKEN` / `BARK_URL` / `WECOM_WEBHOOK` | 可选推送 |

账号 JSON 支持数组或 `{"accounts": [...]}`，字段包括 `name`、`cookie`、`user_agent`、`client_version`、`device_id`、`web_id`、`fingerprint`、`proxy`。

---

<a id="local"></a>
## 🚀 本地快速开始

```bash
python -m pip install -r requirements.txt
```

获取 Cookie：

1. 登录 `https://www.doubao.com/chat/`。
2. 打开开发者工具 Network。
3. 复制请求中的完整 Cookie。
4. 确认包含 `sessionid` 或 `sessionid_ss`。

单账号：

```text
DOUBAOWORK_COOKIE=sessionid=...; sessionid_ss=...
```

多账号：

```text
DOUBAOWORK_COOKIES=
主账号|sessionid=...; sessionid_ss=...
小号|sessionid=...; sessionid_ss=...
```

先只读核对，再领取：

```bash
python doubaowork_daily.py --query
python doubaowork_daily.py
```

---

<a id="qinglong"></a>
## 🐧 青龙面板

```bash
pip3 install requests
```

在青龙环境变量中填写 `DOUBAOWORK_COOKIES`，创建任务：

```text
15 9 * * * python3 doubaowork_daily.py
```

首次接入先手动执行 `--query`，确认活动结构、账号状态和出口网络都正常后再开启定时。同一 Cookie 不要同时在多个执行端提交 claim。

---

<a id="actions"></a>
## ☁️ GitHub Actions

工作流位于 [`.github/workflows/daily.yml`](.github/workflows/daily.yml)，默认 `mode=query` 且只读；只有手动选择 `mode=claim` 才运行领取命令。schedule 保持注释，定时事件也固定只读。

| Secret / Variable | 必填 | 说明 |
| :--- | :---: | :--- |
| `DOUBAOWORK_COOKIES` | 二选一 | 多账号 Cookie |
| `DOUBAOWORK_COOKIE` | 二选一 | 单账号 Cookie |
| `DOUBAOWORK_DEVICE_ID` / `DOUBAOWORK_WEB_ID` / `DOUBAOWORK_FINGERPRINT` | 否 | 设备上下文 |
| `DOUBAOWORK_PROXY` | 否 | 可信 HTTP(S) / SOCKS5 出口 |
| `DOUBAOWORK_CAMPAIGN_GROUP` | 否 | 默认 `pc_incentive_task_mvp_v3` |
| `PUSHPLUS_TOKEN` / `BARK_URL` / `WECOM_WEBHOOK` | 否 | 推送 |

GitHub 托管 runner 的公网出口可能看到 `campaign not found`，而同一 Cookie 在本地或可信出口可见。此时是活动可见性问题，不应伪造设备字段或重复 claim。Actions 主要作为只读诊断，实际领取优先使用本地或可信出口。

---

<a id="cli-output"></a>
## ⌨️ CLI 与输出

| 命令 | 作用 |
| :--- | :--- |
| `python doubaowork_daily.py` | 查询并领取每日奖励 |
| `python doubaowork_daily.py --query` | 只查询，不提交领取 |
| `python doubaowork_daily.py --only 1` | 只处理第一个账号 |
| `python doubaowork_daily.py --no-notify` | 本轮关闭通知 |
| `python doubaowork_daily.py --version` | 显示脚本版本 |

每个账号输出一行脱敏诊断，包含请求路径、HTTP 状态、业务码、消息、`checked_in_today`、pending 状态和设备上下文是否存在；不输出 Cookie、Token、手机号或设备原值。

---

<a id="result-codes"></a>
## 🧾 结果码

| 状态 / 退出码 | 含义 |
| :--- | :--- |
| `claimed` / 退出码 `0` | 详情回读确认今日已完成 |
| `already` | 今日此前已领取 |
| `confirming` | 已提交但详情未确认，pending 保留，后续只读回查 |
| `inactive` | 活动未投放、任务不唯一或条件未满足 |
| `available` / `done` | query 模式下的可领取 / 已领取 |
| 退出码 `1` | 缺少账号、Cookie 失效、限流或至少一个账号异常 |
| HTTP `401/403/429` | 登录失效、直接拒绝或限流 |
| `710010702` | 页面要求官方 SDK 风控字段，需在官方页面完成签到 |

---

<a id="success"></a>
## ✅ 成功判据

landing 回执只有 `checked_in=true` 才算动作被接受。最终完成必须再读取活动详情并看到 `checked_in_today=true`。

HTTP 200、业务 `code=0`、landing 回执或 pending 台账单独出现都不构成成功。活动详情不可见、任务不唯一或确认超时，应报告 `confirming`、`inactive` 或异常。

---

<a id="tests"></a>
## 🧪 测试

```bash
python -m pytest tests -q
python -m py_compile doubaowork_daily.py
git diff --check
```

现有 46 项离线测试覆盖活动组识别、query 缺组详情、claim 写保护、回读状态、pending 台账、固定锁、Cookie 形态、诊断脱敏、限流与风控错误分类。

---

<a id="design"></a>
## 🧱 设计原则

- **只读优先**：`--query` 是默认诊断路径。
- **唯一任务**：出现多个候选时不猜测领取对象。
- **写前落盘**：pending 台账先原子保存，再提交领取。
- **动作与状态分离**：landing 回执和详情回读分别判定。
- **固定锁**：线程锁加跨进程锁，避免同机并发写。
- **最小台账**：只保存 Cookie 哈希、`group/item`、时间、轮数和奖励提示。

---

<a id="privacy"></a>
## 🔒 隐私与数据边界

- Cookie 等同于登录态，只放在环境变量、GitHub Secrets 或本地临时 shell。
- 不提交 Cookie、device_id、web_id、指纹或完整账号信息。
- 台账不保存 Cookie 明文，只保存不可逆账号指纹和最小签到状态。
- 诊断日志只输出字段存在性和脱敏状态。
- Cookie 曾公开时，应立即退出会话或重新登录使旧 Cookie 失效。

---

<a id="layout"></a>
## 📁 目录结构

```text
DoubaoWork-Daily/
├── .github/workflows/daily.yml
├── assets/
├── docs/
├── tests/
├── doubaowork_daily.py
├── CHANGELOG.md
├── LICENSE
├── NOTICE.md
├── requirements.txt
├── ruff.toml
└── README.md
```

---

<a id="release"></a>
## 📦 版本与发布

- 当前版本：`v1.3.4`，补充 Actions 出口诊断与可信出口边界。
- 历史版本保留在 [Releases](https://github.com/L0NE-6/DoubaoWork-Daily/releases)。
- 变更记录见 [CHANGELOG.md](CHANGELOG.md)。

---

<a id="disclaimer"></a>
## ⚠️ 免责声明

本项目用于个人学习与合法自动化实践。使用者应自行确认账号、服务条款、活动规则和部署环境；因误配置、Cookie 泄露、活动可见性变化或账号异常产生的风险由使用者自行承担。

---

<a id="donate"></a>
## ☕ 支持与投喂：微信与支付宝赞赏码

脚本完全免费，赞赏纯自愿，不影响任何功能。

<p align="center">
  <img src="assets/donate-wechat.png" width="260" alt="微信赞赏码" />
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <img src="assets/donate-alipay.jpg" width="260" alt="支付宝收款码" />
</p>

<p align="center"><sub>微信支付（左） · 支付宝（右）</sub></p>

> [!WARNING]
> 收款码仅用于向作者自愿赞赏。任何第三方以代下载、代部署、内部资源、会员群或官方授权为由收费，均与作者无关。

---

<a id="feedback"></a>
## 💬 反馈与贡献：Star History

反馈时请附脱敏后的运行状态和接口阶段，不要提交 Cookie、device_id、web_id 或完整账号信息。

- [Issues](https://github.com/L0NE-6/DoubaoWork-Daily/issues)
- [Pull Requests](https://github.com/L0NE-6/DoubaoWork-Daily/pulls)
- [Releases](https://github.com/L0NE-6/DoubaoWork-Daily/releases)

<a href="https://star-history.com/#L0NE-6/DoubaoWork-Daily&Date">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=L0NE-6/DoubaoWork-Daily&type=Date&theme=dark" />
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=L0NE-6/DoubaoWork-Daily&type=Date&theme=light" />
    <img alt="Doubao Work Daily Star History" src="https://api.star-history.com/svg?repos=L0NE-6/DoubaoWork-Daily&type=Date&theme=light" />
  </picture>
</a>

<div align="center">

[⭐ Star](https://github.com/L0NE-6/DoubaoWork-Daily) · [🍴 Fork](https://github.com/L0NE-6/DoubaoWork-Daily/fork) · L0NE-6 · DAILY SIGNAL

</div>
