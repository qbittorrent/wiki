# qBittorrent gave you a BSOD (Blue Screen of Death). Now what?

Well, the thing is, it was not qBittorrent.
There are three major things that can cause these symptoms.

* Your RAM
* Your HDD/Storage
* Your Power Supply

# How to check RAM

## Linux
In short, on 99% of the Linux distributions out there, there will be a "Memtest", "Check memory", or another similar entry in your boot loader's menu. Select that. 

* https://askubuntu.com/questions/343114/how-to-check-for-errors-in-ram-via-linux  
* https://help.ubuntu.com/community/MemoryTest

## Any OS
* http://www.memtest86.com  
* http://www.howtogeek.com/260813/how-to-test-your-computers-ram-for-problems/

Basically you have to download Memtest86 and put it on a pendrive, CD or DVD.
Boot it, and it will check your RAM.

If you can't get Memtest86 to work, a live Ubuntu ISO will work just as well.
Download the latest Ubuntu LTS version (16.04 at the moment), and put it on a pendrive, CD or DVD.

# How to check the HDD/storage

## Windows
There are many great free tools for this.
* https://hddguardian.codeplex.com  
* http://crystalmark.info/software/CrystalDiskInfo/index-e.html

## Ubuntu
https://askubuntu.com/questions/528072/how-can-i-check-the-smart-status-of-a-drive-on-ubuntu-14-04-through-16-10

## Linux / BSD
https://askubuntu.com/questions/38566/how-can-i-check-the-health-of-my-hard-drive  

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