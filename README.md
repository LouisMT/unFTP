# unFTP

[![Build Status](https://travis-ci.org/bolcom/unFTP.svg)](https://travis-ci.org/bolcom/unFTP) 
[![Docker Pulls](https://img.shields.io/docker/pulls/bolcom/unftp.svg?maxAge=2592000?style=plastic)](https://hub.docker.com/r/bolcom/unftp/)


When you need to FTP, but don't want to.

![logo](logo.png)

unFTP is a FTP(S) server written in [Rust](https://www.rust-lang.org) and built on top of [libunftp](https://github.com/bolcom/libunftp) and the [Tokio](https://tokio.rs) asynchronous run-time. It is **un**like your normal FTP server in that it provides:

- Configurable Authentication (e.g. Anonymous, [PAM](https://en.wikipedia.org/wiki/Linux_PAM) or a JSON file).
- Configurable storage back-ends (e.g. [GCS](https://cloud.google.com/storage/) or filesystem)
- An HTTP server with health endpoints for use for example in Kubernetes for readiness and liveness probes.
- Integration with [Prometheus](https://prometheus.io) for monitoring.
- A proxy protocol mode for use behind proxies like HA Proxy and Nginx.

With unFTP, you can present RFC compliant FTP(S) to the outside world while freeing yourself to use modern APIs and 
techniques on the inside of your perimeter.

## Installation

### Binaries

[Precompiled binaries for unFTP are available](https://github.com/bolcom/unFTP/releases) for Linux and macOS. These 
binaries are static executables.

### From Source

#### Prerequisites

You'll need [Rust](https://rust-lang.org) 1.45 (including `cargo`) or higher to build unFTP. 

Run `make help` to see an overview of the supplied *make* targets.

#### Features

unFTP offers optional features in its Cargo.toml:

- `pam`: enables the PAM authentication module
- `jsonfile_auth`: enables the JSON file authentication module
- `cloud_storage`: enables the Google Cloud Storage (GCS) storage backend

## Usage

Both command line arguments and environment variables are available in unFTP. To show a list of available 
program arguments:

```sh
unftp --help
```

To run with default settings:

```sh
unftp
```

Example running an instance with a filesystem back-end and custom port:

```sh
unftp \
  --root-dir=/home/unftp/data \
  --bind-address=0.0.0.0:2121 \
  --passive-ports=50000-51000 \
  -vv
```

With FTPS enabled:

```sh
# Generate keypair
openssl req \
   -x509 \
   -newkey rsa:2048 \
   -nodes \
   -keyout unftp.key \
   -out unftp.crt \
   -days 3650 \
   -subj '/CN=www.myunftp.domain/O=My Company Name LTD./C=NL'

# Run, pointing to cert and key and require TLS on the control channel
unftp \
  --root-dir=/home/unftp/data \
  --ftps-certs-file=/home/unftp/unftp.crt \
  --ftps-key-file=/home/unftp/unftp.key \
  --ftps-required-on-control-channel=all
```

Enabling the [Prometheus](https://prometheus.io) exporter on (`http://../metrics`), binding to port 8080:

```sh
unftp \
  --bind-address=0.0.0.0:2121 \
  --bind-address-http=0.0.0.0:8080 \
  --root-dir=/home/unftp/data
```

Run with the GCS (Google Cloud Storage) back-end:

```sh
unftp \
  --sbe-type=gcs \
  --sbe-gcs-bucket=mybucket \
  --sbe-gcs-key-file=file
```

Run behind a proxy in [proxy protocol](https://www.haproxy.com/blog/haproxy/proxy-protocol/) mode:

```sh
unftp \
    --proxy-external-control-port=2121
```

Run sending logs to a Redis list:

```sh
unftp \
    --log-redis-host=2121 \
    --log-redis-key=logs-key \
    --log-redis-port=6379
```

Authenticate with credentials stored in a JSON file:

Create a credentials file (e.g. credentials.json):

```json
[
  {
    "username": "alice",
    "password": "12345678"
  },
  {
    "username": "bob",
    "password": "secret"
  }
]
```

```sh
unftp \
    --auth-type=json \
    --auth-json-path=credentials.json
```

## Docker image

The project contains templated Dockerfiles . To get a list of available commands, run:

```sh
make help
```

We offer 3 different options for building an unFTP docker image:

- `scratch`: builds the binary in [rust:slim](https://hub.docker.com/_/rust) and deploys in a `FROM scratch` image.
- `alpine` (default): builds in [rust:slim](https://hub.docker.com/_/rust) and deploy in alpine. This image is built with musl instead of a full-blown libc.
- `alpine-debug`: same images as `alpine` but using the debug build of unftp and adds tools like [lftp](https://lftp.yar.ru/)

To build the alpine docker image:

```sh
make docker-image-alpine
```

Alternatively you can download pre-made images from docker hub:

```sh
docker pull bolcom/unftp:v0.12.1-alpine
```

Example running it:

```sh
docker run \
  -e ROOT_DIR=/ \
  -e UNFTP_LOG_LEVEL=info \
  -e UNFTP_FTPS_CERTS_FILE='/unftp.crt' \
  -e UNFTP_FTPS_KEY_FILE='/unftp.key' \
  -e UNFTP_PASSIVE_PORTS=50000-50005 \
  -e UNFTP_SBE_TYPE=gcs \
  -e UNFTP_SBE_GCS_BUCKET=the-bucket-name \
  -e UNFTP_SBE_GCS_KEY_FILE=/key.json \
  -p 2121:2121 \
  -p 50000:50000 \
  -p 50001:50001 \
  -p 50002:50002 \
  -p 50003:50003 \
  -p 50004:50004 \
  -p 50005:50005 \
  -p 8080:8080 \
  -v /Users/xxx/unftp/unftp.key:/unftp.key  \
  -v /Users/xxx/unftp/unftp.crt:/unftp.crt \
  -v /Users/xxx/unftp/the-key.json:/key.json \
  -ti \
  bolcom/unftp:v0.12.1-alpine
```

## License

You're free to use, modify and distribute this software under the terms of the Apache-2.0 license.

## See also

- [libunftp](https://github.com/bolcom/libunftp)
