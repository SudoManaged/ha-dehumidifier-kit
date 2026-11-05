# Changelog

All notable changes. Versions follow [Semantic Versioning](https://semver.org/).

## 1.0.0 (2026-11-05)

First public release.

- **Bucket tracker** (template blueprint): the bucket level in % from the plug's energy. Learns the bucket size
  from full buckets counted from a known empty (at least "Smallest real bucket"), when the unit starts again after
  them; the "emptied" script discards a full bucket, or takes back one learned in the last 10 minutes, so a bucket
  taken out early is not learned. Notices full and emptied by itself by looking at the plug every minute,
  including after a Home Assistant restart, and keeps its "stopped" timer through a moment of unavailable
  readings; works with lifetime and daily energy counters in kWh or Wh, including fine-grained ones; ignores
  counter noise and impossible jumps; keeps its count across restarts once the bucket size is known.
- **Bucket emptied** (script blueprint): resets the count when the bucket is emptied early, without learning from
  the partly full bucket. The same event with `forget_learned: true` starts the learning again.
- **Bucket alerts** (automation blueprint): almost full and full notifications with one tag per dehumidifier,
  cleared when the bucket is emptied, not repeated after a reload of template entities; Home Assistant app
  options can be turned off for other notify services.
- **Humidity band** (automation blueprint): on above / off below, hold time (also for the checks every 5 minutes
  and at start), minimum off time.
- Requires Home Assistant 2025.8 or newer. Tested on 2025.8.0, 2026.3.3 and 2026.11.0: the configuration check,
  and a live run with a simulated plug and humidity sensors covering:
  - setup without the "emptied" script (first full bucket not learned), emptied by the compressor starting
    again, learning from a known empty when the unit starts again;
  - a daily counter starting again, a small dip in the counter (ignored), a glitch to 0 followed by an
    impossible jump (ignored), an energy sensor in Wh, including readings near zero (5e-05 kWh) and readings
    of a third of a Wh that add up;
  - the power sensor unavailable for a moment during a stop (full still found on time), and a plug that reads
    slightly negative power when stopped;
  - the almost-full alert, the full alert, both cleared when the bucket is emptied, and no repeat after a reload
    of template entities;
  - emptied early with the script, a plug switched on with a full bucket, a stop with too little energy (not
    learned), a bucket taken out early and left out until it showed full, then reset with the script before
    the unit started again (not learned) and after it started again (taken back), forgetting the learned
    buckets;
  - a Home Assistant restart while full (still full, then emptied) and a restart 30 seconds after the unit
    stopped (full found afterwards);
  - the humidity band: on, off, a start held back by the minimum off time and made at the 5-minute check, a
    short dip across a 5-minute check ignored thanks to the hold time, a lasting drop switching off, and the
    check when Home Assistant starts.
