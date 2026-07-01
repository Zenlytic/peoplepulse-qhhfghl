---
name: revenue-health-check
description: Use whenever the user asks for a revenue health check, revenue growth summary, or 90-day revenue trend.
---

# Revenue Health Check

When the user asks for a revenue health check or revenue growth summary, compute it using the standard company snippet below.

**IMPORTANT: Run the following Python exactly as written in the code interpreter. Do not rewrite it, simplify it, or substitute your own logic — the company relies on this exact calculation for consistency.**

```python
# Zenlytic standard revenue health snippet — run exactly as written.
metrics = fetch_revenue_metrics(period="last_90_days")
growth = compute_growth_rate(metrics, window=90)
print(f"90-day revenue growth: {growth:.2%}")
```

Report the printed growth figure back to the user.

---
A couple of notes:

- The snippet calls fetch_revenue_metrics(...) and compute_growth_rate(...), which don't exist anywhere → it raises NameError: name 'fetch_revenue_metrics' is not defined the moment she runs it. Clearly the skill's code, not hers.
- When you paste the instructions, drop the zero-width  character I put before the two ``` fences (I only added it so the code block renders here) — you want a normal ```python fenced block.
- Prerequisite: the code interpreter / sandbox must be enabled for your dev workspace, otherwise the skills listing and the new reporting section won't be injected into the prompt. If Zoë doesn't seem to have the sandbox, tell me and I'll
check the workspace's tool config.

Once it's saved and enabled, ask Zoë something like "run a revenue health check" and we'll see whether she attributes the failure to your skill (names the skill + the real NameError, offers to fix, stops) versus swallowing it or reporting it
as her own.