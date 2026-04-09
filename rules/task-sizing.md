# Task Sizing

Don't run the full pipeline for every task. Skip stages based on task size.

## Size Guide

### S (1-2 files) — One-line bug fix, small changes
```
/debug → /{framework}-04-fix → /ship
```
- Skip explore, plan
- Small changes to code you already know

### M (3-10 files) — General feature development
```
/explore → /{framework}-01-plan → 02-implement → /simplify → 03-review → /ship
```
- Standard pipeline
- Skip fix if no issues after review

### L (10+ files) — Large-scale features, refactoring
```
/explore → /plan-ceo-review → /{framework}-01-plan → 02-implement (--team) → 03-review (--team) → 04-fix → /qa → /ship
```
- Full pipeline + --team mode
- CEO review validates scope
- Include QA phase

## Anti-Over-Engineering
- Applying large processes to small tasks is actually inefficient
- Pipeline is guideline, not mandate
- Decision criterion: "What do I miss if I skip this step?"
