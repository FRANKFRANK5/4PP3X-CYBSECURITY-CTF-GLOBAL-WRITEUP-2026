The Splunk platform helps IT and security teams to ensure their organizations are secure, strong and keeping up on the advancement of technology, and likewise the cybercriminals. In this Conti Ransomware room, we identify ransomware on Splunk logs.

**Scenario:**

Some employees from your company reported that they can’t log into Outlook. The Exchange system admin also reported that he can’t log in to the Exchange Admin Center. After initial triage, they discovered some weird readme files settled on the Exchange server. Below is a copy of the ransomware note.

You are assigned to investigate this situation. Use Splunk to answer the questions below regarding the Conti ransomware.

**Question 1:** Can you identify the location of the ransomware?

**Answer: c:\Users\Administrator\Documents\cmd.exe**

**Step 1:** Based on question 2, have determined that this is a file create event, so we find out in the below article event ID used to filter this type of event.

[https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

We search the EventCode=11 on the search field and hit All Time on the drop down on the upper left, then click on image on interesting fields section. Looking at it we can see highlighted section- cmd.exe executable is stored in a strange location.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*ZfdnXaWetc68qAk0l8eWVw.png)

**Question 2:** What is the Sysmon event ID for the related file creation event?

**Answer: 11**

**Step 2:** Basically, the answer is given on Step 1.

**Question 3:** Can you find the MD5 hash of the ransomware?

**Answer: 290C7DFB01E50CEA9E19DA81A781AF2C**

**Step 3:** To identify this, click on the image file cmd.exe as seen in Step1 and simply remove the event ID filter, then add MD5. We can see a single event returned, and highlighted on the snapshot is the MD5 hash.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*2vhDdWX3iTOSZs2kFsewTw.png)

**Question 4:** What file was saved to multiple folder locations?

**Answer: readme.txt**

**Step 4:** Search this filter “Image=”c:\\Users\\Administrator\\Documents\\cmd.exe” EventCode=11”, and after that click on TargetFileName on the interesting fields section, we can see below result.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*3_W9bLjY49NAdv_dg79M0A.png)

**Question 5:** What was the command the attacker used to add a new user to the compromised system?

[](https://medium.com/download-app?source=promotion_paragraph---post_body_banner_surround_scribble--c0a75d389ae8---------------------------------------)

**Answer: net user /add securityninja hardToHack123$**

**Step 5:** Filter out using CommandLine and wildcard on add to check all commands containing _add_. Below is the result command.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*_V0Uomzlnlp30xXQJjnZ7Q.png)

**Question 6:** The attacker migrated the process for better persistence. What is the migrated process image (executable), and what is the original process image (executable) when the attacker got on the system?

**Answer: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe,C:\Windows\System32\wbem\unsecapp.exe**

**Step 6:** We can use Sysmon Event ID 8 Create Remote Thread, which detects when a process creates a thread in another process. In the interesting fields section, click on TargetImage and it will show two events, click on the second one and you will find source and target locations as below snapshot.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*zd1ZPpPM9fFGYYMjIjchVg.png)

**Question 7:** The attacker also retrieved the system hashes. What is the process image used for getting the system hashes?

**Answer: C:\\Windows\\System32\\lsass.exe**

**Step 7:** Using the EventCode=8 filter alone, check the TargetImage and it will show two events, if we refer to the output from the search used in question 6, we can see that a second process migration takes place between unsecapp.exe and lsass.exe

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*LQU7fvj_odrIcMk1qey2CQ.png)

**Question 8:** What is the web shell the exploit deployed to the system?

**Answer: i3gfPctK1c2x.aspx**

**Step 8:** We get a hint for this one, so we filtered IIS events for POST requests and common web shell file types like .aspx, used the wildcard *.aspx*. Then, under the “cs_uri_stem” in the interesting field, a suspicious looking filename with a “.aspx” file extension is seen, see below snapshot.

![](https://miro.medium.com/v2/resize:fit:354/1*Dkly_GCW3lnac7X67vDaeg.png)

![](https://miro.medium.com/v2/resize:fit:529/1*A479POUCQ2tEjYRy7dydwg.png)

**Question 9:** What is the command line that executed this web shell?

**Answer: attrib.exe -r \\\\win-aoqkg2as2q7.bellybear.local\C$\Program Files\Microsoft\Exchange Server\V15\FrontEnd\HttpProxy\owa\auth\i3gfPctK1c2x.aspx**

**Step 9:** Filtered for the .aspx web shell “i3gfPctK1c2x.aspx” in search field and added commandline filter as well, it returned one event. Click the event to view and find the full commandline, see snapshot for reference.

![](https://miro.medium.com/v2/resize:fit:551/1*ljd_Ft8vN06svYjiRS4mNQ.png)

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:560/1*5a_a2wxJdHlVWLa1izyb-g.png)

**Question 10:** What three CVEs did this exploit leverage?

**Answer: CVE-2020–0796,CVE-2018–13374,CVE-2018–13379**

**Step 10:** We get a hint where external research is required. This article contains a number of CVE’s related to Conti Ransomware. [https://cybersecurityworks.com/blog/ransomware/is-conti-ransomware-on-a-roll.html](https://cybersecurityworks.com/blog/ransomware/is-conti-ransomware-on-a-roll.html)