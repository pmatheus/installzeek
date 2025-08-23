# Zeek with GeoIP Installation Script

Automated installation script for [Zeek Network Security Monitor](https://zeek.org/) with MaxMind GeoIP database integration on Ubuntu 24.04 LTS.

## Features

- Installs Zeek from the official OpenSUSE Build Service repository
- Configures MaxMind GeoLite2 databases (City & ASN)
- Installs and configures the `geoip-conn` package for connection log enrichment
- Auto-detects network interface for monitoring
- Provides geo-enriched connection logs with country, city, coordinates, and ASN data

## Prerequisites

- Ubuntu 24.04 LTS (Noble Numbat)
- Root or sudo access
- MaxMind account for GeoLite2 databases (free tier available)
- Active network interface for packet capture

## MaxMind Setup

1. Create a free MaxMind account at https://www.maxmind.com/en/geolite2/signup
2. Generate a license key:
   - Log into your MaxMind account
   - Navigate to "My License Keys" under Services
   - Click "Generate new license key"
   - Save your Account ID and License Key

## Installation

### Quick Install

```bash
# Clone the repository
git clone https://github.com/pmatheus/installzeek.git
cd installzeek

# Set MaxMind credentials as environment variables
export MAXMIND_ACCOUNT_ID="your_account_id"
export MAXMIND_LICENSE_KEY="your_license_key"

# Run the installation script
sudo -E bash install.sh
```

### Custom Network Interface

By default, the script auto-detects the primary network interface. To specify a different interface:

```bash
# Edit the script before running
sed -i 's/IFACE="eno2"/IFACE="eth0"/' install.sh

# Or modify the node.cfg after installation
sudo vim /opt/zeek/etc/node.cfg
```

## What Gets Installed

- **Zeek**: Network security monitoring framework at `/opt/zeek/`
- **GeoIP Databases**: MaxMind GeoLite2 databases at `/usr/share/GeoIP/`
- **geoip-conn package**: Zeek package for geo-enrichment of connection logs
- **Dependencies**: libpcap, libmaxminddb, geoipupdate

## Configuration Files

- `/opt/zeek/etc/node.cfg` - Zeek node configuration (interface settings)
- `/opt/zeek/share/zeek/site/local.zeek` - Site-specific Zeek policies
- `/etc/GeoIP.conf` - MaxMind database update configuration

## Usage

### Start/Stop Zeek

```bash
# Check status
sudo /opt/zeek/bin/zeekctl status

# Start Zeek
sudo /opt/zeek/bin/zeekctl start

# Stop Zeek
sudo /opt/zeek/bin/zeekctl stop

# Restart Zeek
sudo /opt/zeek/bin/zeekctl restart

# Deploy configuration changes
sudo /opt/zeek/bin/zeekctl deploy
```

### View Logs

Zeek logs are stored in `/opt/zeek/logs/current/`:

```bash
# View connection logs with geo data
tail -f /opt/zeek/logs/current/conn.log

# Parse JSON logs (if JSON output is enabled)
cat /opt/zeek/logs/current/conn.log | jq '.'
```

### Update GeoIP Databases

The databases are updated automatically via systemd timer, or manually:

```bash
sudo geoipupdate
```

## Log Enrichment

The `geoip-conn` package adds nested geo fields to `conn.log`:

- `orig_geo.country_code` - Source country code
- `orig_geo.region` - Source region/state
- `orig_geo.city` - Source city
- `orig_geo.latitude/longitude` - Source coordinates
- `resp_geo.*` - Destination geo data
- `orig_asn/resp_asn` - Autonomous System information

Example enriched connection log entry:
```json
{
  "ts": 1704067200.123456,
  "uid": "CAbcDe1FgHiJ2KlMnO",
  "id.orig_h": "192.168.1.100",
  "id.resp_h": "8.8.8.8",
  "orig_geo": {
    "country_code": "US",
    "region": "California",
    "city": "Mountain View",
    "latitude": 37.4056,
    "longitude": -122.0775
  },
  "orig_asn": {
    "number": 15169,
    "organization": "Google LLC"
  }
}
```

## Troubleshooting

### Check Zeek Installation
```bash
/opt/zeek/bin/zeek --version
```

### Verify GeoIP Support
```bash
/opt/zeek/bin/zeek -e 'print lookup_location(8.8.8.8);'
```

### View Zeek Logs
```bash
sudo /opt/zeek/bin/zeekctl diag
```

### Network Interface Issues
```bash
# List available interfaces
ip link show

# Check interface status
ip addr show dev <interface>
```

### MaxMind Database Issues
```bash
# Check if databases exist
ls -la /usr/share/GeoIP/

# Test database update
sudo geoipupdate -v
```

## Security Considerations

- Store MaxMind credentials securely (use environment variables, not hardcoded values)
- Regularly update GeoIP databases for accuracy
- Monitor disk space as Zeek logs can grow quickly
- Consider log rotation policies for production deployments
- Review captured traffic for sensitive data before sharing logs

## License

This installation script is provided as-is for educational and defensive security purposes.

## Contributing

Issues and pull requests are welcome at https://github.com/pmatheus/installzeek

## Resources

- [Zeek Documentation](https://docs.zeek.org/)
- [MaxMind GeoLite2](https://dev.maxmind.com/geoip/geolite2-free-geolocation-data)
- [Zeek Package Manager](https://packages.zeek.org/)
- [geoip-conn Package](https://github.com/brimsec/geoip-conn)