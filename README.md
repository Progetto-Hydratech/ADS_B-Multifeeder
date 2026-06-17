# ADS-B Multifeeder

Docker Compose stack to feed your ADS-B data to **FlightRadar24**, **FlightAware**, and **ADS-B Exchange** simultaneously, starting from any Beast-compatible ADS-B receiver (e.g. [ultrafeeder](https://github.com/sdr-enthusiasts/docker-adsb-ultrafeeder)).

---

## Requirements

- Docker + Docker Compose
- A running ADS-B decoder exposing a Beast TCP output on port 30005 (e.g. ultrafeeder, dump1090, readsb)
- Keys/IDs for the networks you want to feed (see below)

---

## Quick Start

```bash
cp .env.example .env
nano .env        # fill in your keys and coordinates
docker compose up -d
```

---

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `BEASTHOST` | ✅ | Hostname or IP of your Beast source (e.g. `ultrafeeder`) |
| `BEASTPORT` | ✅ | Beast TCP port (default: `30005`) |
| `LAT` | ✅ | Antenna latitude (e.g. `43.7696`) |
| `LONG` | ✅ | Antenna longitude (e.g. `11.2558`) |
| `ALT` | ✅ | Antenna altitude in feet above sea level (e.g. `164`) |
| `FR24_KEY` | FR24 only | Your FlightRadar24 sharing key |
| `FLIGHTAWARE_FEEDER_ID` | FA only | Your FlightAware feeder UUID |
| `ADSBX_UUID` | ADSBX only | A UUID for ADS-B Exchange |
| `ADSBX_SITENAME` | ADSBX only | Name shown on the ADSBX map (default: `MyFeeder`) |
| `TZ` | ❌ | Timezone (default: `Europe/Rome`) |

---

## How to Get Your Keys

### FlightRadar24

FR24 requires a **sharing key** (16-character string) tied to your feeder. The key is assigned manually by the FR24 support team.

**Step 1** — Send an email to **support@fr24.com** with:
- Your name
- Your antenna coordinates (latitude, longitude, altitude in feet)
- A brief description of your setup (RTL-SDR + Docker, Linux, etc.)

**Step 2** — FR24 support will reply with your sharing key.

**Step 3** — Set the key in your `.env`:
```
FR24_KEY=a1b2c3d4e5f6g7h8
```

> ⚠️ One sharing key = one feeder at a time. If you have multiple receivers, request additional keys from support@fr24.com.

> ℹ️ The FR24 container requires GMT timezone internally — the `TZ` variable is ignored for FR24 (this is normal).

Once running you will see in the container logs:
```
[feed][n]XXXX@blender.prod.fr24.io:8099/UDP
[feed][n]connected via UDP
[feed][n]working
```

---

### FlightAware (PiAware)

FlightAware uses a **feeder UUID** to identify your station. The UUID is generated automatically on first run.

**Step 1** — Create a free account at [flightaware.com](https://www.flightaware.com).

**Step 2** — Start the stack **without** setting `FLIGHTAWARE_FEEDER_ID` (leave it empty). The container will connect to FlightAware and generate a new UUID automatically.

**Step 3** — Read the UUID from the container logs:
```bash
docker logs multifeeder-piaware | grep "my feeder ID"
```
You will see something like:
```
[piaware] my feeder ID is xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

**Step 4** — Claim the feeder to your account by visiting:
```
https://www.flightaware.com/adsb/piaware/claim/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```
Replace the UUID with the one from your logs.

**Step 5** — Save the UUID in your `.env` so it stays stable across restarts:
```
FLIGHTAWARE_FEEDER_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

> ℹ️ If you signed up with Google/Apple and have no password, the claim URL method above (Step 4) is the only way to associate the feeder to your account — no password needed.

Once running you will see in the container logs:
```
[piaware] encrypted session established with FlightAware
[piaware] my feeder ID is xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
[piaware] piaware has successfully sent several msgs to FlightAware!
```

---

### ADS-B Exchange

ADS-B Exchange does **not** require registration. You just need to generate a random UUID and pick a site name.

**Step 1** — Generate a UUID on Linux/macOS:
```bash
uuidgen
# output: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```
Or use an online generator like [uuidgenerator.net](https://www.uuidgenerator.net).

**Step 2** — Set it in your `.env`:
```
ADSBX_UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
ADSBX_SITENAME=MyFeeder
```

**Step 3** — After a few minutes, check your stats at:
```
https://www.adsbexchange.com/api/feeders/?feed=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Once running you will see in the container logs:
```
[adsbexchange-feed] Beast TCP input: Connection established: ultrafeeder port 30005
[adsbexchange-feed] BeastReduce TCP output: Connection established: feed1.adsbexchange.com port 30004
[mlat-client] Handshake complete
```

---

## Example `.env`

```env
# Beast source
BEASTHOST=ultrafeeder
BEASTPORT=30005

# Timezone
TZ=Europe/Rome

# Antenna position
LAT=xx.xxxx
LONG=xx.xxxx
ALT=xxx

# FlightRadar24
FR24_KEY=a1b2c3d4e5f6g7h8

# FlightAware
FLIGHTAWARE_FEEDER_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

# ADS-B Exchange
ADSBX_UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
ADSBX_SITENAME=MyFeeder
```

---

## Deploying on Portainer

If you use Portainer (recommended for Proxmox/Docker setups):

1. Go to **Stacks** → **Add stack**
2. Select **Repository** and point to this repo
3. Under **Environment variables**, add all the variables from the table above
4. Click **Deploy the stack**

> ⚠️ Never commit your `.env` file — it is excluded by `.gitignore`. Always set sensitive values (keys, coordinates) as environment variables in Portainer, not in the compose file.

---

## Integrating with HydraPlanes

This stack is designed to work alongside [HydraPlanes](https://github.com/Sebaf-26/plane-tracker), an ADS-B radar web UI with history and Italian special callsign detection.

Set `BEASTHOST` to the name of the ultrafeeder container in your HydraPlanes stack (default: `ultrafeeder`) and make sure both stacks share the same Docker network, or use the container name directly if on the same host.

---

## License

MIT
