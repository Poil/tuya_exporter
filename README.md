# Tuya Exporter

A bare-bone Prometheus exporter for Tuya smart plugs and heaters, using
[tinytuya](https://github.com/jasonacox/tinytuya) over the local network. It
exports the following gauges, labelled with the device `name`:

- `tuya_consumption_current` (in amps)
- `tuya_consumption_power` (in watts)
- `tuya_consumption_voltage` (in volts)

Supported device categories (from `devices.json`):

| Category | Devices | DPs (current / power / voltage) |
|----------|---------|---------------------------------|
| `cz`     | smart plugs | 18 / 19 / 20 |
| `wk`     | heaters / thermostats | 108 / 109 / 110 |
| other    | fallback | 21 / 22 / 23 |

The protocol version (3.3, 3.4, ...) is read from each device's `version` in
`devices.json`.

## Install

```
$ pip install --user -r requirements.txt
```

## Configuration

Generate `devices.json` (device IDs, IPs, local keys, versions, categories)
with the tinytuya wizard:

```
$ python3 -m tinytuya wizard
```

`devices.json` contains the local keys of your devices: keep it out of git.

## Usage

Export the devices selected with `-d` (repeatable) on the given port:

```
$ tuya_exporter.py -c devices.json -d <device_id> -d <device2_id> -p 9185
```

Devices are polled on each Prometheus scrape, so keep a handful of devices
per instance and scrape at a reasonable interval.

### systemd

`tuya_exporter@.service` runs one instance per group of devices. Each instance
reads `/home/tuya/tuya_exporter/tuya_exporter_<name>.conf` (see
`tuya_exporter.conf.example`):

```
$ sudo cp tuya_exporter@.service /etc/systemd/system/
$ cp tuya_exporter.conf.example tuya_exporter_<name>.conf   # edit DEVICES, PORT, CONFIG
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now tuya_exporter@<name>
```

### Docker

```
$ docker build -t tuya_exporter .
$ docker run -p 9185:9185 -v $PWD/devices.json:/devices.json \
    tuya_exporter -c /devices.json -d <device_id> -p 9185
```

Tuya devices are reached over the local network and the container must be able
to reach them.
