# TradingAgent: 多 Agent 股票决策系统

TradingAgent 是一个面向量化交易决策的多 Agent 协作系统。它通过研究、风控与交易 Agent 的链式推理，整合技术面、基本面与新闻情绪，生成可执行的买卖决策与风险提示。

## 📋 目录

- [系统概述](#系统概述)
- [核心特性](#核心特性)
- [系统架构](#系统架构)
- [安装与配置](#安装与配置)
- [使用方法](#使用方法)
- [工作流程](#工作流程)
- [目录结构](#目录结构)
- [配置说明](#配置说明)
- [输出说明](#输出说明)
- [支持的模型](#支持的模型)

## 🎯 系统概述

TradingAgent 采用多角色协作的图式推理框架：

1. **研究阶段**：多名研究员（多空视角）检索行情、新闻、财报，形成研究观点  
2. **分析阶段**：分析师汇总信号，量化指标并生成初步结论  
3. **风险控制阶段**：风险经理审视波动、仓位与止盈止损建议  
4. **交易决策阶段**：交易员给出最终买/卖/观望及信心评级，并生成执行要点

## ✨ 核心特性

### 1. 多源数据融合
- 支持行情（yfinance/Alpha Vantage）、技术指标、财报、新闻情绪
- 可配置数据供应商，支持本地缓存和在线拉取

### 2. 多角色协作
- Bull/Bear 研究员、分析师、风险经理、交易员分工协作
- 通过图式流程汇总观点，减少单一视角偏差

### 3. 风控内嵌
- 评估波动、风险敞口与仓位建议
- 输出止盈止损区间与执行注意事项

### 4. 配置化与可扩展
- LLM、数据源、缓存目录、讨论轮次均可配置
- 简单替换 Vendor 或模型即可适配不同环境

## 🏗️ 系统架构

### 核心 Agent 组件

| Agent | 功能 | 位置 |
|-------|------|------|
| **BullResearcher / BearResearcher** | 多空视角研究 | `tradingagents/agents/researchers/` |
| **Analyst** | 汇总指标与研究结论 | `tradingagents/agents/analysts/` |
| **ResearchManager** | 组织研究、合成多视角观点 | `tradingagents/agents/managers/research_manager.py` |
| **RiskManager** | 风险评估与风控建议 | `tradingagents/agents/managers/risk_manager.py` |
| **Trader** | 最终交易决策与执行提示 | `tradingagents/agents/trader/trader.py` |

### 数据与工具模块

| 模块 | 功能 | 位置 |
|------|------|------|
| **core_stock_tools / technical_indicators_tools** | 行情与指标获取 | `tradingagents/agents/utils/` |
| **fundamental_data_tools** | 财报与财务指标 | `tradingagents/agents/utils/` |
| **news_data_tools** | 新闻与情绪获取 | `tradingagents/agents/utils/` |
| **dataflows/** | 数据 Vendor 统一路由（yfinance、Alpha Vantage、Google、OpenAI、本地缓存等） | `tradingagents/dataflows/` |
| **graph/** | 图式流程、信号合成与决策输出 | `tradingagents/graph/` |

## 📦 安装与配置

### 环境要求

- Python 3.12

### 安装依赖

```bash
pip install -r requirements.txt
```

### 配置文件

关键配置位于 `tradingagents/default_config.py`。可通过环境变量覆盖：

- `TRADINGAGENTS_DATA_DIR`：默认为空，使用online data
- `TRADINGAGENTS_RESULTS_DIR`：结果输出目录 ./results
- `OPENAI_API_KEY` / `OPENAI_BASE_URL`：如使用 OpenAI 兼容接口, 在tradingagents/.env中配置
- `ALPHA_VANTAGE_API_KEY`：使用Alpha Vantage获取线上数据。可以在https://www.alphavantage.co/support/#api-key 获取api，并同样放到tradingagents/.env中

## 🚀 使用方法

### 运行示例

```bash
python main.py  # 使用默认配置与示例日期/标的
```

或直接在代码中：

```python
from tradingagents.graph.trading_graph import TradingAgentsGraph
from tradingagents.default_config import DEFAULT_CONFIG

ta = TradingAgentsGraph(debug=True, config=DEFAULT_CONFIG.copy())
_, decision = ta.propagate("NVDA", "2024-05-10")
print(decision)
```

### 自定义配置

```python
from tradingagents.graph.trading_graph import TradingAgentsGraph
from tradingagents.default_config import DEFAULT_CONFIG

config = DEFAULT_CONFIG.copy()
config["deep_think_llm"] = "gpt-4o-mini"
config["quick_think_llm"] = "gpt-4o-mini"
config["data_vendors"] = {
    "core_stock_apis": "yfinance",
    "technical_indicators": "yfinance",
    "fundamental_data": "alpha_vantage",
    "news_data": "google",  # openai / alpha_vantage / google / local
}

ta = TradingAgentsGraph(debug=False, config=config)
_, decision = ta.propagate("AAPL", "2024-05-10")
print(decision)
```

## 🔄 工作流程

```
Ticker + Date
    ↓
数据获取 (行情/指标/财报/新闻)
    ↓
Bull/Bear 研究员形成多空观点
    ↓
分析师汇总信号与量化指标
    ↓
风险经理评估风险敞口与执行区间
    ↓
交易员输出最终买/卖/观望决策 + 信心度
```

## 📁 目录结构

```
TradingAgent/
├── main.py                  # 入口脚本
├── run.sh                   # 启动脚本
├── README.md
├── requirements.txt
├── eval_results/            # 样例运行日志
└── tradingagents/
    ├── default_config.py    # 默认配置
    ├── .env                 # 放置环境变量：OPENAI_API_KEY，OPENAI_BASE_URL，ALPHA_VANTAGE_API_KEY
    ├── agents/              # 各类 Agent
    │   ├── researchers/     # Bull/Bear 研究员
    │   ├── analysts/        # 分析师
    │   ├── managers/        # 研究/风险经理
    │   ├── trader/          # 交易员
    │   └── utils/           # 数据与指标工具
    ├── dataflows/           # 数据 Vendor 统一路由与缓存
    ├── graph/               # 决策图与信号处理
    └── README.md            # 代码层使用示例
```

## ⚙️ 配置说明

### 关键配置字段（节选，详见 `default_config.py`）

| 配置项 | 说明 | 示例 |
|--------|------|------|
| `deep_think_llm` | 深度推理模型 | `o4-mini` |
| `quick_think_llm` | 快速推理模型 | `gpt-4o-mini` |
| `max_debate_rounds` | 研究讨论轮数 | `1` |
| `max_risk_discuss_rounds` | 风控讨论轮数 | `1` |
| `data_vendors.core_stock_apis` | 行情数据源 | `yfinance` / `alpha_vantage` / `local` |
| `data_vendors.news_data` | 新闻数据源 | `alpha_vantage` / `openai` / `google` / `local` |
| `data_cache_dir` | 数据缓存目录 | `tradingagents/dataflows/data_cache` |

## 📊 输出说明

- 终端输出：最终决策（买/卖/观望）与信心度
- 日志：在 `eval_results/` 下保存完整状态与日志（示例见 `eval_results/NVDA/TradingAgentsStrategy_logs/`）
- 可在 `debug=True` 时查看完整消息链路与节点输出

## 🤖 支持的模型

- OpenAI 兼容模型（示例：`gpt-4o-mini`、`o4-mini`）
- 可通过 `backend_url` 接入兼容端点