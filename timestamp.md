# Date and Time Calculation

The next 5 formulas are optional in case the need arises. Here are the complete and standalone breakdown of all units of time.

## Years

```bash
floor({timestamp}/31556926)
```

## Months

```bash
floor({timestamp}/2629743)
```

## Days

```bash
floor({timestamp}/86400)
```

## Hours

```bash
floor({timestamp}/3600)
```

## Minutes

```bash
floor({timestamp}/60)
```