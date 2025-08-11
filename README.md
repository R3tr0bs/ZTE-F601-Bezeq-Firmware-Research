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

Like every good researcher who gets their hands on a binary file, the first step is unleashing the ultimate secret weapon: `binwalk` — the digital equivalent of shaking a mall Santa Clause for not getting the pink scooter you asked for at the last 23 years hoping it will fall out of his small pocket(thanks for nothing Santa).

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
ok this is a good lead, now lets see what the heck is going on at the busybox binary.

and we know what time is it?? **Binary Reversing Time!!!!!**
so i open ghidra and saw this beautiful entry function (psudo code):
```cpp

void main(undefined4 param_1,undefined4 *param_2)

{
  undefined4 *puVar1;
  byte *pbVar2;
  bool bVar3;
  
  pbVar2 = (byte *)*param_2;
  pbRam000566e4 = pbVar2;
  if (*pbVar2 == 0x2d) {
    pbVar2 = pbVar2 + 1;
    pbRam000566e4 = pbVar2;
  }
  while (puVar1 = (undefined4 *)(uint)*pbVar2, puVar1 != (undefined4 *)0x0) {
    bVar3 = puVar1 == (undefined4 *)0x2f;
    if (bVar3) {
      puVar1 = (undefined4 *)0x566e4;
    }
    pbVar2 = pbVar2 + 1;
    if (bVar3) {
      *puVar1 = pbVar2;
    }
  }
  FUN_0000c5a0(pbRam000566e4,param_1,param_2);
                    /* WARNING: Subroutine does not return */
  FUN_00036b4c(&UNK_000417bc);
}

```

What does that name mean? Absolutely nothing to me — it sounds like either a secret weapon or a failed Wi-Fi password.  

But here’s the twist: it actually **uses our parameters**. That’s right — this is where the *magic* happens. ✨  

So, I did what any sane, well-adjusted reverse engineer would do:  
I dove straight into the function head-first.  

And what did I get?  
- **25%**: Disassembly  
- **75%**: Existential crisis  
- **100%**: The feeling that my brain just signed up for a spinning class without asking me first.  

Still… this is the heart of the beast, and somewhere in here, BusyBox’s grand plan is hiding.  
so below you will see a dump of code, take a deep breath, and remember, you can win.... 1.... 2.... and here we go:
```cpp

void FUN_0000c5a0(undefined4 param_1,undefined4 param_2,int param_3)

{
  byte bVar1;
  undefined4 *puVar2;
  int iVar3;
  FILE *__stream;
  char *pcVar4;
  int iVar5;
  char *pcVar6;
  int *piVar7;
  char *pcVar8;
  ulong uVar9;
  __uid_t _Var10;
  __gid_t __gid;
  group *pgVar11;
  passwd *ppVar12;
  int iVar13;
  int *piVar14;
  char **ppcVar15;
  uint uVar16;
  char *__s;
  int local_19c;
  char acStack_198 [256];
  stat64 sStack_98;
  char *local_2c [2];
  
  if (((((DAT_00051fa4 == 0) && (iVar3 = stat64("/etc/busybox.conf",&sStack_98), iVar3 == 0)) &&
       ((sStack_98.st_mode & 0xf000) == 0x8000)) &&
      ((sStack_98.st_uid == 0 && ((sStack_98.st_mode & 0x12) == 0)))) &&
     (__stream = fopen64("/etc/busybox.conf","r"), __stream != (FILE *)0x0)) {
    DAT_00051fb0 = 1;
    iVar3 = 0;
    local_19c = 0;
    piVar14 = (int *)0x0;
    do {
      while( true ) {
        while( true ) {
          do {
            pcVar4 = fgets(acStack_198,0x100,__stream);
            piVar7 = piVar14;
            if (pcVar4 == (char *)0x0) {
              iVar3 = ferror(__stream);
              if (iVar3 != 0) {
                pcVar4 = "reading";
                goto LAB_0000c8e4;
              }
              fclose(__stream);
              DAT_00051fac = piVar14;
              goto LAB_0000c924;
            }
            pcVar4 = strchr(acStack_198,10);
            local_19c = local_19c + 1;
            if ((pcVar4 == (char *)0x0) && (iVar5 = feof(__stream), iVar5 == 0)) {
              pcVar4 = "line too long";
              goto LAB_0000c8e4;
            }
            pcVar4 = strchrnul(acStack_198,0x23);
            pcVar4 = (char *)FUN_0000c484(acStack_198,pcVar4);
          } while (*pcVar4 == '\0');
          if (*pcVar4 != '[') break;
          pcVar6 = strchr(pcVar4,0x5d);
          if (((pcVar6 == (char *)0x0) || (pcVar6[1] != '\0')) ||
             (pcVar4 = (char *)FUN_0000c484(pcVar4 + 1), *pcVar4 == '\0')) {
            pcVar4 = "section header";
            goto LAB_0000c8e4;
          }
          iVar3 = strcasecmp(pcVar4,"SUID");
          if (iVar3 == 0) {
            iVar3 = 1;
          }
          else {
            iVar3 = -1;
          }
        }
        if (iVar3 != 1) break;
        pcVar6 = strchr(pcVar4,0x3d);
        if ((pcVar6 == (char *)0x0) ||
           (pcVar4 = (char *)FUN_0000c484(pcVar4,pcVar6), *pcVar4 == '\0')) {
          pcVar4 = "keyword";
          goto LAB_0000c8e4;
        }
        iVar5 = FUN_0000c56c();
        if (iVar5 != 0) {
          piVar7 = (int *)FUN_0003b378(0x14);
          iVar13 = 0;
          *piVar7 = iVar5;
          piVar7[4] = (int)piVar14;
          piVar7[3] = 0;
          pcVar6 = (char *)FUN_0003b184(pcVar6 + 1);
          __s = "Ssx-";
          pcVar4 = pcVar6;
          do {
            pcVar8 = strchrnul(__s,(uint)(byte)pcVar6[iVar13]);
            if (*pcVar8 == '\0') {
              pcVar4 = "mode";
              goto LAB_0000c8e4;
            }
            iVar5 = -0x41794 - iVar13;
            iVar13 = iVar13 + 1;
            piVar7[3] = piVar7[3] | (uint)*(ushort *)(&DAT_000417a4 + (int)(pcVar8 + iVar5) * 2);
            pcVar4 = pcVar4 + 1;
            __s = __s + 5;
          } while (iVar13 != 3);
          pcVar6 = (char *)FUN_0003b184(pcVar4);
          if ((pcVar6 == pcVar4) || (pcVar4 = strchr(pcVar6,0x2e), pcVar4 == (char *)0x0)) {
            pcVar4 = "<uid>.<gid>";
            goto LAB_0000c8e4;
          }
          *pcVar4 = '\0';
          uVar9 = strtoul(pcVar6,local_2c,10);
          piVar7[1] = uVar9;
          if ((*local_2c[0] != '\0') || (pcVar6 == local_2c[0])) {
            ppVar12 = getpwnam(pcVar6);
            if (ppVar12 == (passwd *)0x0) {
              pcVar4 = "user";
              goto LAB_0000c8e4;
            }
            piVar7[1] = ppVar12->pw_uid;
          }
          pcVar4 = pcVar4 + 1;
          uVar9 = strtoul(pcVar4,local_2c,10);
          piVar7[2] = uVar9;
          piVar14 = piVar7;
          if ((*local_2c[0] != '\0') || (pcVar4 == local_2c[0])) {
            pgVar11 = getgrnam(pcVar4);
            if (pgVar11 == (group *)0x0) {
              pcVar4 = "group";
              goto LAB_0000c8e4;
            }
            piVar7[2] = pgVar11->gr_gid;
          }
        }
      }
    } while (iVar3 != 0);
    pcVar4 = "keyword outside section";
LAB_0000c8e4:
    fprintf(stderr,"Parse error in %s, line %d: %s\n","/etc/busybox.conf",local_19c,pcVar4);
    fclose(__stream);
    while (piVar7 != (int *)0x0) {
      piVar14 = (int *)piVar7[4];
      free(piVar7);
      piVar7 = piVar14;
    }
  }
LAB_0000c924:
  DAT_00051fa4 = DAT_00051fa4 + 1;
  DAT_00051fa8 = (undefined4 *)FUN_0000c56c(param_1);
  if (DAT_00051fa8 == (undefined4 *)0x0) {
    if (DAT_00051fa4 == 1) {
      FUN_0000c5a0("busybox",param_2,param_3);
    }
    DAT_00051fa4 = DAT_00051fa4 + -1;
    return;
  }
  pcVar4 = (char *)*DAT_00051fa8;
  DAT_000566e4 = pcVar4;
  if ((*(char **)(param_3 + 4) != (char *)0x0) &&
     (iVar3 = strcmp(*(char **)(param_3 + 4),"--help"), iVar3 == 0)) {
    iVar3 = strcmp(pcVar4,"busybox");
    if (iVar3 == 0) {
      if (*(int *)(param_3 + 8) == 0) {
        DAT_00051fa8 = (undefined4 *)0x0;
      }
      else {
        DAT_00051fa8 = (undefined4 *)FUN_0000c56c();
        if (DAT_00051fa8 != (undefined4 *)0x0) goto LAB_0000c9b0;
      }
    }
    else {
LAB_0000c9b0:
      FUN_0000c4d8();
    }
    DAT_00051fb4 = 1;
    FUN_0000cc58(0,0);
  }
  puVar2 = DAT_00051fa8;
  _Var10 = getuid();
  __gid = getgid();
  piVar14 = DAT_00051fac;
  if (DAT_00051fb0 == 0) {
    bVar1 = *(byte *)(puVar2 + 2);
    if ((bVar1 & 0xf0) == 0x20) {
      _Var10 = geteuid();
      if (_Var10 != 0) {
        pcVar4 = "This applet requires root priviledges!";
LAB_0000cab8:
                    /* WARNING: Subroutine does not return */
        FUN_00036b4c(pcVar4);
      }
      goto LAB_0000cb28;
    }
    if ((bVar1 & 0xf0) != 0) goto LAB_0000cb28;
  }
  else {
    for (; piVar14 != (int *)0x0; piVar14 = (int *)piVar14[4]) {
      if ((undefined4 *)*piVar14 == puVar2) {
        uVar16 = piVar14[3];
        if (piVar14[1] == _Var10) {
          uVar16 = uVar16 >> 6;
          goto LAB_0000ca80;
        }
        if (piVar14[2] == __gid) goto LAB_0000ca7c;
        pgVar11 = getgrgid(piVar14[2]);
        if (pgVar11 == (group *)0x0) goto LAB_0000ca80;
        ppcVar15 = pgVar11->gr_mem;
        goto LAB_0000ca6c;
      }
    }
  }
  setgid(__gid);
  goto LAB_0000cb20;
LAB_0000ca6c:
  if (*ppcVar15 == (char *)0x0) goto LAB_0000ca80;
  ppVar12 = getpwnam(*ppcVar15);
  if ((ppVar12 != (passwd *)0x0) && (ppVar12->pw_uid == _Var10)) goto LAB_0000ca7c;
  ppcVar15 = ppcVar15 + 1;
  goto LAB_0000ca6c;
LAB_0000ca7c:
  uVar16 = uVar16 >> 3;
LAB_0000ca80:
  if ((uVar16 & 1) == 0) {
    pcVar4 = "You have no permission to run this applet!";
    goto LAB_0000cab8;
  }
  if ((piVar14[3] & 0x408U) == 0x408) {
    iVar3 = setegid(piVar14[2]);
    if (iVar3 != 0) {
      pcVar4 = "BusyBox binary has insufficient rights to set proper GID for applet!";
      goto LAB_0000cab8;
    }
  }
  else {
    setgid(__gid);
  }
  if ((piVar14[3] & 0x800U) != 0) {
    iVar3 = seteuid(piVar14[1]);
    if (iVar3 != 0) {
      pcVar4 = "BusyBox binary has insufficient rights to set proper UID for applet!";
      goto LAB_0000cab8;
    }
    goto LAB_0000cb28;
  }
LAB_0000cb20:
  setuid(_Var10);
LAB_0000cb28:
  iVar3 = (*(code *)DAT_00051fa8[1])(param_2,param_3);
                    /* WARNING: Subroutine does not return */
  exit(iVar3);
}

```
ok lots and lots and lots of stuff right here, but lets start from the begining, i wont let you go up, you can but you can trust me as well, the first target we need to see is this file:
`/etc/busybox.conf`, we can see it tries to read it over this function, so lets see what is going on in there.
and......
```bash
find . -iname "*conf"
./etc/mdev.conf
./etc/resolv.conf
./home/httpd/dmenu.conf
./home/httpd/project.conf
./home/httpd/checktoupper.conf
./home/httpd/langcn.conf
./home/httpd/langen.conf
```
shit, we cannot find this file, lets try to find every thing that relates to busybox.
and still, nothing...

ok so lets break down the function, dont worry, ill do it for both of us


# 🧙‍♂️ Function: FUN_0000c5a0 — The BusyBox Gatekeeper
“You shall not pass… unless you have the right UID!”

## 📜 What It Does
This is BusyBox’s bouncer.
Its job is to:

Peek inside /etc/busybox.conf (if it exists) to see what’s allowed.

Parse mysterious runes (config lines) to figure out who can run what.

Check your papers — UID, GID, and permissions.

If you pass, it summons the correct applet.

If you fail… 💥 instant existential crisis (error message + exit).

## 🕵️ Step-by-Step (loosely translated from “C” to “Human”)
* Check config file

* Looks for /etc/busybox.conf.

* If found, makes sure it’s owned by root and isn’t writable by mere mortals.

* Opens it and starts reading one line at a time.

* Parse config

* Strips comments (#) and trims whitespace.

* Recognizes sections like [SUID].

* Reads applet names, modes (S, s, x, -), and UID/GID assignments.

* Builds a “who’s allowed to do what” list.

* Load applet

* Uses your program_path and init params to figure out which BusyBox applet to run.

* Special case: --help just calls the help applet instead.

* Permission check

* Confirms if the current user (or group) matches the required UID/GID.

* If not, checks if you’re at least in the right group’s member list.

* If still no match → DENIED.

* Privilege setup

* If allowed, sets the correct effective UID/GID before running the applet.

* If not allowed, drops the mic and quits.

* Run the applet

* Calls the applet’s function pointer.

* Never returns. (The applet takes over execution.)

## 🤓 TL;DR
This function is basically:
```cpp
if (config says you’re allowed) {
    set up your royal permissions 👑
    run the requested BusyBox applet 🏃‍♂️
} else {
    slap you with a “NO ENTRY” sign 🚫
    and exit dramatically 💀
}
```
## 🐇 Fun Facts
Without /etc/busybox.conf, it still runs — just with less paranoia.

Mess up the file format? You get a “Parse error” roast on stderr.

Perfect place for privilege escalation bugs if misconfigured. 😉
but lets take a deep breath, and start going towards the init.rd files, to see what runs as the system begins, and may give us access to a life full of pleasure and "debug options"

```bash
_F601_V6.0.1P1T12_UPGRADE_BOOTLDR.bin.extracted/_140.extracted/cpio-root/etc/init.d$ file rcS 
rcS: POSIX shell script, ISO-8859 text executable
_F601_V6.0.1P1T12_UPGRADE_BOOTLDR.bin.extracted/_140.extracted/cpio-root/etc/init.d$ file regioncode 
regioncode: ASCII text
_F601_V6.0.1P1T12_UPGRADE_BOOTLDR.bin.extracted/_140.extracted/cpio-root/etc/init.d$ ll
total 20
drwxr-xr-x 2 bash bash 4096 Aug 11 20:12 ./
drwxr-xr-x 7 bash bash 4096 Aug 11 20:12 ../
-rwxr-xr-x 1 bash bash 4716 Aug 11 20:12 rcS*
-rwxr-xr-x 1 bash bash   24 Aug 11 20:12 regioncode*
```

ok 2 shell script executable, allways a good sign.
lets print out rcS:
```bash
_F601_V6.0.1P1T12_UPGRADE_BOOTLDR.bin.extracted/_140.extracted/cpio-root/etc/init.d$ cat rcS 
#!/bin/sh
#	$Id: rcS,v 1.7 2007/10/25 21:58:06 jwessel Exp $
# This is a minmal rcS file for target startup
# Make sure that /proc is mounted.
mount -a

#
#	Assign an address to the loopback device.
#
PATH=/sbin:/bin:/usr/sbin:/usr/bin
runlevel=S
prevlevel=N
umask 022

export PATH runlevel prevlevel
export VERBOSE

#
#	Trap CTRL-C &c only in this shell so we can interrupt subprocesses.
#
trap ":" INT QUIT TSTP

#
#	Call all parts in order.
#
for i in /etc/rcS.d/S??*
do
	# Ignore dangling symlinks for now.
	[ ! -f "$i" ] && continue

	case "$i" in
		*.sh)
			# Source shell script for speed.
			(
				trap - INT QUIT TSTP
				set start
				. $i
			)
			;;
		*)
			# No sh extension, so fork subprocess.
			$i start
			;;
	esac
done

#
# Resume default configuration file
# 0 - Default, 1 - Russia, 2 - Lithuania, 3 - Romania, 4 - Singapore
#
USR_DB_CFG_TYPE_MAX=300
USR_DB_DEFAULT_CFG_XML=/userconfig/cfg/db_default_cfg.xml
USR_DB_USER_CFG_XML=$USR_DB_USER_CFG_XML
USR_DB_BACKUP_CFG_XML=/userconfig/cfg/db_backup_cfg.xml
USR_CFG_TYPE_FILE=/userconfig/flag_type

### ������
ETC_DB_DEFAULT_CFG_XML=/etc/db_default_cfg.xml
ETC_DB_RUSSIA_CFG_XML=/etc/db_default_Russia_cfg.xml
ETC_DB_LITHUANIA_CFG_XML=/etc/db_default_Lithuania_cfg.xml
ETC_DB_ROMANIA_CFG_XML=/etc/db_default_Romania_cfg.xml
ETC_DB_SINGAPORE_CFG_XML=/etc/db_default_Singapore_cfg.xml

### ������, ���ա��½������ϵ��š��Ĵ����������Ϻ������졢���������ա�ɽ�����㶫
ETC_DB_JIANGSU_CFG_XML=/etc/db_default_Jiangsu_cfg.xml
ETC_DB_XINJIANG_CFG_XML=/etc/db_default_Xinjiang_cfg.xml
ETC_DB_HAINANDIANXIN_CFG_XML=/etc/db_default_Hainandianxin_cfg.xml
ETC_DB_SICHUAN_CFG_XML=/etc/db_default_Sichuan_cfg.xml
ETC_DB_HUBEI_CFG_XML=/etc/db_default_Hubei_cfg.xml
ETC_DB_SHANGHAI_CFG_XML=/etc/db_default_Shanghai_cfg.xml
ETC_DB_CHONGQING_CFG_XML=/etc/db_default_Chongqing_cfg.xml
ETC_DB_BEIJING_CFG_XML=/etc/db_default_Beijing_cfg.xml
ETC_DB_ANHUI_CFG_XML=/etc/db_default_Anhui_cfg.xml
ETC_DB_SHANDONG_CFG_XML=/etc/db_default_Shandong_cfg.xml
ETC_DB_GUANGDONG_CFG_XML=/etc/db_default_Guangdong_cfg.xml
ETC_DB_SUZHOU_CFG_XML=/etc/db_default_Suzhou_cfg.xml

ETC_DB_REGIONCODE=/etc/init.d/regioncode

copy_CFGFILE_by_REGIONCODE() {

	regioncode=$1	
	regionname=`busybox awk -F: -v regioncode="$regioncode" '$1==regioncode {print $2}' $ETC_DB_REGIONCODE`		
	echo "==================================================================="	
	echo "region code:$regioncode"	
	echo "region name:$regionname"			
	ETC_DB_CFG_XML=/etc/db_default_${regionname}_cfg.xml	
	if [ -f "$ETC_DB_CFG_XML" ]; then		
		busybox cmp $ETC_DB_CFG_XML $USR_DB_DEFAULT_CFG_XML 1>/dev/null 2>&1
		if [ $? -ne 0 ]; then
			echo "USR_DB_DEFAULT_CFG is different from ETC_DB_CFG!"
		echo "cp $ETC_DB_CFG_XML to $USR_DB_DEFAULT_CFG_XML "		
		cp -f $ETC_DB_CFG_XML  $USR_DB_DEFAULT_CFG_XML	
	else	    
			echo "USER_CFG is same as ETC_CFG, donot need copy"
		fi
	else	    
		echo "current : 0" > $USR_CFG_TYPE_FILE	
	fi	
	echo "==================================================================="
}

if [ ! -f $USR_DB_DEFAULT_CFG_XML ]; then
  cp -f $ETC_DB_DEFAULT_CFG_XML $USR_DB_DEFAULT_CFG_XML
  if [ -f $USR_DB_BACKUP_CFG_XML ]; then
	echo "  $USR_DB_BACKUP_CFG_XML found, deleted"
    rm -f $USR_DB_BACKUP_CFG_XML
  fi
fi

echo `date` > /userconfig/cfg/flag_usrfs
  
if [ ! -f $USR_CFG_TYPE_FILE ]; then
	echo "current : 0" > $USR_CFG_TYPE_FILE
	cp -f $ETC_DB_DEFAULT_CFG_XML $USR_DB_DEFAULT_CFG_XML
else
	idx=`cat $USR_CFG_TYPE_FILE | grep 'current' | sed -e 's/^current \: /\1/'`
	if [ -n "$idx" ]; then
	   
	  copy_CFGFILE_by_REGIONCODE $idx
	  
	  if [ $idx -gt $USR_DB_CFG_TYPE_MAX ]; then
	    echo "current : 0" > $USR_CFG_TYPE_FILE
		echo "  Warning: $idx unsupported, using default setting"
	  fi
	else
	  echo "current : 0" > $USR_CFG_TYPE_FILE
	fi
	
	idx=`cat $USR_CFG_TYPE_FILE | grep 'current' | sed -e 's/^current \: /\1/'`
	if [ $idx -eq 0 ]; then
		busybox cmp $ETC_DB_DEFAULT_CFG_XML $USR_DB_DEFAULT_CFG_XML 1>/dev/null 2>&1
		if [ $? -ne 0 ]; then
			echo "USR_DB_DEFAULT_CFG is different from ETC_DB_CFG!"
			echo "cp $ETC_DB_DEFAULT_CFG_XML to $USR_DB_DEFAULT_CFG_XML "		
	  cp -f $ETC_DB_DEFAULT_CFG_XML $USR_DB_DEFAULT_CFG_XML
		else
			echo "USER_CFG is same as ETC_CFG, donot need copy"
		fi
	fi
fi
echo "  Database default setting is [`cat $USR_CFG_TYPE_FILE`]"

#
# copy some files to /var/tmp/linux-igd, used by UPNP and SNTP
#
mkdir -p /var/tmp/linux-igd
cp -f  /etc/gatedesc.skl     /var/tmp/linux-igd/gatedesc.skl
cp -f  /etc/gateinfoSCPD.xml /var/tmp/linux-igd/gateinfoSCPD.xml
cp -f  /etc/gateicfgSCPD.xml /var/tmp/linux-igd/gateicfgSCPD.xml
cp -f  /etc/gateconnSCPD.xml /var/tmp/linux-igd/gateconnSCPD.xml

pc &

#
# host name
#
hostname -F /proc/csp/boardtype
```

lets start our beloved reality show

### *🛠 rcS – The Router’s Morning Routine*
"Wake up, make coffee, mount /proc." — ZTE F601, probably.

# 1️⃣ Mount All the Things
```sh
Copy
Edit
mount -a
```
Like making sure all your limbs are attached before getting out of bed — this mounts every filesystem in /etc/fstab.

# 2️⃣ Set the Stage
```sh
Copy
Edit
PATH=/sbin:/bin:/usr/sbin:/usr/bin
runlevel=S
prevlevel=N
umask 022
```
PATH – So commands don’t get lost on the way to work.

runlevel=S – Single-user mode. (Safe boots are for Mondays.)

umask 022 – Because letting everyone write to files is a bad idea.

# 3️⃣ Service Roll Call – /etc/rcS.d
For every file that starts with S?? in /etc/rcS.d:

If it ends with .sh, source it (fast and intimate).

Otherwise, fork it into a subprocess (the cold corporate handshake).

# 4️⃣ The Great Region Code Lottery 🎰
Because your router wants to know if it’s in:

Russia 🇷🇺

Singapore 🇸🇬

Lithuania 🇱🇹

Romania 🇷🇴

Or one of 14+ Chinese provinces 🀄

It runs copy_CFGFILE_by_REGIONCODE to pick the right /etc/db_default_<region>_cfg.xml.
If nothing matches? Default config, baby.

# 5️⃣ Backup? What Backup?
If the default user config doesn’t exist:

Copy it from /etc/ defaults.

Delete backups like a rebellious teenager hiding bad grades.

# 6️⃣ UPNP & SNTP Party Pack 🎉
```sh
Copy
Edit
mkdir -p /var/tmp/linux-igd
cp /etc/gatedesc.skl ...
```
Prepares XML files so devices can discover the router and keep time like civilized tech.

# 7️⃣ The Mysterious pc &
Launched in the background.
We don’t know what it does, but it sounds important… or like it could nuke the network from orbit.

# 8️⃣ Name Thyself
```sh
Copy
Edit
hostname -F /proc/csp/boardtype
```
Sets hostname based on the board type. Because identity is important.
and thats it.... i mean thats a lot, but thats it.
and the best part is right here:
```bash
for i in /etc/rcS.d/S??*
do
	# Ignore dangling symlinks for now.
	[ ! -f "$i" ] && continue

	case "$i" in
		*.sh)
			# Source shell script for speed.
			(
				trap - INT QUIT TSTP
				set start
				. $i
			)
			;;
		*)
			# No sh extension, so fork subprocess.
			$i start
			;;
	esac
done
```
it executes every file inside the rcS.d,
now we can look inside and understand what's going on, and also, in case we are a "bad hacker" we can drop there whatever script we want and it will get executed when the router will start, good and bad, but of course we are "doing that for a learning oppertunity and not to be the bad guys :]"
so lets see (for now by name) if there are any good scripts that can run there:

```bash
_F601_V6.0.1P1T12_UPGRADE_BOOTLDR.bin.extracted/_140.extracted/cpio-root/etc/init.d$ ls ../rcS.d/
S01usercfg  S20tsmac  S30network  S43BSPDriver  S50httpd  S99modules
```
ok hell yeah, everything looks just as dangerous and open as i thought it would be... so lets gooo!!!!
the first thing i will look through is the user config, it may set a default password or anything good for first entry :)


after wasting 2 hours of my life understanding what the heck went there, i can say its not important, it loads the base of the drivers, and run a quick test to make sure everything is good, so the next one will be S30network
but also here, i wont waste your time and say, this script just takes the network interfaces configurations from /sys and applys them :)
so httpd maybe the gold mine
ok its just starting up the apache services, could be vulnrable, but for tommorow, not today :).