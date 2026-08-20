# Continuous Bandwidth Consumer

A lightweight Docker container for continuously generating Internet download traffic on a Raspberry Pi or other low-power Linux device.

The container downloads large public test files and immediately discards the received data by sending it to `/dev/null`. This makes it possible to generate sustained network traffic without continuously writing downloaded data to an SD card or other storage.

---

## Why does this exist?

Running a browser such as Chromium simply to generate network traffic is unnecessarily heavy on a Raspberry Pi. A browser consumes considerably more CPU and memory and introduces a graphical environment that is not needed for this purpose.

This project provides a much simpler alternative:

```text
Internet
   ↓
   curl
   ↓
network buffers
   ↓
/dev/null
   ↓
discarded
```

The downloaded payload is never saved as a file.

This makes the container useful when the objective is to keep a network connection busy for an extended period while keeping the system overhead low.

---

## Use cases

This container can be useful for:

- Long-term Internet connection testing
- ISP throughput and stability testing
- Router and firewall testing
- Ethernet and Wi-Fi testing
- VPN throughput testing
- Testing connections behind CGNAT
- Testing network equipment under sustained traffic
- Testing bandwidth limits
- Raspberry Pi network stress testing
- Testing whether throughput changes or degrades over time
- Exercising different Internet routes by downloading from geographically distributed endpoints

It is intentionally not presented as a conventional speed-test application.
A normal speed test measures connection performance for a short period.
This project is intended to generate **continuous, sustained network traffic**.

---

## Features

- Lightweight Alpine Linux container
- Uses `curl` instead of a graphical browser
- Downloads directly to `/dev/null`
- Does not accumulate downloaded files
- Sequentially processes multiple download endpoints
- Automatically continues to the next endpoint if one fails
- Automatically starts the next cycle after the last endpoint
- Configurable CPU limit
- Configurable memory limit
- No additional swap allocation
- Optional bandwidth limiting
- Docker log rotation
- Suitable for Raspberry Pi and other low-power systems

---

## How it works

The container maintains a list of large public test files:

```text
Endpoint 1
   ↓
Endpoint 2
   ↓
Endpoint 3
   ↓
Endpoint 4
   ↓
...
   ↓
Endpoint N
   ↓
repeat
```

Each file is downloaded completely before the next endpoint is attempted.

If an endpoint cannot be reached or the download fails, `curl` returns an error and the shell continues with the next endpoint.

The downloaded data is sent to:

`/dev/null`

Therefore a 10 GB download does not create a 10 GB file on the Raspberry Pi.

---

## Requirements

- Docker
- Docker Compose
- Internet connection
- A Raspberry Pi or Linux system

The container itself does not require a graphical environment.

---

## Installation

1. Create a working directory for the project:

```bash
mkdir bandwidth-consumer
cd bandwidth-consumer
```

2. Download this repository's `docker-compose.yml` file

```bash
curl -L -O https://raw.githubusercontent.com/C-Man-The-Man/bandwidth-consumer/main/docker-compose.yml
```

3. Start the container:

```bash
docker compose up -d
```

4. Check the container:

```bash
docker compose ps
```

---

## Monitoring

1. View the container logs:

```bash
docker compose logs -f
```

The log will show which endpoint is currently being downloaded:

```text
bandwidth-consumer | Downloading: https://fsn1-speed.hetzner.com/10GB.bin
```

2. Monitor resource usage:

```bash
docker stats bandwidth-consumer
```

Example:

```text
CONTAINER ID   NAME                 CPU %   MEM USAGE / LIMIT   MEM %   NET I/O
xxxxxxxxxxxx   bandwidth-consumer   4.5%    3.2MiB / 256MiB     1.2%    179MB / 3.2MB
```

`NET I/O` should continuously increase while traffic is being received.

---

## Storage usage

The downloaded files are not stored.

The important part of the curl command is:

```
-o /dev/null
```

This discards the received data immediately.

For example, downloading a 10 GB file results in approximately 10 GB of network traffic but does not create a 10 GB file on the SD card.

Docker logging is also limited:

```bash
logging:
  driver: json-file
  options:
    max-size: "1m"
    max-file: "2"
```

This prevents container logs from growing indefinitely.

---

## Bandwidth limiting

`curl` supports an optional download-rate limit with `--limit-rate`.

To limit the bandwidth, add the option to the `curl` command.

For approximately 40 Mbps:

`--limit-rate 5M \`

For approximately 80 Mbps:

`--limit-rate 10M \`

For approximately 200 Mbps:

`--limit-rate 25M \`

For approximately 400 Mbps:

`--limit-rate 50M \`

For approximately 800 Mbps:

`--limit-rate 100M \`

`curl` uses bytes per second for this option, so:

```text
1M  ≈ 8 Mbps
5M  ≈ 40 Mbps
10M ≈ 80 Mbps
25M ≈ 200 Mbps
50M ≈ 400 Mbps
100M ≈ 800 Mbps
```

If no `--limit-rate` option is specified, `curl` is allowed to use as much bandwidth as the connection and remote server provide.

---

## Resource limits

The default Compose configuration limits the container to:

```bash
cpus: "0.25"
mem_limit: 256m
memswap_limit: 256m
```

This means:

- Maximum CPU: 25% of one CPU core
- Maximum RAM: 256 MB
- Additional swap: disabled for this container

The container normally uses considerably less memory than the configured limit.

These limits make it possible to run the bandwidth consumer alongside other services on a Raspberry Pi without allowing it to consume unlimited CPU or memory.

---

## Changing the endpoints

The download URLs are defined here:

```bash
urls="
https://fsn1-speed.hetzner.com/10GB.bin
https://nbg1-speed.hetzner.com/10GB.bin
https://hel1-speed.hetzner.com/10GB.bin
https://gra.proof.ovh.net/files/10Gb.dat
"
```

Additional endpoints can be added one per line.

The container processes them sequentially:

`Endpoint 1 → Endpoint 2 → Endpoint 3 → Endpoint 4 → repeat`

There is no intentional parallel downloading in the default configuration.

If an endpoint fails, the container proceeds to the next endpoint.

---

## Additional commands

- Stopping the container

```bash
docker compose down
```

- Start it again:

```bash
docker compose up -d
```

- Restart it:

```bash
docker compose restart
```

---

## Important considerations

The download servers used by this project are public test endpoints. Their availability, bandwidth, URLs, file sizes, and usage policies can change.

A public test server should not be assumed to provide unlimited bandwidth indefinitely.

Use reasonable limits when appropriate and check the terms of the endpoint providers before running sustained traffic for long periods.

This project is intended for legitimate network testing and infrastructure monitoring.

---

## Donations

**Bitcoin wallet address**
```text
bc1qpcfex53u7mqx4dc25gw7j7446amw9vn6743cn5
```

**EVM / Metamask  (ETH, ETC, OCTA, POL, PEAQ, MONAD, BASE etc.)**
```text
0xbE4879888d95B02B2FCaed2FcAeBbcf36829BDC9
```

**Solana wallet address**
```text
7EHWvShXfjLJ2HhzTf4CsHgjKckivfMQMjnEoUAEqau
```

**Sui wallet address**
```text
0x421a5a462f99c2d675d035d0c741ba5765a47c1e28f95d33ad770cd34a36a6ea
```

**Thank you!**
