# Qualification (Végzettség / Képesítés) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer  
> **Verzió:** 0.1 – első tervezet  
> **Utolsó frissítés:** 2025. február  
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md, employment-entity.md, compensation-entity.md, organization-entity.md, position-entity.md

---

## 1. Az entitás célja és hatóköre

A **Qualification** entitáscsalád a természetes személy (Person) megszerzett végzettségeit, képesítéseit, nyelvtudását, szakmai vizsgáit, hatósági engedélyeit és egyéb kompetenciáit nyilvántartja. Ez az entitás az HRMS egyik legszélesebb hatókörű nyilvántartása, mert a magyar foglalkoztatási jog számos ponton **előírja** a végzettség és képesítés dokumentálását, ellenőrzését és nyilvántartását.

**Miért kritikus a Qualification entitás?**

| Felhasználási terület | Jogszabály | Hatás |
|---|---|---|
| **Munkakör betölthetősége** | Kjt. 61. §; Púétv. 4–5. §; Eszjtv.; Kit. | A végzettség meghatározza, milyen pozíciót tölthet be a személy |
| **Besorolás és illetmény** | Kjt. 61. § (fizetési osztály = végzettség); Kttv. (illetménykiegészítés) | A végzettség közvetlenül meghatározza az illetményt |
| **Pedagógus-előmenetel** | Púétv. 96–97. § | A fokozatváltás (Ped.I → Ped.II → Mester) képzettségfüggő |
| **Nyelvpótlék** | Kttv. 141. §; Kit. | Komplex nyelvvizsga → 100% illetményalap pótlék |
| **Kamarai tagság** | Eütev. – MOK, MESZK, MGYK | Egészségügyi munkakör betöltési feltétele |
| **Szakvizsga** | Kit.; Kttv.; Eütev. | Közigazgatási / egészségügyi szakvizsga határidő |
| **NAV bejelentés** | Tbj. – T1041 | A végzettség a bejelentés tartalmi eleme |
| **Továbbképzési kötelezettség** | Púétv. 114. §; Kit.; Kttv.; Eszjtv. | A teljesített továbbképzések nyilvántartása |
| **Pótlék- és kedvezmény-jogosultság** | Púétv. 101. § (mesterfokozat); Eszjtv. 3/2023. OKFŐ 13–16. pont (képesítési pótlék) | Többlet-végzettség → pótlék |

**Tervezési alapelvek:**

- **Person-hez kötött, jogviszony-független:** A végzettség a személyhez tartozik, nem a jogviszonyhoz. Ugyanaz a diploma több jogviszonynál is releváns lehet.
- **Sokféle típus egységes modellben:** Iskolai végzettség, szakképesítés, nyelvvizsga, szakvizsga, hatósági engedély, továbbképzés, kompetencia – mind a Qualification entitáscsalád része, de eltérő property-készlettel.
- **Dokumentum-alapú:** Minden végzettséghez/képesítéshez eredeti okirat tartozik, amelyet a munkáltató **megtekinthet, de másolatot nem készíthet** (Mt. 10. §). A rendszerben a dokumentum metaadatait tároljuk, nem feltétlenül a digitális másolatot.
- **Lejárat és megújítás:** Egyes képesítések (kamarai tagság, hatósági engedély, nyelvvizsga) lejárattal rendelkeznek vagy megújítandók – a rendszernek figyelmeztetnie kell a lejárat előtt.

---

## 2. Entitásstruktúra – áttekintés

```
Person 1 ──── N Qualification                  (egy személynek több végzettsége)
Qualification ──── 1 QualificationType          (végzettség típus törzsadat)
Qualification 1 ──── N QualificationDocument    (igazoló okiratok metaadatai)

Person 1 ──── N LanguageSkill                   (nyelvtudás – külön alentitás)
Person 1 ──── N ProfessionalExam                (szakvizsgák – külön alentitás)
Person 1 ──── N TrainingRecord                  (továbbképzések – Employment-hez is köthető)
Person 1 ──── N License                         (hatósági engedélyek, kamarai tagságok)
```

A szétválasztás indoka: bár mindegyik a „képzettség" témakörébe tartozik, a property-készletük, lejárati logikájuk és jogszabályi hátterük olyannyira eltérő, hogy az egységes „mindent egy táblába" modell nehezen kezelhető lenne. A **Qualification** az iskolai végzettségeket és szakképesítéseket tartalmazza; a többit önálló alentitás kezeli.

---

## 3. Qualification (Iskolai végzettség / Szakképesítés)

### 3.1. Alapadatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `personId` | UUID (FK) | igen | Hivatkozás a Person entitásra | Technikai |
| `qualificationType` | enum | igen | Végzettség típusa (lásd 3.2.) | – |
| `qualificationLevel` | enum | igen | Végzettség szintje (lásd 3.3.) | – |
| `name` | string | igen | Végzettség / képesítés megnevezése (pl. „okleveles jogász", „általános orvos", „villanyszerelő") | Oklevél / bizonyítvány |
| `specialization` | string | nem | Szak / szakirány (pl. „munkajog", „belgyógyászat", „erősáramú") | Oklevél |
| `institution` | string | igen | Kibocsátó intézmény neve | Oklevél |
| `institutionCountry` | string(2) | igen | Kibocsátó intézmény országa (ISO 3166-1 alpha-2) | Külföldi végzettség elismerése: OFIK |
| `dateObtained` | date | igen | Megszerzés dátuma | Oklevél |
| `dateExpiry` | date | nem | Lejárat (ha van – pl. OKJ-s szakképesítéseknél nincs; egyes hatósági képesítéseknél van) | – |
| `diplomaNumber` | string | nem | Oklevél / bizonyítvány száma | Oklevél |
| `isVerified` | boolean | igen | A munkáltató ellenőrizte-e az eredeti okiratot | Mt. 10. § – a munkáltató jogosult megtekinteni |
| `verifiedAt` | datetime | nem | Ellenőrzés dátuma | – |
| `verifiedBy` | string | nem | Ellenőrző személy | – |
| `isForeignQualification` | boolean | igen | Külföldi végzettség-e | – |
| `foreignRecognitionStatus` | enum | nem | Külföldi végzettség elismerési státusza (lásd 3.4.) | 2001. évi C. tv. – külföldi bizonyítványok és oklevelek elismerése |
| `foreignRecognitionDate` | date | nem | Elismerés dátuma | – |
| `foreignRecognitionNumber` | string | nem | Elismerési határozat száma | – |
| `notes` | string | nem | Megjegyzés | – |
| `isActive` | boolean | igen | Aktív (érvényes) végzettség-e | – |

### 3.2. Végzettség típusok (qualificationType)

| Kód | Megnevezés | Leírás |
|---|---|---|
| `SCHOOL_PRIMARY` | Alapfokú iskolai végzettség | Általános iskola 8 osztálya |
| `SCHOOL_SECONDARY_GENERAL` | Középfokú végzettség – érettségi (gimnáziumi) | Gimnáziumi érettségi |
| `SCHOOL_SECONDARY_VOCATIONAL` | Középfokú végzettség – érettségi (szakgimnáziumi/technikumi) | Szakgimnáziumi érettségi + középfokú szakképesítés |
| `VOCATIONAL_QUALIFICATION` | Szakképesítés (érettségi nélkül) | Korábbi OKJ / új Szkt. szerinti szakképesítés, érettségit nem igénylő |
| `VOCATIONAL_QUALIFICATION_ADVANCED` | Emelt szintű szakképesítés / technikus | Technikusi végzettség; OKJ emelt szintű |
| `HIGHER_ED_BA_BSC` | Felsőfokú – alapképzés (BA/BSc) | Bologna-rendszer első ciklus |
| `HIGHER_ED_MA_MSC` | Felsőfokú – mesterképzés (MA/MSc) | Bologna-rendszer második ciklus |
| `HIGHER_ED_UNDIVIDED` | Felsőfokú – osztatlan képzés | Orvos, jogász, gyógyszerész, építészmérnök, állatorvos (5–6 év) |
| `HIGHER_ED_COLLEGE` | Felsőfokú – főiskolai (régi rendszer) | 2006 előtti főiskolai diploma (BA/BSc-vel egyenértékű) |
| `HIGHER_ED_UNIVERSITY` | Felsőfokú – egyetemi (régi rendszer) | 2006 előtti egyetemi diploma (MA/MSc-vel egyenértékű) |
| `POSTGRADUATE_SPECIALIST` | Szakirányú továbbképzés | Egyetem által kibocsátott szakirányú végzettség (pl. pedagógus-szakvizsga, közgazdasági szakértő) |
| `PHD_DLA` | Doktori fokozat (PhD/DLA) | Tudományos fokozat |
| `HABILITATION` | Habilitáció | Egyetemi tanári képesítés |
| `DSC` | MTA doktora (DSc) | Akadémiai doktori cím |
| `OTHER` | Egyéb | Kategorizálatlan |

### 3.3. Végzettség szintek (qualificationLevel) – MKKR/EQF megfeleltetés

A Magyar Képesítési Keretrendszer (MKKR) az Európai Képesítési Keretrendszer (EQF) magyar megfelelője:

| Kód | MKKR szint | EQF szint | Megnevezés | Kjt. fizetési osztály megfelelés |
|---|---|---|---|---|
| `LEVEL_1` | 1 | 1 | Általános iskola alsó tagozat | – |
| `LEVEL_2` | 2 | 2 | Általános iskola 8 osztálya | A, B |
| `LEVEL_3` | 3 | 3 | Szakképesítés (érettségi nélkül) | C |
| `LEVEL_4` | 4 | 4 | Érettségi / középfokú szakképesítés | D, E |
| `LEVEL_5` | 5 | 5 | Emelt szintű szakképesítés / felsőfokú szakképzés (FOSZ) | F |
| `LEVEL_6` | 6 | 6 | BA/BSc / főiskolai | G, H |
| `LEVEL_7` | 7 | 7 | MA/MSc / egyetemi / osztatlan mester | I, J |
| `LEVEL_8` | 8 | 8 | PhD/DLA | (J feletti – speciális pótlék) |

**Megjegyzések:**
- A Kjt. 61. § szerint a **fizetési osztályt** az iskolai végzettség határozza meg: A–B (alapfokú), C (OKJ szakképesítés), D–E (érettségi), F (akkreditált felsőfokú), G–H (BA/főiskolai), I–J (MA/egyetemi). Ez a rendszer automatikusan leképezhető a `qualificationLevel` → `kjt_requiredPayGrade` relációval.
- A Púétv. pedagógus-előmeneteléhez a **végzettség szintje és típusa** egyaránt szükséges (nem elég a diploma szintje, pedagógus szakképzettségnek kell lennie).
- Az EQF szint az EU-s munkaerő-mobilitáshoz és a külföldi végzettségek elismeréséhez szükséges.

### 3.4. Külföldi végzettség elismerési státusz (foreignRecognitionStatus)

| Kód | Megnevezés | Leírás |
|---|---|---|
| `NOT_REQUIRED` | Elismerés nem szükséges | EU/EGT-tagállami végzettség egyes esetekben automatikusan elismert |
| `PENDING` | Elismerés folyamatban | OFIK eljárás alatt |
| `RECOGNIZED_EQUIVALENT` | Elismerve – egyenértékűnek nyilvánítva | A magyar végzettséggel egyenértékű |
| `RECOGNIZED_PARTIAL` | Elismerve – részben | Különbözeti vizsgával kiegészítendő |
| `REJECTED` | Elutasítva | Az elismerési kérelmet elutasították |
| `EU_AUTO_RECOGNIZED` | EU automatikus elismerés (szektorális irányelvek) | Orvos, ápoló, gyógyszerész, fogász, állatorvos, szülésznő, építész – EU 2005/36/EK irányelv |

---

### 3.5. Audit mezők

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `createdAt` | datetime | igen | Rekord létrehozása |
| `createdBy` | string | igen | Létrehozó |
| `updatedAt` | datetime | igen | Utolsó módosítás |
| `updatedBy` | string | igen | Módosító |

---

## 4. LanguageSkill (Nyelvtudás)

### 4.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `personId` | UUID (FK) | igen | Hivatkozás a Person entitásra | Technikai |
| `languageCode` | string(3) | igen | Nyelv kódja (ISO 639-2/B, pl. `hun`, `eng`, `deu`) | – |
| `languageName` | string | igen | Nyelv megnevezése | – |
| `proficiencyLevel` | enum | igen | Tudásszint (lásd 4.2.) | – |
| `hasOfficialExam` | boolean | igen | Rendelkezik-e hivatalos nyelvvizsgával | – |
| `examType` | enum | feltételes | Nyelvvizsga típusa (ha `hasOfficialExam` = true) | – |
| `examLevel` | enum | feltételes | Nyelvvizsga szintje | – |
| `examCenter` | string | nem | Vizsgaközpont megnevezése (pl. „Euroexam", „ECL", „BME") | – |
| `examDate` | date | nem | Vizsga dátuma | – |
| `examCertificateNumber` | string | nem | Bizonyítvány száma | – |
| `isCERFMapped` | boolean | nem | A vizsga CERF szinthez illeszkedik-e | – |
| `cerfLevel` | enum | nem | CERF szint: `A1`, `A2`, `B1`, `B2`, `C1`, `C2` | Közös Európai Referenciakeret |
| `isNativeLanguage` | boolean | nem | Anyanyelv-e | – |
| `isVerified` | boolean | igen | Ellenőrzött-e | – |
| `notes` | string | nem | Megjegyzés | – |
| `isActive` | boolean | igen | Aktív-e | – |
| `createdAt` | datetime | igen | Létrehozás | – |
| `createdBy` | string | igen | Létrehozó | – |
| `updatedAt` | datetime | igen | Módosítás | – |
| `updatedBy` | string | igen | Módosító | – |

### 4.2. Tudásszint (proficiencyLevel)

| Kód | Megnevezés | CERF | Leírás |
|---|---|---|---|
| `BASIC` | Alapfokú | A1–A2 | Egyszerű kommunikáció |
| `INTERMEDIATE` | Középfokú | B1–B2 | Önálló nyelvhasználat |
| `ADVANCED` | Felsőfokú | C1–C2 | Professzionális szintű nyelvhasználat |
| `NATIVE` | Anyanyelvi | – | Anyanyelv |

### 4.3. Magyar nyelvvizsga típusok (examType)

| Kód | Megnevezés | Leírás | Pótlékra jogosít? |
|---|---|---|---|
| `GENERAL_ORAL` | Általános szóbeli | Csak szóbeli vizsgarész | Nem (egyrészes) |
| `GENERAL_WRITTEN` | Általános írásbeli | Csak írásbeli vizsgarész | Nem (egyrészes) |
| `GENERAL_COMPLEX` | Általános komplex (szóbeli + írásbeli) | Komplex (korábban „C típusú") | Igen – Kttv. 141. § |
| `PROFESSIONAL_ORAL` | Szaknyelvi szóbeli | Szaknyelvi, csak szóbeli | Nem |
| `PROFESSIONAL_WRITTEN` | Szaknyelvi írásbeli | Szaknyelvi, csak írásbeli | Nem |
| `PROFESSIONAL_COMPLEX` | Szaknyelvi komplex | Szaknyelvi komplex vizsga | Igen – Kttv. 141. § |

### 4.4. Nyelvvizsga szintek (examLevel)

| Kód | Megnevezés | CERF megfelelés | Kttv. nyelvpótlék mértéke |
|---|---|---|---|
| `BASIC` | Alapfokú | B1 | Alapfokú komplex: illetményalap 50%-a |
| `INTERMEDIATE` | Középfokú | B2 | Középfokú komplex: illetményalap 60%-a |
| `ADVANCED` | Felsőfokú | C1 | Felsőfokú komplex: illetményalap 100%-a |

**Megjegyzések:**
- A **nyelvpótlék** (Kttv. 141. §) kizárólag **komplex** (szóbeli + írásbeli) nyelvvizsgáért jár, egyrészes vizsgáért nem.
- A Kttv. 141. § szerint a nyelvpótlék az illetményalap százalékában kerül megállapításra; a rendszernek a Compensation entitásba kell átvezetnie a pótlékot, ha a feltételek teljesülnek.
- A Kit. is tartalmaz nyelvpótlék-jellegű juttatást, de eltérő szabályozással.
- A Púétv. nem tartalmaz külön nyelvpótlékot, de a pedagógus-munkakörhöz előírt nyelvtudás a foglalkoztatás feltétele.
- Külföldi nyelvvizsgák (Cambridge, Goethe, DELE, DELF stb.) honosítása az OFIK hatáskörébe tartozik – a rendszernek nyilván kell tartania a honosítás tényét.

---

## 5. ProfessionalExam (Szakvizsga)

### 5.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `personId` | UUID (FK) | igen | Hivatkozás a Person entitásra | Technikai |
| `examType` | enum | igen | Szakvizsga típusa (lásd 5.2.) | – |
| `examName` | string | igen | Szakvizsga megnevezése | – |
| `examSpecialization` | string | nem | Szakterület (pl. „belgyógyászat", „pénzügyi-számviteli") | – |
| `examDate` | date | igen | Vizsga dátuma | – |
| `examResult` | enum | igen | Eredmény: `PASSED`, `FAILED`, `EXEMPT` (felmentett) | – |
| `examCertificateNumber` | string | nem | Bizonyítvány száma | – |
| `examInstitution` | string | nem | Vizsgáztató szervezet | – |
| `isExpirable` | boolean | igen | Lejárattal rendelkezik-e | – |
| `expiryDate` | date | nem | Lejárat dátuma (ha van) | – |
| `renewalRequired` | boolean | nem | Megújítás szükséges-e (lejárat előtt) | – |
| `nextRenewalDate` | date | nem | Következő megújítás esedékessége | – |
| `legalBasis` | string | nem | Jogszabályi hivatkozás | – |
| `isVerified` | boolean | igen | Ellenőrzött-e | – |
| `notes` | string | nem | Megjegyzés | – |
| `isActive` | boolean | igen | Aktív (érvényes)-e | – |
| `createdAt` | datetime | igen | Létrehozás | – |
| `createdBy` | string | igen | Létrehozó | – |
| `updatedAt` | datetime | igen | Módosítás | – |
| `updatedBy` | string | igen | Módosító | – |

### 5.2. Szakvizsga típusok (examType)

| Kód | Megnevezés | Kinek kötelező? | Jogszabály | Lejárat |
|---|---|---|---|---|
| `CIVIL_SERVICE_BASIC` | Közigazgatási alapvizsga | Kit./Kttv. alatti köztisztviselők – kinevezéstől számított határidőn belül | Kit. 119. §; Kttv. 29. § | Nincs lejárat |
| `CIVIL_SERVICE_PROFESSIONAL` | Közigazgatási szakvizsga | Kit./Kttv. alatti egyes vezetői és szaktanácsadói pozíciók | Kit. 119. §; Kttv. 29. § (5) | Nincs lejárat |
| `LEGAL_BAR_EXAM` | Jogi szakvizsga | Ügyvéd, bíró, ügyész, jogtanácsos | 2016. évi LXXVI. tv. | Nincs lejárat |
| `MEDICAL_SPECIALIST` | Egészségügyi szakvizsga | Szakorvos, klinikai szakpszichológus, szakgyógyszerész | Eütev. 22–24. § | Nincs lejárat |
| `PEDAGOGICAL_EXAM` | Pedagógus szakvizsga | Mesterpedagógus fokozathoz szükséges (Púétv.) | Púétv. 96. § (6) | Nincs lejárat |
| `PEDAGOGICAL_QUALIFICATION` | Pedagógus-minősítés | Fokozatváltáshoz (Gyakornok → Ped.I → Ped.II → Mester → Kutató) | Púétv. 96–97. § | Nincs lejárat (de ciklusos) |
| `OCCUPATIONAL_SAFETY` | Munkavédelmi vizsga | Munkavédelmi szakember | Mvt. | Megújítandó (5 évente) |
| `FIRE_SAFETY` | Tűzvédelmi vizsga | Tűzvédelmi szakember | 1996. évi XXXI. tv. | Megújítandó |
| `ACCOUNTING_EXAM` | Mérlegképes könyvelő vizsga | Számviteli pozíciókhoz | Szt. 150–151. § | Kreditpontos megújítás |
| `TAX_ADVISOR_EXAM` | Adótanácsadói vizsga | Adótanácsadó, adószakértő | Art. | Regisztrációs megújítás |
| `ELECTRICAL_QUALIFICATION` | Villamos szakképesítés (A–E kategória) | Villamos berendezésen dolgozók | 11/2022. TIFK r. | Megújítandó (5 évente) |
| `OTHER` | Egyéb szakvizsga | – | – | Változó |

---

## 6. TrainingRecord (Továbbképzési nyilvántartás)

### 6.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `personId` | UUID (FK) | igen | Hivatkozás a Person entitásra | Technikai |
| `employmentId` | UUID (FK) | nem | Hivatkozás az Employment-re (ha a továbbképzés konkrét jogviszonyhoz kapcsolódik) | Technikai |
| `trainingType` | enum | igen | Továbbképzés típusa (lásd 6.2.) | – |
| `trainingName` | string | igen | Képzés megnevezése | – |
| `provider` | string | nem | Képzőszervezet / szolgáltató | – |
| `startDate` | date | igen | Képzés kezdete | – |
| `endDate` | date | nem | Képzés vége | – |
| `durationHours` | integer | nem | Időtartam (órában) | – |
| `creditPoints` | decimal | nem | Szerzett kreditpont / tanulmányi pont | Púétv. 114. §; Kit./Kttv. – 499/2021. Korm. r. |
| `creditPointType` | enum | nem | Kreditpont típusa: `PEDAGOGUE_TRAINING` (Púétv. továbbképzési pont), `GOV_TRAINING` (közszolgálati tanulmányi pont), `HEALTH_TRAINING` (eü. kreditpont), `ACCOUNTING_CREDIT` (mérlegképes könyvelő kredit), `OTHER` | – |
| `result` | enum | nem | Eredmény: `COMPLETED`, `IN_PROGRESS`, `FAILED`, `WITHDRAWN` | – |
| `certificateNumber` | string | nem | Tanúsítvány / igazolás száma | – |
| `isMandatory` | boolean | nem | Kötelező továbbképzés-e (jogszabályi előírás) | – |
| `isEmployerFunded` | boolean | nem | Munkáltatói finanszírozású-e | – |
| `cost` | decimal | nem | Képzés költsége (Ft) | – |
| `trainingAgreement` | boolean | nem | Tanulmányi szerződés kötötte-e | Mt. 229. § – tanulmányi szerződés |
| `bondingPeriodEnd` | date | nem | Röghöz kötési idő vége (ha tanulmányi szerződés) | Mt. 229. § (3) |
| `notes` | string | nem | Megjegyzés | – |
| `isActive` | boolean | igen | Aktív-e | – |
| `createdAt` | datetime | igen | Létrehozás | – |
| `createdBy` | string | igen | Létrehozó | – |
| `updatedAt` | datetime | igen | Módosítás | – |
| `updatedBy` | string | igen | Módosító | – |

### 6.2. Továbbképzés típusok (trainingType)

| Kód | Megnevezés | Jellemző jogviszony | Jogszabály |
|---|---|---|---|
| `PUETV_MANDATORY` | Pedagógus kötelező továbbképzés (7 éves ciklus) | Púétv. | Púétv. 114. §; 277/1997. Korm. r. |
| `PUETV_SPECIALIZATION` | Pedagógus szakirányú továbbképzés | Púétv. | Púétv. 114. § |
| `GOV_MANDATORY` | Közszolgálati kötelező továbbképzés | Kit./Kttv. | 499/2021. Korm. r. |
| `GOV_LEADERSHIP` | Vezetőképzés (közszolgálat) | Kit./Kttv. | 499/2021. Korm. r. |
| `HEALTH_MANDATORY` | Egészségügyi kötelező szakmai továbbképzés | Eszjtv. | 52/2012. EMMI r. |
| `HEALTH_SPECIALIZATION` | Egészségügyi szakosító képzés (rezidens) | Eszjtv. | Eütev. 22–24. § |
| `ACCOUNTING_CREDIT` | Mérlegképes könyvelő kreditpontos továbbképzés | Mt. | Szt. 152/A. § |
| `OCCUPATIONAL_SAFETY` | Munkavédelmi továbbképzés | Minden | Mvt. |
| `FIRE_SAFETY` | Tűzvédelmi továbbképzés | Minden | 1996. évi XXXI. tv. |
| `DATA_PROTECTION` | Adatvédelmi továbbképzés | Minden (ahol DPO szükséges) | GDPR 39. cikk |
| `INTERNAL` | Belső képzés (munkáltatói szervezésű) | Minden | – |
| `EXTERNAL_PROFESSIONAL` | Külső szakmai képzés / konferencia | Minden | – |
| `DEGREE_PROGRAM` | Diplomaszerzés (tanulmányi szerződéssel) | Minden | Mt. 229. § |
| `OTHER` | Egyéb | – | – |

---

## 7. License (Hatósági engedély / Kamarai tagság)

### 7.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `personId` | UUID (FK) | igen | Hivatkozás a Person entitásra | Technikai |
| `licenseType` | enum | igen | Engedély típusa (lásd 7.2.) | – |
| `licenseName` | string | igen | Engedély megnevezése | – |
| `licenseNumber` | string | igen | Engedély / nyilvántartási szám | – |
| `issuingAuthority` | string | igen | Kibocsátó hatóság / kamara / szervezet | – |
| `issueDate` | date | igen | Kibocsátás / nyilvántartásba vétel dátuma | – |
| `expiryDate` | date | nem | Lejárat dátuma (null = határozatlan) | – |
| `renewalDate` | date | nem | Legutóbbi megújítás dátuma | – |
| `nextRenewalDue` | date | nem | Következő megújítás esedékessége | – |
| `status` | enum | igen | Státusz: `ACTIVE`, `SUSPENDED`, `EXPIRED`, `REVOKED`, `PENDING` | – |
| `isMandatoryForPosition` | boolean | nem | A jelenlegi munkakör betöltéséhez szükséges-e | Position.QualificationRequirement alapján |
| `legalBasis` | string | nem | Jogszabályi hivatkozás | – |
| `conditions` | string | nem | Az engedélyhez fűzött feltételek / korlátozások | – |
| `isVerified` | boolean | igen | Ellenőrzött-e | – |
| `notes` | string | nem | Megjegyzés | – |
| `isActive` | boolean | igen | Aktív-e | – |
| `createdAt` | datetime | igen | Létrehozás | – |
| `createdBy` | string | igen | Létrehozó | – |
| `updatedAt` | datetime | igen | Módosítás | – |
| `updatedBy` | string | igen | Módosító | – |

### 7.2. Engedély típusok (licenseType)

| Kód | Megnevezés | Kibocsátó | Megújítás | Jogszabály |
|---|---|---|---|---|
| `MOK_MEMBERSHIP` | Magyar Orvosi Kamara tagság | MOK | Éves tagdíjfizetéssel | 2006. évi XCVII. tv. |
| `MESZK_MEMBERSHIP` | Egészségügyi Szakdolgozói Kamara tagság | MESZK | Éves tagdíjfizetéssel | 2003. évi LXXXIII. tv. |
| `MGYK_MEMBERSHIP` | Gyógyszerészi Kamara tagság | MGYK | Éves tagdíjfizetéssel | 2006. évi XCVII. tv. |
| `HEALTH_OPERATIONS_REGISTER` | Egészségügyi működési nyilvántartás | NEAK / OKFŐ | 5 évente megújítandó | Eütev. 7. § |
| `PEDAGOGICAL_REGISTER` | Pedagógus-nyilvántartás (OM-azonosító) | KRÉTA / OH | Folyamatos | Púétv. – pedagógus nyilvántartó |
| `BAR_MEMBERSHIP` | Ügyvédi kamara tagság | MÜK | Éves | 2017. évi LXXVIII. tv. |
| `AUDITOR_LICENSE` | Könyvvizsgálói engedély | MKVK | Éves | 2007. évi LXXV. tv. |
| `TAX_ADVISOR_REGISTRATION` | Adótanácsadó / adószakértő regisztráció | NAV | Éves regisztrációs díj | Art. |
| `DRIVING_LICENSE` | Gépjárművezetői engedély | Közlekedési hatóság | Lejárat van; munkakör-feltétel lehet | – |
| `FORKLIFT_LICENSE` | Targoncavezetői engedély | Vizsgáztató szervezet | Időszakos megújítás | – |
| `CRANE_OPERATOR` | Darukezelői engedély | Vizsgáztató szervezet | Időszakos megújítás | – |
| `ELECTRICAL_LICENSE` | Villamos munkavégzési jogosultság | Munkáltató + hatóság | 5 évente | 11/2022. TIFK r. |
| `SECURITY_CLEARANCE` | Nemzetbiztonsági ellenőrzés | NBF | 5 évente megújítandó | Nbtv. |
| `OTHER` | Egyéb | – | Változó | – |

---

## 8. Továbbképzési kötelezettség nyilvántartása

### 8.1. Pedagógus továbbképzés (Púétv.)

| Jellemző | Leírás | Jogszabály |
|---|---|---|
| Ciklus hossza | 7 év | Púétv. 114. § |
| Kötelező | Igen, minden pedagógus-munkakörben | 277/1997. Korm. r. |
| Elszámolás | Továbbképzési pontszám (120 pont / 7 év) | 277/1997. Korm. r. |
| Mentesség | 55. életév felett, vagy ha a ciklusban mesterpedagógus fokozatot szerez | Púétv. |
| HRMS kezelése | `Employment.puetv_continuousTrainingCycleStart` + `TrainingRecord.creditPoints` összesítés | – |

### 8.2. Közszolgálati továbbképzés (Kit./Kttv.)

| Jellemző | Leírás | Jogszabály |
|---|---|---|
| Ciklus hossza | Éves (tárgyévre meghatározott kötelezettség) | 499/2021. Korm. r. |
| Kötelező | Igen, közszolgálati tisztviselők | 499/2021. Korm. r. |
| Elszámolás | Tanulmányi pont (tárgyévi minimum) | 499/2021. Korm. r. |
| HRMS kezelése | `Employment.gov_trainingCreditsRequired` + `Employment.gov_trainingCreditsCompleted` + `TrainingRecord` | – |

### 8.3. Egészségügyi továbbképzés (Eszjtv.)

| Jellemző | Leírás | Jogszabály |
|---|---|---|
| Ciklus hossza | 5 éves szakmai továbbképzési időszak | 52/2012. EMMI r. |
| Kötelező | Igen, egészségügyi dolgozók | 52/2012. EMMI r. |
| Elszámolás | Kreditpont (250 pont / 5 év) | 52/2012. EMMI r. |
| HRMS kezelése | `TrainingRecord.creditPoints` (creditPointType = `HEALTH_TRAINING`) + ciklus nyilvántartás | – |

### 8.4. Mérlegképes könyvelő (Szt.)

| Jellemző | Leírás | Jogszabály |
|---|---|---|
| Ciklus hossza | Éves (2024-től: évi 42 kredit) | Szt. 152/A. § |
| Kötelező | Igen, regisztrált mérlegképes könyvelők | Szt. 152/A. § |
| HRMS kezelése | `TrainingRecord.creditPoints` (creditPointType = `ACCOUNTING_CREDIT`) | – |

---

## 9. Automatikus események és figyelmeztetések

| Esemény | Kiváltó feltétel | Akció |
|---|---|---|
| **Kamarai tagság lejárat** | `License.expiryDate` – 90 nap | Figyelmeztetés HR-nek és érintettnek |
| **Működési nyilvántartás megújítás** | `License.nextRenewalDue` – 180 nap | Figyelmeztetés; Eszjtv. hatály alatt a munkakör betöltési feltétele |
| **Közigazgatási szakvizsga határidő** | `Employment.gov_civilServiceExamDeadline` közelít | Figyelmeztetés; Kit./Kttv. – mulasztás következményei |
| **Pedagógus továbbképzési ciklus lejárta** | `Employment.puetv_continuousTrainingCycleStart` + 7 év | Figyelmeztetés; teljesített pontok ellenőrzése |
| **Pedagógus-minősítés esedékessége** | `Employment.puetv_nextQualificationDue` | Figyelmeztetés; Púétv. |
| **Egészségügyi továbbképzési ciklus** | 5 éves ciklus lejárta | Kreditpont-összesítés ellenőrzése |
| **Villamos jogosultság megújítás** | `License.nextRenewalDue` (ELECTRICAL_LICENSE) – 90 nap | Figyelmeztetés; munkavédelmi kötelesség |
| **Nemzetbiztonsági ellenőrzés megújítás** | `License.nextRenewalDue` (SECURITY_CLEARANCE) – 180 nap | Figyelmeztetés |
| **Munkaköri alkalmassági vizsga esedékessége** | `Position.healthCheckFrequency` alapján számított időpont | Figyelmeztetés; Mvt. kötelesség |
| **Tanulmányi szerződés röghöz kötés lejárt** | `TrainingRecord.bondingPeriodEnd` | Informálás |
| **Betöltési feltétel nem teljesül** | Employment–Position összevetés: hiányzó Qualification | Figyelmeztetés |
| **Mérlegképes könyvelő kreditpont-hiány** | Éves kredit-kötelezettség nem teljesül | Figyelmeztetés |

---

## 10. GDPR adatkezelési kategorizáció

| Kategória | Property-k | Jogalap | Megőrzési idő |
|---|---|---|---|
| **Iskolai végzettség** | Qualification (SCHOOL_*, HIGHER_ED_*, PHD_DLA, stb.) | Jogi kötelezettség (6(1)c) – besorolás; Szerződés (6(1)b) – munkakör betöltési feltétel | Jogviszony megszűnése + 5 év |
| **Szakképesítés** | Qualification (VOCATIONAL_*) | Jogi kötelezettség (6(1)c) | Jogviszony megszűnése + 5 év |
| **Nyelvtudás** | LanguageSkill | Jogi kötelezettség (6(1)c) – nyelvpótlék; Szerződés (6(1)b) – betöltési feltétel | Jogviszony megszűnése + 5 év |
| **Szakvizsga** | ProfessionalExam | Jogi kötelezettség (6(1)c) – besorolás, betöltési feltétel | Jogviszony megszűnése + 5 év; szakvizsga bizonyítványt korlátlan ideig hivatkozni kell |
| **Továbbképzés** | TrainingRecord | Jogi kötelezettség (6(1)c) – ciklus-nyilvántartás; Jogos érdek (6(1)f) – fejlesztés-nyilvántartás | Jogviszony megszűnése + 5 év; tanulmányi szerződésnél: szerződés + röghöz kötés + 5 év |
| **Hatósági engedély / kamarai tagság** | License | Jogi kötelezettség (6(1)c) – betöltési feltétel | Jogviszony megszűnése + 5 év; aktív licenc ideje alatt folyamatosan |
| **Külföldi elismerés** | foreignRecognitionStatus, foreignRecognitionNumber | Jogi kötelezettség (6(1)c) | Végzettség megőrzési idejével megegyező |

**Megjegyzések:**
- A **végzettségi okiratok másolatának tárolása** kérdéses: az Mt. 10. § szerint a munkáltató az okiratot **megtekintheti**, de másolatot nem készíthet. A gyakorlatban sok munkáltató másolatot tart → GDPR szempontból a szükségesség és célhoz kötöttség mérlegelendő. Az HRMS-ben a **metaadatok** tárolása (diplomaszám, intézmény, dátum, ellenőrzés ténye) elegendő; a digitális másolat tárolásához külön jogalap szükséges.
- A Kjt. és Kit. hatálya alatti közalkalmazottak/köztisztviselők **személyi anyagában** a végzettség adatai jogszabály erejénél fogva nyilvántartandók.

---

## 11. Hozzáférési szintek

| Szerep | Qualification | LanguageSkill | ProfessionalExam | TrainingRecord | License |
|---|---|---|---|---|---|
| **Érintett munkavállaló** | Saját: teljes olvasás; saját kérelmezés (portálon keresztül) | Saját: olvasás/kérelmezés | Saját: olvasás | Saját: olvasás | Saját: olvasás |
| **Közvetlen vezető** | Beosztottak: releváns végzettségek olvasás (betöltési feltétel kontextusban) | Beosztottak: olvasás | Beosztottak: olvasás | Beosztottak: olvasás/jóváhagyás (képzési igény) | Beosztottak: lejárat-figyelés |
| **HR ügyintéző** | Teljes olvasás/írás a hatáskörben | Teljes olvasás/írás | Teljes olvasás/írás | Teljes olvasás/írás | Teljes olvasás/írás |
| **Bérszámfejtő** | Besoroláshoz szükséges adatok olvasás | Nyelvpótlékhoz releváns adatok | Szakvizsgapótlékhoz releváns | – | – |
| **Képzési koordinátor** | Végzettségek olvasás (képzési igény felmérés) | Olvasás | Olvasás | Teljes olvasás/írás | Olvasás |
| **Rendszeradminisztrátor** | Törzsadat karbantartás (QualificationType, FEOR–végzettség mapping) | Törzsadat (nyelvlista, vizsgaközpont) | Törzsadat | Törzsadat | Törzsadat |

---

## 12. Integrációs pontok

| Külső rendszer | Irány | Érintett adatok | Cél |
|---|---|---|---|
| **NAV (T1041)** | kimenő | qualificationLevel (legmagasabb végzettség) | Biztosítotti bejelentés |
| **KIR / KRÉTA** | kétirányú | Pedagógus végzettségek, továbbképzési adatok, minősítési eredmények | Pedagógus-nyilvántartás |
| **OKFŐ / NEAK** | kétirányú | Egészségügyi végzettségek, működési nyilvántartás, kamarai tagság | Eü. dolgozó nyilvántartás |
| **NKE Továbbképzési rendszer** | bejövő | Közszolgálati továbbképzés teljesítési adatok, tanulmányi pontok | Kit./Kttv. továbbképzési kötelezettség |
| **OFIK** | bejövő | Külföldi végzettség elismerési határozat | Külföldi végzettség honosítás |
| **MOK / MESZK / MGYK** | bejövő | Kamarai tagság státusz, érvényesség | Betöltési feltétel ellenőrzés |
| **Önkiszolgáló portál** | bejövő | Munkavállaló által feltöltött végzettség-igénylés + dokumentum | Workflow: jóváhagyás → rögzítés |
| **Toborzási rendszer (ATS)** | bejövő | Jelölt végzettségei (pályázat részeként) | Betöltési feltétel automatikus összevetés |

---

## 13. Kapcsolódó entitások – teljes kép

```
Person 1 ──── N Qualification              (iskolai végzettségek, szakképesítések)
Person 1 ──── N LanguageSkill              (nyelvtudás)
Person 1 ──── N ProfessionalExam           (szakvizsgák)
Person 1 ──── N TrainingRecord             (továbbképzések)
Person 1 ──── N License                    (hatósági engedélyek, kamarai tagságok)

Qualification ──── QualificationDocument    (igazoló okiratok metaadatai)
TrainingRecord ──── Employment             (opcionális kötés jogviszonyhoz)

Position ──── N QualificationRequirement   (betöltési feltételek)
PositionTemplate ──── N QualificationRequirement

QualificationRequirement ←──→ Qualification / LanguageSkill / ProfessionalExam / License
                              (összevetés: a személy teljesíti-e a feltételt?)
```

---

## 14. Nyitott kérdések

1. **Okirat-másolat tárolása:** Az Mt. 10. § tiltja a másolatkészítést. A gyakorlatban sok szervezet mégis tárolja. Az HRMS támogassa-e a digitalizált okmányok feltöltését, vagy csak a metaadatokat (diplomaszám, intézmény, dátum, ellenőrzés ténye)? Ha támogatja, milyen GDPR garanciákkal?
2. **Kompetencia-nyilvántartás:** A formális végzettségeken túl szükséges-e informális kompetenciák (soft skills, programozási nyelvek, eszközismeret) nyilvántartása? Ha igen, önálló Competency entitás vagy a Qualification kiterjesztése?
3. **Végzettség–besorolás automatizmus:** A Kjt.-ben a végzettség egyértelműen meghatározza a fizetési osztályt. Az HRMS automatikusan számolja-e a fizetési osztályt a legmagasabb releváns végzettségből, vagy manuális besorolás marad?
4. **Többszörös végzettség kezelése:** Ha valakinek több felsőfokú végzettsége van, melyik a „releváns" a besorolás szempontjából? A rendszer kezelje-e a „besorolás alapjául szolgáló végzettség" jelölését?
5. **Pedagógus-minősítés (Púétv.) elhelyezése:** A minősítés egyszerre szakvizsga-jellegű (ProfessionalExam) és az Employment besorolási mezője (puetv_careerStage). Hol legyen a „master record"?
6. **Szakmai gyakorlat (tapasztalat) nyilvántartása:** A Position.QualificationRequirement tartalmazhat „minimum X év tapasztalat" feltételt. Honnan számoljuk a tapasztalatot – az Employment historikus adataiból, vagy önálló ExperienceRecord entitásból?
7. **Továbbképzési terv:** A kötelező továbbképzések teljesítés-nyilvántartásán túl szükséges-e egyéni fejlesztési terv (IDP – Individual Development Plan) entitás, amely a tervezett képzéseket is tartalmazza?
