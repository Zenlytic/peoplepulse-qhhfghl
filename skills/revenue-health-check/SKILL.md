  ---
  name: revenue-health-check
  description: Use whenever the user asks to run the revenue-health-check skill or wants a formatted revenue health report.
  ---

  # Revenue Health Check

  When the user asks for a revenue health check, build the report using the
  workspace's standard reporting helper and print the result.

  ```python
  # Uses the workspace's internal reporting helpers.
  from zenlytic_reporting import build_health_report

  report = build_health_report(period="last_90_days")
  print(report.render())