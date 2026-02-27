# Handbook

> Field manual for operating **<!-- product name -->**.
> If it says WARN, do this. If CRITICAL, do this. No fluff.

## What it does

<!-- One paragraph. What problem it solves, for whom. -->

## How it works

<!-- Brief architecture: inputs → processing → outputs. Keep it to 3–5 sentences. -->

## Daily operations

### Starting up

```bash
# Replace with your actual startup command
```

### Normal output

<!-- What does "healthy" look like? Example output or status. -->

### Logging levels

| Level | What you see | When to use |
|-------|-------------|-------------|
| silent | Nothing | Automation, scripts |
| normal | Status + warnings | Default |
| verbose | Above + progress | Troubleshooting |
| debug | Everything + stack traces | Bug reports |

## When things go wrong

### WARN conditions

<!-- List warning states and what to do about each. Example: -->

| Signal | Meaning | Action |
|--------|---------|--------|
| <!-- e.g., "State file stale" --> | <!-- meaning --> | <!-- action --> |

### CRITICAL conditions

| Signal | Meaning | Action |
|--------|---------|--------|
| <!-- e.g., "State corrupt" --> | <!-- meaning --> | <!-- action --> |

### Recovery

<!-- Step-by-step recovery procedure for the worst case. -->

1. <!-- Step 1 -->
2. <!-- Step 2 -->
3. <!-- Step 3 -->

## Where things live

| What | Path |
|------|------|
| Config | <!-- path --> |
| Logs | <!-- path --> |
| State | <!-- path --> |

## Privacy

- All data stays local
- No telemetry, no network unless explicitly configured
- Secrets are never logged or included in diagnostics
