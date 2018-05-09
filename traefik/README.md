# Traefik

Traefik is de reverse proxy die we gebruiken. Traefik maakt automatisch LetsEncrypt certificaten aan voor de domeinen die toegevoegd worden. Er hoeft dus geen apart werk danwel kosten gemaakt te worden voor HTTPS.

## Configuratie

De globale configuratie voor Traefik is te vinden in `traefik.toml`.

We maken gebruik van `[file]` backend voor Qlik hosts.

De 'regels' voor (Qlik) hosts zijn te vinden in `rules/server.hippoline.nl.toml`.

Om een nieuwe verwijzing toe te voegen kun je een bestaande regel kopieren en aanpassen met de gegevens voor de nieuwe server.

## Notes

In de globale config staat `insecureSkipVerify` op `true`. Dit maakt het mogelijk om door te verwijzen naar Qlik (Sense) servers die intern een self-signed certificaat gebruiken. `insecureSkipVerify` houdt echter in dat er geen interne checks gedaan worden op het certificaat. Dat moet, omdat de self-signed certs niet officieel geldig zijn. Het betekent echter wel dat de veiligheid lager is dan met een officieel gesigned certificaat.

