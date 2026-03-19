# Install Payjoin Mailroom

## Stop existing ohttp
~~~
docker stop bob-ohttp-relay
systemctl stop nginx
systemctl disable nginx
~~~

## Docker compose
~~~
mkdir /root/payjoin-mailroom
cd /root/payjoin-mailroom
vi docker-compose.yml
~~~
~~~
services:
  payjoin-mailroom:
    image: payjoin/payjoin-mailroom:payjoin-mailroom-0.1.0
    restart: always
    command: ["--config", "/config/config.toml"]
    environment:
      RUST_LOG: "info"
    volumes:
      - ./config.toml:/config/config.toml:ro
      - ./data:/data
    ports:
      - "443:443"
    logging:
      driver: "json-file"
      options:
        max-size: "500m"
        max-file: "10"
~~~
~~~
vi config.toml
~~~
~~~
# Payjoin Mailroom recommended configuration
#
# Configuration can also be set via environment variables with the `PJ_`
# prefix.  Nested values use double underscores as separators, e.g.
# PJ_TELEMETRY__OPERATOR_DOMAIN="your-domain.example.com"

# Address and port to listen on
listener = "[::]:443"

# --- ACME TLS (requires `acme` feature) ---
[acme]
# Domain names for the TLS certificate
domains = ["pj.bobspacebkk.com"]
# Contact addresses for the ACME account
contact = ["mailto:teemie@satsdays.com"]

# --- Telemetry (requires `telemetry` feature) ---
[telemetry]
# OpenTelemetry Protocol (OTLP) endpoint to export telemetry to
endpoint = "https://otlp-gateway-prod-us-west-0.grafana.net/otlp"
# Authentication token for the OTLP endpoint (available upon request)
auth_token = "<base64 instanceID:token>"
# The domain you are running the payjoin-mailroom from.
# This serves as an identifier for metrics collection.
operator_domain = "your-domain.example.com"

# --- Access-control (requires `access-control` feature) ---
[access_control]
# ISO 3166-1 alpha-2 country codes whose requests should be blocked.
blocked_regions = ["CU", "IR", "KP", "SY"]

# --- V1 protocol ---
# Uncomment the [v1] section to enable V1 fallback support.
# (address screening requires `access-control` feature)
[v1]
# URL to periodically fetch an updated blocked-address list from.
blocked_addresses_url = "https://raw.githubusercontent.com/0xB10C/ofac-sanctioned-digital-currency-addresses/refs/heads/lists/sanctioned_addresses_XBT.txt"
~~~

~~~
docker compose up
~~~
