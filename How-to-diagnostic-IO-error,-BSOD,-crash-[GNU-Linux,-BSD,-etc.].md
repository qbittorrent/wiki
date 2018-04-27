# This wiki page is still work in progress! Please, check back later.
# Version 0.2

## qBittorrent gave you a BSOD (Blue Screen of Death). Now what?

Well, the thing is, it was not qBittorrent.
There are four major things that can cause a freeze or BSOD.
The numbers mean how hard is it to check on a scale of 5.

* Drivers, faulty devices (1/5)
* Your RAM (2/5)
* Your HDD/Storage (2/5)
* Your Power Supply (4/5)
* Something else

# How to check RAM

## Windows

* https://www.cnet.com/how-to/test-your-ram-with-windows-memory-diagnostic-tool/

## Linux
In short, on 99% of the Linux distributions out there, there will be a "Memtest", "Check memory", or another similar entry in your boot loader's menu. Select that. 

* https://askubuntu.com/questions/343114/how-to-check-for-errors-in-ram-via-linux  
* https://help.ubuntu.com/community/MemoryTest

## Any OS
* https://www.memtest86.com  
* https://www.howtogeek.com/260813/how-to-test-your-computers-ram-for-problems/

Basically you have to download Memtest86 and put it on a pendrive, CD or DVD.
Boot it, and it will check your RAM.

If you can't get Memtest86 to work, a live Ubuntu ISO will work just as well.
Download the latest Ubuntu LTS version (16.04 at the moment), and put it on a pendrive, CD or DVD.

# How to check the HDD/storage

## Windows
There are many great free tools for this.
* https://hddguardian.codeplex.com  
* https://crystalmark.info/en/software/crystaldiskinfo/

## Ubuntu
* https://askubuntu.com/questions/528072/how-can-i-check-the-smart-status-of-a-drive-on-ubuntu-14-04-through-16-10

If you can't figure out how to use that tool, read below.

## Linux / BSD
* https://askubuntu.com/questions/38566/how-can-i-check-the-health-of-my-hard-drive  

You want to check:
* Reallocated_Sector_Ct
* Current_Pending_Sector
* Power_On_Hours

* Reallocated_Sector_Ct:
> Once this reaches 1, or any higher, the HDD is a ticking bomb. Back up and replace ASAP. Only store throwaway data on it.

* Current_Pending_Sector: Big trouble. Answer from AskUbuntu:
> A sector is marked pending when a read fails. The pending sector will be marked reallocated if a subsequent write fails. If the write succeeds, it is removed from current pending sectors and assumed to be ok. (The exact behavior could differ slightly and I'll go into that later, but this is a close enough approximation for now.)

* Power_On_Hours:
> How many hours your HDD ran. HDDs usually die around 20-30k hours. Some may run up to 60k. It is totally random, and there is no guarantee a hard drive will run to X hours. It's however a good idea to back up more often and worry once you reach a lot of hours.

# How to check PSU
This one is very tricky.
The best and easiest method is to switch the PSU to an other one. Use the computer, try every application, game and whatnot. If the computer is now stable, then it was your PSU.

Of course, most people don't have a second PSU lying around.
In this case it's usually a good test to put load on your PC.

## Windows

Run the two programs at the same time.
Prime95 stresses your CPU, Furmark stresses your GPU.

If your PC is stable after a few minutes, just close them.
If you have a BSOD, or the computer restarts/turns off, you have a bad PSU.

# Something else
Check if the cables to your HDD/things are connected firmly, and nothing is lose. This can also cause trouble.
If nothing helped, your motherboard can also be faulty.
