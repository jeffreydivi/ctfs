# PivotPlaneHinge
*CTF: CUCTF 2020*
*Category: Web*
*Writeup Author: Jeffrey DiVincent*

## Introduction

I tend to be more experienced with Web development, so I figured this would be a good challenge to try to attempt. However, before now, I never worked with WebSockets in any capacity (which was the core of this challenge), nor have I worked extensively with Burp Suite (I tend to use Charles Proxy more often). If beginners take anything out of this, is to just *try*! I see CTFs as a learning tool more than anything, so it ultimately doesn't matter if you fail to solve a challenge. What matters is that you *learned* something.

This challenge eventually inspired me to take on a [WebSockets project a week or two later for KnightHacks 2020](https://github.com/jeffreydivi/VoteyMcVotespace).

## First Observations

![Top of challenge page.](https://raw.githubusercontent.com/jeffreydivi/ctfs/master/2020-2021/cuctf_2020/misc/PivotPlaneHinge/images/0.png)

When loading the page, you are given an edit of the TempleOS website with a text box to control a WebSockets connection. All connection activities are then written to the top of the page.

For web debugging, I tend to use Charles Proxy, but it complained about the lack of secure WebSockets on my version, so I opted to use *Burp Suite*.

I noticed that the button that said `cat flag.txt` was just sending that string through the WebSockets connection and the server behaved differently as a result. I also noticed that the server allowed Burp to edit the contents of the WebSocket messages between the client and server, meaning they aren't being validated for integrity. This made me think: *maybe we can spoof a WebSockets message?*

### Initial Response on Connection

```json
{
  "introText": "Received websocket connect requestm: http://pivotplanehinge.ctf.cuctf.io:8079 : map[Accept:[*/*] Accept-Encoding:[gzip, deflate] Accept-Language:[en-US,en;q=0.5] Cache-Control:[no-cache] Connection:[keep-alive, Upgrade] Dnt:[1] Origin:[http://pivotplanehinge.ctf.cuctf.io:8079] Pragma:[no-cache] Sec-Websocket-Extensions:[permessage-deflate] Sec-Websocket-Key:[MS16YSchcX4qELfLc6NWoQ==] Sec-Websocket-Version:[13] Upgrade:[websocket] User-Agent:[Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:81.0) Gecko/20100101 Firefox/81.0]] Host: pivotplanehinge.ctf.cuctf.io:8079",
  "connectedHosts": [
    "admin 8.8.8.8",
    "user1 pivotplanehinge.ctf.cuctf.io:8079"
  ]
}
```

I also noticed the `connectedHosts` property with an `admin` user at `8.8.8.8` and `user1` at the challenge domain. Maybe this is what we can spoof?

## Spoofing the Hostname

### Find-and-replace
My first attempt at spoofing the hostname (so I can connect as `admin`) was to replace all instances of `pivotplanehinge.ctf.cuctf.io` in the request/response with `8.8.8.8` (string replacement). However, this messes up the HTTP connection and redirects the page to `8.8.8.8` (which we don't want). I tried a more focused find-and-replace to see what would be accepted, but this was fruitless. Some things were fine to edit (such as the user agent) while other ones (like anything referencing WebSockets) unsurprisingly broke WebSockets.

### Accessing from IP
I also tried to access the challenge from its IP address, `35.186.188.9`. The page loads, but no WebSocket connections are accepted.

### Editing the `hosts` file
I added a temporary entry to my `hosts` file to redirect the hostname `pph` to the challenge IP. The challenge did load as expected.

I also tried to replace `pph` in my entry with `8.8.8.8`. This did not work, and I'm shocked it didn't break Windows.

### PowerShell commands
After this, I tried to see if there were any Windows commands that can change routing so `8.8.8.8` loads the challenge page. After some searching, I found the `netsh` and `route` utilities, neither which did the trick.

### Back to *Burp Suite*
My initial attempt to search for a solution that used Burp Suite was fruitless, but after a while of searching, I stumbled upon [this forum post](https://forum.portswigger.net/thread/proxy-match-and-replace-target-ip-address-port-of-request-not-just-host-header-be63a7f1), which was the key to this challenge. I followed the directions it suggested to configure request handling to redirect to the challenge IP.

![Configuring request handling in Burp Suite](https://raw.githubusercontent.com/jeffreydivi/ctfs/master/2020-2021/cuctf_2020/misc/PivotPlaneHinge/images/1.png)

This did the trick nicely! I was able to load the challenge from `8.8.8.8`, sent the `cat flag.txt` command, and I got the flag! 🎉

![Loading PivotPlaneHinge from 8.8.8.8](https://raw.githubusercontent.com/jeffreydivi/ctfs/master/2020-2021/cuctf_2020/misc/PivotPlaneHinge/images/2.png)
