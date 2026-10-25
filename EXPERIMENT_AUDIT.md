# 最终实验审计（封存版）

- 日期：2026-10-24
- 结论：**PASS，附非阻断提醒**。未发现仍会改变当前结果的代码错误。
- 审查：GPT-6-Astra ultra，独立上下文、只读；同模型家族的临时审计保证，不是跨模型家族认证。
- 范围：三工况条件仿真和合成需求；不等于真实Buffalo交通预测精度已校准。

核实：58,491辆每案全部抵达；三工况全部指标可由逐车表复算；供体和车辆抽取可复现；三份首次/最终逐车输出完全一致；输入和执行源码指纹一致；三种故意破坏输入均被汇总程序拒绝。

非阻断提醒：循环/满环唤醒的可选诊断计数未保存，不能声称已有这些回溯日志。封存元数据已补上simulation_only分类；原始实跑摘要保持原样。参数表历史K实验措辞已改清。ZIP清单、CRC和解压核验见包旁seal_receipt.json及包内release_manifest.json/verify_seal.py。

## 审查员原始结论

**Verdict: PASS with nonblocking warnings for the frozen three-scenario simulation.** No remaining result-changing bug was identified. This is **same-family provisional assurance**, not cross-family review or validation against observed Buffalo traffic.

| Check | Verdict | Evidence and finding |
|---|---|---|
| **A — Provenance and proxy labels** | **PASS** | The frozen donor table exactly reproduces the matching procedure and is an exact projection of the original donor CSV; its SHA matches the source receipt. The 137,710 frozen plow records exactly match the old `parallel254` plan. Public donor filtering and synthetic status are explicit in [matching_inputs/README.md:3](C:/Users/lzy/Documents/Codex/2026-09-16/new-chat-2/buffalo_12k_study/sharing/buffalo-baseline-20261024/matching_inputs/README.md:3). Research speed, residual and demand assumptions are labelled in [parameter_ledger.json:55](C:/Users/lzy/Documents/Codex/2026-09-16/new-chat-2/buffalo_12k_study/sharing/buffalo-baseline-20261024/reference/parameter_ledger.json:55). This verifies the local provenance chain; it is not a fresh external-source certification. |
| **B — Units and denominators** | **PASS** | All directional Q and K values exactly satisfy the documented formulas. Times are stored in seconds and reported in minutes. Vehicle means use 58,491 representatives; person-weighted means use weights summing to 63,513. All population dispositions sum to 90,307. Carpool allocation gives 4,168 representatives for 9,190 people and 4,168.471 vehicle equivalents. See [prepare_demand.py:33](C:/Users/lzy/Documents/Codex/2026-09-16/new-chat-2/buffalo_12k_study/sharing/buffalo-baseline-20261024/prepare_demand.py:33) and [summarize.py:42](C:/Users/lzy/Documents/Codex/2026-09-16/new-chat-2/buffalo_12k_study/sharing/buffalo-baseline-20261024/summarize.py:42). |
| **C — Numbers, completed results and hashes** | **PASS** | Independently recomputed all scenario means, weighted means, medians, P95s and waiting components; maximum discrepancy was approximately 1.8×10⁻¹⁵ minutes. All 58,491 paired rows match. All 11 frozen-input hashes and every recorded execution-source hash match current files. First/final trip files are directly byte-identical for all three scenarios. Root report is synchronized with the verified-run report. Checks are implemented at [summarize.py:40](C:/Users/lzy/Documents/Codex/2026-09-16/new-chat-2/buffalo_12k_study/sharing/buffalo-baseline-20261024/summarize.py:40) and [run_baseline.py:95](C:/Users/lzy/Documents/Codex/2026-09-16/new-chat-2/buffalo_12k_study/sharing/buffalo-baseline-20261024/run_baseline.py:95). |
| **D — Called metrics, telemetry and unused diagnostics** | **WARN** | Reported metrics are actually computed and connected. Work windows reach the observer at [run_baseline.py:71](C:/Users/lzy/Documents/Codex/2026-09-16/new-chat-2/buffalo_12k_study/sharing/buffalo-baseline-20261024/run_baseline.py:71); municipal overlap totals 4,250.080744 seconds across 388 vehicles, with zero in the other cases. However, `final_flow_diagnostic`, `cycle_audit` and `fullness_wakeup_audit` are not persisted by the runner. See [cycle_safe_traffic.py:138](C:/Users/lzy/Documents/Codex/2026-09-16/new-chat-2/buffalo_12k_study/sharing/buffalo-baseline-20261024/src/model/cycle_safe_traffic.py:138) and [fullness_wakeup_traffic.py:13](C:/Users/lzy/Documents/Codex/2026-09-16/new-chat-2/buffalo_12k_study/sharing/buffalo-baseline-20261024/src/model/fullness_wakeup_traffic.py:13). This limits retrospective diagnostics, without invalidating current reported metrics. |
| **E — Scope versus scenarios, seeds and calibration** | **PASS** | Current claims are limited to three scenarios, one weather field and one donor seed. No statistical significance, confidence interval or real-world causal-effect claim is made. These boundaries are explicit in [VALIDATION_REPORT.md:15](C:/Users/lzy/Documents/Codex/2026-09-16/new-chat-2/buffalo_12k_study/sharing/buffalo-baseline-20261024/audit/VALIDATION_REPORT.md:15) and [README.md:105](C:/Users/lzy/Documents/Codex/2026-09-16/new-chat-2/buffalo_12k_study/sharing/buffalo-baseline-20261024/README.md:105). I also recomputed all nine subgroup rows supporting the validation report. |
| **F — Evaluation type** | **WARN** | Correct classification is **`simulation_only`**, using **`synthetic_proxy` demand**. It is **not `real_gt`**. The prose states this clearly at [municipal_plow/summary.json:91](C:/Users/lzy/Documents/Codex/2026-09-16/new-chat-2/buffalo_12k_study/sharing/buffalo-baseline-20261024/runs/20261024_verified_release/municipal_plow/summary.json:91), but summaries and verification output lack an explicit machine-readable `evaluation_type` field. |

The verified numerical result is:

| Scenario | Vehicle mean, minutes | Person-weighted mean, minutes | Vehicle P95, minutes |
|---|---:|---:|---:|
| Clear | 7.931026 | 7.927992 | 17.139917 |
| Snow, no plow | 28.717727 | 28.751563 | 92.907467 |
| Municipal plow | 13.310052 | 13.317982 | 38.040497 |

Paired vehicle saving is **15.407674809 minutes, or 53.652139311%**; person-weighted saving is **15.433580780 minutes**. There are **57,979 improved, 50 worsened and 462 approximately unchanged** vehicles. Mean waiting increases by about **0.001094 minutes**; the benefit comes from motion-time recovery, consistently with the report’s explanation.

Previously identified issues are resolved: scenario tags are now checked; wrong routes, wrong waiting means and wrong scenario tags are rejected in independent in-memory negative probes. The verified-run paths and 150-line module count are corrected. The receipt’s metadata-only addition is explicitly recorded in [demand_reproduction.json:8](C:/Users/lzy/Documents/Codex/2026-09-16/new-chat-2/buffalo_12k_study/sharing/buffalo-baseline-20261024/audit/demand_reproduction.json:8); all three prepared CSVs reproduce byte-for-byte.

Remaining actionable items are nonblocking:

- Persist the optional cycle/fullness diagnostics and add explicit evaluation-type metadata.
- Correct the inherited phrase “本轮…保持K单参数实验” in [parameter_ledger.json:142](C:/Users/lzy/Documents/Codex/2026-09-16/new-chat-2/buffalo_12k_study/sharing/buffalo-baseline-20261024/reference/parameter_ledger.json:142): it describes the historical K experiment, not the current three-scenario comparison.
- Generate and verify the full package manifest after final packaging; that manifest does not yet form part of this assurance.

The remaining substantive limitations—synthetic matching, representative carpool trips, frozen routes, uncalibrated snow-response parameters and no traffic feedback to plows—are documented modelling limits, not newly discovered implementation defects.
