### It is very likely that your storage or system has some issue, and there is not much we can do to fix that.

The Issue tracker is no place for helping with your computer.  
But, you are always welcome to make a topic on our forums, and we will try to help.

Use this wiki article to find out more about what's causing trouble.
And always try to fix the problem first yourself.  
Knowing how to fix computer problems such as this can come in very handy, and it is not hard to do at all.  
Just don't shy away from a little challenge. 

## Check the  `Execution Log`.
* On the top bar, click `View`, `Log` and then `Show`
* Click the new `Execution Log` tab on top

There are quite some data in here but worry not. Just scroll to the bottom and look out for _I/O errors_.  
It may contain useful information about why it's happening.  
Even if you can't make heads or tails of the lines, it will be helpful for the others who will try to help.

## Next step
There are two things that can cause (most likely) IO errors and stopped downloads.
* Permission problems
* Faulty drives

### Permission problems [Windows]

* If you have the default Download location set up, such as the `C:\Users\YOURUSERNAME\Downloads\` folder, your permissions should be fine. 
* If you have a custom Download location, make sure you have permissions to read and write there. From Windows 7, permissions have changed. Nowadays, you don't have full write permission by default to your entire C drive, for example, you can only write under your ``C:\Users\YOURUSERNAME` by default. You can, of course, change this - but I don't really recommend it. If you must make a new folder on C, I would make a simple `C:\a` folder, change permissions of that, and put everything I need inside. That way I have no mess, the permissions are alright, and so on.
* If you used a drive on some other PC, Windows remembers the permissions. For example, if you used an external hard drive or a drive with another Windows, or you re-installed your system - you might not have proper permissions anymore.

[How to manage permissions on Windows](https://technet.microsoft.com/en-us/library/cc754344(v=ws.11).aspx)

### Permission problems [Linux]

If you are a **simple user**, and you have your Download folder under your `/home` - which is the default behaviour - simply use `chown` to fix your permissions.

1. Open Terminal
2. Type: `sudo chown -R USERNAME:USERNAME /home/USERNAME`
Of course, USERNAME is your username. 

If you don't know what's your username, run the command `whoami`, it will tell you.
That's it, ty to download now, things should be working fine.

If you are a GNU/**advanced user**, check if you belong to the same group as the owner. If the group has RW permissions. Use chmod, chown to fix the permissions. 

### Hardware problems [Windows]

Check my [[other Wiki article|How to diagnose IO error, BSOD, crash (Windows)]]. See HDD section.

### Hardware problems [Linux]

Check my [other Wiki article](https://github.com/qbittorrent/qBittorrent/wiki/How-to-diagnostic-IO-error,-BSOD,-crash-%5BGNU-Linux,-BSD,-etc.%5D). See HDD section.