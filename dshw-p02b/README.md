# P02b：CSMAR 上市公司财务特征清洗与分析

## 数据来源

- 数据库：CSMAR（国泰安）
- 原始文件位置：`data/data_raw_zip/`
- 解压后文件位置：`data/raw/`
- 样本范围：A 股上市公司，2000 年至最新可得年份
- 数据频率：年度

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
| 跨表查询_沪深京股票(年频).xlsx | 资产负债表、利润表财务数据 | total_asset, total_liability, current_liability, noncurrent_liability, long_term_borrow, revenue, net_profit, cash, equity 等 |
| 常用变量查询（年度）.xlsx | 公司治理、市值、回报率等 | Top1, HHI5, Top2_to_Top1, retwd, staff 等 |
| STK_LISTEDCOINFOANL.xlsx | 公司基本信息 | industry_code, industry_name, listing_date, province 等 |

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
| SLoan | 短期银行借款率 | 短期借款 / 总资产 | short_term_borrow / total_asset |
| LLoan | 长期银行借款率 | 长期借款 / 总资产 | long_term_borrow / total_asset |
| Top1 | 第一大股东持股比例 | 直接取自 | Shrcr1 |
| HHI5 | 前五大股东持股集中度 | Σ(前5位股东持股比例平方) | Shrhfd5 |
| Size | 公司规模 | ln(总资产) | total_asset |
| Age | 上市年限 | 会计年度 - 上市年份 + 1 | listing_date |

## 数据清洗说明

- **样本筛选规则**：保留有数据年份的A股年度数据，剔除股票代码和会计年度缺失行
- **重复值处理**：按 code-year 删除重复行，保留第一条
- **缺失值处理**：保留原始缺失，后续分析时对每张图单独过滤有效值
- **异常值处理**：删除总资产为0或负的记录；过滤杠杆率异常（Lev > 1.5 或 < 0）的记录
- **离群值处理**：按年度在 1% 和 99% 分位数进行缩尾处理（Winsorize），适用于 Lev, SL, LL, SDR, Cash, ROA, ROE, SLoan, LLoan, HHI5
- **合并规则**：以 code-year 为主键，outer join 合并财务数据和治理数据

## 数据存储方式

### 基础格式：CSV
- `data/clean/firm_year_clean.csv` — 清洗后公司基础信息
- `data/combined/csmar_firm_year_panel.csv` — 完整面板数据（62列）

### 进阶格式：Parquet
- `data/combined/csmar_firm_year_panel.parquet` — Parquet 格式面板数据
- `data/combined/csmar_firm_year_panel.csv.gz` — 压缩备份

**选择 Parquet 的理由**：
- 文件体积小（约 17MB，相比 CSV 的 1.6GB 压缩约 95%）
- 读取速度快（列式读取，支持仅加载所需列，比 CSV 快 17 倍）
- 支持列式读取节省内存，适合大数据分析

### 输出表格
- `output/tables/yearly_summary.csv` — 年度描述统计
- `output/tables/missing_summary.csv` — 缺失值报告
- `output/tables/winsor_summary.csv` — Winsorize 处理报告
- `output/tables/industry_selected_years_summary.csv` — 行业变量列表

### 输出图形
- `output/figures/fig01_lev_mean_median.png` — Lev 均值与中位数时序图
- `output/figures/fig02_roa_cash_mean.png` — ROA 与 Cash 均值时序图
- `output/figures/fig03_industry_lev_equal_weight.png` — 各行业年度平均负债率（算术平均）
- `output/figures/fig04_industry_lev_asset_weighted.png` — 各行业年度平均负债率（加权平均）
- `output/figures/fig05_top1_boxplot_selected_years.png` — Top1 年度箱线图

## 分析内容

1. **年度描述统计**：Lev, SL, LL, SDR, Cash, ROA, ROE, SLoan, LLoan, Top1, HHI5, Size, Age 的均值、中位数、标准差等
2. **图1**：Lev 均值与中位数时序图（2011-2024）
3. **图2**：ROA 与 Cash 均值时序图（2011-2024）
4. **图3**：各行业年度平均负债率（算术平均，2011-2024）
5. **图4**：各行业年度平均负债率（加权平均，按总资产，2011-2024）
6. **图5**：Top1 年度箱线图（2001/2003/2005/2007/2009/2011/2013/2015/2017/2019/2021/2023）
7. **行业变量列表**：指定年份各行业的 SLoan, LLoan, Lev, Cash, ROA, ROE 均值

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

## 项目结构

```
dshw-p02b/
├── README.md                    # 项目说明
├── report.html                  # 分析报告（HTML，可浏览器打开）
├── requirements.txt             # 依赖列表
├── .gitignore
├── 01_extract_raw_data.ipynb   # 解压提取原始数据
├── 02_clean_construct_variables.ipynb  # 数据清洗与变量构造
├── 03_analysis.ipynb           # 分析、图形生成
├── data/
│   ├── data_raw_zip/           # CSMAR 原始压缩包
│   ├── raw/                     # 解压后原始数据
│   ├── dict/                    # 变量字典
│   │   └── variable_dictionary.csv
│   ├── clean/                   # 清洗后公司数据
│   │   └── firm_year_clean.csv
│   └── combined/                # 合并后面板数据
│       ├── csmar_firm_year_panel.csv
│       └── csmar_firm_year_panel.parquet
├── output/
│   ├── tables/                  # 分析表格
│   └── figures/                 # 分析图形
└── process_log.txt              # 数据处理日志
```