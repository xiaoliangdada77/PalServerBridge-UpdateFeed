# PalServerBridge

## Languages

- [English](README.md)
- [Chinese (Simplified)](README-CN.md)

PalServerBridge is a UE4SS C++ plugin for Palworld dedicated servers. It exposes in-game server capabilities to external tools, making it useful for admin dashboards, Discord bots, event systems, anti-cheat audits, and server automation.

This repository provides the public project entry point and update notes. See [UPDATE.md](UPDATE.md) for the latest release information.

## Features

- Provides HTTP APIs for external programs to read and operate Palworld server data.
- Queries online players, player identity data, inventory items, pal storage, and party pals.
- Supports giving and deleting items, deleting specific pals, sending chat messages, announcements, and private messages.
- Supports server-side actions through APIs, including coordinate-based pal spawning.
- Designed for admin panels, Discord commands, PvP bounties, event rewards, and import audits.
- Release packages include a compatibility-verified UE4SS build and a self-built PalSchema, reducing version-mismatch issues between dependencies.

## Use Cases

- Server admin panels: inspect player profiles, inventories, pal data, and handle abnormal assets.
- Discord bots: provide query, reward, debit, announcement, and event-management commands.
- Event systems: trigger rewards, spawn targets, and broadcast event state.
- Anti-cheat audits: read pal SaveParameter data and inspect passives, IVs, level, rank, and other fields.

## Updates

Release notes, compatibility changes, API changes, and fixes are published in [UPDATE.md](UPDATE.md).

## Support

If PalServerBridge helps your server, you can support ongoing maintenance through the donation QR codes below.

| WeChat | Alipay |
| --- | --- |
| ![WeChat donation QR code](src/wechat-donation.jpg) | ![Alipay donation QR code](src/alipay-donation.jpg) |
