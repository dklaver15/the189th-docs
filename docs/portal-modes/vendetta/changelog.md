# Changelog

Notable changes to Vendetta. Add a new section at the top whenever a build is shipped.

## v0.3.0 _(current)_

- Full state machine: Lobby → RoundStart → RoundActive → RoundEnd
- HVT ping system with `WorldIcon` refresh interval
- HVT HP buff applied/cleared per round
- Custom 6-column scoreboard
- Anti-streak HVT selection (previous HVT excluded next round)

### Known issues

- Splash screen indefinite display under some conditions — verify whether reproducible at v0.3.0.

## v0.2.x

_TODO — fill in earlier history. Suggested format:_

- Feature added / removed / changed
- Bug fixed
- Tuning adjustment (with old → new value)

## v0.1.0

_TODO — first playable build._

---

## Template for new entries

```markdown
## vX.Y.Z _(YYYY-MM-DD)_

### Added
- ...

### Changed
- ...

### Fixed
- ...

### Tuning
- HVT HP multiplier: 1.5 → 1.75
- Ping interval: 5s → 8s
```
