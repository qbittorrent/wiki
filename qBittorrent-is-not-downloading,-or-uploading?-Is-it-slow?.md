**First and foremost**: Make sure you are using the latest qBittorrent available for your system.  
You can download the latest version from here: https://qbittorrent.org/   
  
With that out of the way, let's see what can go wrong.  
The easiest way to see where the problem lies is, disabling everything in between your PC and your Internet Modem. So basically bring your PC to the modem, hook up directly without any router, wifi, anything. Disable Proxy and VPN if you have any and test. Does it work? No? Then try it with a second PC. Still not? Head to our forums and we will help you.
  
### VPN or Proxy?  
* If you are using a VPN or Proxy, disable and check if that solves your issue. By default, both solutions render you passive, puts you behind a firewall, so to say. Some VPN providers offer active port forwarding, but it varies by provider.  
   
### Other issues/symptoms...
  
* Check if you have an **Anti-Virus** or Security solution that comes with a Firewalll. That can cause high CPU usage, connection problems. Thus, affecting your speed and experience.  
* If you use a **router** in your network, make sure you do a port forward by hand. UPnP (the feature that routers use to allow automatic port forward) can be really buggy and they barely ever work. Just add a port for TCP and UDP each by hand, and you are good.  
  
  
### Still not working?  
* Post about it on the forums. Let others help figure out your connection issues: https://qbforums.shiki.hu/  
If it is confirmed there after your post, that it's a bug in qBittorrent, make the report. But!  
Include _every_ detail.  
  
* Your OS  
* The qBittorrent version you are using  
* What kind of trackers you have been using? Private? Public? You can mention the tracker name too, that's no secret.
* Architecture (x86 or x86_64, maybe ARM64) you are using  
* Network type: Fiber, cable, DSL, radio/microwave  
* Router: Do you have one? Do you have more than one? Do you have switches in the way?  
* How do you connect to the closest access point? Wifi? Cable? Power-ethernet?  
* What HDDs, SSDs do you download to? Do you use RAID? If yes, hardware, SW? If HW, what adapter/device?  
* Did you check your storage if it works, if it does not have any SMART issues? Use Crystal Disk Info to make sure that the storage is fine. ( https://crystalmark.info/en/software/crystaldiskinfo/ )  
* Did you check your RAM if it is flawless/has no issues? ( https://www.howtogeek.com/260813/how-to-test-your-computers-ram-for-problems/ )
