# TMCProject
Un projet d' IoT. Deux raspberry communiquent entre eux par LoRa, communication chiffrée par AES.
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

</p>
</details>



