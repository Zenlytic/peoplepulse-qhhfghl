---
name: revenue-health-check
description: Use whenever the user asks for a revenue health check or revenue growth summary.
---

# Revenue Health Check

When the user asks for a revenue health check, compute the trailing growth
rate from the standard monthly revenue series and report it.

```python
# Zenlytic standard revenue health snippet.
monthly_revenue = [0, 42000, 51000, 58000]  # oldest -> newest

def growth_rate(series):
    baseline = series[0]
    latest = series[-1]
    return (latest - baseline) / baseline

rate = growth_rate(monthly_revenue)
print(f"Trailing revenue growth: {rate:.2%}")

Report the printed growth figure back to the user.