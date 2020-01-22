# Introduction

This guide assumes you have setup your WebUI correctly and that you are sure your problem is not due to a network/reverse proxy/firewall/etc. issue.

If you have everything setup correctly and the WebUI stops updating or never loads past a blank page in the first place, the most likely cause is a crash in the Javascript code. This guide helps you pinpoint the cause of the crash, so that you can report it or even fix it yourself.

# First things first

1. Clear your browser's cache and restart it.

2. Check if the problem persists.

# Obtain a stack trace of the WebUI code

This helps figure out _where_ exactly the WebUI is crashing, and is essential for then trying to figure out _why_.

1. Open the browser's web developer tools (Ctrl+Shift+I in Firefox) and go to the `Console` tab.

2. In the top right, activate all message categories by clicking on them so that they are highlighted.

    ![cat](https://user-images.githubusercontent.com/17580742/72902338-31fc7600-3d23-11ea-8a6a-794c80447811.png)

   If your issue only happens with slow connections, or you would like to simulate a slow connection (suppose you suspect there is a race condition and you want a better chance at manually triggering it), go to the `Network` tab and select `GPRS` (or something a bit faster) from the `No Throttling` drop-down menu:

    ![throttle](https://user-images.githubusercontent.com/17580742/72903713-a7694600-3d25-11ea-84e3-b17fa81b940d.png)

3. With the developer tools opened, reload the page and execute any additional steps that lead to the problem.

4. If you see something like this pop up in the console, click on the arrow (left side, next to the timestamp) to open the stack trace:

    before:
    ![typeerror1](https://user-images.githubusercontent.com/17580742/72902695-d67eb800-3d23-11ea-9bfb-92ca73e899e9.png)
    after:
    ![typeerror2](https://user-images.githubusercontent.com/17580742/72902698-d8487b80-3d23-11ea-9920-8fa58e988bdb.png)

This is enough data for a useful issue report. Now you can create an issue report containing:

- the contents of the stack trace (preferably pasted, not screencapped).
- as a screencap of what the WebUI looks like.
- Anything else that could be useful, such as a description of the steps that led to the crash. See the [contributing guidelines](https://github.com/qbittorrent/qBittorrent/blob/master/CONTRIBUTING.md) for more tips and instructions on what to include when posting an issue report.

# Next steps

If you have web development experience, you already know where to go from here. Submit a PR if you can fix the issue :).

If you don't have web development experience but want to investigate further and possibly attempt to fix the problem, you can click the stack trace lines to go to the problematic code in the debugger. From there you can setup breakpoints, watchpoints, step through the code, etc. Reload the page and just do some old-school debugging. Search for tutorials online about debugging web applications and using the browsers developer tools.

This wiki page may be updated in the future with specific steps/techniques that happen to be particularly useful/relevant to qBittorrent.