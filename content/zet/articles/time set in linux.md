```markdown
---
title: "Restore and Synchronize Your Laptop's Time After Battery Removal"
date: "2025-04-25T14:00:00+05:30"
draft: false
tags:
  - "Linux"
  - "System Administration"
  - "Timedatectl"
  - "hwclock"
categories:
  - "Tutorial"
  - "Hardware"
series: []
description: "How to restore and keep your Linux laptop’s system and hardware clocks in sync after disconnecting the RTC battery."
cover: "/images/time-sync-cover.jpg"
---

A sudden battery removal can reset your laptop’s hardware clock (RTC), leaving your system with incorrect time and date. This post walks you through verifying current settings, configuring your time zone, enabling NTP sync, and ensuring your RTC stays accurate—even offline.

<!--more-->

## 1. Verify Current Time Settings

### Check Systemd Time Status
```bash
timedatectl status
```
This displays:
- **Local time** (system clock)
- **Universal time (UTC)**
- **RTC time** (hardware clock)
- **NTP status**
- **Configured time zone**

### Inspect the Hardware Clock
```bash
sudo hwclock --show
```
If the RTC shows a past date (e.g., 1970), it confirms the clock reset when the battery was removed.

## 2. Set Your Time Zone

### List Available Zones
```bash
timedatectl list-timezones
```

### Apply Your Region’s Zone
```bash
sudo timedatectl set-timezone Asia/Kolkata
```
This updates `/etc/localtime`, ensuring logs and timestamps reflect your locale.

## 3. Enable Automatic Time Sync (NTP)

To keep the system clock accurate:
```bash
sudo timedatectl set-ntp true
```
This activates `systemd-timesyncd` (or your NTP daemon) to poll time servers automatically.

## 4. Synchronize the Hardware Clock

Once the system clock is correct, write it to the RTC:
```bash
sudo hwclock --systohc
```

> **Note:** Linux defaults to storing the RTC in UTC. For dual-boot systems using local RTC (e.g., Windows), run:
```bash
sudo timedatectl set-local-rtc 1 --adjust-system-clock
```
This configures systemd to interpret the RTC as local time and immediately reconciles both clocks.

## 5. (Optional) Manual Setting Without Network

If you’re offline, set both clocks manually:

1. **Set system clock**:
   ```bash
   sudo date --set="2025-04-25 14:30:00"
   ```
2. **Update RTC**:
   ```bash
   sudo hwclock --systohc
   ```

Or directly:
```bash
sudo hwclock --set --date="2025-04-25 14:30:00"
sudo hwclock --hctosys
```

## Conclusion

- **Time zone:** Ensures local timestamps match your location.  
- **NTP:** Automatically maintains system clock accuracy.  
- **RTC sync:** Guarantees correct hardware clock across power cycles.

Follow these steps to maintain reliable timekeeping on your Linux laptop, even after RTC battery removal.
```          
