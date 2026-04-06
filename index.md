---
layout: default
---
# Linux Server Hardening & SSH Brute Force Simulation


I used ubuntu server on Virtualbox as a target, and Kali linux also on Virtualbox as attacker.


After basic installation of both machines I configured ssh service and changed default port to `2222`

![Branching](https://raw.githubusercontent.com/Aleksander9191/SecurityProjects/43bee26df1e142090b56fc73995b468c7f772369/ssh%20config.png)

rule with port didn't apply as **ssh.socket** was running and ignoring my  `sshd_config` change so it had to be turned off

![Branching](https://raw.githubusercontent.com/Aleksander9191/SecurityProjects/a71c6cf55f24638f194ada5c4af59c4952bedec2/ssh%20socket.png)

Then, it was time for firewall installation:
```Bash
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp
sudo ufw enable
```
blocking all incoming traffic and allowing ssh on custonm port

next, installation **fail2ban** on a server and its basic configutation


**fail2ban**  is a security tool that monitors system logs for suspicious activity, such as repeated failed login attempts. When it detects such behavior, it automatically blocks the offending IP address using firewall rules for a specified period

![Branching](https://raw.githubusercontent.com/Aleksander9191/SecurityProjects/468818efd4ff42980b495a2af8fc299bf935aab7/jDDOCJD.png)

`maxretry` - allowed password attempts

`bantime` - ban duration in seconds

Then I had to configure network on both machines -  Host-only on both so they comunicate only between each other


After succesful ping between devices I sent an attack from Kali
```Bash
hydra -l admin1 -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.108 -s 2222
```
As fail2ban was active I could display failed login attempts real time from logs
```Bash
tail -f /var/log/auth.log
```
![Branching](https://raw.githubusercontent.com/Aleksander9191/SecurityProjects/468818efd4ff42980b495a2af8fc299bf935aab7/atak.png)



Kali ip was added to banned ip list in fail2ban status:

![Branching](https://raw.githubusercontent.com/Aleksander9191/SecurityProjects/a906d5fc5db1a1bbf1a0b0a875435c76bd54bb0d/fail2ban%20jail%20ip.png)


In the end I used grep to get clear display of attempts from logs
```Bash
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c

```
this variation shows number of attempted logins and its ports (in this case we had 2 attemps on every shown port)

```Bash
grep "Failed password" /var/log/auth.log | awk '{print $9}' | sort | uniq -c
```
this on the other hand shows number of attempts from a specific IP 

![Branching](https://raw.githubusercontent.com/Aleksander9191/SecurityProjects/404a5359aa953f7ef4e3be1ae1fd9f9b97a00aa1/grep.png)


As it is my first ever experience with publish my lab practices its a bit clunky, and basic - later on it will develop better fluency and "flow", gained from experience





