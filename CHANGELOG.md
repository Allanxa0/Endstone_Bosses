# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

### Added
- Boss and miniboss types: Ravager, Evoker, Blaze, Vindicator, Warden, Miniboss, Breeze, Bogged and Piglin Brute.
- Health and damage multipliers per type, configurable in `config.toml`.
- `/bosses` command (op by default) with a form to edit each boss type.
- Spawn zones per boss type, registered from the player's current position and dimension, saved in `data.json`.
- Automatic boss spawning with a per-type cooldown that starts on death; one boss per zone at a time.
- Custom boss name and per-type cooldown, editable from the form.
- Configurable `spawn_broadcast` announcement in `config.toml`.
- Colored, icon-based forms.
- Dynamic boss levels driven by remaining health, with configurable thresholds, damage bonus, cooldown scaling and level and health shown in the name tag.
- Automatic boss regeneration after a configurable time without taking damage.
- Combat intelligence that picks abilities from recent damage, hit frequency, number of attackers and nearby players instead of fixed health percentages.
- Silverfish minions with configurable health, damage, name, cooldown and cap, removed when the boss dies.
- Explosion ring, lightning strikes with telegraphed positions and per-target cooldown, and a rage phase triggered by heavy burst damage.
- Negative effects (Poison, Wither, Levitation, Nausea, Blindness, Hunger, Slowness, Weakness) combined by family and severity budget, with a grace period and low-health protection.
- Full tuning of combat in `config.toml`: levels, regeneration, ability chances and cooldowns, minion stats, effect intensity and duration, and messages.
- Damage tracking per boss fight, with a Top 5 announced in chat on death showing each player's position, damage and reward.
- Rewards per boss type: ground drops, a reward list for each of the Top 5 positions and drops for its minions, each with minimum and maximum amount and its own drop chance.
- In-game reward editor under `/bosses` > Recompensas: add the item in hand or pick one from your inventory (name, lore and enchantments are kept), then edit amounts and chance or delete it. Saved in `plugins/bosses/rewards.json`.
- `/bossrank` shows the live damage ranking of the nearest boss fight and `/bossrewards` lists each boss's rewards (`bosses.command.info`, everyone by default).
- Rewards, minion drops and rankings only apply to bosses and minions spawned by the plugin, never to other entities of the same type outside the dimensions with spawn zones.
- Boss state (health, damage ranking, position and zone) is saved in `plugins/bosses/state.json` and restored when the boss chunk loads again, including after a server restart.
- An untagged entity of a boss type that a player hits inside a dimension with spawn zones is adopted as a boss.

### Changed
- Boss health is now an exact value in points (1 to 10000, default is the vanilla health of the entity) instead of a multiplier; damage keeps its multiplier. Previous health multipliers are not converted and each boss starts with its default health.
- Pigman is now Ravager, Piglin is now Evoker and Bruga is now Vindicator; Piglin Brute is a separate boss. Settings, zones, rewards and saved state of the old types carry over to the new ones.
- The reward chance is now typed directly instead of using a slider; it accepts decimals such as `0.1` from 0.01 % to 100 %, so very rare drops can be configured.
- The default `combat.think_interval` is now 60 ticks (3 s) instead of 20, which lowers the per-boss cost; existing `config.toml` files keep their value. The health shown in the name tag refreshes at that interval.
- Health, damage and cooldown in the boss attributes form are now typed numbers (decimals allowed, values outside the allowed range are adjusted) instead of sliders.
- The negative effect notification is now a chat message that includes the boss name; `messages.curse` accepts `{boss}` and `{effects}`.
- Boss settings (multipliers, name, cooldown) are now stored in `plugins/bosses/data.json` instead of `config.toml`.

### Fixed
- Bosses that disappeared when their chunk unloaded are now recreated at their last position, with their saved health, maximum health and damage ranking, when a player comes near, instead of being replaced by a fresh boss.
- Server crash when a boss chunk unloaded while the plugin still held a reference to the boss: the plugin no longer keeps references to entities, so unloaded bosses keep their health and ranking and are restored on load.
- A boss in an unloaded chunk no longer lets its zone spawn a duplicate.
- Server crash (segfault) while the boss spawner scanned the world: bosses now only spawn when a player is within 64 blocks of the zone, and only the affected dimension is scanned.
