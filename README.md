# Serverless-Time-Series-Monitor
基于 Cloudflare Workers + D1 构建的高频异构时序数据流监控与自适应平滑预测边缘计算架构
# 📈 全球 QDII 基金量化套利与综合监控终端
**Global QDII Quant & Arbitrage Terminal**

[![Cloudflare Workers](https://img.shields.io/badge/Cloudflare-Workers-F38020?logo=cloudflare&logoColor=white)](#)
[![Cloudflare D1](https://img.shields.io/badge/Cloudflare-D1-F38020?logo=cloudflare&logoColor=white)](#)
[![Cloudflare KV](https://img.shields.io/badge/Cloudflare-KV-F38020?logo=cloudflare&logoColor=white)](#)
[![ECharts](https://img.shields.io/badge/ECharts-5.5.0-E43961?logo=apache-echarts&logoColor=white)](#)

一个基于 **Cloudflare Workers + D1 + KV** 的 QDII 基金全景监控、估值推演、量化研究、统计套利、实盘复盘、AI 决策与第三方程序快照输出系统。系统采用纯 Serverless 架构，无需自建服务器，适合个人量化研究与套利观察场景。

主要面向：
- 纳斯达克相关 QDII / ETF / LOF
- 标普相关 QDII / ETF / LOF
- 日经相关基金历史研究
- 跨境 ETF / LOF 溢价观察
- 配对套利、轮动切换、动态估值与实盘记录

---

## 目录

- [功能总览](#功能总览)
- [核心能力详解](#核心能力详解)
- [后端 API 能力](#后端-api-能力)
- [数据存储设计](#数据存储设计)
- [认证与安全](#认证与安全)
- [Cron 与自动化能力](#cron-与自动化能力)
- [第三方程序接入](#第三方程序接入)
- [技术架构](#技术架构)
- [快速部署](#快速部署)
- [系统设置说明](#系统设置说明)
- [雷达推送说明](#雷达推送说明)
- [AI 模型支持](#ai-模型支持)
- [适用场景](#适用场景)
- [更新日志](#更新日志)
- [免责声明](#免责声明)
- [License](#license)

---

## 功能总览

当前系统已实现以下完整能力，前端对应面板如下：

### 1. 盘中监控
- 实时抓取汇率、指数、期货、基金市价
- 推导盘中实时净值与动态溢价率
- 盘中自动/手动刷新，实时监控大屏
- 按资产类别分组展示盘中监控图表
- 雷达卡片、告警/推送/排序管理

### 2. 实时估值
- 动态溢价率排序
- 实时市价 / 净值 / 官方净值 / 盘中涨跌 / 晚间预估净值
- 估值公式说明

### 3. 深度研究与统计套利
- 多基金历史序列比较、差值序列绘图、净值涨跌幅对比
- 静态/动态布林带回测、多组标准差网格回测
- 历史触发明细弹窗、配对套利参数矩阵
- 图表联动与触发记录

### 4. 水位 / 极值
- 历史极值、时间分位、空间分位、位置水位判断

### 5. 量化矩阵（多因子决策）
- 绝对空间：历史均值、标准差、Z-Score、时间分位
- 相对估值：同类均值、同类偏离 Spread、Peer Z-Score
- 动量趋势：7日/30日均线、DIF、Momentum
- 综合诊断信号：左侧抄底 / 右侧持有 / 高估切换等提示

### 6. 网格执行
- 挂单位推导：S2/S1/均值中枢/R1/R2 执行区间计算
- 适合 T+0 挂单、分批买卖、区间交易、做 T 执行参考

### 7. 统计套利 / 多基轮动
- 多基金轮动池构建、相对均值回归信号、动态 Z-Score 指令
- 满仓轮转策略全局寻优、轮动图表与动态策略矩阵

### 8. 实盘战报
- 多账户（主账户/子账户1/子账户2）交易流水录入、增删改查
- 实盘收益曲线、策略超额收益对比、历史战报矩阵

### 9. 数据审计
- 历史缺失数据增量抓取、单基金全历史同步
- 历史数据分页查询与图表展示
- 估算净值修正、批量修正、数据库物理硬复权、CSV 导出

### 10. AI 决策
- 多模型路由（Google / NVIDIA / 自定义代理 / ModelScope）
- 流式对话输出，全盘量化矩阵自动注入
- 自动让 AI 依据宏观环境、量化矩阵、深度对比、回测结果给出配对套利/高低切换/抄底逃顶建议

### 11. 估值修正
- 日级修正因子管理、批量修正

### 12. 系统设置
- 自动刷新频率、推送通道 Webhook/Key、雷达配置等用户偏好持久化保存

---

## 核心能力详解

### 实时净值推演引擎

完整 QDII 盘中估值推演链路，核心包含：

- T-2 官方净值
- 隔夜标的涨幅
- 人民币中间价影响
- 盘中外盘期指影响
- 离岸人民币 CNH 盘中影响
- 管理费日损耗近似扣减

最终可得 **今晚预估净值（T-1）**、**盘中实时净值** 与 **动态溢价率**，使系统不仅能看"涨跌"，还能看"涨跌是否合理"。

---

### 雷达监控系统

#### 支持的雷达类型
- 单基金雷达
- 双基金相对价差雷达

#### 支持的策略要素
- 统计窗口 `windowN`
- 买入系数 `kBuy`
- 卖出系数 `kSell`
- 自定义买线 `customBuy` / 卖线 `customSell`

#### 信号输出
`WATCH` / `BUY` / `SELL`

#### 附加能力
- 买报/卖报独立开关，买推/卖推独立开关
- 推送冷却防重复
- 声音告警静音状态持久化
- 雷达卡片前移/后移排序

---

### 声音报警系统

- 使用原生 `AudioContext`，支持独立静音、测试试播、全局防抖
- 适合盘中盯盘场景

---

### 多通道消息推送系统

接入通道：**Bark / PushDeer / 钉钉机器人 / 飞书机器人**

已实现：
- 测试推送接口
- 雷达推送接口（含推送状态日志返回）
- 多通道同时群发
- Cron / 前端共用推送链路

---

## 后端 API 能力

### 调度与运维
- `GET /cron` — 手动 Cron 调度中心 UI
- `POST /api/run_cron` — 手动执行 Cron 抓取任务

### 用户与配置
- `GET /api/prefs`
- `POST /api/prefs`

### 实时与统计
- `GET /api/realtime_dashboard`
- `GET /api/stats`
- `GET /api/chart_data`
- `GET /api/data`
- `GET /api/summary`

### 历史抓取
- `POST /api/trigger`
- `POST /api/fetch_fund_history`

### 修正与校验
- `GET /api/corrections`
- `POST /api/corrections`
- `POST /api/update_est_nav`
- `POST /api/db_split_adjust`
- `POST /api/bulk_corrections`
- `GET /api/verify_error`

### 实盘交易
- `GET /api/trades`
- `POST /api/trades`
- `PUT /api/trades`
- `DELETE /api/trades`

### AI 与推送
- `POST /api/chat`
- `POST /api/test_push`
- `POST /api/radar_push`

### 第三方快照
- `GET /api/agent_snapshot`

---

## 数据存储设计

### D1 数据库
主要用于保存：
- 基金历史数据
- 实盘交易流水
- 历史估值底稿

### KV
主要用于保存：
- 用户偏好 `USER_CONFIG_PREFS`
- 当日修正因子 `CORRECT_日期`
- 宏观缓存 `LATEST_MACRO_CACHE`
- 第三方快照 `AGENT_SNAPSHOT_V1`
- 其他临时或调度状态数据

---

## 认证与安全

### 1. 登录密码
- 通过 `ADMIN_TOKEN` 保护，支持前端登录与路径注入登录形式

### 2. API Header 鉴权
所有受保护 API 可通过以下方式访问：

```http
Authorization: YOUR_ADMIN_TOKEN
```

### 3. 第三方快照读取
`/api/agent_snapshot` 已支持 Header 认证与可扩展直访参数认证。

---

## Cron 与自动化能力

### 已实现内容
- Worker `scheduled()` 定时任务
- 手动 Cron 调度中心
- 自动净值更新、盘前/盘中逻辑执行
- 自动快照生成、宏观缓存刷新检查

### 当前 Cron 时间表

#### 盘前 / 盘中触发
```cron
*/3 1-8 * * *
```
对应北京时间约 `09:00 - 16:59`，每 3 分钟执行一次盘中流程。

#### 非盘中净值更新
```cron
*/3 0,14 * * *
```
对应 UTC `0` 点（北京时间 `08:00-08:59`）和 UTC `14` 点（北京时间 `22:00-22:59`），每 3 分钟执行一次净值更新任务。

---

## 第三方程序接入

为方便 Python、自动化工具、智能体读取数据，程序提供快照接口：

```http
GET /api/agent_snapshot
```

返回内容主要包含：
- `rates`：汇率、期货、指数宏观因子
- `radar`：雷达状态快照
- `realtime_valuation`：实时净值推演与估值明细

项目同时提供示例工具 `agent_snapshot_analyzer.py`，可自动请求快照、输出字段说明表、整合分析雷达与估值数据。

适合：OpenClow、Python 脚本、自动报告系统、第三方智能工具。

---

## 技术架构

本系统采用纯 Serverless 方案，无需自建服务器：

| 层级 | 技术 |
|------|------|
| 后端计算 | Cloudflare Workers |
| 数据库 | Cloudflare D1 (SQLite) |
| 配置与状态 | Cloudflare KV |
| 前端界面 | Vanilla JS + HTML + CSS + ECharts 5 |
| 定时任务 | Cloudflare Cron Triggers |

---

## 快速部署

### 1. 准备环境

确保已安装 Node.js 与 Wrangler CLI：

```bash
npm install -g wrangler
wrangler login
```

### 2. 创建 D1 数据库

```bash
wrangler d1 create fund-history-db
```

记录返回的 `database_name` 和 `database_id`。

### 3. 创建 KV 命名空间

```bash
wrangler kv:namespace create FUND_KV
```

记录返回的 KV `id`。

### 4. 配置 `wrangler.toml`

```toml
name = "qdii-quant-monitor"
main = "worker.js"
compatibility_date = "2024-03-20"

[[d1_databases]]
binding = "DB"
database_name = "fund-history-db"
database_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

[[kv_namespaces]]
binding = "FUND_KV"
id = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

[triggers]
crons = ["*/3 1-8 * * *", "*/3 0,14 * * *"]

[vars]
DEFAULT_FEE_RATE = "0.008"
```

> 如果你的入口文件是 `src/index_new.js`，请将 `main` 改成你的实际路径：
>
> ```toml
> main = "src/index_new.js"
> ```

### 5. 注入运行时密钥

```bash
wrangler secret put ADMIN_TOKEN
wrangler secret put GOOGLE_API_KEY
wrangler secret put NVIDIA_API_KEY
wrangler secret put MODELSCOPE_API_KEY
wrangler secret put CUSTOM_API_KEY
```

### 6. 初始化数据库

#### 表 1：基金历史数据

```sql
CREATE TABLE IF NOT EXISTS fund_history (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    Code TEXT NOT NULL,
    Price_Date TEXT NOT NULL,
    NAV_Date TEXT,
    Market_Price REAL,
    Official_NAV REAL,
    Static_Premium REAL,
    Est_NAV REAL,
    Scrape_Time TEXT,
    UNIQUE(Code, Price_Date)
);
```

#### 表 2：交易流水

```sql
CREATE TABLE IF NOT EXISTS trade_journal (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    account TEXT DEFAULT 'Main',
    trade_date TEXT NOT NULL,
    fund_from TEXT,
    fund_to TEXT,
    qty_from REAL DEFAULT 0,
    price_from REAL DEFAULT 0,
    qty_to REAL DEFAULT 0,
    price_to REAL DEFAULT 0,
    actual_alpha REAL DEFAULT 0,
    note TEXT,
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

> 如你的当前版本还包含其他业务表，请根据实际代码补充建表语句。

### 7. 发布部署

```bash
wrangler deploy
```

部署完成后，访问 Cloudflare 分配的域名即可使用。

---

## 系统设置说明

系统设置中可配置：
- 自动刷新频率
- 推送通道 Webhook / Key（Bark / PushDeer / 钉钉机器人 / 飞书机器人）
- 其他用户偏好项

系统提供一键测试推送、雷达触发推送、多通道同时发送。

---

## 雷达推送说明

### 推送开关
- 买点推送开关（独立）
- 卖点推送开关（独立）
- 声音报警开关（独立）

### 推送内容包含
- 信号类型、监控组合、实时 Spread、买点 S / 卖点 R、窗口参数、触发时间

### 注意事项
- 雷达推送可由前端盘中监控逻辑触发，也可扩展为服务端 Cron 无人值守推送
- 页面切到后台且监控已暂停时，前端通常不会继续触发推送
- 多设备同时开启同一页面监控时，若都在本地运行判断逻辑，可能出现重复推送风险

---

## AI 模型支持

系统支持多模型路由，可接入不同提供方的聊天模型接口（Google / NVIDIA / 自定义代理 / ModelScope）。AI 不只作为普通聊天，而是被当作"量化投顾分析引擎"使用，可用于：

- AI 投顾与数据解读
- 交易建议辅助、策略总结
- 风险提示生成

---

## 适用场景

- QDII 日内监控
- 动态溢价套利
- 纳指 / 标普相关基金比较
- 左侧抄底 / 右侧确认
- 同类高低切换
- 轮动资金池管理
- 历史策略复盘
- 半自动化投顾分析
- 第三方程序读取结构化快照数据

---

## 更新日志

# 版本更新 V3.6.7

**🚀 核心更新：报警推送通道全面升级**
* **通道更迭**：系统正式下线 PushDeer 推送通道，全面迁移并接入“Server酱 (ServerChan)”。
* **体验升级**：得益于 Server酱的底层支持，现在的监控报警通知原生支持 Markdown 富文本格式，数据网格与溢价状态展现更加清晰美观。
* **无缝过渡**：Dashboard 设置面板已同步重构，支持直接录入和管理 Server酱的 SendKey（支持多 Key 并存），并自动打通底层 KV 持久化存储。
* **稳定性优化**：重构了底层的雷达推送网络请求逻辑，采用更稳定的原生 POST 架构提交表单数据，提升推送触达的可靠性。

 

# 版本更新 V3.6.5

**🚀 核心更新**
* **快照持久化**：新增盘中估算净值（Est_NAV）每 3 分钟自动写入 KV 缓存。
* **修正即时重算**：执行手动修正时，自动全链路重算并强制覆写当日快照。
* **历史数据闭环**：官方净值落库时，自动校验并回填当日估算快照至 D1 数据库。
* **前端性能优化**：实时大盘接口优先直出 KV 快照，前端识别标记后跳过本地重算。

## 版本更新说明：v3.6.2

本次更新重点修正 Tab 盘中监控底部"状态机阶段"展示逻辑，使前端阶段说明与后端真实估值计算口径一致。

### 核心更新

- 修正盘中监控阶段显示逻辑
  - 不再使用 `updatedFunds.length > outdatedFunds.length` 的多数表决判断阶段一
  - 改为与后端双护盾状态机一致：美股开盘优先、净值日期逐只基金判断
  - 区分"全部已更新""部分已更新""全部未更新"三种非美股开盘状态

- 前端状态说明与后端真实计算对齐
  - 美股开盘时显示阶段二：期指护盾开启，期指因子强制为 0
  - 全部基金 `nav_date === spot_date` 时显示阶段一：现货护盾开启，现货因子为 0，期指正常参与
  - 部分基金日期追平时显示过渡状态：已更新基金杀死现货，未更新基金继续使用现货涨幅补算
  - 全部基金日期未追平时显示阶段三：现货与期指均正常参与估值

- 增强白盒调试可读性
  - 增加已更新、未更新和总样本数量提示
  - 明确阶段显示是估值状态，不再误导为固定北京时间段

### 修复与改进

- 修复阶段一在净值陆续更新时不易出现或显示不准确的问题
- 修复前端阶段文案与后端 `effective_fut_pct` / `effective_spot_pct` 真实计算口径不一致的问题
- 提升盘中监控状态机排查体验，便于判断当前估值使用的是现货、期指还是双护盾后的有效因子

---

## 版本更新说明：v3.6.0

本次更新重点优化了 Cron 定时任务、宏观缓存维护和雷达推送执行链路，提升盘中实时任务的稳定性与执行效率。

### 核心更新

- 优化 Cloudflare Cron 调度逻辑
  - 明确区分盘中实时任务与非盘中净值更新任务
  - `*/3 1-7 * * *` 用于北京时间 09:00-15:59 盘中实时任务
  - `*/3 0,14 * * *` 用于北京时间 08:00、22:00 时段净值/历史数据任务
  - 增加 Cron 执行日志，输出 UTC 时间、北京时间与当前 Cron 表达式

- 优化宏观 KV 快照维护机制
  - `LATEST_MACRO_CACHE` 统一由 Cron 盘中任务维护
  - 普通 `/api/realtime_dashboard` 访问不再主动写入宏观 KV
  - 保留 5 分钟冷却机制，减少 KV 写入频率
  - API 异常时继续支持从 KV 缓存兜底恢复

- 优化盘中实时任务链路
  - `executeIntradayRealtimePass()` 通过 `buildRealtimeDashboardData(env, ctx, "cron")` 拉取实时盘面
  - Cron 模式下允许按冷却时间刷新宏观缓存
  - 盘中任务完成后同步刷新 AI 快照

- 优化雷达 Cron 推送性能
  - `executeRadarCronPush()` 新增 `realtimeData` 参数
  - 雷达扫描优先复用盘中任务已拉取的 `rates` 与 `funds`
  - 避免同一次 Cron 中重复调用 `fetchRealtimeRates()` / `fetchRealtimeFunds()`
  - 雷达模块只保留宏观缓存异常兜底读取，不再重复写入 `LATEST_MACRO_CACHE`

- 增强 Cron 可观测性
  - 增加 `[CRON ENTER]`、`[CRON EXPR]`、`[CRON TIME]`、`[CRON EXIT]` 等日志
  - 便于排查 Cron 是否触发、命中哪个分支、是否完成任务

### 修复与改进

- 修复盘中 Cron 内部重复拉取实时行情的问题
- 修复雷达 Cron 与实时大屏数据口径可能不一致的问题
- 优化宏观缓存写入职责，避免多入口重复维护 KV
- 明确 Cloudflare Cron UTC 时间与北京时间的对应关系
- 为后续 Cron 异常排查提供更清晰日志基础

---

### V3.5.1

- 新增链式轮转收益算法 computeChainRotationAlphas()
  - 支持按账户独立计算，避免 Main/Sub1/Sub2 混算
  - 支持 A➔B➔A 双基金闭环
  - 支持 A➔B➔C➔A / A➔B➔C➔D➔A 多基金链式闭环
  - 按 trade_date + id 正序回溯交易路径
  - 自动识别链路断点并重新开启新链
  - 排除 CASH / OTC 等非基金换仓场景

- 优化实盘战报 loadTrades() 累计利差逻辑
  - 累计利差由单纯累计 actual_alpha 升级为累计有效利差
  - 优先使用手工录入或数据库 actual_alpha
  - 当 actual_alpha 为 0 时，自动采用链式闭环计算结果
  - 表格新增链式闭环标识与链路备注展示
  - 利差显示精度由 2 位小数提升至 3 位小数

- 重构新增/编辑交易时的自动利差核算
  - 保存交易时自动将当前草稿并入历史账本进行闭环检测
  - 当当前交易构成链式闭环时，自动写入本轮捕获利差
  - 弹窗提示闭环路径与自动核算结果
  - 编辑模式下自动排除原记录，避免重复计算

- 重构"智能核算"按钮逻辑
  - 由原先的 A/B 反向匹配升级为链式闭环核算
  - 支持未闭环时提示当前识别路径
  - 支持闭环后自动填充 trade_alpha 输入框
  - 提升三角轮动、多基金轮动场景下的核算准确性

- 重构"一键智能修正"历史利差逻辑
  - 使用链式轮转算法批量回溯历史记录
  - 自动修正 actual_alpha 为 0 但可识别闭环收益的记录
  - 自动补充链式闭环路径备注
  - 支持批量写入数据库并刷新实盘战报

- 修复/增强
  - 解决三角轮转场景下实收利差不累计的问题
  - 解决 A➔B➔C➔A 路径无法自动识别收益的问题
  - 提升 T+0 多基金轮动套利的实盘收益统计准确性
  - 改善小幅利差因两位小数显示为 0.00% 的可读性问题


### V3.5.0
- 架构重构：正式完成"统一服务端雷达推送体系"升级，前端自动推送逻辑全面迁移至 Cloudflare Worker 服务端执行。
- 新增：服务端统一雷达扫描引擎 `executeRadarCronScan()`，支持在 Cron 中集中执行所有雷达信号检测、状态判断与 Webhook 推送。
- 新增：`scheduled()` → `executeIntradayRealtimePass()` 已接入统一雷达扫描流程，实现实时盘面拉取、雷达信号计算、服务端 cooldown 控制、Webhook 推送的一体化自动执行。
- 新增：实现服务端统一配置读取 `loadUserPrefsForCron()`，支持从 KV `USER_CONFIG_PREFS` 中读取用户雷达与 Webhook 配置。
- 优化：前端 `saveUserPrefs()` 已支持通过 `/api/prefs` 自动同步用户配置至 Cloudflare KV，实现配置云端持久化。
- 新增：Cron 支持完全脱离浏览器运行，即使用户关闭页面、设备锁屏或离线，服务端仍可持续执行雷达扫描与推送。
- 重构：雷达 cooldown 状态统一迁移至服务端 `shouldSkipRadarPushByCooldown()` 管理，移除前端状态机依赖。
- 优化：禁用前端 `triggerRadarPush()` 自动推送逻辑，避免双重推送、前后端状态不一致、cooldown 冲突、多设备重复通知等问题。
- 修复：修正旧版雷达配置中 `pushSellEnabled` / `pushBuyEnabled` 字段缺失时被错误识别为开启状态的问题。
- 修复：服务端推送判断逻辑由 `!== false` 调整为 `=== true`，避免 `undefined → true` 导致误推送。
- 新增：新增雷达时强制写入完整字段结构（`pushBuyEnabled` / `pushSellEnabled` / `alertBuyEnabled` / `alertSellEnabled`），提高历史数据兼容性与系统稳定性。
- 优化：新增雷达数据标准化（Normalize）机制，可自动补齐旧版 KV 数据缺失字段，增强版本升级兼容能力。
- 优化：增强服务端日志输出，可清晰区分雷达扫描状态、cooldown 命中状态、Webhook 推送结果、Cron 执行链路。
- 兼容：保留原有页面 UI 雷达展示、卡片状态更新与本地交互逻辑，但推送权限已完全收归服务端统一管理。
- 架构升级：系统正式由"浏览器驱动型雷达工具"升级为"Cloudflare Worker 服务端量化监控平台"。


### V3.4.2
- 新增：`scheduled()` 中支持同时触发宏观缓存与第三方快照更新逻辑。
- 新增：将原有 `LATEST_MACRO_CACHE` 的 5 分钟冷却写入机制抽离为服务端公共函数，可在 Cron 中主动触发检查。
- 保留：原有 `/api/realtime_dashboard` 内部基于访问触发的 5 分钟异步写 KV 机制继续有效，不影响原页面访问链路。
- 新增：Cron 触发时可同步执行实时盘面逻辑、宏观缓存刷新检查、第三方快照更新。
- 优化：增加更清晰的 Cron 日志输出，便于区分盘前/盘中实时任务、非盘中净值更新任务、宏观缓存写入状态、快照更新状态。
- 兼容：网页访问触发与 Cron 定时触发可并存，支持被动刷新与主动刷新双通道机制。

### V3.4.1
- 新增：`/api/agent_snapshot` 支持通过浏览器直接访问快照数据。
- 优化：在原有 `Authorization` 请求头认证基础上，新增支持通过 URL 参数 `?token=` 传递访问密码。
- 兼容：保留原有程序化访问方式，第三方工具仍可继续通过请求头方式读取快照。
- 优化：便于 OpenClow、Python 脚本及其他自动化工具以更灵活的方式读取静态快照数据。
- 安全提示：`?token=` 方式适合临时调试和浏览器直访，不建议长期在公开环境下使用，以避免密码出现在地址栏、浏览器历史与日志中。

### V3.4.0
- 新增 `/api/agent_snapshot` 快照接口
- 新增 `agent_snapshot_analyzer.py` 数据分析示例脚本
- 支持网页访问初始化时生成快照
- 支持临时停用 Cron 对快照 API 的自动更新

### V3.3.5
- 新增：Cloudflare Worker `scheduled()` 增加更明显的 Cron 运行日志，便于排查定时任务是否真正触发、命中了哪个分支、执行是否完成。
- 调整：盘前/盘中定时任务采用 UTC `*/3 1-8 * * *`，每 3 分钟执行一次，逻辑等价于一次实时盘面拉取流程。
- 调整：非盘中净值更新任务采用 UTC `*/3 0,14 * * *`，分别在 UTC `0` 点和 `14` 点所在小时内每 3 分钟执行一次。
- 优化：Cron 分流日志增加 UTC 时间、北京时间、Cron 表达式、命中分支、执行完成与异常信息，便于线上快速定位问题。
- 兼容：不影响现有前端页面访问、盘中监控展示与原有接口调用逻辑。

### V3.3.3
- 新增：Tab1 盘中监控雷达卡片支持顺序调整，可通过按钮进行前移/后移。
- 优化：抽离雷达推送公共 Webhook 发送函数，统一 Bark / PushDeer / 钉钉 / 飞书推送逻辑。
- 优化：保留 `/api/radar_push` 接口，但改为复用公共推送函数，便于前端触发与服务端 Cron 触发共用同一套发送链路。
- 新增：为 Cron 自动雷达推送补齐服务端基础能力，支持在无人打开页面时由 Worker 定时触发自动扫描并推送。
- 兼容：不影响前端现有手动测试推送、普通雷达推送与盘中监控逻辑。

### v3.2.1
- 新增雷达自动推送定时生效机制。
- 默认仅允许北京时间 `09:00-16:00` 触发雷达自动推送，其余时间自动静默。
- 系统设置中的测试推送不受时间限制，可随时验证推送通道连通性。

### v3.2.0
- 狙击雷达完成方向级拆分：买点报警开关、卖点报警开关、买点推送开关、卖点推送开关。
- 买点与卖点信号支持独立控制报警和推送行为。
- 保留旧版 `pushEnabled` 配置兼容性，历史雷达数据可自动映射到买推/卖推。

### v3.1.x
- 接入 Bark / PushDeer / 钉钉 / 飞书推送通道
- 支持系统设置面板测试推送
- 推送能力与雷达逻辑打通
- 推送与声音报警能力分离

### v3.0.0
- 重构警报提示层
- 优化前端渲染稳定性
- 改进独立静音逻辑
- 提升整体响应速度与可维护性

---

## 免责声明

1. 本项目中的估值推演、统计指标、雷达信号、回测结果及 AI 输出内容，仅供学习、研究与辅助参考，不构成任何投资建议。
2. 跨境 ETF / QDII 产品存在汇率波动、流动性、溢价偏离、交易时差、申赎限制等多种风险。
3. 第三方数据接口可能存在延迟、缺失、变更或失效，使用时请结合人工复核。
4. 使用者应自行承担投资决策风险。
