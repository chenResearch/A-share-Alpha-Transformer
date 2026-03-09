# A-Share Alpha Transformer

基于 A 股量价数据的选股研究项目。项目使用 `TuShare` 获取数据，完成股票池清洗、Alpha 因子构建与检验，并分别用 `LightGBM` 和 `MLP` 进行训练、预测和策略表现评估。

## 项目流程

1. 数据获取：使用 `tushare` 下载股票量价数据和基础资料。
2. 数据处理：剔除科创板，去除 `ST` 股票，保留主板/中小板/创业板。
3. 因子提取：基于开高低收、成交量、成交额构建量价因子。
4. 因子检验：通过分组收益、累计收益、IC、ICIR 评估因子有效性。
5. 模型训练：使用 `2015.01` 至 `2020.04` 数据建模。
6. 模型预测：对 `2020.04` 之后的数据进行样本外预测。
7. 模型评估：计算收益、Sharpe、波动率、胜率、换手率、回撤等指标，并与沪深 300 对比。

## 项目结构

```text
A-Share Alpha Transformer/
├── part01-download-data.ipynb   # 数据下载
├── part02-alpha-gen.ipynb       # 因子生成（早期版本）
├── part02-alpha-gen-v2.ipynb    # 因子生成（改进版本）
├── part03-alpha-test.ipynb      # 因子检验
├── part04-alpha-lgb.ipynb       # LightGBM 训练、预测、回测
├── part05-alpha-mlp.ipynb       # MLP 训练、预测、回测
└── README.md
```

## 各部分说明

- `part01-download-data.ipynb`：通过 `TuShare Pro` 下载股票列表、交易日历和历史行情。
- `part02-alpha-gen-v2.ipynb`：构建量价 Alpha 因子，是后续建模的主要因子来源。
- `part03-alpha-test.ipynb`：进行单因子检验，包括分组收益和 IC 分析。
- `part04-alpha-lgb.ipynb`：使用 `LightGBM` 进行训练、打分、选股和收益评估。
- `part05-alpha-mlp.ipynb`：使用 `MLP` 进行训练、打分、选股和收益评估。

## 数据与股票池

### 数据来源

- `TuShare`：`stock_basic`、`pro_bar`、`index_daily`
- 行业分类文件：`a_stock_industry.csv` 或 `a_stock_industry.xlsx`

### 主要字段

- 量价字段：`open`、`high`、`low`、`close`、`pre_close`、`vol`、`amount`、`pct_chg`
- 衍生字段：`vwap`、`ret`、`ret1`、`ret2`、`open_up`

### 股票池规则

- 保留 `主板`、`中小板`、`创业板`
- 剔除 `科创板`
- 剔除名称中含 `ST` 的股票

## 因子与模型

### 因子构建

项目基于常见量价公式构建 Alpha 因子，包括：

- `ts_sum`、`sma`
- `ts_min`、`ts_max`
- `delay`、`delta`
- `rank`、`ts_rank`
- `correlation`、`covariance`

### 因子检验

`part03-alpha-test.ipynb` 主要使用以下方式评估单因子：

- 分组收益
- 多空收益
- 累计收益曲线
- IC 与 ICIR

### 模型设置

- 训练集：`2015.01` 至 `2020.04`
- 测试集：`2020.04` 之后
- 模型：`LightGBM`、`MLP`
- 预测逻辑：按交易日对股票预测值 `pred` 排序，选取得分最高的一组构建组合

## 评价指标

样本外阶段主要评估：

- 年化收益率 `return`
- 夏普比率 `sharpe`
- 波动率 `std`
- 胜率 `winratio`
- 换手率 `turnover`
- 平均持仓数 `stock_num`
- 月度最大回撤 `mdd_month`
- 月度胜率 `month_winratio`

同时使用 `000300.SH` 作为沪深 300 基准，比较策略与指数的相对表现。



## 项目特点

- 按量化选股完整流程组织
- 同时包含单因子分析和机器学习建模
- 使用时间切分进行样本内外验证
- 支持收益、Sharpe、回撤等多维度评估

## 后续可改进方向

- 增加基本面或行业中性化因子
- 使用滚动训练或 Walk-Forward 验证
- 加入更细致的交易成本和滑点建模
- 将 Notebook 进一步模块化、工程化
