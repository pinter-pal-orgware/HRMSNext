# Position (Munkakör / Álláshely) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer  
> **Verzió:** 0.1 – első tervezet  
> **Utolsó frissítés:** 2025. február  
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md, employment-entity.md, compensation-entity.md, organization-entity.md

---

## 1. Az entitás célja és hatóköre

A **Position** entitás a szervezeten belüli munkakört vagy álláshelyet reprezentálja – azt a „helyet" a szervezetben, amelyet egy vagy több természetes személy betölt (Employment-en keresztül). A Position az Employment és a szervezeti struktúra (OrgUnit) közötti összekötő kapocs: meghatározza, **milyen feladatot, milyen feltételekkel, milyen besorolási keretben** kell ellátni.

A magyar jogban a „munkakör" és az „álláshely" fogalma jogviszony-típusonként eltérő tartalommal bír:

| Fogalom | Jogviszony | Jelentés | Jogszabály |
|---|---|---|---|
| **Munkakör** | Mt. (munkaviszony) | A munkaszerződés kötelező tartalmi eleme; a munkavállaló által ellátandó feladatok összessége | Mt. 45. § (1) |
| **Munkakör** | Kjt. (közalkalmazott) | Besorolás alapja; a fizetési osztályt a munkakör betöltéséhez előírt végzettség határozza meg | Kjt. 61–63. § |
| **Munkakör** | Púétv. (köznevelés) | Pedagógus-munkakör vagy nevelő-oktató munkát közvetlenül segítő munkakör; a pedagógus-előmenetel hatálya alá tartozás alapja | Púétv. 4–5. §; 88. § |
| **Munkakör** | Eszjtv. (egészségügy) | Egészségügyi szolgálati munkakör; az illetménytáblázat sorát határozza meg | Eszjtv. 8. §; 528/2020. Korm. r. |
| **Álláshely** | Kit. (kormányzati igazgatás) | Az álláshely-alapú személyügyi igazgatás alapegysége; besorolási kategória, illetménysáv és feladatleírás kötődik hozzá | Kit. 35–40. § |
| **Álláshely** | Kttv. (közszolgálat) | Hasonló a Kit.-hez, de önálló szabályozás | Kttv. |
| **Álláshely** | Küt. (különleges jogállású) | A szerv alaplétszámába tartozó álláshely; a besorolási kategóriát a szerv vezetője határozza meg | Küt. 14–16. § |

**Tervezési alapelvek:**

- **Munkakör ≠ személy:** A Position a szervezeti struktúra része, nem a személyé. Egy Position betöltetlen is lehet, és egy Position-t több személy is betölthet (pl. részmunkaidős megosztás).
- **Egységes modell, típus-specifikus kiterjesztéssel:** A „közös mag + jogviszony-specifikus kiterjesztés" mintát követjük, hasonlóan az Employment entitáshoz.
- **Kétszintű modell:** **PositionTemplate** (munkaköri sablon / munkakör-katalógus) és **Position** (konkrét álláshely a szervezetben). A sablon a munkakör általános jellemzőit tartalmazza (feladatok, követelmények, FEOR-kód), a Position a konkrét szervezeti egységhez rendelt, költségvetési szempontból tervezett helyet.
- **A besorolás hordozója:** A Position határozza meg az Employment besorolási kereteit (Kjt. fizetési osztály, Kit. besorolási kategória stb.).

---

## 2. Entitásstruktúra – áttekintés

```
PositionTemplate 1 ──── N Position              (egy sablon, több konkrét álláshely)
OrgUnit 1 ──── N Position                        (egy szervezeti egységben több álláshely)
Position 1 ──── N Employment                      (egy álláshelyen egy vagy több jogviszony)
Position N ──── N QualificationRequirement        (betöltési feltételek)
Position 1 ──── N PositionHistory                 (historikus változások)
```

---

## 3. PositionTemplate (Munkaköri sablon / Munkakör-katalógus)

A PositionTemplate a szervezetben alkalmazott munkakörök „típus-leírása" – nem kötődik konkrét szervezeti egységhez vagy személyhez.

### 3.1. Alapadatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `organizationId` | UUID (FK) | igen | Melyik szervezethez tartozik a sablon | Technikai |
| `code` | string | igen | Munkakör kódja (belső azonosító) | Belső szabályzat |
| `name` | string | igen | Munkakör megnevezése (pl. "Általános iskolai tanár", "Belgyógyász szakorvos", "Pénzügyi ügyintéző") | Mt. 45. §; Kjt.; Púétv.; Kit. |
| `feorCode` | string(6) | igen | FEOR-08 kód (Foglalkozások Egységes Osztályozási Rendszere) | KSH; T1041 bejelentés |
| `feorName` | string | nem | FEOR megnevezés | KSH |
| `iscoCode` | string | nem | ISCO-08 kód (nemzetközi besorolás) | EU statisztikai adatszolgáltatás |
| `applicableEmploymentTypes` | array[enum] | igen | Milyen jogviszony-típusokhoz használható ez a munkakör-sablon | Jogállási törvények |
| `category` | enum | igen | Munkakör kategória (lásd 3.2.) | – |
| `description` | text | nem | Munkaköri leírás (általános) | Belső szabályzat; SZMSZ |
| `isActive` | boolean | igen | Aktív sablon-e | – |

### 3.2. Munkakör-kategóriák (category)

| Kód | Megnevezés | Leírás | Tipikus jogviszony |
|---|---|---|---|
| `EXECUTIVE` | Felsővezető / magasabb vezető | Intézményvezető, főigazgató, kórházigazgató, jegyző | Minden jogviszony-típus |
| `MANAGER` | Vezető | Osztályvezető, tagintézmény-vezető, részlegvezető | Minden |
| `PROFESSIONAL` | Szakértő / szaktisztviselő | Diplomás szakmai munkakör (mérnök, jogász, orvos, pedagógus, köztisztviselő) | Minden |
| `ADMINISTRATOR` | Ügyintéző / ügykezelő | Adminisztratív munkakör | Minden |
| `SKILLED_WORKER` | Szakmunkás | Szakképzettséget igénylő fizikai munkakör | Mt.; Kjt. |
| `UNSKILLED_WORKER` | Segédmunkás / betanított munkás | Szakképzettséget nem igénylő fizikai munkakör | Mt.; Kjt. |
| `PEDAGOGUE` | Pedagógus | A Púétv. szerinti pedagógus-munkakör | Púétv.; Kjt. (Gyvt.) |
| `PEDAGOGUE_SUPPORT` | Nevelő-oktató munkát közvetlenül segítő | Pedagógiai asszisztens, gyógypedagógiai asszisztens, dajka (ha NOKS) | Púétv. |
| `HEALTH_PROFESSIONAL` | Egészségügyi dolgozó (szakképzett) | Orvos, szakorvos, ápoló, gyógytornász, labordiagnosztikus | Eszjtv. |
| `HEALTH_WORKER` | Egészségügyben dolgozó | Egészségügyi szolgáltató működőképességét biztosító munkakör | Eszjtv. |
| `PUBLIC_SERVANT` | Köztisztviselő | Közigazgatási feladatot ellátó | Kit.; Kttv.; Küt. |

### 3.3. Betöltési feltételek (QualificationRequirement)

A munkakör betöltéséhez előírt képesítési és egyéb feltételek – ezek a Position-höz is kötődhetnek, de alapértelmezetten a PositionTemplate-ből öröklődnek.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `positionTemplateId` | UUID (FK) | feltételes | Hivatkozás a sablonra | Technikai |
| `positionId` | UUID (FK) | feltételes | Hivatkozás a konkrét álláshelyre (ha az felülírja a sablont) | Technikai |
| `requirementType` | enum | igen | Követelmény típusa (lásd alább) | – |
| `description` | string | igen | Követelmény leírása | – |
| `isMandatory` | boolean | igen | Kötelező feltétel-e (vagy csak előnyt jelentő) | – |
| `legalBasis` | string | nem | Jogszabályi hivatkozás | – |

**Követelmény típusok (requirementType):**

| Kód | Megnevezés | Leírás | Példa |
|---|---|---|---|
| `EDUCATION_LEVEL` | Iskolai végzettség szintje | Alapfokú, középfokú, felsőfokú (BA/BSc, MA/MSc, PhD) | Kjt. 61. § – a fizetési osztály meghatározója |
| `EDUCATION_FIELD` | Végzettség szakterülete | Konkrét szak, képzési terület | „Orvostudományi diploma"; „Pedagógus szakképzettség" |
| `PROFESSIONAL_QUALIFICATION` | Szakképzettség / szakképesítés | OKJ / új szakképzési rendszer szerinti | „Ápoló szakképesítés"; „Villanyszerelő" |
| `PROFESSIONAL_EXAM` | Szakvizsga | Közigazgatási szakvizsga, jogi szakvizsga, egészségügyi szakvizsga | Kit. – közigazgatási szakvizsga; Eütev. – orvosi szakvizsga |
| `LANGUAGE` | Nyelvtudás | Nyelv + szint (A1–C2 vagy alap/közép/felső) | Kttv. – nyelvpótlékhoz; Kit. |
| `EXPERIENCE_YEARS` | Szakmai gyakorlat (évek) | Minimális tapasztalat | „Min. 5 év vezetői tapasztalat" |
| `CLEARANCE` | Biztonsági átvilágítás / nemzetbiztonsági ellenőrzés | Meghatározott szintű átvilágítás szükségessége | Kit. egyes munkakörök; Hszt. |
| `CLEAN_RECORD` | Büntetlen előélet | Általános vagy speciális erkölcsi bizonyítvány | Minden közszférás jogviszony; Eszjtv. 2. § (3)–(5) |
| `CHAMBER_MEMBERSHIP` | Kamarai tagság | Orvosi, gyógyszerészi, ügyvédi kamara | Eütev.; ügyvédi tv. |
| `HEALTH_FITNESS` | Egészségügyi alkalmasság | Munkaköri alkalmassági vizsgálat | 33/1998. NM r.; Mvt. |
| `AGE_REQUIREMENT` | Életkori feltétel | Minimális vagy maximális életkor | Hszt. – felső korhatár |
| `REGISTRATION` | Hatósági nyilvántartásba vétel | Működési nyilvántartás | Eütev. – működési nyilvántartás; Púétv. – pedagógus-nyilvántartás |
| `OTHER` | Egyéb feltétel | Nem kategorizálható | „B kategóriás jogosítvány" |

---

## 4. Position (Konkrét álláshely / szervezeti munkakör)

### 4.1. Alapadatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `positionTemplateId` | UUID (FK) | igen | Hivatkozás a munkaköri sablonra | Technikai |
| `organizationId` | UUID (FK) | igen | Szervezet | Technikai |
| `orgUnitId` | UUID (FK) | igen | Szervezeti egység, amelyhez az álláshely tartozik | Kit. 35. §; SZMSZ |
| `positionCode` | string | igen | Álláshely egyedi kódja a szervezeten belül | Kit. – álláshely-azonosító; belső szabályzat |
| `positionTitle` | string | igen | Az álláshely konkrét megnevezése (örökölhető a PositionTemplate-ből, de felülírható) | Mt. 45. §; Kit.; Kjt. |
| `siteId` | UUID (FK) | nem | Munkavégzési hely (telephely) | Mt. 45. § – munkavégzés helye |
| `reportingToPositionId` | UUID (FK) | nem | Közvetlen felettes álláshely (szervezeti alá-fölérendeltség) | SZMSZ; belső szabályzat |
| `status` | enum | igen | Álláshely státusza (lásd 4.2.) | – |
| `effectiveFrom` | date | igen | Álláshely érvényesség kezdete | – |
| `effectiveTo` | date | nem | Álláshely érvényesség vége (null = aktív) | – |

### 4.2. Álláshely státuszok

| Kód | Megnevezés | Leírás |
|---|---|---|
| `PLANNED` | Tervezett | Jóváhagyott, de még nem betölthető (pl. költségvetési jóváhagyásra vár) |
| `VACANT` | Betöltetlen / üres | Betölthető, nincs hozzá aktív Employment |
| `OCCUPIED` | Betöltött | Aktív Employment kapcsolódik hozzá |
| `PARTIALLY_OCCUPIED` | Részben betöltött | Részmunkaidős betöltés, van fennmaradó kapacitás |
| `FROZEN` | Befagyasztott | Költségvetési vagy szervezeti okból átmenetileg nem tölthető be |
| `ABOLISHED` | Megszüntetett | Az álláshely megszűnt |

---

### 4.3. Kapacitás és betöltöttség

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `headcount` | decimal | igen | Tervezett létszám (FTE – Full Time Equivalent); 1.0 = egy teljes munkaidős hely | Költségvetési tervezés; Kit. – alaplétszám |
| `currentFTE` | decimal | számított | Jelenlegi betöltöttség FTE-ben (a kapcsolódó Employment-ek weeklyHours/40 összege) | – |
| `currentHeadcount` | integer | számított | Jelenlegi betöltöttség fő-ben | – |
| `vacancyFTE` | decimal | számított | Üres FTE (`headcount` – `currentFTE`) | – |
| `isSharedPosition` | boolean | nem | Megosztott álláshely-e (több személy tölti be részmunkaidőben) | – |
| `maxOccupants` | integer | nem | Maximálisan hány személy töltheti be (alapértelmezés: 1 egész álláshelynél) | – |

**Megjegyzések:**
- A **Kit.** szerinti álláshely-alapú igazgatásnál az álláshely az alapegység: a szervezet alaplétszáma az álláshelyek összessége. Az álláshely betöltéséhez a szakmai követelmények teljesülése szükséges.
- A **Kjt.**-ben a munkakör a besorolás alapja, de formálisan nem „álláshely-alapú" a rendszer. Ennek ellenére a gyakorlatban a szervezetek álláshelyet tartanak nyilván.
- Az **FTE** számítás a részmunkaidős foglalkoztatásnál kritikus: egy 0.5 FTE-s álláshely 50%-ban betöltött, ha egy teljes munkaidős dolgozó van rajta – nem, ha egy fél munkaidős van rajta.

---

### 4.4. Besorolási keret – jogviszony-specifikus

Az álláshely meghatározza a besorolás **kereteit** – az Employment-ben a konkrét besorolás jelenik meg.

#### 4.4.1. Mt. – Munkaviszony

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `mt_salaryRangeMin` | integer | nem | Bérsáv alsó határa (Ft/hó) | Belső bérpolitika |
| `mt_salaryRangeMax` | integer | nem | Bérsáv felső határa | Belső bérpolitika |
| `mt_salaryRangeMid` | integer | nem | Bérsáv közepe (target) | Belső bérpolitika |
| `mt_requiresQualification` | boolean | igen | Szakképzettséget igénylő munkakör-e (garantált bérminimum vs. minimálbér) | Mt. 153. § |
| `mt_isExecutive` | boolean | nem | Vezető állású munkavállaló-e (Mt. 208–211. §) | Mt. 208. § – eltérő munkajogi szabályok |
| `mt_isRemoteWorkEligible` | boolean | nem | Távmunkavégzésre alkalmas-e | Mt. 196. § |

#### 4.4.2. Kjt. – Közalkalmazotti jogviszony

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `kjt_requiredPayGrade` | enum | igen | Előírt fizetési osztály (A–J) – a munkakör betöltéséhez szükséges végzettség alapján | Kjt. 61. § |
| `kjt_alternativePayGrade` | enum | nem | Megengedett alternatív fizetési osztály (ha a jogszabály lehetővé teszi) | Kjt. 61. § (3) – felmentés |
| `kjt_leaderLevel` | enum | nem | Vezetői szint: `NONE`, `LEADER`, `SENIOR_LEADER` | Kjt. 20–24. § |
| `kjt_sectorCode` | string | nem | Ágazati kód (pl. szociális, kulturális, közgyűjteményi) | Ágazati vhr.-ek |

#### 4.4.3. Púétv. – Köznevelési foglalkoztatotti jogviszony

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `puetv_positionType` | enum | igen | `PEDAGOGUE` (pedagógus-munkakör), `PEDAGOGUE_SUPPORT` (nevelő-oktató munkát közvetlenül segítő) | Púétv. 4–5. § |
| `puetv_isPedagogueCareer` | boolean | számított | A pedagógus-előmenetel hatálya alá tartozik-e (a positionType-ból vezethető le) | Púétv. |
| `puetv_weeklyTeachingHoursMin` | integer | nem | Heti kötelező tanítási óraszám alsó határa | Púétv. 88. § – munkakörtől függ (óvónő: 32, tanár: 22–26 stb.) |
| `puetv_weeklyTeachingHoursMax` | integer | nem | Heti kötelező tanítási óraszám felső határa | Púétv. 88. § |
| `puetv_requiredSpecialization` | array[string] | nem | Szükséges pedagógus-szakképzettség(ek) (pl. „matematika-fizika szak", „óvodapedagógus") | 326/2013. Korm. r. |
| `puetv_leaderLevel` | enum | nem | Vezetői szint: `NONE`, `DEPUTY_HEAD`, `HEAD`, `MULTI_HEAD` (többcélú intézményvezető) | Púétv. 68–70. § |

#### 4.4.4. Eszjtv. – Egészségügyi szolgálati jogviszony

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `eszjtv_jobCategory` | enum | igen | Munkakör-kategória az illetménytáblázatban (orvos, szakorvos, fogorvos, gyógyszerész, diplomás ápoló, ápoló, asszisztens stb.) | 528/2020. Korm. r. – illetménytáblázat |
| `eszjtv_workerType` | enum | igen | `HEALTH_PROFESSIONAL` (eü. dolgozó) / `HEALTH_WORKER` (eü.-ben dolgozó) | Eütev. 4. § a)–c) |
| `eszjtv_requiredRegistration` | enum | nem | Szükséges hatósági nyilvántartás: `MOK` (orvosi kamara), `MESZK` (egészségügyi szakdolgozói kamara), `MGYK` (gyógyszerészi kamara) | Eütev. 7. § |
| `eszjtv_leaderLevel` | enum | nem | Vezetői szint: `NONE`, `LEADER`, `SENIOR_LEADER` (orvosigazgató, ápolási igazgató stb.) | 528/2020. Korm. r. 5. § |
| `eszjtv_isOnCallEligible` | boolean | nem | Ügyeletre beosztható munkakör-e | Eszjtv.; Eütev. 12/D. § |

#### 4.4.5. Kit./Kttv./Küt. – Kormányzati és közszolgálat

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `gov_classificationCategory` | enum | igen | Álláshely besorolási kategóriája | Kit. 81–83. §; Kttv.; Küt. 14–16. § |
| `gov_salaryBandMin` | integer | igen | Illetménysáv alsó határa (Ft/hó) | Kit. 114. § |
| `gov_salaryBandMax` | integer | igen | Illetménysáv felső határa (Ft/hó) | Kit. 114. § |
| `gov_civilServiceExamRequired` | boolean | igen | Közigazgatási szakvizsga szükséges-e | Kit. 119. §; Kttv. |
| `gov_civilServiceExamDeadlineMonths` | integer | nem | Szakvizsga letételének határideje a kinevezéstől (hónapokban) | Kit.; Kttv. |
| `gov_clearanceLevel` | enum | nem | Szükséges biztonsági szint: `NONE`, `BASIC`, `NATIONAL_SECURITY`, `TOP_SECRET` | Nbtv. |
| `gov_assetDeclarationRequired` | boolean | nem | Vagyonnyilatkozat-tételi kötelezettség az álláshelyhez kötődik-e | Kit. 157–160. §; Kttv. |
| `gov_isCompetitive` | boolean | nem | Pályázat útján betölthető-e | Kit. 44. §; Kttv.; Küt. |

---

### 4.5. Munkakörülmények és munkaidő-paraméterek

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `defaultWorkScheduleType` | enum | nem | Alapértelmezett munkarend (általános, egyenlőtlen, kötetlen, osztott) | Mt. 97. § |
| `defaultWeeklyHours` | decimal | nem | Alapértelmezett heti munkaidő | Mt. 92. § |
| `isNightWorkRequired` | boolean | nem | Az álláshely éjszakai munkavégzést igényel-e | Mt. 89. § |
| `isShiftWorkRequired` | boolean | nem | Műszakos munkarend | Mt. 97. § |
| `isOnCallRequired` | boolean | nem | Ügyelet/készenlét előírható-e | Mt. 110–112. §; Eszjtv. |
| `hazardCategory` | enum | nem | Veszélyességi kategória: `NONE`, `LOW`, `MEDIUM`, `HIGH` | Mvt.; 33/1998. NM r. |
| `healthCheckFrequency` | enum | nem | Munkaköri alkalmassági vizsgálat gyakorisága: `YEARLY`, `BIENNIAL`, `OTHER` | 33/1998. NM r. |
| `specialWorkConditions` | array[enum] | nem | Speciális munkakörülmények (lásd alább) | Mvt.; ágazati rendeletek |

**Speciális munkakörülmények (specialWorkConditions):**

| Kód | Megnevezés | HRMS hatás |
|---|---|---|
| `HAZARDOUS_MATERIALS` | Veszélyes anyagokkal való munka | Veszélyességi pótlék; fokozott egészségügyi ellenőrzés |
| `IONIZING_RADIATION` | Ionizáló sugárzás | Dózisnyilvántartás; speciális alkalmasság |
| `BIOLOGICAL_HAZARD` | Biológiai veszély (fertőzésveszély) | Fertőzésveszélyességi pótlék (egészségügy) |
| `NOISE` | Zajterhelés | Hallásvizsgálat; zajpótlék |
| `OUTDOOR` | Szabadtéri munkavégzés | Időjárási pótlék; speciális munkaruha |
| `VDU_WORK` | Képernyő előtti munkavégzés (napi 4 óra felett) | 50/1999. EüM r. – szemészeti vizsgálat |
| `DRIVING` | Gépjárművezetés (munkakör részeként) | Vezetői alkalmasság; „B" kategória követelmény |
| `PHYSICAL_LOAD` | Nehéz fizikai munka | Fizikai alkalmasság; teherbírás vizsgálat |

---

### 4.6. Pályázati és toborzási jellemzők

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `isPublicRecruitment` | boolean | nem | Nyilvános pályázat szükséges-e a betöltéshez | Kit. 44. §; Kjt. 20/A. §; Kttv. |
| `recruitmentChannel` | enum | nem | Előírt közzétételi csatorna: `KOZIGALLAS_GOV_HU` (közigállás), `INTERNAL`, `PUBLIC`, `NONE` | Kit. – közszolgálati álláspályázat; Kjt.; Kttv. |
| `probationRequired` | boolean | nem | Próbaidő kötelező-e az álláshelyen | Mt. 45. § (3); Eszjtv. 3. § (kötelező) |
| `defaultProbationMonths` | integer | nem | Próbaidő alapértelmezett hossza (hónapban) | Mt. (max. 3); Eszjtv. (3, max. 4) |

---

### 4.7. Helyettesítés és feladatátadás

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `deputyPositionId` | UUID (FK) | nem | Helyettesítő álláshely (ki helyettesít, ha a betöltő távol van) |
| `substitutePositionId` | UUID (FK) | nem | Felfelé helyettesítés (a betöltő kit helyettesít szükség esetén) |

---

### 4.8. Audit mezők

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `createdAt` | datetime | igen | Rekord létrehozása |
| `createdBy` | string | igen | Létrehozó |
| `updatedAt` | datetime | igen | Utolsó módosítás |
| `updatedBy` | string | igen | Módosító |

---

## 5. FEOR és foglalkozási besorolás

### 5.1. A FEOR szerepe

A **FEOR-08** (Foglalkozások Egységes Osztályozási Rendszere, 2008-as változat) a foglalkozások magyar statisztikai besorolási rendszere. Az HRMS szempontjából:

- **Kötelező:** A T1041 (biztosítotti bejelentés) tartalmazza a FEOR-kódot → a NAV felé kötelező adat.
- **Minimálbér/garantált bérminimum meghatározó:** A FEOR főcsoport alapján dönthető el, hogy a munkakör szakképzettséget igényel-e (1–4. főcsoport → garantált bérminimum; 5–9. főcsoport → minimálbér, de ez nem mechanikus, a tényleges követelmény számít).
- **KSH adatszolgáltatás:** A munkaügyi statisztikák FEOR-kód alapján készülnek.

### 5.2. FEOR struktúra

| Szint | Számjegyek | Példa | Megnevezés |
|---|---|---|---|
| Főcsoport | 1 jegy | `2` | Felsőfokú képzettség önálló alkalmazását igénylő foglalkozások |
| Csoport | 2 jegy | `24` | Üzleti, jogi és társadalomtudományi foglalkozások |
| Alcsoport | 3 jegy | `242` | Jogász foglalkozások |
| Foglalkozás | 4 jegy | `2421` | Ügyvéd, jogtanácsos |

Az HRMS-nek a **4-jegyű FEOR-kódot** kell nyilvántartania (a PositionTemplate-en), és a NAV bejelentéshez a **6-jegyű kiegészített FEOR-kódot** (FEOR + 2 jegyű kiegészítő).

---

## 6. Historikus adatkezelés

### 6.1. PositionHistory

| Property | Típus | Leírás |
|---|---|---|
| `id` | UUID | PK |
| `positionId` | UUID (FK) | Érintett álláshely |
| `changeType` | enum | Változás típusa |
| `validFrom` | date | Érvényesség kezdete |
| `validTo` | date | Érvényesség vége |
| `previousValue` | JSON | Korábbi érték |
| `newValue` | JSON | Új érték |
| `changeReason` | string | Indoklás |
| `legalBasis` | string | Jogalap |
| `changedAt` | datetime | Változás rögzítése |
| `changedBy` | string | Rögzítő |

**Változástípusok:**

| Kód | Megnevezés | Tipikus eset |
|---|---|---|
| `RECLASSIFICATION` | Átsorolás | Kjt. fizetési osztály változás; Kit. besorolási kategória módosítás |
| `TITLE_CHANGE` | Megnevezés változás | Munkaköri cím módosítása |
| `ORG_UNIT_CHANGE` | Szervezeti egység változás | Átszervezés |
| `SALARY_BAND_CHANGE` | Bérsáv / illetménysáv módosítás | Éves felülvizsgálat; jogszabályváltozás |
| `REQUIREMENT_CHANGE` | Betöltési feltétel változás | Jogszabályi változás (pl. új képesítési előírás) |
| `STATUS_CHANGE` | Státuszváltozás | Betöltött → üres; befagyasztás |
| `HEADCOUNT_CHANGE` | FTE változás | Létszámbővítés vagy -csökkentés |
| `ABOLISHMENT` | Megszüntetés | Szervezeti döntés; költségvetési ok |

---

## 7. Üzleti szabályok és validációk

| Szabály | Leírás | Jogszabály |
|---|---|---|
| FEOR-kód validáció | A megadott FEOR-kódnak léteznie kell a FEOR-08 kódlistában | KSH |
| Besorolási konzisztencia | Kjt.: a Position `kjt_requiredPayGrade`-je összhangban kell legyen a PositionTemplate betöltési feltételeivel (iskolai végzettség szintje) | Kjt. 61. § |
| Employment–Position kompatibilitás | Az Employment `employmentType`-ja benne kell legyen a Position `applicableEmploymentTypes` listájában | Jogállási törvények hatályi rendelkezései |
| FTE túltöltés figyelmeztetés | Ha `currentFTE` > `headcount`, figyelmeztetés (túltöltött álláshely) | Költségvetési fegyelem |
| Betöltési feltétel ellenőrzés | Employment létrehozásakor a Person Qualification-jeit össze kell vetni a Position QualificationRequirement-jeivel | Kjt. 61. §; Kit.; Eütev.; Púétv. |
| Vezetői szint konzisztencia | Ha `kjt_leaderLevel` / `puetv_leaderLevel` / `eszjtv_leaderLevel` / `gov` értéke nem `NONE`, a munkakör-kategóriának `EXECUTIVE` vagy `MANAGER`-nek kell lennie | Logikai konzisztencia |
| Pedagógus-munkakör → Púétv. | Ha `puetv_positionType` = `PEDAGOGUE`, az Employment-ben a pedagógus-előmenetel alkalmazandó | Púétv. |
| Közigazgatási szakvizsga határidő | Ha `gov_civilServiceExamRequired` = true és az Employment-ben `gov_civilServiceExamPassed` = false, a rendszer figyelmeztet a határidőre | Kit.; Kttv. |
| Megszüntetett álláshelyre nem létesíthető Employment | Ha `status` = `ABOLISHED`, új Employment nem hozható létre | Logikai szabály |

---

## 8. GDPR adatkezelési kategorizáció

| Kategória | Property-k | Jogalap | Megjegyzés |
|---|---|---|---|
| **Álláshely alapadatok** | Szinte minden Position és PositionTemplate property | Nem személyes adat | Az álláshely a szervezet struktúrájának része, nem személyes adat |
| **Betöltöttségi adatok** | currentFTE, currentHeadcount (számított, Employment-ből) | Származtatott személyes adat (közvetetten azonosítható, ha 1 fős egység) | Aggregált szinten nem személyes; egyéni szinten az Employment kezeli |
| **Vezetői adat** | headPersonId (ha van az OrgUnit-on) → Position-ön keresztül | Jogi kötelezettség (közszférában közzétett) | Közérdekű adat a közszférában |

**Megjegyzések:**
- A Position entitás adatai **döntő többségében nem személyes adatok**, mivel a szervezeti struktúra és a munkakör-leírás a szervezet jellemzője, nem a személyé.
- Az összekötés a személlyel az **Employment entitáson** keresztül történik – ott érvényesülnek a GDPR szabályok.
- A közszférában a pozíció betöltőjének neve és illetménye **közérdekű adat** lehet (Info tv.) – ezt az Employment és Compensation entitásokon kell kezelni.

---

## 9. Integrációs pontok

| Külső rendszer | Irány | Érintett adatok | Cél |
|---|---|---|---|
| **NAV (T1041)** | kimenő | feorCode (PositionTemplate-ből) | Biztosítotti bejelentés – FEOR-kód kötelező mező |
| **KSH (OSAP – munkaügyi statisztika)** | kimenő | feorCode, headcount, currentFTE, positionCategory | Munkaügyi statisztikai adatszolgáltatás |
| **Közigállás (kozigallas.gov.hu)** | kimenő | Álláspályázati adatok: positionTitle, QualificationRequirements, gov_classificationCategory, gov_salaryBandMin/Max | Közszolgálati álláshirdetés |
| **KIR (Köznevelési Információs Rendszer)** | kétirányú | puetv_positionType, puetv_requiredSpecialization, pedagógus-létszám | Pedagógus-munkakör nyilvántartás |
| **OKFŐ / NEAK** | kimenő | eszjtv_jobCategory, eszjtv_workerType | Egészségügyi munkakör nyilvántartás |
| **Költségvetési tervezés** | kimenő | headcount, gov_salaryBandMin/Max, kjt_requiredPayGrade | Bérköltség-tervezés |
| **Toborzási rendszer (ATS)** | kimenő | PositionTemplate adatok, QualificationRequirements, bérsáv | Állásmeghirdetés |
| **Munkaköri leírás generátor** | kimenő | PositionTemplate.description, QualificationRequirements, munkakörülmények | Dokumentumgenerálás |

---

## 10. Hozzáférési szintek

| Szerep | PositionTemplate | Position | QualificationRequirement |
|---|---|---|---|
| **Munkavállaló** | Saját munkakör olvasás | Saját pozíció olvasás; szervezeti ábra (betöltők nélkül) | Saját pozíció követelményei |
| **Közvetlen vezető** | Saját egység munkakörök | Saját egység + alárendelt pozíciók (státusz, betöltöttség) | Olvasás |
| **HR ügyintéző** | Olvasás / javaslattétel | Teljes olvasás/írás a hatáskörben | Teljes olvasás/írás |
| **Szervezetfejlesztő** | Teljes olvasás/írás | Teljes olvasás/írás | Teljes olvasás/írás |
| **Pénzügyi / kontrolling** | Bérsáv adatok olvasás | headcount, FTE, bérsáv – aggregált | – |
| **Toborzási specialista** | Olvasás | Üres pozíciók olvasás | Olvasás (hirdetéshez) |
| **Rendszeradminisztrátor** | FEOR törzsadat karbantartás | Technikai mezők | Törzsadat karbantartás |

---

## 11. Kapcsolódó entitások – teljes kép

```
PositionTemplate 1 ──── N Position                  (sablon → konkrét álláshelyek)
PositionTemplate 1 ──── N QualificationRequirement   (sablonhoz kötött betöltési feltételek)
Position 1 ──── N QualificationRequirement           (pozícióhoz kötött, sablont felülíró feltételek)

Organization 1 ──── N PositionTemplate               (szervezeti munkakör-katalógus)
OrgUnit 1 ──── N Position                            (szervezeti egységhez rendelt álláshelyek)
Site 1 ──── N Position                               (munkavégzési hely)

Position 1 ──── N Employment                          (betöltő jogviszonyok)
Position ──── Position (reportingToPositionId)        (szervezeti alá-fölérendeltség – self-ref)
Position ──── Position (deputyPositionId)             (helyettesítési lánc)

Position ──── PositionHistory                          (historikus változások)
```

---

## 12. Nyitott kérdések

1. **PositionTemplate szükségessége:** Kisebb szervezeteknél (pl. egy óvoda 20 dolgozóval) a kétszintű modell (sablon + konkrét álláshely) túl komplex lehet. Szükség van-e egyszerűsített módra, ahol a Position közvetlenül tartalmazza a munkakör-leírást sablon nélkül?
2. **Munkaköri leírás tárolása:** A részletes munkaköri leírás (feladatok, felelősségek, hatáskörök) a Position entitásban legyen (strukturált mezők vagy szabad szöveg), vagy önálló dokumentumként (EmploymentDocument entitás)?
3. **Szervezeti ábra vs. beszámolási struktúra:** A `reportingToPositionId` a szervezeti hierarchiát (OrgUnit fastruktúra) tükrözi, vagy ettől eltérő beszámolási láncot? Mátrix szervezeteknél a kettő eltérhet.
4. **FEOR karbantartás:** A FEOR-08 kódlista a KSH-tól származik és időnként frissül. A rendszer saját FEOR törzsadatot tartson, vagy integráljon a KSH kódlistájából?
5. **Pedagógus óraszám-keret:** A Púétv. szerinti heti kötelező tanítási óraszám munkakörtől és intézménytípustól függ. Ez a Position szintjén kezelendő (mint most), vagy az Employment szintjén (mivel a tényleges beosztás személyfüggő)?
6. **Álláshely-gazdálkodás és költségvetés:** Szükséges-e a Position entitásnak költségvetési vonzatot (éves bérköltség-terv) is hordoznia, vagy az a Compensation / pénzügyi modul feladata?
7. **Közalkalmazotti munkaköri bértábla:** A Kjt. ágazati végrehajtási rendeletei (pl. szociális, kulturális) saját munkakör-jegyzéket tartalmaznak, amelyek a fizetési osztályt határozzák meg. Ezeket a QualificationRequirement-ben kezeljük, vagy külön munkakör-bértábla törzsadat szükséges?

---

### 12.1. Javaslatok és válaszok

#### 12.1.1. PositionTemplate szükségessége – Egyszerűsített mód kis szervezeteknek

**Probléma:** Kis szervezeteknél (pl. 20 fős óvoda) a kétszintű modell (PositionTemplate + Position) túl komplex:
- 10 különböző munkakör esetén 10 sablon + 20 konkrét álláshely → 30 rekord
- A sablon-Position szétválasztás értéke kicsi, ha minden sablon csak 1-2 álláshelyhez kapcsolódik

**Javaslat: Opcionális PositionTemplate + Inline mód (Extended ajánlás)**

**MVP megoldás:**
Minden Position kötelezően hivatkozik egy PositionTemplate-re. Kis szervezeteknél 1:1 megfeleltetés (1 sablon = 1 álláshely).

```typescript
// MVP: Kötelező PositionTemplate
Position {
  id: UUID
  positionTemplateId: UUID FK → PositionTemplate  // KÖTELEZŐ
  organizationId: UUID FK
  orgUnitId: UUID FK
  positionCode: string
}

PositionTemplate {
  id: UUID
  organizationId: UUID FK
  code: string
  name: string
  feorCode: string
}
```

**Korlátok:** Kis szervezeteknél felesleges bonyolultság.

**Extended megoldás (ajánlott):**
`positionTemplateId` opcionális. Ha null, a Position tartalmazza a munkakör-leírást közvetlenül („inline mód").

```typescript
Position {
  id: UUID PK
  positionTemplateId: UUID FK → PositionTemplate  // OPCIONÁLIS
  organizationId: UUID FK
  orgUnitId: UUID FK
  positionCode: string
  positionTitle: string

  // Inline munkakör-leírás (ha positionTemplateId = null)
  inlinePositionName: string  // Ha nincs sablon, közvetlenül itt a név
  inlineFeorCode: string(6)
  inlineFeorName: string
  inlineCategory: enum
  inlineDescription: text
  inlineApplicableEmploymentTypes: array[enum]

  // Hatályosság
  effectiveFrom: date
  effectiveTo: date
  status: enum (PLANNED, VACANT, OCCUPIED, PARTIALLY_OCCUPIED, FROZEN, ABOLISHED)

  // Computed properties
  workingPositionName: string (computed)  // = positionTemplateId ? template.name : inlinePositionName
  workingFeorCode: string (computed)      // = positionTemplateId ? template.feorCode : inlineFeorCode
}

// Validáció
function validatePosition(position) {
  if (position.positionTemplateId === null) {
    // Inline mód: kötelező inline mezők
    if (!position.inlinePositionName || !position.inlineFeorCode || !position.inlineCategory) {
      throw new ValidationError('Inline módban kötelező: inlinePositionName, inlineFeorCode, inlineCategory');
    }
  } else {
    // Sablon mód: inline mezők NULL-ok
    if (position.inlinePositionName !== null || position.inlineFeorCode !== null) {
      throw new ValidationError('Sablon mód esetén az inline mezők NULL-ok kell legyenek');
    }
  }
}
```

**Használat példa:**

```typescript
// 1. Nagyobb szervezet (100+ fő): PositionTemplate használata
PositionTemplate {
  id: 'template-001'
  organizationId: 'org-001'
  code: 'HR-UGYINT'
  name: 'HR ügyintéző'
  feorCode: '333902'  // Személyzeti ügyintéző
  category: ADMINISTRATOR
}

Position {
  id: 'pos-001'
  positionTemplateId: 'template-001'  // Sablon használata
  organizationId: 'org-001'
  orgUnitId: 'orgunit-hr'
  positionCode: 'POS-HR-001'
  positionTitle: 'HR ügyintéző (Budapest)'
  inlinePositionName: null  // Nincs inline
}

Position {
  id: 'pos-002'
  positionTemplateId: 'template-001'  // Ugyanaz a sablon
  organizationId: 'org-001'
  orgUnitId: 'orgunit-hr'
  positionCode: 'POS-HR-002'
  positionTitle: 'HR ügyintéző (Debrecen)'
}

// 2. Kis szervezet (20 fő): Inline mód
Position {
  id: 'pos-003'
  positionTemplateId: null  // NINCS sablon
  organizationId: 'org-ovoda-001'
  orgUnitId: 'orgunit-ovoda-001'
  positionCode: 'POS-OVONO-001'
  positionTitle: 'Óvodapedagógus'

  // Inline mezők
  inlinePositionName: 'Óvodapedagógus'
  inlineFeorCode: '234101'  // Óvodapedagógus
  inlineCategory: PEDAGOGUE
  inlineDescription: 'Óvodai nevelő-oktató tevékenység ellátása'
  inlineApplicableEmploymentTypes: ['KOZNEVELESI']
}
```

**Strategic megoldás:**
Automatikus sablon-konverzió: Ha egy inline Position-t többször is létrehoznak hasonló adatokkal, a rendszer javasol sablont.

```javascript
async function suggestTemplateCreation(organizationId) {
  // Lekérdezés: inline pozíciók csoportosítása FEOR-kód szerint
  const inlinePositions = await db.query(`
    SELECT
      inlineFeorCode,
      inlinePositionName,
      COUNT(*) AS positionCount
    FROM Position
    WHERE organizationId = ?
      AND positionTemplateId IS NULL
      AND effectiveTo IS NULL
    GROUP BY inlineFeorCode, inlinePositionName
    HAVING COUNT(*) >= 3
  `, [organizationId]);

  // Ha 3+ azonos FEOR-kódú inline pozíció van, javaslat sablon létrehozására
  const suggestions = inlinePositions.map(row => ({
    feorCode: row.inlineFeorCode,
    positionName: row.inlinePositionName,
    positionCount: row.positionCount,
    recommendation: `${row.positionCount} db "${row.inlinePositionName}" pozíció létezik. Érdemes sablont létrehozni!`
  }));

  return suggestions;
}

// Sablon létrehozása inline Position-ből
async function convertInlineToTemplate(positionId) {
  const position = await Position.findById(positionId);

  if (position.positionTemplateId !== null) {
    throw new Error('Ez a pozíció már sablon-alapú');
  }

  // 1. Új PositionTemplate létrehozása
  const template = await PositionTemplate.create({
    organizationId: position.organizationId,
    code: `TEMPLATE-${position.inlineFeorCode}`,
    name: position.inlinePositionName,
    feorCode: position.inlineFeorCode,
    category: position.inlineCategory,
    description: position.inlineDescription,
    applicableEmploymentTypes: position.inlineApplicableEmploymentTypes
  });

  // 2. Ugyanolyan inline pozíciók átállítása a sablonra
  await db.query(`
    UPDATE Position
    SET
      positionTemplateId = ?,
      inlinePositionName = NULL,
      inlineFeorCode = NULL,
      inlineCategory = NULL,
      inlineDescription = NULL
    WHERE organizationId = ?
      AND inlineFeorCode = ?
      AND positionTemplateId IS NULL
  `, [template.id, position.organizationId, position.inlineFeorCode]);

  return template;
}
```

**Jogszabályi háttér:**
- **Mt. 45. § (1):** A munkaszerződés kötelező tartalmi eleme a munkakör meghatározása – de nincs előírás sablonról.
- **Kit. 35. §:** Álláshely-leírás kötelező, de sablon-katalógus nem.
- **Kjt.:** Nincs kötelező munkakör-katalógus, de javasolt (ágazati szabályozók tartalmaznak munkakör-jegyzéket).

**GDPR:**
Nem releváns (munkakörök nem személyes adatok).

**Implementációs prioritás:**
- **MVP:** Kötelező PositionTemplate (egyszerűbb implementáció, de kis szervezeteknél bonyolult)
- **Extended:** Opcionális PositionTemplate + inline mód (ajánlott – rugalmas, skálázható)
- **Strategic:** Automatikus sablon-javaslat és konverzió

**Ajánlás:**
Extended modell (opcionális template) bevezetése, mert:
1. Kis szervezetek (< 50 fő) számára egyszerűsíti a használatot
2. Nagy szervezetek számára megtartja a sablonok előnyeit (konzisztencia, karbantarthatóság)
3. Szervezet növekedésével zökkenőmentesen átállhat inline → template módra

---

#### 12.1.2. Munkaköri leírás tárolása – Strukturált vs. Dokumentum

**Probléma:** A részletes munkaköri leírás (feladatok, felelősségek, hatáskörök, kompetenciák) tárolási módja:
- **Strukturált mezőkben:** Position/PositionTemplate entitásban (feladatLista, felelossegek, hatáskor stb.)
- **Dokumentum formátumban:** Külön EmploymentDocument entitásban (PDF, Word)

**Javaslat: Hibrid modell – Strukturált mag + Dokumentum-hivatkozás (Extended ajánlás)**

**MVP megoldás:**
Egyszerű `description` text mező a PositionTemplate/Position-ben. Nincs strukturáltság.

```typescript
PositionTemplate {
  // ...
  description: text  // Szabad szöveges munkaköri leírás
}
```

**Korlátok:**
- Nem kereshető strukturáltan (pl. „mely munkakörökhöz kell szerződéskötési jogosultság?")
- Nehéz verzionálni, összehasonlítani

**Extended megoldás (ajánlott):**
Strukturált munkaköri leírás (`PositionDescription` entitás) + opcionális dokumentum-hivatkozás.

```typescript
PositionDescription {
  id: UUID PK
  positionTemplateId: UUID FK → PositionTemplate  // VAGY
  positionId: UUID FK → Position

  // Strukturált mezők
  purpose: text  // A munkakör célja, lényege (1-2 mondat)

  // Feladatok (JSON array vagy külön PositionTask entitás)
  tasks: json  // [{ task: "...", priority: "HIGH" }, ...]

  // Felelősségek és hatáskörök
  responsibilities: json  // [{ area: "Személyzeti", description: "..." }, ...]
  authorities: json  // [{ authority: "Szerződéskötés 500k-ig", legalBasis: "..." }, ...]

  // Beszámolási viszony
  reportsTo: string  // Kinek számol be (örökölhető Position.reportingToPositionId-ból)
  supervises: string  // Kit felügyel

  // Kompetenciák (opcionális)
  requiredCompetencies: json  // [{ competency: "Tárgyalási készség", level: "ADVANCED" }, ...]

  // Teljesítménymutatók (opcionális)
  kpis: json  // [{ kpi: "Beérkező kérelmek feldolgozási ideje", target: "< 3 nap" }, ...]

  // Munkakörnyezet
  workEnvironment: text  // Munkavégzés körülményei, fizikai követelmények
  travelRequirement: enum (NONE, OCCASIONAL, FREQUENT, CONSTANT)  // Utazási igény

  // Verzió
  version: integer
  effectiveFrom: date
  effectiveTo: date
  approvedBy: UUID FK → Person
  approvedAt: datetime

  // Dokumentum-hivatkozás
  documentId: UUID FK → Document  // Hivatalos munkaköri leírás dokumentum (PDF, DOCX)
  documentVersion: string

  // Audit
  createdAt: datetime
  createdBy: string
  updatedAt: datetime
  updatedBy: string
}

// Alternatíva: PositionTask külön entitás (ha nagyon részletes feladatlista szükséges)
PositionTask {
  id: UUID PK
  positionDescriptionId: UUID FK → PositionDescription
  taskCategory: enum (CORE, ADDITIONAL, OCCASIONAL)
  taskDescription: text
  priority: enum (HIGH, MEDIUM, LOW)
  frequency: enum (DAILY, WEEKLY, MONTHLY, QUARTERLY, ANNUAL, AD_HOC)
  estimatedTimePercentage: decimal(5,2)  // Az összes munkaidő hány %-a
  sortOrder: integer
}

// Document entitás bővítés (már létező entitás)
Document {
  id: UUID PK
  documentType: enum (..., JOB_DESCRIPTION, ...)  // Munkaköri leírás típus
  fileName: string
  fileUrl: string
  version: string
  effectiveFrom: date
  effectiveTo: date
  approvedBy: UUID FK → Person
}
```

**Használat példa:**

```sql
-- Lekérdezés: Mely munkakörökhöz kell angol nyelvtudás?
SELECT
  pt.name AS positionName,
  qr.description AS requirementDescription,
  pd.purpose
FROM PositionTemplate pt
JOIN QualificationRequirement qr ON pt.id = qr.positionTemplateId
LEFT JOIN PositionDescription pd ON pt.id = pd.positionTemplateId
WHERE qr.requirementType = 'LANGUAGE'
  AND qr.description LIKE '%angol%'
  AND qr.isMandatory = true;

-- Lekérdezés: Mely munkakörökhöz tartozik szerződéskötési jogosultság?
SELECT
  pt.name AS positionName,
  JSON_EXTRACT(pd.authorities, '$[*].authority') AS authorities
FROM PositionTemplate pt
JOIN PositionDescription pd ON pt.id = pd.positionTemplateId
WHERE JSON_CONTAINS(pd.authorities, '{"authority": "Szerződéskötés"}', '$');
```

**Workflow példa: Munkaköri leírás jóváhagyása**

```javascript
async function approvePositionDescription(positionDescriptionId, approverId) {
  const description = await PositionDescription.findById(positionDescriptionId);

  if (description.approvedBy !== null) {
    throw new ValidationError('Ez a munkaköri leírás már jóváhagyott');
  }

  // 1. Munkaköri leírás jóváhagyása
  description.approvedBy = approverId;
  description.approvedAt = new Date();
  description.effectiveFrom = new Date();
  await description.save();

  // 2. Korábbi verziók lezárása
  await PositionDescription.update({
    effectiveTo: new Date()
  }, {
    where: {
      positionTemplateId: description.positionTemplateId,
      version: { $lt: description.version },
      effectiveTo: null
    }
  });

  // 3. Dokumentum generálása (opcionális)
  const document = await generateJobDescriptionDocument(description);
  description.documentId = document.id;
  description.documentVersion = document.version;
  await description.save();

  return description;
}

// Dokumentum generálás (pl. PDF)
async function generateJobDescriptionDocument(positionDescription) {
  const template = await PositionTemplate.findById(positionDescription.positionTemplateId);

  const pdfContent = `
Munkaköri leírás

Munkakör megnevezése: ${template.name}
FEOR kód: ${template.feorCode}

1. A munkakör célja:
${positionDescription.purpose}

2. Feladatok:
${JSON.parse(positionDescription.tasks).map((t, i) => `${i + 1}. ${t.task}`).join('\n')}

3. Felelősségek és hatáskörök:
...

Jóváhagyta: ${positionDescription.approvedBy}
Jóváhagyás dátuma: ${positionDescription.approvedAt}
  `;

  const document = await Document.create({
    documentType: 'JOB_DESCRIPTION',
    fileName: `Munkakori_leiras_${template.code}_v${positionDescription.version}.pdf`,
    fileUrl: await uploadPdfToStorage(pdfContent),
    version: `v${positionDescription.version}`,
    effectiveFrom: positionDescription.effectiveFrom,
    approvedBy: positionDescription.approvedBy
  });

  return document;
}
```

**Strategic megoldás: Kompetencia-alapú munkaköri leírás**

Kompetencia-modell integrálása a munkaköri leírásba.

```typescript
Competency {
  id: UUID PK
  organizationId: UUID FK
  code: string
  name: string  // pl. "Kommunikációs készség", "Vezetői kompetencia"
  category: enum (TECHNICAL, BEHAVIORAL, LEADERSHIP)
  description: text
  levels: json  // [{ level: 1, name: "Alapszint", description: "..." }, ...]
}

PositionCompetency {
  id: UUID PK
  positionDescriptionId: UUID FK → PositionDescription
  competencyId: UUID FK → Competency
  requiredLevel: integer  // Mely szinten szükséges (1-5)
  importance: enum (MUST_HAVE, SHOULD_HAVE, NICE_TO_HAVE)
  assessmentMethod: enum (INTERVIEW, TEST, CERTIFICATION, OBSERVATION)
}
```

**Jogszabályi háttér:**
- **Mt. 46. §:** A munkáltató köteles munkaköri leírást adni – nincs előírt forma.
- **Kit. 38. §:** Álláshely-leírás kötelező tartalma: feladatok, hatáskörök, felelősségek, betöltési feltételek.
- **Kjt. / Kttv.:** Munkaköri leírás kötelező, tartalmi követelmény nincs.

**GDPR:**
A PositionDescription nem tartalmaz személyes adatot (kivéve approvedBy, ami FK → Person). Jogalap: GDPR 6(1)(f) – jogos érdek (szervezeti működés).

**Implementációs prioritás:**
- **MVP:** `description` text mező
- **Extended:** PositionDescription entitás strukturált mezőkkel + dokumentum-hivatkozás (ajánlott)
- **Strategic:** Kompetencia-modell integrációja

**Ajánlás:**
Extended modell (strukturált PositionDescription) bevezetése akkor, ha:
1. Teljesítménymenedzsment-rendszer használata tervezett (feladatok → KPI-ok)
2. Kompetencia-alapú értékelés szükséges
3. Munkaköri leírások gyakori frissítése és verzionálása
4. Közszféra (Kit., Kttv.) – kötelező álláshely-leírás részletes tartalommal

---

#### 12.1.3. Szervezeti ábra vs. beszámolási struktúra

**Probléma:** A `reportingToPositionId` jelentése nem egyértelmű:
- **Szervezeti hierarchia:** Az OrgUnit fastruktúrát tükrözi (osztályvezető → főosztályvezető)
- **Beszámolási lánc:** Funkcionális beszámolás (projektmunkatárs → projektvezető, DE szervezetileg a mérnöki osztályhoz tartozik)

Mátrix szervezeteknél a kettő eltérhet.

**Javaslat: `reportingToPositionId` = funkcionális beszámolás, OrgUnit = szervezeti hierarchia (Extended ajánlás)**

**MVP megoldás:**
`reportingToPositionId` a szervezeti hierarchiát tükrözi. Egy Position-nek egy felettese van.

```typescript
Position {
  // ...
  orgUnitId: UUID FK → OrgUnit  // Szervezeti egység
  reportingToPositionId: UUID FK → Position  // Közvetlen felettes (1:1 kapcsolat OrgUnit hierarchiával)
}
```

**Korlátok:**
- Mátrix szervezetben nem működik (funkcionális + projekt beszámolás)
- Rugalmatlan

**Extended megoldás (ajánlott):**
`reportingToPositionId` = **funkcionális beszámolás** (elsődleges), `orgUnitId` = szervezeti hierarchia. Másodlagos beszámolási vonalak külön entitásban.

```typescript
Position {
  id: UUID PK
  organizationId: UUID FK → Organization
  orgUnitId: UUID FK → OrgUnit  // SZERVEZETI hierarchia

  // Elsődleges funkcionális beszámolás
  reportingToPositionId: UUID FK → Position  // FUNKCIONÁLIS beszámolás
  reportingRelationshipType: enum (
    DIRECT,           // Közvetlen beszámolás
    FUNCTIONAL,       // Funkcionális beszámolás
    ADMINISTRATIVE,   // Adminisztratív beszámolás
    DOTTED_LINE       // Szaggatott vonal (weak)
  )

  // ...
}

// Másodlagos beszámolási vonalak (mátrix szervezet)
PositionReportingRelationship {
  id: UUID PK
  positionId: UUID FK → Position
  reportsToPositionId: UUID FK → Position
  relationshipType: enum (
    FUNCTIONAL,       // Funkcionális vonal (pl. projekt)
    ADMINISTRATIVE,   // Adminisztratív vonal (pl. HR)
    DOTTED_LINE,      // Szaggatott vonal
    MATRIX            // Mátrix kapcsolat
  )

  // Súlyozás (pl. teljesítményértékeléshez)
  weight: decimal(3,2)  // 0.0-1.0 (pl. 0.6 elsődleges, 0.4 másodlagos)

  // Jóváhagyási jogosultság
  hasApprovalAuthority: boolean  // Van-e jóváhagyási jogköre (pl. szabadság, távollét)

  // Hatályosság
  effectiveFrom: date
  effectiveTo: date
}
```

**Használat példa:**

```typescript
// 1. Egyszerű hierarchia (versenyszféra, kis szervezet)
// Szervezeti struktúra:
//   OrgUnit: Mérnöki osztály → Műszaki igazgatóság
// Beszámolási lánc:
//   Position: Mérnök → Osztályvezető mérnök → Műszaki igazgató

Position {
  id: 'pos-mernok-001'
  orgUnitId: 'orgunit-mernoki-osztaly'  // Mérnöki osztály
  reportingToPositionId: 'pos-osztályvezetomernök'  // Közvetlen felettes
  reportingRelationshipType: DIRECT
}

// 2. Mátrix szervezet
// Szervezeti struktúra:
//   OrgUnit: Mérnöki osztály (funkcionális otthon)
// Beszámolási lánc:
//   Elsődleges: Projekt vezető (60%)
//   Másodlagos: Osztályvezető mérnök (40%)

Position {
  id: 'pos-projekt-mernok-001'
  orgUnitId: 'orgunit-mernoki-osztaly'  // Szervezeti hierarchia
  reportingToPositionId: 'pos-projektvezeto'  // Elsődleges beszámolás (projekt)
  reportingRelationshipType: MATRIX
}

PositionReportingRelationship {
  positionId: 'pos-projekt-mernok-001'
  reportsToPositionId: 'pos-osztályvezetomernök'  // Másodlagos beszámolás (funkcionális)
  relationshipType: FUNCTIONAL
  weight: 0.4
  hasApprovalAuthority: true  // Szabadságot ő is jóváhagyhatja
  effectiveFrom: '2025-01-01'
  effectiveTo: null
}
```

**SQL lekérdezés: Teljes beszámolási lánc**

```sql
-- Lekérdezés: Egy munkavállaló összes felettese (közvetlen + mátrix)
WITH RECURSIVE ReportingChain AS (
  -- Kezdőpont: munkavállaló pozíciója
  SELECT
    p.id AS positionId,
    p.positionTitle,
    p.reportingToPositionId AS supervisorPositionId,
    p.reportingRelationshipType,
    1.0 AS weight,
    1 AS level,
    'PRIMARY' AS lineType
  FROM Employment e
  JOIN Position p ON e.positionId = p.id
  WHERE e.id = 'emp-uuid'

  UNION ALL

  -- Rekurzió: felettes pozíciók
  SELECT
    p_super.id,
    p_super.positionTitle,
    p_super.reportingToPositionId,
    p_super.reportingRelationshipType,
    1.0,
    rc.level + 1,
    'PRIMARY'
  FROM ReportingChain rc
  JOIN Position p_super ON rc.supervisorPositionId = p_super.id
  WHERE p_super.reportingToPositionId IS NOT NULL
)
SELECT
  rc.level,
  rc.lineType,
  rc.positionTitle,
  rc.weight
FROM ReportingChain rc

UNION

-- Másodlagos beszámolási vonalak
SELECT
  1 AS level,
  'SECONDARY' AS lineType,
  p_sec.positionTitle,
  prr.weight
FROM Employment e
JOIN Position p ON e.positionId = p.id
JOIN PositionReportingRelationship prr ON p.id = prr.positionId
JOIN Position p_sec ON prr.reportsToPositionId = p_sec.id
WHERE e.id = 'emp-uuid'
  AND prr.effectiveTo IS NULL

ORDER BY lineType, level;
```

**Teljesítményértékelés súlyozással:**

```javascript
async function calculatePerformanceRating(employmentId) {
  const employment = await Employment.findById(employmentId);
  const position = await Position.findById(employment.positionId);

  // 1. Elsődleges felettes értékelése
  const primarySupervisor = await Position.findById(position.reportingToPositionId);
  const primaryRating = await getPerformanceRating(employmentId, primarySupervisor.id);
  const primaryWeight = 1.0;  // Alapértelmezés: 100%

  // 2. Másodlagos felettesek értékelése (mátrix)
  const secondaryRelationships = await PositionReportingRelationship.findAll({
    where: {
      positionId: position.id,
      effectiveTo: null
    }
  });

  let totalWeight = primaryWeight;
  let weightedRatingSum = primaryRating * primaryWeight;

  for (const rel of secondaryRelationships) {
    const secondaryRating = await getPerformanceRating(employmentId, rel.reportsToPositionId);
    weightedRatingSum += secondaryRating * rel.weight;
    totalWeight += rel.weight;
  }

  // 3. Súlyozott átlag
  const finalRating = weightedRatingSum / totalWeight;

  return {
    finalRating: finalRating,
    primaryRating: primaryRating,
    secondaryRatings: secondaryRelationships.map(rel => ({
      supervisorPositionId: rel.reportsToPositionId,
      rating: rel.rating,
      weight: rel.weight
    })),
    totalWeight: totalWeight
  };
}
```

**Strategic megoldás: Szervezeti ábra vs. Beszámolási diagram**

Két különálló nézet:
- **Organizational Chart:** OrgUnit hierarchia alapján
- **Reporting Structure:** reportingToPositionId + PositionReportingRelationship alapján

```javascript
// Szervezeti ábra (OrgUnit alapú)
async function generateOrganizationalChart(orgUnitId) {
  const orgUnit = await OrgUnit.findById(orgUnitId);
  const positions = await Position.findAll({
    where: { orgUnitId: orgUnitId, effectiveTo: null }
  });

  return {
    orgUnitName: orgUnit.name,
    positions: positions.map(p => ({
      positionTitle: p.positionTitle,
      status: p.status,
      currentEmployees: p.currentHeadcount
    })),
    childOrgUnits: await getChildOrgUnits(orgUnitId)
  };
}

// Beszámolási diagram (reportingToPositionId alapú)
async function generateReportingStructure(positionId) {
  const position = await Position.findById(positionId);

  const directReports = await Position.findAll({
    where: { reportingToPositionId: positionId, effectiveTo: null }
  });

  const matrixReports = await db.query(`
    SELECT p.*, prr.relationshipType, prr.weight
    FROM Position p
    JOIN PositionReportingRelationship prr ON p.id = prr.positionId
    WHERE prr.reportsToPositionId = ?
      AND prr.effectiveTo IS NULL
  `, [positionId]);

  return {
    position: position.positionTitle,
    directReports: directReports.map(p => p.positionTitle),
    matrixReports: matrixReports.map(m => ({
      position: m.positionTitle,
      relationshipType: m.relationshipType,
      weight: m.weight
    }))
  };
}
```

**Jogszabályi háttér:**
- **Mt.:** Nincs közvetlen szabályozás a beszámolási struktúrára, de a munkaköri leírásban (Mt. 46. §) rögzíteni kell.
- **Kit. / Kttv.:** Álláshely-leírásban szerepelnie kell a közvetlen felettesnek.
- **Közszféra:** Általában egyértelmű, hierarchikus beszámolás (mátrix ritkább).

**GDPR:**
Nem releváns (munkakörök nem személyes adatok).

**Implementációs prioritás:**
- **MVP:** `reportingToPositionId` = szervezeti hierarchia
- **Extended:** `reportingToPositionId` = funkcionális beszámolás + PositionReportingRelationship másodlagos vonalakhoz (ajánlott mátrix szervezeteknek)
- **Strategic:** Külön szervezeti ábra és beszámolási diagram generálás

**Ajánlás:**
Extended modell (reportingToPositionId + PositionReportingRelationship) bevezetése akkor, ha:
1. Mátrix szervezet (funkcionális + projekt / regionális + funkcionális)
2. Teljesítményértékelés több felettestől
3. Komplex jóváhagyási folyamatok (szabadság, távollét, kiadások) több szintről

---

#### 12.1.4. FEOR karbantartás – Saját törzsadat vs. Integráció

**Probléma:** A FEOR-08 (Foglalkozások Egységes Osztályozási Rendszere) kódlista:
- Forrás: KSH
- Frissítési gyakoriság: Évente 1-2 alkalommal
- Használat: NAV T1041 bejelentés, KSH statisztikai adatszolgáltatás

**Kérdés:** Saját FEOR törzsadat (manuális karbantartás) vagy KSH integráció?

**Javaslat: Saját FEOR törzsadat + periodikus KSH szinkronizáció (Extended ajánlás)**

**MVP megoldás:**
FEOR kód string mezőként tárolva a PositionTemplate-ben, nincs külön törzsadat.

```typescript
PositionTemplate {
  // ...
  feorCode: string(6)  // pl. "234101"
  feorName: string     // pl. "Óvodapedagógus"
}
```

**Korlátok:**
- FEOR-kód validáció nincs (hibás kódok)
- Nem kezeli a FEOR hierarchiát (főcsoport, alcsoport, foglalkozás)
- Frissítések nehézkes manuális munka

**Extended megoldás (ajánlott):**
Önálló `FEOR` törzsadat entitás + periodikus szinkronizáció a KSH kódlistájával.

```typescript
FEOR {
  id: UUID PK
  code: string(6) UNIQUE  // FEOR-08 kód (pl. "234101")
  name: string  // Foglalkozás megnevezése

  // Hierarchia
  level: integer  // 1 = főcsoport, 2 = csoport, 3 = alcsoport, 4 = foglalkozás
  parentCode: string(6) FK → FEOR.code  // Szülő kategória
  path: string  // Materialized path (pl. "/2/23/234/234101/")

  // Metaadatok
  mainGroupCode: string(1)  // Főcsoport kód (1-9)
  mainGroupName: string     // Főcsoport megnevezés
  groupCode: string(2)      // Csoport kód (pl. "23")
  groupName: string
  subGroupCode: string(3)   // Alcsoport kód (pl. "234")
  subGroupName: string

  // ISCO megfeleltetés
  iscoCode: string  // ISCO-08 kód
  iscoName: string

  // Státusz
  isActive: boolean
  deprecatedDate: date  // Mikor lett érvénytelen (ha törölték a kódlistából)
  replacedByCode: string(6) FK → FEOR.code  // Mivel helyettesítették

  // Forrás
  sourceSystem: enum (KSH, MANUAL)
  lastSyncedAt: datetime

  // Audit
  createdAt: datetime
  createdBy: string
  updatedAt: datetime
  updatedBy: string
}

// PositionTemplate módosítás
PositionTemplate {
  // ...
  feorCode: string(6) FK → FEOR.code  // KÖTELEZŐ, foreign key
}
```

**FEOR hierarchia példa:**

```
1 - Jogalkotók, felső vezetők
  11 - Jogalkotók és felsőszintű állami vezetők
    111 - Jogalkotók és felsőszintű állami tisztviselők
      111001 - Országgyűlési képviselő
      111002 - Miniszter
  12 - Vállalkozások felső vezetői
    121 - Vállalkozások vezetői
      121001 - Vezérigazgató

2 - Felsőfokú képzettséget igénylő foglalkozások
  23 - Oktatás és nevelés foglalkozásai
    234 - Óvodai és általános iskolai nevelők
      234101 - Óvodapedagógus
      234102 - Tanító
```

**KSH szinkronizáció workflow:**

```javascript
// 1. FEOR kódlista letöltése KSH-tól (CSV/XML)
async function syncFeorFromKSH() {
  const kshData = await fetchFeorListFromKSH();  // Külső API vagy CSV letöltés

  const stats = {
    newCodes: 0,
    updatedCodes: 0,
    deprecatedCodes: 0
  };

  for (const row of kshData) {
    const existingFeor = await FEOR.findOne({ where: { code: row.code } });

    if (!existingFeor) {
      // Új FEOR kód
      await FEOR.create({
        code: row.code,
        name: row.name,
        level: row.level,
        parentCode: row.parentCode,
        mainGroupCode: row.mainGroupCode,
        mainGroupName: row.mainGroupName,
        iscoCode: row.iscoCode,
        isActive: true,
        sourceSystem: 'KSH',
        lastSyncedAt: new Date()
      });
      stats.newCodes++;
    } else {
      // Meglévő FEOR kód frissítése
      if (existingFeor.name !== row.name) {
        existingFeor.name = row.name;
        existingFeor.lastSyncedAt = new Date();
        await existingFeor.save();
        stats.updatedCodes++;
      }
    }
  }

  // 2. Deprecated kódok kezelése (KSH-ban már nincs, de nálunk van)
  const allFEORCodes = kshData.map(r => r.code);
  const deprecatedFeors = await FEOR.findAll({
    where: {
      code: { $notIn: allFEORCodes },
      isActive: true,
      sourceSystem: 'KSH'
    }
  });

  for (const feor of deprecatedFeors) {
    feor.isActive = false;
    feor.deprecatedDate = new Date();
    await feor.save();
    stats.deprecatedCodes++;

    // Riasztás: ez a FEOR kód még használatban van?
    const usageCount = await PositionTemplate.count({
      where: { feorCode: feor.code }
    });
    if (usageCount > 0) {
      await sendAlert({
        type: 'FEOR_DEPRECATED',
        message: `FEOR kód ${feor.code} (${feor.name}) deprecated, de ${usageCount} db PositionTemplate használja`,
        feorCode: feor.code,
        affectedPositions: usageCount
      });
    }
  }

  return stats;
}

// 2. Validáció: FEOR kód érvényessége
function validateFeorCode(feorCode) {
  const feor = await FEOR.findOne({ where: { code: feorCode } });

  if (!feor) {
    throw new ValidationError(`Érvénytelen FEOR kód: ${feorCode}`);
  }

  if (!feor.isActive) {
    if (feor.replacedByCode) {
      throw new ValidationError(
        `FEOR kód ${feorCode} (${feor.name}) érvénytelen, használja helyette: ${feor.replacedByCode}`
      );
    } else {
      throw new ValidationError(
        `FEOR kód ${feorCode} (${feor.name}) érvénytelen (deprecated: ${feor.deprecatedDate})`
      );
    }
  }

  return feor;
}
```

**FEOR hierarchikus lekérdezés:**

```sql
-- Lekérdezés: Összes pedagógus munkakör (FEOR főcsoport 23)
SELECT
  pt.name AS positionName,
  f.code AS feorCode,
  f.name AS feorName,
  f.mainGroupName
FROM PositionTemplate pt
JOIN FEOR f ON pt.feorCode = f.code
WHERE f.mainGroupCode = '23'  -- Oktatás és nevelés
  AND f.isActive = true;

-- Lekérdezés: FEOR hierarchia (fastruktúra)
WITH RECURSIVE FeorTree AS (
  -- Gyökér: főcsoport
  SELECT code, name, parentCode, level, 0 AS depth, code AS rootCode
  FROM FEOR
  WHERE level = 1 AND code = '2'  -- Felsőfokú képzettséget igénylő foglalkozások

  UNION ALL

  -- Gyerekek
  SELECT f.code, f.name, f.parentCode, f.level, ft.depth + 1, ft.rootCode
  FROM FEOR f
  JOIN FeorTree ft ON f.parentCode = ft.code
  WHERE f.isActive = true
)
SELECT
  REPEAT('  ', depth) || name AS hierarchicalName,
  code,
  level
FROM FeorTree
ORDER BY code;
```

**Strategic megoldás: Automatikus FEOR javaslat**

A rendszer a munkakör neve és leírása alapján javasol FEOR kódot (gépi tanulás vagy keyword-alapú).

```javascript
async function suggestFeorCode(positionName, description) {
  // Egyszerű keyword-alapú javaslat
  const keywords = {
    'pedagógus|tanár|tanító|oktató': ['234%'],  // Oktatás és nevelés
    'orvos|szakorvos': ['221%'],  // Egészségügyi foglalkozások
    'ápoló|asszisztens': ['222%'],
    'mérnök': ['214%'],  // Mérnökök
    'ügyintéző|adminisztrátor': ['333%', '411%']
  };

  for (const [pattern, feorPatterns] of Object.entries(keywords)) {
    const regex = new RegExp(pattern, 'i');
    if (regex.test(positionName) || regex.test(description)) {
      const suggestions = await FEOR.findAll({
        where: {
          code: { $like: feorPatterns },
          isActive: true
        },
        limit: 5
      });
      return suggestions;
    }
  }

  return [];
}
```

**Jogszabályi háttér:**
- **NAV bejelentés (T1041):** FEOR kód kötelező mező a foglalkoztatás bejelentésénél.
- **KSH statisztikai adatszolgáltatás:** FEOR kód alapján történik a létszámadatok összesítése.

**GDPR:**
Nem releváns (FEOR törzsadat nem személyes adat).

**Implementációs prioritás:**
- **MVP:** FEOR kód string mezőként + manuális karbantartás
- **Extended:** FEOR entitás + KSH szinkronizáció (ajánlott)
- **Strategic:** Automatikus FEOR javaslat + gépi tanulás

**Ajánlás:**
Extended modell (FEOR entitás + szinkronizáció) bevezetése, mert:
1. Validálja a FEOR kódokat (hibás kódok elkerülése)
2. NAV bejelentés hibaarány csökken
3. KSH változásokat automatikusan követi
4. Hierarchikus lekérdezések (pl. „összes egészségügyi munkakör")

---

#### 12.1.5. Pedagógus óraszám-keret – Position vs. Employment szint

**Probléma:** A Púétv. szerinti heti kötelező tanítási óraszám:
- **Munkakörtől függ:** Óvodapedagógus: 32 óra, általános iskolai tanár: 22 óra, gimnáziumi tanár: 26 óra
- **Személyfüggő is lehet:** Igazgató: csökkentett óraszám (vezetői feladatok miatt), pályakezdő: mentorált

**Kérdés:** Hol tároljuk?
- **Position szintjén:** A munkakör előírja az óraszámot (általános, sablonszerű)
- **Employment szintjén:** A konkrét jogviszony határozza meg (személyre szabott)

**Javaslat: Position = keret (min-max), Employment = konkrét óraszám (Extended ajánlás)**

**MVP megoldás:**
Óraszám csak az Employment szintjén van tárolva.

```typescript
Employment {
  // ...
  puetv_weeklyTeachingHours: integer  // Konkrét heti óraszám
}
```

**Korlátok:**
- Nincs sablonszerű útmutatás (mi a "normál" óraszám erre a munkakörre?)
- Nehéz validálni (mi az elfogadható tartomány?)

**Extended megoldás (ajánlott):**
Position = óraszám-keret (min-max), Employment = konkrét óraszám (a keret határain belül).

```typescript
// Position szint (munkaköri keret)
Position {
  // ...
  // Púétv. specifikus mezők
  puetv_positionType: enum (PEDAGOGUE, PEDAGOGUE_SUPPORT, OTHER)
  puetv_isPedagogueCareer: boolean

  // Óraszám-keret (tartomány)
  puetv_weeklyTeachingHoursMin: integer  // Alsó határ (pl. igazgató: 0 óra)
  puetv_weeklyTeachingHoursMax: integer  // Felső határ (pl. tanár: 26 óra)
  puetv_standardWeeklyTeachingHours: integer  // "Normál" óraszám (pl. 22 óra)

  // Csökkentés indoklása
  puetv_reductionReason: enum (
    NONE,
    MANAGEMENT,       // Vezetői feladatok (Púétv. 69. §)
    MENTORING,        // Mentori feladatok
    METHODOLOGICAL,   // Módszertani feladatok
    SPECIAL_TASK,     // Különleges feladatok
    HEALTH            // Egészségügyi ok
  )
}

// Employment szint (konkrét óraszám)
Employment {
  // ...
  puetv_weeklyTeachingHours: integer  // KÖTELEZŐ (ha KOZNEVELESI jogviszony és pedagógus)
  puetv_weeklyTeachingHoursReduction: integer  // Csökkentés mértéke
  puetv_reductionReason: enum  // Csökkentés indoka (örökölhető Position-ből)
  puetv_reductionApprovedBy: UUID FK → Person
  puetv_reductionApprovedAt: datetime

  // Tényleges óraszám (opcionális, ha az Employment-et már betöltötték)
  puetv_actualWeeklyTeachingHours: integer  // Számított a tanóra-beosztásból
}

// Validáció
function validateTeachingHours(employment, position) {
  const hours = employment.puetv_weeklyTeachingHours;
  const min = position.puetv_weeklyTeachingHoursMin;
  const max = position.puetv_weeklyTeachingHoursMax;

  if (hours < min || hours > max) {
    throw new ValidationError(
      `Heti tanítási óraszám (${hours}) nem felel meg a munkaköri keretnek (${min}-${max} óra)`
    );
  }

  // Ha van csökkentés, indoklás kötelező
  const standard = position.puetv_standardWeeklyTeachingHours;
  if (hours < standard && !employment.puetv_reductionReason) {
    throw new ValidationError(
      `A csökkentett óraszám indoklása kötelező (standard: ${standard}, tényleges: ${hours})`
    );
  }

  return true;
}
```

**Használat példa:**

```typescript
// 1. Általános iskolai tanár munkakör
Position {
  id: 'pos-altalanos-tanar'
  positionTitle: 'Általános iskolai tanár'
  puetv_positionType: PEDAGOGUE
  puetv_weeklyTeachingHoursMin: 0      // Igazgató esetén 0 is lehet
  puetv_weeklyTeachingHoursMax: 22     // Púétv. 88. § (1) – általános iskolai tanár: max 22 óra
  puetv_standardWeeklyTeachingHours: 22
}

// 2. Konkrét jogviszony: normál pedagógus
Employment {
  id: 'emp-001'
  positionId: 'pos-altalanos-tanar'
  employmentType: KOZNEVELESI
  puetv_weeklyTeachingHours: 22  // Standard óraszám
  puetv_weeklyTeachingHoursReduction: 0
}

// 3. Konkrét jogviszony: osztályfőnök (csökkentett óraszám)
Employment {
  id: 'emp-002'
  positionId: 'pos-altalanos-tanar'
  employmentType: KOZNEVELESI
  puetv_weeklyTeachingHours: 20  // Csökkentett óraszám (osztályfőnöki feladatok miatt: +2 óra csökkentés)
  puetv_weeklyTeachingHoursReduction: 2
  puetv_reductionReason: SPECIAL_TASK
  puetv_reductionApprovedBy: 'person-igazgato'
  puetv_reductionApprovedAt: '2025-09-01'
}

// 4. Konkrét jogviszony: igazgató (jelentősen csökkentett óraszám)
Employment {
  id: 'emp-003'
  positionId: 'pos-igazgato'  // Külön igazgatói munkakör
  employmentType: KOZNEVELESI
  puetv_weeklyTeachingHours: 4  // Púétv. 69. § – igazgató: max. 4 óra tanítás
  puetv_weeklyTeachingHoursReduction: 18
  puetv_reductionReason: MANAGEMENT
}
```

**Óraszám-követés (tényleges vs. előírt):**

```javascript
// Tényleges óraszám számítása (tanóra-beosztásból)
async function calculateActualTeachingHours(employmentId, academicYear, semester) {
  // Lekérdezés: hány tanórát tart a pedagógus a megadott félévben
  const lessons = await db.query(`
    SELECT SUM(weeklyHours) AS totalHours
    FROM Lesson
    WHERE teacherEmploymentId = ?
      AND academicYear = ?
      AND semester = ?
  `, [employmentId, academicYear, semester]);

  return lessons.totalHours || 0;
}

// Óraszám-eltérés ellenőrzése
async function checkTeachingHoursCompliance(employmentId) {
  const employment = await Employment.findById(employmentId);
  const position = await Position.findById(employment.positionId);

  const prescribedHours = employment.puetv_weeklyTeachingHours;
  const actualHours = await calculateActualTeachingHours(employmentId, '2025/2026', 1);

  if (actualHours < prescribedHours) {
    return {
      status: 'UNDER_ALLOCATED',
      prescribedHours: prescribedHours,
      actualHours: actualHours,
      difference: prescribedHours - actualHours,
      message: `${employment.personId} pedagógusnak ${prescribedHours} óra lenne kötelező, de csak ${actualHours} órát tart`
    };
  } else if (actualHours > prescribedHours) {
    return {
      status: 'OVER_ALLOCATED',
      prescribedHours: prescribedHours,
      actualHours: actualHours,
      difference: actualHours - prescribedHours,
      message: `${employment.personId} pedagógus túlóráz: ${actualHours} óra (előírt: ${prescribedHours})`
    };
  } else {
    return { status: 'COMPLIANT', prescribedHours, actualHours };
  }
}
```

**Strategic megoldás: Óraszám-tervezés és órarend-generálás**

Integráció az órarend-tervező modullal.

```typescript
LessonPlan {
  id: UUID PK
  institutionId: UUID FK → OrgUnit (iskola)
  academicYear: string  // "2025/2026"
  semester: integer  // 1 vagy 2
  status: enum (DRAFT, APPROVED, ACTIVE, ARCHIVED)
}

Lesson {
  id: UUID PK
  lessonPlanId: UUID FK → LessonPlan
  subject: string  // Tantárgy (pl. "Matematika")
  grade: string    // Évfolyam (pl. "3.A")
  teacherEmploymentId: UUID FK → Employment
  weeklyHours: integer  // Heti óraszám (pl. 5 óra matematika a 3.A-ban)
  classroom: string
}

// Óraszám-tervezés validáció
function validateLessonPlan(lessonPlan) {
  const teachers = await db.query(`
    SELECT
      e.id AS employmentId,
      e.puetv_weeklyTeachingHours AS prescribedHours,
      SUM(l.weeklyHours) AS allocatedHours
    FROM Lesson l
    JOIN Employment e ON l.teacherEmploymentId = e.id
    WHERE l.lessonPlanId = ?
    GROUP BY e.id, e.puetv_weeklyTeachingHours
  `, [lessonPlan.id]);

  const errors = [];
  for (const teacher of teachers) {
    if (teacher.allocatedHours > teacher.prescribedHours) {
      errors.push({
        employmentId: teacher.employmentId,
        error: `Túlallokáció: ${teacher.allocatedHours} óra (max: ${teacher.prescribedHours})`
      });
    } else if (teacher.allocatedHours < teacher.prescribedHours) {
      errors.push({
        employmentId: teacher.employmentId,
        warning: `Alulallokáció: ${teacher.allocatedHours} óra (kötelező: ${teacher.prescribedHours})`
      });
    }
  }

  return errors;
}
```

**Jogszabályi háttér:**
- **Púétv. 88. § (1):** Heti kötelező tanítási óraszám munkakörtől függ:
  - Óvodapedagógus: 32 óra
  - Általános iskolai tanító, tanár: 22 óra
  - Gimnáziumi, szakgimnáziumi tanár: 24 óra (2026. szeptember 1-től 26 óra)
- **Púétv. 69. §:** Intézményvezető csökkentett óraszámban tanít (legfeljebb heti 4 óra).

**GDPR:**
Employment.puetv_weeklyTeachingHours személyes adat (foglalkoztatási adat). Jogalap: GDPR 6(1)(c) – jogi kötelezettség (Púétv. alapján nyilvántartandó).

**Implementációs prioritás:**
- **MVP:** Óraszám csak Employment szintjén
- **Extended:** Position = keret (min-max), Employment = konkrét (ajánlott)
- **Strategic:** Órarend-tervező integráció, óraszám-eltérés monitoring

**Ajánlás:**
Extended modell (Position keret + Employment konkrét) bevezetése köznevelési intézményekben, mert:
1. Munkaköri előírások egyértelműek (Position szint)
2. Személyre szabott óraszám (Employment szint: igazgató, osztályfőnök, mentori feladatok)
3. Validálható (óraszám a keret határain belül)
4. Óraterv-tervezés során segít (tényleges vs. előírt óraszám összehasonlítása)

---

#### 12.1.6. Álláshely-gazdálkodás és költségvetés – Position költségvetési vonzata

**Probléma:** Az álláshely nem csak szervezeti, hanem költségvetési egység is:
- **Alaplétszám:** Kit., Kttv., Küt. – álláshely-alapú létszámgazdálkodás
- **Bérköltség-tervezés:** Egy álláshely éves bérköltség-terve (pl. 8 M Ft/év)

**Kérdés:** A Position entitásban legyen költségvetési adat (plannedAnnualSalaryCost), vagy az a Compensation / pénzügyi modul feladata?

**Javaslat: Position = tervezett FTE + bérsáv, külön PositionBudget entitás (Extended ajánlás)**

**MVP megoldás:**
Position csak szervezeti adat, nincs költségvetési vonzat. A bérköltség a Compensation modulban van.

```typescript
Position {
  // ...
  headcount: decimal  // FTE
  // NINCS költségvetési adat
}
```

**Korlátok:**
- Nem lehet álláshely-szinten költségvetést tervezni
- Létszám-tervezés és bérköltség-tervezés szétválik

**Extended megoldás (ajánlott):**
Position tartalmaz bérsávot (min-max), külön PositionBudget entitás az éves költségvetésre.

```typescript
Position {
  id: UUID PK
  // ...meglévő mezők...

  // FTE
  headcount: decimal  // Tervezett létszám (1.0 = 1 FTE)

  // Bérsáv (Position szinten tartomány)
  salaryRangeMin: integer  // Bérsáv alsó határa (Ft/hó bruttó)
  salaryRangeMax: integer  // Bérsáv felső határa
  salaryRangeMid: integer  // Bérsáv közepe (target)

  // Költségvetési kategorizáció
  budgetCategory: enum (OPERATIONAL, CAPITAL, PROJECT)
  costCenterId: UUID FK → CostCenter

  // Státusz
  status: enum (PLANNED, VACANT, OCCUPIED, PARTIALLY_OCCUPIED, FROZEN, ABOLISHED)
}

// Külön PositionBudget entitás (éves költségvetési tervezés)
PositionBudget {
  id: UUID PK
  positionId: UUID FK → Position
  fiscalYear: integer  // Költségvetési év (pl. 2026)

  // Tervezett költségek
  plannedGrossSalary: integer  // Tervezett bruttó bér (Ft/hó)
  plannedAnnualGrossSalary: integer  // Éves bruttó bér (Ft/év, 12 hó)
  plannedEmployerContributions: integer  // Járulékok (szoc.cho, SZOCHO)
  plannedTotalAnnualCost: integer  // Teljes éves költség (bér + járulék)

  // Tervezett FTE
  plannedFTE: decimal  // Tervezett FTE (örökölhető Position.headcount-ból)

  // Költségvetési jóváhagyás
  budgetApprovalStatus: enum (DRAFT, SUBMITTED, APPROVED, REJECTED, FROZEN)
  approvedBy: UUID FK → Person
  approvedAt: datetime

  // Tényleges költségek (év során)
  actualGrossSalary: integer  // Tényleges bruttó bér (számított Compensation-ből)
  actualAnnualCost: integer  // Tényleges éves költség (year-to-date)

  // Eltérés
  variance: integer (computed)  // plannedTotalAnnualCost - actualAnnualCost
  variancePercentage: decimal(5,2) (computed)

  // Megjegyzés
  notes: text

  // Audit
  createdAt: datetime
  createdBy: string
  updatedAt: datetime
  updatedBy: string
}
```

**Használat példa:**

```typescript
// 1. Új álláshely költségvetési tervezése (2026)
Position {
  id: 'pos-hr-001'
  positionCode: 'POS-HR-001'
  positionTitle: 'HR ügyintéző'
  headcount: 1.0  // 1 FTE
  salaryRangeMin: 450000  // 450k Ft/hó
  salaryRangeMax: 650000  // 650k Ft/hó
  salaryRangeMid: 550000  // 550k Ft/hó (target)
  status: VACANT
}

PositionBudget {
  id: 'pb-001'
  positionId: 'pos-hr-001'
  fiscalYear: 2026

  plannedGrossSalary: 550000  // Tervezett bruttó bér (bérsáv közepe)
  plannedAnnualGrossSalary: 6600000  // 12 hó * 550k
  plannedEmployerContributions: 858000  // 13% (SZOCHO)
  plannedTotalAnnualCost: 7458000  // Teljes éves költség

  plannedFTE: 1.0
  budgetApprovalStatus: APPROVED
  approvedBy: 'person-cfo'
  approvedAt: '2025-11-15'
}

// 2. Álláshely betöltése (Employment létrehozása)
Employment {
  id: 'emp-001'
  personId: 'person-001'
  positionId: 'pos-hr-001'
  startDate: '2026-01-15'

  // Kompenzáció
  baseGrossSalary: 520000  // Tényleges bér (bérsávon belül)
}

Compensation {
  employmentId: 'emp-001'
  effectiveFrom: '2026-01-15'
  grossAmount: 520000  // Tényleges bruttó bér
}

// 3. Költségvetés-eltérés számítása (év közben)
PositionBudget {
  id: 'pb-001'
  // ...
  actualGrossSalary: 520000  // Tényleges bér (az Employment-ből számítva)
  actualAnnualCost: 7181600  // 12 hó * 520k * 1.13
  variance: 276400  // Megtakarítás (tervezetthez képest)
  variancePercentage: 3.7  // 3.7% megtakarítás
}
```

**Költségvetés-tervezés workflow:**

```javascript
// 1. Éves költségvetés generálása (összes pozícióra)
async function generateAnnualBudget(organizationId, fiscalYear) {
  const positions = await Position.findAll({
    where: {
      organizationId: organizationId,
      effectiveTo: null,
      status: { $in: ['PLANNED', 'VACANT', 'OCCUPIED', 'PARTIALLY_OCCUPIED'] }
    }
  });

  const budgets = [];
  for (const position of positions) {
    // Tervezett bér = bérsáv közepe (salaryRangeMid)
    const plannedGrossSalary = position.salaryRangeMid || position.salaryRangeMin;
    const plannedAnnualGrossSalary = plannedGrossSalary * 12;
    const plannedEmployerContributions = Math.round(plannedAnnualGrossSalary * 0.13);  // SZOCHO 13%
    const plannedTotalAnnualCost = plannedAnnualGrossSalary + plannedEmployerContributions;

    const budget = await PositionBudget.create({
      positionId: position.id,
      fiscalYear: fiscalYear,
      plannedGrossSalary: plannedGrossSalary,
      plannedAnnualGrossSalary: plannedAnnualGrossSalary,
      plannedEmployerContributions: plannedEmployerContributions,
      plannedTotalAnnualCost: plannedTotalAnnualCost,
      plannedFTE: position.headcount,
      budgetApprovalStatus: 'DRAFT'
    });

    budgets.push(budget);
  }

  return budgets;
}

// 2. Költségvetés-eltérés számítása (tényleges vs. tervezett)
async function calculateBudgetVariance(organizationId, fiscalYear) {
  const budgets = await PositionBudget.findAll({
    where: { fiscalYear: fiscalYear },
    include: [{ model: Position, where: { organizationId: organizationId } }]
  });

  for (const budget of budgets) {
    // Tényleges költség számítása (year-to-date)
    const actualCost = await db.query(`
      SELECT SUM(c.grossAmount * 12 * 1.13) AS totalCost
      FROM Employment e
      JOIN Compensation c ON e.id = c.employmentId
      WHERE e.positionId = ?
        AND e.startDate <= ?
        AND (e.endDate IS NULL OR e.endDate >= ?)
        AND c.effectiveTo IS NULL
    `, [budget.positionId, `${fiscalYear}-12-31`, `${fiscalYear}-01-01`]);

    budget.actualAnnualCost = actualCost.totalCost || 0;
    budget.variance = budget.plannedTotalAnnualCost - budget.actualAnnualCost;
    budget.variancePercentage = (budget.variance / budget.plannedTotalAnnualCost) * 100;
    await budget.save();
  }

  return budgets;
}

// 3. Összesített költségvetési riport
async function generateBudgetReport(organizationId, fiscalYear) {
  const summary = await db.query(`
    SELECT
      COUNT(*) AS totalPositions,
      SUM(pb.plannedFTE) AS totalPlannedFTE,
      SUM(pb.plannedTotalAnnualCost) AS totalPlannedCost,
      SUM(pb.actualAnnualCost) AS totalActualCost,
      SUM(pb.variance) AS totalVariance,
      AVG(pb.variancePercentage) AS avgVariancePercentage
    FROM PositionBudget pb
    JOIN Position p ON pb.positionId = p.id
    WHERE p.organizationId = ?
      AND pb.fiscalYear = ?
  `, [organizationId, fiscalYear]);

  return summary;
}
```

**Költséghely-szintű költségvetés:**

```sql
-- Lekérdezés: Költségvetés költséghelyenként
SELECT
  cc.code AS costCenterCode,
  cc.name AS costCenterName,
  COUNT(DISTINCT p.id) AS positionCount,
  SUM(pb.plannedFTE) AS plannedFTE,
  SUM(pb.plannedTotalAnnualCost) AS plannedTotalCost,
  SUM(pb.actualAnnualCost) AS actualTotalCost,
  SUM(pb.variance) AS totalVariance
FROM PositionBudget pb
JOIN Position p ON pb.positionId = p.id
LEFT JOIN CostCenter cc ON p.costCenterId = cc.id
WHERE p.organizationId = 'org-001'
  AND pb.fiscalYear = 2026
GROUP BY cc.id, cc.code, cc.name
ORDER BY plannedTotalCost DESC;
```

**Strategic megoldás: Forgatókönyv-alapú költségvetés-tervezés**

Többváltozós költségvetés (optimista, realista, pesszimista forgatókönyvek).

```typescript
BudgetScenario {
  id: UUID PK
  organizationId: UUID FK
  fiscalYear: integer
  scenarioType: enum (OPTIMISTIC, REALISTIC, PESSIMISTIC, CUSTOM)
  name: string
  description: text

  // Feltételezések
  assumedSalaryIncrease: decimal(5,2)  // Átlagos béremelés % (pl. 8%)
  assumedHeadcountGrowth: decimal(5,2)  // Létszámnövekedés % (pl. 5%)

  totalPlannedCost: integer (computed)
}
```

**Jogszabályi háttér:**
- **Kit. 35. § (4):** Álláshely-alapú létszámgazdálkodás – a szervezet alaplétszámát az álláshelyek száma határozza meg.
- **Áht.:** Költségvetési szervek költségvetési tervezése kötelező (bérköltség tervezése).

**GDPR:**
PositionBudget nem tartalmaz személyes adatot (tervezett, nem tényleges bérek).

**Implementációs prioritás:**
- **MVP:** Position csak szervezeti adat, költségvetés a Compensation modulban
- **Extended:** Position bérsáv + PositionBudget entitás (ajánlott)
- **Strategic:** Forgatókönyv-alapú költségvetés-tervezés

**Ajánlás:**
Extended modell (Position bérsáv + PositionBudget) bevezetése akkor, ha:
1. Álláshely-alapú létszámgazdálkodás (Kit., Kttv., Küt.)
2. Költségvetési tervezés szükséges (közszféra, nagyobb szervezetek)
3. Bérköltség-eltérés monitoring (tervezett vs. tényleges)
4. Költséghely-szintű bérköltség-aggregáció

---

#### 12.1.7. Közalkalmazotti munkaköri bértábla – QualificationRequirement vs. Külön törzsadat

**Probléma:** A Kjt. ágazati végrehajtási rendeletei (szociális, kulturális, közgyűjteményi) saját munkakör-jegyzéket tartalmaznak, amelyek a fizetési osztályt határozzák meg. Példa:

- **150/1992. (XI. 20.) Korm. rendelet (szociális):**
  „Szociális asszisztens" → E fizetési osztály
  „Szociális munkás" → F fizetési osztály

- **150/1992. (XI. 20.) Korm. rendelet 2. sz. melléklet (kulturális):**
  „Könyvtáros" → F fizetési osztály
  „Levéltáros" → G fizetési osztály

**Kérdés:** Ezeket a munkakör-jegyzékeket:
1. QualificationRequirement entitásban kezeljük (követelmény: „szociális munkás végzettség")
2. Külön `PayGradeMatrix` vagy `JobCatalog` törzsadat szükséges?

**Javaslat: Külön JobCatalog (munkakör-jegyzék) törzsadat (Extended ajánlás)**

**MVP megoldás:**
QualificationRequirement-ben tárolva, szöveges formában.

```typescript
QualificationRequirement {
  // ...
  requirementType: 'EDUCATION_FIELD'
  description: 'Szociális munkás végzettség (150/1992. Korm. r.)'
  isMandatory: true
}

Position {
  // ...
  kjt_requiredPayGrade: 'F'  // Manuálisan beállítva
}
```

**Korlátok:**
- Nem strukturált → nincs konzisztencia (egyik helyen "szociális munkás", másik helyen "szoc. munkás")
- Fizetési osztály és végzettség közötti kapcsolat nem automatikus
- Nehéz karbantartani (jogszabály-változásnál minden Position-t manuálisan frissíteni kell)

**Extended megoldás (ajánlott):**
Külön `JobCatalog` (munkakör-jegyzék) törzsadat az ágazati munkaköröknek.

```typescript
JobCatalog {
  id: UUID PK
  code: string UNIQUE  // Munkakör kódja (belső, pl. "KJT-SZOC-001")
  name: string  // Munkakör hivatalos megnevezése
  sector: enum (SOCIAL, CULTURAL, LIBRARY, ARCHIVE, MUSEUM, EDUCATION, HEALTH)  // Ágazat

  // Jogszabályi hivatkozás
  legalBasis: string  // pl. "150/1992. (XI. 20.) Korm. r. 2. sz. melléklet"
  legalReference: string  // Konkrét §

  // Kjt. besorolás
  kjt_requiredPayGrade: enum (A, B, C, D, E, F, G, H, I, J)  // Fizetési osztály
  kjt_alternativePayGrade: enum  // Megengedett alternatív (ha van)

  // Végzettségi követelmény (automatikusan hozzárendelhető)
  requiredEducationLevel: enum (PRIMARY, SECONDARY, TERTIARY_BA, TERTIARY_MA, TERTIARY_PHD)
  requiredEducationField: string  // pl. "Szociális munka", "Könyvtártudomány"
  requiredProfessionalQualification: string  // pl. "Szociális asszisztens OKJ"

  // FEOR kapcsolat (ha van)
  feorCode: string(6) FK → FEOR.code

  // Státusz
  isActive: boolean
  deprecatedDate: date
  replacedByJobCatalogId: UUID FK → JobCatalog

  // Metaadatok
  description: text
  notes: text

  // Audit
  createdAt: datetime
  createdBy: string
  updatedAt: datetime
  updatedBy: string
}

// PositionTemplate módosítás
PositionTemplate {
  // ...
  jobCatalogId: UUID FK → JobCatalog  // Hivatkozás a munkakör-jegyzékre (opcionális)

  // HA jobCatalogId van, akkor örökölhető:
  kjt_requiredPayGrade: enum (computed)  // JobCatalog.kjt_requiredPayGrade-ből
}
```

**Használat példa:**

```typescript
// 1. Szociális munkakör-jegyzék (150/1992. Korm. r.)
JobCatalog {
  id: 'jc-szoc-001'
  code: 'KJT-SZOC-ASSZISZTENS'
  name: 'Szociális asszisztens'
  sector: SOCIAL
  legalBasis: '150/1992. (XI. 20.) Korm. rendelet 1. sz. melléklet'

  kjt_requiredPayGrade: 'E'
  kjt_alternativePayGrade: null

  requiredEducationLevel: SECONDARY
  requiredProfessionalQualification: 'Szociális asszisztens OKJ 54 761 01'
  feorCode: '532101'  // Szociális asszisztens

  isActive: true
}

JobCatalog {
  id: 'jc-szoc-002'
  code: 'KJT-SZOC-MUNKAS'
  name: 'Szociális munkás'
  sector: SOCIAL
  legalBasis: '150/1992. (XI. 20.) Korm. rendelet 1. sz. melléklet'

  kjt_requiredPayGrade: 'F'

  requiredEducationLevel: TERTIARY_BA
  requiredEducationField: 'Szociális munka (BA/MA)'
  feorCode: '263401'  // Szociális munkás

  isActive: true
}

// 2. Kulturális munkakör-jegyzék (150/1992. Korm. r. 2. sz. melléklet)
JobCatalog {
  id: 'jc-kult-001'
  code: 'KJT-KULT-KONYVTAROS'
  name: 'Könyvtáros'
  sector: LIBRARY
  legalBasis: '150/1992. (XI. 20.) Korm. rendelet 2. sz. melléklet'

  kjt_requiredPayGrade: 'F'
  requiredEducationLevel: TERTIARY_BA
  requiredEducationField: 'Könyvtártudomány (BA)'
  feorCode: '262201'  // Könyvtáros

  isActive: true
}

// 3. PositionTemplate hivatkozás a JobCatalog-ra
PositionTemplate {
  id: 'template-001'
  organizationId: 'org-szoc-001'
  code: 'SZOC-MUNKAS'
  name: 'Szociális munkás'
  jobCatalogId: 'jc-szoc-002'  // Hivatkozás a munkakör-jegyzékre

  // Automatikusan öröklődik:
  kjt_requiredPayGrade: 'F'  // = JobCatalog.kjt_requiredPayGrade
  feorCode: '263401'  // = JobCatalog.feorCode
}

// 4. QualificationRequirement automatikus generálása
QualificationRequirement {
  positionTemplateId: 'template-001'
  requirementType: 'EDUCATION_LEVEL'
  description: 'Felsőfokú végzettség (BA/MA)'
  isMandatory: true
  legalBasis: '150/1992. (XI. 20.) Korm. r.'
}

QualificationRequirement {
  positionTemplateId: 'template-001'
  requirementType: 'EDUCATION_FIELD'
  description: 'Szociális munka szakirány'
  isMandatory: true
  legalBasis: '150/1992. (XI. 20.) Korm. r.'
}
```

**Automatikus követelmény-generálás:**

```javascript
async function generateQualificationRequirementsFromJobCatalog(positionTemplateId) {
  const template = await PositionTemplate.findById(positionTemplateId);

  if (!template.jobCatalogId) {
    return [];  // Nincs munkakör-jegyzék hivatkozás
  }

  const jobCatalog = await JobCatalog.findById(template.jobCatalogId);

  const requirements = [];

  // 1. Végzettségi szint
  if (jobCatalog.requiredEducationLevel) {
    requirements.push({
      positionTemplateId: positionTemplateId,
      requirementType: 'EDUCATION_LEVEL',
      description: mapEducationLevelToHungarian(jobCatalog.requiredEducationLevel),
      isMandatory: true,
      legalBasis: jobCatalog.legalBasis
    });
  }

  // 2. Szakterület
  if (jobCatalog.requiredEducationField) {
    requirements.push({
      positionTemplateId: positionTemplateId,
      requirementType: 'EDUCATION_FIELD',
      description: jobCatalog.requiredEducationField,
      isMandatory: true,
      legalBasis: jobCatalog.legalBasis
    });
  }

  // 3. Szakképesítés
  if (jobCatalog.requiredProfessionalQualification) {
    requirements.push({
      positionTemplateId: positionTemplateId,
      requirementType: 'PROFESSIONAL_QUALIFICATION',
      description: jobCatalog.requiredProfessionalQualification,
      isMandatory: true,
      legalBasis: jobCatalog.legalBasis
    });
  }

  // QualificationRequirement-ek létrehozása
  for (const req of requirements) {
    await QualificationRequirement.create(req);
  }

  return requirements;
}

function mapEducationLevelToHungarian(level) {
  const map = {
    PRIMARY: 'Általános iskolai végzettség',
    SECONDARY: 'Középfokú végzettség',
    TERTIARY_BA: 'Felsőfokú végzettség (alapképzés, BA/BSc)',
    TERTIARY_MA: 'Felsőfokú végzettség (mesterképzés, MA/MSc)',
    TERTIARY_PHD: 'Doktori végzettség (PhD/DLA)'
  };
  return map[level] || level;
}
```

**Jogszabály-változás kezelése:**

```javascript
// Ha változik a jogszabály (pl. egy munkakör fizetési osztálya emelkedik E-ről F-re)
async function updateJobCatalogPayGrade(jobCatalogId, newPayGrade, legalBasis) {
  const jobCatalog = await JobCatalog.findById(jobCatalogId);

  // 1. JobCatalog frissítése
  jobCatalog.kjt_requiredPayGrade = newPayGrade;
  jobCatalog.legalBasis = legalBasis;
  jobCatalog.updatedAt = new Date();
  await jobCatalog.save();

  // 2. Érintett PositionTemplate-ek frissítése
  const templates = await PositionTemplate.findAll({
    where: { jobCatalogId: jobCatalogId }
  });

  for (const template of templates) {
    template.kjt_requiredPayGrade = newPayGrade;
    await template.save();

    console.log(`PositionTemplate ${template.code} frissítve: fizetési osztály ${newPayGrade}`);
  }

  // 3. Értesítés az érintett Employment-ekről
  const employments = await db.query(`
    SELECT e.id, e.personId, p.positionTitle
    FROM Employment e
    JOIN Position p ON e.positionId = p.id
    JOIN PositionTemplate pt ON p.positionTemplateId = pt.id
    WHERE pt.jobCatalogId = ?
      AND e.endDate IS NULL
  `, [jobCatalogId]);

  // Értesítés HR részére: ezeknek az dolgozóknak a besorolását felül kell vizsgálni
  await sendAlert({
    type: 'JOB_CATALOG_CHANGE',
    message: `${jobCatalog.name} munkakör fizetési osztálya változott (${newPayGrade}). ${employments.length} db aktív dolgozót érint.`,
    jobCatalogId: jobCatalogId,
    affectedEmployments: employments.length
  });

  return { jobCatalog, affectedTemplates: templates.length, affectedEmployments: employments.length };
}
```

**Strategic megoldás: Ágazati munkakör-katalógusok importálása**

Jogszabályból automatikus JobCatalog generálás (OCR, strukturált adatforrás).

```javascript
async function importJobCatalogFromRegulation(regulationFile, sector) {
  // Példa: 150/1992. Korm. r. 1. sz. melléklet (CSV formátumban)
  const rows = await parseCSV(regulationFile);

  for (const row of rows) {
    await JobCatalog.create({
      code: `KJT-${sector.toUpperCase()}-${row.code}`,
      name: row.jobTitle,
      sector: sector,
      legalBasis: row.legalBasis,
      kjt_requiredPayGrade: row.payGrade,
      requiredEducationLevel: row.educationLevel,
      requiredEducationField: row.educationField,
      feorCode: row.feorCode,
      isActive: true,
      sourceSystem: 'IMPORT'
    });
  }
}
```

**Jogszabályi háttér:**
- **Kjt. 61. §:** A fizetési osztályt a munkakör betöltéséhez előírt iskolai végzettség határozza meg.
- **150/1992. (XI. 20.) Korm. rendelet:**
  - 1. sz. melléklet: Szociális és gyermekvédelmi területen foglalkoztatottak
  - 2. sz. melléklet: Kulturális munkakörök (könyvtár, levéltár, múzeum)
- **257/2000. (XII. 26.) Korm. rendelet:** Közgyűjteményi dolgozók munkakör-jegyzéke

**GDPR:**
Nem releváns (munkakör-jegyzék nem személyes adat).

**Implementációs prioritás:**
- **MVP:** QualificationRequirement szöveges leírásban + Position.kjt_requiredPayGrade manuális
- **Extended:** JobCatalog törzsadat + automatikus követelmény-generálás (ajánlott)
- **Strategic:** Jogszabály-importálás, automatikus frissítés

**Ajánlás:**
Extended modell (JobCatalog törzsadat) bevezetése akkor, ha:
1. Kjt. szerinti jogviszony (közalkalmazott)
2. Szociális, kulturális, közgyűjteményi ágazat (150/1992. Korm. r., 257/2000. Korm. r.)
3. Gyakori jogszabály-változások (munkakör-jegyzék frissítése)
4. Konzisztens besorolás biztosítása (fizetési osztály automatikusan a munkakörből)

---

### 12.2. Összefoglaló döntési mátrix

| Kérdés | MVP megoldás | Extended megoldás (ajánlott) | Strategic megoldás | Implementációs ajánlás |
|--------|--------------|------------------------------|---------------------|------------------------|
| **PositionTemplate szükségessége** | Kötelező PositionTemplate (1:1 kis szervezeteknél) | Opcionális PositionTemplate + inline mód | Automatikus sablon-javaslat és konverzió | Extended – rugalmas, skálázható kis és nagy szervezetek számára |
| **Munkaköri leírás tárolása** | `description` text mező | PositionDescription entitás strukturált mezőkkel + dokumentum-hivatkozás | Kompetencia-modell integrációja | Extended – ha teljesítménymenedzsment, kompetencia-alapú értékelés, közszféra |
| **Szervezeti ábra vs. beszámolási struktúra** | `reportingToPositionId` = szervezeti hierarchia | `reportingToPositionId` = funkcionális beszámolás + PositionReportingRelationship | Külön szervezeti ábra és beszámolási diagram | Extended – mátrix szervezet, kettős beszámolási viszony |
| **FEOR karbantartás** | FEOR kód string + manuális karbantartás | FEOR entitás + KSH szinkronizáció | Automatikus FEOR javaslat + gépi tanulás | Extended – validáció, NAV compliance, hierarchikus lekérdezések |
| **Pedagógus óraszám-keret** | Óraszám csak Employment szintjén | Position = keret (min-max), Employment = konkrét óraszám | Órarend-tervező integráció, óraszám-monitoring | Extended – köznevelés, óraterv-tervezés, validálható óraszám |
| **Álláshely-gazdálkodás és költségvetés** | Position csak szervezeti adat | Position bérsáv + PositionBudget entitás | Forgatókönyv-alapú költségvetés-tervezés | Extended – álláshely-alapú létszámgazdálkodás (Kit., Kttv.), költségvetés-tervezés |
| **Közalkalmazotti munkaköri bértábla** | QualificationRequirement szöveges + manuális kjt_requiredPayGrade | JobCatalog törzsadat + automatikus követelmény-generálás | Jogszabály-importálás, automatikus frissítés | Extended – Kjt. jogviszony, szociális/kulturális ágazat, jogszabály-változások |

**Általános implementációs ajánlás:**
- **MVP:** Kis szervezetek (< 50 fő), egyszerű szervezeti struktúra, versenyszféra
- **Extended:** Közepes-nagy szervezetek (50+ fő), közszféra (Kjt., Kit., Kttv., Púétv.), mátrix szervezet, költségvetési tervezés
- **Strategic:** Multinacionális vállalatok, komplex compliance követelmények, AI-alapú optimalizáció

**Jogszabályi megfelelés:**
Minden megoldás (MVP, Extended, Strategic) GDPR- és magyar jogszabály-kompatibilis, de az Extended megoldás jobban támogatja:
- Közszférás szervezetek speciális igényeit (álláshely-alapú létszámgazdálkodás, ágazati munkakör-jegyzékek)
- Audit követelményeket (munkaköri leírás verzionálás, költségvetés-eltérés nyomon követése)
- Jogviszony-típusok sokszínűségét (Mt., Kjt., Kit., Kttv., Púétv., Eszjtv.)

**Következő lépés:**
Az MVP és Extended megoldások implementálása, Strategic megoldások opcionálisan, ha konkrét üzleti igény merül fel.
