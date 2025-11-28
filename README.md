# Valheim-projekti Veikka Hieta ja Duy Phan
Miniprojekti h6 Tero Karvisen Palvelinten hallinta -kurssille 2025.

Tavoitteena konfiguroida ja luoda toimiva pelipalvelin Valheimiin.

**Yhdistäminen toimii**

<img width="889" height="271" alt="image" src="https://github.com/user-attachments/assets/456bfe6a-766e-4008-b8ce-59ebcde805d1" />


**Pelissä**

<img width="876" height="418" alt="image" src="https://github.com/user-attachments/assets/f4c6e2d4-3f20-4e46-9cdb-62c909702b46" />




## Aikataulu

Projekti aloitusaika 27. marraskuuta klo 20.00.<br>

Projekti saatiin päätökseen ...

# Ympäristö
## VirtualBox: 

- **Käyttöjärjestelmä:** Debian 13 Trixie <br>
- **System:** - Base memory: 2048 <br>
- **Display:** Video Memory: 16 MB <br>
- **Network:** Bridged Adapter <br>

## Isäntäkone

- **Malli:** Acer Aspire A315-42G
- **Prosessori:** AMD Ryzen 7 3700U with Radeon Vega Mobile Gfx (2.3 GHz)
- **Näytönohjain:** AMD Radeon(TM) RX Vega 10 Graphics, Radeon 540X series
- **RAM:** 8 GB
- **Järjestelmä tyyppi:** 64-bit käyttöjärjestelmä, x64-pohjainen prosessori
- **Internet:** Lähiverkko, 100 MB/s

## Testaus
- **Peli:** Valheim (Steam)

## Lisätiedot
- **VirtualBox:**
[Virtuaalikoneen käyttöönotto ja asennus](https://www.virtualbox.org/)

- **ISO-tiedosto:**
[Latauslinkki](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/debian-live-13.2.0-amd64-xfce.iso)

- **Tietoa Valheimista:**
[Valheim Wiki](https://valheim.fandom.com/wiki/Valheim_Wiki)


# Toteutus
## Serverin pystyttäminen

```
KOMENNOT:
sudo apt-get update
sudo apt install -y curl wget vim tar gzip lib32gcc-s1 lib32stdc++6


# new user for cleanliness and keeping server isolated
sudo useradd -m steam
sudo passwd steam

sudo su - steam

# proper bash
sudo chsh -s /bin/bash steam

sudo su - steam

# 
mkdir ~/steamcmd
cd ~/steamcmd

wget https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz
tar -xvf steamcmd_linux.tar.gz

# run once to initialize
./steamcmd.sh +quit

# installing dedicated server
./steamcmd.sh +login anonymous +app_update 896660 validate +quit
./steamcmd.sh +login anonymous +app_update 896660 validate +quit

# FIX
mkdir -p ~/.local/share/Steam/steamapps/common/
mv ~/Steam/steamapps/common/Valheim\ dedicated\ server/ ~/.local/share/Steam/steamapps/common/

#TEST
ls ~/.local/share/Steam/steamapps/common/


nano ~/start_valheim.sh

# script
#!/bin/bash
export LD_LIBRARY_PATH=./linux64:$LD_LIBRARY_PATH
export SteamAppId=892970

SERVER_DIR="$HOME/.local/share/Steam/steamapps/common/Valheim dedicated server"

cd "$SERVER_DIR"

./valheim_server.x86_64 \
  -name "MyValheimServer" \
  -port 2456 \
  -world "MyWorld" \
  -password "mypassword" \
  -public 0

chmod +x ~/start_valheim.sh

# RUN
./start_valheim.sh

# ENABLE FIREWALL (AS ROOT)
sudo ufw allow 2456:2458/udp
sudo ufw enable

```
## Salt-iplementointi

**Yhdistäminen toimii**

<img width="715" height="502" alt="image" src="https://github.com/user-attachments/assets/0d5b710a-0cbf-413f-8c1b-262523850631" />

**Serverin lisääminen pelissä**

<img width="716" height="265" alt="image" src="https://github.com/user-attachments/assets/e75c9095-ada9-4cce-80ee-46b162b0c837" />

**Serveri pelissä (IP-osoite peitetty)**

<img width="1609" height="945" alt="image" src="https://github.com/user-attachments/assets/c688b077-acf0-4f30-a634-b2b3f07e2520" />



<h2>Lähteet</h2>
Karvinen, Tero 2025: Palvelinten hallinta. Linkki: https://terokarvinen.com/palvelinten-hallinta/#alustava-aikataulu <br>
Heinonen, Johanna 2025: How To Install Linux To VirtualBox. Linkki: https://github.com/johannaheinonen/johanna-test-repo/blob/main/linux-20082025.md <br>
VirtualBox: Download VirtualBox. Linkki: https://www.virtualbox.org/wiki/Downloads <br>
ChatGPT !!!!!!!!! <br>
