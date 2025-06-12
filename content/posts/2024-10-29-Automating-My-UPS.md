---
title: "Automating My UPS"
date: 2024-10-29
draft: false
tags:
  - Homelab
---

## Introduction

In an era where every second of downtime can have serious repercussions, maintaining a reliable power supply for your essential devices is crucial. Automating your UPS management not only enhances reliability but also optimizes battery usage during unexpected outages. In this post, we’ll delve into how to automate your UPS using the NUT (Network UPS Tools) framework alongside a custom Go program. This approach not only streamlines power management but also ensures your UPS batteries are used efficiently during limited power situations, safeguarding your equipment when it matters most.

The current UPS I am using is a [Nilox UPS PREMIUM LINE](https://www.niloxtech.com/datasheet/nxgcli12001x7v2).

## Why

So, why not dive into this project? Building systems like these is not only enjoyable, but it also allows me to explore a variety of technologies and fields that I'm less familiar with, ultimately broadening my knowledge and research horizons. Moreover, commercial UPS systems with automatic shutdown capabilities can be quite costly. By combining an affordable UPS with some programming skills, we can create a smart UPS solution that provides valuable insights into our home’s power usage and ensures a safe shutdown of connected devices, even when I'm not around.


## Nut

To work with the UPS I am using the [Nut](https://networkupstools.org/) (Network UPS Tools), running it in a [Docker container](https://github.com/instantlinux/docker-tools/tree/main/images/nut-upsd).

I first found what was my `VENDORID` by using `docker run --rm --entrypoint /bin/ls instantlinux/nut-upsd /usr/lib/nut`

```console
[nutdev1]
  driver = "nutdrv_qx"
    port = "auto"
    vendorid = "0665"
    productid = "5161"
    product = "USB to Serial"
    vendor = "INNO TECH"
    bus = "001"
```

With this in mind, I've followed the README.md from the github repo and configured the `Dockerfile` as shown below:

```console
FROM alpine:3.20
MAINTAINER Rich Braun "docker@instantlinux.net"
ARG BUILD_DATE
ARG VCS_REF
LABEL org.label-schema.build-date=$BUILD_DATE \
    org.label-schema.license=GPL-2.0 \
    org.label-schema.name=nut-upsd \
    org.label-schema.vcs-ref=$VCS_REF \
    org.label-schema.vcs-url=https://github.com/instantlinux/docker-tools
ARG NUT_VERSION=2.8.2-r0
ENV API_USER=upsmon \
    API_PASSWORD=CHANGETHIS \
    DESCRIPTION=UPS_Brun0x \
    DRIVER=nutdrv_qx \
    GROUP=nut \
    MAXAGE=15 \
    NAME=ups \
    POLLINTERVAL= \
    PORT=auto \
    SDORDER= \
    SECRET=CHANGETHIS \
    SERIAL= \
    SERVER=master \
    USER=nut \
    VENDORID=0665
HEALTHCHECK CMD upsc $NAME@localhost:3493 2>&1|grep -q stale && \
    kill -SIGTERM -1 || true

RUN echo '@edge http://dl-cdn.alpinelinux.org/alpine/edge/community' \
      >>/etc/apk/repositories && \
    echo '@edgemain http://dl-cdn.alpinelinux.org/alpine/edge/main' \
      >>/etc/apk/repositories && \
    apk add --no-cache dash@edgemain && \
    apk add --update --no-cache nut@edge=$NUT_VERSION \
      busybox@edgemain linux-pam@edgemain \
      libcrypto3@edgemain libssl3@edgemain \
      libusb musl net-snmp-libs util-linux

RUN apk add --no-cache tzdata
ENV TZ=Europe/Lisbon

EXPOSE 3493
COPY entrypoint.sh /usr/local/bin/
ENTRYPOINT /usr/local/bin/entrypoint.sh
```

To be able to grab logs from ups I had to edit the `entrypoint.sh` and add the following line:

```bash
/usr/bin/upslog -u $USER -s ups -i 2 -l /var/log/upslog.txt -f "%TIME @Y-@m-@d @H:@M:@S% %VAR battery.charge% %VAR input.voltage% %VAR ups.load% [%VAR ups.status%]"
```

Then I created a `docker-compose.yml` as shown below:

```yml
services:
  nut-upsd:
    build:
      context: .
    container_name: nut-upsd-new-brun0
    environment:
      - "API_USER=upsmon"
      - "API_PASSWORD=CHANGETHIS"
      - "DESCRIPTION=UPS_Brun0x"
      - "DRIVER=nutdrv_qx"
      - "GROUP=nut"
      - "NAME=ups"
      - "PORT=auto"
      - "SECRET=CHANGETHIS"
      - "USER=nut"
      - "VENDORID=0665"
    privileged: true
    restart: always
    ports:
      - 3493:3493
    volumes:
      - /home/brun0/nut/nut/upslog.txt:/var/log/upslog.txt
```

After completing these steps, I successfully created the Docker container:
- Starting
  - `docker compose up -d --build --force-recreate`
- View logs
  - `docker compose logs -f`
- Stoping
  - `docker compose down`
  

{{< figure src="/ups/nut-docker-logs.png" class="post-image" >}}



## Prometheus Nut exporter

To view the UPS logs more effectively, I utilized the [Prometheus exporter for NUT](https://github.com/HON95/prometheus-nut-exporter
) with the following `docker-compose.yml`:

```yml
services:
  nut-exporter:
    # Stable v1
    image: hon95/prometheus-nut-exporter:1
    environment:
      - TZ=Europe/Lisbon
      - HTTP_PATH=/metrics
    restart: always
    ports:
      - "9995:9995/tcp"
```

{{< figure src="/ups/nut-prometheus.png" class="post-image" >}}


## AlertManager

The alerting aspect is crucial, as I want to receive notifications during power outages and keep a record of those alerts. Since I was already using Prometheus for other projects, I decided to implement a new rule in Prometheus Alertmanager to monitor the UPS status and alert me in case of power loss.

For this I created a ruled (`ups.rules.yml`) in `/etc/prometheus` with the following:

```yml
groups:
  - name: ups-alerts
    rules:
      - alert: UPSStatusCritical
        expr: nut_status{ups="ups"} == 2
        for: 3m
        labels:
          severity: critical
        annotations:
          summary: "UPS status is critical and its running on battery"
          description: "The UPS status is currently critical (status = 2)."
```

This rule will monitor the NUT Prometheus exporter statistics page and trigger an alert in Prometheus if the `nut_status` equals 2 (indicating a power loss).

Then added the rule to the `prometheus.yml`:

```yml
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
   - "ups.rules.yml"
```


And finaly configured Alertmanager to make a web call to `NutGMonitor` (running on 192.168.30.13:9999) whenever a new alert is triggered (explained in the next section):

```yml
route:
  group_by: ['ups-alerts']
  receiver: 'webhook'
  group_wait: 30s
  group_interval: 1m
  repeat_interval: 1m

receivers:
  - name: 'webhook'
    webhook_configs:
      - url: 'http://192.168.30.13:9999/alertmanager'
        send_resolved: true
```

## NutGMonitor

Following the initial setup of the UPS, the goal is to continuously monitor its performance and automatically shut down connected devices when needed.

[NutGMonitor](https://github.com/BrunoTeixeira1996/NutGMonitor/) is a Go program designed to perform specific actions on the UPS whenever an event occurs.

When the UPS stops receiving power, it triggers a warning to the Prometheus Alert Manager after 2 minutes. The Alert Manager then forwards this information to NutGMonitor.

If the power loss continues for approximately 5 minutes, NutGMonitor initiates the shutdown of devices connected to the UPS.

The power-off mechanism used in NutGMonitor is customized for the target system. For example, when working with a Linux machine, I create an `off` bash script containing `halt -f -f -p`. I then generate an SSH key for connecting from the NutGMonitor instance to the target. To restrict access, I add this key to the target's `authorized_keys` with the options `no-pty,no-X11-forwarding,command="sudo /root/off"`, ensuring it can only execute the off script.

Once all targets are powered down, NutGMonitor makes sure to shutdown itself, effectively disconnecting it from the UPS.

NutGMonitor also accounts for brief power losses followed by quick restorations that can occur before the 2-minute threshold is reached, potentially preventing Alertmanager from receiving updates. To tackle this issue, I monitor these power off/on events using upslog within the NUT Docker container, as shown previously.


## Conclusion

I've set up a reliable system that keeps me safe during unexpected power outages (it works for me :)), even when I'm away from home. With the integration of a Prometheus exporter, I can monitor my UPS in real-time, allowing me to define new metrics and visualize them whenever I want.

While I can safely shut down all devices when I'm at home, I can't always be there when the power goes out. That's where NutGMonitor comes in; it helps me manage and gracefully shut down every device connected to my UPS. This not only eases my mind during outages but also protects my equipment from potential damage caused by abrupt shutdowns. Overall, this setup gives me peace of mind, knowing that I’m prepared for any power interruptions, whether I'm home or away.