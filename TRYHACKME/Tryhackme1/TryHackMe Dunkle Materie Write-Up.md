


![](https://miro.medium.com/v2/resize:fit:205/0*7J0gZdtf0F6NcJfd.png)

## Intro

This TryHackMe room is rated medium difficulty, awarding 300 points upon full completion. We are given access to a virtual machine and two analysis artifacts: a **Procmon CSV logfile** and a **PCAP file** containing captured network traffic.

[

## Dunkle Materie

### Investigate the ransomware attack using ProcDOT.

tryhackme.com



](https://tryhackme.com/room/dunklematerieptxc9?source=post_page-----b37cbb0860c0---------------------------------------)

The room relies heavily on ProcDOT, a visualization tool designed to correlate process activity, file system operations, registry changes, and network traffic into an interactive graph. Throughout this lab, we will use the ProcDOT application provided with the VM to load and correlate both data sources, analyze the resulting graphs, and piece together the malware’s behavior step by step.

Also, the following documentation proved to be useful: [https://www.procdot.com/onlinedocumentation.htm](https://www.procdot.com/onlinedocumentation.htm)

### Scenario

_Investigate the ransomware attack using ProcDOT_.

The firewall alerted the Security Operations Center that one of the machines at the Sales department, which stores all the customers’ data, contacted the malicious domains over the network. When the Security Analysts looked closely, the data sent to the domains contained suspicious base64-encoded strings. The Analysts involved the Incident Response team in pulling the Process Monitor and network traffic data to determine if the host is infected. But once they got on the machine, they knew it was a ransomware attack by looking at the wallpaper and reading the ransomware note.

Can you find more evidence of compromise on the host and what ransomware was involved in the attack?

## Tasks

1. _Provide the two PIDs spawned from the malicious executable. (In the order as they appear in the analysis tool)_

First, let’s load up our file to ProcDOT from the ‘Monitoring Logs’ section, then, in ‘Render Configuration’, click the three dots(…) and wait for the file analysis to finish. We will be greeted with a new window. Looking at it, nothing seems weird, until we see one of the processes with name **exploreer.exe**. That is suspicious, as **explorer.exe** is a core Windows process. for By searching it’s name, we find 2 processes:

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*cXQK8ilK5L-CNst5tq10Rw.png)

The wanna-be explorer.exe Process

You can see process PIDs listed as well.

**_Answer: 8644,7128_**

_2. Provide the full path where the ransomware initially got executed? (Include the full path in your answer)_

My thinking was, that for the process creation, the executable file must be initiated. So, another process must start it before, right? Let’s select the process with PID 4: System, and click refresh.

![](https://miro.medium.com/v2/resize:fit:235/1*DuxKFtW_glDlK3RJnzL-5g.png)

Woah, this looks scary

Let’s search for explorer.exe. We can use the shortcut CTRL+F. We can see some results highlighted, but I think it is best to zoom in, and see what’s what. Unfortunately, the (+) or (-) keys do not work in the provided VM, so we need to manually zoom in from ‘View > Graph > Zoom in’. Zooming in, we see a separate red cell.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*Mlt2d1XmPBbj-f3gOlPV6g.png)

We found the answer to the question.

**_Answer: c:\users\sales\appdata\local\temp\exploreer.exe_**

3. _This ransomware transfers the information about the compromised system and the encryption results to two domains over HTTP POST. What are the two C2 domains? (no space in the answer)_

Skimming through the documentation, it seems that URLs and domains are usually encapsulated in blue cells. We can see many of them, but we specifically need the ones, that were initiated by the process in question. Literally just a bit left on the graph we can see some interesting ones, pointing to PID 7128:

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*52apTmS7Kl5z4hdI9YdfpQ.png)

There are AT LEAST 3 URLs associated with the process

Now, I don’t think that I would view cisco.com as a C2 domain, that leaves us with the other 2: mojobiden.com and paymenthacks.com. Just to make sure, let’s feed them to VirusTotal. For both of them, looking in details, they confirm our suspicion.

**_Answer: mojobiden.com,paymenthacks.com_**

_4. What are the IPs of the malicious domains? (no space in the answer)_

We literally can see the IPs on the cells.

![](https://miro.medium.com/v2/resize:fit:306/1*7zGz20diS5LX7qvKP432jA.png)

Pretty visible, huh?

**_Answer: 146.112.61.108,206.188.197.206_**

[](https://medium.com/write?source=promotion_paragraph---post_body_banner_jsw_blocks--b37cbb0860c0---------------------------------------)

_5. Provide the user-agent used to transfer the encrypted data to the C2 channel._

The user agent is not visible directly in ProcDOT, but we can still obtain it by opening the URL in Wireshark or by following the TCP stream. Since ProcDOT’s built-in TCP stream view is somewhat unreliable, it is better to open the traffic in Wireshark by right-clicking on the relevant cell and selecting “Open Packets in Wireshark.”

Let’s do this for **paymenthacks.com**.

We know, from the previous question, that the request was POST. so let’s apply a filter: **http.request.method==”POST”**.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*1pe_raMt93cRCjZvHZFVjw.png)

Gives us nothing

Now let’s look at the second domain, which gives us results. Follow TCP Stream.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*fLyGczUI7rgvy0Sr-SLKAQ.png)

User-Agent is visible.

**_Answer: Firefox/89.0_**

_6. Provide the cloud security service that blocked the malicious domain._

Remember about cisco.com and why it appeared in the graph? Well, the question itself gives us a hint. Looking at the same screen and scrolling it down, we can see that Cisco Umbrella has blocked the malicious domain with HTTP response status code 403: Forbidden

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*wkhSiQkRbRuKNFdNKlkFkQ.png)

**_Answer: Cisco Umbrella_**

_8. Find the PID (Process ID) of the process which attempted to change the background wallpaper on the victim’s machine._

Searching for “wallpaper” keyword on the current graph did not give me any results. Another thing to remember is, that per user, Wallpaper settings are stored under windows registry `HKEY_CURRENT_USER\Control Panel\Desktop`. Let’s change the render configuration to PID 7128: exploreer.exe, to see what it has initiated. Click refresh again. Quickly searching for “wallpaper” gives us it’s results.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*87CxY4j2YPW-PExJzgIZvQ.png)

The Registry keys are a bit off tho what I wrote a bit earlier, but doesn’t matter

We can see, that this process attempted changing the background wallpaper on the victim’s machine.

**_Answer: 4892_**

_7. Provide the name of the bitmap that the ransomware set up as a desktop wallpaper._

I answered the question 8 earlier then 7, because we first find the registry entries and the process ID, and that’s what is visible immediately. However, to answer the current question, we need to right click on the first registry cell > Details, and on ‘Data’ key, click ‘View’ for the value

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*i-biQQPHAxTbQWS8NXYE4g.png)

We can see the name of the bitmap here.

**_Answer: ley9kpi9r.bmp_**

_9. The ransomware mounted a drive and assigned it the letter. Provide the registry key path to the mounted drive, including the drive letter._

The same process, with PID 4892 also has changed another registry value, mounting a drive and assigning it the letter Z:, and we can see it on the graph right below the wallpaper registry cells.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*-wX3xjrbh83ruENMH8kCDg.png)

One of 4 registries, that answers this question

**_Answer: HKLM\SYSTEM\MountedDevices\DosDevices\Z:_**

_10. Now you have collected some IOCs from this investigation. Provide the name of the ransomware used in the attack. (external research required)_

A bit earlier we searched for the domains externally on VirusTotal anyway, and my eyes caught something interesting, associated with the domains under details, where google results are also showing up. In each one of them, paymenthacks.com is mentioned. Opening up the first search result, we are directed to this website: [https://www.group-ib.com/blog/blackmatter-ransomware/](https://www.group-ib.com/blog/blackmatter-ransomware/). Just to make sure, we search for the domain on the page, and indeed, it is the one:

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*D8qUn3erBjavqnt5w8uVZQ.png)

**_Answer: Blackmatter Ransomware_**