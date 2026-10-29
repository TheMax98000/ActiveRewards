# ActiveRewards

**Daily login, streaks and AFK-proof playtime rewards** in one plugin, for Paper and Folia. Players get a reward each
day they join, a bigger one for their streak, and rewards for the time they really play: an AFK farm, a water stream,
pistons, a minecart loop or an auto-clicker do not move the counter.

Today a server owner buys two or three plugins for this (a daily rewards plugin, a playtime rewards plugin, an
anti-AFK plugin), and the paid ones are often no longer updated. ActiveRewards does the three, works out of the box
(a 7-day cycle and playtime tiers are included), and never gives a reward twice: not by spamming the menu, not on two
servers at once, not after a crash.

> **Status:** version 1.0.0 is in final testing, not released yet. Free edition: SpigotMC, CurseForge and
> [Releases](../../releases/latest); full edition: Payhip (links when published).

## Editions

Active time (the anti-AFK) and the Bedrock menus are never limited in the free edition.

| | Free | Full (8.99 USD) |
|---|---|---|
| Daily login cycle | 1 cycle, up to 7 days | unlimited cycles (one per permission), any length |
| Streaks and streak rewards | yes | yes |
| Playtime tiers | up to 5 | unlimited |
| Active time (AFK farms do not count) | yes | yes |
| Native Bedrock menus, Java chest menu, leaderboards | yes | yes |
| Storage | SQLite | SQLite or MySQL/MariaDB, several servers sharing one database |
| Today's playtime (and tiers on it) | no | yes |
| PlaceholderAPI | no | yes |
| Import from vanilla statistics or CSV | no | yes |
| LuckPerms actions (`group-add`, `group-temp`) | no | yes |
| EssentialsX `/afk` counted as inactive | no | yes |
| Languages | server language | each player's game language |

The free edition loads the start of a longer config (the first 7 days, the first 5 tiers), ignores the rest and says
so once in the console.

## Daily login and streaks

- **Rolling day, by default**: a player can claim again when `claim-cooldown` (20 h) has passed since their last claim.
  No midnight reset, so no time zone problem, no host change problem, nothing missed while the server is off.
- **Calendar day, as an option** (`day-mode: calendar`): one claim per date in the `timezone` of the config.
- **Streaks**: one missed day is forgiven by default (`grace-days: 1`). A lost streak sends the player back to day 1 of
  the cycle (`reset-cycle-on-streak-loss: false` lets them continue where they were). `streak-reset: false` never
  resets streaks; the permission `activerewards.streak.keep` protects one player.
- **Streak rewards** at 7, 30, 100 days (your choice), once per streak. The best streak is kept.
- **Catch-up**, as an option: the next claim also gives the skipped days of the cycle (`catch-up-max`), without saving
  a lost streak.
- **Reminder** when a player joins with a reward waiting: a clickable message on Java, the form on Bedrock.
- **Limit per IP address**, as an option (`max-claims-per-ip`).
- **Several cycles** (full edition): a VIP cycle with its own permission, claimed together with the default one.

## Active time: AFK-proof playtime

Each player has two counters: time online and **active time**. A player is active while the server sees a sign of a
real person:

- the head turns (on Java and on Bedrock);
- the player moves, and nothing explains the move: not in a vehicle, not in water or a bubble column, not pushed by a
  piston;
- a chat message or a command;
- a click, a block broken or placed, a fishing rod cast: only if the head or the position changed since the previous
  one. An auto-clicker that never moves does not count, and neither does an AFK fishing farm. A real angler moves
  the camera from time to time and stays active. On a phone or tablet, a fish reeled in always counts: with a
  finger you fish without moving the camera.

After `afk-after` (5 min) without any sign, active time stops; it starts again at the next sign. **That is the only
consequence by default**: no kick. A kick or a console command are options (`afk-action`).

Tested with bots: water stream, bubble column, pistons pushing the player, minecart loop, auto-clicker, AFK fishing
farm and a still player: active time stops. Turning the head, walking, building, fishing normally: everything counts.

Full edition: a player marked AFK by EssentialsX (`/afk`) is inactive too, and a **today's playtime** counter
(active time since midnight) can carry tiers ("1 hour today").

## Playtime tiers and rewards

- **Once**: "10 hours of active time", given once per player.
- **Repeatable**: "every 30 minutes of active time".
- On active time by default; on time online or today's time as an option. A tier can require a permission.
- **Ranks on playtime**: a tier that runs a LuckPerms command, or in the full edition `group-add: vip` and
  `group-temp: "vip 7d"` (the duration adds to the time already left).

The same actions everywhere (cycle days, streak rewards, playtime tiers): console command (`{player}`, `{uuid}`), item
(material, amount, name, `custom-model-data`), message, sound, and a chance in % on each action. An item that does not
fit in the inventory goes to a **delivery box** opened from the menu. Never dropped on the ground.

## Menus

`/daily` opens the main menu: the days of the cycle (claimed, today, coming), the playtime tiers with progress, the
delivery box and the leaderboards (active time, streak, best streak; top 10, refreshed every 5 min).

- **Java**: a chest menu, readable without configuring anything.
- **Bedrock players** (detected by Floodgate) get native Bedrock forms with the same screens as buttons. Without
  Floodgate everyone gets the chest menu.

## Never a reward twice

The database is the only source of truth. A reward is written to the database before it is given:

- a claim is one conditional database update: 30 clicks in one second, or the same player claiming on two servers in
  the same second, give one reward;
- each delivery has a row, marked delivered after its actions ran;
- a crash between the two (less than a tick) loses that reward instead of giving it twice. At startup the plugin lists
  undelivered rewards in the console and in `/ar pending`; an admin gives one with `/ar pending give <id>` (once) or
  drops it. Nothing is given again on its own.

Tested by killing the server (`kill -9`) during a delivery.

## Commands

| Command | What it does | Permission (default) |
|---|---|---|
| `/daily` (alias `/rewards`) | main menu | `activerewards.use` (everyone) |
| `/daily claim`, `/daily top` | claim, leaderboards | `activerewards.use` |
| `/playtime [player]` | active time, time online, today's time | `activerewards.use`; another player: `activerewards.playtime.others` (op) |
| `/ar give <player> <cycle> <day>`, `/ar give <player> tier <tier>`, `/ar give <player> streak <days>` | give a reward again | `activerewards.admin` (op) |
| `/ar playtime <player> set\|add\|remove <duration>` | fix a playtime, offline players too | `activerewards.admin` |
| `/ar streak <player> set <n>\|reset` | fix a streak | `activerewards.admin` |
| `/ar pending [give\|drop <id>]` | rewards interrupted by a crash | `activerewards.admin` |
| `/ar import vanilla\|csv <file> [--overwrite] [--grant]` | import (full edition) | `activerewards.admin` |
| `/ar export`, `/ar reload`, `/ar version` | JSON backup of all data, reload, version | `activerewards.admin` |

Other permissions: `activerewards.streak.keep` (never lose a streak), `activerewards.afk.bypass` (staff: never counted
inactive).

## PlaceholderAPI (full edition)

`%activerewards_streak%`, `_best_streak%`, `_next_claim%` (formatted), `_next_claim_seconds%`, `_active%`,
`_active_seconds%`, `_raw%` (time online), `_raw_seconds%`, `_today%`, `_today_seconds%`, `_rank_active%`,
`_top_active_<n>_name%`, `_top_active_<n>_value%`.

## Import (full edition)

- `/ar import vanilla`: the `PLAY_ONE_MINUTE` statistic of every known player, offline players too. It fills the time
  online, and the active time as well (vanilla cannot tell AFK time apart; the command says so).
- `/ar import csv <file>`: a file in `plugins/ActiveRewards/` with the columns
  `uuid,raw_seconds,active_seconds,streak,last_claim_epoch`, exported from your old plugin. A line that cannot be read
  is skipped and counted.
- An import never lowers a value unless `--overwrite`. Tiers the player already reached are marked received without
  giving them; `--grant` gives them instead.

## Configuration

`plugins/ActiveRewards/config.yml` (settings), `daily.yml` (cycles and streak rewards), `playtime.yml` (playtime
tiers), all commented. It works without any change (SQLite).

```yaml
language: en              # server log, and players whose game language has no text file
update-check: true
day-mode: rolling         # or calendar
claim-cooldown: 20h       # rolling mode
timezone: system          # written in clear at first start
grace-days: 1
streak-reset: true
reset-cycle-on-streak-loss: true
loop: true                # back to day 1 after the last day (false: stay on the last day)
catch-up: false
catch-up-max: 3
max-claims-per-ip: 0      # 0: no limit
afk-after: 5m
afk-action: none          # or kick, or command:<console command with {player}>
server-name: server1      # different on each server sharing a database
storage:
  type: sqlite            # or mysql (full edition): host, port, database, user, password
```

```yaml
# daily.yml
cycles:
  default:
    permission: ""
    days:
      1:
        - {item: BREAD, amount: 8}
      7:
        - {item: DIAMOND, amount: 2}
        - {sound: minecraft:ui.toast.challenge_complete}
streak-tiers:
  7:
    - {item: GOLDEN_APPLE, amount: 1}
```

```yaml
# playtime.yml
tiers:
  ten-hours:
    at: 10h
    actions:
      - {item: DIAMOND, amount: 3}
  every-30-minutes:
    every: 30m
    actions:
      - {item: EXPERIENCE_BOTTLE, amount: 4}
  vip:                      # full edition
    at: 50h
    actions:
      - {group-temp: "vip 7d"}
```

Texts: `plugins/ActiveRewards/lang/en.yml` and `fr.yml` (MiniMessage). Names of cycles, tiers and given items are
language keys. In the full edition each player gets the language of their game when a file exists; add a language by
copying a file (`de.yml`, `es.yml`...).

## Several servers (full edition)

Set `storage.type: mysql` with the same MySQL or MariaDB database on every server, and a different `server-name` on
each. Streaks and claims are shared (the same player claiming on two servers in the same second gets one reward), and
playtime adds up: each server adds its share every 60 s and when the player leaves. No Redis, no proxy plugin. The
MySQL driver comes with Paper.

## Requirements

- Paper or Folia 1.21.4 to 26.x, Java 21+ (tested on Paper 1.21.11, Paper 26.3 and Folia 26.3)
- Optional: [Floodgate](https://geysermc.org/download), for the Bedrock menus.
- Optional (full edition): [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/),
  [LuckPerms](https://luckperms.net/) (tested with 5.5), [EssentialsX](https://essentialsx.net/) (tested with 2.22).
- No license server, no external service.

## Data

`plugins/ActiveRewards/data.db` (SQLite) or your MySQL database, tables prefixed `ar_`. The schema is migrated
automatically between versions. `/ar export` writes all the data to a JSON file. Delivery log:
`plugins/ActiveRewards/logs/activerewards.log`.

## Support

Bug reports and questions: [issues](https://github.com/TheMax98000/ActiveRewards/issues).

## Also by the author

- [CrossplayMarket](https://www.spigotmc.org/resources/crossplaymarket-free.139834/): auction house with native
  Bedrock menus, duplication-proof.
- [CrossplayHomes](https://www.spigotmc.org/resources/crossplayhomes.139801/): homes and TPA with native Bedrock
  menus.
- [CrossplayShield](https://www.spigotmc.org/resources/crossplayshield.139762/): anti-exploit for cross-play servers
  that does not kick Bedrock players.
- [BedrockGuard](https://www.spigotmc.org/resources/bedrockguard-free.139718/): anti-cheat built for Bedrock players.
