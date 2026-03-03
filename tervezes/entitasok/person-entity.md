# Person (Személy) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer  
> **Verzió:** 0.1 – első tervezet  
> **Utolsó frissítés:** 2025. február  
> **Kapcsolódó dokumentum:** erintett-torvenyek.md

---

## 1. Az entitás célja és hatóköre

A **Person** entitás a természetes személy jogviszony-független alapadatait tartalmazza. Ez az HRMS központi entitása, amelyhez az összes foglalkoztatási jogviszony (Employment), végzettség (Qualification), bankszámla (BankAccount) stb. kapcsolódik.

**Tervezési alapelvek:**

- **Jogviszony-független:** A Person entitás nem tartalmaz jogviszony-specifikus adatokat (pl. besorolás, illetmény, munkakör). Ezek az Employment és kapcsolódó entitásokba kerülnek.
- **Egy személy – egy rekord:** Akkor is, ha ugyanaz a személy több jogviszonnyal rendelkezik (pl. részmunkaidős tanár + óraadó másik intézményben), egyetlen Person rekord tartozik hozzá.
- **GDPR-konform:** Csak a jogszabály vagy a szerződés teljesítése szempontjából szükséges adatok tárolása; célhoz kötöttség; adattakarékosság.
- **Időbeliség:** A változó adatoknál (név, lakcím, családi állapot) historikus nyilvántartás szükséges – az aktuális állapot mellett a korábbi értékek és azok érvényességi időszaka is tárolódik.

---

## 2. Property-k

### 2.1. Azonosítók

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `taxId` | string(10) | igen | Adóazonosító jel | Szja tv. 3. §; Tbj.; Art. |
| `socialSecurityNumber` | string(9) | igen | TAJ-szám | Tbj. 36. §; Ebtv. |
| `documentNumber` | string | nem | Személyazonosító okmány száma | Mt. 10. § – okirat bemutatása kérhető, másolat nem készíthető; csak az azonosításhoz szükséges adatok rögzíthetők |
| `documentType` | enum | nem | Okmány típusa (személyi igazolvány, útlevél, jogosítvány, tartózkodási engedély) | Mt. 10. § |
| `documentExpiry` | date | nem | Okmány érvényessége | Lejáratfigyeléshez (különösen külföldi munkavállalóknál) |

**Megjegyzések:**
- A **TAJ-szám** és **adóazonosító jel** kezelése jogszabályi kötelezettség (GDPR 6(1)c).
- Személyi igazolvány másolata **nem** tárolható (Mt. 10. § (2), NAIH állásfoglalás). Csak az azonosításhoz szükséges adatok (szám, típus, érvényesség) jegyezhetők fel.
- Külföldi munkavállalóknál a munkavállalási engedély/EU kártya nyilvántartása szükséges lehet.

---

### 2.2. Személyes alapadatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `titlePrefix` | string | nem | Előtag (Dr., Prof., ifj. stb.) | Mt.; Kjt.; közszférás nyilvántartás |
| `familyName` | string | igen | Családi név (születési) | Mt. 44. § tájékoztatási köt.; Tbj. bejelentés |
| `givenNames` | string | igen | Utónév/utónevek (születési) | Mt. 44. §; Tbj. |
| `marriedName` | string | nem | Házassági név (ha eltér a születési névtől) | Gyakorlati szükséglet; Tbj. bejelentés |
| `preferredName` | string | nem | Használt név (az a név, amelyen a személy a szervezeten belül ismert) | Gyakorlati; nem jogszabályi |
| `previousNames` | array | nem | Korábbi nevek és érvényességi időszakok | Historikus nyilvántartás; Tbj. |
| `birthDate` | date | igen | Születési dátum | Mt. (pótszabadság, munkavédelmi korhatár); Tbj.; Szja tv. (25 év alattiak kedvezménye) |
| `birthPlace` | string | igen | Születési hely | Tbj. bejelentés; személyazonosítás |
| `birthName` | string | igen | Születési név (anyja neve nélkül, teljes születési név) | Tbj. bejelentés |
| `motherBirthName` | string | igen | Anyja születési neve | Tbj.; személyazonosítás – magyar jogban kiemelt azonosító adat |
| `gender` | enum | igen | Nem (férfi / nő) | Mt. (várandósság alatti védelem); Tbj.; statisztikai adatszolgáltatás (KSH) |
| `nationality` | string | igen | Állampolgárság | Munkaengedély-kötelezettség vizsgálata; EU/EGT vs. harmadik országbeli |
| `nationalities` | array | nem | További állampolgárságok | Többes állampolgárság esetén |

**Megjegyzések:**
- A **születési név + anyja születési neve + születési hely + születési dátum** együttesen a magyar jog szerinti elsődleges természetes személyazonosítók.
- Az Mt. 10. § alapján az okirat (személyi igazolvány) bemutatása kérhető a fenti adatok ellenőrzéséhez, de másolat nem készíthető róla.
- A `gender` property-nél az Mt. és a Tbj. bináris (férfi/nő) kategóriákat használ; a GDPR adattakarékossági elve alapján csak akkor tárolható, ha konkrét jogszabályi célhoz kötött (pl. várandóssági védelem, statisztika).

---

### 2.3. Elérhetőségek

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `permanentAddress` | Address | igen | Állandó lakcím | Mt. 44. §; Tbj. bejelentés; Szja tv. (illetékesség) |
| `temporaryAddress` | Address | nem | Ideiglenes lakcím | Tbj. |
| `mailingAddress` | Address | nem | Levelezési cím (ha eltér a lakcímtől) | Gyakorlati |
| `personalPhone` | string | nem | Személyes telefonszám | Mt. 10. § – csak ha szükséges; GDPR 6(1)f jogos érdek (vészhelyzeti elérhetőség) |
| `personalEmail` | string | nem | Személyes e-mail cím | GDPR 6(1)f – korlátozottan; értesítések |
| `workPhone` | string | nem | Munkahelyi telefon | Employment entitásba is kerülhet |
| `workEmail` | string | nem | Munkahelyi e-mail | Employment entitásba is kerülhet |

**Address altípus:**

| Property | Típus | Leírás |
|---|---|---|
| `country` | string(2) | Országkód (ISO 3166-1 alpha-2) |
| `postalCode` | string | Irányítószám |
| `city` | string | Település |
| `streetAddress` | string | Utca, házszám, emelet, ajtó |
| `validFrom` | date | Érvényesség kezdete |
| `validTo` | date | Érvényesség vége (null = aktuális) |

**Megjegyzések:**
- A lakcím **historikusan** nyilvántartandó, mert az adóbevalláshoz és a járulékfizetéshez szükséges az adott időszakban érvényes cím.
- A személyes telefonszám és e-mail kezelése **jogos érdek** (GDPR 6(1)f) alapján lehetséges, de érdekmérlegelési teszt szükséges. A munkáltató elsősorban vészhelyzeti kapcsolattartás céljából kezelheti.

---

### 2.4. Családi és személyes körülmények

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `maritalStatus` | enum | nem | Családi állapot (nőtlen/hajadon, házas, elvált, özvegy, élettárs, bejegyzett élettárs) | Mt. (első házasok kedvezménye); Szja tv. |
| `maritalStatusDate` | date | nem | Családi állapot változásának dátuma | Szja tv. – első házasok kedvezménye igénylése |
| `spouses` | array[Spouse] | nem | Házastárs(ak) adatai | Szja tv. 46. § (közös adózás); 29/A. § (első házasok kedvezménye); 29/C. § (családi kedvezmény megosztása) |
| `dependents` | array[Dependent] | nem | Eltartottak / gyermekek | Mt. (pótszabadság); Szja tv. (CSOK, családi kedvezmény); Tbj. |
| `disabilityStatus` | enum | nem | Megváltozott munkaképesség / fogyatékosság státusza | Mt. 120. § (pótszabadság); Szja tv. (személyi kedvezmény); Szocho tv. (kedvezmény); Mvt. |
| `disabilityPercentage` | integer | nem | Egészségi állapot százalékos mértéke | Rehabilitációs hozzájárulás; Szocho kedvezmény |
| `disabilityValidUntil` | date | nem | Határozat érvényessége | Lejáratfigyelés |

**Spouse (Házastárs) altípus:**

| Property | Típus | Leírás |
|---|---|---|
| `name` | string | Házastárs neve |
| `taxId` | string(10) | Adóazonosító jel (közös adózáshoz és kedvezményekhez KÖTELEZŐ) |
| `socialSecurityNumber` | string(9) | TAJ szám (opcionális) |
| `birthDate` | date | Születési dátum (első házasok kedvezménye ellenőrzéséhez) |
| `isJointTaxation` | boolean | Közös adózást választották-e (Szja tv. 46. §) |
| `jointTaxationStartYear` | integer | Közös adózás kezdő éve |
| `firstMarriageForBoth` | boolean | Mindkettő első házassága-e (első házasok kedvezménye - Szja tv. 29/A. §) |
| `familyAllowanceShare` | decimal | Családi kedvezmény megosztási aránya (0-100%) |
| `validFrom` | date | Házasság/bejegyzett élettársi kapcsolat kezdete |
| `validTo` | date | Érvényesség vége (válás/haláleset dátuma - historikus nyilvántartás) |
| `isEmployedInSameOrg` | boolean | Ugyanabban a szervezetben dolgozik-e |
| `relatedPersonId` | UUID | Ha a rendszerben van (Person FK - elkerüli a duplikációt) |

**Dependent (Eltartott) altípus:**

| Property | Típus | Leírás |
|---|---|---|
| `name` | string | Eltartott neve |
| `birthDate` | date | Születési dátum |
| `relationship` | enum | Kapcsolat típusa (gyermek, örökbefogadott gyermek, nevelt gyermek, unoka) |
| `taxId` | string | Eltartott adóazonosító jele (családi kedvezményhez) |
| `disability` | boolean | Tartósan beteg / fogyatékos-e (emelt pótszabadság, emelt kedvezmény) |
| `sharedCustody` | boolean | Megosztott felügyelet |

**Megjegyzések:**
- A **házastársi adatok** kezelése harmadik fél személyes adatainak kezelését jelenti → GDPR 6(1)(c) jogalap (jogi kötelezettség: Szja tv. közös adózás, kedvezmények). A házastársat is tájékoztatni kell az adatkezelésről (GDPR 14. cikk), vagy a munkavállaló nyilatkozik, hogy tájékoztatta.
- **Közös adózás (Szja tv. 46. §):** Mindkét házastárs adóazonosító jele kötelező a közös bevalláshoz.
- **Első házasok kedvezménye (Szja tv. 29/A. §):** 24 hónap, max 33 eFt/hó adókedvezmény; mindkét fél első házassága szükséges; házastárs adóazonosító jele kötelező.
- **Családi kedvezmény megosztása (Szja tv. 29/C. §):** Gyermek után járó kedvezmény házastársak között megosztható; házastárs adóazonosító jele kötelező.
- **Minimalizálás:** Csak adóbevalláshoz szükséges adatok tárolhatók (név, TAJ, adóazonosító, születési dátum); lakcím, telefonszám NEM.
- **Megőrzési idő:** Jogosultság fennállása + elévülési idő (5 év).
- A gyermekek adatai **több jogszabályi célhoz** szükségesek: Mt. szerinti pótszabadság (gyermekenként 2–7 nap az életkortól függően), Szja tv. szerinti családi kedvezmény, apasági szabadság, GYED/GYES jogosultság.
- A **fogyatékossági/megváltozott munkaképességi adatok** különleges személyes adatnak minősülnek (GDPR 9. cikk) → kezelésük jogalapja: jogi kötelezettség (GDPR 9(2)b) – munkavállaló által benyújtott határozat alapján.
- A családi állapot kezelése csak konkrét jogszabályi cél (pl. első házasok kedvezménye) esetén indokolt; önmagában a „családi állapot" nem szükséges minden HRMS-ben.

---

### 2.5. Bankszámla és kifizetési adatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `bankAccounts` | array[BankAccount] | igen | Bankszámlaszám(ok) | Mt. 158. § – munkabér átutalása; a munkáltató köteles a munkavállaló által megjelölt bankszámlára utalni |

**BankAccount altípus:**

| Property | Típus | Leírás |
|---|---|---|
| `accountNumber` | string | Bankszámlaszám (magyar 3x8 formátum vagy IBAN) |
| `bankName` | string | Számlavezető bank neve |
| `iban` | string | IBAN (külföldi munkavállalóknál) |
| `swift` | string | SWIFT/BIC kód |
| `isPrimary` | boolean | Elsődleges számla-e |
| `validFrom` | date | Érvényesség kezdete |
| `validTo` | date | Érvényesség vége |

**Megjegyzések:**
- A bankszámlaszám kezelése a **szerződés teljesítéséhez** szükséges (GDPR 6(1)b) – a munkabér kifizetéséhez.
- Historikusan nyilvántartandó a bérszámfejtés és az utólagos ellenőrzés miatt.

---

### 2.6. Vészhelyzeti kapcsolattartó

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `emergencyContacts` | array[EmergencyContact] | nem | Vészhelyzeti kapcsolattartók | Mvt.; GDPR 6(1)f jogos érdek |

**EmergencyContact altípus:**

| Property | Típus | Leírás |
|---|---|---|
| `name` | string | Kapcsolattartó neve |
| `relationship` | string | Kapcsolat (házastárs, szülő, testvér stb.) |
| `phone` | string | Telefonszám |
| `email` | string | E-mail (opcionális) |

**Megjegyzések:**
- Harmadik személy személyes adatainak kezelése → a GDPR szerint az érintettet (vészhelyzeti kontakt) is tájékoztatni kell, vagy a munkavállalónak nyilatkoznia kell, hogy tájékoztatta.
- Jogalapja: **jogos érdek** (GDPR 6(1)f) – a munkahelyi baleset esetén értesítendő személy biztosítása; érdekmérlegelési teszt szükséges.

---

### 2.7. Audit mezők

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `createdAt` | datetime | igen | Rekord létrehozásának időpontja |
| `createdBy` | string | igen | Létrehozó felhasználó azonosítója |
| `updatedAt` | datetime | igen | Utolsó módosítás időpontja |
| `updatedBy` | string | igen | Utolsó módosító felhasználó |
| `isActive` | boolean | igen | Aktív rekord-e (soft delete) |
| `dataSource` | enum | nem | Adatforrás (manuális, migráció, integráció, önkiszolgáló) |

---

## 3. Historikus adatkezelés (temporal pattern)

Az alábbi property-k **időszakos érvényességgel** kezelendők (nem felülírás, hanem új verzió létrehozása):

| Property-csoport | Indok |
|---|---|
| Név (familyName, givenNames, marriedName) | Névváltozás (házasság, válás) – a korábbi név a régi dokumentumokon érvényes |
| Lakcím (permanentAddress, temporaryAddress) | Adóbevallás, járulékfizetés az adott időszakban érvényes cím alapján |
| Családi állapot | Kedvezmények igénybevételi jogosultsága |
| Házastárs (spouses) | Válás, újraházasodás → korábbi házastársi adatok megőrzése adóbevallási ellenőrzéshez |
| Bankszámla | Korábbi utalások visszakereshetősége |
| Okmányadatok | Lejárt okmányok nyilvántartása |
| Eltartottak | Gyermek 16/18 éves korának betöltése → pótszabadság, kedvezmény változása |

**Megvalósítási minta:**

```
PersonHistory {
  personId: UUID (FK → Person)
  fieldGroup: enum (name | address | maritalStatus | spouse | bankAccount | document | dependent)
  validFrom: date
  validTo: date (null = aktuális)
  data: JSON
  changedAt: datetime
  changedBy: string
  changeReason: string
}
```

---

## 4. GDPR adatkezelési kategorizáció

| Kategória | Property-k | Jogalap | Megőrzési idő |
|---|---|---|---|
| **Azonosító adatok** | taxId, socialSecurityNumber, documentNumber | Jogi kötelezettség (6(1)c) | Jogviszony megszűnése + 5 év (Art.), TB: +5 év (Tbj.) |
| **Személyazonosító adatok** | familyName, givenNames, birthDate, birthPlace, motherBirthName | Jogi kötelezettség (6(1)c) + Szerződés (6(1)b) | Jogviszony megszűnése + 5 év |
| **Kapcsolattartási adatok** | personalPhone, personalEmail, permanentAddress | Szerződés (6(1)b) + Jogos érdek (6(1)f) | Jogviszony megszűnésekor felülvizsgálandó; törlés, ha nincs más jogalap |
| **Különleges adatok** | disabilityStatus, disabilityPercentage | Jogi kötelezettség (9(2)b) | Határozat érvényessége + jogviszony megszűnése + 3 év |
| **Házastársi adatok** | spouses[].* | Jogi kötelezettség (6(1)c) – Szja tv. közös adózás, kedvezmények | Jogosultság fennállása + elévülési idő (5 év); harmadik fél tájékoztatása kötelező |
| **Eltartottak adatai** | dependents[].* | Jogi kötelezettség (6(1)c) – adókedvezmény, pótszabadság | Jogosultság fennállása + elévülési idő (5 év) |
| **Bankszámla adatok** | bankAccounts[].* | Szerződés (6(1)b) | Jogviszony megszűnése + 8 év (Szt. – számviteli bizonylat) |
| **Vészhelyzeti kontakt** | emergencyContacts[].* | Jogos érdek (6(1)f) | Jogviszony megszűnésekor törlendő |
| **Audit adatok** | createdAt, createdBy, updatedAt, updatedBy | Jogos érdek (6(1)f) + Elszámoltathatóság (GDPR 5(2)) | A kapcsolódó adat megőrzési idejével megegyező |

---

## 5. Hozzáférési szintek (javasolt)

| Szerep | Olvasási jog | Írási jog | Megjegyzés |
|---|---|---|---|
| **Érintett munkavállaló** | Saját összes adat | Korlátozott (elérhetőségek, bankszámla, vészhelyzeti kontakt, házastárs, eltartottak) | GDPR 15. cikk – hozzáférési jog; önkiszolgáló portál |
| **Közvetlen vezető** | Név, munkavégzési hely, elérhetőség | Nincs | Szükséges és arányos mérték |
| **HR ügyintéző** | Összes, a hatáskörébe tartozó szervezeti egységen belül | Összes (audit naplózással) | Feladatellátáshoz szükséges |
| **Bérszámfejtő** | Azonosítók, bankszámla, házastárs, eltartottak, adókedvezmények | Bankszámla, házastárs, eltartottak | Bérszámfejtési feladat (adóbevallás) |
| **Rendszeradminisztrátor** | Technikai mezők | Technikai mezők | Érdemi személyes adatokhoz nem fér hozzá |

---

## 6. Validációs szabályok

| Property | Validáció | Megjegyzés |
|---|---|---|
| `taxId` | 10 számjegy; 10. jegy = ellenőrző szám (MOD 11 algoritmus) | Magyar adóazonosító jel formátum |
| `socialSecurityNumber` | 9 számjegy; CDV ellenőrzés | TAJ-szám formátum |
| `birthDate` | Nem lehet jövőbeli; a személy életkora 14–100 év között (Mt. 34. § – 16 éves kortól, kivételesen 15 évestől) | Fiatalkorú munkavállaló speciális szabályok |
| `permanentAddress.postalCode` | Magyar cím esetén 4 számjegy | – |
| `bankAccounts[].accountNumber` | Magyar: 3×8 számjegy (ellenőrző számjegyes); vagy IBAN formátum | IBAN: HU + 2 ellenőrző + 24 számjegy |
| `nationality` | ISO 3166-1 alpha-2 kód | – |
| `gender` | enum: `MALE`, `FEMALE` | Jogszabályi bejelentésekhez szükséges bináris mező |

---

## 7. Integrációs pontok

| Külső rendszer | Irány | Érintett property-k | Cél |
|---|---|---|---|
| **NAV (T1041)** | kimenő | taxId, socialSecurityNumber, familyName, givenNames, birthDate, birthPlace, motherBirthName, nationality, permanentAddress | Biztosítotti bejelentés |
| **NAV (08-as bevallás)** | kimenő | taxId, járulék- és adóadatok | Havi adó- és járulékbevallás |
| **NEAK** | kimenő | socialSecurityNumber | Egészségbiztosítási jogviszony |
| **KSH** | kimenő | gender, birthDate, nationality (aggregált) | Statisztikai adatszolgáltatás (létszám, munkaügy) |
| **Bérszámfejtő rendszer** | kétirányú | Azonosítók, bankszámla, eltartottak | Bérszámfejtés |
| **Önkiszolgáló portál** | bejövő | Elérhetőségek, bankszámla, vészhelyzeti kontakt | Munkavállaló által kezdeményezett módosítások (jóváhagyási workflow-val) |

---

## 8. Kapcsolódó entitások (hivatkozás)

```
Person 1 ──── N Employment         (jogviszonyok)
Person 1 ──── N Qualification      (végzettségek, képesítések)
Person 1 ──── N PersonHistory      (historikus adatok)
Person 1 ──── N BankAccount        (bankszámlák – historikus)
Person 1 ──── N Document           (okmányok, igazolások)
Person 1 ──── N Spouse             (házastárs/ak – historikus)
Person 1 ──── N Dependent          (eltartottak)
Person 1 ──── N EmergencyContact   (vészhelyzeti kontaktok)
Person o──── o Person              (Spouse.relatedPersonId – ha mindketten dolgoznak a szervezetnél)
```

---

## 9. Nyitott kérdések

1. **Fénykép tárolása:** A munkavállalói fénykép kezelése GDPR szempontból problémás (biometrikus adat, ha azonosításra használják). Szükséges-e a rendszerben? Ha igen, milyen jogalapon?
2. **Nemzetiségi hovatartozás:** Egyes közszférás törvények (Púétv. – nemzetiségi pótlék) igényelhetik, de különleges adat (GDPR 9. cikk). Hogyan kezeljük?
3. **Előző munkáltató adatai:** A jogviszonyban töltött idő beszámításához (Kjt., Eszjtv., Kttv.) szükségesek lehetnek korábbi foglalkoztatási adatok. Ez a Person vagy az Employment entitásba tartozzon?
4. **Külföldi munkavállalók:** Munkavállalási engedély, EU kártya, tartózkodási engedély nyilvántartása – önálló entitás vagy a Person Document alentitása?
5. **Több szervezet közötti megosztás:** Ha a személy több munkáltatónál is dolgozik (pl. kirendelés, többes jogviszony), hogyan kezeljük az adatszuverenitást?

---

### 9.1. Javaslatok és válaszok

#### 9.1.1. Fénykép tárolása

**Javaslat:** Csak akkor tároljuk, ha **objektíven szükséges** (pl. belépőkártya, azonosítás).

**GDPR jogalap:**
- Ha **nem** használják automatizált arc-felismerésre → nem minősül biometrikus adatnak (GDPR 9. cikk)
- Jogalap: GDPR 6(1)(f) – jogos érdek (objektum- és vagyonvédelem, beléptetés)
- Ha arc-felismerésre használják → biometrikus adat → GDPR 9(2)(b) szükséges (foglalkoztatási jog) vagy explicit hozzájárulás

**Megvalósítás:**
```
EmployeePhoto {
  id: UUID PK
  personId: UUID FK
  employmentId: UUID FK (opcionális - ha jogviszony-specifikus)
  photoURL: string (tárolási útvonal)
  purpose: enum (badge, identification, directory)
  consentGiven: boolean
  consentDate: date
  capturedAt: date
  validUntil: date (pl. jogviszony megszűnésekor törlendő)
  gdprCategory: enum (basic, biometric)
}
```

**Alternatíva:** Ha nem feltétlenül szükséges, **ne tároljuk**.

**GDPR adatkezelési kategorizáció:**
- Ha csak megjelenítésre használják (belépőkártya, telefonkönyv): alapadat, GDPR 6(1)(f) jogos érdek
- Ha arc-felismeréshez: biometrikus adat, GDPR 9. cikk, megőrzési idő: jogviszony megszűnésekor törlendő

---

#### 9.1.2. Nemzetiségi hovatartozás

**Javaslat:** Csak akkor kezeljük, ha **jogszabályi kötelezettség** (pl. Púétv. 65. § nemzetiségi pótlék).

**GDPR jogalap:**
- GDPR 9. cikk – különleges adatkategória (etnikai származás)
- Kivétel: GDPR 9(2)(b) – foglalkoztatási jog szerinti kötelezettség

**Megvalósítás:**
- **NEM** a Person entitásba tartozik (túl széles körű hozzáférés)
- **CompensationElement** vagy önálló entitás:

```
NationalityBenefit {
  id: UUID PK
  employmentId: UUID FK
  nationalityGroup: enum (hungarian, german, slovak, romanian, serbian, croatian, slovenian, other)
  benefitType: string "Nemzetiségi pótlék"
  benefitEligible: boolean
  declarationDate: date
  selfDeclaration: boolean (önkéntes nyilatkozat - nem kötelező)
  consentGiven: boolean
  validFrom: date
  validTo: date
  documentId: UUID FK → Document (nemzetiségi igazolvány, ha van)
  legalBasis: string "Púétv. 65. §"
}
```

**Fontos szabályok:**
- **Önkéntes nyilatkozat** alapján (a munkavállaló dönt, nyilatkozik-e)
- **Minimalizálás:** csak a pótlék jogosultsága rögzítendő, nem a teljes nemzetiségi/etnikai háttér
- **Célhoz kötöttség:** kizárólag a pótlék megállapításához használható
- **Hozzáférés korlátozása:** csak HR/bérszámfejtő, audit naplózással
- **Megőrzési idő:** jogosultság megszűnése + elévülési idő (5 év)

---

#### 9.1.3. Előző munkáltató adatai

**Javaslat:** **Employment entitásba** tartozzon, mert jogviszony-specifikus adat.

**Indokok:**
- Kjt. 71. §, Kttv. 132. §, Kit. 136. §, Eszjtv. 43. §: jogviszonyban töltött idő beszámítása (fizetési fokozat, jubileumi jutalom)
- Különböző jogviszonytípusoknál eltérő lehet a beszámítandó idő
- Nem a személy, hanem az adott jogviszony jellemzője

**MVP megvalósítás** (Employment entitásban, egyszerű változat):

```
Employment {
  ...
  priorServiceYears: decimal (beszámított korábbi szolgálati idő - évek)
  priorServiceMonths: integer (hónapok a törtév pontosításához)
  priorServiceStartDate: date (korábbi szolgálat kezdete - referencia)
  priorServiceDocumentId: UUID FK → Document (munkáltatói igazolás)
  priorServiceVerified: boolean (igazolással alátámasztott-e)
  priorServiceNotes: text (megjegyzés)
}
```

**Részletes megvalósítás** (önálló entitás, ha több korábbi munkáltató nyilvántartása szükséges):

```
PriorEmployment {
  id: UUID PK
  personId: UUID FK
  employmentId: UUID FK (melyik jelenlegi jogviszonyhoz számít be)
  employerName: string
  employerType: enum (private, public, nonprofit, foreign)
  employmentType: enum (mt, kjt, kttv, kit, puetv, eszjtv, kut, foreign)
  positionTitle: string
  startDate: date
  endDate: date
  yearsOfService: decimal (számított)
  isCreditedForSeniority: boolean (beszámít-e a jelenlegi jogviszonyba)
  isPensionCredited: boolean (nyugdíjjárulék-fizetés volt-e)
  documentId: UUID FK → Document (munkáltatói igazolás, munkakönyv)
  verificationDate: date
  notes: text
}
```

**Ajánlás:**
- **MVP fázis:** Employment entitás mezői (egyszerű verzió)
- **Extended fázis:** PriorEmployment entitás (ha részletes karriertörténet nyilvántartás szükséges)

**GDPR adatkezelés:**
- Jogalap: GDPR 6(1)(c) jogi kötelezettség (fizetési fokozat megállapítása)
- Megőrzési idő: jogviszony megszűnése + 5 év (ellenőrzési kötelezettség)

---

#### 9.1.4. Külföldi munkavállalók (munkavállalási engedély, EU kártya)

**Javaslat:** Kezdetben **Document entitás kategóriája**, de ha komplex workflow szükséges → **önálló WorkPermit entitás**.

**MVP megoldás – Document entitás:**

```
Document {
  ...
  documentCategory: enum (..., work_permit, residence_permit, eu_card, visa, posted_worker_a1)
  permitType: enum (single_permit, EU_blue_card, seasonal_work, intra_corporate_transfer, posted_worker, etc.)
  issueDate: date
  expiryDate: date (KRITIKUS - lejáratfigyelés kötelező!)
  issuingAuthority: string (pl. "Bevándorlási és Menekültügyi Hivatal")
  permitNumber: string
  workRestrictions: text (pl. "csak építőiparban", "max 6 hónap/év")
  isRenewable: boolean
  renewalNoticeDays: integer (hány nappal a lejárat előtt kell értesíteni)
}
```

**Extended megoldás – önálló WorkPermit entitás** (ha engedélyezési workflow, értesítések, megújítás kezelése szükséges):

```
WorkPermit {
  id: UUID PK
  personId: UUID FK
  employmentId: UUID FK (ha jogviszony-specifikus)
  permitType: enum (single_permit, EU_blue_card, seasonal_work, posted_worker, intra_corporate_transfer, etc.)
  permitNumber: string
  issueDate: date
  expiryDate: date
  issuingAuthority: string
  issuingCountry: string(2) (ISO 3166-1)
  workRestrictions: text
  occupationRestrictions: text (FEOR kód vagy szöveges leírás)
  geographicRestrictions: text (pl. csak Budapest)
  maxWorkHoursPerWeek: integer
  status: enum (pending, approved, active, expired, revoked, renewal_in_progress)
  renewalRequestDate: date
  renewalDeadline: date
  expiryNotificationSent: boolean
  notificationDate: date
  documentId: UUID FK → Document (szkennelt engedély)
  legalBasis: string "2007. évi II. tv. (harmadik országbeli állampolgárok beutazásáról és tartózkodásáról)"
  notes: text
}
```

**Kapcsolódó workflow igények:**
- **Lejáratfigyelés:** Automatikus értesítés 60/30/7 nappal a lejárat előtt (HR, munkavállaló, vezető)
- **Megújítási folyamat:** Workflow támogatás (dokumentumok összegyűjtése, benyújtás nyomon követése)
- **Munkaszerződés blokkolása:** Ha nincs érvényes engedély, nem lehet új Employment-et aktiválni
- **NAV bejelentés integráció:** T1041 bejelentés ellenőrzése (csak érvényes engedéllyel)

**Ajánlás:**
- **MVP:** Document entitás + kategorizálás (work_permit, residence_permit)
- **Extended:** WorkPermit entitás + lejáratfigyelő értesítések + workflow

**GDPR adatkezelés:**
- Jogalap: GDPR 6(1)(c) jogi kötelezettség (2007. II. tv. foglalkoztatási tilalom)
- Megőrzési idő: jogviszony megszűnése + 5 év (ellenőrzési kötelezettség - Munkavédelmi Hatóság, NAV)

---

#### 9.1.5. Több szervezet közötti megosztás

**Javaslat:** **Person központi törzsadat, Employment szervezet-specifikus.**

**Forgatókönyvek és megoldások:**

##### A) Azonos HRMS rendszer, több jogi entitás

```
Person (központi törzs, egyszer tárolva)
  ├── Employment #1 (Organization A, Kjt., főállás)
  ├── Employment #2 (Organization B, Mt., részmunkaidős)
  └── Employment #3 (Organization C, Óraadó megbízás)

Architektúra:
- Person entitás: NEM tartalmaz organizationId mezőt (központi)
- Employment entitás: Kapcsolótábla Person ↔ Organization között
- Hozzáférés-szabályozás: Row-level security (RLS) - szervezetenként elkülönített
- Adatkezelés: Minden szervezet önállóan kezeli a saját Employment adatait
```

**Technikai megvalósítás:**
```sql
-- Row-level security példa (PostgreSQL)
CREATE POLICY employment_organization_isolation ON employment
  USING (organization_id = current_setting('app.current_organization_id')::uuid);
```

##### B) Kirendelés / kikölcsönzés (Mt. 53. §, Kjt. 41-42. §)

```
Person
  └── Employment (eredeti munkáltató - Organization A)
       └── Assignment (kirendelés Organization B-hez)
            - assignmentType: enum (secondment, posted_worker, temporary_assignment)
            - hostOrganizationId: UUID FK
            - assignmentStartDate: date
            - assignmentEndDate: date
            - workLocation: Site FK
            - hostSupervisorId: UUID FK (fogadó szervezetnél a vezető)
            - costSharing: decimal (költségmegosztás %-ban)
            - assignmentAgreementId: UUID FK → Document
```

**Szabályok:**
- **Employment megmarad** az eredeti munkáltatónál (nem szűnik meg)
- **Assignment entitás** rögzíti a kirendelést
- **Bérszámfejtés:** Továbbra is az eredeti munkáltató végzi
- **Munkaidő-nyilvántartás:** Fogadó szervezet is rögzítheti (DailyTimeRecord.assignmentId)
- **Hozzáférés:** Fogadó szervezet korlátozott adatokat lát (név, munkavégzési hely, elérhetőség)

##### C) Különböző HRMS rendszerek (több munkáltató, külön rendszerek)

**Probléma:** Adatszuverenitás, GDPR közös adatkezelés

**Megoldás 1 – Független rendszerek (ajánlott):**
```
HRMS A (Organization A)
  └── Person_A (saját adatok)
       └── Employment_A

HRMS B (Organization B)
  └── Person_B (saját adatok, független)
       └── Employment_B

Kapcsolat: Nincs automatikus adatcsere
Azonosítás: TAJ-szám / adóazonosító (nemzeti azonosító)
```

**Megoldás 2 – Központi Identity Provider:**
```
Központi SSO / Identity Provider (pl. nemzeti e-személyi rendszer)
  ├── HRMS A (API-n keresztül lekérdezi az alapadatokat)
  └── HRMS B (API-n keresztül lekérdezi az alapadatokat)

Jogalap:
- GDPR 6(1)(c) – jogi kötelezettség (ha állami rendszer)
- GDPR 6(1)(a) – hozzájárulás (ha magánszféra)
```

**Megoldás 3 – Közös adatkezelői megállapodás (GDPR 26. cikk):**
```
Organization A ↔ Organization B
  - Közös adatkezelői megállapodás
  - Világos felelősségi körök
  - API integráció szabályozott adatcseréhez
  - Munkavállaló tájékoztatása kötelező

Megosztható adatok:
- Név, TAJ, adóazonosító (jogszabályi kötelezettség)
- Lakcím (csak ha szükséges - érdekmérlegelés)
- NEM osztható meg: teljesítményértékelés, fegyelmi ügyek, egészségügyi adatok
```

**Megvalósítási lépések:**

1. **Person entitás szerkezet:**
   - **NEM** tartalmaz `organizationId` mezőt (központi törzsadat)
   - Kapcsolat az Employment entitáson keresztül jön létre

2. **Hozzáférés-szabályozás (RBAC + RLS):**
   ```
   Szabály: Felhasználó csak a saját szervezetéhez tartozó Employment adatokat látja

   SELECT p.*, e.*
   FROM person p
   JOIN employment e ON p.id = e.person_id
   WHERE e.organization_id = current_user_organization_id
   ```

3. **Audit napló:**
   - Minden cross-organization adathozzáférés naplózása kötelező
   - Munkavállaló GDPR 15. cikk alapján betekintési jogot kap

4. **Adatmegosztási mátrix:**

| Adatkategória | Eredeti munkáltató | Fogadó szervezet (kirendelés) | Másik munkáltató (külön HRMS) |
|---|---|---|---|
| Név, TAJ, adóazonosító | Teljes | Csak név | Nincs hozzáférés |
| Lakcím | Teljes | Nincs | Nincs |
| Bankszámla | Teljes | Nincs | Nincs |
| Teljesítményértékelés | Teljes | Korlátozott (csak kirendelés alatti) | Nincs |
| Munkaszerződés | Teljes | Nincs | Nincs |

**GDPR megfelelés:**

- **Közös adatkezelés (GDPR 26. cikk):** Ha két munkáltató együttesen határozza meg az adatkezelés céljait és eszközeit
- **Adattovábbítás (GDPR 6. cikk):** Ha önálló adatkezelők közötti adatátadás történik
- **Tájékoztatási kötelezettség:** Munkavállalót előzetesen tájékoztatni kell, ha adatai több szervezethez kerülnek
- **Adatalany jogok:** Munkavállaló mindkét szervezettől kérheti adatai törlését/helyesbítését

**Ajánlás:**
- **MVP:** Person központi, Employment szervezetenként elkülönített, row-level security
- **Extended:** Assignment entitás kirendeléshez, audit napló cross-organization hozzáférésekhez
- **Strategic:** API integráció külső rendszerekkel (ha szükséges), közös adatkezelői megállapodással

---

### 9.2. Összefoglaló döntési mátrix

| Kérdés | Rövid válasz | Implementáció | Prioritás |
|---|---|---|---|
| **1. Fénykép** | Csak ha objektíven szükséges | `EmployeePhoto` entitás, hozzájárulással | Extended |
| **2. Nemzetiség** | Csak jogszabályi kötelezettség esetén | `NationalityBenefit` entitás, Employment kapcsolattal | Extended (csak Púétv. szektorban) |
| **3. Előző munkáltató** | Employment entitásba tartozik | `priorServiceYears`, `priorServiceStartDate` mezők | MVP |
| **4. Munkavállalási engedély** | Document kategória (MVP), WorkPermit entitás (Extended) | Document: `work_permit` kategória + lejáratfigyelés | MVP (kategória), Extended (entitás) |
| **5. Több szervezet** | Person központi, Employment szervezet-specifikus | Row-level security, Assignment entitás kirendeléshez | MVP (RLS), Extended (Assignment) |
