
Création de `add_adminconfid.ldif` et modification du `olcRootPW`:
![[Pasted image 20230320110326.png]]

Connect to config with `jxplorer`:
![[Pasted image 20230320113554.png]]


### Add people and groups OU

![[Pasted image 20230320141406.png]]
command to add it to ldap:
```bash
root@ServeurLDAP:/etc/ldap $ ldapadd -H ldapi:/// -x -D "cn=admin,dc=centralesupelec,dc=fr" -f ./add_people_groups.ldif -W
Enter LDAP Password: 
adding new entry "dc=centralesupelec,dc=fr"

adding new entry "ou=people,dc=centralesupelec,dc=fr"

adding new entry "ou=groups,dc=centralesupelec,dc=fr"
```


KERBEROS:

Récupération de ticket kerberos pour l'user vagrant:
`sudo kinit vagrant` vient récupérer un ticket.

Configuration du fichier samba avec les informations permettant de se connecter à l'AD.

`/etc/samba/smb.conf`
```python
[global]
    interfaces = eth1
    security = ADS
    idmap config * : backend = tdb
    idmap config * : range = 3000-7999
    idmap config DOMAIN.INT : backend = rid
    idmap config DOMAIN.INT : range = 10000-999999
    winbind use default domain = yes
    winbind offline logon = false
    winbind enum users = yes
    winbind enum groups = yes
    template homedir = /home/vagrant
    template shell = /bin/bash
    bind interfaces only = yes
    realm = DOMAIN.INT
    workgroup = DOMAINE
```
A savoir que le workgroup est le NetBios name que l'on a configuré lors de l'installation de l'AD.
Ne pas oublier de redémarrer le service smbd puis s'enregistrer auprès de l'AD pour récupérer des tickets kerberos:
![[Pasted image 20230321112644.png]]
Puis on regarde la liste des tickets récupérés:
```bash
	sudo klist
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: vagrant@DOMAIN.INT

Valid starting       Expires              Service principal
21/03/2023 11:15:48  21/03/2023 21:15:48  krbtgt/DOMAIN.INT@DOMAIN.INT
	renew until 22/03/2023 11:15:46
21/03/2023 11:22:55  21/03/2023 21:15:48  cifs/ADDS.DOMAIN.INT@DOMAIN.INT
21/03/2023 11:22:55  21/03/2023 21:15:48  ldap/adds.domain.int@DOMAIN.INT
```
On vérifie que la machine a bien été ajoutée dans le LDAP avec la console `mmc`.
![[Pasted image 20230321112948.png]]
On résout les problèmes de résolution DNS en modifiant le fichier `/etc/hosts` en le remplaçant par le fichier suivant:
```
127.0.0.1	localhost

10.0.20.150	client-linux-ad.DOMAIN.INT client-linux-ad
```
Une fois terminé:
![[Pasted image 20230321113807.png]]
Ça fonctionne !


Utilisation de `windbind`:
On configure le fichier `/etc/nsswitch.conf` pour y ajouter `windbind`:
on modifie les deux lignes `passwd` & `group` et on y ajoute windbind:
```
passwd:         files systemd winbind
group:          files systemd winbind
```
On vérifie que cela fonctionne bien:
![[Pasted image 20230321114416.png]]
```bash
$ sudo getent passwd
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
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:101:101:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:104:110::/nonexistent:/usr/sbin/nologin
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
vagrant:x:1000:1000:vagrant,,,:/home/vagrant:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
postfix:x:107:115::/var/spool/postfix:/usr/sbin/nologin
vboxadd:x:998:2::/var/run/vboxadd:/sbin/nologin
user:x:1001:1001:,,,:/home/user:/bin/bash
usbmux:x:106:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
rtkit:x:108:117:RealtimeKit,,,:/proc:/usr/sbin/nologin
dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
avahi:x:110:119:Avahi mDNS daemon,,,:/run/avahi-daemon:/usr/sbin/nologin
saned:x:111:120::/var/lib/saned:/usr/sbin/nologin
colord:x:112:121:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin
pulse:x:113:122:PulseAudio daemon,,,:/run/pulse:/usr/sbin/nologin
lightdm:x:114:124:Light Display Manager:/var/lib/lightdm:/bin/false
administrator:*:10500:10513::/home/vagrant:/bin/bash
guest:*:10501:10513::/home/vagrant:/bin/bash
vagrant:*:11000:10513::/home/vagrant:/bin/bash
krbtgt:*:10502:10513::/home/vagrant:/bin/bash
```
On ajoute un user dans l'AD avec l'aide de la console mmc:
![[Pasted image 20230321114919.png]]
Puis on check si l'utilisateur est bien retrouvé avec la commande `getenv passwd`:
![[Pasted image 20230321114956.png]]
On le retrouve bien.
`first_user:*:11105:10513::/home/vagrant:/bin/bash` Cependant, on remarque que l'utilisateur se retrouve avec le home du vagrant. Il va falloir changer la configuration du smb.conf pour pouvoir lui attribuer son propre home:
`template homedir = /home/%U`
![[Pasted image 20230321115230.png]]Cela fonctionne bien. On se connecte à l'utilisateur:
![[Pasted image 20230321115312.png]]
