---
layout: post
title: "ERR_CONNECTION_REFUSED"
date: 2015-08-27 22:19
comments: true
categories: 
---

Today I have faced with the problem installing Basic Authentication feature into the Web Server role on Windows 2012 R2. The wizard kept throwing various errors, including scary OutOfMemoryException. A quick googling that has found a suggestion to run `netsh http show iplisten` and add 127.0.0.1 (aka Home Sweet Home) to the list if it's not there. I gave it a try without giving it a thought first.

The initial problem has not been solved - the wizard kept failing to add that feature, and I finally resolved it with the mighty PowerShell:

```powershell
Import-Module ServerManager
Add-WindowsFeature Web-Basic-Auth
```

Later on I had to browse for a website hosted on that server, and I suddenly saw *This webpage is not available* message. Hmm... First off, I've verified that the website works locally - and it did. This gave me another hint and I checked whether the bindings are set up correctly. And they were! Finally, I started to think that it's Basic Authentication feature to blame - yeah, I know, that was a stupid assumption, but hey, stupid assumptions feel very comfortable for brain when it faces with the magic...

Anyway, fortunately I recalled that quick dumb action I did with netsh, and the magic has immediately turned into the dust, revealing someone's ignorance... Turns out, if `iplisten` does not list anything, it means listen to everything, any IP address. When you add something there, it starts listening to that IP address only. 

Thus, it was all resolved by deleting 127.0.0.1 from that list with `netsh http delete iplisten ipaddress=127.0.0.1`.

Want some quick conclusion? **Think first, then act!!!**
> Written with [StackEdit](https://stackedit.io/).