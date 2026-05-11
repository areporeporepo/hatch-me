# hatch-me

Hatch yourself as a Codex baby from your date of birth. The moon phase the
day you were born is computed accurately and baked into the seed.

A first-person fork of [`hatch-pet`](https://github.com/codex-pets/codex-pets)
and [`hatch-mom`](https://github.com/openai/skills) (Apache 2.0).

```bash
cp -R skills/hatch-me ${CODEX_HOME:-$HOME/.codex}/skills/
python3 ${CODEX_HOME:-$HOME/.codex}/skills/hatch-me/scripts/hatch.py --dob 1995-08-14T03:20 --tz -07:00
```

```
DOB ─┐
     │   ┌──────────────────────────┐
     ├──▶│ scripts/moon_phase.py    │── phase, illumination, age ─┐
     │   │ Meeus AA2 Ch. 47/48/49   │                              │
     │   └──────────────────────────┘                              ▼
     │                                                ┌──────────────────────────┐
     │                                                │ scripts/hatch.py          │
     └───────────────────────────────────────────────▶│ BLAKE2b(salt|dob|moon)    │
                                                      │ → SplitMix64 → Soul       │
                                                      └────────────┬─────────────┘
                                                                   ▼
                                       ~/.codex/memories/babies/<name>/baby.json
                                                + ASCII birth card on stdout
```

Apache 2.0
