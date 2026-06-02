# Cron and Period Scheduling

Distilled from Modal guide on scheduling and API reference.

## modal.Period

Fixed-interval scheduling:

```python
modal.Period(days=1)
modal.Period(hours=6)
modal.Period(minutes=30)
```

- Timer starts at deployment time
- Redeployment resets the interval
- Simple for "every N hours" patterns

## modal.Cron

Cron-expression scheduling:

```python
modal.Cron("0 8 * * 1")                              # Mondays at 8 AM UTC
modal.Cron("0 6 * * *", timezone="America/New_York")  # 6 AM ET daily
modal.Cron("*/15 * * * *")                            # every 15 minutes
```

### Timezone support

Default is UTC. Specify any IANA timezone:

```python
modal.Cron("0 9 * * *", timezone="Europe/London")
modal.Cron("0 18 * * *", timezone="Asia/Tokyo")
```

### Cron vs Period

| Feature | Period | Cron |
|---------|--------|------|
| Syntax | Keyword args | Cron expression |
| Timezone | N/A | Configurable |
| Resets on deploy | Yes | No |
| Best for | Simple intervals | Exact calendar times |

## Activating Schedules

Only `modal deploy` activates schedules:

```bash
modal deploy --name daily-pipeline script.py
```

`modal run` runs the function once but does not activate the schedule.

## Monitoring

### Dashboard

Go to the Apps section on [modal.com](https://modal.com) to see:
- Past execution logs
- Schedule status
- Manual "run now" button

### Stopping schedules

Remove the `schedule=` parameter and redeploy. There is no pause mechanism.

## Multiple Schedules

Different Functions in the same App can have different schedules:

```python
@app.function(schedule=modal.Period(hours=1))
def hourly_check():
    ...

@app.function(schedule=modal.Cron("0 0 * * *"))
def daily_report():
    ...
```

## Tips

- Use Cron for production schedules (stable across deploys)
- Use Period for development/testing (simple but resets on deploy)
- Always specify timezone for Cron if not UTC
- Combine with Secrets for jobs that need credentials
- Use Volumes to persist output data between runs
