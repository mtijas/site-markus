---
Title: SSH-avaimet GitHubista
Description: SSH-avainten haku omalta GitHub-tililtäsi
Template: withsubmenu
---

# SSH-avaimet OpenSSH:n ymmärtämällä tavalla

Pieni muistilista OpenSSH-avainten hakuun GitHub-tililtäsi etäkoneelle kirjautumista varten.

Oletan, että olet sen käyttäjän kotihakemistossa jolle haluat avaimet lisätä. Oletan myös sinun olevan kirjautuneena tänä käyttäjänä. Jollet ole: *cd*, *sudo* ja *chown* auttanevat sopivissa kohden käytettyinä.

## Tiedoston luonti ja oikeudet

Jollei kotihakemistossasi ole *.ssh*-hakemistoa ja sen alla *authorized_keys*-tiedostoa, voit luoda ne kahdella ensimmäisellä komennolla alla. Kaksi jälkimmäistä komentoa asettavat käyttöoikeudet kuntoon, jotta OpenSSH suostuu käyttämään lisäämiäsi avaimia. 

```bash
mkdir .ssh
touch .ssh/authorized_keys
chmod 700 .ssh
chmod 600 .ssh/authorized_keys
```

Ajatuksena on, että vain sinä käyttäjänä voit lukea ja muokata tätä avaintiedostoa. Oletuksena OpenSSH-demonia ajetaan Ubuntussa sinä käyttäjänä, jonka tunnuksella olet palvelimelle yhdistämässä, joten se pääsee aina lukemaan tämän tiedoston ongelmitta.

## Avainten haku GitHubista

Huom! Komento korvaa kaikki olemassa olevat avaimet. Lisää halutessasi *tee*-komentoon lippu `-a`, jotta avaimet vain lisätään tiedoston loppuun.

```bash
curl https://github.com/<käyttäjätunnus>.keys | tee .ssh/authorized_keys
```

Ja tarkistus, että avaimet oikeasti menivät paikoilleen:

```bash
cat .ssh/authorized_keys
```

Kuten voidaan havaita, *cat*-komennon tuloste täsmää *curl*-komennon tulosteeseen, joten avaimet menivät paikoilleen onnistuneesti.
