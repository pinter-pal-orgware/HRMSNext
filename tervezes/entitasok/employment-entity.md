# Employment (Jogviszony) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer  
> **Verzió:** 0.1 – első tervezet  
> **Utolsó frissítés:** 2025. február  
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md

---

## 1. Az entitás célja és hatóköre

Az **Employment** entitás egy konkrét természetes személy (Person) és egy munkáltató (Organization) között fennálló foglalkoztatási jogviszonyt reprezentálja. Ez az HRMS legkomplexebb entitása, mert a magyar foglalkoztatási jog fragmentált szerkezete miatt **hétféle jogviszony-típust** kell egységes modellben kezelnie, amelyek eltérő property-készlettel, besorolási rendszerrel, illetményszámítási logikával és jogosultsági szabályokkal rendelkeznek.

**Tervezési alapelvek:**

- **Jogviszony-típus mint diszkriminátor:** Az `employmentType` mező határozza meg, mely property-k kötelezőek, opcionálisak vagy irrelevánsak. A modell „közös mag + jogviszony-specifikus kiterjesztés" elvre épül.
- **Egy személy – több jogviszony:** Ugyanaz a Person több Employment rekordhoz is kapcsolódhat (pl. részmunkaidős + megbízási; többes jogviszony az egészségügyben; óraadó + főállás a köznevelésben).
- **Temporális modell:** A jogviszony életciklusa és a jogviszony egyes property-jeinek változásai (pl. munkakör-változás, átsorolás) is historikusan nyilvántartandók.
- **A jogviszony az „aggregáció gyökere":** A Compensation, TimeTracking, Leave és más operatív entitások az Employment-hez kapcsolódnak, nem közvetlenül a Person-höz.

---

## 2. Jogviszony-típusok és azok jellemzői

### 2.1. Támogatott jogviszony-típusok

| Kód | Jogviszony neve | Elsődleges törvény | Háttér-jogszabály | Létrejötte |
|---|---|---|---|---|
| `MUNKAVISZONY` | Munkaviszony | Mt. | – | Munkaszerződés |
| `KOZALKALMAZOTT` | Közalkalmazotti jogviszony | Kjt. | Mt. | Kinevezés |
| `KOZNEVELESI` | Köznevelési foglalkoztatotti jogviszony | Púétv. | Mt. (részben) | Kinevezés |
| `EGESZSEGUGYI` | Egészségügyi szolgálati jogviszony | Eszjtv. | Mt. | Eü. szolgálati munkaszerződés |
| `KORMANYZATI` | Kormányzati szolgálati jogviszony | Kit. | Mt. (korlátozott) | Kinevezés |
| `KOZSZOLGALATI` | Közszolgálati jogviszony (önkormányzati) | Kttv. | Mt. (korlátozott) | Kinevezés |
| `KULONLEGES` | Közszolgálati jogviszony (különleges jogállású szerv) | Küt. | Kit./Kttv. mintára | Kinevezés |

### 2.2. Property-érvényességi mátrix

Az alábbi mátrix jelzi, hogy egy-egy property-csoport melyik jogviszony-típusnál releváns:

| Property-csoport | Mt. | Kjt. | Púétv. | Eszjtv. | Kit. | Kttv. | Küt. |
|---|---|---|---|---|---|---|---|
| Alapadatok (2.1–2.3) | ● | ● | ● | ● | ● | ● | ● |
| Munkakör / álláshely | ● | ● | ● | ● | ● | ● | ● |
| Besorolás – fizetési osztály/fokozat | – | ● | – | – | – | – | – |
| Besorolás – pedagógus fokozat | – | – | ● | – | – | – | – |
| Besorolás – eü. fizetési fokozat | – | – | – | ● | – | – | – |
| Besorolás – besorolási kategória | – | – | – | – | ● | ● | ● |
| Próbaidő | ● | ● | ● | ● (kötelező 3 hó) | ● | ● | ● |
| Felmentési/felmondási védelem | ● | ● | ● | ● | ● | ● | ● |
| Összeférhetetlenség | – | ◐ | ◐ | ● (engedélyköteles) | ● | ● | ● |
| Eskütétel | – | – | – | – | ● | ● | ● |
| Vagyonnyilatkozat | – | – | – | – | ● | ● | ● |
| Közigazgatási szakvizsga | – | – | – | – | ● | ● | ◐ |
| Minősítés/teljesítményértékelés | ◐ | ● | ● | ◐ | ● | ● | ● |
| Továbbképzési kötelezettség | – | ◐ | ● (7 éves ciklus) | ◐ | ● | ● | ● |

● = kötelező / releváns, ◐ = opcionális / ágazattól függő, – = nem releváns

---

## 3. Property-k

### 3.1. Azonosítók és alapadatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `personId` | UUID (FK) | igen | Hivatkozás a Person entitásra | Technikai |
| `organizationId` | UUID (FK) | igen | Munkáltató szervezet | Mt. 45. §; minden jogállási törvény |
| `orgUnitId` | UUID (FK) | nem | Szervezeti egység | Szervezeti struktúra |
| `employmentType` | enum | igen | Jogviszony típusa (lásd 2.1.) – **diszkriminátor mező** | Meghatározza az alkalmazandó jogszabályt |
| `employmentSubType` | enum | nem | Alaptípuson belüli altípus (pl. Mt.: vezető állású, távmunkás, egyszerűsített, alkalmi; Kjt.: ágazat) | Mt. 198–211. §; Kjt. ágazati vhr.-ek |
| `employmentNumber` | string | nem | Munkáltatói törzsszám / iktatószám | Belső nyilvántartás |
| `externalId` | string | nem | Külső rendszer azonosító (migrációhoz, integrációhoz) | Technikai |

---

### 3.2. Jogviszony időbeli jellemzői

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `startDate` | date | igen | Jogviszony kezdete | Mt. 45. §; minden jogállási törvény – a munkaszerződés / kinevezés kötelező tartalmi eleme |
| `endDate` | date | nem | Jogviszony vége (null = határozatlan idejű) | Mt. 45. §; jogviszony megszűnés/megszüntetés |
| `contractDuration` | enum | igen | Határozatlan / határozott idejű | Mt. 192. § (határozott max. 5 év); Kjt.; Kit. stb. |
| `fixedTermEndDate` | date | feltételes | Határozott idő lejárta (ha `contractDuration` = határozott) | Mt. 192. § |
| `fixedTermReason` | string | feltételes | Határozott idő indoka | Mt. 192. § – objektív ok szükséges |
| `probationStartDate` | date | nem | Próbaidő kezdete | Mt. 45. § (3) – max. 3 hónap; Eszjtv. 3. § – kötelező 3 hó, max. 4 hó |
| `probationEndDate` | date | nem | Próbaidő vége | – |
| `seniorityDate` | date | igen | Jogviszonyban töltött idő számításának kezdőnapja (beszámítások után) | Kjt. – fizetési fokozat; Eszjtv. – fizetési fokozat; Kit./Kttv. – besorolás; Mt. – felmondási idő, végkielégítés |
| `previousServiceYears` | decimal | nem | Korábbi beszámítható jogviszonyok hossza (években) | Kjt. 87. §; Eszjtv. 8. § (8)–(9); Kit.; Kttv. |

**Megjegyzések:**
- A `seniorityDate` az egyik legkritikusabb mező: ebből számítódik a fizetési fokozat (Kjt., Eszjtv.), a besorolási kategória (Kit., Kttv.), a felmondási idő, a végkielégítés és a jubileumi jutalom jogosultság. A korábbi jogviszonyok beszámítási szabályai **jogállási törvényenként eltérők**.
- Az Eszjtv. külön felsorolja, mely korábbi jogviszonyok számíthatók be (közalkalmazotti, közszolgálati, kormányzati, kormánytisztviselői, bírói, ügyészi stb.).
- A próbaidő az Eszjtv. hatálya alatt **kötelezően 3 hónap** (max. 4 hónap), míg az Mt. szerint opcionális és max. 3 hónap.

---

### 3.3. Munkavégzés jellemzői

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `positionId` | UUID (FK) | igen | Betöltött munkakör / álláshely | Mt. 45. § – a munkakör a munkaszerződés kötelező eleme |
| `jobTitle` | string | igen | Munkakör megnevezése | Mt. 45. §; FEOR besoroláshoz |
| `feorCode` | string(6) | igen | FEOR-kód (Foglalkozások Egységes Osztályozási Rendszere) | KSH; T1041 bejelentés |
| `workPlace` | string | igen | Munkavégzés helye | Mt. 45. § – munkaszerződés kötelező eleme |
| `workPlaceType` | enum | nem | Munkavégzés típusa (telephelyi, távmunka, home office, változó) | Mt. 196–197/A. § |
| `workScheduleType` | enum | igen | Munkarend típusa (általános, egyenlőtlen, kötetlen, osztott) | Mt. 97. § |
| `weeklyHours` | decimal | igen | Heti munkaidő (órában) | Mt. 92. § – teljes: 40 óra; részmunkaidő: ettől kevesebb |
| `dailyHours` | decimal | igen | Napi munkaidő (órában) | Mt. 92. § – teljes: napi 8 óra |
| `isFullTime` | boolean | igen | Teljes munkaidős-e | Mt. 92. § |
| `workTimeFramePeriod` | integer | nem | Munkaidőkeret hossza (napokban, ha alkalmazandó) | Mt. 93–94. § |
| `nightWorkEligible` | boolean | nem | Éjszakai munkára beosztható-e | Mt. 89. § – korlátozások: várandós, kisgyermekes |

---

### 3.4. Besorolás – közös mező

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `classificationScheme` | enum | feltételes | Alkalmazott besorolási rendszer (kjt_grade, puetv_career, eszjtv_grade, kit_category, kttv_category, kut_category, none) | Jogviszony-típus határozza meg |

---

### 3.5. Kjt.-specifikus besorolás (közalkalmazottak)

*Feltétel: `employmentType` = `KOZALKALMAZOTT`*

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `kjt_payGrade` | enum | igen | Fizetési osztály (A, B, C, D, E, F, G, H, I, J) | Kjt. 61. § – iskolai végzettség alapján |
| `kjt_payStep` | integer(1–14) | igen | Fizetési fokozat (1–14) | Kjt. 62. § – közalkalmazotti jogviszonyban töltött idő alapján |
| `kjt_guaranteedSalary` | integer | igen | Garantált illetmény (a fizetési osztály × fokozat szerinti összeg) | Kjt. 66. § |
| `kjt_salaryCategory` | string | nem | Ágazati besorolási kategória (pl. szociális: S1–S14) | Ágazati végrehajtási rendeletek |
| `kjt_leaderLevel` | enum | nem | Magasabb vezetői / vezetői megbízás szintje | Kjt. 20–24. § |
| `kjt_leaderAllowance` | integer | nem | Vezetői pótlék összege | Kjt. 70. § |
| `kjt_qualificationAllowance` | integer | nem | Illetménykiegészítés (végzettség alapján) | Kjt. 66. § ágazati vhr. |
| `kjt_jubileeDate25` | date | nem | 25 éves jubileumi jutalom esedékessége | Kjt. 78. § |
| `kjt_jubileeDate30` | date | nem | 30 éves jubileumi jutalom esedékessége | Kjt. 78. § |
| `kjt_jubileeDate35` | date | nem | 35 éves jubileumi jutalom esedékessége | Kjt. 78. § |
| `kjt_jubileeDate40` | date | nem | 40 éves jubileumi jutalom esedékessége | Kjt. 78. § |

---

### 3.6. Púétv.-specifikus besorolás (köznevelési foglalkoztatottak)

*Feltétel: `employmentType` = `KOZNEVELESI`*

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `puetv_careerStage` | enum | igen | Pedagógus-előmeneteli fokozat: `GYAKORNOK`, `PEDAGOGUS_I`, `PEDAGOGUS_II`, `MESTERPEDAGOGUS`, `KUTATOTANAR` | Púétv. 96–97. § |
| `puetv_payCategory` | integer | igen | Fizetési kategória (1–16) – a szakmai gyakorlat ideje alapján | Púétv. 98. § |
| `puetv_baseSalary` | integer | igen | Illetmény az illetménytáblázat alapján | Púétv. 98. § + éves Korm. rendelet |
| `puetv_salaryDeviation` | decimal | nem | Illetményeltérítés mértéke (±20%) – 2025.09.01-től | Púétv. 99. § |
| `puetv_salaryDeviationValidFrom` | date | nem | Eltérítés érvényessége (szept. 1. – aug. 31.) | Púétv. 99. § |
| `puetv_opportunityAllowance` | integer | nem | Esélyteremtési illetményrész | Púétv. 100. § |
| `puetv_masterDegreeAllowance` | integer | nem | Mesterfokozat után járó illetménynövekedés | Púétv. 101. § |
| `puetv_subjectAllowance` | integer | nem | Egyes tantárgyak után járó illetménynövekedés | Púétv. 101. § |
| `puetv_qualificationDate` | date | nem | Utolsó minősítés dátuma | Púétv. – minősítési rendszer |
| `puetv_qualificationResult` | enum | nem | Minősítés eredménye (megfelelt / nem felelt meg / kiválóan megfelelt) | Púétv. |
| `puetv_nextQualificationDue` | date | nem | Következő kötelező minősítés esedékessége | Púétv. |
| `puetv_continuousTrainingCycleStart` | date | nem | Továbbképzési ciklus kezdete (7 éves) | Púétv. 114. § |
| `puetv_continuousTrainingCredits` | integer | nem | Szerzett továbbképzési pontszám az aktuális ciklusban | Púétv. 114. § |
| `puetv_teachingHoursPerWeek` | integer | nem | Heti kötelező tanítási óraszám (kötött munkaidőn belül) | Púétv. 88. § |
| `puetv_jubileeEntitlementDate` | date | nem | Köznevelési foglalkoztatotti jutalom esedékessége | Púétv. 108. § |

---

### 3.7. Eszjtv.-specifikus besorolás (egészségügyi szolgálati jogviszony)

*Feltétel: `employmentType` = `EGESZSEGUGYI`*

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `eszjtv_payStep` | integer | igen | Fizetési fokozat (az eü. szolgálati jogviszonyban töltött idő alapján) | Eszjtv. 8. § (8) |
| `eszjtv_baseSalary` | integer | igen | Illetmény a fizetési fokozat alapján | Eszjtv. 8–9. § + Korm. rendelet |
| `eszjtv_workerCategory` | enum | igen | Foglalkoztatott kategóriája: `EU_DOLGOZO` (eü. dolgozó, Eütev. 4.§ a)), `EU_BEN_DOLGOZO` (eü.-ben dolgozó, Eütev. 4.§ b)), `REZIDENS` | Eszjtv. 1. § (3) |
| `eszjtv_onCallRate` | decimal | nem | Ügyeleti díj mértéke | 528/2020. Korm. r.; 3/2023. OKFŐ utasítás |
| `eszjtv_standbyRate` | decimal | nem | Készenléti díj mértéke | Eszjtv.; 528/2020. Korm. r. |
| `eszjtv_voluntaryOvertimeConsent` | boolean | nem | Önként vállalt többletmunka vállalása | Eszjtv. – Eütev. 12/D. § |
| `eszjtv_additionalEmploymentPermit` | boolean | nem | További jogviszony engedélyezése | Eszjtv. 4. § – engedélyköteles |
| `eszjtv_additionalEmploymentPermitDate` | date | nem | Engedély dátuma | Eszjtv. 4. § |
| `eszjtv_qualificationAllowance` | integer | nem | Képesítési pótlék (további szakképesítés, doktori fokozat) | 3/2023. OKFŐ utasítás 13–16. pont |
| `eszjtv_leaderCategory` | enum | nem | Magasabb vezetői / vezetői megbízás | 528/2020. Korm. r. 5. § |
| `eszjtv_leaderAllowance` | integer | nem | Vezetői juttatás | 3/2023. OKFŐ utasítás 17. pont |
| `eszjtv_chamberMembership` | string | nem | Kamarai tagsági szám (MOK, MESZK) | Eütev. – kamarai tagság kötelező |

---

### 3.8. Kit./Kttv./Küt.-specifikus besorolás (kormányzati és közszolgálat)

*Feltétel: `employmentType` ∈ {`KORMANYZATI`, `KOZSZOLGALATI`, `KULONLEGES`}*

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `gov_positionCategory` | enum | igen | Álláshely besorolási kategóriája | Kit. 81–83. §; Kttv.; Küt. 14–16. § |
| `gov_salaryBandMin` | integer | nem | Illetménysáv alsó határa | Kit. – sávos illetmény |
| `gov_salaryBandMax` | integer | nem | Illetménysáv felső határa | Kit. |
| `gov_actualSalary` | integer | igen | Tényleges illetmény (a sávon belül) | Kit.; Kttv. |
| `gov_salarySupplementPercent` | decimal | nem | Illetménykiegészítés (%) | Kttv. – pl. felsőfokú végzettség után |
| `gov_languageAllowance` | integer | nem | Nyelvpótlék összege | Kttv. 141. § |
| `gov_civilServiceExamPassed` | boolean | nem | Közigazgatási szakvizsga letéve | Kit.; Kttv. – egyes pozíciókhoz kötelező |
| `gov_civilServiceExamDate` | date | nem | Szakvizsga dátuma | Kit.; Kttv. |
| `gov_civilServiceExamDeadline` | date | nem | Szakvizsga letételének határideje | Kit.; Kttv. |
| `gov_oathDate` | date | nem | Eskütétel dátuma | Kit. 38. §; Kttv.; Küt. |
| `gov_assetDeclarationDue` | boolean | nem | Vagyonnyilatkozat-tételi kötelezettség fennáll-e | Kit.; Kttv. |
| `gov_assetDeclarationLastDate` | date | nem | Utolsó vagyonnyilatkozat dátuma | Kit.; Kttv. |
| `gov_performanceRating` | enum | nem | Utolsó teljesítményértékelés eredménye (kivételes, jó, megfelelő, nem megfelelő) | Kit.; Kttv. |
| `gov_performanceRatingDate` | date | nem | Utolsó teljesítményértékelés dátuma | Kit.; Kttv. |
| `gov_trainingCreditsRequired` | integer | nem | Tárgyévi továbbképzési kötelezettség (tanulmányi pont) | Kit.; Kttv.; 499/2021. Korm. r. |
| `gov_trainingCreditsCompleted` | integer | nem | Teljesített tanulmányi pontok | Kit.; Kttv. |

---

### 3.9. Jogviszony megszűnése / megszüntetése

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `terminationType` | enum | feltételes | Megszűnés/megszüntetés jogcíme | Mt. 63–86. §; Kjt.; Púétv. stb. |
| `terminationDate` | date | feltételes | Jogviszony utolsó napja | – |
| `terminationInitiator` | enum | feltételes | Ki kezdeményezte (munkáltató / munkavállaló / közös megegyezés / törvény erejénél fogva) | – |
| `noticePeriodStart` | date | nem | Felmondási / felmentési idő kezdete | Mt. 68–70. §; Kjt.; Púétv. 50–52. § |
| `noticePeriodEnd` | date | nem | Felmondási / felmentési idő vége | – |
| `noticePeriodExemptionStart` | date | nem | Munkavégzés alóli felmentés kezdete | Mt. 70. § (2); Kjt.; Púétv. |
| `severancePayEntitlement` | boolean | nem | Végkielégítésre jogosult-e | Mt. 77. §; Kjt.; Púétv. |
| `severancePayAmount` | integer | nem | Végkielégítés összege | Mt. 77. §; jogviszonyban töltött idő alapján |

**A `terminationType` enum lehetséges értékei:**

| Csoport | Értékek | Alkalmazandó jogviszonyoknál |
|---|---|---|
| **Megszűnés** (automatikus) | `EXPIRY` (határozott idő lejárta), `DEATH`, `EMPLOYER_DISSOLUTION` (jogutód nélküli megszűnés), `STATUTORY` (törvény erejénél fogva) | Mind |
| **Munkavállalói megszüntetés** | `EMPLOYEE_RESIGNATION` (felmondás), `EMPLOYEE_IMMEDIATE` (azonnali hatályú), `CIVIL_SERVANT_RESIGNATION` (lemondás) | Mt.; Kjt./Kit./Kttv./Küt./Púétv. |
| **Munkáltatói megszüntetés** | `EMPLOYER_TERMINATION` (felmondás), `EMPLOYER_IMMEDIATE` (azonnali hatályú), `DISMISSAL` (felmentés) | Mt.; Kjt./Kit./Kttv./Küt./Púétv. |
| **Közös** | `MUTUAL_AGREEMENT` (közös megegyezés) | Mind |
| **Specifikus** | `PROBATION_TERMINATION` (próbaidő alatti azonnali hatály), `TRANSFER` (áthelyezés), `INCOMPATIBILITY` (összeférhetetlenség) | Jogviszony-függő |

---

### 3.10. Felmondási / felmentési védelem

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `protectionStatus` | array[enum] | nem | Aktuálisan fennálló felmondási/felmentési védelem | Mt. 65. § (3); Kjt.; Púétv. |

**Lehetséges védelmi státuszok:**

| Kód | Leírás | Jogszabály |
|---|---|---|
| `PREGNANCY` | Várandósság | Mt. 65. § (3) a) |
| `MATERNITY_LEAVE` | Szülési szabadság | Mt. 65. § (3) b) |
| `PARENTAL_LEAVE` | Gyermek gondozása céljából igénybe vett fizetés nélküli szabadság | Mt. 65. § (3) c) |
| `PATERNITY_LEAVE` | Apasági szabadság | Mt. 65. § (3) – 2025-ös módosítás |
| `CHILDCARE_ABSENCE` | Gyermek ápolása miatti keresőképtelenség | Mt. 65. § (3) d) |
| `MILITARY_SERVICE` | Önkéntes tartalékos katonai szolgálat | Mt. 65. § (3) e) |
| `REHABILITATION` | Rehabilitációs ellátás | Mt. 65. § (3) |
| `PRE_RETIREMENT` | Öregségi nyugdíjkorhatár előtti 5 év | Mt. 66. § (4); Kjt.; Kttv. |

**Megjegyzések:**
- A védelmi státuszok **automatikusan kalkulálhatók** a Person (birthDate → PRE_RETIREMENT), Leave (szülési szabadság, fizetés nélküli szabadság) és egyéb entitásokból.
- A rendszernek figyelmeztetnie kell a HR ügyintézőt, ha védett személyre vonatkozó megszüntetési műveletet próbál indítani.

---

### 3.11. Audit és metaadatok

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `status` | enum | igen | Jogviszony állapota: `DRAFT`, `ACTIVE`, `SUSPENDED`, `NOTICE_PERIOD`, `TERMINATED` |
| `createdAt` | datetime | igen | Rekord létrehozásának időpontja |
| `createdBy` | string | igen | Létrehozó felhasználó |
| `updatedAt` | datetime | igen | Utolsó módosítás időpontja |
| `updatedBy` | string | igen | Utolsó módosító felhasználó |
| `dataSource` | enum | nem | Adatforrás (manuális, migráció, integráció) |

---

## 4. Historikus adatkezelés

### 4.1. Jogviszony-szintű változások (EmploymentHistory)

Az alábbi property-k változásakor **új historikus rekord** keletkezik:

| Változás típusa | Érintett property-k | Tipikus eset |
|---|---|---|
| Munkakör-változás | positionId, jobTitle, feorCode | Belső áthelyezés, előléptetés |
| Szervezeti egység változás | orgUnitId | Szervezeti átrendezés |
| Munkaidő-változás | weeklyHours, dailyHours, isFullTime | Részmunkaidőre váltás, visszaváltás |
| Munkarend-változás | workScheduleType, workTimeFramePeriod | Egyenlőtlen munkarend bevezetése |
| Munkavégzési hely változás | workPlace, workPlaceType | Telephely-változás, távmunkára váltás |
| Besorolás-változás | Bármely jogviszony-specifikus besorolási mező | Átsorolás, előmenetel, fokozatváltás |

**Megvalósítási minta:**

```
EmploymentHistory {
  employmentId: UUID (FK → Employment)
  fieldGroup: enum
  validFrom: date
  validTo: date (null = aktuális)
  previousValue: JSON
  newValue: JSON
  changeReason: enum (PROMOTION, RECLASSIFICATION, ORG_CHANGE, SCHEDULE_CHANGE, LEGAL_CHANGE, CORRECTION)
  legalBasis: string (pl. "Kjt. 64. § automatikus fokozatlépés")
  changedAt: datetime
  changedBy: string
}
```

### 4.2. Automatikus események (naptár-alapú)

A rendszernek az alábbi eseményeket **automatikusan** kell generálnia vagy figyelmeztetést küldenie:

| Esemény | Forrás | Jogszabály |
|---|---|---|
| Fizetési fokozat lépés (Kjt.) | `seniorityDate` + `kjt_payStep` | Kjt. 62. § – 3 évente automatikus |
| Fizetési fokozat lépés (Eszjtv.) | `seniorityDate` + `eszjtv_payStep` | Eszjtv. 8. § |
| Fizetési kategória lépés (Púétv.) | `seniorityDate` + `puetv_payCategory` | Púétv. 98. § |
| Jubileumi jutalom esedékessége | `seniorityDate` → 25/30/35/40 év | Kjt. 78. § |
| Köznevelési foglalkoztatotti jutalom | Púétv. szerinti szakmai gyakorlat | Púétv. 108. § |
| Próbaidő lejárta | `probationEndDate` | Minden jogviszony-típus |
| Határozott idő lejárta | `fixedTermEndDate` | Mt. 192. § |
| Közigazgatási szakvizsga határidő | `gov_civilServiceExamDeadline` | Kit.; Kttv. |
| Továbbképzési ciklus lejárta | `puetv_continuousTrainingCycleStart` + 7 év | Púétv. 114. § |
| Minősítés esedékessége | `puetv_nextQualificationDue` | Púétv. |
| Okmány lejárat (kamarai tagság) | `eszjtv_chamberMembership` | Eütev. |
| Nyugdíjkorhatár elérése | `Person.birthDate` + jogszabályi nyugdíjkorhatár | Mt. 66. § (4) – védelmi idő |
| Vagyonnyilatkozat esedékessége | `gov_assetDeclarationLastDate` + ciklus | Kit.; Kttv. |

---

## 5. GDPR adatkezelési kategorizáció

| Kategória | Property-k | Jogalap | Megőrzési idő |
|---|---|---|---|
| **Jogviszony alapadatok** | employmentType, startDate, endDate, contractDuration, positionId, jobTitle, workPlace | Szerződés (6(1)b) + Jogi kötelezettség (6(1)c) | Jogviszony megszűnése + 5 év (Art.) |
| **Besorolási adatok** | Összes jogviszony-specifikus besorolási mező | Jogi kötelezettség (6(1)c) | Jogviszony megszűnése + 5 év |
| **Munkaidő-paraméterek** | weeklyHours, dailyHours, workScheduleType | Jogi kötelezettség (6(1)c) – Mt. 134. § | Jogviszony megszűnése + 3 év (munkaügyi per elévülés) |
| **Megszüntetési adatok** | terminationType, terminationDate, severancePayAmount | Jogi kötelezettség (6(1)c) | Jogviszony megszűnése + 5 év; peres ügy esetén annak lezárásáig |
| **Védelmi státuszok** | protectionStatus | Jogi kötelezettség (6(1)c) | Védelem fennállása + 3 év |
| **Teljesítményértékelés** | gov_performanceRating, puetv_qualificationResult | Jogi kötelezettség (6(1)c) + Jogos érdek (6(1)f) | Következő értékelésig + 3 év |
| **Vagyonnyilatkozat** | gov_assetDeclarationDue, gov_assetDeclarationLastDate | Jogi kötelezettség (6(1)c) | Jogszabályi megőrzési idő |
| **Audit adatok** | createdAt/By, updatedAt/By, EmploymentHistory | Elszámoltathatóság (GDPR 5(2)) | Az érintett adat megőrzési idejével megegyező |

---

## 6. Üzleti szabályok és validációk

### 6.1. Általános szabályok

| Szabály | Leírás | Jogszabály |
|---|---|---|
| Jogviszony-típus és munkáltató összhangja | A munkáltató szektora meghatározza az alkalmazható jogviszony-típusokat (pl. önkormányzati hivatal → `KOZSZOLGALATI`, versenyszféra → `MUNKAVISZONY`) | Jogállási törvények hatályi rendelkezései |
| Határozott idő maximuma | Mt. hatály alatt max. 5 év (meghosszabbítás + láncolás együtt) | Mt. 192. § (2) |
| Próbaidő maximuma | Mt.: max. 3 hó; Eszjtv.: kötelező 3 hó, max. 4 hó; próbaidő nem hosszabbítható | Mt. 45. § (5); Eszjtv. 3. § |
| Minimális munkaidő | Részmunkaidő: nincs minimum; napi munkaidő max. 12 óra (munkaidőkeretben) | Mt. 99. § |
| Munkaviszony létesítési életkor | Főszabály: 16 éves kortól; nappali tagozatos 15 éves iskolai szünetben | Mt. 34. § |

### 6.2. Besorolási szabályok

| Szabály | Leírás | Jogszabály |
|---|---|---|
| Kjt. fizetési osztály | A betöltött munkakörhöz előírt iskolai végzettség határozza meg (A–J) | Kjt. 61. § |
| Kjt. fizetési fokozat | A közalkalmazotti jogviszonyban töltött idő alapján automatikusan lép; 3 évente | Kjt. 62. § |
| Púétv. fokozatváltás | Minősítés alapján; Gyakornok → Ped.I kötelező minősítés; további fokozatoknál pályázat | Púétv. 96–97. § |
| Eszjtv. fizetési fokozat | Eü. szolgálati jogviszonyban töltött idő alapján; beszámítható korábbi jogviszonyok listája az Eszjtv. 8. § (9)-ben | Eszjtv. 8. § (8)–(9) |
| Kit./Kttv. besorolás | Álláshely besorolási kategóriája + sávos illetmény a sávon belül | Kit. 81–83. § |

---

## 7. Integrációs pontok

| Külső rendszer | Irány | Érintett property-k | Cél |
|---|---|---|---|
| **NAV (T1041)** | kimenő | employmentType, startDate, endDate, feorCode, weeklyHours, isFullTime, workPlace | Biztosítotti bejelentés, változás-bejelentés, kijelentés |
| **NAV (08-as bevallás)** | kimenő | Besorolási adatok → bérszámfejtésen keresztül | Havi adó- és járulékbevallás |
| **KIR (Köznevelési Információs Rendszer)** | kétirányú | puetv_careerStage, puetv_payCategory, puetv_qualificationResult | Pedagógus nyilvántartás |
| **OKFŐ (egészségügy)** | kimenő | eszjtv_* adatok | Egészségügyi szolgáltatói nyilvántartás |
| **MÁK (kincstár)** | kétirányú | Besorolási adatok, illetmény | Költségvetési szervek bérszámfejtése |
| **Bérszámfejtő modul** | kétirányú | Minden besorolási és munkaidő-adat | Illetmény-/bérszámfejtés |
| **Szabadság-nyilvántartó modul** | kimenő | weeklyHours, dailyHours, employmentType, seniorityDate | Szabadságjogosultság számítás |

---

## 8. Kapcsolódó entitások

```
Person 1 ──── N Employment         (egy személy több jogviszonnyal)
Organization 1 ── N Employment     (egy szervezetnél több jogviszony)
OrgUnit 1 ──── N Employment        (szervezeti egységhez rendelve)
Position 1 ──── N Employment       (munkakör / álláshely)

Employment 1 ──── N Compensation         (javadalmazás-elemek)
Employment 1 ──── N TimeRecord           (munkaidő-nyilvántartás)
Employment 1 ──── N LeaveEntitlement     (szabadság-jogosultságok)
Employment 1 ──── N LeaveRequest         (szabadság-igénylések)
Employment 1 ──── N PerformanceReview    (teljesítményértékelések)
Employment 1 ──── N TrainingRecord       (továbbképzési nyilvántartás)
Employment 1 ──── N EmploymentHistory    (historikus változások)
Employment 1 ──── N EmploymentDocument   (munkaszerződés, kinevezés, módosítások)
```

---

## 9. Nyitott kérdések

1. **Egyidejű jogviszonyok korlátozása:** Hogyan kezelje a rendszer, ha az Eszjtv. hatálya alatti dolgozó engedély nélkül létesít további jogviszonyt? Validáció vagy csak figyelmeztetés?
2. **Jogviszony-átalakulás:** A korábbi közalkalmazotti jogviszonyok köznevelési foglalkoztatotti vagy eü. szolgálati jogviszonnyá alakultak. A migrációnál az eredeti jogviszony lezárása + új létrehozása, vagy az eredeti módosítása a javasolt megközelítés?
3. **Kirendelés és áthelyezés:** Külön entitás (Secondment) vagy az Employment módosításaként kezeljük?
4. **Megbízási/vállalkozási szerződések:** A nem munkaviszony jellegű foglalkoztatási formákat (megbízási, vállalkozási) kezeli-e a rendszer, vagy azok kívül esnek az HRMS hatókörén?
5. **Vegyes munkáltató:** Ha egy szervezet pl. közalkalmazottat és munkaviszonyban álló dolgozót is foglalkoztat (pl. önkormányzat: Kttv. + Mt.), a szervezet szintjén vagy a jogviszony szintjén kezeljük az eltérést?
6. **Illetmény vs. bér elhelyezése:** A jelenlegi modell a besorolási adatokat (garantált illetmény, fokozat) az Employment-en tartja, a tényleges kifizetés-elemeket (pótlékok, juttatások, levonások) a Compensation entitáson. Ez a szétválasztás megfelelő, vagy szükséges-e az illetmény-alapadatokat is a Compensation-be mozgatni?
7. **Közfoglalkoztatotti jogviszony:** Szükséges-e a közfoglalkoztatotti jogviszonyt is támogatni (1993. évi III. tv. + Mt. speciális szabályok)?

---

### 9.1. Javaslatok és válaszok

#### 9.1.1. Egyidejű jogviszonyok korlátozása

**Probléma:** Eszjtv. 4. § szerint további jogviszony csak a munkáltató előzetes engedélyével létesíthető. Hogyan kezelje ezt a rendszer?

**Javaslat: Többszintű validáció + workflow**

**1. Validációs szintek:**

```
a) PREVENTÍV ellenőrzés (Employment létrehozásakor):
   - Ha Person-nek van aktív Employment-je WHERE employmentType = 'EGESZSEGUGYI'
   - ÉS új Employment.employmentType = 'EGESZSEGUGYI' VAGY egyéb jogviszony
   - AKKOR ellenőrzi: eszjtv_additionalEmploymentPermit = TRUE
   - Ha FALSE → BLOKKOLJA a mentést, hibaüzenet:
     "Eszjtv. 4. § - További jogviszony csak előzetes engedéllyel létesíthető"

b) FIGYELMEZTETŐ ellenőrzés (nem Eszjtv. jogviszonyoknál):
   - Mt./Kjt./Kit./Kttv./Púétv. esetén CSAK figyelmeztetés
   - "A munkavállalónak már van aktív jogviszonya.
      Ellenőrizze az összeférhetetlenségi szabályokat!"

c) RIPORT ellenőrzés (compliance monitoring):
   - Napi/heti riport: Eszjtv. dolgozók, akiknek >1 aktív jogviszonyuk van
   - Jelzi, ha engedély hiányzik vagy lejárt
```

**2. Employment entitás bővítése:**

```
Employment {
  ...
  // Általános mezők (minden jogviszony-típusnál)
  additionalEmploymentPermitRequired: boolean (számított mező)
  additionalEmploymentPermitGranted: boolean
  additionalEmploymentPermitDate: date
  additionalEmploymentPermitValidUntil: date
  additionalEmploymentPermitDocumentId: UUID FK → Document

  // Eszjtv.-specifikus (már létező)
  eszjtv_additionalEmploymentPermit: boolean
  eszjtv_additionalEmploymentPermitDate: date
}
```

**3. Üzleti szabály:**

| Jogviszony típus | Többes jogviszony szabálya | Rendszer viselkedés |
|---|---|---|
| **Eszjtv.** | Előzetes engedély KÖTELEZŐ (Eszjtv. 4. §) | **Blokkoló validáció** - nem menthető engedély nélkül |
| **Kit./Kttv./Küt.** | Bejelentési kötelezettség, összeférhetetlenség vizsgálata | **Figyelmeztető** - menthető, de jelzi |
| **Kjt.** | Ágazati szabályok (pl. oktatás: max. 1.5 állás) | **Figyelmeztető** + szabály konfiguráció szervezetenként |
| **Púétv.** | Bejelentési kötelezettség (max. heti 40 óra összesen) | **Validáció**: összesített heti óraszám ≤ 40 |
| **Mt.** | Nincs korlátozás (versenytilalom külön vizsgálandó) | **Nincs validáció** |

**4. Workflow támogatás:**

```
AdditionalEmploymentRequest {
  id: UUID PK
  personId: UUID FK
  existingEmploymentId: UUID FK (jelenlegi jogviszony)
  proposedEmploymentType: enum
  proposedEmployer: string
  proposedWeeklyHours: decimal
  requestDate: date
  approvalStatus: enum (pending, approved, rejected)
  approvedBy: UUID FK (vezető/HR)
  approvalDate: date
  rejectionReason: text
  validUntil: date (engedély érvényessége)
}
```

**Ajánlás:**
- **MVP:** Preventív blokkoló validáció Eszjtv.-nél, figyelmeztető máshol
- **Extended:** AdditionalEmploymentRequest workflow, automatikus riportok
- **Strategic:** Összeférhetetlenség-elemző motor (vagyonnyilatkozat, közpénz-felhasználás keresztellenőrzése)

---

#### 9.1.2. Jogviszony-átalakulás (migráció vs. kontinuitás)

**Probléma:** Kjt. jogviszony → Púétv. vagy Eszjtv. átalakulás (2013, 2015). Hogyan kezeljük historikusan?

**Javaslat: Lezárás + új jogviszony + átmeneti kapcsolat**

**Indokok:**
1. **Jogszabályi érv:** A jogviszony MEGSZŰNT (Kjt.) és ÚJ jogviszony LÉTREJÖTT (Púétv./Eszjtv.)
2. **Besorolási rendszer eltérése:** Kjt. fizetési osztály/fokozat ≠ Púétv. fokozat/kategória ≠ Eszjtv. fizetési fokozat
3. **Jogosultságok eltérése:** Szabadság, pótlékok, illetményszámítás szabályai eltérőek
4. **Audit követelmény:** Világosan látható kell legyen, hogy az átalakulás MIKOR történt

**Megvalósítás:**

```
Scenario: Közalkalmazott pedagógus átalakulása (2013. szept. 1.)

1) EREDETI Employment (Kjt.):
   {
     id: emp-001
     employmentType: KOZALKALMAZOTT
     startDate: 2005-09-01
     endDate: 2013-08-31  ← LEZÁRÁS
     terminationType: LEGAL_TRANSFORMATION
     terminationNote: "Átalakulás köznevelési foglalkoztatottá (Púétv.)"
     status: TERMINATED

     // Besorolási adatok megőrzése
     kjt_payGrade: E
     kjt_payStep: 8
     kjt_guaranteedSalary: 180000
   }

2) ÚJ Employment (Púétv.):
   {
     id: emp-002
     employmentType: KOZNEVELESI
     startDate: 2013-09-01
     endDate: null
     status: ACTIVE

     // Besorolási adatok (új rendszer)
     puetv_careerStage: PEDAGOGUS_I
     puetv_payCategory: 8
     puetv_baseSalary: 195000

     // KONTINUITÁS biztosítása
     seniorityDate: 2005-09-01  ← MEGMARAD!
     previousServiceYears: 0  (mert már a seniorityDate-ben benne van)

     // Kapcsolat az előző jogviszonyhoz
     transformedFromEmploymentId: emp-001
   }

3) EmploymentTransformation kapcsolótábla (opcionális, Extended verzió):
   {
     id: trans-001
     previousEmploymentId: emp-001
     newEmploymentId: emp-002
     transformationType: LEGAL_TRANSFORMATION
     transformationDate: 2013-09-01
     legalBasis: "Púétv. Záró rendelkezések 5. §"
     carryOverRules: JSON {
       "seniorityDate": "preserved",
       "annualLeaveEntitlement": "recalculated",
       "jubileeEntitlement": "preserved"
     }
   }
```

**Alternatív megközelítés - NEM ajánlott:**

```
ELVETETT verzió: Employment módosítása

Employment {
  employmentType: KOZALKALMAZOTT → KOZNEVELESI  ❌ PROBLÉMA!
  // Melyik besorolási mező érvényes?
  kjt_payGrade: E  ← Releváns volt 2005-2013-ig
  puetv_careerStage: PEDAGOGUS_I  ← Releváns 2013-tól

  // Historikus lekérdezésnél: melyik illetményt használjuk 2012-ben?
  // → Komplex időbeli szűrés szükséges minden besorolási mezőhöz
}

→ Az EmploymentHistory entitás NEM elégséges, mert:
   - A besorolási RENDSZER változott, nem csak az értékek
   - Jogosultsági szabályok eltérőek (szabadság, jubileum, pótlékok)
```

**Migráció lépései:**

```sql
-- 1. Lezárjuk a régi jogviszonyt
UPDATE employment
SET endDate = '2013-08-31',
    terminationType = 'LEGAL_TRANSFORMATION',
    status = 'TERMINATED'
WHERE employmentType = 'KOZALKALMAZOTT'
  AND organization_sector = 'education'
  AND startDate < '2013-09-01';

-- 2. Létrehozzuk az új jogviszonyt
INSERT INTO employment (
  personId,
  employmentType,
  startDate,
  seniorityDate,  -- ← MEGŐRIZZÜK!
  puetv_careerStage,
  puetv_payCategory,
  transformedFromEmploymentId
)
SELECT
  personId,
  'KOZNEVELESI',
  '2013-09-01',
  seniorityDate,  -- ← Kontinuitás
  map_kjt_to_puetv_stage(kjt_payGrade),
  kjt_payStep,  -- Kezdetben azonos kategória
  id AS transformedFromEmploymentId
FROM employment
WHERE ...;

-- 3. Frissítjük a kapcsolódó entitásokat
UPDATE compensation_element
SET employmentId = new_employment_id
WHERE employmentId IN (old_employment_ids);
```

**GDPR megfelelés:**

- **Jogalap:** GDPR 6(1)(c) - jogi kötelezettség (jogviszony nyilvántartás)
- **Megőrzési idő:** Mindkét jogviszony (régi + új) adatait meg kell őrizni a jogviszony megszűnése + 5 év ideig
- **Lekérdezhetőség:** Az érintett munkavállaló mindkét jogviszony adataihoz hozzáférhet (GDPR 15. cikk)

**Ajánlás:**
- **MVP:** Lezárás + új Employment létrehozása, `transformedFromEmploymentId` mező
- **Extended:** EmploymentTransformation entitás részletes átmeneti szabályokkal
- **Migráció:** SQL script automatikus átkonverzióhoz, mapping táblákkal (Kjt. → Púétv./Eszjtv.)

---

#### 9.1.3. Kirendelés és áthelyezés

**Javaslat: Külön Assignment entitás (ajánlott)**

**Indokok:**

1. **Jogviszony NEM változik** - Az eredeti munkáltató marad a jogviszony alanya
2. **Bérszámfejtés** - Továbbra is az eredeti munkáltató fizet
3. **Munkaügyi felelősség** - Megosztott (Mt. 53-53/C. §, Kjt. 41-42. §)
4. **Több kirendelés egyidejűleg** - Lehetséges (ritka, de előfordul)

**Megvalósítás:**

```
Assignment {
  id: UUID PK
  employmentId: UUID FK  (eredeti jogviszony - NEM változik)
  assignmentType: enum (secondment, posted_worker, temporary_assignment, substitution)

  // Fogadó szervezet adatai
  hostOrganizationId: UUID FK (ha ugyanabban az HRMS rendszerben van)
  hostOrganizationName: string (ha külső)
  hostOrgUnitId: UUID FK
  hostSiteId: UUID FK (munkavégzés helye)
  hostSupervisorId: UUID FK (Employment - fogadó szervezetnél a vezető)
  hostPositionTitle: string (munkakör a fogadó szervezetnél)

  // Időbeli jellemzők
  assignmentStartDate: date
  assignmentEndDate: date (lehet null, ha határozatlan)
  expectedDuration: integer (hónapokban)

  // Pénzügyi rendezés
  costSharingType: enum (full_sender, full_host, shared, other)
  costSharingPercent: decimal (ha shared)
  costSharingAgreement: text

  // Jogi alap
  assignmentAgreementId: UUID FK → Document (kirendelési megállapodás)
  legalBasis: string (Mt. 53. §, Kjt. 41. §, Kit. 50. §, stb.)
  requiresEmployeeConsent: boolean (Mt. 53. § - munkavállaló beleegyezése szükséges)
  employeeConsentDate: date
  employeeConsentDocumentId: UUID FK → Document

  // Munkaügyi adatok
  workScheduleAtHost: UUID FK → WorkScheduleTemplate (ha eltér)
  supervisoryResponsibility: enum (sender, host, shared)
  performanceReviewResponsibility: enum (sender, host, joint)

  // Státusz
  status: enum (planned, active, extended, completed, terminated, cancelled)
  terminationReason: string
  actualEndDate: date

  // Megjegyzések
  notes: text
  createdAt: datetime
  createdBy: string
}
```

**Assignment vs. Employment módosítás:**

| Szempont | Assignment entitás (AJÁNLOTT) | Employment módosítás (NEM ajánlott) |
|---|---|---|
| **Jogviszony alanya** | Megmarad az eredeti munkáltató ✅ | Átkerül a fogadó munkáltatóhoz ❌ |
| **Bérszámfejtés** | Eredeti munkáltató ✅ | Fogadó munkáltató ❌ |
| **Visszahelyezés** | Assignment.status = completed ✅ | Employment újabb módosítása ❌ |
| **Audit trail** | Világos (Assignment entitás) ✅ | EmploymentHistory elemzése szükséges ❌ |
| **Több kirendelés** | Lehetséges (több Assignment rekord) ✅ | NEM lehetséges ❌ |
| **Jogszabályi megfelelés** | Mt. 53. § logikája ✅ | Félrevezető ❌ |

**Különleges esetek:**

```
1) VÉGLEGES ÁTHELYEZÉS (nem kirendelés):
   - Employment.orgUnitId módosítása
   - EmploymentHistory rögzíti a változást
   - Changeeason: TRANSFER

2) KIRENDELÉS (ideiglenes):
   - Employment NEM változik
   - Assignment entitás létrehozása
   - Assignment.status = active

3) HELYETTESÍTÉS (substitution):
   - Assignment.assignmentType = substitution
   - Assignment.hostPositionTitle = "Osztályvezető helyettes"
   - Időtartam: helyettesített személy távollétéig

4) KÜLFÖLDRE KIKÜLDÖTT MUNKAVÁLLALÓ (Mt. 196/B. §):
   - Assignment.assignmentType = posted_worker
   - Assignment.hostOrganizationName = "Német leányvállalat GmbH"
   - Külön A1 igazolás (Document) szükséges
```

**Kapcsolódó workflow igények:**

- **Kirendelés indítása:** Jóváhagyási workflow (eredeti vezető + HR + fogadó vezető)
- **Költségek elszámolása:** Automatikus költségmegosztás bérszámfejtéskor
- **Lejárat figyelés:** Értesítés 30 nappal a kirendelés vége előtt (meghosszabbítás vagy visszahelyezés)
- **Munkaidő-nyilvántartás:** DailyTimeRecord.assignmentId hivatkozás

**Ajánlás:**
- **MVP:** Assignment entitás (egyszerű verzió: hostOrganizationId, dátumok, status)
- **Extended:** Teljes Assignment entitás + költségmegosztás + workflow
- **Strategic:** Nemzetközi kiküldés támogatása (A1 igazolás, social security coordination)

---

#### 9.1.4. Megbízási/vállalkozási szerződések

**Javaslat: Igen, de külön employmentType + korlátozott property-készlet**

**Indokok:**

1. **Gyakorlati szükséglet:** Sok szervezet foglalkoztat megbízottakat (óraadó tanár, orvos szakértő, IT tanácsadó)
2. **Adózási kötelezettség:** NAV T1041 bejelentés szükséges (!) - megbízási díj is szerepel
3. **Statisztikai igény:** Teljes munkaerő-állomány nyilvántartása (FTE számításhoz)
4. **Bérszámfejtés integráció:** Megbízási díj is a payroll-ban kerül feldolgozásra

**NEM indokok:**
- ❌ Teljes HR folyamatok (szabadság, túlóra, teljesítményértékelés) → ezek NEM relevánsak
- ❌ Összetett besorolási rendszer → nincs fizetési fokozat megbízásnál

**Megvalósítás:**

```
Employment {
  employmentType: enum (
    MUNKAVISZONY,
    KOZALKALMAZOTT,
    KOZNEVELESI,
    EGESZSEGUGYI,
    KORMANYZATI,
    KOZSZOLGALATI,
    KULONLEGES,
    MEGBIZASI,        ← ÚJ
    VALLALKOZASI      ← ÚJ
  )

  // Megbízási/vállalkozási specifikus mezők
  contract_type: enum (civil_contract, service_contract, occasional_employment)
  contract_subject: text (megbízás tárgya)
  deliverable: text (teljesítendő feladat)
  fee_type: enum (fixed, hourly, per_deliverable)
  fee_amount: decimal
  invoicing_required: boolean (vállalkozási szerződésnél: igen)
  taxId: string (vállalkozónál: adószám)

  // NEM releváns mezők megbízásnál (null értékek):
  workScheduleType: null
  weeklyHours: null (nincs munkaidő)
  dailyHours: null
  probationStartDate: null (nincs próbaidő)
  seniorityDate: null (nincs jogviszonyban töltött idő)
  kjt_payGrade: null (nincs besorolás)
  ...
}
```

**Property-érvényességi mátrix (bővítve):**

| Property-csoport | Mt. | Kjt. | Púétv. | Eszjtv. | Kit. | Kttv. | Küt. | **Megbízási** | **Vállalkozási** |
|---|---|---|---|---|---|---|---|---|---|
| Alapadatok | ● | ● | ● | ● | ● | ● | ● | ● | ● |
| Munkaidő | ● | ● | ● | ● | ● | ● | ● | – | – |
| Besorolás | – | ● | ● | ● | ● | ● | ● | – | – |
| Megbízási díj | – | – | – | – | – | – | – | ● | ● |
| Szabadság | ● | ● | ● | ● | ● | ● | ● | – | – |
| Túlóra | ● | ● | ● | ● | ● | ● | ● | – | – |
| NAV T1041 bejelentés | ● | ● | ● | ● | ● | ● | ● | ● | ❌ (számlaalapú) |

**Alternatív megoldás (NEM ajánlott):**

```
Contractor entitás (külön az Employment-től)

Contractor {
  id: UUID PK
  personId: UUID FK
  organizationId: UUID FK
  contractType: enum (civil, service)
  ...
}

PROBLÉMA:
- Duplikált logika (Person ↔ Contractor vs. Person ↔ Employment)
- Bérszámfejtés integráció bonyolultabb
- NAV T1041 kétféle forrásból töltendő
```

**Validációs szabályok:**

```
IF employmentType IN ('MEGBIZASI', 'VALLALKOZASI'):
  - workScheduleType MUST BE null
  - weeklyHours MUST BE null
  - Compensation modul: csak "fee" típusú elem engedélyezett
  - Leave modul: NEM érhető el
  - TimeTracking modul: NEM érhető el (vagy opcionális óra-nyilvántartás projekthez)
  - PerformanceReview: opcionális (teljesítés-igazolás)
```

**NAV T1041 bejelentés:**

| Mezőnév (T1041) | Munkaviszony | Megbízási | Vállalkozási |
|---|---|---|---|
| Jogviszony jellege | 1 (munkaviszony) | 2 (megbízási) | – (nincs bejelentés) |
| FEOR kód | Kötelező | Kötelező | – |
| Foglalkoztatás kezdete | startDate | startDate | – |
| Díjazás összege | Bér | Megbízási díj | – (számla) |

**Scope korlátok:**

| Funkció | Munkaviszony | Megbízási/vállalkozási |
|---|---|---|
| Person törzsadat | ✅ Teljes | ✅ Teljes (adózási adatok nélkül) |
| Employment alapadatok | ✅ Teljes | ✅ Korlátozott (időtartam, díj) |
| Compensation | ✅ Komplex (bér + pótlékok) | ✅ Egyszerű (fix díj vagy óradíj) |
| TimeTracking | ✅ Kötelező (Mt. 134. §) | ❌ Nem releváns |
| Leave | ✅ Szabadság, betegszabadság | ❌ Nem releváns |
| Performance | ✅ Teljesítményértékelés | ◐ Teljesítés-igazolás (opcionális) |
| Payroll | ✅ Teljes (adó, járulék) | ✅ Egyszerűsített (SZJA, szocho) |

**Ajánlás:**
- **MVP:** `MEGBIZASI` employmentType, korlátozott property-készlet, NAV T1041 integráció
- **Extended:** `VALLALKOZASI` employmentType (számlaalapú, nincs T1041)
- **Out of scope:** Komplex vállalkozói nyilvántartás (ERP funkció)

---

#### 9.1.5. Vegyes munkáltató (szervezet vs. jogviszony szintű eltérés)

**Javaslat: Jogviszony szintű kezelés (employmentType per Employment)**

**Indokok:**

1. **Realitás:** Önkormányzat = Kttv. (tisztviselők) + Mt. (fizikai munkások, karbantartók)
2. **Jogszabályi követelmény:** Egy szervezet többféle jogállást is alkalmazhat
3. **Flexibilitás:** Szervezeti átstrukturálás esetén nem kell a teljes Organization entitást átírni

**Architektúra:**

```
Organization {
  id: org-001
  name: "Budapest Főváros XV. kerület Rákospalota-Újpest Önkormányzata"
  sector: "public"  ← Általános besorolás
  legalForm: "municipality"
  applicableEmploymentTypes: ['KOZSZOLGALATI', 'MUNKAVISZONY']  ← Engedélyezett típusok
}

Employment #1:
  organizationId: org-001
  employmentType: KOZSZOLGALATI  ← Közszolgálati tisztviselő
  positionId: pos-001 (jegyző)
  gov_positionCategory: A
  ...

Employment #2:
  organizationId: org-001
  employmentType: MUNKAVISZONY  ← Mt. dolgozó
  positionId: pos-002 (takarító)
  weeklyHours: 40
  ...
```

**Validációs szabályok:**

```javascript
// Employment létrehozásakor
function validateEmploymentType(employment, organization) {
  const allowedTypes = organization.applicableEmploymentTypes;

  if (!allowedTypes.includes(employment.employmentType)) {
    throw new ValidationError(
      `A(z) ${organization.name} szervezet nem alkalmazhat ${employment.employmentType} jogviszony-típust.
       Engedélyezett típusok: ${allowedTypes.join(', ')}`
    );
  }

  // Szektor-alapú ellenőrzés
  if (organization.sector === 'private' &&
      ['KOZALKALMAZOTT', 'KORMANYZATI', 'KOZSZOLGALATI'].includes(employment.employmentType)) {
    throw new ValidationError(
      'Versenyszférás szervezet nem alkalmazhat közszolgálati jogviszonyt'
    );
  }

  if (organization.sector === 'public' &&
      organization.legalForm === 'ministry' &&
      employment.employmentType !== 'KORMANYZATI') {
    throw new Warning(  // csak figyelmeztetés, nem blokkoló
      'Figyelem: Minisztérium jellemzően kormányzati szolgálati jogviszonyt alkalmaz (Kit.)'
    );
  }
}
```

**Organization.applicableEmploymentTypes konfigurálása:**

| Szervezet típusa | Tipikus applicableEmploymentTypes | Példa |
|---|---|---|
| Versenyszféra | `['MUNKAVISZONY']` | Kft., Zrt. |
| Minisztérium | `['KORMANYZATI', 'MUNKAVISZONY']` | Belügyminisztérium |
| Önkormányzat | `['KOZSZOLGALATI', 'MUNKAVISZONY']` | Budapest XV. ker. |
| Köznevelési intézmény | `['KOZNEVELESI', 'MUNKAVISZONY']` | Általános iskola (pedagógus + karbantartó) |
| Kórház | `['EGESZSEGUGYI', 'MUNKAVISZONY']` | Kórház (orvos + takarító) |
| Egyetem | `['KOZALKALMAZOTT', 'MUNKAVISZONY']` | Egyetem (oktató + adminisztrátor) |

**Jelentési igények:**

```sql
-- Szervezeti létszám-statisztika jogviszony-típusonként
SELECT
  o.name,
  e.employmentType,
  COUNT(*) AS headcount,
  SUM(CASE WHEN e.isFullTime THEN 1 ELSE e.weeklyHours/40 END) AS fte
FROM employment e
JOIN organization o ON e.organizationId = o.id
WHERE e.status = 'ACTIVE'
GROUP BY o.id, e.employmentType;
```

**ELVETETT alternatíva:**

```
Organization.defaultEmploymentType = 'KOZSZOLGALATI'  ❌ NEM ELÉG!

PROBLÉMA:
- Vegyes szervezetnél MELYIK a default?
- Mi van, ha 80% Mt., 20% Kttv.? → default félrevezető
```

**Ajánlás:**
- **MVP:** `employmentType` per Employment, `Organization.applicableEmploymentTypes` validációs lista
- **Extended:** Automatikus figyelmeztetések szektor-jogviszony inkonzisztenciánál
- **Strategic:** Reporting dashboard: létszám jogviszony-típusonként, szervezetenként

---

#### 9.1.6. Illetmény vs. bér elhelyezése (Employment vs. Compensation)

**Javaslat: Megtartjuk a jelenlegi szétválasztást (ajánlott)**

**Jelenlegi architektúra (helyes):**

```
Employment {
  // BESOROLÁSI ADATOK (státusz, jogviszony-jellemző)
  kjt_payGrade: E  (fizetési osztály)
  kjt_payStep: 8   (fizetési fokozat)
  kjt_guaranteedSalary: 180000  (garantált illetmény)

  puetv_careerStage: PEDAGOGUS_II
  puetv_payCategory: 10
  puetv_baseSalary: 220000

  // EZT MEGTARTJUK! ✅
}

CompensationElement {
  // TÉNYLEGES KIFIZETÉSI ELEMEK (tranzakciós adatok)
  employmentId: FK
  compensationTypeId: FK  (pl. "Garantált illetmény", "Túlórapótlék", "13. havi")
  amount: 180000
  validFrom: 2025-01-01
  validTo: null

  // PÓTLÉKOK, JUTTATÁSOK, LEVONÁSOK
  // Túlórapótlék, nyelvpótlék, vezetői pótlék, stb.
}
```

**MIÉRT helyes a szétválasztás?**

| Szempont | Employment (besorolás) | Compensation (kifizetés) |
|---|---|---|
| **Természet** | Státusz, jogállás | Tranzakciós adat |
| **Változás gyakorisága** | Ritkán (évente, besoroláskor) | Gyakran (havi bérszámfejtés, pótlékok) |
| **Időbeliség** | Historikus (EmploymentHistory) | Időszakos érvényességgel (validFrom/validTo) |
| **Jogszabályi hivatkozás** | Kjt. 61-62. § (besorolás) | Kjt. 66-75. § (illetmény-kiegészítések) |
| **Audit** | Jogviszony-módosítás | Bérszámfejtési tétel |
| **Lekérdezési logika** | "Milyen besorolású?" | "Mennyit kap havonta?" |

**Példa - Túlórapótlék esete:**

```
Employment (NEM változik):
  kjt_payGrade: E
  kjt_guaranteedSalary: 180000

CompensationElement (VÁLTOZIK havonta):
  compensationTypeId: "Túlórapótlék"
  amount: 25000  (januárban)
  amount: 18000  (februárban - kevesebb túlóra)
  validFrom: 2025-01-01
  validTo: 2025-01-31

→ Ha az illetményt a Compensation-be tennénk:
   - Minden hónapban új CompensationElement kell (garantált illetmény + pótlék)
   - A besorolási adatok (payGrade, payStep) elvesznek
   - Jogszabályi hivatkozások nehezebben értelmezhetők
```

**ELVETETT alternatíva:**

```
Compensation {
  baseSalary: 180000  ← Ez lenne a garantált illetmény
  payGrade: E         ← Duplikáció!
  payStep: 8          ← Duplikáció!
}

Employment {
  // Mi marad itt? Csak a munkakör?
}

PROBLÉMÁK:
❌ Besorolás és illetmény szétválik
❌ Audit trail: melyik változott - a besorolás vagy az illetmény?
❌ Jogszabályi ellenőrzés: honnan tudjuk, hogy a garantált illetmény megfelel-e a fokozatnak?
❌ Jelentések: "Hány E fizetési osztályú dolgozónk van?" → JOIN-olni kell a Compensation-nel
```

**Kompromisszumos megoldás (opcionális, Extended verzió):**

```
Employment {
  // Besorolási adatok (státusz) - MEGTARTJUK
  kjt_payGrade: E
  kjt_payStep: 8
  kjt_guaranteedSalary: 180000  ← Referencia érték (jogszabály szerinti minimum)
}

CompensationElement {
  // Tényleges illetmény-kifizetés
  compensationTypeId: "KJT_GARANTALT_ILLETMENY"
  amount: 180000
  calculatedFrom: {
    payGrade: "E",
    payStep: 8,
    formula: "kjt_salary_table[E][8]"
  }
  validFrom: 2025-01-01
}

→ Validáció:
   IF CompensationElement.calculatedFrom.payGrade != Employment.kjt_payGrade
      → WARNING: "Besorolás és illetmény-tétel inkonzisztens!"
```

**Ajánlás:**

1. ✅ **MEGTARTJUK** a jelenlegi szétválasztást:
   - Employment: besorolási adatok (payGrade, payStep, baseSalary - referencia)
   - Compensation: kifizetési elemek (garantált illetmény + pótlékok + juttatások)

2. **MVP:** Egyszerű szétválasztás
   - Employment.kjt_guaranteedSalary csak informatív (jogszabály szerinti minimum)
   - CompensationElement tartalmazza a tényleges kifizetést

3. **Extended:** Automatikus szinkronizálás
   - Ha Employment.kjt_payStep változik → automatikus CompensationElement generálás/módosítás
   - Validációs szabályok: CompensationElement.amount >= Employment.kjt_guaranteedSalary

4. **Strategic:** Illetményszámító motor
   - Besorolási adatok alapján kalkulálja a garantált illetményt
   - Jogszabály-változásnál (illetménytáblázat módosítás) automatikus újraszámítás

**Konklúzió:** A jelenlegi modell **HELYES**. A besorolás és a kifizetés két különböző absztrakciós szint - ezt a szétválasztást meg kell őrizni.

---

#### 9.1.7. Közfoglalkoztatotti jogviszony

**Javaslat: Igen, MVP-ben is szükséges lehet (célcsoporttól függően)**

**Jogszabályi háttér:**

- **Flt. (1991. évi IV. tv.)** 37/A-37/F. § - Közfoglalkoztatás
- **Mt.** speciális szabályok (egyszerűsített foglalkoztatás)
- **Startmunka program** (2011-2022 fáziskivezetés alatt)

**Indokok PRO:**

1. ✅ **Önkormányzati HRMS:** Sok önkormányzat alkalmaz(ott) közfoglalkoztatottakat
2. ✅ **Statisztikai igény:** Teljes munkaerő-állomány (közfoglalkoztatott is FTE)
3. ✅ **NAV T1041 bejelentés:** Kötelező (jogviszony jellege: közfoglalkoztatás)
4. ✅ **Bérszámfejtés:** Külön támogatási szabályok (100% állami támogatás részben)

**Indokok KONTRA:**

1. ❌ **Fáziskivezetés:** 2022-től fokozatos megszüntetés, 2025-re minimális létszám
2. ❌ **Speciális szabályok:** Komplex támogatási rendszer (már nem releváns)
3. ❌ **Célcsoport:** Főleg kis önkormányzatok (nagyvállalatok nem érintettek)

**Megvalósítás (ha szükséges):**

```
Employment {
  employmentType: KOZFOGLALKOZTATASI  ← ÚJ típus
  employmentSubType: enum (
    NATIONAL_PROGRAM,      // Nemzeti közfoglalkoztatási program
    MUNICIPAL_PROGRAM,     // Önkormányzati program
    TEMPORARY_PUBLIC_WORK  // Ideiglenes közmunka
  )

  // Közfoglalkoztatási specifikus mezők
  kf_programId: UUID FK → PublicWorkProgram
  kf_supportPercent: decimal (pl. 100% = teljes állami támogatás)
  kf_supportAmount: integer (havonta)
  kf_workCategory: enum (simple, skilled, social_purpose)
  kf_maxDuration: integer (hónapokban - pl. max 12 hónap)
  kf_isRenewable: boolean
  kf_renewalCount: integer

  // Egyszerűsített szabályok
  probationStartDate: null  (nincs próbaidő)
  weeklyHours: 40  (általában teljes munkaidő)
  workScheduleType: GENERAL
}
```

**Property-érvényességi mátrix (bővítve):**

| Property-csoport | Mt. | Kjt. | ... | **Közfoglalkoztatási** |
|---|---|---|---|---|
| Alapadatok | ● | ● | ... | ● |
| Próbaidő | ● | ● | ... | ❌ (Flt. - nincs próbaidő) |
| Besorolás | – | ● | ... | ❌ (nincs fokozat) |
| Illetmény | ● | ● | ... | ● (minimálbér vagy garantált bér) |
| Állami támogatás | – | – | ... | ● |
| Szabadság | ● | ● | ... | ● (Mt. szerint) |
| Munkaidő | ● | ● | ... | ● (általában 8 óra/nap) |

**NAV T1041 bejelentés:**

| Mező | Érték |
|---|---|
| Jogviszony jellege | 9 (közfoglalkoztatási jogviszony) |
| FEOR kód | Kötelező |
| Foglalkoztatás kezdete | startDate |
| Díjazás | Minimálbér vagy garantált bér |

**Scope döntés:**

| Szempont | MVP-ben | Extended-ben |
|---|---|---|
| **Célcsoport: önkormányzatok** | ✅ Igen | ✅ Igen |
| **Célcsoport: versenyszféra** | ❌ Nem | ❌ Nem |
| **Célcsoport: költségvetési szervek** | ◐ Ritkán | ✅ Igen |

**Ajánlás:**

1. **Célcsoport-függő:**
   - Ha az HRMS **önkormányzati szektornak** készül → **Igen, MVP-ben is**
   - Ha **versenyszférának** → **Nem szükséges**
   - Ha **általános célú** → **Extended fázisban**, konfiguráció alapján kapcsolható

2. **MVP implementáció (ha szükséges):**
   ```
   employmentType: KOZFOGLALKOZTATASI
   kf_supportPercent: decimal
   kf_maxDuration: integer
   ```

3. **Extended implementáció:**
   - Teljes PublicWorkProgram entitás
   - Támogatási számítás automatizálása
   - Integrációk (pl. MÁK, helyi önkormányzati rendszerek)

4. **Alternatíva (ha nem önkormányzati célcsoport):**
   ```
   employmentSubType: OCCASIONAL_EMPLOYMENT  (Mt. alkalmi munka)
   // Hasonló egyszerűsített szabályok, de Mt. szerint
   ```

**Konklúzió:** Célcsoporttól függ. Önkormányzati HRMS-nél MVP, egyébként Extended vagy Out of Scope.

---

### 9.2. Összefoglaló döntési mátrix

| Kérdés | Rövid válasz | Implementáció | Prioritás |
|---|---|---|---|
| **1. Egyidejű jogviszonyok** | Blokkoló validáció (Eszjtv.), figyelmeztető (mások) | `additionalEmploymentPermit` mezők + validációs szabályok | MVP (validáció), Extended (workflow) |
| **2. Jogviszony-átalakulás** | Lezárás + új Employment + transformedFromEmploymentId | Külön Employment rekordok, átmeneti kapcsolat | MVP |
| **3. Kirendelés** | Külön Assignment entitás (ajánlott) | Assignment entitás, Employment NEM változik | Extended |
| **4. Megbízási szerződések** | Igen, külön employmentType | `MEGBIZASI`, `VALLALKOZASI` típusok, korlátozott property-k | MVP (megbízási), Extended (vállalkozási) |
| **5. Vegyes munkáltató** | Jogviszony szintű (employmentType per Employment) | `Organization.applicableEmploymentTypes` validációs lista | MVP |
| **6. Illetmény elhelyezése** | Megtartjuk jelenlegi (Employment = besorolás, Compensation = kifizetés) | Nincs változtatás, jelenlegi architektúra helyes ✅ | MVP (nincs teendő) |
| **7. Közfoglalkoztatás** | Célcsoport-függő (önkormányzat: igen, versenyszféra: nem) | `KOZFOGLALKOZTATASI` employmentType + speciális mezők | MVP (ha önkormányzati), Extended (egyébként) |
