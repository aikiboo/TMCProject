# TMCProject
Un projet d' IoT. Deux raspberry communiquent entre eux par LoRa, communication chiffrée par AES.
Les fichiers des rapberry sont hébergés et utilisés depuis un ordinateur hote par connection ethernet.


<details><summary>Installation de l'host</summary>
<p>

Afin de commencer l'installation de l'host nous allons créer notre répertoire ainsi que télécharger, et monter l'OS de notre raspberry. 

```
	$ mkdir RASPI
	$ cd RASPI
	$ mkdir client
	$ mkdir boot
	$ wget https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2022-09-26/2022-09-22-raspios-bullseye-arm64-lite.img.xz
	$ tar -xJf 2022-09-22-raspios-bullseye-arm64-lite.img.xz
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
```

Nos systèmes de fichiers sont désormais presque prêt pour démarrer notre raspberry, il reste cependant la configuration à faire. Pour ce faire nous allons devoir activer le NFS.
```
	$ sudo apt install nfs-kernel-server
```

On modifie également le fichier /etc/exports en ajoutant à la fin les lignes:
```
/home/florian/RASPI/client *(rw,sync,no_subtree_check,no_root_squash)
/home/florian/RASPI/boot *(rw,sync,no_subtree_check,no_root_squash
``` 

Puis on active et relance nos services.

```
sudo systemctl enable nfs-kernel-server
sudo systemctl enable rpcbind
sudo systemctl restart nfs-kernel-server
```

On récupère ensuite le nom de notre interface réseau Ethernet.


</p>
</details>




