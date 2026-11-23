# Contributing

Contributions are welcome, particularly ones that close the deployment gaps listed in the
README under **What this repo does not do yet**. Those are, in rough priority order:

1. Patient-grouped and temporal validation splits
2. A pretreatment-only risk model (excluding `SMS_received` and other post-treatment variables)
3. Probability calibration (reliability curves, Brier score) and utility-based thresholds
4. AUPRC alongside ROC AUC
5. Causal diagnostics: propensity overlap, covariate balance, doubly robust cross-fitting
6. Budget-constrained policy evaluation against treat-all / risk-based / random baselines
7. Bootstrap confidence intervals at the patient level
8. Subgroup fairness analysis
9. Inductive graph construction for the GCN

## Development setup

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
pip install -e .
pytest
```

## Before opening a pull request

- `black src tests` and `ruff check src tests` pass
- `pytest` passes
- New metrics are computed from confusion matrices, not from cached scalars
- Any claim about a *causal* effect is accompanied by the diagnostic that supports it

## Reporting issues

Please include the config file used, the stage that failed, package versions
(`pip freeze`), and whether the GCN dependencies were installed.
