# MES Barcode Gateway

Python service for a Raspberry Pi on the production line. It reads several serial barcode scanners at once and forwards each scan to the [MES Daemon](../web/MES-Daemon/README.md) with the station number.

## Flow

```mermaid
flowchart LR
  A[Serial scanners] --> B[One thread per scanner] --> C[Check] --> D[Trigger] --> E[MES Daemon]
```

| File | Purpose |
| --- | --- |
| `mainv2.py` | Current reader |
| `main.py` | First, simpler reader |
| `OnRead/Check.py` | Accepts or rejects a scan |
| `OnRead/Trigger.py` | Sends the scan to the API |
| `settings_example.py` | Configuration template |
| `Dashboard/` | Web tool that maps USB ports to station numbers |

## Reliability

- Automatic reconnection, with stale bytes discarded after connecting.
- Accepts CR and LF endings; rejects scans that are too short, too long or joined together.
- Ignores the same barcode on one scanner for two seconds.
- Five-second request timeout. Network errors and 5xx answers are retried twice, 4xx answers are not.

## Configuration

Copy `settings_example.py` to `settings.py`.

| Setting | Meaning |
| --- | --- |
| `SCANNERS` | List of `PORT`, `PORT_BAUD` and `ID` (the MES `scanner_assignment`) |
| `SERVER_IP`, `SERVER_PORT`, `SERVER_ENDPOINT` | Daemon address, endpoint `api/hand-scanner` |

Stable device names under `/dev/serial/by-path/` avoid port changes after a reboot.

## Run

```bash
python3 -m venv venv && . venv/bin/activate
pip install -r requirements.txt
python mainv2.py
```

## Port mapping dashboard

`Dashboard/serial_mapper.py` is a Flask-SocketIO page. It detects serial devices, shows a QR code to scan, recognizes which port received it, lets you assign a station number and exports the mapping.

```bash
pip install -r Dashboard/requirements.txt
cd Dashboard && python serial_mapper.py
```
