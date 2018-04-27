## qBittorrent gave you a lockup or BSOD (Blue Screen of Death). Now what?

First of all, if you get a BSOD, and it's not `irql_not_less_or_equal`, there is a good chance you will be able to debug the BSOD. It is a very powerful and useful tool to find why your Windows system just crashed.  

You must also know that the cause of your lockup/BSOD was not qBittorrent.
There are four major things that can cause a freeze or BSOD.
The numbers mean how hard is it to check on a scale of 5.

* [Drivers](#how-to-diagnose-bsod) (1/5)
* [Your RAM](#how-to-check-ram) (2/5)
* [Your HDD/Storage](#how-to-check-the-hdd-or-storage) (2/5)
* [Your Power Supply](#how-to-check-psu) (4/5)
* [Something else](#something-else) (5/5)

***

## How to diagnose BSOD

! Take a picture of the BSOD message, every detail. Use your phone, or anything you have at your disposal. You have a few seconds until Windows finishes writing out the memory dump. **Wait until it says 100%/ready, otherwise you will have nothing to work with.**

**If you get the `irql_not_less_or_equal`, it is most likely a hardware issue.**

1. Install the [Windows SDK](https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk). You only need to select the "Debugging tools" at the installation, which is only a mere ~250MB.
2. Launch the freshly installed "windbg" _as Administrator_ tool.
3. Add remote symbols. 
File -> Symbol File Path: `srv*c:\symbols*https://msdl.microsoft.com/download/symbols`
The `c:\symbols` will be your local symbol cache. I would suggest using something more "writeable".
4. File -> Open Crash Dump.
5. Navigate to `C:\Windows\`
6. Open `memory.dmp`
7. Wait. Once you see "Ready", use the `!analyze -v` command.
8. It will tell you the most likely culprit, but the other loaded things are also suspects. For me, the "most likely" thing was a Windows DLL, but my Avast file scanner service was also there. Removed Avast - boom - no BSOD ever since.


***

## How to check RAM

### Windows in-built tool

* https://www.cnet.com/how-to/test-your-ram-with-windows-memory-diagnostic-tool/

### Alternatives
In case Windows's in-built test does not work for you.
* https://www.memtest86.com  
* https://www.howtogeek.com/260813/how-to-test-your-computers-ram-for-problems/

You have to download Memtest86 and put it on a pendrive, CD or DVD.
Boot it, and it will check your RAM.
I usually stop after 1 PASS, but you can wait until 2 PASS or so. If there are no errors during the test, your ram is OK.

If you can't get Memtest86 to work, a live Ubuntu ISO will work just as well.
Download the latest Ubuntu LTS version (16.04 at the moment), and put it on a pendrive, CD or DVD.


***


## How to check the HDD or storage

### Windows
There are many great free tools for this.
* https://hddguardian.codeplex.com  
* https://crystalmark.info/en/software/crystaldiskinfo/

### Ubuntu
If you can't boot Windows, you can use an [Ubuntu Live CD](https://www.howtogeek.com/191054/how-to-create-bootable-usb-drives-and-sd-cards-for-every-operating-system/) to diagnose your HDD.
* https://askubuntu.com/questions/528072/how-can-i-check-the-smart-status-of-a-drive-on-ubuntu-14-04-through-16-10
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

***

## How to check PSU
This one is very tricky.
The best and easiest method is to switch the PSU to an other one.
Use the computer, try every application, game and whatnot.
If the computer is now stable, then it was your PSU.

Of course, most people don't have a second PSU lying around.
In this case, it's usually a good test to put load on your PC.

### Windows

Run the two programs at the same time.
[Prime95](https://www.mersenne.org/download/) stresses your CPU, [FurMark](http://www.ozone3d.net/benchmarks/fur/) stresses your GPU.

If your PC is stable after a few minutes, just close them.
If you have a BSOD, or the computer restarts/turns off, you have a bad PSU.

Your PSU might also be able to deliver stable power during stress (not likely), so also test some other situations where you had a freeze/lockup.

## Something else
Check if the cables to your HDD/things are connected firmly, and they are not loose. This can also cause trouble.
If nothing helped, your motherboard can also be faulty. An anti-virus can also cause BSOD, but it will show up with "windbg".
