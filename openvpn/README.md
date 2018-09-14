# OpenVPN Docker container

HippoLine gebruikt OpenVPN voor het verbinden met het netwerk.

Acties in dit document voer je in principe uit op het betreffende Linux (en Docker systeem). Zorg dat je *altijd* in de `hlos-network` root-folder zit bij het uitvoeren van onderstaande acties. (ie. de map waar docker-compose.yml in te vinden is)

## Setup domein

Voor het verbinden met de OpenVPN server moet een *geldige domeinnaam* worden gebruikt. Dit is bijvoorbeeld `openvpn.hippoline.nl`. Zorg dat er een DNS-verwijzing (domeinnaamverwijzing) is van de gewenste domeinnaam naar het IP adres van deze server.

## (Eerste) Setup Server

Voor de eerste setup moeten we e.e.a. instellen voordat de VPN server klaar voor gebruik is.

Zorg dat je op de server bent ingelogd met een gebruiker die `docker` mag uitvoeren.

- Ga op de (docker) server naar de `hlos-network` map; hierin staat het `docker-compose.yml` bestand

```bash
cd [MAPNAAM]

ls # hier moet (o.a.) 'docker-compose.yml' tussen staan
```

- Initialiseer de configuratie bestanden en certificaten

Tijdens het genereren van de configuratiebestanden wordt je gevraagd om een wachtwoord. Dit is de private key en die **moet altijd geheim blijven**. 

*Deze key heb je ook nodig bij het aanmaken van gebruikersconfiguraties*; **zorg dus dat je de private key opslaat!**

```bash
docker-compose run --rm openvpn ovpn_genconfig -u udp://VPN.SERVERNAME.COM
docker-compose run --rm openvpn ovpn_initpki
```

- Zet indien nodig de lees en schrijfrechten van het zgn. data volume goed

```bash
sudo chown -R $(whoami): ./openvpn/data
```

- Je kunt nu de VPN server starten

```bash
docker-compose up -d openvpn
```

## Aanmaken VPN gebruiker

In de VPN configuratie krijgt elke gebruiker een eigen certificaat, waarmee verbinding kan worden gemaakt met de VPN server. Zo kan per gebruiker bepaald worden of deze toegang krijgt of niet.

> NB: Er zit (standaard) geen beveiliging op het gebruikerscertificaat. Iedereen met toegang tot zo'n certificaat kan dus de VPN op komen. Bij het vermoeden dat een certificaat van gebruiker op straat ligt, moet dit certificaat ZSM worden ingetrokken.

### Gebruikersconfiguratie aanmaken

Een gebruikerscertificaat en -configuratie wordt zoals onderstaand aangemaakt. 

Tijdens het aanmaken wordt gevraagd om de *private key* (deze is aangemaakt tijdens de setup van de server); zorg dat je deze bij de hand hebt.

```bash
export CLIENTNAME="hippoline_[CLIENT_NAAM]" # eg. "hippoline_jos"

# MET een wachtwoord (recommended)
docker-compose run --rm openvpn easyrsa build-client-full $CLIENTNAME
# ZONDER een passphrase (not recommended)
docker-compose run --rm openvpn easyrsa build-client-full $CLIENTNAME nopass
```

- Haal de configuratie op

```bash
docker-compose run --rm openvpn ovpn_getclient $CLIENTNAME > $CLIENTNAME.ovpn
```

- Het bestand is nu te vinden in de huidige map

```bash
ls

# output: (eg.) hippoline_jos.ovpn
```

### Ophalen configuratie

- Download het configuratiebestand naar je lokale systeem
    1. Open een nieuw terminal venster
    2. Voer het volgende in

```bash
scp [USER]@[SERVER-IP]:[MAPNAAM]/[CLIENTCONFIG].ovpn [LOKALE-MAP]

# eg. scp jos@10.80.10.10:~/projects/docker-compose/hippoline_jos.ovpn ~/Desktop/hippoline_jos.ovpn
```

Het configuratiebestand staat nu op je lokale systeem en kan je vanaf hier ofwel opslaan op een veilige plek voor eigen gebruik; of doorsturen naar de betreffende gebruiker.

## Installatie VPN Client

Indien de OpenVPN client nog niet is geinstalleerd moet dit eerst gebeuren.

- Ga naar de [OpenVPN downloadpagina](https://openvpn.net/index.php/open-source/downloads.html)
- Download en installeer de Windows installer

### Toevoegen 'OVPN' gebruikersconfig

Wanneer de OpenVPN client geinstalleerd staat kun je je eigen VPN gebruikersconfig toevoegen.

- Dubbelklik op het configuratiebestand: ie. `hippoline_jos.ovpn`

#### Windows (her)installatie troubleshooting

> Dubbelklikken voegt de verbinding niet toe

In de Windows taskbar, rechtsklik rechtsonderin op het OpenVPN icoon en voeg op die manier de configuratie toe.

> Er bestaat al een verbinding met dezelfde naam

Verwijder het bestand in `C:\Users\[USERNAME]\OpenVPN\Config`, of overschrijf deze handmatig met het nieuwe bestand.

## Verbinden met VPN

Wanneer de OpenVPN client geinstalleerd is en het `.ovpn` bestand is toegevoegd kun je verbinding maken.

In de Windows taakbalk staat rechtsonderin het OpenVPN icoon.

- Rechtsklik op het OpenVPN icoon -> Connect --> Kies de juiste verbinding

## Toegang gebruiker ontzeggen

- Achterhaal de 'CLIENTNAME' van de gebruiker. Eg. `hippoline_jos`.

- Verwijder de gebruikersconfiguratie

```bash
export CLIENTNAME="hippoline_[CLIENT_NAAM]" # eg. "hippoline_jos"

# KEEP the corresponding crt, key and req files.
docker-compose run --rm openvpn ovpn_revokeclient $CLIENTNAME

# OR

# REMOVE the corresponding crt, key and req files.
docker-compose run --rm openvpn ovpn_revokeclient $CLIENTNAME remove
```

## Naslagwerk

- <https://github.com/kylemanna/docker-openvpn/blob/master/docs/docker-compose.md>