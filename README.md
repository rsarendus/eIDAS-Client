# eIDAS-Client





# TAUST - lähteülesanne




1. Java ***rakendus***, mis

2. ühendub RIA eIDAS konnektorteenuse külge

3. pakub kasutajale UI-d, kus kasutaja valib riigi, kust ta tuleb ja vajutab "Autendi mind"

4. ja rakendus suunab konnektorteenusesse ja tagasi tulles teatab kasutajale "Welcome to Estonia, Javier Garcia!",

    1. juhul kui autentimise teeb CEF Validation Service

    2. või kasutaja reaalse nime, kui autentimise teeb mõne reaalse riigi eIDAS autentimisteenus (sõltub RIA konnektorteenuse seadistusest)

6. seejuures logides nii SAML-sõnumi saatmise kui ka vastuvõtmise

7. ja andes kasutajale mõistliku teate, kui midagi peaks untsu minema

8. ja mis on piisavalt lihtne ja dokumenteeritud, et seda saab kergesti lõimida Eesti e-teenustesse, sh TARA-sse

    1. järgib [eIDAS konnektorteenuse liidese spetsifikatsiooni](https://github.com/e-gov/eIDAS-Connector/blob/master/Spetsifikatsioon.md)

    2. kasutab Java komponente. mille ühilduvust lõimitava e-teenuse Java-tarkvaraga võiks eeldada

9. ja Nortalil on võimekus vajadusel nõustada liidestujaid.

10. omab ka teatud omadusi, millega saab automaatselt eIDAS-autentimist testida.



eIDAS-Client ühe visioonina võiks koosneda kahest osast: 1) Java ***teek*** (package), nimega nt `ee.ria.eidas`. Teegis oleksid klassid jm artefaktid, mida eeldatavalt liidestuja saab ja soovib otseselt kasutada; 2) ümbrise v kesta pakett, mis teostaks seda, mida liidestuja tõenäoliselt otse üle ei võta (nt lipukeste kuvamine).



eIDAS-Client-i ***tootena*** võiks välja pakkuda eIDAS adapteri nime all.



~~Kui spetsifikatsioon, UI jm dokumentatsioon oleks inglise keeles, võiks teek potentsiaalselt huvi pakkuda teiste EL riikide konnektorite külge liidestujatele (võimalus promoda teeki CEF-is, ehk isegi saada tellimusi CEF-st või teistest riikidest).~~



EIDAS KLIENT
-----------

Eidas klient on Euroopa Liidu poolt pakutava ülepiirilise isikutuvastusteenuse näidis klient. Teenus on Java veebirakendus, mis pakub isiku autententimisprotsessi algatamiseks ja lõpptulemuse kontrollimiseks lihtsat veebiliidest.

EIDAS klient järgib [RIA eIDAS konnektorteenuse liidese spetsifikatsiooni](https://github.com/e-gov/eIDAS-Connector/blob/master/Spetsifikatsioon.md).

KUIDAS TOIMIB?
----------

Näidis klient publitseerib oma metadata otspunkti, töötleb isikutuvastusprotsessi päringuid ja vastuseid vastavalt [eIDAS konnektorteenuse liidese spetsifikatsioonis](https://github.com/e-gov/eIDAS-Connector/blob/master/Spetsifikatsioon.md) viidatud nõuetele.

Eidas kliendi kasutajaliides kuvab isikutuvastuse tulemuse nii inimloetaval (html) kui ka masinloetaval kujul (json).

### SAML metaandmete vahetus

Eidas konnektorteenuse SAML metaandmete laadimine ja kliendi enda metaandmete avaldamine toimub pärast eIDAS kliendi teenuse käivitamist (kuid enne kasutaja päringute teenindamist).

SAML metadata laadimine:
1. Eidas klient genereerib oma metaandmed teenuse käivitamise korral ja publitseerib tulemuse XML failina kohalikku failisüsteemi. Genereeritud xml faili alusel kuvatakse sisu metadata otspunkti poole pöördujatele.

2. eIDAS konnektorteenuse metaandmed loetakse sisse rakenduse käivitamisel (eenevalt seadistatud URLilt) ja puhverdatakse (vt paigaldusjuhendit, et kontrollida puhvri aegumist jm parameetreid).


### Isikutuvastusprotsess

Lihtsustatult toimib isikutuvastuse protsess eIDAS kliendi ja eIDAS konnektorteenuse vahel järgmiselt (vt Joonis 1.)

1. Kasutaja navigeerib avalehele `/start`, täpsustamata parameetreid. EIDAS klient kuvab sihtriikide valiku.

2. Kasutaja valib sihtriigi ja soovi korral autentimistaseme ning kas ta soovib tulemusi masinloetaval või inimloetaval kujul. Kasutaja vajutab 'Suuna'. Veebileht teeb taas päringu avalehele `/start`, kuid koos valitud parameetritega. Eidas kliendi serveri poolel pannakse kokku `SAMLRequest` parameetri sisu ja tagastatakse kasutajale ümbersuunamisvorm, mis suunatakse automaatselt riiklikku eIDAS konnektorteenusesse.

3. Veebibrauser suunab kasutaja automaatselt eIDAS konnektorteenusse koos `SAMLRequest`, `RelayState` ja `country` parameetritega, kus teostatakse järgnevate sammudena ära kogu ülepiirilise isikutuvastuse sammud. Sealhulgas suunatakse kasutaja ümber sihtriigi eIDAS Node teenusesse, vajadusel küsitakse kasutaja nõusolekut andmete avaldamiseks ning teostatakse isikutuvastus.

4. Peale ülepiirilise isikutuvastusprotsessi läbimist suunab eIDAS konnektorteenus, tulemuse eIDAS kliendi `/returnUrl` aadressile koos `SAMLResponse` ja `RelayState` parameetriga. EIDAS klient valideerib vastuse, dekrüpteerib sisu ning kuvab tuvastatud isiku andmed. NB! Vorming on juhitav RelayState parameetri abil (inimloetav kuju, vs masinloetav)

// Vaja teha! Kuidas toimib toetatud sihtriikide valiku automaatne uuendamine? //

<img src='doc/img/EidasClient-Isikutuvastus.png'>
Joonis 1.


TEENUSE ÜLESEHITUS
-----------

Eidas klient paketeeritakse kas kõiki sõltuvusi sisaldava ja veebiserverit sisaldava veebirakendusena (`fat jar`) või standardse veebiarhiivina (`war`).

Eidas kliendi teenuskiht toetub Spring Security ja CEF-i näidistarkvaras v1.14 kasutatavatele alusteekidele.

<img src='doc/img/EidasClient.png'></img>

Joonis 2.

### Pakutavad liidesed

Ülepiirilise isikutuvastuse päringu algatamiseks ja tagasituleva vastuse vajalike otspunktide loetelu on toodud Tabelis 1. Otspunktide täpsem kirjeldus toodud LISA 1.

| Otspunkt        | Toetatud meetodid | Selgitus  |
| ------------- | :------: | :-------------|
| `/start`  | GET |	Algatab isikutuvastusprotsessi valitud riigi eIDAS sõlmpunkti vastu. Kui riigikoodi ei täpsustata parameetriga `Country`, kuvatakse inimloetav html vorm riigivalikuga. Lisaparameetrite loetelu vt LISA 1. |
| `/returnUrl`  | POST |	Isikutuvastuse tulemuse vastuvõtt. Isikuandmete või vea kuvamine vastavalt parameetritele (vt LISA 1). |
| `/metadata`  | GET |	SAML 2.0 standardijärgne metadata otspunkt. Vajalik eIDAS konnektorteenuse ja kliendi vahelise usalduse loomiseks. |
Tabel 1.

### Nõutud liidesed

Toimimiseks vajab eIDAS klient teenus eIDAS konnektorteenust ning genereeritud võtmeid koos seadistusega.

| Komponent        | Selgitus |
| ------------- | :----- |
| `eIDAS konnektorteenus` | eIDAS klient vajab ligipääsu eIDAS konnektorteenuse SAML 2.0 metadata ja isikutuvastuse päringu vastuvõtu otspunktidele (eIDAS konnektorteenuse metadata vastuses viidatud kui `AssertionConsumerService`)|
| `Võtmehoidla` | SAML vastuste allkirjastamiseks vajalikke võtmeid hoitakse võtmehoidlates (pkcs12, jks). |
| `Konfiguratsioon` | Teenuse juhtimine ja seadistus toimib läbi keskse konfiguratsioonifaili. |
Tabel 2.


PAIGALDUSJUHEND
-------------------

// Täpsustamist vajavad nõuded keskkonnale ja juhised kliendi kokkuehitamiseks //
// Täpsustamist vajavad seadistamise detailid (sh konnektorteenuse registreerimine, võtmete import, krüptoalgoritmide, otspunktide, https-i eripärade ja logimise seadistus) //

LISA 1 - TEENUSE LIIDESE SPETSIFIKATSIOON
-----------

### /start

Algatab autentimisprotsessi või kui vajalikud parameetrid puuduvad, kuvab kasutajale sihtriigid ülepiirilise autentimise alustamiseks. Võimalike parameetrite loetelu toodud tabelis 3.

| Parameetri nimi        | Kohustuslik           | Selgitus  |
| ------------- |:-------------:| :-----|
| `Country` |	Ei | Parameeter, mis määrab ära tuvastatava kodaniku riigi. Väärtus peab olema vastama ISO 3166-1 alpha-2 standardis toodule. <br>Kui määramata, kuvatakse HTML vorm riigivalikuga. |
| `LoA` |	Ei | Parameeter, mis määrab ära nõutava isikutuvastuse taseme. Lubatud väärtused: `low`, `substantial`, `high`. <br>Kui parameeter määramata, siis vaikimisi loetakse väärtuseks `low`. |
Tabel 3.

### /returnUrl

Võtab vastu ja töötleb eIDAS konnektorteenusest tuleva vastuse. Kuvab tulemuse inimloetaval või masinloetaval kujul (vt parameeter `RelayState`). Lubatud parameetrite loetelu toodud tabelis 4.

| Parameetri nimi        | Kohustuslik           | Selgitus  |
| ------------- |:-------------:| :-----|
| `SAMLResponse` | Jah | Parameeter, milles tagastatakse sisaldab Bas64e kodeeritud SAML `Response` päring. Vastus peab olema allkirjastatud ja isiku kohta käivad väited krüpteeritud (eIDAS Node privaatvõtmega). |
| `RelayState` | Ei | Parameetri alusel otsustatkse millisel kujul kasutajale vastus kuvada. <ul><li>Kui parameeter `RelayState` eksisteerib ja selle väärtus on `showHtmlReport`, kuvatakse autentimise tulemus inimloetaval kujul html vormina. </li><li>Muul juhul, kui parameetrit ei täpsustata, saadetakse otspunkti poole pöördujale vastus masinloetaval kujul (json).</li></ul> |
Tabel 4.

### /metadata

Publitseerib eIDAS klient teenuse metaandmed. Parameetrid puuduvad.


LISA 2 - ADMIN funktsionaalsus
---------------

// Vajab täpsustamist //



