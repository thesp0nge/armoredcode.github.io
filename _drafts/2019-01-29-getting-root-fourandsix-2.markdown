---
layout: post
title: "Getting root: FourAndSix 2"
image: 
bootstrap: true
tags: []
categories: [ ]
author: thesp0nge
---

https://www.vulnhub.com/entry/fourandsix-201,266/
asciicast 224148
Target: 192.168.56.105
La prima scansione nn porta a nulla... solo 22,111 e 2049 aperte. Provo una scansione su tutte le porte ed in più trovo una 613.
Enumerando i mountpoint nfs, trovo /home/user/storage... provo a montarlo
Lo share ha dentro un archivio 7z di backup che è protetto da password... provo a craccare la password...

uso john per prendere l'hash e successivamente craccarlo
potrebbe mancare: apt install libcompress-raw-lzma-perl 


trovo la password: chocolate (john hash --wordlist=/usr/share/wordlists/rockyou.txt)
estraggo l'archivio... immagini di hello kitty e 2 chiavi ssh

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDClNemaX//nOugJPAWyQ1aDMgfAS8zrJh++hNeMGCo+TIm9UxVUNwc6vhZ8apKZHOX0Ht+MlHLYdkbwSinmCRmOkm2JbMYA5GNBG3fTNWOAbhd7dl2GPG7NUD+zhaDFyRk5gTqmuFumECDAgCxzeE8r9jBwfX73cETemexWKnGqLey0T56VypNrjvueFPmmrWCJyPcXtoLNQDbbdaWwJPhF0gKGrrWTEZo0NnU1lMAnKkiooDxLFhxOIOxRIXWtDtc61cpnnJHtKeO+9wL2q7JeUQB00KLs9/iRwV6b+kslvHaaQ4TR8IaufuJqmICuE4+v7HdsQHslmIbPKX6HANn user@fourandsix2

Usando la chiave mi viene chiesta una passphrase... provo ancora con john...

/usr/share/john/ssh2john.py id_rsa > rsa
con john impiego troppo, provo un altro approccio.
Uso un dizionario, passandolo ad ssh_keygen... trovo la passphrase 12345678

sono loggato come user... sono nel grppo wheel, interessante
nel file /etc/doas.conf, simile a sudo, l'utente user può eseguire less con i privilegi di root senza password...
eseguo less, con 'v' entro nell'editor e da lì eseguo bin/sh... root
#!/bin/bash

while IFS='' read -r line || [[ -n "$line" ]]; do
	if ssh-keygen -c -C "user@fourandsix2" -P $line -f id_rsa; then
		echo $line
		break
	fi
done < "$1"

