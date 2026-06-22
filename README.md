# Daily IEEE Digest

每天自动筛选并发送 IEEE 论文邮件，默认包含：

- 论文标题
- 作者
- 摘要
- DOI 链接
- 期刊与分区信息

这个项目适合想要“每天自动收到几篇目标方向新论文”的用户，默认运行在 GitHub Actions 上，不需要常驻本地电脑。

## Quick Start

第一次使用，通常只需要 3 步。

### 1. Fork 本仓库

点击 GitHub 右上角 `Fork`，复制到你自己的仓库。

### 2. 配置邮箱和 5 个 Secrets

下面以 163 邮箱为例说明，这是仓库当前提供的默认配置路径：

1. 登录 163 邮箱网页版
2. 打开 `设置`
3. 找到 `POP3/SMTP/IMAP`
4. 开启 `SMTP`
5. 生成客户端授权码
6. 记下这个授权码，下面填 `SMTP_PASS` 时要用

然后进入你的 GitHub 仓库：

`Settings -> Secrets and variables -> Actions -> New repository secret`

添加下面 5 个值：

```text
SMTP_HOST=smtp.163.com
SMTP_PORT=465
SMTP_USER=your_163_email@163.com
SMTP_PASS=your_163_smtp_authorization_code
MAIL_TO=your_receiver@example.com
```

说明：

- `SMTP_USER`：发件邮箱
- `SMTP_PASS`：邮箱 SMTP 授权码，不是登录密码
- `MAIL_TO`：收件邮箱

可选：

```text
MAIL_FROM=your_163_email@163.com
```

如果不填，默认使用 `SMTP_USER` 作为发件人。

### 3. 手动运行一次

进入：

`Actions -> Daily IEEE Digest -> Run workflow`

如果日志里出现：

```text
Email sent.
```

说明已经配置成功。

## Default Behavior

仓库默认配置会：

- 每天自动发送 2 篇论文
- 关注电子、通信、天线、微波、雷达等方向
- 优先从 TWC、TAP、TMTT 检索
- 自动过滤 `Publication Information`、`Information for Authors`、`editorial` 等非论文页面
- 自动避免重复发送

摘要来源按这个顺序回退：

1. Crossref
2. DOI / 落地页元数据
3. OpenAlex

如果前面的来源没有摘要，会自动继续尝试后面的来源。

## What To Change

大多数用户只需要改 1 个文件：

[`config/journals.json`](F:/学科类竞赛/保研面试材料/模拟面试与项目准备问题/专业课复习与英语阅读/每日英文阅读/config/journals.json)

通常只看这 3 项：

- `journals`
- `include_keywords`
- `exclude_keywords`

### Example: Only Change Keywords

如果你不想改期刊，只想改方向，可以只改：

```json
"include_keywords": [
  "antenna",
  "beamforming",
  "communication",
  "mimo",
  "microwave",
  "radar",
  "wireless"
]
```

以及：

```json
"exclude_keywords": [
  "bio",
  "chemical",
  "materials",
  "polymer",
  "thin film"
]
```

## Local Preview

如果你想先在本地预览，但不发邮件：

```bash
python scripts/daily_ieee_digest.py --config config/journals.json
```

如果你想在本地直接测试 SMTP：

```powershell
$env:SMTP_HOST="smtp.163.com"
$env:SMTP_PORT="465"
$env:SMTP_USER="your_163_email@163.com"
$env:SMTP_PASS="your_163_smtp_authorization_code"
$env:MAIL_TO="your_receiver@example.com"
python scripts\daily_ieee_digest.py --config config\journals.json --send
```

也可以用：

```powershell
powershell -ExecutionPolicy Bypass -File scripts\send_test_163.ps1
```

## Advanced Configuration

只有在你明确想自定义行为时，再看这一节。

### Change Send Count

修改 [`.github/workflows/daily-ieee-digest.yml`](F:/学科类竞赛/保研面试材料/模拟面试与项目准备问题/专业课复习与英语阅读/每日英文阅读/.github/workflows/daily-ieee-digest.yml) 里的：

```yaml
DIGEST_MAX_ARTICLES: "2"
```

例如改成每天 5 篇：

```yaml
DIGEST_MAX_ARTICLES: "5"
```

### Change Send Time

默认是北京时间早上 8 点附近，使用 GitHub Actions 定时触发。

如果你想改成北京时间 21:30，可以把 cron 改成：

```yaml
- cron: "30 13 * * *"
```

并建议同时把：

```yaml
DIGEST_NOT_BEFORE: "21:00"
```

### Change Search Range

默认配置：

```yaml
DIGEST_DAYS_BACK: "1095"
DIGEST_ROWS_PER_JOURNAL: "100"
```

含义：

- `DIGEST_DAYS_BACK`：向前检索多少天
- `DIGEST_ROWS_PER_JOURNAL`：每个期刊从 Crossref 拉多少候选记录

## FAQ

### 为什么不是严格整点发送？

GitHub Actions 的定时触发可能有延迟，所以 workflow 里用了主触发加备份触发，再结合本地日期去重，尽量保证每天只发一次。

### 为什么有时摘要来源不一样？

不同论文在不同元数据源上的完整度不一样。这个项目会优先使用更直接的来源，如果拿不到，再自动回退到其他公开学术元数据源。

### 为什么不会重复发同一篇论文？

已经发送过的 DOI 会写入：

[`data/sent_history.json`](F:/学科类竞赛/保研面试材料/模拟面试与项目准备问题/专业课复习与英语阅读/每日英文阅读/data/sent_history.json)

后续运行时会自动跳过这些 DOI。

### 如果邮箱授权码泄露了怎么办？

去邮箱后台立即停用或重新生成授权码，然后更新仓库里的 `SMTP_PASS` secret。

## Recommended Open-Source Usage

如果你准备把这个仓库公开给别人用，建议保持“开箱即用”：

- 默认期刊先给一个合理样例
- 默认关键词先给一个合理样例
- README 首页只保留 `Fork -> 配邮箱 -> 配 Secrets -> Run workflow`

这样别人第一次看到就能直接跑起来，再按需改配置。
