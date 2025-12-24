Author: Richard Quillen
Description: Use Docker compose to create and install a minimal container for Nautobot
Check out Nautobot's GH: https://github.com/nautobot/nautobot-docker-compose

# Prerequisites

This guide is for setting up a Nautobot PoC on a Debian distro using Nautobot's docker-compose source from GH.

I am specifically running on Ubuntu.

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg'
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

then

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

and finally

```bash
sudo apt update
```
## Install Docker

You should be able to install docker with:

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

and

```bash
sudo apt install -y docker-compose-plugin
```

Confirm your version:

```bash
docker --version
docker compose version

```
and have it return something like

```bash
Docker version 28.2.2, build 28.2.2-0ubuntu1~24.04.1
Docker Compose version v5.0.0
```

## Install Poetry
Poetry is used to resolve Python dependencies

```bash
sudo apt install python3-poetry
```

## Create your directory setup

```bash
mkdir -p ~/nautobot-compose && cd ~/nautobot-compose
```

## Clone the repo

```bash
git clone https://github.com/nautobot/nautobot-docker-compose.git
cd nautobot-docker-compose
```

## Resolve dependencies

```bash
poetry shell
poetry install
```

# Pre-setup

## Setup local environment files

```bash
cp environments/local.example.env environments/local.env
cp environments/creds.example.env environments/creds.env
```
In environments/local.env, change "NAUTOBOT_CREATE_SUPERUSER=true

## Setup local invoke file

```bash
cp invoke.example.yml invoke.yml
```

## Do not require elevation for docker

```bash
sudo usermod -aG docker $USER
newgrp docker
```

# Build it

Run 

```bash
invoke build
```

It will go on for a bit and finally spit out:

```bash
 Image yourrepo/nautobot-docker-compose:local Built
```

# Spin it up

```bash
invoke start
```

Assuming all went well, you should see 

```bash
Container nautobot-docker-compose-celery_beat-1 Started
```

## Check health

After the container initially starts, you may see this:

```bash
rj@hellolat:~/nautobot-compose/nautobot-docker-compose$ curl -v http://localhost:8080/
* Host localhost:8080 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:8080...
* Connected to localhost (::1) port 8080
> GET / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/8.5.0
> Accept: */*
> 
* Recv failure: Connection reset by peer
* Closing connection
curl: (56) Recv failure: Connection reset by peer
```

Just give it a minute and try again. Eventually it should show:

```bash
rj@hellolat:~/nautobot-compose/nautobot-docker-compose$ curl -v http://localhost:8080/
* Host localhost:8080 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:8080...
* Connected to localhost (::1) port 8080
> GET / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 302 Found
< Date: Wed, 24 Dec 2025 18:07:05 GMT
< Server: WSGIServer/0.2 CPython/3.12.12
< Content-Type: text/html; charset=utf-8
< Location: /login/?next=/
< X-Content-Type-Options: nosniff
< Referrer-Policy: same-origin
< Cross-Origin-Opener-Policy: same-origin
< X-Frame-Options: DENY
< Content-Length: 0
< Vary: Cookie, origin
< 
* Connection #0 to host localhost left intact
```

# Access through browser

In your browser, navigate to http://localhost:8080.

Default login credentials are admin/admin.

Congrats! You now have a local PoC instance of Nautobot running on your Debian distro