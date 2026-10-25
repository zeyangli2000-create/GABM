# Buffalo snow commute baseline

这是布法罗正常通勤假设下的清雪研究 baseline，封存版本为 **20261024-v1.0**。比较同一组汽车通勤需求在无雪、有雪不清扫、有雪按既有规则清扫三种工况中的通勤时间。**AI 调度尚未接入。**

## 先下载哪个文件

仓库首页保存说明、运行指南和审计；**全部源代码、复跑所需最小输入、完成结果和离线报告，放在 Releases 的完整 ZIP 中**。

1. 打开 [封存版本发布页](https://github.com/zeyangli2000-create/GABM/releases/tag/20261024-v1.0)。此链接在该版本发布后生效。
2. 下载名为 **Buffalo_Baseline_20261024-v1.0_SEALED.zip** 的附件。
3. 同时下载同名 `.sha256` 文件用于完整性核对。
4. 解压后，进入 `buffalo-baseline-20261024` 文件夹，先打开 `report/index.html` 看结果，再按包内 `README.md` 运行。

GitHub 自动生成的 `Source code (zip)` 只包含当前仓库首页材料，不是完整实验包。不要把它与上面的封存 ZIP 混淆。

## 研究人口与车辆

| 项目 | 数量 |
| --- | ---: |
| 登记合成人员 | 90,307 |
| 有合法市区路线 | 90,074 |
| ACS 相似供体匹配 | 89,855 |
| 合成汽车通勤者 | 63,513 |
| 参与交通仿真的车辆代表 | 58,491 |
| 通勤方式未知 | 219 |
| 路线未知 | 233 |

方式由公开 2022 ACS 纽约州 PUMS 的 Erie 成年在业供体统计匹配而来。匹配原始共同字段为年龄、性别、居住 PUMA、家庭或集体居住范围；固定随机数种子为 20261024。它不是对真实个人日常习惯的识别。

汽车人数不等于车辆数。拼车者按乘员数换算车辆当量，再固定抽取车辆代表；未选中的乘员通过人员权重计入通勤时间。未知人员保留在去向表，没有冒称不出行或已经抵达。

## 封存结果

| 工况 | 车辆平均市区片段时间 分钟 | 人员加权平均 分钟 | 抵达车辆 |
| --- | ---: | ---: | ---: |
| 无雪 | 7.931 | 7.928 | 58,491 |
| 有雪不清扫 | 28.718 | 28.752 | 58,491 |
| 按既有规则清扫 | 13.310 | 13.318 | 58,491 |

同车比较，清扫相对不清扫平均减少约 15.408 分钟。**这是当前模型里的清扫收益，不是 AI 收益，也不是 Buffalo 实测效益。** 当前改善主要来自行驶速度恢复，平均排队等待没有明显减少。

气象有效期是 2022-12-24 01:00 至 12-25 01:00 EST。采用正常通勤反事实需求，不重现节日真实出行或禁行。比较的是已冻结市区通勤片段，不能直接当成全城门到门平均通勤时间。

## Windows 与 VS Code 运行

先安装 Python 3.13 和 VS Code 的 Python 扩展。把完整 ZIP 解压到一个普通文件夹，VS Code 选择“文件 → 打开文件夹”，打开含 `run_baseline.py` 的 `buffalo-baseline-20261024`。

在该文件夹终端执行：

```powershell
py -3.13 verify_seal.py
py -3.13 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe run_baseline.py --check
.\.venv\Scripts\python.exe tests/contract_probe.py
.\.venv\Scripts\python.exe run_baseline.py --case all --out runs/my_run
.\.venv\Scripts\python.exe summarize.py --run runs/my_run
```

第一步应显示 `PASS`。完成后打开 `runs/my_run/report/index.html`。再次运行换一个新输出目录，例如 `runs/my_run2`，程序拒绝覆盖非空结果。

报告可离线查看，不需要启动 8525 服务。默认重放冻结的清扫记录；重新计算派工可使用 `--case municipal_plow --replan`，详见 [完整运行指南](BASELINE_RUN_GUIDE.md)。

## 完整包里包含什么

```text
buffalo-baseline-20261024/
  README.md                   原始运行说明
  run_baseline.py              三工况入口
  match_commute_modes.py       通勤方式匹配
  prepare_demand.py            人员换算车辆代表
  summarize.py                结果汇总
  requirements.txt            固定依赖
  verify_seal.py               文件完整性检查
  src/model/                  17个模型依赖模块
  snapshot/                   冻结道路 天气和派工等输入
  matching_inputs/            最小人口特征与公开供体
  prepared/                   固定车辆需求与人口去向
  runs/20261024_verified_release/ 三工况完成结果
  report/index.html           完成结果与机制的离线报告
  reference/                  参数依据与历史版本标记
  tests/                      机制检查
  audit/                      回归与解释核查
```

本首页不是源代码目录。直接下载本仓库而不下载 Release 附件，不会得到完整运行输入。

## 模型做了什么

同一批人使用相同路线、出发时间和权重。逐路已有雪深与后续新雪更新道路状态。清扫发生在实际车道覆盖带上，清扫后用残雪重算速度，后续落雪可触发重复任务。容量表示单位时间允许通过的车辆数，储车量表示路段里最多容纳的车辆数。

速度、残雪、并行作业、起扫阈值等参数中有明确研究设定；市政文件提供优先框架与服务组织依据，并未直接规定全部数值和精确派工时序。当前交通队列不会反过来阻塞铲车。参数和机制见包内 `reference/parameter_ledger.json` 以及 `report/index.html`。

## 版本与完整性

- 封存版本：`20261024-v1.0`。
- 完整 ZIP：45,425,958 bytes，约 43.32 MiB。
- 包内文件：107 个。
- SHA256：`f269d094f4de5861225df8457a5f1712cd2dc8c6674586049f30bb9fd736b7aa`。
- 下载后可以在 PowerShell 用 `Get-FileHash -Algorithm SHA256 "完整ZIP的路径"` 对照。
- 后续改动另起版本，不覆盖这个封存包。源码与结果的适用范围见 [审计说明](EXPERIMENT_AUDIT.md)。

## 来源与使用范围

- 原交通项目：[njiang8/Traffic_Dynamics](https://github.com/njiang8/Traffic_Dynamics)。
- 合成人口方法与入口：[njiang8/geo-synthetic-pop-usa](https://github.com/njiang8/geo-synthetic-pop-usa)。
- 原始供体来自公开 ACS 2022 PUMS；本包仅保留最小匹配字段。
- 政策、数据与参数来源详见 [UPSTREAM_AND_DATA.md](UPSTREAM_AND_DATA.md)。

当前没有给整个衍生项目另选 MIT 或其他统一开源许可证；请保留各上游的来源与使用条件。此次协作按现有私有仓库组织，详见 [LICENSING.md](LICENSING.md)。包内不含学校登录信息或私人 SharePoint 家庭收入文件。
