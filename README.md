# Zabbix Template: Libre Hardware Monitor for Windows

This Zabbix 7.0 template allows you to monitor Windows desktops and laptops using **Libre Hardware Monitor** via its built-in HTTP server. It automatically discovers all hardware sensors (Temperature, Load, Fan speed, Voltage, Clock frequency, Power, Storage data, etc.) and creates items with correct units.

## Features

- **Automatic Discovery (LLD)**: Uses JavaScript preprocessing to traverse the hierarchical JSON structure of Libre Hardware Monitor.
- **Configurable Triggers**: Temperature triggers for CPU, GPU, and HDD with user-defined thresholds.
- **Filterable Alerts**: Support for ignore-lists to skip "limit" or "status" sensors (e.g., "Warning Temperature") that shouldn't trigger alerts.

## 1. Setting up Libre Hardware Monitor

1.  **Download**: Get the latest version from [Libre Hardware Monitor](https://github.com/LibreHardwareMonitor/LibreHardwareMonitor).
2.  **Extract**: Extract the ZIP file to a permanent location (e.g., `C:\Program Files\LibreHardwareMonitor`).
3.  **Run as Administrator**: Right-click `LibreHardwareMonitor.exe` and select **Run as Administrator** (required to access all hardware sensors).

or

install it with Winget
```
winget install LibreHardwareMonitor.LibreHardwareMonitor
```

### Configuration for Auto-Start

To ensure continuous monitoring, configure these settings within the application:
1.  **Options** -> **Run On Windows Startup**: Enable.
2.  **Options** -> **Minimize On Close**: Enable.
3.  **Options** -> **Start Minimized**: Enable.
4.  **Options** -> **Minimize To Tray**: Enable.

### Activate Web Server

The Zabbix template fetches data from the LHM web server:
1.  Go to **Options** -> **Remote Web Server** -> **Run**.
2.  (Optional) Go to **Options** -> **Remote Web Server** -> **Port** to change the default port (`8085`). If you change it, update the Master Item in the Zabbix template.

## 2. Windows Firewall Configuration

You must allow incoming connections to the Libre Hardware Monitor port (default: `8085`).

1.  Open **Windows Defender Firewall with Advanced Security**.
2.  Select **Inbound Rules** and click **New Rule...**
3.  **Rule Type**: Port.
4.  **Protocol and Ports**: TCP, Specific local ports: `8085`.
5.  **Action**: Allow the connection.
6.  **Profile**: Domain, Private (and Public if necessary).
7.  **Name**: "Zabbix via Libre Hardware Monitor Web Server".

## 3. Zabbix Template Installation

1.  **Import**: In Zabbix web interface, go to **Data collection** -> **Templates** -> **Import**.
2.  **Select File**: Choose downloaded yaml template.
3.  **Assign to Host**:
    - Link the template to your Windows host.
    - Ensure the host has an interface with the correct IP address/DNS name.
    - The template uses `{HOST.CONN}` to connect to LHM.

## 4. User Macros

You can customize thresholds and ignore-lists using the following macros on the template or host level:

| Macro | Default Value | Description |
| :--- | :--- | :--- |
| `{$LHM.CPU.TEMP.MAX}` | `80` | CPU temperature threshold for HIGH alert. |
| `{$LHM.GPU.TEMP.MAX}` | `85` | GPU temperature threshold for HIGH alert. |
| `{$LHM.HDD.TEMP.MAX}` | `60` | HDD temperature threshold for WARNING alert. |
| `{$LHM.CPU.TEMP.IGNORE}`| `Warning\|Critical` | Regex for CPU sensors to ignore for triggers. |
| `{$LHM.GPU.TEMP.IGNORE}`| `Warning\|Critical` | Regex for GPU sensors to ignore for triggers. |
| `{$LHM.HDD.TEMP.IGNORE}`| `Warning\|Critical` | Regex for HDD sensors to ignore for triggers. |

## 5. Troubleshooting

- **No Data**: Verify that the LHM Web Server is running and accessible from the Zabbix Server/Proxy using `curl http://<CLIENT_IP>:8085/data.json`.
- **Permission Denied**: Ensure Libre Hardware Monitor is running with **Administrator privileges**.
- **JSONPath Errors**: Ensure you are using Zabbix 7.0 or newer, as this template utilizes latest schema features.
