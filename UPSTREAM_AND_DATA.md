# 来源与许可记录

- 原交通项目：https://github.com/njiang8/Traffic_Dynamics
- 合成人口方法与下载入口：https://github.com/njiang8/geo-synthetic-pop-usa
- Buffalo2022–2023计划：https://www.buffalony.gov/DocumentCenter/View/11064
- Buffalo2024–2025计划：https://www.buffalony.gov/DocumentCenter/View/13734/CitySnowPlan-2024-25
- 本版本保留17个模型依赖模块；删除无调用旧入口和辅助函数，增加匹配与三场景入口。snapshot/manifest.json保存输入哈希，每案summary.json记录实际执行源码哈希。
- 输入包是已公开合成人口筛出的研究代理、逐路天气与计划快照，不是个人GPS，也不包含校园账号。
- 当前导出没有给整个衍生项目添加新的开源许可证。公开发布前应保留并核对各上游代码/数据的许可；需要先协作时可创建私有仓库。
- 原始纽约州库约3.4GB、原始气象栅格和此前全部探索实验不在此包；本包复跑的是已冻结的合法路线与天气输入。生成这些输入的历史脚本保留在原研究项目。

- 20261020：储车单位25ft=7.62m，FHWA-HRT-06-108第4章；1900为理想饱和流，2000为高速工程代理。reference/parameter_ledger.json列明研究假设。
- 原src/model/lane_capacity.py默认8m仅用于历史生成；本包运行入口读取已冻结的7.62m容量CSV，不调用旧默认重建K。
- Census ACS2022 B08015/B08141/B25044原值、误差和官方来源已保留；旧版需求为90,074辆一人一车代理；本版经供体方式匹配与拼车换算，为63,513名汽车通勤者、58,491辆车辆代表。
- NYSDOT2012最大允许积雪是作业目标，不是起扫阈值；Belvidere新增3in复扫为外地机制参考。本包阈值仍为研究对照2.54/5.08cm。

- 20261024：当前结果为runs/20261024_verified_release，旧版完成记录明确归档于reference/historical_20261020_NOT_CURRENT_RESULTS，不能用于验证本轮。
- 供体：公开ACS2022纽约州PUMS的Erie4,304人最小字段。matching_inputs不含私人SharePoint家庭收入，车辆换算依据ACS2022定义印刷页102；具体代表抽样是研究近似。
