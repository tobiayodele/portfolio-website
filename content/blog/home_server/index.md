+++
author = "Tobi Ayodele"
title = "Swapping my Streaming Subscriptions for a Home Server"
date = "2026-09-14"
description = "I built and configured a self-hosted home server to replace my streaming subscriptions and measured how it performed and how much it costs to run."
tags = [
    "home-server",
    "linux",
    "self-host",
]
categories = [
    "linux",

]
+++


Google One, Crunchyroll, Netflix, Spotify, YouTube Premium, Amazon Prime. I currently pay for a lot of subscriptions. I don't want to pay for that many subscriptions (I'm a broke uni student). As a long-term viewer of Linus Tech Tips and lesser-time r/homelab lurker, I decided the best way to fix this was to build my very own home server. But am I even saving money? Is it even a suitable solution? TLDR: *(Yes, I think so!)*.

This blog post delves into how I built my own home server running Ubuntu Server version 24.04, and how (and whether) you should too!

## Hardware

Home labs don't need to be run on top-spec systems. I was fortunate to have an old Dell Latitude 5310, an old business laptop rocking an Intel i5-10310U CPU with 16GB of DDR4 RAM and a 256GB SSD. These can be found refurbished on Amazon for about £200-£250 at the time of writing and can be found for even cheaper on used sites like Ebay.

| Spec | Details |
|---|---|
| CPU | Intel Core i5-10310U, 4 cores / 8 threads, 1.7 GHz (boost to 4.4 GHz) |
| RAM | 16 GB DDR4 |
| Storage | 256 GB SSD |


There are advantages to using a laptop for your home server, namely an inbuilt screen and keyboard, built-in Wi-Fi/Ethernet, and usually cheaper second-hand prices when used business laptops are sold en masse as companies upgrade. One could also argue that the battery acts like a mini-UPS, preventing your server from instantly shutting down if the power goes out.

The screaming negative of using a laptop for your home server, however, is a lack of upgradability and modularity, being limited to the same CPU chip and likely 1-2 RAM and SSD slots.

But I'm on a battle to save money, not spend money, so the Dell has to stay. However, I would soon be pleasantly surprised at how well it performed.

Storage is likely to be the most important consideration of any home lab, but the internal storage of the laptop was limited to only 1 SSD slot, and the current price of SSDs due to the AI-caused shortage means a new SSD would be an expensive investment. After scavenging around my home, I was able to find and harvest two 2TB WD Red NAS hard drives from an ancient D-Link NAS system, and paired with a Ugreen USB 3.0 2-Bay Enclosure I was able to mount both hard drives as external storage on Linux.

> **Note:** This is theoretically slower than an internal SATA connection, but in practice the mechanical hard drives themselves are likely to be the bottleneck here. USB 3.0 has a theoretical max speed of approximately 5Gbps (625MBps), whereas the mechanical hard drive is likely only to reach 100-200MBps (0.8-1.6Gbps).

This is all the hardware I used for my own home lab, but if you wish to buy your own, look for an Intel CPU which supports Quick Sync, such as a used Intel Core i5/i7 from the 11th generation onwards. Quick Sync provides dedicated hardware for video encoding and decoding (see the [transcoding section below](#transcoding)), reducing the workload on the CPU. Lower-power CPUs like modern Intel N-Series CPUs may still be applicable for lighter media home lab systems, however heavier workloads such as machine learning may benefit from additional CPU cores. If you're building a PC, a discrete GPU would be even better suited to machine-learning tasks.

> **Note:** I think the beauty of home labs is repurposing the hardware you already have, but I have a relatively comparable setup below for comparison reasons.

|  | My setup | Budget equivalent |
|---|---|---|
| CPU | i5-10310U, 4C/8T | Used i5-8500T/9500T |
| RAM | 16 GB | 16 GB |
| OS / application storage | 256 GB SSD | 256 GB SSD |
| Extra bulk storage | 2 × 2 TB HDD | 2 × 2 TB HDD |
| Storage connection | USB enclosure | USB enclosure |
| System cost | £0 (already owned) | £100-200 for a used business mini PC |
| Bulk storage cost | £0 (already owned) | £60-80 |
| Storage connection cost | £25 for USB enclosure | £25 for USB enclosure |
| **Total cost** | **£25** | **£200-300** |



## Well, what's running on my home server?

### Immich

For me this is the most important part of my home server.

[Immich](https://immich.app/) is a self-hosted photo and video management platform. This aims to replace my Google Photos subscription.

Immich's containers are stored on the server's SSD, which also contains the Immich database file, allowing for faster access to the database. The media library itself is stored on one of the 2TB hard drives.

One of the features that is particularly useful to me is Immich's machine learning. It analyses the library to recognise faces and identify duplicate photos, making it much easier to find specific photos if I don't know when they were taken. The main downside of this feature is that it is relatively computationally expensive, however it only runs as a background task, so it does not provide consistent additional stress to the server.

{{< figure src="image1.png" alt="Immich web interface showing a photo library timeline" caption="Immich web interface" >}}

The Immich client is also available on iOS and Android, with a web interface which can be accessed through Windows and macOS systems, allowing me to back up photos straight from my personal devices to my server automatically.

### Jellyfin

If you've perused any sort of home lab related media, you've very likely heard of [Jellyfin](https://jellyfin.org/), and for good reason. Jellyfin serves as a self-hosted replacement for movie subscription services (like Netflix, Amazon Prime and Hulu), allowing me to collect and stream my own collection of films and TV shows.

Jellyfin runs directly on my server, using a 2TB hard drive as bulk storage for the media library. As someone who has collected a lot of physical media, filling my library involved ripping physical DVDs and Blu-Ray discs so they can be stored as digital files.

The most useful feature of Jellyfin is being able to automatically sort episodes by played/not played and provide metadata like episode descriptions.

However, Jellyfin comes with a significant limitation: **transcoding**.

### Transcoding

To begin with, media is encoded and often lossy-compressed so it takes up less storage space. It is then decoded again so it can be played. This is handled by **codecs**. Common video codecs include H.264, H.265 and AV1, and popular sound codecs include SBC, AAC and aptX.

However, not every device has support for every codec and you may, for example, want to adjust the settings (like limiting the bitrate or resolution on cellular data to reduce usage), meaning that the video needs to be ***transcoded*** into a codec which your device supports. This creates a serious workload on the server, especially with multiple concurrent streams, and can be worsened by transcoding to a different codec while simultaneously reducing the bitrate and resolution.

{{< figure src="image2.png" alt="Jellyfin playback info panel showing transcoding details for Akira movie" caption="Jellyfin playback info showing transcoding details" >}}


> **Note:** As an example, storing media in AV1 can save a significant amount of storage space over other video codecs, however this comes with a large con: many older devices don't support AV1 decoding, meaning that the server will be forced to transcode AV1.

The best way to avoid transcoding is to store your media at a resolution that most of your devices can run natively, with a codec which your devices can decode, and at a bitrate which still allows for reasonable performance. However, the screenshot above shows the difficulty in trying to do this. Jellyfin is fortunate enough to provide reasoning for why the server must transcode, and in many cases this is due to the subtitle codec not being supported, particularly on the web player. Furthermore, the browser can't support this number of audio channels, forcing the server to transcode from AV1 to H.264 and OPUS to AAC.

Jellyfin also shows the transcoding speed, measured as a multiple of the natural frame rate of the media you're watching. For example, Akira, like many movies, has a natural frame rate of 24fps, whereas my server is able to transcode at 47fps, giving:

$$\frac{47\ \text{fps}}{24\ \text{fps}} = 1.96\text{x}$$

When transcoding, a transcoding speed of 1x means you're transcoding exactly at real-time speed, with <1x meaning that the transcoding is slower than real-time (meaning the stream will eventually buffer and lag), whereas >1x means that transcoding is faster than the video is being watched, so even though it is providing additional workload to the server, this is likely to not affect the viewer.

Transcoding performance can also be improved with hardware acceleration, for example Intel Quick Sync on modern Intel CPUs.

### Tailscale

Before this point, my server is only accessible to devices on my local network. The issue with this is that as soon as I leave home, I lose access to both my Immich and Jellyfin libraries. This becomes a damning disadvantage when comparing to the streaming and cloud services I aim to replace, which can be accessed wherever you have an internet connection.

A method of fixing this problem would be to open and forward ports on your router to your server and run a reverse proxy (see: Nginx) which points my personal domain name to my home IP. However, this is overkill for accessing applications on my server, requires a domain name, and opening ports on your home server increases its attack surface. [Internet scanner bots](https://en.wikipedia.org/wiki/Web_crawler) automatically probe IP addresses and may attempt to exploit the open port.

An easier method which also reduces the attack surface is [**Tailscale**](https://tailscale.com/). Tailscale is a VPN software which is based on the open-source WireGuard protocol. Each device running Tailscale is assigned its own private Tailscale network IP address. These addresses can then be used to access the services running on the server (or even remotely SSH into the server) from another device, wherever, without needing to open any ports on my router, yay!

Though, Tailscale only provides connectivity, and does not make the services inherently secure, so strong password habits and security settings must still be set up on the server.

## Well, how much power does the server draw?

Measuring the wall power consumption of a laptop would have been much easier with a physical wall power meter. Unfortunately, I don't have one of those, so I had to get a little crafty in measuring the power consumption of my server. Linux has measurements for the current and voltage, and using the electrical power equation derived from Ohm's Law (P = IV) I can get the battery power consumption when the server is not running on AC.

To measure this I used the inline script:

```bash
for i in {1..360}; do
  date
  cat /sys/class/power_supply/BAT0/current_now
  cat /sys/class/power_supply/BAT0/voltage_now
  sleep 10
done > power.txt
```

This works by logging the date, the current, and the voltage, then sleeping for 10 seconds and repeating 360 times (an hour) into `power.txt`.

This command doesn't do all the work, printing a new line for each field.

```
Date 1
Current 1
Voltage 1
Date 2
Current 2
Voltage 2
...
```

The following Python script calculates the battery power estimate using the electrical power equation derived from Ohm's Law, then converts these results into a CSV and calculates the average battery power consumption. Linux reports the current and voltage information in microamps and microvolts, so this is converted to amps and volts before calculating power in watts:

```python
import csv
import sys
from datetime import datetime, timezone

#voltage_now and current_now stored in microvolts/microamps
CURRENT_SCALE = 1e-6
VOLTAGE_SCALE = 1e-6

#Timestamp is Day name, Day number, Month, Hour, Minute, Second, Timezone, Year
#e.g. Thu 10 Sep 19:39:55 UTC 2026
TIMESTAMP_FORMAT = "%a %d %b %H:%M:%S %Z %Y"

#convert to unix
def parse_timestamp_to_unix(ts_str: str) -> int:
    dt = datetime.strptime(ts_str.strip(), TIMESTAMP_FORMAT)
    dt = dt.replace(tzinfo=timezone.utc)
    return int(dt.timestamp())


def convert(input_path: str, output_path: str) -> None:
    f =  open(input_path, "r")
    lines=[]
    for line in f:
        lines.append(line.strip())
    f.close()

    rows = []
    #every three lines (to record time, current then power)
    for i in range(0, len(lines) - 2, 3):
        time= lines[i]
        current= lines[i+1]
        voltage =  lines[i + 2]

        unix_time = parse_timestamp_to_unix(time)
        current = float(current)
        voltage = float(voltage)
        
        # P=IV
        power = (current * CURRENT_SCALE) * (voltage * VOLTAGE_SCALE)

        rows.append([unix_time, current, voltage, power])
 

    f = open(output_path, "w", newline="")
    writer = csv.writer(f)
    writer.writerow(["unix_time", "Current (A)", "Voltage (V)", "Power (W)"])
    for i in range (len(rows)):
        writer.writerow(rows[i])
    f.close()

    print("Wrote {} rows to {}".format(len(rows), output_path))

    #calculate average power consumption
    samples = 0
    total = 0
    for row in rows:
        total = total + row[3]
        samples = samples + 1
    
    average_power_consumption = total / samples
    print("Average power consumption over {} samples is {} W".format(samples,round(average_power_consumption, 3)))

if __name__ == "__main__":

    convert(sys.argv[1], sys.argv[2])
```

The idle estimated battery power consumption was 5.6W, which is extremely reasonable for a persistently online server.

## Transcoding Test

Apart from measuring idle power consumption, I also wanted to stress test the server using a service that I already use. As mentioned before, transcoding can provide a significant workload to the server. By forcing Jellyfin to transcode  by reducing the bitrate from the native bitrate and measuring the average battery power consumption, I aimed to get a better idea of how the computer performs under stress.

To measure this, I used the [Jellyfin Test Videos](https://syd1.mirror.jellyfin.org/test-videos/) to test different levels of transcoding:

- 1080p AVC 20M to 8M
- 4K HEVC 60M to 8M
- 4K HEVC 60M HDR to 8M

The 1080p AVC video should be the least demanding to transcode, while the 4K HEVC 60M HDR should be the most demanding, as Jellyfin also needs to do tone mapping down to SDR.

> **Note:** These videos are 30s long and so are run on repeat in Jellyfin over the time recorded. Each time the video repeats it must transcode the video again, and when the video ends this may result in a drop in power consumption not representative of the load during transcoding.

Each video was tested by loading the video and letting it run a bit to settle before running the script to measure its power consumption.

| Transcoding | Average battery power consumption (W) |
|---|---|
| 1080p AVC 20M to 8M | 14.5W |
| 4K HEVC 60M to 8M | 28.2W |
| 4K HEVC 60M HDR to 8M | 29.5W |

As expected, the 4K HEVC 60M HDR transcoding had the highest average battery consumption, although the difference compared to the 4K HEVC SDR video was relatively small.

Using the [Ofgem electricty price cap for October 2026](https://www.ofgem.gov.uk/information-consumers/energy-advice-households/energy-price-cap-unit-rates-and-standing-charges), the price is 26.32 pence per kWh. This means the estimated yearly cost of the server if it were run 24/7 at idle is:

$$\frac{5.6\text{W} \times 24 \times 365}{1000} = 49.1\ \text{kWh/year}$$

$$49.1\ \text{kWh/year} \times £0.2632/\text{kWh} = £12.91/\text{year}$$

If I decided to instead transcode 4K HEVC HDR videos for a year straight, it would cost an estimated:

$$\frac{29.5\text{W} \times 24 \times 365}{1000} = 258.4\ \text{kWh/year}$$

$$258.4\ \text{kWh/year} \times £0.2632/\text{kWh} = £68.02/\text{year}$$

The actual energy cost is likely to be somewhere between these two values, likely leaning towards the lower end of the range, as a lot of my media is stored as 1080p AVC, so it does not need to be transcoded. In either case, this is still significantly cheaper per year than my subscriptions.

## Conclusion

To conclude, building and setting up a home server can be a cheap way to replace some of your current streaming subscriptions, especially if you already have old tech laying around, with a lot of open-source maintained services to help you on your journey. However, a lot of time is expected to set up and maintain a home lab.

