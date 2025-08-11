# ZTE-F601-Bezeq-Firmware-Research
## 🛠️ Essential Firmware Reversing Starter Pack  

Before you start ripping apart innocent little IoT devices, make sure you’ve got the essentials.  
Remember: if it has a chip, it probably has secrets. And you’re here to… “borrow” them.  

---

⚠️ **Disclaimer (a funny but important one):**  
This reversing adventure is focused on **ZTE F601 firmware** — kindly provided and allowed by **Bezeq**.  
So yes, we’re playing with permission. No shady stuff, no ski-mask required.  
Think of it as “firmware archaeology” instead of “firmware burglary.”  

---

1. 🔍 **Binwalk**  
   _The Swiss Army knife of firmware extraction. It’s like a can opener… but for binary blobs._  
   ➡ [Get Binwalk](https://github.com/ReFirmLabs/binwalk)  

2. 🖥️ **Virtual Machine (Kali Linux / Ubuntu)**  
   _Because accidentally nuking your host OS with `dd` is only funny when it happens to someone else._  
   _(Pro tip: VM snapshots are the Ctrl+Z of firmware reversing.)_  

3. 🛰️ **Serial Console + USB-UART Adapter**  
   _When you need to sweet-talk a device into giving you its secrets… one baud rate at a time._  
   _(Yes, “speed dating” for hardware exists, and it’s called a UART session.)_  

4. 📡 **Firmware-Mod-Kit**  
   _Unpack, tinker, repack — basically a firmware surgery toolkit._  
   _Just try not to leave any “debug ports” open unless you like unexpected visitors._  
   ➡ [Firmware-Mod-Kit](https://github.com/rampageX/firmware-mod-kit)  

5. 🐙 **Ghidra / IDA / Radare2**  
   _Because staring at raw hex for 8 hours straight will make you start hearing beeps._  
   _(Also, your brain deserves a disassembler — it’s cheaper than therapy.)_  

6. 🎧 **Playlist**  
   _Whether you’re dumping flash or watching progress bars like it’s Netflix, tunes keep you sane._  
   Here’s my [Firmware Bricking Playlist](https://music.youtube.com/playlist?list=PLsaTaTh7fDWEq_JMpTKHqwNzmM3tDFQFy&si=RsgwNSH1G1gUVFfa).  

7. ⚡ **Caffeine**  
   _Because “coffee overflow” is the only overflow you want while reversing._  
   _(Energy drinks also work, but they might give you a stack overflow in real life.)_  

8. 🧪 **A sacrificial ZTE F601**  
   _It’s okay to brick it — Bezeq literally told us it’s fine. Just don’t accidentally improve it too much or they might start charging rent._  

---

### 🥷 Pro Hacker Mode (Optional but Encouraged)  
- **Logic Analyzer / Oscilloscope** — For when you want to see the *actual electrons gossiping*.  
- **Hot Air Rework Station** — Because sometimes you just have to yeet a chip off the board.  
- **JTAGulator** — Like Tinder for finding hidden debug pins, but less awkward.  
- **Post-it Notes** — The only persistence layer that hackers trust.  

---

💡 **Final Tip:** Always document *everything*. Future-you will thank past-you when you forget why there’s a ZTE ONT sitting in your freezer.  

---

## 🚀 Let The Firmware Breaking Begin

### 🛡️ D.O.D — *Definition Of Done*

This is where I lay out the *mission objectives* — a.k.a. the reasons I’ll be riding a caffeine high and making questionable life choices for the next few days.

💤 **Sleep Goal:** My personal best is 3 days without sleep. The challenge: break the record without hallucinating that the UART port is plotting against me. (Last time, it claimed to have ice cream in my fridge… it was telling the truth, but I paid for it.)

🎯 **Main Target:**
Hunt down an **open debug feature** that’s still accessible to the user. Once found, I’ll decide if it’s:

* 🏴‍☠️ A hacker’s fantasy come true (bursting with juicy exploits)
* 🥚 A quirky hidden Easter egg (fun but harmless)

Either way, the process is the real thrill — and by “thrill,” I mean slowly questioning every career decision while watching endless hex dumps.

> *Remember: firmware doesn’t break… it **reveals its true personality** under pressure.*


### 🚦 Let's Go!

Like every good researcher who gets their hands on a binary file, the first step is unleashing the ultimate secret weapon: `binwalk` — the digital equivalent of shaking a Christmas present to guess what’s inside.

```bash
binwalk F601_V6.0.1P1T12_UPGRADE_BOOTLDR.bin 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
256           0x100           uImage header, header size: 64 bytes, header CRC: 0x76A78931, created: 2016-03-07 19:25:33, image size: 4815272 bytes, Data Address: 0x40008000, Entry Point: 0x40008000, data CRC: 0xCD5E8389, OS: Linux, CPU: ARM, image type: OS Kernel Image, compression type: lzma, image name: "Linux Kernel Image"
320           0x140           LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 17348096 bytes
4939560       0x4B5F28        CRC32 polynomial table, little endian
```

So here’s the loot from our first scan:

* **uImage** — Basically the kernel. Think of it as the brain, but compressed so it can fit in its tiny plastic skull.
* **LZMA compressed data** — Probably the file system. Like a mystery piñata filled with binaries instead of candy.
* **CRC32 polynomial table** — A checksum to make sure nothing got corrupted. (Or in firmware terms: the device’s way of saying “Don’t mess with my stuff.”)

The takeaway? We’ve got the brain, the body, and the lock… now all we need is the key. Or a really big hammer.
And if you’ve followed my past research logs, you already know — I’m not really a “find the key” kind of guy. Keys are for people who knock politely. I’m more of a **hammer** enthusiast.

So let’s bring out the sledge:

```bash
binwalk -Me F601_V6.0.1P1T12_UPGRADE_BOOTLDR.bin
```

This command is basically saying: *“Forget subtlety, let’s break in, flip the furniture, and shake the binaries out of the firmware until they spill their secrets.”*

No lockpicks, no finesse — just a direct invitation for the binary to reveal what it’s hiding… whether it likes it or not.

And like stealing candy from a baby… or stealing a baby from a candy store… or however that saying actually goes, we ended up with this shiny loot drop:

```bash
_F601_V6.0.1P1T12_UPGRADE_BOOTLDR.bin.extracted/_140.extracted/cpio-root
```

Inside? The **magical file system** itself — the treasure chest at the end of our first dungeon.

This is the moment for a small victory dance 💃 — preferably one that says *“Yes, I just pried open your firmware”* without also saying *“Yes, I’ve been awake for 48 hours straight.”*


### 🔧 The Mysterious `init` Script  

Found an `init` script at the top of the filesystem. It’s a legendary one-liner bash script that looks like this:

```bash
#!/bin/sh
# Copyright (C) 2006 OpenWrt.org


exec /bin/busybox init
```

ok this neat, that means that we have the program ./bin/busybox that will run everything! also OpenWrt? what is that?

OpenWrt.org is like the secret sauce for your router! It's an open-source Linux-based firmware that transforms your boring, factory-default router into a customizable, feature-packed powerhouse. Think of it as giving your router a superhero cape, enabling advanced networking features, better performance, and even the ability to run apps. It's the ultimate playground for network enthusiasts and tinkerers. Just be careful—once you go OpenWrt, you might never look at stock firmware the same way again! and its the **best** thing a hacker can detect, why? because now it's more then just a box, its a magic box (i mean, busybox but you get it).
soooooo we just need to find a vulnerability at the designited router version we got there? thats it?