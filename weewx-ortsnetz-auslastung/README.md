# Uploader to Ortsnetz-Auslastung for WeeWX
Uploader for the Ortsnetzauslastung project

## Prerequisites

You need a WeeWX services that fetches data from your PV system. Refer to
the documentation of that service for the names of the observation types
of grid voltages and grid frequency.

Examples for such services are:
* [weewx-photovoltaics](https://github.com/roe-dl/weewx-photovoltaics)
* [weewx-snmp](https://github.com/roe-dl/weewx-snmp)
* [WeeWX-MQTTSubscribe](https://github.com/weewx-mqtt/subscribe)

## Maintained archive

This directory is part of the mirrored upstream repository. The catalog
archive **0.1.post1** adds a root-level installer and managed module path
for this WeeWX uploader. Its other repository files are not installed as
WeeWX services.
It targets WeeWX `>=5.5.2,<6` and Python `>=3.10,<3.15`.

## Install

Use `weectl` from the Python environment running WeeWX. With stock WeeWX 5,
install the published archive:

```sh
weectl extension install https://github.com/weewx-packages/catalog/releases/download/ortsnetz-auslastung/v0.1.post1/ortsnetz-auslastung-0.1.post1.tar.gz --config /path/to/weewx.conf
```

Replace `/path/to/weewx.conf` with your station configuration. You can also
download that URL and pass the local archive to the same command. Use elevated
permissions only when your station files require them.

The installer adds `user.ortsnetzauslastunguploader.RESTful` to WeeWX's REST
services. It needs no Python packages beyond WeeWX. Configure the observations
and device model in `weewx.conf`, not `skin.conf`. For a three-phase source
using the installer's observation names:

```ini
[StdRESTful]
    [[Ortsnetz-Auslastung]]
        smartmeter_model = YOUR_METER_MODEL
        l1_v = pmGridVoltageL1
        l2_v = pmGridVoltageL2
        l3_v = pmGridVoltageL3
        grid_frequency_hz = none
        plant_capacity_kwp = none
```

Replace these names with the voltage observations supplied by your collection
service. It must register the observations' units so WeeWX can convert them to
volts. Set `l2_v = none` and `l3_v = none` for a single-phase source.

Restart WeeWX yourself and inspect its log after an archive event. Set
`skip_upload = true` and `log_url = true` first to inspect the JSON without
posting. Set `skip_upload = false` when ready to submit it. The worker sends
HTTPS JSON to `https://www.ortsnetz-auslastung.de/v1/measurements`, without an
API key. It sends your WeeWX station latitude and longitude unless you set
`latitude` and `longitude` overrides in this section. Check the project's
[API](../API.md) before publishing location and grid data.

## Configuration instructions

The configuration section in `weewx.conf` looks like this:

```
[StdRESTful]
    ...
    [[Ortsnetz-Auslastung]]
        # enable or disable the uploader
        enable = true
        # dry run (skip the upload)
        skip_upload = false
        # minimum interval between uploads in seconds (at least 300)
        post_interval = 300
        # log data to send
        log_url = false
        # name of the measuring device
        smartmeter_model = replace_me
        # software (optional, default "WeeWX X.X.X, uploader X.X")
        #integration_version = replace_me
        # observation types to get the grid measurements from
        # (optional. What follows, are the defaults)
        l1_v = pmGridVoltageL1
        l2_v = pmGridVoltageL2
        l3_v = pmGridVoltageL3
        grid_frequency_hz = heataccuMainsFrequency
        plant_capacity_kwp = installedPVPeakPower
        # name of the observation type to get a forecast form
        # (optional, default no value)
        #pv_forecast_kwh = replace_me

...
[Engine]
    [[Services]]
        ...
        restful_services = ..., user.ortsnetzauslastunguploader.RESTful
```

* `enable`: Switch the uploader on and off. Optional. Default is on.
* `skip_upload`: If set to `true` run in dry run mode. Calculate data
  but do not upload them.
* `post_interval`: Minimum interval between uploads in seconds. Default is
  300; smaller values are limited to 300. The uploader runs on archive events,
  so a longer archive interval also means less frequent uploads.
* `log_url`: If set to `true` log data to send on syslog.
* `smartmeter_model`: Set it to the name of the measuring device that
  measures the grid voltages.
* `integration_version`: Normally you do **not** need this key. The 
  default is enough. Set it only, if you really know what you do.
* `l1_v`, `l2_v`, `l3_v`: Names of the observation types of the grid
  voltages. Mandatory. If you have a one phase system, set `l2_v` and 
  `l3_v` to `none`.
* `grid_frequency_hz`: Name of the observation type of the grid
  frequency. Optional. Set to `none` if your measuring device does not
  provide the frequency.
* `pv_forecast_kwh`: Name of the observation type that provides a 
  forecast of the daily earnings. Optional. Default is not provided.

## Tests

With WeeWX installed, run from the repository root:

```shell
python -m unittest discover -s weewx-ortsnetz-auslastung -p 'test_*.py'
```

The tests check dry-run behavior and the minimum upload interval without
making network requests.

## Links

Project websites of the Ortsnetz-Auslastung project:
* https://www.ortsnetz-auslastung.de/
* https://github.com/thomaslehmann1234/datenerfassung-ortsnetz-auslastung

WeeWX:
* [weather software WeeWX](https://weewx.com)
* [fetch data from E3/DC converter](https://github.com/roe-dl/weewx-photovoltaics)

## Limits and license

This is an uploader for existing observations, not a meter driver. Uploads
follow archive events with a minimum 300-second interval. Absent phases use
`-1`; optional frequency and PV values use Hz, kWp and kWh after conversion.
`log_url` also logs the payload, including coordinates. The service does not
create a separate timer or cron job.

Copyright 2026 Johanna Karen Roedenbeck. The uploader is GNU GPLv3 or later.
The surrounding project also includes MIT-licensed work; see the
[root license notes](../README.md#lizenz-und-herkunft).
