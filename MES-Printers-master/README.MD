# MESP – Network Label Printer Sender

> Part of the [MES platform](../README.md). Called by the [MES Daemon](../web/MES-Daemon/README.md) to print labels on network printers. Addresses are placeholders.

`mesp` is a lightweight Python utility used to send PDF labels directly to Zebra-compatible network printers.  
It converts PDF files into ZPL format and sends them over TCP (port `9100`) without using CUPS.

Designed for MES environments where each post (station) is mapped to a specific printer and label format.

---

## Features

- Direct PDF → ZPL printing
- Multiple printer support via configuration
- Different label converters per printer
- Simple CLI interface
- No external printing services required

---

## Requirements

- Python **3.x**
- Network access to Zebra-compatible printers
- Printers listening on port `9100`
- A `converters` module providing:
  - `convert_label_small`
  - `convert_label_big`

---

## Project Structure

```

.
├── main.py
├── converters.py
└── README.md

````

---

## Printer Configuration

Printers are defined in `PRINTERS_CONFIG` using a **post ID** as the key.

Each printer entry includes:
- `address`: Printer IP and port
- `converter`: Function used to convert the PDF into ZPL

Example:
```python
"13": {
    "address": ("192.0.2.42", 9100),
    "converter": convert_label_big
}
````

---

## Usage

### Command Line

```bash
mesp <post_id> <pdf_file>
```

Example:

```bash
mesp 13 label.pdf
```

---

## Required Alias

To use `mesp` as a command, you **must** define the following alias:

```bash
alias mesp="python3 /path/main.py"
```

Add it to your shell configuration file:

* `~/.bashrc`
* or `~/.zshrc`

Then reload:

```bash
source ~/.bashrc
```

---

## Exit Codes

* `0` → Print sent successfully
* `1` → Error (invalid arguments, file not found, unknown post, or printer connection failure)

---

## How It Works

1. Reads the PDF file as binary
2. Converts it to ZPL using the assigned converter
3. Opens a TCP socket to the printer
4. Sends the ZPL payload
5. Closes the connection

---

## Notes

* Socket timeout is set to **10 seconds**
* Errors are intentionally silent for easy integration into scripts and pipelines
* Suitable for MES backends, cron jobs, or automation tools

---