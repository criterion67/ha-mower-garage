# Home Assistant robot mower with a garage dock

This is how I run a MOVA LiDAX Ultra 2000 AWD robot mower from Home Assistant. The mower's dock is inside my garage, so most of the logic is about getting the garage door and the mower to work together safely.

It is shared as is, for reference. It is built around my yard, my entity names and my hardware, so expect to adapt it rather than drop it in.

## What you need

* The **Dreame Lawn Mower** custom integration from HACS: [EvotecIT/homeassistant-dreamelawnmower](https://github.com/EvotecIT/homeassistant-dreamelawnmower)
* A garage door opener with **open, stop and close** in Home Assistant (mine is an Athom ESPHome opener, `cover.garage_door`)
* A contact sensor on any gate the mower has to pass through (mine is `binary_sensor.rear_yard_gate`, on = open)
* Zones set up in the MOVAhome app (mine: 1 front yard, 2 side yard, 3 rear yard)

## What is here

| Folder | Contents |
|---|---|
| [scripts](scripts) | The 4 mower scripts, in the same format as `scripts.yaml` |
| [Automations](https://github.com/criterion67/HA-Automations/tree/main/_Uncategorized) | The 8 `mower_*` automations live in my HA-Automations repo, which is updated nightly from my live config |

### Scripts

| Script | What it does |
|---|---|
| `mower_zone_cycle` | Core outbound sequence. Forces Zone mode and reads it back, checks the gate for rear yard jobs, opens the door 2.5 seconds then stops it (about 21 inches), starts the zone mow, and recalls the mower and closes the door if it never leaves the dock |
| `mower_full_yard_chain` | Starts a chained Rear, Front, Side run (leg one only; the chain automation does the rest) |
| `mower_front_side_chain` | Starts a chained Front, Side run |
| `mower_send_home` | Opens the door first, then sends the mower home |

### Automations (in HA-Automations)

| Automation | What it does |
|---|---|
| `mower_garage_return` | Opens the door when the mower is returning, closes it after docking, and sends error alerts |
| `mower_outbound_door_close` | Closes the door once the mower has cleared it |
| `mower_unattended_start` | Opens the door when the mower starts on its own (for example resuming after a recharge), only if it is really mowing |
| `mower_chain_advance` | Starts the next leg of a chained run after a healthy dock |
| `mower_gate_reminders` | Pauses and alerts if the mower heads for a zone with the gate in the wrong position |
| `mower_incomplete_job_alert` | Alerts when a job ends without completing |
| `mower_restart_recovery` | Warns if Home Assistant restarts mid mow |
| `mower_notification_actions` | Handles the Close Garage and Leave Open buttons on phone alerts |

## Helpers to create first

| Helper | Type | Notes |
|---|---|---|
| `input_boolean.mower_cycle_active` | Toggle | Raised during a mower cycle. Suppresses my normal garage door announcements and arms the return automation |
| `input_boolean.mower_full_yard_cycle` | Toggle | Used by the gate reminder automation |
| `input_select.mower_chain_leg` | Dropdown | Options in this order: `Idle`, `Rear Yard`, `Front Yard`, `Waiting For Gate`, `Side Yard`. Leave the initial value unset so it survives restarts |

## Things to change for your setup

* `script.notify_alert` and `script.notify_clear` are my own notification wrappers and are not included. Replace them with your own `notify.mobile_app_*` calls.
* TTS goes to `media_player.bedroom_group` through `tts.home_assistant_cloud`.
* Mower entities are named `jason_momower_*` because the mower is named Jason MoMower.
* The 2.5 second door timing and the 120 second clearance wait were measured on my door and my mower. Measure yours.

## Lessons learned

* **Zone mode does not stick.** If the mower is in All area mode it silently ignores the zone list. It also resets after a Home Assistant restart or integration reload. Every script sets Zone and reads it back before doing anything.
* **Multi zone jobs did not work for me.** The mower finishes the first zone and heads for the dock. That is why the full yard run is a chain of single zone jobs.
* **Do not trust the docked flag by itself.** Mine flickers on for up to about 30 seconds while the mower is still out on the lawn. Use a `for:` hold and a recheck before closing the door.
* **Pulse openers reverse.** Sending open to a door that is already open or stopped partway closes it. Every open is guarded by a check that the door is closed.
* **Photo eyes are not a safety net here.** The mower spends about a minute turning near the dock before it drives out, so a door that closes too early just shuts in front of it.

The descriptions and `note:` fields in each file explain why each step exists.
