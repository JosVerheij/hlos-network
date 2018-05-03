# OpenVPN Docker container

HippoLine gebruikt OpenVPN voor het verbinden met het netwerk.

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

Een gebruikerscertificaat en -configuratie wordt zoals onderstaand aangemaakt. Je kunt ervoor kiezen om een wachtwoord op te geven (veilig) of niet. Dit wachtwoord heb je 

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

## Verbinden met VPN

Wanneer de OpenVPN client geinstalleerd is en het `.ovpn` bestand is toegevoegd kun je verbinding maken.

In de Windows taakbalk staat rechtsonderin het OpenVPN icoon.

- Rechtsklik op het OpenVPN icoon -> Connect --> Kies de juiste verbinding

