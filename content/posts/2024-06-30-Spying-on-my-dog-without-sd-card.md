---
title: "Spying on my dog without a SD Card"
date: 2024-06-30
draft: false
toc: true
tags:
  - Go
---

## Index

- [Introduction](#introduction)
- [How](#how)
- [The Tool](#the-tool)

# Introduction

Recently, I bought a camera to record Alex (my dog) when he is alone at home. However, after some initial research, I found that this camera only allows recording with an SD card. Additionally, when the SD card is being used for recording, I can't use the RTSP (Real-Time Streaming Protocol).

Since I don't want to buy multiple SD cards for the several cameras I plan to have in the future, the best solution would be to have a program that records and saves the footage to a specific location, such as an HDD, without needing an SD card.

# How

With this in mind I built a small architecture first so It would be easier using Go.

I decided to use the following:
- Programming language (Go)
- RTSP reader (FFMPEG)
- Hosting server (Basic linux server)

After enumerating all parts I built the following architecture:

![Untitled-2024-06-11-1602](https://github.com/BrunoTeixeira1996/gocam/assets/12052283/1a85532c-0d32-4704-8ec0-8a08a937ed72)

The main goal of this project was:
- To be able to record multiple cameras simultaneously
- To be able to stop the recording at any given time without losing the already recorded data
- To use a simple config file for the setup
- To have a simple web UI showing recordings that are currently running
- To have a JSON file containing all recordings marked as finished and canceled
- To use cURL to receive and send information, allowing integration with third-party apps

# The Tool

After implementing all the features needed to record my little buddy, [gocam](https://github.com/BrunoTeixeira1996/gocam) was born.