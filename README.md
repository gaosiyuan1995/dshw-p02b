# P02b：CSMAR 上市公司财务特征清洗与分析

## 数据来源

- 数据库：CSMAR（国泰安）
- 原始文件位置：`data/data_raw_zip/`
- 解压后文件位置：`data/raw/`
- 样本范围：A股上市公司

## 数据覆盖范围

| 数据类别 | 年份范围 | 数据来源文件 |
|----------|----------|-------------|
| 治理变量（Top1, HHI5, 市值, 回报率等） | 2000-2023 | 常用变量查询（年度）.xlsx |
| 财务变量（总资产, 负债, 净利润, 现金流等） | 2011-2024 | 跨表查询_沪深京股票(年频).xlsx |
| 公司信息（行业, 上市日期等） | 最新 | STK_LISTEDCOINFOANL.xlsx |

**重要说明**：受 CSMAR 数据源本身限制，财务报表数据最早只到 2011 年。因此 2000-2010 年的杠杆率（Lev）、ROA、Cash 等财务指标为缺失状态，作业要求"2000年至今"为理想目标，实际分析基于数据可得性展开。

## 原始文件说明

| 文件名 | 主要内容 | 使用变量 |
|--------|---------|---------|
| 跨表查询_沪深京股票(年频).xlsx | 资产负债表、利润表财务数据 | total_asset, total_liability, revenue, net_profit, cash, equity 等 |
| 常用变量查询（年度）.xlsx | 公司治理、市值、回报率等 | Top1, HHI5, retwd, staff 等 |
| STK_LISTEDCOINFOANL.xlsx | 公司基本信息 | industry_code, listing_date 等 |

## 变量构造说明

| 变量 | 含义 | 计算方式 | 原始变量来源 |
|------|------|---------|------------|
| Lev | 总负债率 | 总负债 / 总资产 | total_liability / total_asset |
| SL | 流动负债率 | 流动负债 / 总资产 | current_liability / total_asset |
| LL | 长期负债率 | 非流动负债 / 总资产 | noncurrent_liability / total_asset |
| SDR | 短债比率 | 流动负债 / 总负债 | current_liability / total_liability |
| Cash | 现金比率 | 货币资金 / 总资产 | cash / total_asset |
| ROA | 总资产收益率 | 净利润 / 总资产 | net_profit / total_asset |
| ROE | 净资产收益率 | 净利润 / 所有者权益 | net_profit / equity |
| LLoan | 长期银行借款率 | 长期借款 / 总资产 | long_term_borrow / total_asset |
| Top1 | 第一大股东持股比例 | 直接取自 | Shrcr1 |
| HHI5 | 前五大股东持股集中度 | Σ(前5位股东持股比例平方) | Shrhfd5 |
| Size | 公司规模 | ln(总资产) | total_asset |
| Age | 上市年限 | 会计年度 - 上市年份 + 1 | listing_date |

## 数据清洗说明

- **样本筛选规则**：保留有数据年份的A股年度数据，剔除股票代码和会计年度缺失行
- **重复值处理**：按 code-year 删除重复行
- **缺失值处理**：保留原始缺失，后续分析时对每张图单独过滤有效值
- **离群值处理**：按年度在 1% 和 99% 分位数进行缩尾处理（Winsorize），适用于 Lev, SL, LL, SDR, Cash, ROA, ROE, LLoan, HHI5
- **合并规则**：以 code-year 为主键，outer join 合并财务数据和治理数据

## 存储方式

- 主要格式：Parquet（`data/combined/csmar_firm_year_panel.parquet`，17MB）
- 压缩备份：CSV.gz（`data/combined/csmar_firm_year_panel.csv.gz`，10MB）
- 选择 Parquet 的理由：文件体积小、读取速度快、支持列式读取节省内存，适合大数据分析

## 分析内容

1. 年度描述统计（Lev, SL, LL, SDR, Cash, ROA, ROE, LLoan, Top1, HHI5, Size, Age）
2. 图1：Lev 均值与中位数时序图（2011-2024，财务数据实际覆盖期）
3. 图2：ROA 与 Cash 均值时序图（2011-2024）
4. 图3：各行业年度平均负债率（算术平均）
5. 图4：各行业年度平均负债率（加权平均，按总资产）
6. 图5：Top1 年度箱线图（2001/2003/2005/2007/2009/2011/2013/2015/2017/2019/2021/2023）

## GitHub 仓库

https://github.com/gaosiyuan1995/dshw-p02b

## 如何运行

1. 安装依赖：`pip install -r requirements.txt`
2. 将 CSMAR 原始压缩包放入 `data/data_raw_zip/`
3. 运行 `01_extract_raw_data.ipynb` 解压并识别原始数据
4. 运行 `02_clean_construct_variables.ipynb` 清洗数据并构造变量
5. 运行 `03_analysis.ipynb` 生成表格、图形和分析结果
6. 打开 `report.html` 阅读完整报告

## 变量字典

完整变量映射见 `data/dict/variable_dictionary.csv`，共 32 个变量，来源覆盖 3 个原始数据文件。
