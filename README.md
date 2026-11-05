# Dehumidifier kit for Home Assistant

[![Validate](https://github.com/SudoManaged/ha-dehumidifier-kit/actions/workflows/validate.yml/badge.svg)](https://github.com/SudoManaged/ha-dehumidifier-kit/actions/workflows/validate.yml)
![Home Assistant 2025.8+](https://img.shields.io/badge/Home%20Assistant-2025.8%2B-41BDF5)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Know how full your dehumidifier's bucket is, get told before it fills up, and run the unit from a humidity
sensor, using nothing more than the smart plug it is plugged into.

A free, open-source set of Home Assistant blueprints by SudoManaged. It runs entirely on your own Home Assistant:
no add-ons, no custom integrations, no cloud services, no accounts. It started as automations that run every day
on our own Home Assistant, and was then generalized and tested for release.

## Contents

- [What is in the kit](#what-is-in-the-kit)
- [Requirements and tested versions](#requirements-and-tested-versions)
- [What you need](#what-you-need)
- [Installation](#installation)
- [How the bucket tracker works](#how-the-bucket-tracker-works)
- [1. Find your unit's numbers](#1-find-your-units-numbers)
- [2. Add the bucket tracker](#2-add-the-bucket-tracker)
- [3. Add the "bucket emptied" script](#3-add-the-bucket-emptied-script)
- [4. Add the bucket alerts](#4-add-the-bucket-alerts)
- [5. Add the humidity band](#5-add-the-humidity-band)
- [6. Show it on a dashboard](#6-show-it-on-a-dashboard)
- [The bucket sensor's attributes](#the-bucket-sensors-attributes)
- [Using it in your own automations](#using-it-in-your-own-automations)
- [How accurate is it?](#how-accurate-is-it)
- [Several dehumidifiers](#several-dehumidifiers)
- [Restarts and power cuts](#restarts-and-power-cuts)
- [Starting the learning again](#starting-the-learning-again)
- [Troubleshooting](#troubleshooting)
- [Limitations](#limitations)
- [Updating and removing](#updating-and-removing)
- [Security](#security)
- [Reporting problems and contributing](#reporting-problems-and-contributing)
- [More free Home Assistant tools from SudoManaged](#more-free-home-assistant-tools-from-sudomanaged)
- [Changelog and license](#changelog-and-license)

## What is in the kit

| Blueprint | Type | What it does |
|---|---|---|
| [Dehumidifier bucket tracker](blueprints/template/sudomanaged/dehumidifier_bucket.yaml) | template (a sensor) | A sensor with the bucket level in %. Learns the bucket's size from the plug's energy readings and notices by itself when the bucket is full and when it was emptied. |
| [Dehumidifier bucket emptied](blueprints/script/sudomanaged/dehumidifier_bucket_emptied.yaml) | script | Resets the count when you empty the bucket before it is full, and starts the count from a known empty when you set the tracker up. |
| [Dehumidifier bucket alerts](blueprints/automation/sudomanaged/dehumidifier_bucket_alerts.yaml) | automation | "Almost full" and "full" notifications to your phone (or another notify action), cleared when the bucket is emptied. |
| [Dehumidifier humidity band](blueprints/automation/sudomanaged/dehumidifier_humidity_band.yaml) | automation | Switches the plug on above one humidity and off below another, with a hold time against short bumps and a minimum off time for the compressor. |

The humidity band works on its own, without the bucket tracker. The alerts and the script need the bucket
tracker. The bucket tracker needs the plug to be in charge of the unit (see [What you need](#what-you-need)).

## Requirements and tested versions

- **Home Assistant 2025.8 or newer.** 2025.8 is the first release where a trigger-based template sensor can show
  **unknown**, which the bucket tracker needs before it has learned the bucket size. Each blueprint declares
  this, and Home Assistant refuses to import the automation and script blueprints from the UI on an older
  release; a hand-copied blueprint file is not checked, so check your version first.
- The **logbook** integration (part of Home Assistant's default configuration), for the alerts.
- All four pass Home Assistant's configuration check on **2025.8.0**, **2026.3.3** and **2026.11.0**. They were
  also run live on all three with a simulated plug and humidity sensors; see [CHANGELOG.md](CHANGELOG.md) for the
  list of situations tested.
- Every change to this repository is checked automatically against Home Assistant 2025.8.0 and the current
  release (see [`.github/workflows/validate.yml`](.github/workflows/validate.yml)).

## What you need

1. **A dehumidifier that starts again by itself when its power comes back.** Most current units have this
   ("auto-restart" in the manual). Test it: with the unit running, pull the plug, wait a minute, plug it back
   in. If it starts running again without you pressing a button, it works with this kit. If it waits in standby,
   the humidity band cannot start it, and the bucket tracker would take the standby for a full bucket.
2. **The unit set to run continuously** (its own humidity target at its lowest, or its "continuous" mode), so the
   plug is in charge. A unit whose own humidistat stops the compressor and fan looks, to the plug, exactly like a
   unit with a full bucket. (If your unit keeps its fan running when its humidistat is satisfied, and the fan
   draws more than "Stopped below", the tracker can tell the two apart and this does not matter.)
3. **A smart plug that reports power (W) and energy (kWh or Wh) to Home Assistant**, rated for the unit.
   Dehumidifiers draw several hundred watts with a short surge when the compressor starts, so use a plug rated
   for at least the unit's rated current (15 A / 1800 W plugs are common in North America, 16 A / 3680 W in
   Europe). We use TP-Link Tapo P115 plugs through Home Assistant's built-in TP-Link Smart Home integration; any
   plug with a power sensor and an energy sensor works.
4. **For the humidity band:** a humidity sensor in the same room. Put it away from the unit's air outlet: the dry
   air coming out of the unit reads lower than the room.

## Installation

**The automation and script blueprints** can be imported with these buttons (each opens your own Home Assistant
through my.home-assistant.io):

| Blueprint | Import |
|---|---|
| Dehumidifier bucket emptied (script) | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FSudoManaged%2Fha-dehumidifier-kit%2Fblob%2Fmain%2Fblueprints%2Fscript%2Fsudomanaged%2Fdehumidifier_bucket_emptied.yaml) |
| Dehumidifier bucket alerts (automation) | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FSudoManaged%2Fha-dehumidifier-kit%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fsudomanaged%2Fdehumidifier_bucket_alerts.yaml) |
| Dehumidifier humidity band (automation) | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FSudoManaged%2Fha-dehumidifier-kit%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fsudomanaged%2Fdehumidifier_humidity_band.yaml) |

**The bucket tracker** is a *template* blueprint, which you use from `configuration.yaml` (step 2), so copy that
file by hand. Or copy all four by hand, keeping the folder names:

| File in this repository | Copy to |
|---|---|
| `blueprints/template/sudomanaged/dehumidifier_bucket.yaml` | `/config/blueprints/template/sudomanaged/dehumidifier_bucket.yaml` |
| `blueprints/script/sudomanaged/dehumidifier_bucket_emptied.yaml` | `/config/blueprints/script/sudomanaged/dehumidifier_bucket_emptied.yaml` |
| `blueprints/automation/sudomanaged/dehumidifier_bucket_alerts.yaml` | `/config/blueprints/automation/sudomanaged/dehumidifier_bucket_alerts.yaml` |
| `blueprints/automation/sudomanaged/dehumidifier_humidity_band.yaml` | `/config/blueprints/automation/sudomanaged/dehumidifier_humidity_band.yaml` |

After copying files by hand, restart Home Assistant (or reload automations and scripts).

## How the bucket tracker works

A dehumidifier pulls water out of the air with a compressor, and the water it collects follows the energy the
compressor uses. The plug already counts that energy. So:

- **Counting.** Every time the plug's energy counter goes up, the tracker adds the difference to `energy_kwh`,
  the energy used since the bucket was last emptied.
- **Full.** When the bucket is full, the unit stops by itself while the plug stays on: the power drops to a few
  watts or less. The tracker looks at the plug every minute; once the power has stayed below **Stopped below**
  for about **Stopped for** minutes (5 W and 10 minutes by default; give or take a minute, since it checks once a
  minute), it marks the bucket `full` and shows 100%.
- **Emptied.** When you empty a full bucket and put it back, the unit starts again and the compressor's power
  goes above **Compressor running above**. At the next minute the tracker sees it: the count starts again at 0.
  If you empty the bucket before it is full, run the "bucket emptied" script: the count starts again at 0.
- **Learning.** When the unit starts again after a full bucket, that bucket's energy is saved in `buckets`, and
  the bucket size, `capacity_kwh`, is the average of the last 3. The level is `energy_kwh / capacity_kwh` in %.
  A full bucket is only learned when it makes sense:
  - its count started at a known empty (the first bucket after you set the tracker up may have been part full
    already, so it is not learned unless you ran the "bucket emptied" script when you set it up);
  - it used at least **Smallest real bucket** kWh;
  - the unit itself started again after it, and you did not run the "bucket emptied" script. If you take the
    bucket out early and it stays out longer than **Stopped for**, the tracker shows it as full; the script then
    makes sure that part-full bucket does not count, whether you run it before you put the bucket back or up to
    10 minutes after the unit starts again (it takes back a bucket learned in the last 10 minutes). So after a
    really full bucket, just empty it and put it back; you do not need the script.

```
  plug on, compressor running                  unit stops, plug still on
  energy_kwh counts up ─────────────────────▶  (under 5 W for 10 minutes)
        ▲                                               │
        │                                               ▼
  compressor starts again: bucket               full: 100%, this bucket's
  learned, count = 0           ◀──────────────  energy waits in pending_kwh
```

Until the first learned bucket the tracker does not know the bucket's size, so the level is **unknown**, apart
from 100 while the bucket is full (the energy is still counted in `energy_kwh`). If you already know the size,
enter it as "Known bucket size" to get a level from the start. On the unit we built this for, a full bucket takes
12 to 14 kWh.

**Bad readings.** A plug's energy counter sometimes misbehaves. The tracker ignores readings that are not
numbers; ignores a small drop (less than 10%) as noise; treats a bigger drop as the counter starting again (a
daily counter at midnight, or a plug that was reset); and ignores an increase that would mean more than 4 kW
since the previous reading, which no plug can deliver.

## 1. Find your unit's numbers

The defaults suit most units, but take two minutes to check them against your own plug:

1. Open the plug's **power** sensor in Home Assistant and look at its history over a few hours of the unit
   running.
2. Note three levels:
   - **Compressor running:** the high level, often 300 to 700 W.
   - **Fan only:** many units run the fan for a few minutes before the compressor starts, often 30 to 80 W.
   - **Stopped:** with the bucket full (or the bucket taken out), usually 0 to 3 W.
3. Pick:
   - **Stopped below:** above the stopped level and below the fan-only level. The default is 5 W.
   - **Compressor running above:** above the fan-only level and below the compressor level. The default is 150 W.

On the units we built this for: about 500 to 700 W running, about 58 W fan-only for the first 3 minutes, and 0 W
stopped with the bucket full.

## 2. Add the bucket tracker

Add this to `configuration.yaml` (if you already have a `template:` section, add the list item under it), with
your plug's entities:

```yaml
template:
  - use_blueprint:
      path: sudomanaged/dehumidifier_bucket.yaml
      input:
        plug_switch: switch.bedroom_dehumidifier_plug
        power_sensor: sensor.bedroom_dehumidifier_plug_current_consumption
        energy_sensor: sensor.bedroom_dehumidifier_plug_today_s_consumption
        bucket_name: bedroom
    name: Bedroom dehumidifier bucket
    unique_id: bedroom_dehumidifier_bucket
```

| Setting | Default | Range | Explanation |
|---|---|---|---|
| `plug_switch` | (required) | | The plug's switch. |
| `power_sensor` | (required) | | The plug's power sensor, in W. |
| `energy_sensor` | (required) | | One of the plug's energy counters, in kWh or Wh. A lifetime ("total") counter or a counter that starts again every day ("today") both work; the tracker notices when the counter starts again. |
| `bucket_name` | (required) | | A short name, lowercase letters, digits and underscores, for example `bedroom`. The "bucket emptied" script uses the same name. Give every dehumidifier its own name. |
| `full_watts` | `5` | 0.5 to 50 W | Power below this, while the plug is on, means the unit has stopped. |
| `full_minutes` | `10` | 1 to 60 min | How long it must stay stopped before the bucket counts as full. |
| `running_watts` | `150` | 20 to 2000 W | Power above this means the compressor is running. The compressor running again after a full stop means the bucket was emptied. |
| `min_bucket_kwh` | `1` | 0.1 to 50 kWh | A stop with less energy than this since the last empty still shows as full, but is not used to learn the bucket size (for example the unit was switched off at its own button soon after an empty). About a third of a normal bucket works well. |
| `initial_capacity_kwh` | `0` | 0 to 100 kWh | A bucket size you already know, to get a level before the first learned bucket. `0` means learn it. Learned buckets replace it. |
| `learn_buckets` | `3` | 1 to 10 | How many recent full buckets are averaged for the bucket size. |

Home Assistant does not check these ranges for a blueprint used from YAML, so stay inside them.

Check the configuration (**Developer tools, YAML, Check configuration**) and restart Home Assistant. You now have
`sensor.bedroom_dehumidifier_bucket`. It shows **unknown** (or 100 while the bucket is full) until the first
learned bucket, or right away a level if you entered a known size.

**Then empty the bucket and run the "bucket emptied" script once** (step 3). That starts the count from a known
empty, so the first full bucket teaches the bucket size. Without it, the first full bucket is not learned and
learning starts with the second.

The sensor is created by a template, so it does not belong to a device and its settings live in
`configuration.yaml`. You can still change its name, icon and area in the UI.

## 3. Add the "bucket emptied" script

Run it once when you set the tracker up (right after emptying the bucket), and whenever you empty the bucket
before it is full. You do not need it after a full bucket: the tracker notices the unit starting again, and learns
from that bucket. If you run it after a full bucket anyway (before, or up to 10 minutes after, the unit starts
again), the count starts again correctly, but that bucket is not learned.

From the UI: **Settings, Automations & scenes, Scripts, Create script, Create new script from a blueprint**,
choose **Dehumidifier bucket emptied**, enter the same bucket name (`bedroom`), and save it as, for example,
"Bedroom bucket emptied".

Or in `scripts.yaml`:

```yaml
bedroom_bucket_emptied:
  alias: Bedroom bucket emptied
  use_blueprint:
    path: sudomanaged/dehumidifier_bucket_emptied.yaml
    input:
      bucket_name: bedroom
```

Running it fires the event `dehumidifier_bucket_emptied` with `bucket: bedroom`. Anything else that fires that
event resets the tracker too, for example a button on the wall through an automation:

```yaml
actions:
  - event: dehumidifier_bucket_emptied
    event_data:
      bucket: bedroom
```

## 4. Add the bucket alerts

From the UI: **Settings, Automations & scenes, Create automation, Create new automation from a blueprint**,
choose **Dehumidifier bucket alerts**.

| Setting | Default | Explanation |
|---|---|---|
| Bucket sensor | (required) | The sensor from step 2. |
| Almost full at (%) | `85` (50 to 100) | The level for the "almost full" alert. Set 100 for only the "full" alert. |
| Notify action | `notify.notify` | Where alerts go. For one phone use `notify.mobile_app_<your_phone>`; with the Home Assistant app installed, `notify.notify` sends to every phone that has the app. Notify groups defined in YAML work too. Notify *entities* (`notify.send_message`, including notify groups made in the UI) are not supported. |
| Home Assistant app options | on | One tag per dehumidifier, so "full" replaces "almost full"; high priority with `ttl: 0`, so Android delivers it at once (iOS ignores these two); and the phone notification is cleared when the bucket is emptied. Turn it off if your notify action is not the Home Assistant app: it would otherwise receive the text "clear_notification" when the bucket is emptied. |
| Also show alerts inside Home Assistant | on | A Home Assistant notification (the bell), dismissed when the bucket is emptied. |

What you get, titled with the sensor's name:

- **"Bedroom dehumidifier bucket: about 86% full"**: "It has used 12.2 of about 14.2 kWh per bucket since
  it was last emptied. Empty it soon."
- **"Bedroom dehumidifier bucket: full"**: "The dehumidifier stopped with its plug on: the bucket is full
  (or the unit was switched off at its own button). Empty it to start it again."

Each alert is also written to the logbook. Until the tracker knows the bucket size (learned, or entered as a known
size), the level is unknown, so the first alert you get is "full". Home Assistant clears its own notifications (the
bell) when it restarts; the phone notification stays. A sensor coming back after a reload of template entities
does not send the alerts again (this needs the `unique_id` from step 2).

## 5. Add the humidity band

From the UI: **Settings, Automations & scenes, Create automation, Create new automation from a blueprint**,
choose **Dehumidifier humidity band**.

| Setting | Default | Range | Explanation |
|---|---|---|---|
| Humidity sensor | (required) | | The room's humidity sensor. |
| Smart plug | (required) | | The plug's switch. |
| Turn on above (%) | `55` | 20 to 90 | The plug is switched on when the humidity rises above this. |
| Turn off below (%) | `48` | 15 to 85 | The plug is switched off when the humidity falls below this. Must be lower than "Turn on above"; the automation does nothing if it is not. |
| Humidity must stay there for (minutes) | `2` | 0 to 30 | The humidity must stay above (or below) the number this long before the plug is switched. Rides over short bumps such as a shower or an open door. |
| Minimum off time (minutes) | `5` | 0 to 60 | How long the plug must have been off before it is switched on again, so the compressor can rest. |

Every 5 minutes, and when Home Assistant starts, the automation also checks the humidity, so a missed change or a
restart never leaves the unit in the wrong state, and a start held back by the minimum off time happens at a later
check. These checks use the same hold time: if the humidity goes back inside the band during it, nothing is
switched. Between the two numbers it leaves the plug as it is.

Home Assistant forgets when a switch last changed when it restarts, so right after a restart the minimum off time
counts from the restart: the unit can start up to the minimum off time, plus 5 minutes, plus the hold time after
Home Assistant comes back.

To pause it (for example while you are away with the windows open), turn the automation off. If you switch the
plug off by hand while the automation is on, it is switched back on at a later check when the humidity is above
the top of the band (once the minimum off time has passed).

The band we use: on above 55%, off below 48%. The US EPA recommends keeping indoor humidity below 60%, ideally
between 30% and 50%.

## 6. Show it on a dashboard

A tile for the bucket: tap it for the level history, hold it to say you emptied the bucket (with a confirmation).

```yaml
type: tile
entity: sensor.bedroom_dehumidifier_bucket
name: Bucket
tap_action:
  action: more-info
hold_action:
  action: perform-action
  perform_action: script.bedroom_bucket_emptied
  confirmation:
    text: Did you empty the bedroom dehumidifier's bucket?
```

To see the numbers behind it, a Markdown card:

```yaml
type: markdown
content: >-
  {% set e = 'sensor.bedroom_dehumidifier_bucket' %}
  {% set level = states(e) %}
  **{{ level ~ '%' if level | is_number else 'Learning the bucket size' }}**
  {{ '(full)' if state_attr(e, 'full') else '' }}
  {{ (state_attr(e, 'energy_kwh') or 0) | round(2) }} of about {{ state_attr(e, 'capacity_kwh') or '?' }} kWh since
  {{ (state_attr(e, 'emptied_at') | as_datetime | as_local).strftime('%b %-d, %-I:%M %p') if state_attr(e, 'emptied_at') else 'it was set up' }}.
  Last buckets: {{ (state_attr(e, 'buckets') or []) | join(', ') or 'none yet' }} kWh.
```

## The bucket sensor's attributes

| Attribute | Meaning |
|---|---|
| `full` | `true` from when the bucket counts as full until it is emptied. |
| `energy_kwh` | Energy used since the bucket was last emptied (or since the sensor was set up), in kWh. |
| `capacity_kwh` | The bucket size in kWh: the average of `buckets`, or the known size you entered. Empty until known. |
| `buckets` | The energy of the most recent learned buckets, newest last, in kWh. |
| `pending_kwh` | The energy of the full bucket that will be learned when the unit starts again after it. Empty otherwise, including for a full bucket that will not be learned (its count did not start at a known empty, or it used less than "Smallest real bucket"). |
| `learned_at` | When the last bucket was learned; the "bucket emptied" script takes that bucket back within 10 minutes. |
| `emptied_at` | When the bucket was last emptied (by the compressor running again after a full stop, or the script). |
| `last_counter` | The plug's energy counter at the last reading, in kWh, used to work out the next difference. |
| `last_counter_at` | When `last_counter` was read, used to spot impossible jumps. |
| `low_since` | Since when the power has been below "Stopped below" with the plug on (empty while it is not). |
| `bucket_name` | The bucket name this sensor answers to. |

## Using it in your own automations

A lamp by the unit that is blue while its bucket is full, and off once it is emptied:

```yaml
triggers:
  - trigger: template
    value_template: "{{ state_attr('sensor.bedroom_dehumidifier_bucket', 'full') }}"
    id: full
  - trigger: template
    value_template: "{{ not state_attr('sensor.bedroom_dehumidifier_bucket', 'full') }}"
    id: emptied
actions:
  - if:
      - condition: trigger
        id: full
    then:
      - action: light.turn_on
        target:
          entity_id: light.bedroom_lamp
        data:
          color_name: blue
    else:
      - action: light.turn_off
        target:
          entity_id: light.bedroom_lamp
```

The learned bucket size as its own sensor, for a history graph of how it changes over the seasons. Add this item
under your existing `template:` list (a second `template:` key would replace the first one):

```yaml
  - sensor:
      - name: Bedroom bucket size
        unit_of_measurement: kWh
        state: "{{ state_attr('sensor.bedroom_dehumidifier_bucket', 'capacity_kwh') }}"
        availability: "{{ state_attr('sensor.bedroom_dehumidifier_bucket', 'capacity_kwh') is number }}"
```

## How accurate is it?

The level is an estimate from energy, not a measurement of the water. On the unit we built this for, three buckets
in a row took 14.2, 12.1 and 12.4 kWh: within about 10% of their average. Expect a bucket to take less energy in
warm, humid weather (the unit pulls more water per kWh) and more in cool, drier air, which is why the size is an
average of recent buckets and the "almost full" alert defaults to 85%. The "full" state is not an estimate: it
comes from the unit stopping by itself.

## Several dehumidifiers

Add one `use_blueprint` item per unit under `template:`, each with its own plug, `bucket_name`, `name` and
`unique_id`. Make one "bucket emptied" script and one alerts automation per unit; one humidity band per room.

## Restarts and power cuts

- **Once the bucket size is known,** the tracker keeps its count, learned buckets, full state and "stopped since"
  across Home Assistant restarts (or while the bucket is full, since the level is then 100).
- **Before that** (while the level is unknown), a restart starts the count again: Home Assistant does not restore
  a sensor whose state is unknown. The bucket in progress is then not learned, so the bucket size is never learned
  from a bucket that was only partly counted; learning waits for the next one. Entering a known bucket size
  avoids this.
- **While Home Assistant was down:** the tracker looks at the plug every minute, not only at changes, so within a
  minute of coming back it sees a unit that stopped (and marks the bucket full once the power has stayed low for
  "Stopped for" minutes), or a full bucket whose unit is running again (emptied).
- **Energy used while Home Assistant was down** is counted at the plug's next reading, as long as the counter did
  not start again in the meantime (with a daily counter, energy used before midnight while Home Assistant was
  down is missed).
- **After a power cut** the unit restarts by itself (see [What you need](#what-you-need)); if the bucket was full,
  it stays full until it is emptied.

## Starting the learning again

If the learned size is wrong (for example after a stop that was not a full bucket was learned), forget the learned
buckets: open **Developer tools, Events**, enter the event type `dehumidifier_bucket_emptied` and this event data,
then click **Fire event**:

```yaml
bucket: bedroom
forget_learned: true
```

This empties `buckets` and starts the count from 0, as an "emptied" would; empty the bucket first, so the next
full bucket is learned.

## Troubleshooting

- **The level stays unknown.** The tracker has not learned a bucket yet. Wait for a full bucket (the first one
  after setup only counts if you ran the "bucket emptied" script when you set it up), or enter a known size.
  Check `energy_kwh` in the sensor's attributes (**Developer tools, States**) to see that energy is being
  counted.
- **It says full, but the bucket is not.** The unit stopped for another reason: it was switched off at its own
  button, its own humidistat stopped it (set it to continuous), or it waits in standby after its power came back
  (see [What you need](#what-you-need)). Raise `full_minutes` if your unit pauses for a while during defrost.
- **It never says full.** Look at the plug's power while the bucket is full: if it is above `full_watts`, raise
  `full_watts` (it must stay below the fan-only power).
- **It does not notice the bucket was emptied.** The compressor's power never got above `running_watts`: lower it
  (it must stay above the fan-only power). Or run the "bucket emptied" script.
- **The level is far off.** A stop that was not a full bucket may have been learned. See
  [Starting the learning again](#starting-the-learning-again).
- **The plug turns back on after I switched it off.** The humidity band is on and the humidity is above the top
  of the band. Turn the automation off to pause it.
- **"clear_notification" arrived as a message.** Your notify action is not the Home Assistant app; turn off "Home
  Assistant app options" in the alerts automation.

## Limitations

- The level is an estimate from energy (see [How accurate is it?](#how-accurate-is-it)).
- A unit that stops for a reason other than a full bucket, with its plug on, looks like a full bucket: it shows
  full, the "full" alert is sent, and the stop resets the count when the unit runs again. If enough energy was
  counted, that stop is also learned as a bucket. Run the "bucket emptied" script after such a stop, or
  [start the learning again](#starting-the-learning-again) if a wrong size was learned.
- A bucket you empty without the unit having stopped (and without the script) is not noticed: the count keeps
  going, and the next full bucket is learned too large. Use the script, or the hold action on the dashboard
  tile.
- "Full" and "emptied" are noticed at the next minute, not the moment they happen.
- The tracker's settings live in `configuration.yaml`; Home Assistant has no UI for template blueprints.

## Updating and removing

- **Updating:** replace the blueprint files with the new versions and restart Home Assistant (for the automation
  and script blueprints, **Re-import blueprint** in the blueprint's menu also works). The bucket sensor keeps its
  count and learned buckets, as with any restart (see [Restarts and power cuts](#restarts-and-power-cuts)).
- **Removing:** delete the automations and scripts made from the blueprints, delete the `template:` entry that
  uses the bucket tracker, and restart. Because the sensor had a `unique_id`, its entity stays listed under
  **Settings, Devices & services, Entities** as unavailable; delete it there. Then delete the blueprint files.

## Security

- The blueprints use no webhooks and open nothing to the network: they read your plug and humidity sensor and
  switch your plug.
- Nothing is sent outside your Home Assistant except the notifications you configure.
- To report a security problem privately, use **Report a vulnerability** on this repository's
  [Security tab](../../security). Please do not open a public issue for security problems. See
  [SECURITY.md](SECURITY.md).

## Reporting problems and contributing

- **Something does not work:** open an [issue](../../issues) with your Home Assistant version, your plug, what
  you expected and what happened. The plug's power history around the problem and the bucket sensor's attributes
  help a lot; remove anything private from them first.
- **Your unit's numbers:** if you set it up on a unit, an issue with the model, its running, fan-only and stopped
  power and the kWh per bucket helps others pick their settings.
- **Improvements:** pull requests are welcome. Keep the blueprints self-contained (no custom integrations) and
  documented in this README.

## More free Home Assistant tools from SudoManaged

- [Webhook alerts](https://github.com/SudoManaged/ha-webhook-alerts)
- [Bluetooth self-heal](https://github.com/SudoManaged/ha-bluetooth-self-heal)
- [Yealink desk phone call log](https://github.com/SudoManaged/ha-yealink-call-log)
- [Internet failover alerts](https://github.com/SudoManaged/ha-internet-failover)

## Changelog and license

See [CHANGELOG.md](CHANGELOG.md). Released under the [MIT License](LICENSE). Copyright (c) 2026 SudoManaged.
