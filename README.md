# Synology Health - DSM 7 Monitoring Plugin for CheckMK 2.4

This plugin enables monitoring of Synology NAS systems via the official DSM 7 Web API.
<img width="1353" height="238" alt="image" src="https://github.com/user-attachments/assets/c8c3fae8-72df-43c4-8d13-20db2da3df94" />


## Features

- **RAID/Volume Status** - Monitor all storage pools and volumes
- **Disk Status** - Individual hard drives including temperature and S.M.A.R.T.
- **System Info** - Model, firmware, uptime
- **Hardware Status** - Temperature sensors
- **Utilization** - CPU, RAM, Network
- **UPS Status** - UPS monitoring (if available)

## Requirements

### Synology NAS
- DSM 7.0 or higher
- Admin account with API access
- Web API enabled (enabled by default)

### CheckMK
- Version 2.4.0 or higher

## Installation

### Method 1: MKP Package (Recommended)

```bash
# As site user
su - SITENAME

# Add and enable the package
mkp add synology_health-1.1.0.mkp
mkp enable synology_health 1.1.0

# Restart Apache
omd restart apache
```

### Method 2: Via Web Interface

1. Go to Setup → Maintenance → Extension packages
2. Click "Upload package"
3. Select the `.mkp` file
4. Activate changes

## Configuration

### 1. Create Synology User (Recommended)

Create a dedicated monitoring user on the NAS:

1. Open DSM → Control Panel → User & Group
2. Create new user (e.g., `checkmk_monitor`)
3. Add to **administrators** group
4. Set password

> **Note:** Admin privileges are required for full API access.

### 2. Create Host in CheckMK

1. Setup → Hosts → Add host
2. Enter hostname and IP address of the NAS

### 3. Create Special Agent Rule

1. Setup → Agents → Other integrations → Server hardware → Synology Health
2. Create new rule:

| Setting | Value |
|---------|-------|
| Username | `checkmk_monitor` |
| Password | (your password) |
| Port | `5001` (HTTPS) or `5000` (HTTP) |
| Protocol | HTTPS (recommended) |
| SSL Verification | Depends on certificate |

3. Set condition to match your Synology host
4. Save and activate changes

### 4. Service Discovery

1. Open host → Service configuration
2. Run "Full service scan"
3. Activate new services

## Monitored Services

After discovery, the following services are created:

| Service | Description |
|---------|-------------|
| Synology System Info | Model, firmware, uptime, CPU info |
| Synology Health | Overall system health status |
| Synology Pool {name} | Status per storage pool |
| Synology Volume {name} | Status per volume with usage |
| Synology Disk {name} | Status per hard drive with temp |
| Synology Temperature System | System temperature sensor |
| Synology CPU | CPU utilization with load averages |
| Synology Memory | RAM utilization |
| Synology Network {interface} | Network interface status |
| Synology UPS | UPS status (if enabled) |

## Troubleshooting

### Test Agent Manually

```bash
# As site user
su - SITENAME

# Run agent directly
./local/lib/python3/cmk_addons/plugins/synology_health/libexec/agent_synology_health \
  --host 192.168.1.100 \
  --username admin \
  --password 'secret' \
  --no-verify-ssl
```

### Common Problems

**Error: "Failed to authenticate"**
- Check username/password
- Disable 2-factor authentication or use app password
- Check for account lockout after too many failed attempts

**Error: SSL Certificate**
- Use `--no-verify-ssl` option, or
- Install valid certificate on the NAS

**No Services Found**
- Check agent output (see above)
- Verify API responses in debug mode

### Debug Mode

```bash
./agent_synology_health --host ... --debug 2>&1 | less
```

## API Documentation

The plugin uses the following Synology APIs:

| API | Purpose |
|-----|---------|
| SYNO.API.Auth | Authentication |
| SYNO.Core.System | System information |
| SYNO.Core.System.Status | Health status |
| SYNO.Storage.CGI.Storage | RAID/Volume info |
| SYNO.Core.System.Utilization | CPU/RAM |
| SYNO.Core.Network.Interface | Network |
| SYNO.Core.ExternalDevice.UPS | UPS |

## Thresholds

| Check | Warning | Critical |
|-------|---------|----------|
| CPU | 80% | 95% |
| Memory | 80% | 95% |
| System Temperature | 50°C | 60°C |
| Disk Temperature | 45°C | 55°C |
| Swap Usage | 50% | - |

## License

MIT License

## Changelog

### 1.1.0
- Initial public Release
- Support for DSM 7.x
- CheckMK 2.4 compatible
