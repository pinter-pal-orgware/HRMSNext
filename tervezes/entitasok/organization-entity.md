# Organization / OrgUnit (Szervezet / Szervezeti egység) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer  
> **Verzió:** 0.1 – első tervezet  
> **Utolsó frissítés:** 2025. február  
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md, employment-entity.md, compensation-entity.md

---

## 1. Az entitás célja és hatóköre

Az **Organization** entitás a munkáltató jogi személyt, az **OrgUnit** entitás a szervezeten belüli hierarchikus struktúrát reprezentálja. Együttesen adják az HRMS szervezeti gerincét, amelyhez minden jogviszony (Employment), munkakör (Position) és javadalmazás (Compensation) kapcsolódik.

A magyar jogban a „munkáltató" fogalma jogviszony-típusonként eltérő jogi tartalommal bír, és a szervezet szektora alapvetően meghatározza, **milyen jogviszony-típusokat** alkalmazhat. Egy integrált HRMS-nek ezt a kapcsolatot kell modellezni.

**Tervezési alapelvek:**

- **Organization = jogi személy (munkáltató):** A jogviszonyok mindig egy jogi személyhez kötődnek. Egy Organization alatt több OrgUnit létezhet.
- **OrgUnit = szervezeti egység:** Fastruktúra – a hierarchia tetejétől a legkisebb szervezeti egységig. Nem jogi személy, de a hozzáférés-kezelés, a munkáltatói jogkör delegálása és a reporting alapja.
- **Szektor-meghatározás:** A szervezet szektora (versenyszféra, központi közigazgatás, önkormányzat, köznevelés, egészségügy stb.) határozza meg az alkalmazható jogállási törvényeket.
- **Többmunkáltatós modell:** A rendszer több Organization-t is kezelhet (holding, fenntartó + intézmények, önkormányzat + intézményei).
- **Temporális modell:** A szervezeti struktúra változásai (átszervezés, egyesülés, szétválás, fenntartóváltás) historikusan nyilvántartandók.

---

## 2. Organization (Szervezet / Munkáltató)

### 2.1. Azonosítók

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `name` | string | igen | A szervezet hivatalos neve | Cégjegyzék; törzskönyvi nyilvántartás |
| `shortName` | string | nem | Rövid név / közismert név | Gyakorlati |
| `taxNumber` | string(11) | igen | Adószám (8+1+2 formátum) | Art.; NAV bejelentések |
| `statisticalCode` | string(4) | nem | KSH statisztikai számjel (az adószám részeként is megjelenik) | KSH |
| `registrationNumber` | string | nem | Cégjegyzékszám (cégek) vagy törzskönyvi azonosító (költségvetési szervek) | Ctv.; Áht. |
| `socialSecurityRegNumber` | string | nem | TB törzsszám | Tbj. – járulékfizetési bejelentés |
| `omIdentifier` | string | nem | OM-azonosító (köznevelési/felsőoktatási intézmény) | Nkt.; Púétv. |
| `healthcareProviderId` | string | nem | Egészségügyi szolgáltatói kód (ÁNTSZ/NEAK) | Eszjtv.; Eütev. |
| `externalId` | string | nem | Külső rendszer azonosító (migráció, integráció) | Technikai |

---

### 2.2. Szektoriális besorolás

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `sector` | enum | igen | A szervezet szektora (lásd 2.3.) | Meghatározza az alkalmazható jogviszony-típusokat |
| `legalForm` | enum | igen | Jogi forma | Ctv.; Ptk.; Áht. |
| `ownershipType` | enum | igen | Tulajdonosi forma | Mt. 204. § (köztulajdon); Áht. |
| `applicableEmploymentLaws` | array[enum] | számított | Alkalmazható jogállási törvények – a `sector` és `legalForm` alapján a rendszer határozza meg | – |
| `isPublicSector` | boolean | számított | Közszférás munkáltató-e (köztulajdonban álló) | Mt. 204. §; Áht. |
| `isStateOwned` | boolean | számított | Állami tulajdonú-e | Áht.; Nvtv. |
| `isPublicInterestEntity` | boolean | nem | Közérdeklődésre számot tartó gazdálkodó-e | Szt. – közzétételi kötelezettségek |
| `publicDataObligationType` | enum | nem | Közérdekű adatok közzétételének kötelezettsége szintje | Info tv.; Áht. – 2025-ös módosítás: KIF közzétételi kötelezettség |

### 2.3. Szektor enum értékek és jogszabályi megfeleltetés

| Kód | Megnevezés | Jellemző jogviszony-típusok | Irányadó jogszabályok |
|---|---|---|---|
| `PRIVATE` | Versenyszféra | `MUNKAVISZONY` | Mt. |
| `STATE_ENTERPRISE` | Állami vállalat / köztulajdonban álló gazdasági társaság | `MUNKAVISZONY` (Mt. 204–207. § speciális szabályok) | Mt. + sajátos szabályok |
| `CENTRAL_GOV` | Központi közigazgatás (minisztérium, kormányhivatal, központi hivatal) | `KORMANYZATI` + `MUNKAVISZONY` (fizikai dolgozók) | Kit. + Mt. |
| `LOCAL_GOV` | Önkormányzati igazgatás (jegyzői/polgármesteri hivatal) | `KOZSZOLGALATI` + `MUNKAVISZONY` | Kttv. + Mt. |
| `SPECIAL_BODY` | Különleges jogállású szerv | `KULONLEGES` + `MUNKAVISZONY` | Küt. + Mt. |
| `PUBLIC_EDUCATION_STATE` | Köznevelés – állami/tankerületi/önkormányzati fenntartás | `KOZNEVELESI` + `MUNKAVISZONY` (nem pedagógus fizikai) | Púétv. + Mt. |
| `PUBLIC_EDUCATION_PRIVATE` | Köznevelés – egyházi/magán/alapítványi fenntartás | `MUNKAVISZONY` (Púétv. pedagógus-előmenetel alkalmazandó) | Púétv. (részben) + Mt. |
| `HEALTHCARE_STATE` | Egészségügy – állami/önkormányzati | `EGESZSEGUGYI` + `MUNKAVISZONY` (nem eü. dolgozók részben) | Eszjtv. + Mt. |
| `HEALTHCARE_CHURCH` | Egészségügy – egyházi (Eszjtv. alá optált) | `EGESZSEGUGYI` | Eszjtv. (fenntartói döntés) |
| `HEALTHCARE_PRIVATE` | Egészségügy – magán | `MUNKAVISZONY` | Mt. |
| `SOCIAL_SERVICES` | Szociális és gyermekvédelmi intézmény | `KOZALKALMAZOTT` + `MUNKAVISZONY` | Kjt. + Mt. |
| `CULTURE` | Kulturális intézmény (múzeum, könyvtár, levéltár) | `KOZALKALMAZOTT` + `MUNKAVISZONY` | Kjt. + Mt. |
| `HIGHER_EDUCATION` | Felsőoktatás | `MUNKAVISZONY` (modellváltás után) vagy `KOZALKALMAZOTT` (ha maradt Kjt. alatt) | Mt. / Kjt. + Nftv. |
| `VOCATIONAL_EDUCATION` | Szakképzés (SZC, technikum) | `MUNKAVISZONY` (Szkt. speciális szabályokkal) | Szkt. + Mt. |
| `PUBLIC_WORKS` | Közfoglalkoztatás | `MUNKAVISZONY` (speciális) | 1993. évi III. tv. + Mt. |

**Megjegyzések:**
- Egy szervezeten belül **többféle jogviszony-típus** is előfordulhat. Például egy önkormányzat: a hivatalban `KOZSZOLGALATI` (Kttv.), a fenntartott szociális intézményben `KOZALKALMAZOTT` (Kjt.), a fenntartott óvodában `KOZNEVELESI` (Púétv.), a karbantartóknál `MUNKAVISZONY` (Mt.).
- Az `applicableEmploymentLaws` mező **számított**: a `sector` és `legalForm` alapján a rendszer automatikusan meghatározza, milyen jogviszony-típusokat lehet létrehozni az adott szervezetben. Ez az Employment entitás létrehozásakor validációs szabályként működik.

### 2.4. Jogi forma (legalForm)

| Kód | Megnevezés | Tipikus szektor |
|---|---|---|
| `KFT` | Korlátolt felelősségű társaság | Versenyszféra |
| `ZRT` | Zártkörűen működő részvénytársaság | Versenyszféra, állami vállalat |
| `NYRT` | Nyilvánosan működő részvénytársaság | Versenyszféra |
| `BT` | Betéti társaság | Versenyszféra |
| `EV` | Egyéni vállalkozó (ha munkáltató) | Versenyszféra |
| `SZOVETKEZET` | Szövetkezet | Versenyszféra |
| `NONPROFIT_KFT` | Nonprofit Kft. | Szociális, kulturális |
| `NONPROFIT_ZRT` | Nonprofit Zrt. | Szociális, kulturális |
| `EGYESULET` | Egyesület | Civil |
| `ALAPITVANY` | Alapítvány (köz- vagy magánalapítvány) | Felsőoktatás, köznevelés, szociális |
| `KOLTSEGVETESI_SZERV` | Költségvetési szerv | Közigazgatás, egészségügy, köznevelés |
| `ONKORMANYZAT` | Önkormányzat | Helyi közigazgatás |
| `TANKERULETI_KOZPONT` | Tankerületi központ | Köznevelés |
| `EGYHAZI_JOGI_SZEMELY` | Egyházi jogi személy | Köznevelés, egészségügy, szociális |
| `EGYEB` | Egyéb | – |

### 2.5. Tulajdonosi forma (ownershipType)

| Kód | Megnevezés | HRMS hatás |
|---|---|---|
| `CENTRAL_STATE` | Központi állami | Kit. alkalmazandó; közérdekű adatok közzététele; vagyonnyilatkozat |
| `LOCAL_GOV` | Önkormányzati | Kttv. alkalmazandó; közérdekű adatok közzététele |
| `STATE_ENTERPRISE` | Állami tulajdonú gazdasági társaság | Mt. + köztulajdon speciális szabályai (Mt. 204–207. §) |
| `CHURCH` | Egyházi | Mt. vagy választás szerint Eszjtv./Kjt.; speciális munkajogi eltérések |
| `PRIVATE_DOMESTIC` | Belföldi magán | Mt. |
| `PRIVATE_FOREIGN` | Külföldi magán | Mt. |
| `FOUNDATION` | Alapítványi (köz- vagy magán) | Alapítványi felsőoktatás: Mt.; egyéb: Kjt. vagy Mt. |
| `MIXED` | Vegyes tulajdon | Tulajdoni arány alapján Mt. 204. § alkalmazhatóság vizsgálata |

---

### 2.6. Kapcsolattartási és székhely adatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `headquarterAddress` | Address | igen | Székhely címe | Ctv.; Áht. |
| `mailingAddress` | Address | nem | Levelezési cím | Gyakorlati |
| `phone` | string | nem | Központi telefonszám | – |
| `email` | string | nem | Központi e-mail | – |
| `website` | string | nem | Honlap URL | – |
| `mainActivityNACE` | string(6) | igen | Fő tevékenységi kör (TEÁOR/NACE kód) | KSH; cégjegyzék |

---

### 2.7. Fenntartói / tulajdonosi kapcsolat

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `parentOrganizationId` | UUID (FK) | nem | Fenntartó / anyavállalat / irányító szerv | Áht.; Nkt.; Eszjtv. |
| `parentRelationType` | enum | nem | Kapcsolat típusa: `OWNER` (tulajdonos), `MAINTAINER` (fenntartó), `SUPERVISOR` (felügyeleti szerv), `PARENT_COMPANY` (anyavállalat) | – |
| `maintainerType` | enum | nem | Fenntartó típusa (köznevelésnél kötelező): `STATE`, `LOCAL_GOV`, `CHURCH`, `FOUNDATION`, `PRIVATE` | Nkt. 2. § (3)–(4) |

**Megjegyzések:**
- A köznevelésben a **fenntartó személye** alapvetően meghatározza a jogviszony-típust: állami/tankerületi/önkormányzati fenntartás → `KOZNEVELESI` (Púétv.); egyházi/magán → `MUNKAVISZONY` (de a pedagógus-előmenetel alkalmazandó).
- Az egészségügyben az Eszjtv. hatálya kizárólag az **állami és önkormányzati** fenntartású szolgáltatókra terjed ki; egyházi fenntartó dönthet az alkalmazásról; magánszolgáltató kívül esik.
- Költségvetési szerveknél az **irányító szerv** (pl. egy minisztérium mint a tankerületi központok irányítója) is releváns.

---

### 2.8. Működési jellemzők

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `foundedDate` | date | nem | Alapítás / létrehozás dátuma | Cégjegyzék; alapító okirat |
| `terminatedDate` | date | nem | Megszűnés dátuma | Cégjegyzék; törzskönyv |
| `employeeCountCategory` | enum | nem | Létszámkategória (mikro: <10, kis: 10–49, közepes: 50–249, nagy: 250+) | KSH; Mvt.; Ebktv.; GDPR (DPO kijelölési kötelezettség) |
| `collectiveBargainingAgreement` | string | nem | Hatályos kollektív szerződés azonosítója | Mt. 276–284. §; Kjt. |
| `dataProtectionOfficerRequired` | boolean | számított | DPO kijelölési kötelezettség fennáll-e | GDPR 37. cikk – kötelező ha: közfeladatot lát el, nagymértékű adatkezelés, különleges adatok rendszeres kezelése |
| `dataProtectionOfficerName` | string | nem | DPO neve | GDPR 37–39. cikk |
| `workSafetyCategory` | enum | nem | Munkavédelmi kategória | Mvt. – veszélyességi osztály |
| `fiscalYearStart` | date | nem | Üzleti év kezdete (ha eltér jan. 1-től) | Szt. |

---

### 2.9. Audit mezők

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `createdAt` | datetime | igen | Rekord létrehozása |
| `createdBy` | string | igen | Létrehozó |
| `updatedAt` | datetime | igen | Utolsó módosítás |
| `updatedBy` | string | igen | Módosító |
| `isActive` | boolean | igen | Aktív szervezet-e |
| `dataSource` | enum | nem | Adatforrás |

---

## 3. OrgUnit (Szervezeti egység)

### 3.1. Alapadatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `organizationId` | UUID (FK) | igen | Melyik szervezethez tartozik | Technikai |
| `parentOrgUnitId` | UUID (FK) | nem | Szülő szervezeti egység (null = gyökér, azaz a szervezet legfelső szintje) | Technikai – fastruktúra |
| `code` | string | igen | Szervezeti egység kódja (belső azonosító) | Belső szabályzat; SZMSZ |
| `name` | string | igen | Szervezeti egység megnevezése | SZMSZ |
| `shortName` | string | nem | Rövid megnevezés | Gyakorlati |
| `orgUnitType` | enum | igen | Szervezeti egység típusa (lásd 3.2.) | – |
| `level` | integer | számított | Hierarchia szintje (0 = gyökér) | Számított a parentOrgUnitId láncolatból |
| `path` | string | számított | Materialized path (pl. "/1/3/7/") a gyors lekérdezéshez | Technikai |
| `sortOrder` | integer | nem | Megjelenítési sorrend az azonos szinten belül | Gyakorlati |

### 3.2. Szervezeti egység típusok (orgUnitType)

| Kód | Megnevezés | Leírás | Tipikus szektor |
|---|---|---|---|
| `COMPANY` | Cég / szervezet (gyökér) | A legfelső szint, megegyezik az Organization-nel | Mind |
| `DIVISION` | Ágazat / divízió | Nagyvállalati szint | Versenyszféra |
| `DEPARTMENT` | Főosztály / igazgatóság | Funkcionális egység | Mind |
| `UNIT` | Osztály / csoport | Operatív egység | Mind |
| `TEAM` | Csapat / munkacsoport | Kisebb operatív egység | Mind |
| `SITE` | Telephely / munkavégzési hely | Fizikai lokáció alapú egység | Versenyszféra; egészségügy |
| `INSTITUTION` | Intézmény | Fenntartó alatti intézmény (iskola, kórház, szociális intézmény) | Közszféra |
| `FACULTY` | Kar / tagozat | Felsőoktatás, nagy köznevelési intézmények | Felsőoktatás |
| `WARD` | Osztály (egészségügyi) | Kórházi osztály, rendelő | Egészségügy |
| `SCHOOL_SECTION` | Iskolai tagintézmény / feladatellátási hely | Többtelephelyes intézmény | Köznevelés |
| `COST_CENTER` | Költséghely | Nem szervezeti, hanem pénzügyi egység (opcionálisan kezelhető itt is) | Mind |

---

### 3.3. Működési adatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `address` | Address | nem | Szervezeti egység címe (ha eltér a szervezet székhelyétől – telephelyek) | Mt. 45. § – munkavégzési hely |
| `isLegallyIndependent` | boolean | nem | Önálló jogi személyiséggel rendelkezik-e (pl. intézmény fenntartó alatt) | Áht.; Ptk. |
| `costCenterCode` | string | nem | Költséghely-kód (pénzügyi rendszer felé) | Kontrolling |
| `budgetCode` | string | nem | Költségvetési fejezet / alcím kód (költségvetési szerveknél) | Áht. |
| `headPositionId` | UUID (FK) | nem | A szervezeti egység vezetőjének munkakör-azonosítója | SZMSZ |
| `headPersonId` | UUID (FK) | nem | A szervezeti egység aktuális vezetőjének Person azonosítója | Számított az Employment-ből |
| `effectiveFrom` | date | igen | Szervezeti egység érvényessége kezdete | – |
| `effectiveTo` | date | nem | Szervezeti egység érvényessége vége (null = aktuális) | – |
| `isActive` | boolean | igen | Aktív-e | – |

---

### 3.4. Jogviszony-specifikus attribútumok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `defaultEmploymentType` | enum | nem | Az egységben jellemző jogviszony-típus (tájékoztató, nem kényszer) | – |
| `applicableCollectiveAgreement` | string | nem | Ha az adott egységre eltérő kollektív szerződés vonatkozik | Mt. 276. § – üzemi szintű KSZ |
| `employerRightsExerciser` | string | nem | Munkáltatói jogkör gyakorlójának megnevezése / beosztása | Mt. 20. §; Kit. 28. §; Kttv.; Kjt. |
| `employerRightsPersonId` | UUID (FK) | nem | Munkáltatói jogkör tényleges gyakorlója (Person) | Mt. 20. § – a munkáltatói jogkör átruházható |

**Megjegyzések:**
- A **munkáltatói jogkör gyakorlásának** nyilvántartása kritikus: a közszférában pontosan szabályozott, hogy ki gyakorolhatja a munkáltatói jogkört (kinevezés, felmentés, fegyelmi stb.) és ki az, akire ez átruházható. A Kit., Kttv., Küt. és Kjt. külön-külön szabályozza a delegálás kereteit.
- Az Mt. 20. § szerint a munkáltatói jogokat a munkáltató képviselője gyakorolja, de ez átruházható. Az HRMS-nek nyilván kell tartania, hogy egy adott szervezeti egységben ki a munkáltatói jogkör gyakorlója, mert ez határozza meg pl. ki írhatja alá a munkaszerződést, ki kezdeményezhet felmondást.

---

### 3.5. Audit mezők

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `createdAt` | datetime | igen | Rekord létrehozása |
| `createdBy` | string | igen | Létrehozó |
| `updatedAt` | datetime | igen | Utolsó módosítás |
| `updatedBy` | string | igen | Módosító |

---

## 4. Telephely (Site) – opcionális alentitás

Nagyobb szervezeteknél, ahol a fizikai munkavégzési helyek önálló nyilvántartást igényelnek (több telephely, fióktelep), érdemes a telephelyet önálló entitásként kezelni az OrgUnit mellett.

### 4.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `organizationId` | UUID (FK) | igen | Szervezet | Technikai |
| `name` | string | igen | Telephely neve | Ctv. |
| `siteType` | enum | igen | Típus: `HEADQUARTERS` (székhely), `BRANCH` (fióktelep), `SITE` (telephely), `REMOTE_OFFICE` (kirendeltség) | Ctv. |
| `address` | Address | igen | Telephely címe | Ctv.; Art. |
| `phone` | string | nem | Telefonszám | – |
| `isRegistered` | boolean | igen | Cégjogi értelemben bejegyzett telephely/fióktelep-e | Ctv. |
| `taxRegistrationSiteId` | string | nem | NAV telephely-azonosító | Art. |
| `effectiveFrom` | date | igen | Érvényesség kezdete | – |
| `effectiveTo` | date | nem | Érvényesség vége | – |
| `isActive` | boolean | igen | Aktív-e | – |

**Megjegyzések:**
- A NAV felé a **T1041 bejelentésen** a munkavégzés helyét kell megadni – ez a telephely címe.
- Az Mt. 45. § szerinti „munkavégzés helye" lehet egy konkrét telephely, de lehet „változó munkavégzési hely" is.
- Költségvetési szerveknél a székhelyen kívüli **feladatellátási helyek** nyilvántartása szükséges.

---

## 5. Szervezeti hierarchia kezelése

### 5.1. Hierarchia modell

```
Organization (jogi személy – munkáltató)
└── OrgUnit (gyökér – "Cég")
    ├── OrgUnit (Divízió / Igazgatóság)
    │   ├── OrgUnit (Főosztály)
    │   │   ├── OrgUnit (Osztály)
    │   │   │   └── OrgUnit (Csoport)
    │   │   └── OrgUnit (Osztály)
    │   └── OrgUnit (Főosztály)
    └── OrgUnit (Intézmény – ha fenntartói struktúra)
        ├── OrgUnit (Tagintézmény)
        └── OrgUnit (Tagintézmény)
```

### 5.2. Tipikus szektorspecifikus hierarchiák

**Versenyszféra (nagyvállalat):**
```
Holding Zrt.
└── Cégcsoport
    ├── Termelő Kft.
    │   ├── Gyár 1 (telephely)
    │   │   ├── Termelési osztály
    │   │   └── Minőségbiztosítás
    │   └── Gyár 2 (telephely)
    └── Szolgáltató Kft.
        ├── Ügyfélszolgálat
        └── IT
```

**Önkormányzat + intézmények:**
```
Város Önkormányzata (Organization: LOCAL_GOV)
├── Polgármesteri Hivatal (OrgUnit: INSTITUTION → Kttv.)
│   ├── Jegyzői Iroda
│   ├── Pénzügyi Osztály
│   └── Hatósági Osztály
├── Napköziotthonos Óvoda (Organization: PUBLIC_EDUCATION_STATE → Púétv.)
│   ├── Központi telephely
│   └── Tagóvoda
├── Szociális Központ (Organization: SOCIAL_SERVICES → Kjt.)
│   ├── Idősek Otthona
│   └── Családsegítő Szolgálat
└── Városüzemeltetési Kft. (Organization: STATE_ENTERPRISE → Mt.)
    ├── Zöldfelület-kezelés
    └── Közvilágítás
```

**Tankerületi központ + iskolák:**
```
XY Tankerületi Központ (Organization: CENTRAL_GOV → Kit./Púétv.)
├── Központi Igazgatás (Kit.)
├── Általános Iskola 1 (OrgUnit: INSTITUTION → Púétv.)
│   ├── Alsó tagozat
│   └── Felső tagozat
├── Általános Iskola 2 (OrgUnit: INSTITUTION → Púétv.)
└── Gimnázium (OrgUnit: INSTITUTION → Púétv.)
```

**Kórház:**
```
XY Kórház (Organization: HEALTHCARE_STATE → Eszjtv.)
├── Igazgatóság
│   ├── Orvosigazgató
│   └── Ápolási Igazgató
├── Belgyógyászati Osztály (OrgUnit: WARD)
├── Sebészeti Osztály (OrgUnit: WARD)
├── Sürgősségi Betegellátó Osztály (OrgUnit: WARD)
├── Labordiagnosztika
└── Gazdasági Igazgatóság
```

### 5.3. Fenntartó – intézmény kapcsolat

| Minta | Modellezés | Magyarázat |
|---|---|---|
| **Egy jogi személy, több telephely** | Egy Organization + több OrgUnit (SITE típusú) | Pl. egy Kft. több irodával |
| **Fenntartó + önálló intézmény** | Két Organization, parentOrganizationId kapcsolattal | Pl. önkormányzat + önálló óvoda (ha önálló jogi személy) |
| **Fenntartó + nem önálló intézmény** | Egy Organization + OrgUnit (INSTITUTION típusú) | Pl. tankerületi központ + az alá tartozó iskolák |
| **Holding struktúra** | Több Organization, parentOrganizationId láncolattal | Cégcsoport |

---

## 6. Historikus adatkezelés

### 6.1. Szervezeti változások nyilvántartása (OrgChangeEvent)

| Property | Típus | Leírás |
|---|---|---|
| `id` | UUID | PK |
| `eventType` | enum | Változás típusa (lásd alább) |
| `effectiveDate` | date | Hatályba lépés dátuma |
| `affectedOrgUnitIds` | array[UUID] | Érintett szervezeti egységek |
| `description` | string | Változás leírása |
| `legalBasis` | string | Jogalap (pl. közgyűlési határozat, fenntartói döntés, Korm. rendelet) |
| `createdAt` | datetime | Rögzítés dátuma |
| `createdBy` | string | Rögzítő |

**Változástípusok:**

| Kód | Megnevezés | Hatás |
|---|---|---|
| `CREATION` | Új szervezeti egység létrehozása | Új OrgUnit rekord |
| `DISSOLUTION` | Szervezeti egység megszüntetése | OrgUnit `effectiveTo` beállítása; érintett Employment-ek átvezetése |
| `MERGE` | Összevonás | Több OrgUnit → egy új; Employment-ek átvezetése |
| `SPLIT` | Szétválás | Egy OrgUnit → több új |
| `RENAME` | Átnevezés | OrgUnit `name` változás (historikus) |
| `REORGANIZATION` | Átszervezés (hierarchia-változás) | OrgUnit `parentOrgUnitId` változás |
| `TRANSFER` | Fenntartóváltás / szervezet átadás | Organization szintű változás; jogviszony-átalakulás lehetősége |
| `LEGAL_SUCCESSION` | Jogutódlás | Organization megszűnés + új Organization létrehozás; Mt. 36. § szerinti jogviszony-folytonosság |

### 6.2. OrgUnit verzionálás

Az OrgUnit rekordok `effectiveFrom` / `effectiveTo` mezőkkel verzionálhatók. Egy átszervezésnél:
1. A régi OrgUnit lezáródik (`effectiveTo` = átszervezés előtti nap)
2. Új OrgUnit jön létre (`effectiveFrom` = átszervezés napja)
3. Az Employment rekordok `orgUnitId`-je az új egységre mutat (EmploymentHistory rekord keletkezik)

---

## 7. GDPR adatkezelési kategorizáció

| Kategória | Property-k | Jogalap | Megjegyzés |
|---|---|---|---|
| **Szervezeti alapadatok** | name, taxNumber, registrationNumber, address | Nem személyes adat | Jogi személyek adatai nem minősülnek személyes adatnak (GDPR 14. preambulumbekezdés) |
| **Szervezeti egység adatok** | OrgUnit összes property | Nem személyes adat (főszabály) | Kivéve: `headPersonId`, `employerRightsPersonId` – ezek FK-k személyes adatra |
| **Munkáltatói jogkör gyakorló** | employerRightsPersonId, headPersonId | Jogi kötelezettség (6(1)c) + Jogos érdek (6(1)f) | A munkáltatói jogkör gyakorlójának kiléte nem titkos (jogszabályi követelmény) |
| **Közérdekű adatok** | Közszférás szervezeteknél: vezetők neve, beosztása, illetménye | Jogi kötelezettség (Info tv.) | Közzétételi kötelezettség |

**Megjegyzések:**
- A szervezeti adatok döntő többsége **nem személyes adat** → a GDPR nem vonatkozik rájuk.
- Kivétel: ahol természetes személyekre hivatkozunk (szervezeti egység vezetője, munkáltatói jogkör gyakorlója) – ezek személyes adat jellegű FK-k.
- A közszférás szervezeteknél a szervezeti struktúra és a vezetők adatai **közérdekű adatnak** minősülnek (Info tv. III. fejezet) → közzétételi kötelezettség.

---

## 8. Validációs szabályok

| Szabály | Leírás | Megjegyzés |
|---|---|---|
| Adószám formátum | 8 számjegy + kötőjel + 1 ÁFA-kód + kötőjel + 2 számjegyű területi kód | Magyar adószám |
| Ciklikus hierarchia tilalom | Egy OrgUnit nem lehet saját magának őse (parentOrgUnitId láncolatban nem fordulhat elő ciklus) | Technikai validáció |
| Árva egység tilalom | Minden OrgUnit-nak vagy gyökérnek kell lennie, vagy érvényes szülővel kell rendelkeznie | Referenciális integritás |
| Szektor–jogviszony konzisztencia | Employment létrehozásánál az `employmentType` összhangban kell legyen az Organization `sector` és `applicableEmploymentLaws` mezőjével | Üzleti szabály |
| Aktív szervezethez tartozás | Employment csak `isActive` = true Organization-höz és OrgUnit-hoz hozható létre | Üzleti szabály |
| Fenntartói lánc konzisztencia | Ha az Organization `parentOrganizationId`-vel rendelkezik, a szülő Organization-nek léteznie és aktívnak kell lennie | Referenciális integritás |

---

## 9. Integrációs pontok

| Külső rendszer | Irány | Érintett adatok | Cél |
|---|---|---|---|
| **Céginformációs szolgálat (e-cégjegyzék)** | bejövő | name, taxNumber, registrationNumber, legalForm, headquarterAddress | Szervezeti adatok validálása, frissítése |
| **MÁK törzskönyvi nyilvántartás** | bejövő/kimenő | Költségvetési szervek adatai, törzskönyvi azonosító | Költségvetési szervek nyilvántartása |
| **NAV** | kimenő | taxNumber, siteAddress | Munkáltatói bejelentés, T1041 |
| **KIR (Köznevelési Információs Rendszer)** | kétirányú | omIdentifier, intézményi adatok, fenntartói adatok | Köznevelési intézmény-nyilvántartás |
| **NEAK** | kimenő | healthcareProviderId | Egészségügyi szolgáltatói nyilvántartás |
| **KSH (OSAP)** | kimenő | Létszámadatok szervezeti egységenként, TEÁOR, telephely | Statisztikai adatszolgáltatás |
| **Pénzügyi / ERP rendszer** | kétirányú | costCenterCode, budgetCode | Költséghely-szinkronizáció |
| **Active Directory / IAM** | kimenő | OrgUnit hierarchia, szervezeti egység kódok | Jogosultságkezelés szinkronizáció |

---

## 10. Hozzáférési szintek

| Szerep | Organization | OrgUnit | Megjegyzés |
|---|---|---|---|
| **Munkavállaló** | Saját szervezet neve, címe | Saját szervezeti egység és annak felettes lánca | Szervezeti ábra olvasása |
| **Közvetlen vezető** | Saját szervezet | Saját egység + alárendelt egységek | Szervezeti ábra a saját fában |
| **HR ügyintéző** | Hatásköri szervezet(ek) | Hatásköri szervezeti egységek teljes fája | Teljes olvasás; struktúra-módosítás jogosultság külön kezelhető |
| **Szervezetfejlesztő / HR vezető** | Összes | Összes | Struktúra tervezés, átszervezés |
| **Pénzügyi / kontrolling** | Összes (költséghely szinten) | Költséghely-kódok | Aggregált létszám- és bérköltség adatok |
| **Rendszeradminisztrátor** | Összes (konfigurációs szinten) | Összes | Törzsadatok karbantartása, szektor-beállítás |

---

## 11. Kapcsolódó entitások – teljes kép

```
Organization 1 ──── N OrgUnit                 (szervezeti egységek fája)
Organization 1 ──── N Site                     (telephelyek)
Organization 1 ──── N Employment               (jogviszonyok)
Organization ──── Organization                  (fenntartó/anyavállalat kapcsolat – self-ref)

OrgUnit ──── OrgUnit                            (hierarchia – self-ref)
OrgUnit 1 ──── N Employment                     (szervezeti egységbe sorolt jogviszonyok)
OrgUnit 1 ──── N Position                       (szervezeti egységhez rendelt munkakörök)
OrgUnit ──── Person (headPersonId)              (szervezeti egység vezetője)
OrgUnit ──── Person (employerRightsPersonId)    (munkáltatói jogkör gyakorlója)
```

---

## 12. Nyitott kérdések

1. **OrgUnit vs. Site szétválasztás:** Szükséges-e a telephely önálló entitásként, vagy elegendő az OrgUnit `SITE` típussal? Önálló entitás mellett szól: a telephely adószám-kiegészítéssel rendelkezik, NAV felé külön bejelentendő. Ellene szól: egyszerűbb modell.
2. **Költséghely kezelése:** A költséghely (cost center) a szervezeti hierarchia része (OrgUnit), vagy önálló, párhuzamos dimenzió? Sok szervezetben a költséghely-struktúra nem egyezik meg a szervezeti struktúrával.
3. **Mátrix szervezet:** Hogyan kezeljük, ha egy munkavállaló szervezetileg az „A" egységbe tartozik, de funkcionálisan (projekt, feladatkör) a „B" egységben dolgozik? Dupla OrgUnit hivatkozás az Employment-en, vagy külön Assignment entitás?
4. **Jogi személy vs. szervezeti egység határvonal:** Például egy tankerületi központ: maga a tankerületi központ egy jogi személy, az alá tartozó iskolák viszont nem azok. De az iskoláknak mégis van OM-azonosítójuk, saját vezetőjük, önálló költségvetésük. Organization vagy OrgUnit?
5. **Időutazás (point-in-time query):** A szervezeti struktúra historikus lekérdezése (pl. „hogyan nézett ki a szervezet 2024. január 1-jén?") kritikus követelmény? Ha igen, a verzionálás megvalósítási mintáját (bitemporal model, slowly changing dimension) el kell dönteni.
6. **Szervezeti struktúra és hozzáférés-kezelés kapcsolata:** Az HRMS jogosultságkezelése szervezeti egységhez kötött (pl. „HR ügyintéző a Belgyógyászati Osztályra")? Ha igen, az OrgUnit hierarchia egyben a hozzáférési fa is.

---

### 12.1. Javaslatok és válaszok

#### 12.1.1. OrgUnit vs. Site szétválasztás

**Probléma:** A telephely kezelése eltérő dimenziókban történhet:
- **OrgUnit-ként:** Egyszerűbb modell, kevesebb entitás
- **Önálló Site entitásként:** Külön adatmodell a fizikai telephelyeknek (NAV bejelentés, adószám-kiegészítő, munkavédelmi dokumentumok)

**Javaslat: Site önálló entitásként (Extended ajánlás)**

**MVP megoldás:**
Site típusú OrgUnit használata elegendő. A `Site` tábla opcionális.

```typescript
// MVP: Site típusú OrgUnit
OrgUnit {
  id: UUID
  organizationId: UUID FK
  orgUnitType: 'SITE'
  name: 'Gyár 1 - Székesfehérvár'
  address: Address
  taxRegistrationSiteId: string  // NAV telephely-azonosító
}
```

**Extended megoldás (ajánlott):**
Site önálló entitás, cross-reference az OrgUnit-tal. Ez lehetővé teszi, hogy:
- Egy telephely több szervezeti egységnek is otthont adjon
- Egy szervezeti egység több telephelyen is jelen legyen
- A fizikai lokáció és a szervezeti hierarchia függetlenül változhasson

```typescript
Site {
  id: UUID PK
  organizationId: UUID FK
  code: string  // Telephely kód (belső)
  name: string
  siteType: enum (HEADQUARTERS, BRANCH, SITE, REMOTE_OFFICE, WAREHOUSE, FACTORY)
  address: Address
  phone: string
  email: string

  // NAV/adózási adatok
  taxRegistrationSiteId: string  // NAV telephely-azonosító (T1041)
  isRegistered: boolean  // Bejegyzett telephely/fióktelep-e (Ctv.)
  registrationNumber: string  // Cégjegyzékben bejegyzett fióktelep száma

  // Munkavédelmi adatok
  workSafetyCategory: enum
  workSafetyOfficerId: UUID FK → Person
  firstAidOfficerId: UUID FK → Person
  fireSafetyOfficerId: UUID FK → Person

  // Működési adatok
  openingHours: string
  accessControlType: enum (none, card, guard, biometric)
  parkingCapacity: integer
  publicTransportAccess: text

  // Hatályosság
  effectiveFrom: date
  effectiveTo: date
  isActive: boolean

  // Audit
  createdAt: datetime
  createdBy: string
  updatedAt: datetime
  updatedBy: string
}

// Kapcsolótábla: szervezeti egység - telephely
OrgUnitSite {
  id: UUID PK
  orgUnitId: UUID FK → OrgUnit
  siteId: UUID FK → Site
  isPrimary: boolean  // Elsődleges telephely-e az egység számára
  effectiveFrom: date
  effectiveTo: date
}

// Employment bővítés
Employment {
  // ...
  primarySiteId: UUID FK → Site  // Elsődleges munkavégzési hely
  isVariableWorkplace: boolean  // Változó munkavégzési hely (Mt. 45. §)
  allowedSiteIds: array[UUID FK → Site]  // Engedélyezett telephelyek (ha több)
}
```

**Használat példa:**

```sql
-- Lekérdezés: Mely szervezeti egységek működnek a Székesfehérvár telephelyen?
SELECT ou.name, ous.isPrimary
FROM OrgUnit ou
JOIN OrgUnitSite ous ON ou.id = ous.orgUnitId
JOIN Site s ON ous.siteId = s.id
WHERE s.name = 'Gyár 1 - Székesfehérvár'
  AND ous.effectiveTo IS NULL;

-- Lekérdezés: Hány dolgozó dolgozik a Székesfehérvár telephelyen?
SELECT s.name, COUNT(DISTINCT e.personId) AS headcount
FROM Site s
JOIN Employment e ON e.primarySiteId = s.id
WHERE e.endDate IS NULL
  AND s.id = 'site-uuid'
GROUP BY s.id, s.name;
```

**Jogszabályi háttér:**
- **Art. (adózási adatszolgáltatás):** A NAV felé a munkavégzés helyét (telephely címét) kell bejelenteni.
- **Mt. 45. §:** A munkaszerződésben a munkavégzés helyét meg kell határozni. Lehet konkrét cím vagy „változó munkavégzési hely".
- **Ctv. 16-17. §:** A fióktelep és a telephely cégjegyzékbe való bejegyzése kötelező.
- **Mvt. 54. § (5):** Munkavédelmi felelősöket telephelyenként kell kijelölni.

**GDPR:**
A Site entitás nem tartalmaz személyes adatot, kivéve a munkavédelmi felelősök azonosítóit (FK → Person). Ezek kezelése GDPR 6(1)(c) – jogi kötelezettség (Mvt.).

**Implementációs prioritás:**
- **MVP:** OrgUnit `SITE` típussal
- **Extended:** Önálló Site entitás + OrgUnitSite kapcsolótábla (ajánlott nagyobb szervezeteknek, gyártó cégeknek, multi-site szolgáltatóknak)
- **Strategic:** Site-alapú munkarend-automatizálás, telephelyek közötti mozgás nyilvántartása (kirendelés, delegálás)

**Ajánlás:**
Önálló Site entitás bevezetése akkor, ha:
1. A szervezetnek 5+ telephelye van
2. Telephely-szintű reporting szükséges (létszám, bérköltség telephely szerint)
3. NAV, KSH telephely-szintű adatszolgáltatás kötelező
4. Mátrix szervezet: egy szervezeti egység több telephelyen dolgozik

---

#### 12.1.2. Költséghely kezelése

**Probléma:** A költséghely-struktúra gyakran nem egyezik meg a szervezeti struktúrával:
- Szervezetileg egy osztály, de 3 költséghelyre könyvelnek
- Egy költséghely több szervezeti egységet fed le (pl. közös adminisztráció)

**Javaslat: Költséghely párhuzamos dimenzióként (Extended ajánlás)**

**MVP megoldás:**
Költséghely-kód az OrgUnit-on tárolva, 1:1 kapcsolat.

```typescript
OrgUnit {
  // ...
  costCenterCode: string  // Költséghely-kód
}
```

**Extended megoldás (ajánlott):**
Önálló CostCenter entitás + kapcsolótábla az OrgUnit-tal. Egy OrgUnit több költséghelyhez, egy költséghely több OrgUnit-hoz kapcsolódhat.

```typescript
CostCenter {
  id: UUID PK
  organizationId: UUID FK → Organization
  code: string  // Költséghely-kód (egyedi az Organization-ön belül)
  name: string
  description: text

  costCenterType: enum (
    OPERATIONAL,      // Működési
    PRODUCTION,       // Termelési
    ADMINISTRATIVE,   // Adminisztratív
    SERVICE,          // Szolgáltató
    INVESTMENT,       // Beruházási
    PROJECT           // Projekt
  )

  // Hierarchia (költséghelyek is lehetnek hierarchikus struktúrában)
  parentCostCenterId: UUID FK → CostCenter
  level: integer (számított)
  path: string (számított)

  // Pénzügyi adatok
  budgetCode: string  // Költségvetési kód (költségvetési szerveknél)
  responsiblePersonId: UUID FK → Person
  budgetOwnerId: UUID FK → Person
  isActive: boolean

  // Hatályosság
  effectiveFrom: date
  effectiveTo: date

  // ERP integráció
  externalSystemCode: string  // SAP, Oracle, NAV stb.

  // Audit
  createdAt: datetime
  createdBy: string
  updatedAt: datetime
  updatedBy: string
}

// Kapcsolótábla: szervezeti egység - költséghely (N:M)
OrgUnitCostCenter {
  id: UUID PK
  orgUnitId: UUID FK → OrgUnit
  costCenterId: UUID FK → CostCenter
  isPrimary: boolean  // Elsődleges költséghely-e az egység számára
  allocationPercentage: decimal(5,2)  // Költségallokáció % (ha megosztott)
  effectiveFrom: date
  effectiveTo: date
}

// Employment bővítés (ha személy szintű költséghely-választás szükséges)
Employment {
  // ...
  costCenterId: UUID FK → CostCenter  // VAGY
  costCenterCode: string  // Legacy rendszerek integráció esetén
}

// Position bővítés
Position {
  // ...
  defaultCostCenterId: UUID FK → CostCenter
}
```

**Használat példa:**

```sql
-- Lekérdezés: Melyik költséghelyre mennyi bérköltség esik?
SELECT
  cc.code AS costCenterCode,
  cc.name AS costCenterName,
  COUNT(DISTINCT e.personId) AS headcount,
  SUM(c.grossAmount) AS totalGrossSalary
FROM CostCenter cc
LEFT JOIN Employment e ON e.costCenterId = cc.id
LEFT JOIN Compensation c ON c.employmentId = e.id
WHERE e.endDate IS NULL
  AND c.effectiveTo IS NULL
GROUP BY cc.id, cc.code, cc.name
ORDER BY totalGrossSalary DESC;

-- Lekérdezés: Egy szervezeti egység mely költséghelyekre könyvel?
SELECT
  ou.name AS orgUnitName,
  cc.code AS costCenterCode,
  oucc.allocationPercentage,
  oucc.isPrimary
FROM OrgUnit ou
JOIN OrgUnitCostCenter oucc ON ou.id = oucc.orgUnitId
JOIN CostCenter cc ON oucc.costCenterId = cc.id
WHERE oucc.effectiveTo IS NULL
  AND ou.id = 'orgunit-uuid';
```

**Költségallokáció példa (megosztott költséghely):**

```javascript
// Validáció: Egy OrgUnit-hoz tartozó költséghelyek allokációs százalékának összege 100%
function validateCostCenterAllocation(orgUnitId) {
  const allocations = db.query(`
    SELECT SUM(allocationPercentage) AS total
    FROM OrgUnitCostCenter
    WHERE orgUnitId = ? AND effectiveTo IS NULL
  `, [orgUnitId]);

  if (allocations.total !== 100.00 && allocations.total !== null) {
    throw new ValidationError(
      `A költséghely-allokáció összege ${allocations.total}%, nem 100%`
    );
  }
}

// Bérköltség felosztása költséghelyenként
function allocateSalaryToCostCenters(employment, grossSalary) {
  const allocations = db.query(`
    SELECT cc.id, cc.code, oucc.allocationPercentage
    FROM OrgUnitCostCenter oucc
    JOIN CostCenter cc ON oucc.costCenterId = cc.id
    WHERE oucc.orgUnitId = ? AND oucc.effectiveTo IS NULL
  `, [employment.orgUnitId]);

  return allocations.map(alloc => ({
    costCenterId: alloc.id,
    costCenterCode: alloc.code,
    allocatedAmount: grossSalary * (alloc.allocationPercentage / 100)
  }));
}
```

**Jogszabályi háttér:**
- **Áht. VIII. fejezet:** Költségvetési szervek költségvetési gazdálkodása, költséghelyi nyilvántartás.
- **Szt. 15-16. §:** Analitikus nyilvántartások vezetése (költséghely-szintű könyvelés).
- **KSH OSAP jelentések:** Létszámadatok költséghely szerint (ha releváns).

**GDPR:**
A CostCenter entitás nem tartalmaz személyes adatot, kivéve a költséghely-felelős azonosítóját (FK → Person). Ez GDPR 6(1)(f) – jogos érdek alapján kezelhető (pénzügyi elszámolás célja).

**Implementációs prioritás:**
- **MVP:** costCenterCode az OrgUnit-on (string mező)
- **Extended:** Önálló CostCenter entitás + OrgUnitCostCenter kapcsolótábla (ajánlott)
- **Strategic:** Költséghely-hierarchia, automatikus költségallokáció, projekt-alapú költséghelyek (Project entitás integrációja)

**Ajánlás:**
Önálló CostCenter entitás bevezetése akkor, ha:
1. A költséghely-struktúra nem egyezik meg a szervezeti struktúrával
2. Egy OrgUnit több költséghelyre könyvel (megosztott költségek)
3. Költséghely-szintű reporting és kontrolling követelmény
4. ERP rendszerrel történő integráció (SAP CO, Oracle EBS)

---

#### 12.1.3. Mátrix szervezet kezelése

**Probléma:** Mátrix szervezetben egy munkavállaló egyszerre tartozik:
- **Funkcionális vezetőhöz** (szervezeti vonal, pl. „HR osztály")
- **Projekt/termékvonal vezetőhéz** (funkcionális vonal, pl. „CRM projekt")

**Javaslat: Assignment entitás + Employment elsődleges/másodlagos OrgUnit (Extended ajánlás)**

**MVP megoldás:**
Employment-en két OrgUnit FK: `primaryOrgUnitId` és `secondaryOrgUnitId`.

```typescript
Employment {
  // ...
  primaryOrgUnitId: UUID FK → OrgUnit  // Szervezeti vonal
  secondaryOrgUnitId: UUID FK → OrgUnit  // Funkcionális vonal
  primarySupervisorId: UUID FK → Person
  secondarySupervisorId: UUID FK → Person
}
```

**Korlátok:** Maximum 2 szervezeti vonal, nem rugalmas.

**Extended megoldás (ajánlott):**
Employment-en csak az elsődleges OrgUnit, kiegészítve **AssignmentToOrgUnit** entitással a további szervezeti vonalakhoz.

```typescript
Employment {
  // ...
  primaryOrgUnitId: UUID FK → OrgUnit  // Elsődleges szervezeti vonal (funkcionális otthon)
  primarySupervisorId: UUID FK → Person
}

AssignmentToOrgUnit {
  id: UUID PK
  employmentId: UUID FK → Employment
  assignmentType: enum (
    FUNCTIONAL,         // Funkcionális vonal (mátrix)
    PROJECT,            // Projekt-alapú hozzárendelés
    TASK_FORCE,         // Ideiglenes munkacsoport
    MENTORING,          // Mentorálás (mentor szervezeti egysége)
    DOTTED_LINE,        // "Szaggatott vonal" (weak reporting)
    SHARED_RESOURCE     // Megosztott erőforrás (pl. shared service center)
  )

  secondaryOrgUnitId: UUID FK → OrgUnit
  secondarySupervisorId: UUID FK → Person  // Funkcionális/projekt vezető

  // Munkaidő-allokáció
  allocationPercentage: decimal(5,2)  // Hány %-ban dolgozik ezen a vonalon (pl. 40%)

  // Reporting viszony
  reportingRelationshipType: enum (
    DIRECT,             // Közvetlen beszámolási viszony
    INDIRECT,           // Közvetett
    MATRIX,             // Mátrix (kettős beszámolás)
    CONSULTATIVE        // Tanácsadói (nincs közvetlen beszámolás)
  )

  // Hatályosság
  effectiveFrom: date
  effectiveTo: date

  // Jóváhagyások (pl. szabadság, távollét)
  requiresApproval: boolean  // Szükséges-e a másodlagos vezető jóváhagyása is?
  approvalType: enum (any_supervisor, all_supervisors, primary_only, secondary_only)

  // Audit
  createdAt: datetime
  createdBy: string
  updatedAt: datetime
  updatedBy: string
}
```

**Használat példa:**

```sql
-- Lekérdezés: Kik dolgoznak a CRM projekten?
SELECT
  p.name AS employeeName,
  ou_primary.name AS functionalUnit,
  ou_secondary.name AS projectUnit,
  a.allocationPercentage AS projectAllocation
FROM Employment e
JOIN Person p ON e.personId = p.id
JOIN OrgUnit ou_primary ON e.primaryOrgUnitId = ou_primary.id
JOIN AssignmentToOrgUnit a ON a.employmentId = e.id
JOIN OrgUnit ou_secondary ON a.secondaryOrgUnitId = ou_secondary.id
WHERE ou_secondary.name = 'CRM projekt'
  AND a.effectiveTo IS NULL
  AND e.endDate IS NULL;

-- Lekérdezés: Egy munkavállaló melyik szervezeti vonalakban dolgozik?
SELECT
  CASE WHEN a.id IS NULL THEN 'PRIMARY' ELSE a.assignmentType END AS lineType,
  CASE WHEN a.id IS NULL THEN ou_primary.name ELSE ou_secondary.name END AS orgUnitName,
  CASE WHEN a.id IS NULL THEN 100.00 ELSE a.allocationPercentage END AS allocation
FROM Employment e
JOIN OrgUnit ou_primary ON e.primaryOrgUnitId = ou_primary.id
LEFT JOIN AssignmentToOrgUnit a ON a.employmentId = e.id AND a.effectiveTo IS NULL
LEFT JOIN OrgUnit ou_secondary ON a.secondaryOrgUnitId = ou_secondary.id
WHERE e.personId = 'person-uuid'
  AND e.endDate IS NULL;
```

**Workflow példa: Szabadság-jóváhagyás mátrix szervezetben**

```javascript
async function getLeaveApprovers(leaveRequest, employment) {
  const approvers = [];

  // Elsődleges vezető (mindig szükséges)
  const primarySupervisor = await Person.findById(employment.primarySupervisorId);
  approvers.push({
    personId: primarySupervisor.id,
    role: 'PRIMARY_SUPERVISOR',
    orgUnitId: employment.primaryOrgUnitId,
    required: true
  });

  // Másodlagos szervezeti vonalak (AssignmentToOrgUnit)
  const assignments = await AssignmentToOrgUnit.findAll({
    where: {
      employmentId: employment.id,
      effectiveTo: null,
      requiresApproval: true
    }
  });

  for (const assignment of assignments) {
    if (assignment.approvalType === 'all_supervisors' ||
        assignment.approvalType === 'secondary_only') {
      const secondarySupervisor = await Person.findById(assignment.secondarySupervisorId);
      approvers.push({
        personId: secondarySupervisor.id,
        role: 'SECONDARY_SUPERVISOR',
        orgUnitId: assignment.secondaryOrgUnitId,
        assignmentType: assignment.assignmentType,
        required: assignment.approvalType === 'all_supervisors'
      });
    }
  }

  return approvers;
}

// Validáció: Időallokáció összege max. 100%
function validateTotalAllocation(employmentId) {
  const assignments = db.query(`
    SELECT SUM(allocationPercentage) AS totalSecondary
    FROM AssignmentToOrgUnit
    WHERE employmentId = ? AND effectiveTo IS NULL
  `, [employmentId]);

  // Elsődleges vonal implicit allokációja
  const primaryAllocation = 100.00 - (assignments.totalSecondary || 0);

  if (primaryAllocation < 0) {
    throw new ValidationError(
      `Túlallokáció: a másodlagos vonalak összege ${assignments.totalSecondary}%, ami meghaladja a 100%-ot`
    );
  }

  return { primaryAllocation, secondaryAllocation: assignments.totalSecondary };
}
```

**Strategic megoldás:**
**Project** entitás bevezetése, AssignmentToProject kapcsolótáblával. Projektek szervezeti egységektől függetlenül kezelhetők.

```typescript
Project {
  id: UUID PK
  organizationId: UUID FK → Organization
  code: string
  name: string
  projectType: enum (INTERNAL, CLIENT, PRODUCT, INITIATIVE)
  projectManagerId: UUID FK → Person
  startDate: date
  endDate: date
  status: enum (planning, active, on_hold, completed, cancelled)
  budgetCode: string
  costCenterId: UUID FK → CostCenter
}

AssignmentToProject {
  id: UUID PK
  employmentId: UUID FK → Employment
  projectId: UUID FK → Project
  role: string  // Szerepkör a projekten (pl. "Developer", "Analyst")
  allocationPercentage: decimal(5,2)
  effectiveFrom: date
  effectiveTo: date
}
```

**Jogszabályi háttér:**
- **Mt.:** Nincs közvetlen szabályozás a mátrix szervezetre, de a munkaköri leírásban (Mt. 46. §) rögzíteni kell a beszámolási viszonyt.
- **Közszféra:** Kit., Kttv., Kjt. – a munkáltatói jogkör gyakorlója egyértelműen meghatározott, mátrix viszony esetén is csak egy vezető gyakorolhatja a munkáltatói jogköröket (kinevezés, fegyelmi stb.).

**GDPR:**
AssignmentToOrgUnit tartalmaz személyes adatot (employmentId, secondarySupervisorId). Jogalap: GDPR 6(1)(b) – szerződés teljesítése (munkaszerződés alapján a beszámolási viszony rögzítése szükséges).

**Implementációs prioritás:**
- **MVP:** `secondaryOrgUnitId` az Employment-en
- **Extended:** AssignmentToOrgUnit entitás (ajánlott mátrix szervezeteknek)
- **Strategic:** Project entitás + AssignmentToProject + időkövetés projekt szinten

**Ajánlás:**
AssignmentToOrgUnit bevezetése akkor, ha:
1. Mátrix szervezet (funkcionális + termékvonal / regionális + funkcionális)
2. Projekt-alapú munkavégzés (dolgozók projektek között mozognak)
3. Megosztott erőforrások (shared service center)
4. Kettős beszámolási viszony (functional + project manager)

---

#### 12.1.4. Jogi személy vs. szervezeti egység határvonal

**Probléma:** Speciális esetek, ahol a szervezeti egység "majdnem" jogi személy:
- **Tankerületi központ alá tartozó iskolák:** Saját OM-azonosító, saját igazgató, saját költségvetés, DE nem önálló jogi személy
- **Önkormányzati intézmények:** Óvoda, szociális központ – lehetnek önálló jogi személyek VAGY a fenntartó szervezeti egységei

**Javaslat: `isLegallyIndependent` flag az OrgUnit-on + egyértelmű döntési mátrix (Extended ajánlás)**

**MVP megoldás:**
Minden jogi személy = Organization. Szervezeti egységek = OrgUnit.

**Extended megoldás (ajánlott):**
OrgUnit bővítés jogi személyiség-jellemzőkkel, egyértelmű döntési logikával.

```typescript
OrgUnit {
  // ...meglévő mezők...

  // Jogi személyiség jellemzők
  isLegallyIndependent: boolean  // Önálló jogi személyiség-e
  legalEntityType: enum (
    NONE,                    // Nem jogi személy
    BUDGETARY_INSTITUTION,   // Költségvetési szerv (Áht. 9. §)
    SUBSIDIARY,              // Leányvállalat (cégjegyzékben bejegyzett)
    BRANCH,                  // Fióktelep (részben önálló)
    PUBLIC_FOUNDATION        // Közalapítvány
  )

  // Ha önálló jogi személy, de a fenntartó Organization alatt van:
  taxNumber: string(11)          // Saját adószám (ha van)
  registrationNumber: string     // Saját törzskönyvi azonosító (ha van)
  omIdentifier: string           // OM-azonosító (köznevelés)
  healthcareProviderId: string   // NEAK-kód (egészségügy)

  // Pénzügyi önállóság
  hasOwnBudget: boolean          // Saját költségvetéssel rendelkezik-e
  budgetCode: string             // Költségvetési kód (ha van)

  // Vezetői önállóság
  headPositionId: UUID FK → Position
  headPersonId: UUID FK → Person
  hasEmployerRights: boolean     // Gyakorolhatja-e a munkáltatói jogköröket

  // Kapcsolódó Organization (ha önálló jogi személy, de megjelenítjük OrgUnit-ként is)
  linkedOrganizationId: UUID FK → Organization
}
```

**Döntési mátrix: Organization vs. OrgUnit**

| Kérdés | Organization | OrgUnit (isLegallyIndependent=true) | OrgUnit (isLegallyIndependent=false) |
|--------|--------------|-------------------------------------|--------------------------------------|
| Van saját adószáma? | Igen | Igen (ritka) | Nem |
| Cégjegyzékben/törzskönyvben bejegyzett? | Igen | Igen | Nem |
| Fenntartói viszonyban van más szervezettel? | Lehet (parentOrganizationId) | Igen (parentOrgUnitId → fenntartó Organization) | Igen |
| Gyakorolhatja a munkáltatói jogköröket? | Igen | Igen (korlátozott) | Nem (a fenntartó gyakorolja) |
| NAV T1041 bejelentés | Saját név alatt | Saját név alatt (ha önálló munkáltató) | Fenntartó neve alatt |
| Példa | Tankerületi központ; Önkormányzat; Kft. | Önkormányzati intézmény (önálló jogi személy); egyházi iskola (ha önálló) | Tankerületi központ alá tartozó iskola; polgármesteri hivatal osztályai |

**Használati példák:**

**1. Tankerületi központ + iskolák**

```typescript
// Tankerületi központ = Organization
Organization {
  id: 'org-tankerulet-001'
  name: 'Budapest XV. Tankerületi Központ'
  sector: CENTRAL_GOV
  legalForm: KOLTSEGVETESI_SZERV
  taxNumber: '12345678-2-42'
}

// Iskola 1 = OrgUnit (NEM önálló jogi személy, de saját OM-azonosítója van)
OrgUnit {
  id: 'orgunit-iskola-001'
  organizationId: 'org-tankerulet-001'
  parentOrgUnitId: null  // Közvetlenül a tankerületi központ alá tartozik
  orgUnitType: INSTITUTION
  name: 'XV. kerületi Általános Iskola'

  isLegallyIndependent: false
  legalEntityType: NONE
  omIdentifier: '031234'  // Saját OM-azonosító VAN
  hasOwnBudget: true      // Saját költségvetés VAN (de a tankerület keretében)
  budgetCode: '001-ALT-ISK-01'

  headPositionId: 'pos-igazgato-001'
  headPersonId: 'person-igazgato-001'
  hasEmployerRights: false  // Igazgató NEM gyakorolhatja a munkáltatói jogköröket (a tankerületi központ vezetője gyakorolja)
}

// Employment a tankerületi központhoz tartozik, de az iskolába van beosztva
Employment {
  id: 'emp-001'
  personId: 'person-pedagogus-001'
  organizationId: 'org-tankerulet-001'  // Munkáltató = tankerületi központ
  orgUnitId: 'orgunit-iskola-001'        // Munkavégzés helye = iskola
  employmentType: KOZNEVELESI
}
```

**2. Önkormányzat + önálló jogi személyiségű óvoda**

```typescript
// Önkormányzat = Organization
Organization {
  id: 'org-onkormanyzat-001'
  name: 'Miskolc Megyei Jogú Város Önkormányzata'
  sector: LOCAL_GOV
  legalForm: ONKORMANYZAT
}

// Óvoda = Külön Organization (önálló jogi személy)
Organization {
  id: 'org-ovoda-001'
  name: 'Miskolc Napköziotthonos Óvoda'
  sector: PUBLIC_EDUCATION_STATE
  legalForm: KOLTSEGVETESI_SZERV
  taxNumber: '98765432-2-05'
  omIdentifier: '031567'

  parentOrganizationId: 'org-onkormanyzat-001'
  parentRelationType: MAINTAINER
  maintainerType: LOCAL_GOV
}

// Employment az óvodához tartozik
Employment {
  id: 'emp-002'
  personId: 'person-ovono-001'
  organizationId: 'org-ovoda-001'  // Munkáltató = óvoda (önálló jogi személy)
  employmentType: KOZNEVELESI
}
```

**3. Önkormányzat + nem önálló szociális intézmény**

```typescript
// Önkormányzat = Organization
Organization {
  id: 'org-onkormanyzat-002'
  name: 'Eger Város Önkormányzata'
  sector: LOCAL_GOV
}

// Szociális központ = OrgUnit (NEM önálló jogi személy)
OrgUnit {
  id: 'orgunit-szoc-001'
  organizationId: 'org-onkormanyzat-002'
  orgUnitType: INSTITUTION
  name: 'Egri Szociális Központ'

  isLegallyIndependent: false
  legalEntityType: NONE
  hasOwnBudget: true
  budgetCode: '002-SZOC-01'

  headPositionId: 'pos-igazgato-002'
  hasEmployerRights: false  // A polgármester/jegyző gyakorolja
}

// Employment az önkormányzathoz tartozik
Employment {
  id: 'emp-003'
  personId: 'person-gondozo-001'
  organizationId: 'org-onkormanyzat-002'  // Munkáltató = önkormányzat
  orgUnitId: 'orgunit-szoc-001'           // Munkavégzés helye = szociális központ
  employmentType: KOZALKALMAZOTT
}
```

**Döntési logika (kód):**

```javascript
function determineOrganizationVsOrgUnit(entity) {
  // 1. Van saját adószáma? → Organization
  if (entity.hasTaxNumber) {
    return { type: 'Organization', reason: 'Saját adószám' };
  }

  // 2. Cégjegyzékben/törzskönyvben bejegyzett önálló jogi személy? → Organization
  if (entity.isRegisteredLegalEntity) {
    return { type: 'Organization', reason: 'Cégjegyzékben bejegyzett' };
  }

  // 3. Gyakorolja a munkáltatói jogköröket TELJES KÖRŰEN? → Organization
  if (entity.hasFullEmployerRights) {
    return { type: 'Organization', reason: 'Teljes munkáltatói jogkör' };
  }

  // 4. Van fenntartója ÉS a fenntartó gyakorolja a munkáltatói jogköröket? → OrgUnit
  if (entity.hasMaintainer && !entity.hasFullEmployerRights) {
    return {
      type: 'OrgUnit',
      isLegallyIndependent: entity.hasOwnBudget || entity.hasOwnIdentifier,
      reason: 'Fenntartói viszony, korlátozott önállóság'
    };
  }

  // 5. Default: OrgUnit
  return { type: 'OrgUnit', isLegallyIndependent: false, reason: 'Szervezeti egység' };
}
```

**Jogszabályi háttér:**
- **Áht. 9. §:** Költségvetési szerv jogi személyiség (önálló jogi személyiségű vs. nem önálló).
- **Nkt. 7. § (1):** Köznevelési intézmény lehet önálló jogi személyiségű vagy sem.
- **Ptk. 3:1-3:3. §:** Jogi személy fogalma.

**GDPR:**
Nem releváns (jogi személyek adatai).

**Implementációs prioritás:**
- **MVP:** Minden jogi személy = Organization
- **Extended:** OrgUnit.isLegallyIndependent + döntési logika (ajánlott)
- **Strategic:** OrgUnit.linkedOrganizationId – ha egy intézmény egyszerre jelenik meg Organization-ként ÉS a fenntartó OrgUnit hierarchiájában

**Ajánlás:**
Extended modell alkalmazása közszférás szervezeteknél (köznevelés, egészségügy, önkormányzat), ahol a fenntartói viszonyok és a részleges önállóság gyakori.

---

#### 12.1.5. Időutazás (point-in-time query) – Historikus lekérdezések

**Probléma:** A szervezeti struktúra idővel változik (átszervezés, összevonás, szétválás). Szükség van arra, hogy:
- Lekérdezzük, hogy „hogyan nézett ki a szervezet 2024. január 1-jén?"
- Egy dolgozó melyik szervezeti egységhez tartozott 2023 júniusában?
- Audit célra nyomon követhető legyen, hogy egy adott időpontban ki volt a szervezeti egység vezetője.

**Javaslat: Slowly Changing Dimension Type 2 (SCD-2) + Bitemporal modell opcionálisan (Extended ajánlás)**

**MVP megoldás:**
OrgUnit `effectiveFrom` / `effectiveTo` mezők. Egy átszervezésnél a régi rekord lezárul, új rekord jön létre.

```typescript
OrgUnit {
  // ...
  effectiveFrom: date  // Érvényesség kezdete
  effectiveTo: date    // Érvényesség vége (null = aktuális)
}

// Példa: Osztály átnevezés
// Régi rekord
OrgUnit {
  id: 'orgunit-001-v1'
  name: 'HR osztály'
  effectiveFrom: '2020-01-01'
  effectiveTo: '2024-12-31'  // Lezáródik
}

// Új rekord (új ID!)
OrgUnit {
  id: 'orgunit-001-v2'
  name: 'Humán Erőforrás Igazgatóság'
  effectiveFrom: '2025-01-01'
  effectiveTo: null
}
```

**Korlátok:**
- A régi és új rekord különböző ID-val rendelkezik → FK-k (Employment.orgUnitId) frissítése szükséges
- Nehéz követni a változások előzményeit

**Extended megoldás (ajánlott): SCD-2 + Business Key + Version Chain**

```typescript
OrgUnit {
  id: UUID PK  // Technikai kulcs (változik verzióváltáskor)
  businessKey: string UNIQUE  // Üzleti kulcs (ÁLLANDÓ, pl. "HR-OSZ-001")
  version: integer  // Verziószám

  // Minden más mező...
  name: string
  parentOrgUnitId: UUID FK → OrgUnit
  // ...

  // Hatályosság (Valid Time – "mikor volt ez igaz a való világban")
  effectiveFrom: date
  effectiveTo: date

  // Verzió-lánc
  previousVersionId: UUID FK → OrgUnit  // Előző verzió
  nextVersionId: UUID FK → OrgUnit      // Következő verzió (null, ha ez az aktuális)

  // Audit (Transaction Time – "mikor rögzítettük az adatbázisban")
  createdAt: datetime
  createdBy: string
  updatedAt: datetime
  updatedBy: string
}

// Employment FK-k a businessKey-re mutatnak (vagy verzió-független ID-re)
Employment {
  // ...
  orgUnitBusinessKey: string FK → OrgUnit.businessKey  // VAGY
  currentOrgUnitId: UUID FK → OrgUnit (computed – a current effectiveTo IS NULL rekord)
}
```

**Point-in-Time lekérdezés:**

```sql
-- Lekérdezés: Szervezeti struktúra 2024. január 1-jén
SELECT
  ou.businessKey,
  ou.version,
  ou.name,
  ou_parent.name AS parentName
FROM OrgUnit ou
LEFT JOIN OrgUnit ou_parent ON ou.parentOrgUnitId = ou_parent.id
WHERE ou.effectiveFrom <= '2024-01-01'
  AND (ou.effectiveTo IS NULL OR ou.effectiveTo >= '2024-01-01');

-- Lekérdezés: Egy munkavállaló melyik szervezeti egységhez tartozott 2023. június 15-én?
SELECT
  p.name AS employeeName,
  ou.name AS orgUnitName,
  ou.effectiveFrom,
  ou.effectiveTo
FROM Employment e
JOIN Person p ON e.personId = p.id
JOIN OrgUnit ou ON ou.businessKey = (
  -- Lekérjük a businessKey-t az aktuális Employment-ből
  SELECT ou_current.businessKey
  FROM OrgUnit ou_current
  WHERE ou_current.id = e.orgUnitId
)
WHERE p.id = 'person-uuid'
  AND e.startDate <= '2023-06-15'
  AND (e.endDate IS NULL OR e.endDate >= '2023-06-15')
  AND ou.effectiveFrom <= '2023-06-15'
  AND (ou.effectiveTo IS NULL OR ou.effectiveTo >= '2023-06-15');
```

**Strategic megoldás: Bitemporal modell (Valid Time + Transaction Time)**

Ha szükséges, hogy ne csak azt tudjuk, „mikor volt ez igaz", hanem azt is, hogy „mikor rögzítettük az adatbázisban" (audit célra):

```typescript
OrgUnit {
  // ...

  // Valid Time (üzleti hatályosság – "mikor volt ez igaz a való világban")
  validFrom: date
  validTo: date

  // Transaction Time (technikai hatályosság – "mikor volt ez igaz az adatbázisban")
  transactionFrom: datetime
  transactionTo: datetime  // Null = aktuális adatbázis-verzió
}

// Példa: 2024. december 20-án rögzítünk egy 2025. január 1-jei átszervezést
OrgUnit {
  id: 'orgunit-001-v2'
  businessKey: 'HR-OSZ-001'
  version: 2
  name: 'Humán Erőforrás Igazgatóság'

  validFrom: '2025-01-01'      // Üzleti hatályosság: 2025. jan. 1-től
  validTo: null

  transactionFrom: '2024-12-20 10:30:00'  // Adatbázisban: 2024. dec. 20-án rögzítve
  transactionTo: null
}
```

**Bitemporal lekérdezés típusok:**

```sql
-- 1. Aktuális állapot (AS-IS)
SELECT * FROM OrgUnit
WHERE validTo IS NULL AND transactionTo IS NULL;

-- 2. Point-in-Time (hogyan nézett ki 2024. jan. 1-jén, a mai tudásunk szerint)
SELECT * FROM OrgUnit
WHERE validFrom <= '2024-01-01' AND (validTo IS NULL OR validTo > '2024-01-01')
  AND transactionTo IS NULL;

-- 3. Historical Point-in-Time (hogyan láttuk a világot 2024. jan. 1-jén, 2024. jan. 1-jei adatbázis szerint)
SELECT * FROM OrgUnit
WHERE validFrom <= '2024-01-01' AND (validTo IS NULL OR validTo > '2024-01-01')
  AND transactionFrom <= '2024-01-01 23:59:59' AND (transactionTo IS NULL OR transactionTo > '2024-01-01 23:59:59');

-- 4. Audit trail (egy rekord összes verziója)
SELECT * FROM OrgUnit
WHERE businessKey = 'HR-OSZ-001'
ORDER BY version DESC;
```

**OrgChangeEvent kiegészítés:**

```typescript
OrgChangeEvent {
  // ...meglévő mezők...

  // Hatályosság vs. végrehajtás
  effectiveDate: date           // Mikor lép hatályba (Valid Time)
  executedDate: date            // Mikor hajtották végre (Transaction Time)
  announcedDate: date           // Mikor jelentették be

  // Előzmény-követés
  affectedOrgUnitBusinessKeys: array[string]
  createdOrgUnitBusinessKeys: array[string]
  terminatedOrgUnitBusinessKeys: array[string]
}
```

**Jogszabályi háttér:**
- **Számviteli törvény (Szt.) 166. § (2):** Számviteli bizonylatok, nyilvántartások 8 évig megőrzendők.
- **Áht.:** Költségvetési szervek átszervezéséről kormányhatározat, önkormányzati határozat – hivatalos dokumentáció.
- **Mt. / Kit. / Kttv.:** Jogviszony-dokumentumok megőrzési ideje 5-8 év, ezért a szervezeti struktúra is ugyanennyi ideig visszakereshető kell legyen.

**GDPR:**
Nem releváns (szervezeti adatok nem személyes adatok).

**Implementációs prioritás:**
- **MVP:** `effectiveFrom` / `effectiveTo` + új rekord verzióváltáskor
- **Extended:** BusinessKey + Version Chain + Point-in-Time lekérdezések (ajánlott)
- **Strategic:** Bitemporal modell (válidi + transaction time) – audit-kritikus környezetben (pénzügyi, közszféra)

**Ajánlás:**
Extended modell (SCD-2 + BusinessKey) bevezetése akkor, ha:
1. Gyakori átszervezések (évente 2+)
2. Audit követelmény (pontosan tudni kell, mikor mi volt az igazság)
3. Reporting történelmi adatokról (pl. "2023-ban mennyi volt a bérköltség az akkori szervezeti struktúra szerint")
4. Közszféra, ahol átszervezéseket jogi dokumentumokkal kell alátámasztani

---

#### 12.1.6. Szervezeti struktúra és hozzáférés-kezelés kapcsolata

**Probléma:** Az HRMS-ben a jogosultságkezelés gyakran szervezeti egység alapú:
- „HR ügyintéző láthatja és módosíthatja a Belgyógyászati Osztály dolgozóinak adatait"
- „Osztályvezető láthatja a saját és az alárendelt osztályok dolgozóit"

**Kérdés:** Az OrgUnit hierarchia egyben a jogosultsági fa is, vagy külön AccessControlList (ACL) / Role-Based Access Control (RBAC) szükséges?

**Javaslat: Hibrid modell – RBAC + OrgUnit Scope (Extended ajánlás)**

**MVP megoldás:**
Role-Based Access Control (RBAC) egyszerű szerepkörökkel, nincs szervezeti egységhez kötés.

```typescript
User {
  id: UUID
  email: string
  role: enum (EMPLOYEE, MANAGER, HR_ADMIN, SUPER_ADMIN)
}

// Jogosultság ellenőrzés
if (user.role === 'HR_ADMIN' || user.role === 'SUPER_ADMIN') {
  // Minden adat elérhető
} else if (user.role === 'MANAGER') {
  // Csak a saját szervezeti egység
} else {
  // Csak saját adatok
}
```

**Korlátok:**
- Nem kezeli, hogy egy HR admin csak bizonyos szervezeti egységekért felelős
- Nem kezeli a delegálást (pl. távollét esetén helyettes)

**Extended megoldás (ajánlott): RBAC + OrgUnit Scope + Hierarchikus propagáció**

```typescript
User {
  id: UUID PK
  email: string
  personId: UUID FK → Person  // Kapcsolat a Person entitással
  isActive: boolean
}

Role {
  id: UUID PK
  code: string UNIQUE  // pl. "HR_ADMIN", "MANAGER", "PAYROLL_CLERK"
  name: string
  description: text
  roleType: enum (SYSTEM, FUNCTIONAL, ORGANIZATIONAL)
}

Permission {
  id: UUID PK
  code: string UNIQUE  // pl. "employment.read", "leave.approve", "payroll.process"
  resource: string     // "Employment", "Leave", "Compensation"
  action: enum (CREATE, READ, UPDATE, DELETE, APPROVE, PROCESS, EXPORT)
  description: text
}

RolePermission {
  roleId: UUID FK → Role
  permissionId: UUID FK → Permission
}

// Felhasználó-szerepkör hozzárendelés SCOPE-pal
UserRoleAssignment {
  id: UUID PK
  userId: UUID FK → User
  roleId: UUID FK → Role

  // SCOPE meghatározás
  scopeType: enum (
    GLOBAL,              // Teljes szervezetre érvényes
    ORGANIZATION,        // Adott Organization-re
    ORGUNIT,             // Adott OrgUnit-ra
    ORGUNIT_TREE,        // Adott OrgUnit + alárendelt egységek (hierarchikus)
    PERSON               // Csak adott személyre (pl. mentor, helyettes)
  )

  scopeOrganizationId: UUID FK → Organization
  scopeOrgUnitId: UUID FK → OrgUnit
  scopePersonId: UUID FK → Person

  // Hierarchikus propagáció
  includeSubOrgUnits: boolean  // Az alárendelt szervezeti egységekre is érvényes-e
  depth: integer  // Hány szintig érvényes (null = végtelen)

  // Hatályosság
  effectiveFrom: date
  effectiveTo: date

  // Delegálás (helyettesítés)
  isDelegated: boolean
  delegatedByUserId: UUID FK → User
  delegationReason: text

  // Audit
  createdAt: datetime
  createdBy: string
}
```

**Használat példa:**

```typescript
// 1. HR admin felelős a Belgyógyászati Osztályért + alárendelt egységekért
UserRoleAssignment {
  userId: 'user-hr-001'
  roleId: 'role-hr-admin'
  scopeType: ORGUNIT_TREE
  scopeOrgUnitId: 'orgunit-belgyogyaszat'
  includeSubOrgUnits: true
  depth: null  // Végtelen mélység
  effectiveFrom: '2025-01-01'
  effectiveTo: null
}

// 2. Osztályvezető csak a saját osztályra
UserRoleAssignment {
  userId: 'user-manager-001'
  roleId: 'role-manager'
  scopeType: ORGUNIT
  scopeOrgUnitId: 'orgunit-hr-osztaly'
  includeSubOrgUnits: false
  effectiveFrom: '2024-01-01'
  effectiveTo: null
}

// 3. Bérszámfejtő globális hozzáféréssel
UserRoleAssignment {
  userId: 'user-payroll-001'
  roleId: 'role-payroll-clerk'
  scopeType: GLOBAL
  effectiveFrom: '2023-01-01'
  effectiveTo: null
}

// 4. Delegálás: HR admin szabadságon, helyettes
UserRoleAssignment {
  userId: 'user-hr-002'  // Helyettes
  roleId: 'role-hr-admin'
  scopeType: ORGUNIT_TREE
  scopeOrgUnitId: 'orgunit-belgyogyaszat'
  includeSubOrgUnits: true
  effectiveFrom: '2025-07-01'
  effectiveTo: '2025-07-15'  // 2 hét
  isDelegated: true
  delegatedByUserId: 'user-hr-001'
  delegationReason: 'Éves szabadság'
}
```

**Jogosultság-ellenőrzés kód:**

```javascript
async function checkPermission(userId, permission, resourceId) {
  // 1. User szerepköreinek lekérése
  const userRoles = await UserRoleAssignment.findAll({
    where: {
      userId: userId,
      effectiveFrom: { $lte: new Date() },
      $or: [
        { effectiveTo: null },
        { effectiveTo: { $gte: new Date() } }
      ]
    },
    include: [{ model: Role, include: [Permission] }]
  });

  // 2. Van-e a kért permission valamelyik szerepkörben?
  const hasPermission = userRoles.some(ura =>
    ura.role.permissions.some(p => p.code === permission)
  );

  if (!hasPermission) {
    return { allowed: false, reason: 'Nincs jogosultság a művelethez' };
  }

  // 3. SCOPE ellenőrzés: az adott erőforrás (pl. Employment) beletartozik-e a scope-ba?
  const resource = await Employment.findById(resourceId);

  for (const ura of userRoles) {
    if (ura.scopeType === 'GLOBAL') {
      return { allowed: true, scope: 'GLOBAL' };
    }

    if (ura.scopeType === 'ORGANIZATION') {
      if (resource.organizationId === ura.scopeOrganizationId) {
        return { allowed: true, scope: 'ORGANIZATION' };
      }
    }

    if (ura.scopeType === 'ORGUNIT') {
      if (resource.orgUnitId === ura.scopeOrgUnitId) {
        return { allowed: true, scope: 'ORGUNIT' };
      }
    }

    if (ura.scopeType === 'ORGUNIT_TREE') {
      const isInTree = await isOrgUnitInTree(
        resource.orgUnitId,
        ura.scopeOrgUnitId,
        ura.depth
      );
      if (isInTree) {
        return { allowed: true, scope: 'ORGUNIT_TREE' };
      }
    }

    if (ura.scopeType === 'PERSON') {
      if (resource.personId === ura.scopePersonId) {
        return { allowed: true, scope: 'PERSON' };
      }
    }
  }

  return { allowed: false, reason: 'Erőforrás kívül esik a jogosultsági körön' };
}

// Segédfüggvény: OrgUnit hierarchia ellenőrzés
async function isOrgUnitInTree(childOrgUnitId, parentOrgUnitId, maxDepth) {
  if (childOrgUnitId === parentOrgUnitId) {
    return true;
  }

  let currentOrgUnit = await OrgUnit.findById(childOrgUnitId);
  let depth = 0;

  while (currentOrgUnit && currentOrgUnit.parentOrgUnitId) {
    depth++;
    if (maxDepth !== null && depth > maxDepth) {
      return false;
    }

    if (currentOrgUnit.parentOrgUnitId === parentOrgUnitId) {
      return true;
    }

    currentOrgUnit = await OrgUnit.findById(currentOrgUnit.parentOrgUnitId);
  }

  return false;
}
```

**SQL nézet: User látható employments**

```sql
CREATE VIEW vw_user_accessible_employments AS
SELECT
  u.id AS userId,
  e.id AS employmentId,
  e.personId,
  e.organizationId,
  e.orgUnitId,
  ura.scopeType
FROM User u
JOIN UserRoleAssignment ura ON u.id = ura.userId
JOIN Employment e ON (
  -- GLOBAL scope
  (ura.scopeType = 'GLOBAL') OR

  -- ORGANIZATION scope
  (ura.scopeType = 'ORGANIZATION' AND e.organizationId = ura.scopeOrganizationId) OR

  -- ORGUNIT scope
  (ura.scopeType = 'ORGUNIT' AND e.orgUnitId = ura.scopeOrgUnitId) OR

  -- ORGUNIT_TREE scope (requires recursive CTE or materialized path)
  (ura.scopeType = 'ORGUNIT_TREE' AND e.orgUnitId IN (
    SELECT id FROM OrgUnit WHERE path LIKE CONCAT((SELECT path FROM OrgUnit WHERE id = ura.scopeOrgUnitId), '%')
  )) OR

  -- PERSON scope
  (ura.scopeType = 'PERSON' AND e.personId = ura.scopePersonId)
)
WHERE ura.effectiveFrom <= CURRENT_DATE
  AND (ura.effectiveTo IS NULL OR ura.effectiveTo >= CURRENT_DATE)
  AND e.endDate IS NULL;
```

**Strategic megoldás: Attribute-Based Access Control (ABAC)**

Komplex szabályok definiálása attribútumok alapján (pl. „csak olyan dolgozók láthatók, akiknek a bérszintje < 500k ÉS a szervezeti egység a Termelés alatt van").

```typescript
AccessPolicy {
  id: UUID PK
  name: string
  description: text

  // Szabály (JSON formátum)
  rule: json  // pl. {"and": [{"user.role": "HR_ADMIN"}, {"resource.orgUnit.path": "/prod/%"}]}

  effect: enum (ALLOW, DENY)
  priority: integer
}
```

**Jogszabályi háttér:**
- **GDPR 32. cikk (1) b:** Személyes adatok kezelése során biztosítani kell a jogosultságkezelést („hozzáférés-korlátozás").
- **Info tv.:** Közszférában az adatkezelő adatvédelmi szabályzata rögzíti a hozzáférési jogosultságokat.
- **Mt. / Kit. / Kttv.:** A munkáltatói jogkör gyakorlója láthatja az általa irányított dolgozók adatait.

**GDPR:**
A hozzáférés-kezelés GDPR compliance szempontból kritikus. Az RBAC + OrgUnit Scope biztosítja, hogy:
- Csak az jogosult hozzáférjen az adatokhoz, akinek üzleti indoka van (need-to-know alapelv)
- Audit trail: ki mikor milyen adatokat nézett meg
- Delegálás esetén időkorlátos hozzáférés (effectiveTo)

**Implementációs prioritás:**
- **MVP:** Egyszerű RBAC (User.role enum)
- **Extended:** RBAC + OrgUnit Scope + hierarchikus propagáció (ajánlott)
- **Strategic:** ABAC – attribútum-alapú jogosultságkezelés komplex szabályokkal

**Ajánlás:**
Extended modell (RBAC + OrgUnit Scope) bevezetése, mert:
1. Az HRMS jellegéből adódóan a jogosultságok szervezeti egység alapúak
2. Hierarchikus szervezetekben a vezető látja az alárendelteket
3. HR adminisztrátorok gyakran területi/divíziós felelősséggel rendelkeznek
4. Delegálás (szabadság, távollét) gyakori követelmény

---

### 12.2. Összefoglaló döntési mátrix

| Kérdés | MVP megoldás | Extended megoldás (ajánlott) | Strategic megoldás | Implementációs ajánlás |
|--------|--------------|------------------------------|---------------------|------------------------|
| **OrgUnit vs. Site szétválasztás** | OrgUnit `SITE` típussal | Önálló Site entitás + OrgUnitSite kapcsolótábla | Site-alapú munkarend-automatizálás, multi-site workforce planning | Extended – ha 5+ telephely, telephely-szintű reporting szükséges |
| **Költséghely kezelése** | `costCenterCode` string az OrgUnit-on | Önálló CostCenter entitás + OrgUnitCostCenter (N:M) | Költséghely-hierarchia, projekt-alapú költséghelyek, ABAC költséghely-jogosultság | Extended – ha költséghely ≠ szervezeti struktúra, ERP integráció |
| **Mátrix szervezet** | `secondaryOrgUnitId` az Employment-en | AssignmentToOrgUnit entitás (N funkcionális vonal) | Project entitás + AssignmentToProject + időkövetés | Extended – ha mátrix szervezet, projekt-alapú munkavégzés |
| **Jogi személy vs. OrgUnit határvonal** | Minden jogi személy = Organization | OrgUnit.isLegallyIndependent + döntési logika | OrgUnit.linkedOrganizationId – dupla reprezentáció | Extended – közszféra (köznevelés, egészségügy, önkormányzat) |
| **Időutazás (point-in-time)** | `effectiveFrom`/`effectiveTo` + új rekord verzióváltáskor | BusinessKey + Version Chain + Point-in-Time lekérdezések (SCD-2) | Bitemporal modell (Valid Time + Transaction Time) | Extended – ha gyakori átszervezés, audit követelmény, történelmi reporting |
| **Hozzáférés-kezelés** | Egyszerű RBAC (User.role enum) | RBAC + OrgUnit Scope + hierarchikus propagáció + delegálás | ABAC – attribútum-alapú jogosultságkezelés | Extended – HRMS jellegéből adódóan szervezeti egység-alapú jogosultságkezelés szükséges |

**Általános implementációs ajánlás:**
- **MVP:** Kis szervezetek (< 50 fő), egyszerű szervezeti struktúra, versenyszféra
- **Extended:** Közepes-nagy szervezetek (50+ fő), közszféra, mátrix szervezet, gyakori átszervezések
- **Strategic:** Multinacionális vállalatok, holding struktúrák, komplex compliance követelmények, audit-kritikus környezet

**Jogszabályi megfelelés:**
Minden megoldás (MVP, Extended, Strategic) GDPR- és magyar jogszabály-kompatibilis, de az Extended megoldás jobban támogatja:
- Közszférás szervezetek speciális igényeit (köznevelés, egészségügy, közigazgatás)
- Audit követelményeket (Ki? Mikor? Mit? kérdések megválaszolása)
- Jogviszony-típusok sokszínűségét (Mt., Kjt., Kit., Kttv., Púétv., Eszjtv.)

**Következő lépés:**
Az MVP és Extended megoldások implementálása, Strategic megoldások opcionálisan, ha konkrét üzleti igény merül fel.
