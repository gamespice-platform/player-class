# GameSpice Player Class

What a player is, as a document rather than as a scene: a catalogue of classes made of numbers and ids, a manager that records a class change as a request the next spawn honours, and a duck-typed bridge that writes a class's numbers onto health and movement.

Read the family [CLAUDE.md](../../CLAUDE.md) first.

**The distributable is `addons/gs_player_class/`.** It requires [GameSpice Core](../gs-core) and [GameSpice Player](../gs-player) (symlinked into `addons/`, gitignored), and nothing else: gs-combat and gs-player-controller are reached by duck typing.

Also requires **GameSpice Profile** (`GSProfileSet`, `GSProfile`, `GSProfileRuntime`, split from GameSpice Core): link `addons/gs_profile` next to `addons/gs_core`.

## Read before

| Read | Before |
| --- | --- |
| [docs/design.md](docs/design.md) | changing anything: requests not setters, content-free validation, counting, pending re-checks, entitlement, presets, the apply bridge |

## Layout

| Path | What |
| --- | --- |
| `core/gs_player_class_def.gd` | `GSPlayerClassDef`: one class, as numbers and ids (health, armour, regen, speed/jump/knockback scales, mass, loadout and char ids, abilities, `limit_per_team`, `requires_entitlement`) |
| `core/gs_player_class_catalogue.gd` | `GSPlayerClassCatalogue`: the set, `get_class_def`, `fallback_for`; presets `single()`, `team_shooter()`, `asymmetric()` |
| `core/gs_player_class_rules.gd` | `GSPlayerClassRules`: when a choice may be made and when it lands |
| `runtime/gs_player_class_manager.gd` | `GSPlayerClassManager`: current, pending, limits, entitlements, mirror; `request`, `force`, `apply_pending`, `def_of`, `options_for` |
| `core/gs_player_class_profiles.gd` | `GSPlayerClassProfiles`: the GameSpice Profile schema for kind `class`; `make_set`, `load_def`, `to_catalogue` ([design](docs/design.md#classes-as-gs-profile-data)) |
| `bridge/gs_player_class_apply.gd` | `GSPlayerClassApply`: `to_health`, `to_movement`, `knockback_scale`, `to_immunity` (the `immune` list onto GameSpice Combat's `GSDamageImmunity`), all duck-typed |
| `examples/class_selftest.tscn` | the suite |

Paths except `examples/` are in `addons/gs_player_class/`.

## Running and testing

The family's two steps ([family CLAUDE.md](../../CLAUDE.md#validating-anything)), then `timeout 120 godot --headless --path . res://examples/class_selftest.tscn`.

## Rules that bite

- A class change is a request recorded for the next spawn; `request()` returns a `GSResult`, refusals carry a sentence. [design](docs/design.md#the-one-idea)
- Every field of `GSPlayerClassDef` is a number or an id; a server validates without loading content. [design](docs/design.md#the-one-idea)
- The accessor is `get_class_def`; never define `get_class` (it silently shadows `Object.get_class`). [design](docs/design.md#get_class_def-not-get_class)
- `count_on_team` ignores pending choices; the limit check does not count the asker against their own class. [design](docs/design.md#counting-counts-what-people-are)
- `apply_pending` re-checks and fires `class_refused` when the choice stopped being legal. [design](docs/design.md#a-pending-choice-can-stop-being-legal)
- Entitlement is the game's `entitled_fn`; this addon only carries the id. [design](docs/design.md#entitlement-is-checked-by-the-game-never-here)
- The manager applies nothing; the game reads `def_of(key)` on spawn. [design](docs/design.md#what-the-manager-does-not-do)
- `GSPlayerClassApply` never names `GSHealth`, `GSFpsTunables` or any class this project does not install; the stub-based suite is the check. [design](docs/design.md#gsplayerclassapply-the-seam-and-why-it-is-duck-typed)
- Pass `to_movement` the untouched `base`, or scales compound on every respawn; `reset_to_full` runs after the maximum is written. [design](docs/design.md#gsplayerclassapply-the-seam-and-why-it-is-duck-typed)
