# hatch-me

Hatch yourself as a Codex baby from your date of birth. The lunar phase the
day you were born is computed from four independent sources and cross-checked
against NASA JPL Horizons before being baked into the seed.

```bash
cp -R skills/hatch-me ${CODEX_HOME:-$HOME/.codex}/skills/
python3 ${CODEX_HOME:-$HOME/.codex}/skills/hatch-me/scripts/hatch.py --dob 1995-08-14T03:20 --tz -07:00
```

```
                              ┌──────────────────────────────┐
                              │ sources (each independent)   │
                              │                              │
                       ┌──────┤   meeus    local AA2 math    │
                       │      │   codex    LLM ask (opt-in)  │
                       │      │   claude   LLM ask (opt-in)  │
                       │      └──────────────────────────────┘
                       │
DOB ─┐                 ▼
     │      illum%, phase_name      ┌──────────────────────────────────┐
     │  ┌────────────────────────┐  │ ground truth                     │
     ├──┤                         ├─▶│   jpl    NASA JPL Horizons HTTP │
     │  └────────────────────────┘  │          quantities 10 + 24      │
     │                              └──────────────┬───────────────────┘
     │                                             ▼
     │                              illum Δ ≤ 1.0% ?
     │                              angle Δ ≤ 0.5° ?
     │                                             │
     │                                             ▼
     │                                ┌──────────────────────────┐
     └───────────────────────────────▶│ scripts/hatch.py          │
                                      │ BLAKE2b(salt|dob|moon)    │
                                      │ → SplitMix64 → Soul       │
                                      └────────────┬─────────────┘
                                                   ▼
                              ~/.codex/memories/babies/<name>/baby.json
                                       + ASCII birth card on stdout
```

## Sources

| name | what it is | offline | speed | notes |
|---|---|:---:|:---:|---|
| `meeus` | local Python implementing Meeus *Astronomical Algorithms* (2nd ed., 1998) Ch. 47/48/49 + Table 49.A + 14 planetary corrections | ✓ | <1 ms | the default; deterministic, no network |
| `jpl` | NASA JPL Horizons HTTP API, body 301 (Moon) from 500@399 (Earth geocenter), quantities 10 (Illu%) + 24 (S-T-O phase angle) | — | ~500 ms | authoritative ground truth |
| `codex` | shell-out to local `codex` CLI asking the LLM for the phase | — | ~5 s | for comparison only; LLMs are unreliable on specific dates |
| `claude` | shell-out to local `claude` CLI asking the LLM for the phase | — | ~5 s | same caveat |

## Cross-verify

```bash
python3 skills/hatch-me/scripts/moon_phase.py --date 1995-08-14T10:20:00Z --verify
#   moon phase at 1995-08-14T10:20:00Z
#   ──────────────────────────────────────────────────────────────────────────────
#   source         phase                illum%   phase°   Δ vs JPL
#   ──────────────────────────────────────────────────────────────────────────────
#   jpl-horizons   Waning Gibbous      83.8492  47.3955   ── (ground truth)
#   meeus          Waning Gibbous      83.8900  47.3280   +0.0408% / -0.0675° ✓

# Add the LLM sources too (opt-in, slow, may hallucinate):
python3 skills/hatch-me/scripts/moon_phase.py --date 1995-08-14T10:20:00Z --verify --with-llm
```

Tolerances against JPL: illumination ±1.0%, phase angle ±0.5°. `--verify` exits
non-zero if any source falls outside.

## Self-test

```bash
python3 skills/hatch-me/scripts/moon_phase.py --self-test
```

Checks four reference events: 2000-01-06 new moon, 2024-04-08 eclipse, 2024-04-23
full moon, 1969-07-20 Apollo 11 landing.

Apache 2.0
