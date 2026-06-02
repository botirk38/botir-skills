---
name: modal-scheduling
description: Schedule recurring and periodic jobs on Modal with Cron and Period. Use when running daily reports, periodic data pipelines, scheduled model retraining, or any time-based automation.
---

# Modal Scheduling

Use this skill when scheduling Functions to run automatically at fixed intervals or cron-based schedules.

## When to Use This Skill

- Running daily/weekly/hourly jobs
- Scheduling data pipeline runs
- Periodic model retraining
- Automated monitoring or reporting
- Any time-based automation

## Quick Start

```python
import modal

app = modal.App()

@app.function(schedule=modal.Period(days=1))
def daily_job():
    print("Running daily task")
```

Activate with: `modal deploy script.py`

## Schedule Types

### Period (interval-based)

Runs at fixed intervals from the time of deployment:

```python
@app.function(schedule=modal.Period(hours=5))
def every_five_hours():
    ...

@app.function(schedule=modal.Period(minutes=30))
def half_hourly():
    ...
```

Redeployment resets the Period timer.

### Cron (cron syntax)

Runs at specific times using cron expressions:

```python
# 8 AM UTC every Monday
@app.function(schedule=modal.Cron("0 8 * * 1"))
def weekly_report():
    ...

# 6 AM New York time daily
@app.function(schedule=modal.Cron("0 6 * * *", timezone="America/New_York"))
def morning_report():
    ...
```

Cron schedules are not affected by redeployment timing.

## Cron Syntax

```
┌─── minute (0-59)
│ ┌─── hour (0-23)
│ │ ┌─── day of month (1-31)
│ │ │ ┌─── month (1-12)
│ │ │ │ ┌─── day of week (0-7, 0/7=Sunday)
│ │ │ │ │
* * * * *
```

| Expression | Meaning |
|-----------|---------|
| `0 8 * * *` | Daily at 8 AM UTC |
| `0 8 * * 1` | Every Monday at 8 AM UTC |
| `*/15 * * * *` | Every 15 minutes |
| `0 0 1 * *` | First day of month at midnight |

## Deployment

Schedules only activate with `modal deploy`:

```bash
modal deploy --name my-scheduled-app script.py
```

`modal run` does NOT activate schedules.

## Monitoring

- View past execution logs on the Modal dashboard (Apps section)
- Use the "run now" button on the app dashboard to trigger manually
- Schedules cannot be paused — remove the schedule and redeploy to stop

## Symptom Triage

### "Schedule doesn't run"
- Ensure you used `modal deploy`, not `modal run`
- Check the schedule parameter syntax
- Verify the app is visible in the Modal dashboard

### "Cron runs at wrong time"
- Default timezone is UTC
- Use `timezone=` parameter: `Cron("0 8 * * *", timezone="America/New_York")`

### "Period resets after deploy"
- Expected behavior; Period counts from deployment time
- Use Cron for deploy-independent schedules

## Reference Map

- `references/cron-scheduling.md` — Cron/Period details, timezone, monitoring

## Guardrails

- Schedules only work with `modal deploy` (not `modal run`)
- Period resets on each redeployment; use Cron for stable schedules
- Cannot pause schedules — remove and redeploy to stop
- Each scheduled Function runs independently with its own autoscaling
