# Synology Health - DSM 7 Plugin for CheckMK 2.4

This plugin enables monitoring of Synology NAS systems via the official DSM 7 Web API.
We needed to use this instead of SNMP, because of latency and bandwidth problem of a NAS which is located on the other side of the world :-)

<img width="1353" height="238" alt="image" src="https://github.com/user-attachments/assets/c8c3fae8-72df-43c4-8d13-20db2da3df94" />

## Features

- **RAID/Volume Status** - Monitor all storage pools and volumes with usage statistics
- **Disk Status** - Individual hard drives including temperature, S.M.A.R.T. status, and error counts
- **System Info** - Model, firmware version, uptime, CPU and RAM info
- **Hardware Status** - System temperature monitoring
- **System Health** - Overall health status and crash detection
- **Utilization** - CPU usage with load averages, RAM usage with details
- **Network Interfaces** - Status, speed, and IP configuration
- **UPS Status** - UPS monitoring (if enabled)

## Requirements

### Synology NAS
- DSM 7.0 or higher
- Admin account with API access
- Web API enabled (enabled by default)

### CheckMK
- Version 2.3.0p1 or higher (optimized for 2.4.x)

## Installation

### Method 1: MKP Package (Recommended)

```bash
# As site user
su - SITENAME

# Add and enable the package
mkp add synology_health-1.2.0.mkp
mkp enable synology_health 1.2.0

# Restart Apache
omd restart apache
```

### Method 2: Via Web Interface

1. Go to **Setup → Maintenance → Extension packages**
2. Click **Upload package**
3. Select the `.mkp` file
4. Activate changes

## Configuration

### 1. Create Synology User (Recommended)

Create a dedicated monitoring user on the NAS:

1. Open DSM → **Control Panel** → **User & Group**
2. Create new user (e.g., `checkmk_monitor`)
3. Add to **administrators** group
4. Set a strong password
5. Optionally disable 2-factor authentication for this account

> **Note:** Admin privileges are required for full API access.

### 2. Create Host in CheckMK

1. **Setup → Hosts → Add host**
2. Enter hostname and IP address of the NAS
3. Save and activate changes

### 3. Create Special Agent Rule

1. **Setup → Agents → Other integrations → Server hardware → Synology Health**
2. Click **Add rule** and configure:

| Setting | Value |
|---------|-------|
| Username | `checkmk_monitor` |
| Password | (your password) |
| Port | `5001` (HTTPS) or `5000` (HTTP) |
| Protocol | HTTPS (recommended) |
| SSL Verification | Skip verification for self-signed certs |

3. Under **Conditions**, set the rule to match your Synology host
4. Save and activate changes

### 4. Service Discovery

1. Open the host → **Service configuration**
2. Click **Full service scan**
3. Accept the new services
4. Activate changes

## Monitored Services

After discovery, the following services are created:

| Service | Description |
|---------|-------------|
| **Synology System Info** | Model, firmware, uptime, CPU/RAM info |
| **Synology Health** | Overall system health, crash detection |
| **Synology Pool {name}** | Storage pool status, RAID type, disk count |
| **Synology Volume {name}** | Volume status with usage (used/total/free) |
| **Synology Disk Drive {n}** | Disk status, model, temperature, S.M.A.R.T. |
| **Synology Temperature System** | System temperature sensor |
| **Synology CPU** | CPU utilization with user/system/other, load averages |
| **Synology Memory** | RAM utilization with used/available/cached |
| **Synology Network {interface}** | Interface status, speed, IP address |
| **Synology UPS** | UPS status, charge, runtime (if enabled) |

## Disk Monitoring Details

The **Synology Disk** service monitors:

- **Disk Status**: normal, initialized, crashed, etc.
- **S.M.A.R.T. Status**: healthy, warning, critical, failing
- **Temperature**: with thresholds (Warning: 45°C, Critical: 55°C)
- **Uncorrectable Errors (UNC)**: bad sector count
- **SSD Remaining Life**: percentage remaining (for SSDs)
- **Vendor/Model/Serial**: disk identification
- **API Threshold Warnings**: Synology's built-in health alerts

## Thresholds

| Check | Warning | Critical |
|-------|---------|----------|
| CPU Usage | 80% | 95% |
| Memory Usage | 80% | 95% |
| System Temperature | 50°C | 60°C |
| Disk Temperature | 45°C | 55°C |
| Swap Usage | 50% | - |
| SSD Remaining Life | 20% | 10% |

## Troubleshooting

### Test Agent Manually

```bash
# As site user
su - SITENAME

# Run agent directly
./local/lib/python3/cmk_addons/plugins/synology_health/libexec/agent_synology_health \
  --host 192.168.1.100 \
  --username admin \
  --password 'yourpassword' \
  --no-verify-ssl
```

### Common Problems

**Error: "Failed to authenticate"**
- Verify username and password
- Check if 2-factor authentication is enabled (disable or use app password)
- Check for account lockout after failed login attempts
- Ensure the user is in the administrators group

**Error: SSL Certificate**
- Use `--no-verify-ssl` option in the rule, or
- Install a valid SSL certificate on the NAS

**No Services Found**
- Check agent output manually (see above)
- Verify the host is reachable on port 5001 (HTTPS) or 5000 (HTTP)
- Check firewall rules

**Services Show UNKNOWN**
- Run a full service scan to refresh
- Check if API responses have changed (run agent manually)

### Check Agent Output

```bash
# View full agent output
./agent_synology_health --host <IP> --username <user> --password '<pass>' --no-verify-ssl

# Check specific sections
./agent_synology_health ... | grep -A20 "<<<synology_storage"
./agent_synology_health ... | grep -A10 "<<<synology_utilization"
```

## API Documentation

The plugin uses the following Synology DSM 7 APIs:

| API | Purpose |
|-----|---------|
| `SYNO.API.Auth` | Authentication and session management |
| `SYNO.Core.System` | System information (model, firmware, CPU, RAM) |
| `SYNO.Core.System.Status` | Health status and crash detection |
| `SYNO.Storage.CGI.Storage` | Storage pools, volumes, and disk information |
| `SYNO.Core.System.Utilization` | CPU and memory utilization |
| `SYNO.Core.Network.Interface` | Network interface status |
| `SYNO.Core.ExternalDevice.UPS` | UPS information |

## Updating the Plugin

```bash
# Disable old version
mkp disable synology_health

# Remove old version
mkp remove synology_health <old_version>

# Add new version
mkp add synology_health-<new_version>.mkp
mkp enable synology_health <new_version>

# Restart
omd restart apache
```

After updating, run a **Full service scan** on your Synology hosts to detect any new or renamed services.

## License

MIT License

## Author

Michael Righter

## Changelog

### 1.2.0
- initial public release
- Support for DSM 7.x
- CheckMK 2.4 compatible
