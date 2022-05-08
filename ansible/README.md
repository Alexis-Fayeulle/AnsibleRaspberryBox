# Ansible Raspberry Pi Box

Projet pour créer avec un raspberry une box qui prend en entrée un iphone et en sortie 1 reseau local et 2 wifi  
Projet pour surtout sauvegarder ce code et la methode pour de faire
Je n'ai pas pour volonté de faire quelque chose qui marche partout, mais qui marche assez bien pour chez moi

## Réseau et environment

J'ai a ma disposition

- un écran
- un clavier
- une souris
- un telephone pour me fournir un hotspot wifi avec internet
- un réseau local sans dhcp ni gateway (un switch tout seul on peut dire, pas de source)
- un nas en ip fixe
- une carte wifi en usb à onde porteuse que j'ai achetée il y a longtemps (2.4Ghz)
- une clef usb wifi Archer T3U
- un iphone 5S avec un forfait 4G
- un ordinateur qui exécutera l'ansible, connecté au reseau ET a une autre source d'internet (wifi par exemple)

Sur l'ordinateur, pensez à installer ansible  
Mon ordinateur :

```
$ ansible --version
ansible [core 2.12.4]
  (...)
```

## Installation du raspberry

J'utilise une image de raspberry daté de 2019-04-08  
"Raspbian Stretch Full"  
Le fichier s'appelle "2019-04-08-raspbian-stretch-full.img"  
Copiez l'image sur la carte SD du raspberry

Branchez le raspberry a l'écran, au clavier, a la souris et au réseau  
NE PAS BRANCHER PLUS, ou sinon udev vas etre en capacité de definir les carte reseau differentes que ce que l'on a l'habitude  
Définissez une IP fixe à votre raspberry et activez le ssh

```
$ cd /etc/network/interfaces.d
$ nano eth0
allow-hotplug eth0
iface eth0 inet static
  address 192.168.2.1
  netmask 255.255.255.0
  gateway 192.168.2.1
  dns-nameservers 4.4.4.4
  dns-nameservers 8.8.8.8

$ sudo systemctl enable ssh
$ sudo nano /etc/ssh/sshd_config
(...)
PubkeyAuthentication yes
(...)
AuthorizedKeysFile      /etc/ssh/authorized_keys
(...)
$ sudo systemctl start ssh
```

À ce moment, il faut faire la première connection avec l'ordinateur  
Sur l'ordinateur definir une ip fixe puis faire ssh (le raspberry)  
Pour moi j'ai un host et une config  
Voila ce que cela donne

Pour définir l'IP fixe, je vous laisse faire, je ne connais pas votre système  
En tout cas voila les informations qu'il vous faut  
IP : 192.168.2.x  
masque : 255.255.255.0  
Routeur / Gateway : RIEN, sinon le système vas tenter de chercher internet sur ce réseau, mais on sait qu'il n'y en a pas !

```
# commandes equivalentes
$ ssh rasp
$ ssh pi@192.168.2.1
```

À ce moment une verification de clef vous demande

```
The authenticity of host 'rasp (192.168.2.1)' can't be established.
ED25519 key fingerprint is SHA256:___________________________________________.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

À la place des "_" vous avez un hash que l'on peut retrouver depuis les fichiers du raspberry  
Notez bien le "ED25519", c'est important pour savoir quelle clef est utilisé

```
$ cd /etc/ssh
$ sudo ssh-keygen -lf ssh_host_ed25519_key
256 SHA256:___________________________________________ root@raspberrypi (ED25519)
```

Vérifiez que le hash est le même et sur l'ordinateur et dites "yes" dans ce cas  
À ce moment, vous devez entrer un mot de passe pour l'utilisateur "pi"  
Mot de passe : "raspberry"  
Pensez bien à le changer et de le noter dans un endroit sécurisé

Sur l'ordinateur, copiez votre clef publique  
Et collez la dans `/etc/ssh/authorized_keys`

```
$ cat ~/.ssh/id_rsa.pub
ssh-rsa ____..  ..____ atyklaxas@atyklaxas-ordinateur
```

```
$ nano /etc/ssh/authorized_keys
ssh-rsa ____..  ..____ atyklaxas@atyklaxas-ordinateur
```

Vous avez maintenant crée un access facile a votre raspberry  
Vous pouvez maintenant tester cela  
Sur l'ordinateur :

```
$ ssh rasp
Linux raspberrypi 5.15.32-v7+ #1538 SMP Thu Mar 31 19:38:48 BST 2022 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun May  8 19:20:43 2022 from 192.168.2.89
pi@raspberrypi:~ $
```

## Premier tag - network-pre-connect

Sur l'ordinateur, clonez le depot et placez vous dans le dossier ansible  
Je ne vous explique pas comment marche ansible, d'autre tuto existe deja sur internet

```
$ cd ~
$ git clone git@github.com:AtyKlaxas/AnsibleRaspberryBox.git
$ cd AnsibleRaspberryBox/ansible
```

Ensuite lancer le playbook raspberry.yml avec le tag network-pre-connect  
Pensez bien a faire votre fichier inventory et .vault

```
$ ansible-playbook --vault-password-file pass.txt -i inventory raspberry.yml --tags network-pre-connect
```

Maintenant votre raspberry est connecté à internet depuis votre telephone

## Second tag - install

```
$ ansible-playbook --vault-password-file pass.txt -i inventory raspberry.yml --tags install
```

Maintenant vous avez tous les paquets qu'il nous faudra pour créer la box  
De plus le pilote de la carte wifi est compilé et installé

## Troisième tag - config

```
$ ansible-playbook --vault-password-file pass.txt -i inventory raspberry.yml --tags config
```

Maintenant tout les element à configurer sont configuré  
Il ne manque que de les démarrer

## Quatrième tag - startup

```
$ ansible-playbook --vault-password-file pass.txt -i inventory raspberry.yml --tags startup
```

Et voilà, raspberry box opérationnel !  
À noter que chez moi j'ai encore des bugs  
ça veut dire que cela peut encore changer d'ici la parce que mon antenne wifi marche de façon étrange

