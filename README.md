# Valheim peliserveri - projekti, Veikka Hieta ja Duy Phan
Miniprojekti h6, [Tero Karvisen Palvelinten hallinta -kurssille 2025](https://terokarvinen.com/palvelinten-hallinta/).

Projektin tavoitteena on konfiguroida ja luoda toimiva Valheim-pelipalvelin sekä toteuttaa sen hallinta idempotenssiksi Saltin avulla.

<img width="889" height="271" alt="image" src="https://github.com/user-attachments/assets/456bfe6a-766e-4008-b8ce-59ebcde805d1" />

**Yhdistäminen toimii (Debianos = Hahmon nimi)**

<img width="876" height="418" alt="image" src="https://github.com/user-attachments/assets/f4c6e2d4-3f20-4e46-9cdb-62c909702b46" />

**Pelissä (Pelin nimi täsmää yhdistävän ID:n kanssa)**



## Aikataulu

Projekti aloitusaika 27. marraskuuta klo 20.00.<br>

Projekti saatiin päätökseen 28.11. klo 17.15.

# Ympäristö
## VirtualBox

- **Käyttöjärjestelmä:** Debian 13 Trixie <br>
- **System:** - Base memory: 2048 <br>
- **Display:** Video Memory: 16 MB <br>
- **Network:** Bridged Adapter <br>


## Isäntäkone

- **Malli:** Acer Aspire A315-42G (kannettava tietokone)
- **Prosessori:** AMD Ryzen 7 3700U with Radeon Vega Mobile Gfx (2.3 GHz)
- **Näytönohjain:** AMD Radeon(TM) RX Vega 10 Graphics, Radeon 540X series
- **RAM:** 8 GB
- **Järjestelmä tyyppi:** 64-bit käyttöjärjestelmä, x64-pohjainen prosessori
- **Internet:** Lähiverkko, 100 MB/s

  
## Testaus

- **Rauta:** Toinen tietokone (pöytä), Windows 11 - käyttöjärjestelmällä
- **Peli:** Valheim (Steam)


## Lisätiedot

- **VirtualBox:**
[Virtuaalikoneen käyttöönotto ja asennus](https://www.virtualbox.org/)

- **ISO-tiedosto:**
[Latauslinkki](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/debian-live-13.2.0-amd64-xfce.iso)

- **Tietoa Valheimista:**
[Valheim Wiki](https://valheim.fandom.com/wiki/Valheim_Wiki)


# Toteutus
## Asennukset ja serverin pystyttäminen

**Pakettien asennus:**
```
sudo apt-get update
sudo apt install -y curl wget vim tar gzip lib32gcc-s1 lib32stdc++6
```

**Luodaan uusi käyttäjä siisteyden ylläpitämiseen ja palvelimen eristämiseen:**
```
sudo useradd -m steam
sudo passwd steam

sudo su - steam
```

**Vaihdetaan oletuskomentorivi bashiin:**
```
sudo chsh -s /bin/bash steam

sudo su - steam
```

**Uusi hakemisto ja SteamCMD:n asennus:**
```
mkdir ~/steamcmd

wget https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz
tar -xvf steamcmd_linux.tar.gz
```

**Suoritetaan komento kerran alustamista varten:**
```
./steamcmd.sh +quit
```

**Asennetaan dedikoitu serveri:**
```
mkdir -p /home/steam/valheim_server
cd steamcmd
./steamcmd.sh +force_install_dir /home/steam/valheim_server +login anonymous +app_update 896660 validate +quit
```

**Testi, jolla nähdään että serveri löytyy**

`ls -l /home/steam/valheim_server`

**Luodaan serverin käynnistys skripti:**
```
nano ~/start_valheim.sh

#!/bin/bash

# Library path
export LD_LIBRARY_PATH="/home/steam/valheim_server:$LD_LIBRARY_PATH"

# Steam App ID
export SteamAppId=896660

# Change to server folder
cd /home/steam/valheim_server || exit 1

# Start the server
./valheim_server.x86_64 \
  -name "MyValheimServer" \
  -port 2456 \
  -world "MyWorld" \
  -password "mypassword" \
  -public 0
```

**Muutetaan tiedosto suoritettavaksi:**
```
chmod +x ~/start_valheim.sh
```

**Testataan serverin manuaalista käynnistä:**
```
./start_valheim.sh
```

**Automatisoidaan serveri (Lähtee päälle kun virtuaalikone käynnistyy):**
```
sudo nano /etc/systemd/system/valheim.service

[Unit]
Description=Valheim Dedicated Server
After=network.target

[Service]
User=steam
Group=steam
Type=simple
WorkingDirectory=/home/steam/valheim_server
ExecStart=/home/steam/start_valheim.sh
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

**Käynnistetään daemonit uudelleen:**
```
sudo systemctl daemon-reload
sudo systemctl enable --now valheim.service
sudo systemctl restart valheim.service
sudo systemctl status valheim.service
```

<img width="885" height="452" alt="image" src="https://github.com/user-attachments/assets/acec92a3-3bbb-491c-a578-78f676b4c176" />

Tältä statuksen ja lokitietojen tulisi näyttää kun serverin käynnistys on automatisoitu ja lähtee päälle. 

## Palomuurin salliminen porteille (Vain nämä portit turvallisuussyistä!)
```
sudo ufw allow 2456:2458/udp
sudo ufw enable
```

## Salt-implementaatio

Saltin rakentamista varten olemme aikaisemmin asentaneet Saltin virtuaalikoneelle. Asennusohjeet löytyvät [Teron sivuilta](https://terokarvinen.com/install-salt-on-debian-13-trixie/).

**Luodaan ensin uusi hakemisto salttia varten:**
```
sudo mkdir -p ~/srv/salt/valheim
cd /srv/salt/valheim
```

**Luodaan kaikki state-tiedostot `sudo nano "nimi"` komennolla skripteineen:**

**dependencies.sls**
```
install_dependencies:
  pkg.installed:
    - pkgs:
      - curl
      - wget
      - vim
      - tar
      - gzip
      - lib32gcc-s1
      - lib32stdc++6
```

**steamcmd.sls**
```
create_steam_user:
  user.present:
    - name: steam
    - home: /home/steam
    - shell: /bin/bash
 
install_steamcmd:
  file.directory:
    - name: /home/steam/steamcmd
    - user: steam
    - group: steam
    - mode: 755
 
download_steamcmd:
  cmd.run:
    - name: wget https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz -O /home/steam/steamcmd/steamcmd_linux.tar.gz
    - creates: /home/steam/steamcmd/steamcmd_linux.tar.gz
    - user: steam
    - cwd: /home/steam/steamcmd
 
extract_steamcmd:
  cmd.run:
    - name: tar -xvf steamcmd_linux.tar.gz
    - cwd: /home/steam/steamcmd
    - creates: /home/steam/steamcmd/steamcmd.sh
    - user: steam
```

**valheim.sls**
```
install-valheim:
  cmd.run:
    - name: "/home/steam/steamcmd/steamcmd.sh +login anonymous +force_install_dir /home/steam/valheim_server +app_update 896660 validate +quit"
    - cwd: /home/steam/steamcmd
    - user: steam
    - unless: test -f "/home/steam/valheim_server/valheim_server.x86_64"
```

**startscript.sls**
```
valheim_startscript:
  file.managed:
    - name: /home/steam/start_valheim.sh
    - source: salt://valheim/start_valheim.sh
    - user: steam
    - group: steam
    - mode: 755
```

**service.sls**
```
valheim_service_file:
  file.managed:
    - name: /etc/systemd/system/valheim.service
    - source: salt://valheim/valheim.service
    - user: root
    - group: root
    - mode: 644
valheim_service:
  service.running:
    - name: valheim
    - enable: True
    - watch:
      - file: valheim_service_file
```

**ufw.sls**
```
allow_valheim_ports:
  cmd.run:
    - name: "ufw allow 2456:2458/udp"
    - unless: "ufw status | grep 2456"
```

**init.sls**
```
include:
  - valheim.dependencies
  - valheim.steamcmd
  - valheim.valheim
  - valheim.startscript
  - valheim.service
  - valheim.ufw
```

**Kopioidaan käynnistysskripti ja palvelu-tiedosto saltille**

```
sudo cp /home/steam/start_valheim.sh /srv/salt/valheim/start_valheim.sh
sudo cp /etc/systemd/system/valheim.service /srv/salt/valheim/valheim.service
```

**Luodaan vielä top.sls jolla voidaan käynnistää kaikki statet yhdellä komennolla:**
```
cd ..
sudo nano top.sls

base:
  '*':
    - valheim
```

**Testataan vielä lopuksi statet:**
```
sudo salt-call --local state.apply
```

<img width="393" height="352" alt="image" src="https://github.com/user-attachments/assets/31bafd56-37bd-4dd0-a737-c46affee5b02" />

Kaikki testit näyttäisivät toimivan.

# Lopputulos

Lopputuloksena olemme saaneet asennettua ja konfiguroitua Valheim-serveri, joka käynnistyy automaattisesti virtuaalikoneen käynnistyessä. Jäljellä on enää testaaminen pelissä.

<img width="716" height="265" alt="image" src="https://github.com/user-attachments/assets/e75c9095-ada9-4cce-80ee-46b162b0c837" />

**Serverin lisääminen pelissä (IP-osoite ja portti virtuaalikoneesta)**

<img width="1609" height="945" alt="image" src="https://github.com/user-attachments/assets/c688b077-acf0-4f30-a634-b2b3f07e2520" />

**Serveri pelissä (IP-osoite peitetty)**

Yhdistetään serveriin, jonka jälkeen se pyytää salasanaa. Salasana on sama kuin minkä olemme määrittäneet "start_valheim.sh" skriptissä. 

<img width="715" height="502" alt="image" src="https://github.com/user-attachments/assets/0d5b710a-0cbf-413f-8c1b-262523850631" />

**Yhdistäminen toimii**

Salasanan syötettyä pelissä, virtuaalikoneeseen tulee tiedot siitä että yhteys on muodostettu ja henkilö pystyy tämän jälkeen pelaamaan serverillä. 😄

# Lähteet

Heinonen, J. 27.8.2025. How To Install Linux To VirtualBox?. GitHub. Luettavissa: https://github.com/johannaheinonen/johanna-test-repo/blob/main/linux-20082025.md. Luettu: 27.11.2025.

Karvinen, T. 20.10.2025. Install Salt on Debian 13 Trixie. Tero Karvinen. Luettavissa: https://terokarvinen.com/install-salt-on-debian-13-trixie/. Luettu: 28.11.2025

Karvinen, T. 26.3.2025. Palvelinten hallinta. Tero Karvinen. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/#alustava-aikataulu. Luettu: 27.11.2025.

VirtualBox. Download VirtualBox. Luettavissa: https://www.virtualbox.org/wiki/Downloads. Luettu: 27.11.2025.

Hyödynnetty ChatGPT 5.1-kielimallia. Syötteenä käytettiin: "I am trying to make a Valheim project for my Linux Debian 13 with Salt, can you help me get started?". 28.11.2025.
