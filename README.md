# TMCProject
Un projet d' IoT. Deux raspberrys communiquent entre eux par LoRa, communication chiffrée par AES.
Les fichiers des rapberry sont hébergés et utilisés depuis un ordinateur hote par connection ethernet.


<details><summary>Installation de l'host</summary>
<p>

Afin de commencer l'installation de l'host nous allons créer notre répertoire ainsi que télécharger, et monter l'OS de notre raspberry. Ici notre path sera /home/florian/TMCProject/RASPI, il faut donc l'adapter à notre situation.  

```
	$ mkdir RASPI
	$ cd RASPI
	$ mkdir client
	$ mkdir boot
	$ wget https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2022-09-26/2022-09-22-raspios-bullseye-arm64-lite.img.xz
	$ unxz 2022-09-22-raspios-bullseye-arm64-lite.img.xz
	$ sudo losetup -fP 2022-09-22-raspios-bullseye-arm64-lite.img
	$ losetup -a | grep rasp
```

Suite à cette commande on récupère le numéro du loop correspondant à notre image. Pour les commandes suivantes ce numéro sera remplacé par un X.

```
	$ sudo mount /dev/loopXp2 /mnt
	$ sudo rsync -xa --progress /mnt/ client/
	$ sudo umount /mnt
	$ sudo mount /dev/loopXp1 /mnt
	$ cp -r /mnt/* boot/
	$ sudo umount /mnt
```

Nos systèmes de fichiers sont désormais presque prêt pour démarrer notre raspberry, il reste cependant la configuration à faire. Pour ce faire nous allons devoir activer le NFS.
```
	$ sudo apt install nfs-kernel-server
```

On modifie également le fichier /etc/exports en ajoutant à la fin les lignes:
```
/home/florian/TMCProject/RASPI/client *(rw,sync,no_subtree_check,no_root_squash)
/home/florian/TMCProject/RASPI/boot *(rw,sync,no_subtree_check,no_root_squash
``` 

Puis on active et relance nos services.

```
	$ sudo systemctl enable nfs-kernel-server
	$ sudo systemctl enable rpcbind
	$ sudo systemctl restart nfs-kernel-server
```

On récupère ensuite le nom de notre interface réseau Ethernet grâce à la commande:
```
	$ ip a
```

On remplacera donc le nom de l'interface par celle-ci dans le script "script_boot_rpi".  On pensera aussi à y modifier le path dans la dernière ligne du script.

On modifie aussi le fichier cmdline.txt :
```
console=serial0,115200 console=tty1 root=/dev/nfs nfsroot=10.20.30.1:/home/florian/TMCProject/RASPI/client,tcp,vers=3 rw ip=dhcp rootwait
```

Puis le fichier /client/etc/fstab:
```
proc /proc proc defaults 0 0
10.20.30.1:/home/florian/TMCProject/RASPI/boot /boot nfs defaults,vers=3 0 0
```

Puis on édit le fichier /client/lib/systemd/system/sshswitch.service afin qu'il ressemble à ça :
```
[Unit]
Description=Turn on SSH if /boot/ssh is present
After=regenerate_ssh_host_keys.service
[Service]
Type=oneshot
ExecStart=/bin/sh -c "systemctl enable --now ssh"
[Install]
WantedBy=multi-user.target
```

Il ne reste donc plu-qu’à démarrer le script et nous connecter à notre raspberry.
</p>
</details>

<details><summary>Mise en place des communications LoRa</summary>
<p>
Afin de faire communiquer nos deux raspberrys nous allons utiliser le LoRa et plus précisément la bibliothèque RadioHead qui nécessite elle même la bibliothèque bcm2835. Mais avant de les installer nous devons activer le bus PCI utilisé par le composant LoRa.
Pour cela nous avons juste à éditer le fichier /RASPI/boot/config.txt
en modifiant les lignes de la façon suivante : 
```
	# Uncomment some or all of these to enable the optional hardware interfaces
	#dtparam=i2c_arm=on
	#dtparam=i2s=on
	dtparam=spi=on
	dtoverlay=gpio-no-irq
``` 
</p>

Nous installons ensuite nos bibliothèques:
```
	$ wget http://www.airspayce.com/mikem/bcm2835/bcm2835-1.71.tar.gz
	$ tar zxvf bcm2835-1.71.tar.gz
	$ cd bcm2835-1.71
	$ ./configure
	$ make
	$ sudo make check
	$ sudo make instal
	$ cd ..
	$ git clone https://github.com/hallard/RadioHead
```

Puis nous allons copier nos l'un des deux programmes rf95_server.cpp et rf_95_client.cpp dans le répertoire RadioHead/exemples/raspi/rf95. Le programme lié au serveur pour le Raspberry lié au serveur et inversement.

</details>

<details><summary>Mise en place du serveur mosquitto</summary>

Sur le Raspberry lié au client LoRa précédemment configuré nous allons devoir installer mosquitto:
```
	$ sudo apt-get install mosquitto
	$ sudo apt-get install mosquitto-clients
``` 
Puis nous éditerons le fichier /etc/mosquitto/mosquito.conf pour le transformer en : 
```
# Place your local configuration in /etc/mosquitto/conf.d/
#
# A full description of the configuration file is at
# /usr/share/doc/mosquitto/examples/mosquitto.conf.example

pid_file /run/mosquitto/mosquitto.pid

persistence true
persistence_location /var/lib/mosquitto/

#log_dest file /var/log/mosquitto/mosquitto.log

include_dir /etc/mosquitto/conf.d

allow_anonymous false
password_file /etc/mosquitto/mosquitto_passwd
```
Nous allons par la suite créer un utilisateur ici "esp" avec pour mot de passe "tmctmctmc":
```
 	$ sudo mosquitto_passwd -c /etc/mosquitto/mosquitto_passwd esp
```

Puis nous copions les fichiers tcp.conf et tls.conf fournit dans le git dans le répertoire /etc/mosquitto/conf.d/

Une fois cela fait nous démarrons le service mosquitto :
```
	$ sudo systemctl enable mosquitto.service
	$ sudo systemctl start mosquitto.service
```

Le serveur mis en place il nous reste donc à configurer nos certificats pour cela nous allons juste à exécuter le script genCertif, préalablement placé dans le répertoire /client/home/pi/CA_ECC/
</details>


