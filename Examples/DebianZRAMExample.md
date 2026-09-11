# [Template] ZRAM on Debian

## Summary

Learning to use ZRAM to make RAM usage more efficient on a low-RAM laptop.

## Background

Laptop has 2GB RAM, so it swaps frequently.

## Experiment

### Attempt 1

Command:

```bash
sudo swapon /dev/zram0
```

Result:

Success.

## Problem

Swap priority did not behave as expected.

## Solution

Changed priority to 100.

## Conclusion

ZRAM is more effective than relying on HDD swap alone.
