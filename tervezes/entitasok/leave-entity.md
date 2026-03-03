# Leave (Szabadság / Távollét) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer  
> **Verzió:** 0.1 – első tervezet  
> **Utolsó frissítés:** 2025. február  
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md, employment-entity.md, compensation-entity.md, organization-entity.md, position-entity.md, qualification-entity.md

---

## 1. Az entitás célja és hatóköre

A **Leave** entitáscsalád a foglalkoztatási jogviszonyhoz (Employment) kapcsolódó szabadságjogosultságokat, szabadságigényléseket, távolléti eseményeket és a hozzájuk tartozó üzleti szabályokat tartalmazza. A magyar munkajog az egyik legkomplexebb szabadságrendszert működteti Európában: az alapszabadság, a pótszabadságok, a betegszabadság, a különféle mentesülések és a fizetés nélküli szabadságok szabályai **jogviszony-típusonként eltérőek**, és szorosan összefonódnak az adó-, társadalombiztosítási és munkaidő-nyilvántartási szabályokkal.

**Miért kritikus a Leave entitás?**

| Felhasználási terület | Jogszabály | Hatás |
|---|---|---|
| **Alapszabadság jogosultság** | Mt. 116. §; Púétv. 88. §; Kjt.; Kit.; Kttv. | Jogviszony-típusonként eltérő mérték |
| **Pótszabadság** | Mt. 117–120. §; Kjt.; Púétv. | Életkor, gyermek, fogyatékosság, munkakör alapján |
| **Szabadság kiadása** | Mt. 122–125. § | Munkáltató adja ki; 7/12 a kiadás szabálya; 15 nap munkavállalói döntés |
| **Betegszabadság** | Mt. 126. § | 15 munkanap/év; 70%-os távolléti díj |
| **Szülési szabadság** | Mt. 127. § | 24 hét (168 nap) |
| **Apasági szabadság** | Mt. 118/A. § (2025-ös módosítás) | 10 munkanap + 4 hónap szülői szabadság |
| **Szülői szabadság** | Mt. 118/B. § | 44 munkanap; távolléti díj 10%-a (ha nem CSED/GYED-ben) |
| **Fizetés nélküli szabadság** | Mt. 128–133. § | Gyermek gondozása (GYES/GYED), hozzátartozó ápolása, tényleges önkéntes katonai szolgálat |
| **Keresőképtelenség (táppénz)** | Tbj.; 1997. évi LXXXIII. tv. | Társadalombiztosítás terhére; NEAK felé elszámolás |
| **Munkaidő-nyilvántartás** | Mt. 134. § | A szabadság és távollét a nyilvántartás része |
| **Bérszámfejtés** | Mt.; Szja tv.; Tbj. | Távolléti díj vs. betegszabadság díjazás vs. TB-ellátás |

**Tervezési alapelvek:**

- **Jogosultság és felhasználás szétválasztása:** A **LeaveEntitlement** (jogosultság) és a **LeaveRequest** / **LeaveRecord** (igénylés / tényleges felhasználás) két különálló entitás. A jogosultság az éves keretet határozza meg, a felhasználás az egyes napokra vonatkozik.
- **Jogviszony-típus vezérelt:** Az alapszabadság mértéke, a pótszabadságok köre és a speciális távolléti jogcímek jogviszony-típusonként eltérőek.
- **Automatikus számítás:** A jogosultságot a rendszer a lehető legnagyobb mértékben automatikusan számolja az Employment, Person és Qualification entitások adataiból.
- **Employment-hez kötött:** A szabadság a jogviszonyhoz tartozik, nem a személyhez. Többes jogviszony esetén minden Employment-hez önálló jogosultság tartozik.
- **Naptári éves ciklus:** A szabadságjogosultság naptári évre szól; év közbeni belépésnél/kilépésnél időarányos.

---

## 2. Entitásstruktúra – áttekintés

```
Employment 1 ──── N LeaveEntitlement          (éves jogosultságok típusonként)
Employment 1 ──── N LeaveRequest              (szabadságigénylések)
Employment 1 ──── N LeaveRecord               (tényleges távolléti napok)
Employment 1 ──── N AbsenceEvent              (egyéb távolléti események: keresőképtelenség, szülési szabadság stb.)

LeaveRequest 1 ──── N LeaveRecord             (egy igénylés több napra vonatkozhat)
AbsenceEvent 1 ──── N LeaveRecord             (egy esemény több napra vonatkozhat)

LeaveEntitlement ──── LeaveType               (szabadság/távollét típus törzsadat)
LeaveRequest ──── LeaveType
AbsenceEvent ──── AbsenceType                 (távolléti esemény típus törzsadat)
```

---

## 3. LeaveType (Szabadság / Távollét típus – törzsadat)

### 3.1. Property-k

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `id` | UUID | igen | PK |
| `code` | string | igen | Egyedi kód |
| `name` | string | igen | Megnevezés |
| `category` | enum | igen | Kategória (lásd 3.2.) |
| `applicableEmploymentTypes` | array[enum] | igen | Mely jogviszony-típusoknál alkalmazható |
| `isPaid` | boolean | igen | Fizetett szabadság-e |
| `payRate` | decimal | nem | Díjazás mértéke (1.0 = 100% távolléti díj; 0.7 = 70%; 0.0 = fizetetlen) |
| `payBase` | enum | nem | Díjazás alapja: `ABSENCE_PAY` (távolléti díj), `BASE_SALARY` (alapbér), `SICK_PAY` (betegszabadság díja), `SOCIAL_INSURANCE` (TB ellátás), `NONE` |
| `deductsFromEntitlement` | boolean | igen | Levon-e az éves szabadságkeretből |
| `requiresApproval` | boolean | igen | Jóváhagyás szükséges-e |
| `requiresDocumentation` | boolean | igen | Dokumentum szükséges-e (pl. orvosi igazolás, születési anyakönyvi kivonat) |
| `maxDaysPerYear` | integer | nem | Éves maximum (ha van) |
| `maxDaysPerOccurrence` | integer | nem | Alkalomankénti maximum (ha van) |
| `isCalendarDay` | boolean | igen | Naptári napban vagy munkanapban számítandó |
| `affectsProtection` | boolean | nem | Felmondási/felmentési védelmet keletkeztet-e |
| `legalBasis` | string | nem | Jogszabályi hivatkozás |
| `isActive` | boolean | igen | Aktív törzsadat-e |

### 3.2. Kategóriák

| Kód | Megnevezés | Leírás |
|---|---|---|
| `ANNUAL` | Rendes szabadság (alap + pót) | Éves jogosultság, munkáltató adja ki |
| `SUPPLEMENTARY` | Pótszabadság | Az alapszabadság feletti jogosultság (gyermek, fogyatékosság stb.) |
| `SICK` | Betegszabadság | Saját keresőképtelenség miatti, munkáltató terhére |
| `MATERNITY` | Szülési szabadság | Anyai szülési szabadság (24 hét) |
| `PATERNITY` | Apasági szabadság | Apa részére (10 munkanap + szülői) |
| `PARENTAL` | Szülői szabadság | Mindkét szülő részére (44 munkanap) |
| `UNPAID` | Fizetés nélküli szabadság | Gyermek gondozása, hozzátartozó ápolása stb. |
| `STUDY` | Tanulmányi szabadság / munkaidő-kedvezmény | Vizsgára felkészülés, vizsgán megjelenés |
| `SPECIAL_PAID` | Egyéb fizetett mentesülés | Véradás, hatósági megjelenés, közeli hozzátartozó halála |
| `ABSENCE` | Igazolt távollét (nem szabadság) | Keresőképtelenség (táppénz), sztrájk, karantén |
| `UNAUTHORIZED` | Igazolatlan távollét | Munkavégzési kötelezettség megszegése |

---

## 4. LeaveEntitlement (Szabadságjogosultság)

### 4.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `employmentId` | UUID (FK) | igen | Hivatkozás az Employment-re | Technikai |
| `leaveTypeId` | UUID (FK) | igen | Szabadság típusa | Technikai |
| `year` | integer | igen | Naptári év | – |
| `baseEntitlement` | decimal | igen | Alapjogosultság (munkanapban) | Mt. 116. §; Púétv. stb. |
| `supplementaryEntitlement` | decimal | igen | Pótszabadság jogosultság (munkanapban) | Mt. 117–120. §; Púétv. stb. |
| `totalEntitlement` | decimal | számított | Összes jogosultság (`baseEntitlement` + `supplementaryEntitlement`) | – |
| `proRataEntitlement` | decimal | számított | Időarányos jogosultság (év közbeni belépésnél/kilépésnél) | Mt. 121. § |
| `carryOverFromPrevYear` | decimal | nem | Előző évről áthozott napok | Mt. 125. § – tárgyévet követő március 31-ig kiadandó |
| `adjustment` | decimal | nem | Kézi korrekció (+/–) – indoklással | – |
| `adjustmentReason` | string | nem | Korrekció indoklása | – |
| `totalAvailable` | decimal | számított | Felhasználható összesen (`proRataEntitlement` + `carryOverFromPrevYear` + `adjustment`) | – |
| `used` | decimal | számított | Felhasznált napok (jóváhagyott LeaveRecord-okból) | – |
| `planned` | decimal | számított | Tervezett, de még nem felhasznált (jóváhagyott, jövőbeni LeaveRequest-ekből) | – |
| `remaining` | decimal | számított | Fennmaradó (`totalAvailable` – `used` – `planned`) | – |
| `calculationDetails` | JSON | nem | Számítás részletei (milyen alapon, milyen pótszabadságok) | – |
| `status` | enum | igen | `CALCULATED`, `ADJUSTED`, `FINALIZED` | – |
| `createdAt` | datetime | igen | Létrehozás | – |
| `createdBy` | string | igen | Létrehozó | – |
| `updatedAt` | datetime | igen | Módosítás | – |
| `updatedBy` | string | igen | Módosító | – |

---

## 5. Szabadságjogosultság számítási szabályok

### 5.1. Mt. – Munkaviszony (versenyszféra)

#### 5.1.1. Alapszabadság (Mt. 116. §)

20 munkanap/év. Az életkor alapján növekszik:

| Életkor (betöltött) | Alapszabadság (munkanap) | Többlet az előzőhöz |
|---|---|---|
| < 25 | 20 | – |
| 25 | 21 | +1 |
| 28 | 22 | +1 |
| 31 | 23 | +1 |
| 33 | 24 | +1 |
| 35 | 25 | +1 |
| 37 | 26 | +1 |
| 39 | 27 | +1 |
| 41 | 28 | +1 |
| 43 | 29 | +1 |
| 45+ | 30 | +1 |

**Számítási szabály:** A magasabb mértékű szabadság abban az évben jár először, amelyben a munkavállaló az adott életkort betölti. Forrás: `Person.birthDate`.

#### 5.1.2. Pótszabadság – gyermek után (Mt. 118. §)

| Feltétel | Pótszabadság (munkanap) |
|---|---|
| 1 gyermek (16 éves korig) | 2 |
| 2 gyermek | 4 |
| 2-nél több gyermek | összesen 7 |
| Fogyatékos gyermek | gyermekenként +2 (a fenti felett) |

**Számítási szabály:** A `Person.dependents` adatból, figyelembe véve a gyermek életkorát és fogyatékossági státuszát. A pótszabadság abban az évben jár utoljára, amelyben a gyermek a 16. életévét betölti.

#### 5.1.3. Egyéb pótszabadságok (Mt. 119–120. §)

| Jogcím | Mérték | Feltétel | Forrás |
|---|---|---|---|
| Fiatal munkavállaló (Mt. 119. §) | 5 munkanap/év | 18 éven aluli | Person.birthDate |
| Fogyatékos munkavállaló (Mt. 120. §) | 5 munkanap/év | Megváltozott munkaképesség / fogyatékosság | Person.disabilityStatus |
| Vak munkavállaló (Mt. 120. §) | 5 munkanap/év | Látássérültség | Person.disabilityStatus |
| Föld alatti munkahely / ionizáló sugárzás (Mt. 120. §) | 5 munkanap/év | Munkakörülmény | Position.specialWorkConditions |

#### 5.1.4. Betegszabadság (Mt. 126. §)

| Jellemző | Érték |
|---|---|
| Éves keret | 15 munkanap/év |
| Díjazás | Távolléti díj 70%-a |
| Teher | Munkáltató |
| Dokumentum | Keresőképtelenséget igazoló orvosi dokumentum |
| Számítás | Év közbeni belépésnél NEM időarányos (de csak a jogviszony fennállása alatt vehető igénybe) |

---

### 5.2. Kjt. – Közalkalmazotti jogviszony

| Jellemző | Eltérés az Mt.-hez képest | Jogszabály |
|---|---|---|
| Alapszabadság | Megegyezik az Mt.-vel (20 nap + életkor) | Kjt. → Mt. 116. § |
| Pótszabadság – gyermek | Megegyezik az Mt.-vel | Kjt. → Mt. 118. § |
| Pótszabadság – fizetési osztály | D–J fizetési osztályban: 1–11 nap pótszabadság (lásd alább) | Kjt. 57. § (3) |
| Betegszabadság | Megegyezik az Mt.-vel (15 nap, 70%) | Kjt. → Mt. 126. § |

**Kjt. 57. § (3) – fizetési osztály szerinti pótszabadság:**

| Fizetési osztály | Pótszabadság (munkanap) |
|---|---|
| A, B, C | 0 |
| D | 1 |
| E | 3 |
| F | 5 |
| G | 7 |
| H | 8 |
| I | 9 |
| J | 11 |

**Számítási szabály:** Az `Employment.kjt_payGrade` alapján automatikusan kalkulálható.

---

### 5.3. Púétv. – Köznevelési foglalkoztatotti jogviszony

| Jellemző | Érték | Jogszabály |
|---|---|---|
| **Alapszabadság – pedagógus** | **50 munkanap/év** (nem függ az életkortól!) | Púétv. 88. § (4) |
| Alapszabadság – NOKS | Az Mt. szerinti (20 + életkor) + a Púétv. szerinti pótszabadság | Púétv. |
| Pótszabadság – gyermek | Mt. szerinti (2/4/7 nap) | Púétv. → Mt. |
| Kiadás sajátossága | A szabadságot elsősorban a nyári szünetben kell kiadni (pedagógus) | Púétv. 88. § (5) |
| Betegszabadság | Mt. szerint (15 nap, 70%) | Púétv. → Mt. |

**Megjegyzések:**
- A pedagógus **50 napos** alapszabadság az egyik legmagasabb Európában. Ez nem bontható alap + pótszabadságra: 50 nap a kiindulás, amelyhez az Mt. szerinti pótszabadságok (gyermek, fogyatékosság) **hozzáadódnak**.
- A NOKS (nevelő-oktató munkát közvetlenül segítő) dolgozók szabadságjogosultsága eltér a pedagógusétól és az Mt. szabályai alapján számítandó, kiegészítve a Púétv. speciális pótszabadságaival.

---

### 5.4. Eszjtv. – Egészségügyi szolgálati jogviszony

| Jellemző | Érték | Jogszabály |
|---|---|---|
| Alapszabadság | Mt. szerinti (20 + életkor) | Eszjtv. → Mt. 116. § |
| Pótszabadság – gyermek | Mt. szerinti | Eszjtv. → Mt. 118. § |
| Pótszabadság – egyéb | Eszjtv. speciális: ügyeletet ellátók, fertőzésveszélyes munkakörök | Eszjtv.; 528/2020. Korm. r. |
| Betegszabadság | Mt. szerint (15 nap, 70%) | Eszjtv. → Mt. |

---

### 5.5. Kit./Kttv./Küt. – Kormányzati és közszolgálat

| Jellemző | Kit. | Kttv. | Jogszabály |
|---|---|---|---|
| Alapszabadság | 25 munkanap (nem életkorfüggő!) | Mt. szerinti (20 + életkor) | Kit. 110. §; Kttv. 99. § |
| Pótszabadság – besorolás | Besorolási kategória alapján | Besorolási kategória alapján (1–11 nap) | Kit. 110. §; Kttv. 99. § |
| Pótszabadság – gyermek | Mt. szerinti | Mt. szerinti | Kit.; Kttv. → Mt. |
| Pótszabadság – vezető | Vezetői szinttől függően 3–10 nap | Hasonló | Kit.; Kttv. |
| Betegszabadság | Mt. szerint | Mt. szerint | Kit.; Kttv. → Mt. |

**Kit. 110. § – alapszabadság 25 munkanap**, amely nem az életkortól, hanem a besorolási kategóriától függően bővül pótszabadságokkal.

---

## 6. Speciális távolléti jogcímek (minden jogviszony-típusnál)

### 6.1. Szülési szabadság (Mt. 127. §)

| Property | Érték |
|---|---|
| Időtartam | 24 hét (168 nap) |
| Számítás | Naptári napban |
| Kezdete | Legkorábban a szülés várható időpontja előtt 4 héttel; ha korábban veszi igénybe, akkor a szülés napjáig |
| Díjazás | CSED (csecsemőgondozási díj) – TB ellátás, nem a munkáltató terhére |
| Felmondási védelem | Igen (Mt. 65. § (3) b)) |
| Forrás | AbsenceEvent (MATERNITY típus) |

### 6.2. Apasági szabadság (Mt. 118/A. § – 2025-ös módosítás)

| Property | Érték |
|---|---|
| Időtartam | 10 munkanap |
| Díjazás | Első 5 nap: távolléti díj 100%-a; második 5 nap: távolléti díj 40%-a |
| Igénybevétel határideje | A születéstől (vagy az örökbefogadási eljárás jogerőssé válásától) számított 2 hónapon belül |
| Dokumentum | Születési anyakönyvi kivonat |
| Forrás | LeaveRequest (PATERNITY típus) |

### 6.3. Szülői szabadság (Mt. 118/B. §)

| Property | Érték |
|---|---|
| Időtartam | 44 munkanap |
| Feltétel | Gyermek 3. életévéig; mindkét szülő jogosult (nem átruházható) |
| Díjazás | Távolléti díj 10%-a; HA a szülő CSED-ben vagy GYED-ben van, akkor az ellátás összege, és a munkáltatónak nem kell díjazást fizetnie |
| Forrás | LeaveRequest (PARENTAL típus) |

### 6.4. Fizetés nélküli szabadságok (Mt. 128–133. §)

| Jogcím | Feltétel | Időtartam | Felmondási védelem | Jogszabály |
|---|---|---|---|---|
| Gyermek gondozása (GYES) | 3 éves korig (ikergyermeknél: tanköteles korig) | A jogosultság idejéig | Igen | Mt. 128. § |
| Gyermek betegsége | 12 éves korig, tartós betegség | Az ápolás idejéig | Igen | Mt. 131. § |
| Hozzátartozó ápolása | Tartós (3 hónapot meghaladó) ápolás | Az ápolás idejéig | Igen | Mt. 131. § |
| Önkéntes katonai szolgálat | Tényleges szolgálat | A szolgálat idejéig | Igen | Mt. 132. § |
| Tanulmányi célú (nem kötelező) | Munkáltató hozzájárulásával | Megegyezés | Nem | Mt. 133. § |

### 6.5. Mentesülések a munkavégzés alól (Mt. 55. §)

| Jogcím | Időtartam | Díjazás | Jogszabály |
|---|---|---|---|
| Állampolgári kötelezettség teljesítése | Szükséges idő | Távolléti díj (ha jogszabály előírja) | Mt. 55. § (1) a) |
| Közeli hozzátartozó halála | 2 munkanap | Távolléti díj | Mt. 55. § (1) b) |
| Általános iskolai tanulmányok | Szükséges idő | Távolléti díj | Mt. 55. § (1) c) |
| Kötelező orvosi vizsgálat | Szükséges idő | Távolléti díj | Mt. 55. § (1) d) |
| Véradás | A véradás napja | Távolléti díj | Mt. 55. § (1) e) |
| Szoptató anya munkaidő-kedvezménye | Napi 2× 1 óra (6 hónapig); napi 1 óra (9 hónapig) | Távolléti díj | Mt. 55. § (1) f) |
| Bíróság / hatóság előtti megjelenés | Szükséges idő | Távolléti díj (ha tanúként idézik) | Mt. 55. § (1) g) |
| Választott szakszervezeti tisztségviselő | Munkaidő-kedvezmény | Távolléti díj | Mt. 274. § (1) |

---

## 7. AbsenceEvent (Távolléti esemény)

Az **AbsenceEvent** a szabadság-jogosultságból nem levonódó, de a munkaidő-nyilvántartásban szereplő távolléti eseményeket kezeli.

### 7.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `employmentId` | UUID (FK) | igen | Hivatkozás az Employment-re | Technikai |
| `absenceType` | enum | igen | Távollét típusa (lásd 7.2.) | – |
| `startDate` | date | igen | Kezdő dátum | – |
| `endDate` | date | igen | Záró dátum | – |
| `startType` | enum | nem | Kezdés módja: `FULL_DAY`, `MORNING`, `AFTERNOON` | Résznapi kezelés |
| `endType` | enum | nem | Befejezés módja: `FULL_DAY`, `MORNING`, `AFTERNOON` | Résznapi kezelés |
| `workDaysCount` | decimal | számított | Érintett munkanapok száma | – |
| `calendarDaysCount` | integer | számított | Érintett naptári napok száma | – |
| `paymentResponsibility` | enum | igen | Díjazási teher: `EMPLOYER` (munkáltató), `SOCIAL_INSURANCE` (TB), `NONE` (nincs díjazás) | – |
| `payRate` | decimal | nem | Díjazás mértéke (0.0–1.0) | – |
| `documentType` | string | nem | Szükséges dokumentum típusa (pl. „keresőképtelenségi igazolás") | – |
| `documentReceived` | boolean | nem | Dokumentum beérkezett-e | – |
| `documentReceivedDate` | date | nem | Dokumentum beérkezésének dátuma | – |
| `relatedPersonName` | string | nem | Érintett személy neve (pl. elhunyt hozzátartozó, beteg gyermek) | – |
| `relatedPersonRelationship` | enum | nem | Kapcsolat típusa | – |
| `notes` | string | nem | Megjegyzés | – |
| `status` | enum | igen | `REPORTED`, `APPROVED`, `ACTIVE`, `COMPLETED`, `CANCELLED` | – |
| `createdAt` | datetime | igen | Létrehozás | – |
| `createdBy` | string | igen | Létrehozó | – |
| `updatedAt` | datetime | igen | Módosítás | – |
| `updatedBy` | string | igen | Módosító | – |

### 7.2. Távolléti esemény típusok (absenceType)

| Kód | Megnevezés | Díjazás | Teher | Jogszabály |
|---|---|---|---|---|
| `SICK_LEAVE` | Betegszabadság | 70% távolléti díj | Munkáltató | Mt. 126. § |
| `SICK_PAY` | Keresőképtelenség – táppénz | Táppénz (60–70% napi átlagjövedelem) | NEAK (TB) | 1997. évi LXXXIII. tv. |
| `ACCIDENT_SICK_PAY` | Baleseti táppénz | 100% napi átlagjövedelem | NEAK (TB) | 1997. évi LXXXIII. tv. |
| `MATERNITY_LEAVE` | Szülési szabadság | CSED (70% napi átlagjövedelem) | NEAK (TB) | Mt. 127. §; 1997. évi LXXXIII. tv. |
| `CHILD_CARE_FEE` | GYED (gyermekgondozási díj) | 70% napi átlagjövedelem (max korlát) | NEAK (TB) | 1997. évi LXXXIII. tv. |
| `CHILD_HOME_CARE_ALLOWANCE` | GYES (gyermekgondozást segítő ellátás) | Fix összeg (öregségi nyugdíjminimum) | Magyar Államkincstár | 1998. évi LXXXIV. tv. |
| `CHILD_NURSING` | Gyermek ápolása miatti keresőképtelenség | Táppénz | NEAK (TB) | 1997. évi LXXXIII. tv. |
| `RELATIVE_NURSING` | Hozzátartozó ápolása | Fizetés nélküli szabadság | Nincs | Mt. 131. § |
| `QUARANTINE` | Hatósági karantén / járványügyi intézkedés | Táppénz | NEAK (TB) | 1997. évi LXXXIII. tv. |
| `MILITARY_SERVICE` | Önkéntes tartalékos katonai szolgálat | Fizetés nélküli szabadság | MH (honvédség) | Mt. 132. § |
| `STRIKE` | Sztrájk | Nincs díjazás | – | 1989. évi VII. tv. |
| `SUSPENSION` | Munkavégzés alóli felfüggesztés (fegyelmi) | Jogviszony-függő (Mt.: nincs; közszféra: alapilletmény) | Munkáltató | Mt.; Kit.; Kttv. |
| `EMPLOYER_IDLE` | Állásidő (munkáltató működési körében felmerülő ok) | Alapbér 100% | Munkáltató | Mt. 146. § (1) |
| `UNAUTHORIZED_ABSENCE` | Igazolatlan távollét | Nincs díjazás; jogkövetkezmény alapja | – | Mt. |

---

## 8. LeaveRequest (Szabadságigénylés)

### 8.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `employmentId` | UUID (FK) | igen | Hivatkozás az Employment-re | Technikai |
| `leaveTypeId` | UUID (FK) | igen | Szabadság típusa | Technikai |
| `leaveEntitlementId` | UUID (FK) | nem | Hivatkozás az érintett jogosultságra | Technikai |
| `startDate` | date | igen | Kezdő dátum | – |
| `endDate` | date | igen | Záró dátum | – |
| `startType` | enum | nem | `FULL_DAY`, `MORNING`, `AFTERNOON` | Résznapi |
| `endType` | enum | nem | `FULL_DAY`, `MORNING`, `AFTERNOON` | Résznapi |
| `workDaysRequested` | decimal | igen | Igényelt munkanapok száma (számított / felülírható fél napnál) | – |
| `reason` | string | nem | Indoklás (a munkavállalónak nem kell indokolnia az Mt. 122. § szerinti 15 napot) | – |
| `isEmployeeInitiated` | boolean | igen | Munkavállaló kezdeményezte-e (15 napos keret) | Mt. 122. § (2) |
| `requestedAt` | datetime | igen | Igénylés időpontja | – |
| `requestedBy` | string | igen | Igénylő (személy vagy a munkáltató nevében) | – |
| `status` | enum | igen | Státusz (lásd 8.2.) | – |
| `approvedBy` | string | nem | Jóváhagyó | – |
| `approvedAt` | datetime | nem | Jóváhagyás időpontja | – |
| `rejectionReason` | string | nem | Elutasítás indoka | – |
| `cancelledAt` | datetime | nem | Visszavonás/törlés időpontja | – |
| `cancelledBy` | string | nem | Visszavonó | – |
| `cancellationReason` | string | nem | Visszavonás indoka | – |
| `createdAt` | datetime | igen | Létrehozás | – |
| `createdBy` | string | igen | Létrehozó | – |
| `updatedAt` | datetime | igen | Módosítás | – |
| `updatedBy` | string | igen | Módosító | – |

### 8.2. Igénylés státuszok

| Kód | Megnevezés | Leírás |
|---|---|---|
| `DRAFT` | Piszkozat | Munkavállaló elmentette, de nem nyújtotta be |
| `SUBMITTED` | Benyújtva | Jóváhagyásra vár |
| `APPROVED` | Jóváhagyva | Vezető/HR elfogadta |
| `REJECTED` | Elutasítva | Vezető/HR elutasította (indoklással) |
| `CANCELLED_BY_EMPLOYEE` | Munkavállaló által visszavonva | A munkavállaló visszavonta az igénylést |
| `CANCELLED_BY_EMPLOYER` | Munkáltató által visszavonva | Munkáltató módosította a szabadság kiadását (Mt. 123. § – gazdaságilag kiemelt érdek) |
| `TAKEN` | Felhasználva | A szabadságot ténylegesen igénybe vette |

---

## 9. LeaveRecord (Tényleges távolléti nap)

Az egyedi napszintű nyilvántartás, amely a munkaidő-nyilvántartás (TimeTracking) alapja.

### 9.1. Property-k

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `id` | UUID | igen | PK |
| `employmentId` | UUID (FK) | igen | Hivatkozás az Employment-re |
| `leaveRequestId` | UUID (FK) | nem | Hivatkozás a szabadságigénylésre (ha szabadságból ered) |
| `absenceEventId` | UUID (FK) | nem | Hivatkozás a távolléti eseményre (ha táppénzből, szülési szabadságból stb. ered) |
| `date` | date | igen | Az adott nap |
| `leaveTypeId` | UUID (FK) | igen | Szabadság/távollét típusa |
| `dayFraction` | decimal | igen | Nap töredéke (1.0 = egész nap, 0.5 = fél nap) |
| `paymentType` | enum | igen | Díjazás típusa: `ABSENCE_PAY`, `SICK_PAY_70`, `SOCIAL_INSURANCE`, `UNPAID`, `EMPLOYER_IDLE` |
| `isWorkDay` | boolean | igen | Az adott nap munkanap-e a dolgozó munkarendje szerint |
| `status` | enum | igen | `PLANNED`, `ACTUAL`, `CANCELLED` |
| `createdAt` | datetime | igen | Létrehozás |
| `createdBy` | string | igen | Létrehozó |

---

## 10. Szabadság kiadásának szabályai

### 10.1. Mt. szabályok (minden jogviszony-típusnál háttérjogszabály)

| Szabály | Leírás | Jogszabály |
|---|---|---|
| Kiadás kötelezettsége | A szabadságot a tárgyévben kell kiadni | Mt. 122. § (1) |
| Munkáltatói kiadás | A szabadságot a munkáltató adja ki | Mt. 122. § (1) |
| Munkavállalói rendelkezés | Az alapszabadság egynegyedét a munkavállaló kérésének megfelelő időpontban kell kiadni (15 napos közlési kötelezettség) | Mt. 122. § (2) |
| Összefüggő kiadás | Legalább 14 egybefüggő naptári napot kell biztosítani évente | Mt. 122. § (3) |
| Nyilatkozat hiánya | Ha a munkavállaló a tárgyévben nem kéri a 15 napot, nem veszíti el → átvihető | Mt. 122. § (2) |
| Átvitel | A ki nem adott szabadságot a következő év március 31-ig kell kiadni | Mt. 125. § |
| Visszahívás | Rendkívüli, indokolt esetben a munkáltató a megkezdett szabadságot megszakíthatja | Mt. 123. § |
| Elhalasztás | A munkáltató gazdaságilag kiemelt érdekből a kiadás időpontját módosíthatja | Mt. 123. § |
| Pénzbeli megváltás | A szabadságot **nem lehet** pénzben megváltani, **kivéve** a jogviszony megszűnésekor (ki nem adott szabadság) | Mt. 125. § (2) |
| Jogviszony megszűnéskori elszámolás | Ki nem adott → pénzben megváltandó (távolléti díj); túl sok kiadott → nem követelhető vissza | Mt. 125. § |

### 10.2. Speciális szabályok jogviszony-típusonként

| Jogviszony | Speciális szabály | Jogszabály |
|---|---|---|
| Púétv. – pedagógus | Szabadságot **elsősorban a nyári szünetben** kell kiadni; a szorgalmi időben főszabály szerint nem | Púétv. 88. § (5) |
| Kjt. – közalkalmazott | Közalkalmazotti tanács véleményezési joga a szabadság kiadási rendjéről | Kjt. |
| Kit. – kormányzati | Szabadságterv készítése kötelező (éves) | Kit. 113. § |
| Kttv. – közszolgálati | Szabadságterv készítése kötelező | Kttv. 101. § |

---

## 11. Üzleti szabályok és validációk

| Szabály | Leírás | Jogszabály |
|---|---|---|
| Jogosultság-túllépés tilalom | Nem igényelhető/hagyható jóvá több szabadság, mint amennyi `remaining` ≥ 0 marad | Logikai szabály |
| 14 napos összefüggő szabadság ellenőrzés | A rendszer figyelmeztet, ha az évben még nem volt legalább 14 naptári napos összefüggő szabadság | Mt. 122. § (3) |
| Tárgyévet követő márc. 31. | Figyelmeztetés, ha a tárgyévi maradék szabadság márc. 31-ig nem kerül kiadásra | Mt. 125. § |
| Betegszabadság keret | Maximum 15 munkanap/év; ezen felül keresőképtelenség → táppénz (TB) | Mt. 126. § |
| Szülési szabadság automatikus kalkuláció | A szülés várható időpontjából vagy tényleges szülés dátumából 24 hét automatikus számítás | Mt. 127. § |
| Apasági szabadság határidő | Születéstől számított 2 hónapon belül igénybe veendő | Mt. 118/A. § |
| Szülői szabadság életkor-feltétel | Gyermek 3. életévéig igényelhető | Mt. 118/B. § |
| Felmondási védelmi figyelmeztetés | Ha a munkavállaló szülési szabadságon, GYES/GYED-en, apasági szabadságon van → védett | Mt. 65. § (3) |
| Részmunkaidős arányosítás | A szabadság munkanapban jár, de a részmunkaidős dolgozó is teljes napot kap (a nap rövidebb) | Mt. 124. § |
| Pedagógus nyári kiadás | Figyelmeztetés, ha pedagógus szabadsága szorgalmi időben kerülne kiadásra | Púétv. 88. § (5) |
| Igazolatlan távollét riasztás | Azonnali értesítés a HR-nek és közvetlen vezetőnek | Belső szabályzat |

---

## 12. Automatikus események és figyelmeztetések

| Esemény | Kiváltó feltétel | Akció |
|---|---|---|
| **Éves jogosultság kalkuláció** | Naptári év kezdete (jan. 1.) | Minden aktív Employment-re LeaveEntitlement generálás |
| **Életkor-alapú jogosultság változás** | Person.birthDate → betöltött 25/28/31/33/35/37/39/41/43/45 | LeaveEntitlement.baseEntitlement frissítés |
| **Gyermek 16. életévének betöltése** | Person.dependents[].birthDate | Pótszabadság jogosultság csökkenés a következő évtől |
| **Betegszabadság keret kimerülés** | 15 munkanap felhasznált | Figyelmeztetés: a következő keresőképtelenségi nap táppénz (TB) |
| **Betegszabadság → táppénz átfordulás** | Betegszabadság keret elfogyott, keresőképtelenség folytatódik | AbsenceEvent típus-váltás: SICK_LEAVE → SICK_PAY |
| **Szabadság-maradék figyelmeztetés** | Okt. 31.: ha >5 nap maradék; Dec. 15.: ha >0 nap maradék | Figyelmeztetés munkavállalónak és vezetőnek |
| **Átvitel figyelmeztetés** | Tárgyévet követő febr. 15.: ha van átvitt, még ki nem adott | Figyelmeztetés: márc. 31-ig kiadandó |
| **14 napos összefüggő szabadság** | Szept. 30.: ha az évben nem volt 14 napos összefüggő | Figyelmeztetés |
| **Szülési szabadság lejárat** | MATERNITY_LEAVE 24 hét vége közelít (4 héttel előtte) | Figyelmeztetés; GYED/GYES igény felmérés |
| **Apasági szabadság határidő** | Születéstől 6 hét eltelt, igénylés még nem történt | Figyelmeztetés |
| **GYES/GYED lejárat** | Gyermek 2./3. életéve közelít | Figyelmeztetés; visszatérési folyamat indítása |
| **Jogviszony megszűnés – szabadság elszámolás** | Employment terminationDate beállítva | Fennmaradó szabadság pénzbeli megváltás kalkuláció |

---

## 13. Ünnepnapok és munkanap-naptár

### 13.1. Magyarországi munkaszüneti napok (Mt. 102. § (1))

| Dátum | Megnevezés |
|---|---|
| Január 1. | Újév |
| Március 15. | Nemzeti ünnep |
| Nagypéntek | Húsvét előtti péntek (mozgó) |
| Húsvéthétfő | Húsvét utáni hétfő (mozgó) |
| Május 1. | A munka ünnepe |
| Pünkösdhétfő | (mozgó) |
| Augusztus 20. | Az államalapítás ünnepe |
| Október 23. | Az 1956-os forradalom ünnepe |
| November 1. | Mindenszentek |
| December 25. | Karácsony első napja |
| December 26. | Karácsony második napja |

### 13.2. Munkanap-áthelyezés (szombati munkanap)

A magyar kormány évente rendeletben (Korm. rendelet) határozza meg a „szombati munkanapokat" – amikor egy köztes hétköznap szabaddá válik, de cserébe egy szombat munkanappá. A rendszernek ezt **éves szinten karbantartható naptárként** kell kezelnie.

### 13.3. WorkCalendar entitás (technikai)

| Property | Típus | Leírás |
|---|---|---|
| `id` | UUID | PK |
| `date` | date | Dátum |
| `dayType` | enum | `WORKDAY`, `WEEKEND`, `PUBLIC_HOLIDAY`, `TRANSFERRED_WORKDAY` (szombati munkanap), `TRANSFERRED_REST_DAY` (áthelyezés miatti szabadnap) |
| `year` | integer | Naptári év |
| `description` | string | Leírás (ünnep neve, rendelet száma) |

---

## 14. GDPR adatkezelési kategorizáció

| Kategória | Property-k | Jogalap | Megőrzési idő |
|---|---|---|---|
| **Szabadságjogosultság** | LeaveEntitlement (összes) | Jogi kötelezettség (6(1)c) – Mt. 134. § | Jogviszony megszűnése + 3 év (elévülés) |
| **Szabadságigénylés** | LeaveRequest (összes) | Jogi kötelezettség (6(1)c) + Szerződés (6(1)b) | Jogviszony megszűnése + 3 év |
| **Távolléti nap** | LeaveRecord (összes) | Jogi kötelezettség (6(1)c) – munkaidő-nyilvántartás | Jogviszony megszűnése + 3 év |
| **Keresőképtelenség** | AbsenceEvent (SICK_*, ACCIDENT_*) | Jogi kötelezettség (6(1)c); **különleges adat (GDPR 9. cikk) – egészségügyi adat** | Jogviszony megszűnése + 5 év (TB elszámolás) |
| **Szülési / szülői / apasági szabadság** | AbsenceEvent (MATERNITY_*, CHILD_*) | Jogi kötelezettség (6(1)c) – TB; **érzékeny adat** (családi helyzet) | Jogviszony megszűnése + 5 év |
| **Igazolatlan távollét** | AbsenceEvent (UNAUTHORIZED) | Jogos érdek (6(1)f) – fegyelmi alapja | Jogviszony megszűnése + 3 év (munkaügyi per) |
| **Jóváhagyási adatok** | approvedBy, approvedAt, rejectionReason | Elszámoltathatóság (GDPR 5(2)) | Az érintett szabadság megőrzési idejével megegyező |

**Megjegyzések:**
- A **keresőképtelenség oka** (diagnózis) **különleges adat** (GDPR 9. cikk – egészségügyi adat). A munkáltató csak a keresőképtelenség tényét és időtartamát ismerheti, a diagnózist **NEM**. Az HRMS-ben a diagnózis-kód **nem tárolható** – csak a távollét típusa (betegszabadság / táppénz / baleseti táppénz).
- A szülési szabadság, GYED, GYES igénybevétele családi körülményekre utal → az adatkezelésnél a szükségesség és célhoz kötöttség elve fokozottan érvényesül.

---

## 15. Hozzáférési szintek

| Szerep | LeaveEntitlement | LeaveRequest | AbsenceEvent | LeaveRecord |
|---|---|---|---|---|
| **Érintett munkavállaló** | Saját: olvasás | Saját: igénylés, visszavonás | Saját: olvasás | Saját: olvasás |
| **Közvetlen vezető** | Beosztottak: olvasás (összesített fennmaradó) | Beosztottak: jóváhagyás/elutasítás; olvasás | Beosztottak: olvasás (típus és időtartam – diagnózis NEM) | Beosztottak: olvasás (naptár) |
| **HR ügyintéző** | Teljes olvasás/írás a hatáskörben | Teljes olvasás/írás | Teljes olvasás/írás (diagnózis NEM) | Teljes olvasás/írás |
| **Bérszámfejtő** | Olvasás | Olvasás (jóváhagyottak) | Olvasás (díjazás típusa, napok száma) | Olvasás |
| **TB ügyintéző** | – | – | Teljes olvasás/írás (TB-ellátás rész) | Olvasás |
| **Rendszeradminisztrátor** | Törzsadat (LeaveType, WorkCalendar) | – | Törzsadat (AbsenceType) | – |

---

## 16. Integrációs pontok

| Külső rendszer | Irány | Érintett adatok | Cél |
|---|---|---|---|
| **Bérszámfejtő modul (payroll)** | kimenő | Havi távolléti napok típusonként, díjazási mód | Távolléti díj / betegszabadság / állásidő elszámolás |
| **NEAK / OEP (társadalombiztosítás)** | kimenő | Keresőképtelenségi adatok, CSED/GYED igénylés | TB-ellátás folyósítás |
| **NAV (08-as bevallás)** | kimenő (payroll-on keresztül) | Távolléti díj, betegszabadság összege | Havi bevallás |
| **Munkaidő-nyilvántartó modul (TimeTracking)** | kétirányú | LeaveRecord → napi nyilvántartás | Mt. 134. § szerinti nyilvántartás |
| **Önkiszolgáló portál** | bejövő | Szabadságigénylés (LeaveRequest); betegség bejelentés | Munkavállaló által kezdeményezett workflow |
| **Naptár-integráció (Outlook, Google)** | kimenő | Jóváhagyott szabadságok, tervezett távollétek | Csapatnaptár szinkronizáció |
| **KIR / KRÉTA (köznevelés)** | kimenő | Pedagógus szabadság és helyettesítés adatok | Tanóra-helyettesítés tervezés |
| **MÁK (kincstár)** | kimenő | Költségvetési szervek távolléti adatai | Kincstári bérszámfejtés |

---

## 17. Kapcsolódó entitások – teljes kép

```
Employment 1 ──── N LeaveEntitlement           (éves jogosultságok)
Employment 1 ──── N LeaveRequest               (szabadságigénylések)
Employment 1 ──── N AbsenceEvent               (távolléti események)
Employment 1 ──── N LeaveRecord                (napi szintű nyilvántartás)

LeaveRequest 1 ──── N LeaveRecord              (igénylés → napi bontás)
AbsenceEvent 1 ──── N LeaveRecord              (esemény → napi bontás)

LeaveEntitlement ──── LeaveType                (típus törzsadat)
LeaveRequest ──── LeaveType
AbsenceEvent ──── AbsenceType

WorkCalendar                                    (éves munkanap-naptár: ünnepnapok, áthelyezések)

Person.birthDate → LeaveEntitlement             (életkor-alapú jogosultság)
Person.dependents → LeaveEntitlement            (gyermek utáni pótszabadság)
Person.disabilityStatus → LeaveEntitlement      (fogyatékossági pótszabadság)
Employment.employmentType → LeaveEntitlement    (jogviszony-típus specifikus szabályok)
Employment.kjt_payGrade → LeaveEntitlement      (Kjt. fizetési osztály pótszabadság)
Position.specialWorkConditions → LeaveEntitlement (munkakörülmény pótszabadság)
```

---

## 18. Nyitott kérdések

1. **Szabadságterv kezelése:** A közszférában (Kit., Kttv.) kötelező az éves szabadságterv. Önálló entitás (`LeavePlan`) vagy a LeaveRequest `DRAFT`/`PLANNED` státusszal?
2. **Helyettesítés kezelése:** A szabadságigényléskor meg kell-e jelölni a helyettesítő személyt? Ha igen, a LeaveRequest tartalmazza (`substitutePersonId`), vagy önálló workflow?
3. **Csapatos szabadság-nézet:** A közvetlen vezető számára a csapat naptár-nézet (ki mikor van szabadságon) milyen szinten legyen elérhető? Típus nélkül (csak „távol") vagy típussal?
4. **TB-ellátások részletes kezelése:** A CSED, GYED, GYES igénylés és elszámolás az HRMS része, vagy külön TB-adminisztrációs modul? Az AbsenceEvent elegendő szintű, vagy kell önálló SocialInsuranceClaim entitás?
5. **Külföldi munkavégzés szabadság-szabályai:** Ha a munkavállaló külföldi kirendelésben dolgozik, a fogadó ország szabadság-szabályai alkalmazandók? Ez az Employment vagy a Leave szintjén kezelendő?
6. **Negatív szabadság-egyenleg:** Engedélyezhető-e, hogy a munkavállaló „előre vegye ki" a jövő évi szabadságát (negatív remaining)? Ha igen, milyen kontrollokkal?
7. **Szabadság-megváltás jogviszony megszűnéskor:** Az elszámolás automatikusan generáljon OneTimePayment rekordot (CompensationElement), vagy manuális?
8. **Koronavírus / járványügyi speciális szabályok:** Szükséges-e a karantén és egyéb járványügyi távollét-típusokat önálló AbsenceType-ként kezelni, vagy elegendő az általános keresőképtelenség?

---

### 18.1. Javaslatok és válaszok

#### 18.1.1. Szabadságterv kezelése

**Probléma:** Kit. 127. § és Kttv. 153. § szerint kötelező az éves szabadságterv készítése és vezetői jóváhagyása. Hogyan kezeljük?

**Javaslat: Önálló LeavePlan entitás (ajánlott Extended verzió) VAGY LeaveRequest.status bővítése (MVP)**

**MVP megoldás – LeaveRequest.status bővítése:**

```
LeaveRequest {
  ...
  status: enum (
    DRAFT,           // Munkavállalói vázlat (még nem nyújtotta be)
    PLANNED,         // ← ÚJ - Szabadságtervben szerepel (év elején)
    SUBMITTED,       // Benyújtva jóváhagyásra
    APPROVED,        // Jóváhagyva
    REJECTED,        // Elutasítva
    TAKEN,           // Kivéve
    CANCELLED        // Visszavonva
  )

  plannedInYear: integer (pl. 2026 - melyik évi szabadságtervbe tartozik)
  approvalLevel: enum (
    PLANNED_APPROVED,      // ← Szabadságterv jóváhagyva (év elején)
    LEAVE_APPROVED         // ← Konkrét szabadság jóváhagyva (kivétel előtt)
  )
}
```

**Használat MVP verzióban:**

```
1. Év elején (január-február):
   - HR létrehoz LeaveRequest-eket status = PLANNED
   - plannedInYear = 2026
   - Vezető áttekinti és jóváhagyja az éves tervet
   - approvalLevel = PLANNED_APPROVED

2. Tényleges szabadság előtt (pl. március):
   - status: PLANNED → SUBMITTED (munkavállaló "aktiválja")
   - Vezető újra jóváhagyja (konkrét szabadság)
   - approvalLevel = LEAVE_APPROVED
   - status: SUBMITTED → APPROVED
```

**Extended megoldás – Önálló LeavePlan entitás:**

```
LeavePlan {
  id: UUID PK
  employmentId: UUID FK
  planYear: integer (2026)
  orgUnitId: UUID FK (szervezeti egység terv)

  // Terv státusza
  status: enum (draft, submitted_to_manager, manager_approved, hr_approved, finalized, published)

  // Jóváhagyók
  submittedBy: UUID FK (munkavállaló vagy HR)
  submittedAt: datetime
  managerApprovalBy: UUID FK (közvetlen vezető)
  managerApprovalAt: datetime
  hrApprovalBy: UUID FK (HR vezető - Kit./Kttv. esetén)
  hrApprovalAt: datetime

  // Összesítő adatok
  totalPlannedDays: integer (tervezett szabadság összesen - számított)
  totalEntitlement: integer (éves jogosultság - referencia)
  remainingUnplanned: integer (még nem tervezett napok - számított)

  // Megjegyzések
  notes: text
  createdAt: datetime
  updatedAt: datetime
}

LeavePlanItem {
  id: UUID PK
  leavePlanId: UUID FK
  leaveRequestId: UUID FK  (kapcsolat a tényleges igényléshez)

  plannedStartDate: date
  plannedEndDate: date
  plannedDays: integer
  leaveTypeId: UUID FK

  // Státusz
  status: enum (planned, approved, modified, cancelled, fulfilled)

  // Ha később módosítják
  actualStartDate: date
  actualEndDate: date
  actualDays: integer
  deviationReason: text
}
```

**Workflow – LeavePlan entitással:**

```
1. JANUÁR (Tervezés):
   LeavePlan (status = draft)
   └── LeavePlanItem #1: jan 15-19 (5 nap)
   └── LeavePlanItem #2: júl 1-14 (10 nap)
   └── LeavePlanItem #3: dec 23-30 (6 nap)

2. FEBRUÁR (Jóváhagyás):
   Vezető áttekinti → LeavePlan.status = manager_approved
   HR jóváhagyja → LeavePlan.status = hr_approved, finalized

3. JÚNIUS (Tényleges szabadság):
   Munkavállaló létrehoz LeaveRequest-et (júl 1-14)
   LeaveRequest.leavePlanItemId = Item #2  (kapcsolat)
   Vezető látja: "ez a szabadságtervben szerepelt" → gyorsított jóváhagyás

4. NOVEMBER (Módosítás):
   Munkavállaló nem tudja dec 23-30-at kivenni
   Helyette: dec 27-31 (3 nap)
   LeavePlanItem #3: status = modified
   Új LeaveRequest létrehozása → normál jóváhagyási folyamat
```

**Jogszabályi megfelelés:**

| Jogviszony | Szabadságterv kötelező? | Jóváhagyás |
|---|---|---|
| **Mt.** | Nem kötelező (ajánlott) | Munkáltató szabályzata szerint |
| **Kit.** | Igen (Kit. 127. §) | Közvetlen vezető + HR vezető |
| **Kttv.** | Igen (Kttv. 153. §) | Közvetlen vezető + jegyző |
| **Kjt.** | Ágazati szabály (általában igen) | Közvetlen vezető |
| **Púétv.** | Igen (Púétv. 90. §) | Intézményvezető |
| **Eszjtv.** | Osztályterv szükséges | Osztályvezető |

**GDPR megfelelés:**

- **Jogalap:** GDPR 6(1)(c) - jogi kötelezettség (Kit., Kttv., Púétv.)
- **Megőrzési idő:** Jogviszony megszűnése + 3 év (munkaügyi per elévülési idő)

**Ajánlás:**

1. **MVP:**
   - LeaveRequest.status = PLANNED
   - LeaveRequest.plannedInYear mezővel
   - Egyszerű jóváhagyás

2. **Extended:**
   - Önálló LeavePlan + LeavePlanItem entitások
   - Többszintű jóváhagyási workflow
   - Terv vs. tényleges eltérés-elemzés (reporting)

3. **Strategic:**
   - Csapat-szintű szabadságterv optimalizáció
   - Automatikus ütközésellenőrzés (max 20% lehet egyszerre szabadságon)
   - AI-alapú szabadság-javaslat (egyenletes elosztás év során)

---

#### 18.1.2. Helyettesítés kezelése

**Probléma:** Szabadság esetén ki helyettesít? Kötelező-e megjelölni? Hogyan kezeljük a rendszerben?

**Javaslat: LeaveRequest mezők bővítése (MVP) + Substitution workflow (Extended)**

**MVP megoldás – LeaveRequest mezők:**

```
LeaveRequest {
  ...
  // Helyettesítés alapadatok
  substituteRequired: boolean (szükséges-e helyettesítés)
  substituteEmploymentId: UUID FK → Employment (helyettesítő személye)
  substituteNotified: boolean (értesítve lett-e)
  substituteNotifiedAt: datetime
  substituteAccepted: boolean (elfogadta-e a helyettesítést)
  substituteNotes: text (helyettesítési megjegyzések, átadandó feladatok)
}
```

**Validációs szabályok:**

```javascript
// Helyettesítés kötelezőségének ellenőrzése
function validateSubstitution(leaveRequest, employment) {
  const requiresSubstitute =
    // Vezetők esetén mindig
    employment.position.isLeaderPosition ||
    // Egyszemélyes munkakörben
    employment.position.headcount === 1.0 ||
    // Kit./Kttv. szabadságnál 5 nap felett
    (employment.employmentType IN ['KORMANYZATI', 'KOZSZOLGALATI'] &&
     leaveRequest.workDaysRequested >= 5) ||
    // Púétv. esetén tanítási időben
    (employment.employmentType === 'KOZNEVELESI' &&
     leaveRequest.startDate.isWithinSchoolYear());

  if (requiresSubstitute && !leaveRequest.substituteEmploymentId) {
    throw new ValidationError(
      'Helyettesítő személy megjelölése kötelező ennél a szabadságnál'
    );
  }
}
```

**Extended megoldás – Substitution entitás + workflow:**

```
Substitution {
  id: UUID PK
  leaveRequestId: UUID FK

  // Helyettesítő adatai
  substituteEmploymentId: UUID FK
  substituteType: enum (
    DIRECT_SUBSTITUTE,        // Közvetlen helyettesítés (feladatok átvétele)
    TASK_REDISTRIBUTION,      // Feladatok szétosztása (több személy)
    WORKFLOW_DELEGATION,      // Workflow átirányítás
    NO_COVERAGE                // Nincs helyettesítés (pl. projekt-alapú munka)
  )

  // Időtartam (lehet eltér a szabadságtól)
  effectiveFrom: date
  effectiveTo: date

  // Jóváhagyás
  proposedBy: enum (EMPLOYEE, MANAGER, HR)
  proposedAt: datetime
  substituteApprovalStatus: enum (pending, accepted, declined, expired)
  substituteApprovedAt: datetime
  substituteDeclineReason: text

  // Feladatok átadása
  handoverDocumentId: UUID FK → Document (átadás-átvételi jegyzőkönyv)
  taskList: text (átadandó feladatok listája)
  accessRightsRequired: JSON (milyen jogosultságok szükségesek)

  // Kompenzáció (ha van)
  substituteCompensationType: enum (none, allowance, time_off)
  substituteCompensationAmount: decimal

  // Státusz
  status: enum (proposed, accepted, active, completed, cancelled)
  completedAt: datetime

  notes: text
}

SubstitutionTask {
  id: UUID PK
  substitutionId: UUID FK
  taskDescription: text
  priority: enum (high, medium, low)
  estimatedHours: decimal
  completedBy: UUID FK → Employment
  completedAt: datetime
  status: enum (pending, in_progress, completed, cancelled)
}
```

**Jogviszony-specifikus szabályok:**

| Jogviszony | Helyettesítés szabályozása | Rendszer viselkedés |
|---|---|---|
| **Mt.** | Nincs jogszabályi előírás | Opcionális mező, munkáltatói szabályzat szerint |
| **Kit./Kttv.** | Jogszabályban nincs, de munkakör-függő (vezetők!) | Kötelező vezetőknél, 5+ nap szabadságnál |
| **Púétv.** | Tanítási időben kötelező helyettesítés | Kötelező tanítási időben, helyettesítési óradíj számítása |
| **Eszjtv.** | Osztályterv szerinti helyettesítés | Kötelező ügyeletnél, műtéti időpontoknál |
| **Kjt.** | Intézményi szabályzat szerint | Opcionális, de ajánlott |

**Púétv. speciális eset – Helyettesítési óradíj:**

```
SubstitutionCompensation {
  id: UUID PK
  substitutionId: UUID FK
  substituteEmploymentId: UUID FK

  // Helyettesített órák
  date: date
  subjectCode: string (tantárgy kódja)
  hoursSubstituted: integer (hány órát helyettesített)

  // Díjazás
  hourlyRate: decimal (Ft/óra - Púétv. 106. § szerinti díj)
  totalCompensation: decimal (számított)

  // Kapcsolódik bérszámfejtéshez
  compensationElementId: UUID FK → CompensationElement
}
```

**Workflow – Helyettesítéssel:**

```
1. Munkavállaló szabadságot igényel (LeaveRequest):
   - startDate: 2026-03-10
   - endDate: 2026-03-14
   - substituteRequired: true (automatikusan kitöltve - vezető)

2. Rendszer javasol helyettesítőket:
   - Ugyanaz a Position (ha több fő)
   - Azonos OrgUnit, hasonló kompetenciák
   - Nincs ütköző szabadság

3. Munkavállaló kiválaszt helyettesítőt:
   - substituteEmploymentId: emp-456
   - taskList: "Projekttervek jóváhagyása, csapatmegbeszélés vezetése"

4. Helyettesítő értesítése:
   - Email/rendszerüzenet: "Kérjük, vállalja a helyettesítést"
   - substituteApprovalStatus: pending

5. Helyettesítő elfogadja:
   - substituteApprovalStatus: accepted
   - Vezető jóváhagyja a szabadságot

6. Szabadság alatt:
   - Substitution.status: active
   - Workflow-k automatikusan átirányítódnak helyettesítőhöz

7. Visszatérés után:
   - Substitution.status: completed
   - Opcionálisan: SubstitutionTask-ok lezárása
```

**IT rendszer integráció:**

```
- Email átirányítás (Out of Office helyettesítővel)
- Workflow átirányítás (jóváhagyások → helyettesítő)
- Jogosultságok ideiglenes delegálása (opcionális, biztonsági audit!)
- Naptár megosztás
```

**GDPR megfelelés:**

- **Jogalap:** GDPR 6(1)(f) jogos érdek (folyamatos üzletmenet biztosítása)
- **Arányosság:** Csak a szükséges feladatok és adatok átadása
- **Tájékoztatás:** Munkavállaló tudja, ki a helyettesítő
- **Megőrzési idő:** Helyettesítés befejezése + 1 év (audit célból)

**Ajánlás:**

1. **MVP:**
   - LeaveRequest.substituteEmploymentId
   - Egyszerű értesítés
   - Manuális feladat-átadás

2. **Extended:**
   - Substitution entitás
   - Helyettesítő jóváhagyási workflow
   - Helyettesítési óradíj automatikus számítása (Púétv.)

3. **Strategic:**
   - Automatikus helyettesítő-javaslat (ML alapú)
   - Workflow-delegálás automatizálása
   - Kompetencia-alapú helyettesítés-tervezés

---

#### 18.1.3. Csapatos szabadság-nézet (vezető számára)

**Probléma:** A vezető látja-e, hogy a beosztottjai MIÉRT vannak távol (szabadság vs. betegség)? GDPR vs. üzleti szükséglet.

**Javaslat: Többszintű láthatóság (konfigurálható)**

**Láthatósági szintek:**

```
LeaveVisibilityLevel {
  NONE = 0              // Semmit nem lát (védett adat)
  STATUS_ONLY = 1       // Csak "távol" / "jelen" státusz
  CATEGORY = 2          // Kategória: "szabadság" / "beteg" / "egyéb"
  TYPE_SUMMARY = 3      // Típus összefoglalva: "fizetett távollét"
  FULL_TYPE = 4         // Teljes típus: "éves szabadság", "betegszabadság"
  FULL_DETAILS = 5      // Teljes részletek (diagnózis NEM! - GDPR 9. cikk)
}
```

**Implementáció – OrgUnit szintű konfiguráció:**

```
OrgUnit {
  ...
  // Vezető számára láthatóság beállítás
  managerLeaveVisibility: enum (
    STATUS_ONLY,
    CATEGORY,
    TYPE_SUMMARY,
    FULL_TYPE
  )

  // Csapat számára láthatóság (opcionális)
  teamLeaveVisibility: enum (
    NONE,
    STATUS_ONLY,
    CATEGORY
  )
}
```

**Employment szintű felülírás (opcionális):**

```
Employment {
  ...
  // Egyedi beállítás (felülírja az OrgUnit beállítást)
  overrideLeaveVisibility: boolean
  customLeaveVisibility: enum (STATUS_ONLY, CATEGORY, FULL_TYPE)
}
```

**Megjelenítés – Példák:**

```
1. STATUS_ONLY (minimális):
   Naptár nézet:
   ┌─────────┬──────┬──────┬──────┐
   │ Név     │ H    │ K    │ Sze  │
   ├─────────┼──────┼──────┼──────┤
   │ Kiss J. │ Jelen│ Távol│ Távol│
   │ Nagy P. │ Távol│ Jelen│ Jelen│
   └─────────┴──────┴──────┴──────┘

2. CATEGORY (kategorizált):
   Naptár nézet:
   ┌─────────┬───────────┬──────┬──────┐
   │ Név     │ H         │ K    │ Sze  │
   ├─────────┼───────────┼──────┼──────┤
   │ Kiss J. │ Jelen     │🏖️ Szabadság│🏖️ Szabadság│
   │ Nagy P. │🤒 Beteg   │ Jelen│ Jelen│
   │ Tóth A. │🏖️ Szabadság│🏖️ Szabadság│ Jelen│
   └─────────┴───────────┴──────┴──────┘

3. FULL_TYPE (részletes):
   Naptár nézet:
   ┌─────────┬─────────────────┬──────┬──────┐
   │ Név     │ H               │ K    │ Sze  │
   ├─────────┼─────────────────┼──────┼──────┤
   │ Kiss J. │ Jelen           │ Éves szabadság│ Éves szabadság│
   │ Nagy P. │ Betegszabadság  │ Jelen│ Jelen│
   │ Tóth A. │ Apasági szabadság│ Apasági szabadság│ Jelen│
   └─────────┴─────────────────┴──────┴──────┘
```

**GDPR megfelelés:**

| Távollét típus | Kategorizálás | GDPR kockázat | Vezető láthatóság |
|---|---|---|---|
| **Éves szabadság** | Szabadság | Nincs (nem különleges adat) | ✅ FULL_TYPE |
| **Betegszabadság** | Beteg | Közép (egészségügyi adat közvetetten) | ◐ CATEGORY ("beteg") - diagnózis NEM! |
| **Keresőképtelenség** | Beteg | Magas (GDPR 9. cikk) | ⚠️ CATEGORY - diagnózis szigorúan tilos! |
| **Szülési szabadság** | Egyéb/Szabadság | Közép (terhesség) | ✅ TYPE vagy CATEGORY |
| **GYED/GYES** | Egyéb | Közép (gyermek) | ✅ TYPE (köztudomású) |
| **Fizetés nélküli (egyéb)** | Egyéb | Alacsony | ✅ TYPE vagy CATEGORY |

**GDPR 9. cikk – Egészségügyi adatok védelme:**

```
❌ TILOS megjeleníteni:
   - Diagnózis (betegség neve)
   - Kezelés típusa
   - Orvosi igazolás részletei

✅ MEGENGEDETT:
   - "Betegszabadság" (anélkül, hogy mi a betegség)
   - "Keresőképtelenség" (táppénz)
   - Napok száma
   - Időtartam
```

**Beállítási útmutató szervezetenként:**

| Szervezet típus | Ajánlott beállítás | Indok |
|---|---|---|
| **Kis csapat (5-10 fő)** | FULL_TYPE | Mindenki tudja, ki mikor van szabadságon |
| **Közepes (10-50 fő)** | CATEGORY | Egyensúly átláthatóság és privacy között |
| **Nagy szervezet (50+ fő)** | CATEGORY vagy TYPE_SUMMARY | GDPR compliance, csak szükséges info |
| **Egészségügy (Eszjtv.)** | CATEGORY (max) | Szigorú GDPR - betegadatok védelme |
| **Közigazgatás (Kit./Kttv.)** | FULL_TYPE (szabadságnál), CATEGORY (betegnél) | Átláthatóság vs. személyiségi jogok |

**Önkiszolgáló beállítás (opcionális - Extended):**

```
EmployeePrivacySettings {
  employmentId: UUID FK

  // Munkavállaló döntheti el, mi látható
  allowManagerToSeeLeaveType: boolean (default: true)
  allowTeamToSeeAbsence: boolean (default: false)

  // Kivételek
  hideSpecificTypes: array[LeaveTypeId] (pl. pszichológiai probléma miatt beteg)
}
```

**Ajánlás:**

1. **MVP:**
   - OrgUnit szintű `managerLeaveVisibility` beállítás
   - CATEGORY szint (szabadság/beteg/egyéb)
   - Fix beállítás, nincs egyedi felülírás

2. **Extended:**
   - Employment szintű felülírás
   - Munkavállaló választhat (opt-out bizonyos típusoknál)
   - Részletes audit napló (ki nézte meg, mikor)

3. **Strategic:**
   - AI-alapú kapacitás-tervezés (túl sok egyidejű távollét esetén figyelmeztetés)
   - Anomália-detektálás (szokatlan távolléti minták)

**Konklúzió:** CATEGORY szint ajánlott a legtöbb szervezetnél (egyensúly átláthatóság és GDPR között).

---

#### 18.1.4. TB-ellátások részletes kezelése (CSED, GYED, GYES)

**Probléma:** A CSED, GYED, GYES igénylése, folyósítása, elszámolása komplex adminisztrációs folyamat. HRMS scope-ja, vagy külön modul?

**Javaslat: HRMS scope-ban marad, de kétszintű (MVP vs. Extended)**

**MVP megoldás – AbsenceEvent bővítése:**

```
AbsenceEvent {
  ...
  absenceType: enum (
    ...,
    CSED,                    // Csecsemőgondozási díj
    GYED,                    // Gyermekgondozási díj
    GYES,                    // Gyermekgondozási segély
    GYET,                    // Gyermeknevelési támogatás
    ...
  )

  // TB-ellátás alapadatok
  socialBenefitType: enum (csed, gyed, gyes, gyet, tavf, rehabilitation, other)
  socialBenefitClaimNumber: string (ügyszám - NEAK)
  socialBenefitStartDate: date
  socialBenefitEndDate: date (számított vagy tényleges)

  // Gyermekhez kapcsolódó adatok (CSED/GYED/GYES esetén)
  childPersonId: UUID FK → Person.dependents (melyik gyermek után)
  childBirthDate: date
  childMultipleBirth: boolean (többes szülés - hosszabb CSED)

  // Ellátás összege
  monthlyBenefitAmount: decimal (Ft/hó - tájékoztató)
  benefitPaymentResponsibility: enum (social_insurance, employer_supplementary)

  // Integrációs adatok
  neakCaseId: string (NEAK ügyszám)
  neakSubmittedAt: datetime
  neakApprovalStatus: enum (pending, approved, rejected, appealed)
  neakApprovalDate: date

  // Kapcsolódó dokumentumok
  birthCertificateId: UUID FK → Document
  medicalCertificateId: UUID FK → Document (CSED - szülésznői igazolás)

  notes: text
}
```

**Használat MVP-ben:**

```
Scenario: GYED igénylés

1. AbsenceEvent létrehozása:
   absenceType: GYED
   socialBenefitType: gyed
   childPersonId: person-dep-001 (2 éves gyermek)
   startDate: 2026-03-01
   endDate: 2028-03-01 (2 év)
   monthlyBenefitAmount: 108000 (tájékoztató - GYED max összeg 2026-ban)

2. NEAK bejelentés (manuális vagy API):
   neakCaseId: "GYED-2026-123456"
   neakSubmittedAt: 2026-02-15
   neakApprovalStatus: pending

3. NEAK jóváhagyás:
   neakApprovalStatus: approved
   neakApprovalDate: 2026-02-20

4. Havi LeaveRecord generálás:
   - Minden hónapban LeaveRecord (naptári napok)
   - Bérszámfejtés: nincs távolléti díj (GYED = TB-ellátás)
```

**Extended megoldás – SocialBenefitClaim entitás:**

```
SocialBenefitClaim {
  id: UUID PK
  employmentId: UUID FK
  absenceEventId: UUID FK (kapcsolat az AbsenceEvent-hez)

  // Ellátás típusa
  benefitType: enum (csed, gyed, gyes, gyet, tavf, tappenz, rehabilitation, sick_pay)
  claimNumber: string (belső ügyszám)

  // Jogosultság
  eligibilityStartDate: date
  eligibilityEndDate: date
  eligibilityBasis: text (pl. "gyermek 2 éves koráig")

  // Összeg
  calculationBasis: decimal (pl. GYED: átlagkereset 70%-a, max 108k)
  calculatedAmount: decimal
  approvedAmount: decimal (NEAK által jóváhagyott)
  currency: string (HUF)

  // Igénylés státusza
  status: enum (draft, submitted, under_review, approved, rejected, appealed, active, suspended, terminated)

  // NEAK integráció
  neakCaseId: string
  neakSubmittedAt: datetime
  neakSubmittedBy: UUID FK (HR ügyintéző)
  neakApprovalStatus: enum (pending, approved, partial_approved, rejected)
  neakApprovalDate: date
  neakRejectionReason: text

  // Dokumentumok
  applicationFormId: UUID FK → Document (kérvény)
  birthCertificateId: UUID FK → Document
  employmentCertificateId: UUID FK → Document (munkáltatói igazolás)
  incomeCertificateId: UUID FK → Document (jövedelemigazolás)

  // Folyósítás
  firstPaymentDate: date
  lastPaymentDate: date
  suspensionStartDate: date (ha szünetel)
  suspensionReason: text

  // Visszatérés munkahelyre
  plannedReturnDate: date
  actualReturnDate: date
  earlyReturnNotificationDate: date (előzetes bejelentés)

  // Audit
  createdAt: datetime
  createdBy: UUID FK
  updatedAt: datetime
  notes: text
}

SocialBenefitPayment {
  id: UUID PK
  socialBenefitClaimId: UUID FK

  paymentMonth: date (2026-03-01)
  paymentAmount: decimal
  paymentDate: date (tényleges folyósítás dátuma)
  paymentStatus: enum (pending, paid, suspended, reclaimed)

  // Visszakövetelés
  reclaimAmount: decimal (ha túlfizetés volt)
  reclaimReason: text

  neakPaymentId: string (NEAK fizetési azonosító)
}
```

**Scope elemzés:**

| Funkció | MVP (AbsenceEvent) | Extended (SocialBenefitClaim) | Külső TB modul |
|---|---|---|---|
| **Ellátás típusa, időtartam** | ✅ | ✅ | ✅ |
| **Jogosultság ellenőrzése** | Manuális | ✅ Automatikus | ✅ |
| **NEAK bejelentés** | Manuális/API | ✅ API integráció | ✅ |
| **Havi folyósítás nyomon követése** | ❌ | ✅ | ✅ |
| **Összeférhetetlenség ellenőrzése** | ❌ | ✅ (pl. GYED + munkaviszony korlátozás) | ✅ |
| **Visszakövetelés kezelése** | ❌ | ◐ Alapszinten | ✅ |
| **Komplex TB-jogi esetek** | ❌ | ❌ | ✅ |

**Jogszabályi követelmények:**

```
CSED (Csecsemőgondozási díj - Tbj. 42/B. §):
- Időtartam: Szülés napjától 168 nap (24 hét)
- Összeg: Átlagkereset 70%-a, max napi limit
- Többes szülés: +30 nap gyermekenként
- Apának is jár (megoszthatják)

GYED (Gyermekgondozási díj - Tbj. 42/C. §):
- Időtartam: CSED után gyermek 2 éves koráig
- Összeg: Átlagkereset 70%-a, max 108.000 Ft/hó (2026)
- Munkaviszony korlátozás: max 30 óra/hét vagy távmunka

GYES (Gyermekgondozási segély - Tbj. 42/D. §):
- Időtartam: Gyermek 3 éves koráig (max)
- Összeg: Összegfixált (kb. 37.000 Ft/hó 2026-ban)
- Nincs munkaviszony-korlátozás

GYET (Gyermeknevelési támogatás - Tbj. 42/E. §):
- Időtartam: 3+ gyermek esetén legkisebb 3-8 éves koráig
- Összeg: Összegfixált (kb. 37.000 Ft/hó)
```

**GDPR megfelelés:**

- **Jogalap:** GDPR 6(1)(c) - jogi kötelezettség (Tbj. bejelentési kötelezettség)
- **Különleges adat:** GDPR 9(2)(b) - foglalkoztatási jog (egészségügyi adat - szülés)
- **Gyermek adatai:** GDPR 6(1)(c) - jogi kötelezettség (GYED/GYES jogosultság)
- **Megőrzési idő:** Ellátás megszűnése + 8 év (Tbj. ellenőrzési idő)

**Ajánlás:**

1. **MVP (kis/közepes szervezet):**
   - AbsenceEvent bővítése TB-mezőkkel
   - Manuális NEAK bejelentés (papír vagy NEAK online felület)
   - Távolléti napok rögzítése (LeaveRecord)

2. **Extended (nagy szervezet, 500+ fő):**
   - SocialBenefitClaim + SocialBenefitPayment entitások
   - NEAK API integráció (e-NEAK rendszer)
   - Automatikus jogosultság-ellenőrzés
   - Munkaviszony-korlátozás monitoring (GYED + munkaidő)

3. **Külső TB-modul (specializált igény):**
   - Különálló TB-adminisztráció modul
   - HRMS csak alapadatokat szolgáltat
   - Visszakövetelés, jogorvoslat, komplex TB-jogi esetek

**Konklúzió:** Extended verzió elegendő a legtöbb szervezetnél. Külső modul csak speciális (pl. TB-kifizetőhely, nagy közigazgatási szerv) esetén indokolt.

---

#### 18.1.5. Külföldi munkavégzés szabadság-szabályai

**Probléma:** Kiküldött munkavállaló (posted worker) szabadsága melyik ország szabályai szerint? Hogyan kezeljük a rendszerben?

**Javaslat: Assignment entitás + LeaveEntitlement felülbírálás**

**Jogszabályi háttér:**

```
EU Kiküldött Munkavállalók Direktíva (96/71/EK):
- Munkavégzés helye szerinti ország MINIMUMKÖVETELMÉNYEI érvényesek
- Szabadság: fogadó ország VAGY küldő ország szabálya (amelyik kedvezőbb)

Példa:
- Magyar munkavállaló → Németországba kiküldve
- Magyarország: 20 nap alap + pót (Mt.)
- Németország: 20 nap alap (BUrlG)
- → Magyar szabály érvényesül (ha kedvezőbb pótszabadság van)
```

**Megvalósítás:**

```
Assignment {
  ...
  // Külföldi kiküldés
  hostCountry: string(2) (ISO 3166-1 - DE, AT, SK, stb.)

  // Szabadság szabályozása
  leaveRegulation: enum (
    HOME_COUNTRY,              // Küldő ország (Magyarország)
    HOST_COUNTRY,              // Fogadó ország
    MOST_FAVORABLE,            // Amelyik kedvezőbb
    CUSTOM                     // Egyedi megállapodás
  )

  // Ha CUSTOM:
  customLeaveEntitlementDays: integer
  customLeaveRules: text

  // A1 igazolás (TB koordináció)
  a1CertificateId: UUID FK → Document
  a1ValidFrom: date
  a1ValidTo: date
}
```

**LeaveEntitlement felülbírálás (opcionális):**

```
LeaveEntitlement {
  ...
  // Külföldi kiküldés miatt módosított jogosultság
  isOverriddenByAssignment: boolean
  assignmentId: UUID FK
  originalEntitlement: integer (eredeti magyar szabály)
  adjustedEntitlement: integer (külföldi szabály vagy legkedvezőbb)
  adjustmentReason: text
}
```

**Számítási logika:**

```javascript
function calculateLeaveEntitlement(employment, year) {
  let baseEntitlement = 20; // Mt. alapszabadság
  let supplementary = calculateSupplementary(employment); // Pótszabadságok

  // Van-e aktív külföldi kiküldés?
  const assignment = getActiveAssignment(employment, year);

  if (assignment && assignment.assignmentType === 'POSTED_WORKER') {
    switch (assignment.leaveRegulation) {
      case 'HOME_COUNTRY':
        // Magyar szabály érvényesül
        return baseEntitlement + supplementary;

      case 'HOST_COUNTRY':
        // Fogadó ország szabálya
        const hostEntitlement = getHostCountryEntitlement(assignment.hostCountry, employment);
        return hostEntitlement;

      case 'MOST_FAVORABLE':
        // Amelyik több
        const hungarianTotal = baseEntitlement + supplementary;
        const hostTotal = getHostCountryEntitlement(assignment.hostCountry, employment);
        return Math.max(hungarianTotal, hostTotal);

      case 'CUSTOM':
        // Egyedi megállapodás
        return assignment.customLeaveEntitlementDays;
    }
  }

  // Nincs kiküldés - magyar szabály
  return baseEntitlement + supplementary;
}

function getHostCountryEntitlement(countryCode, employment) {
  // Külső API vagy törzsadat
  const hostCountryRules = {
    'DE': 20,  // Németország: 20 nap (BUrlG)
    'AT': 25,  // Ausztria: 25 nap
    'SK': 20,  // Szlovákia: 20 nap (+ pót életkor szerint)
    'RO': 20,  // Románia: 20 nap
    'UK': 28,  // UK: 28 nap (bank holidays-vel együtt)
  };

  return hostCountryRules[countryCode] || 20; // default: 20 nap
}
```

**Példa forgatókönyv:**

```
Munkavállaló: Kiss János
Employment: Mt. (magyar jogviszony)
Életkor: 32 év (nincs pótszabadság)
Magyar jogosultság: 20 nap

Assignment:
- Németországba kiküldve (2026. jan 1. - dec 31.)
- leaveRegulation: MOST_FAVORABLE
- hostCountry: DE

Számítás:
- Magyar szabály: 20 nap
- Német szabály: 20 nap (BUrlG)
- Eredmény: 20 nap (azonos)

De ha:
- Életkor: 45 év → Magyar pót: +5 nap (25-30 év jogviszony)
- Magyar szabály: 20 + 5 = 25 nap
- Német szabály: 20 nap
- Eredmény: 25 nap (magyar kedvezőbb) ✅
```

**WorkCalendar integráció:**

```
WorkCalendar {
  ...
  country: string(2) (ISO 3166-1)

  // Ünnepnapok országonként
  // Például:
  // HU: 2026-03-15 (nemzeti ünnep)
  // DE: 2026-10-03 (német egyesítés napja - csak DE-ben)
}

// Szabadságigényléskor:
// - Ha kiküldött dolgozó DE-ben dolgozik
// - 2026-10-03 német ünnepnap → NEM munkanap (nem vonódik le szabadságból)
// - DE 2026-03-15 sima munkanap (magyar ünnep nem számít)
```

**Bonyolult eset – Brexit utáni UK:**

```
UK munkavállaló → Magyarországra kiküldve:
- UK szabály: 28 nap (statutory leave - 20 nap + 8 bank holiday)
- Magyar szabály: 20 nap + pót
- Kérdés: Bank holidays beszámítanak-e?

Megoldás:
Assignment.customLeaveRules: "28 nap éves szabadság, ebből 8 nap bank holiday-ként kerül kiadásra (fix dátumok)"
```

**GDPR megfelelés:**

- **Jogalap:** GDPR 6(1)(c) - jogi kötelezettség (munkaügyi szabályozás)
- **Adattovábbítás:** Ha nem-EU ország → megfelelőségi vizsgálat (GDPR 44-49. cikk)
- **Megőrzési idő:** Assignment megszűnése + 5 év

**Ajánlás:**

1. **MVP (ritka kiküldés):**
   - Assignment entitás (egyszerű verzió)
   - Manuális LeaveEntitlement módosítás HR által
   - leaveRegulation: HOME_COUNTRY (magyar szabály)

2. **Extended (gyakori kiküldés):**
   - Automatikus számítás: MOST_FAVORABLE logika
   - Ország-specifikus szabályok törzsadata (CountryLeaveRules)
   - WorkCalendar országkód szerinti szűrés

3. **Strategic (multinacionális cég):**
   - Teljes nemzetközi szabadság-menedzsment
   - Több ország egyidejű kiküldése (komplex eset)
   - Adózási implikációk (183 napos szabály)

**Konklúzió:** Extended verzió ajánlott EU-n belüli kiküldéseknél. MVP elegendő, ha ritkán fordul elő.

---

#### 18.1.6. Negatív szabadság-egyenleg

**Probléma:** Engedélyezhető-e, hogy a munkavállaló előre vegye a jövő évi szabadságát? Ha igen, milyen kontrollokkal?

**Javaslat: Konfiguráció alapján engedélyezhető, szigorú szabályokkal**

**Jogszabályi háttér:**

```
Mt. 116. § (1): "A munkavállalót munkaviszonyának fennállásának minden naptári éve után rendes szabadság illeti meg."
→ Jogosultság ÉVENTE keletkezik
→ "Előre venni" a jövő évi szabadságot jogszabályban NINCS szabályozva
→ Munkáltatói döntés (belső szabályzat)

Kit./Kttv./Púétv.: Hasonló, nincs kifejezett szabályozás
```

**Implementáció – Organization szintű szabályzat:**

```
Organization {
  ...
  // Szabadság-szabályzat
  allowNegativeLeaveBalance: boolean (default: false)
  maxNegativeLeaveDays: integer (ha allowNegativeLeaveBalance = true, pl. -5 nap)
  negativeLeaveRequiresApproval: enum (
    MANAGER_ONLY,              // Csak közvetlen vezető
    MANAGER_AND_HR,            // Vezető + HR jóváhagyás
    HR_DIRECTOR                // HR vezető jóváhagyás kötelező
  )
  negativeLeaveRestrictions: JSON {
    "allowOnlyForPermanentEmployee": true,  // Csak határozatlan időre
    "minimumSeniorityMonths": 12,           // Min 1 év jogviszony
    "excludeProbation": true,               // Próbaidő alatt tiltva
    "carryForwardMandatory": true           // Következő évben le kell dolgozni
  }
}
```

**LeaveEntitlement bővítése:**

```
LeaveEntitlement {
  ...
  // Egyenleg mezők
  totalAvailable: integer (alapjogosultság + kiegészítő + előző évről áthozott)
  used: integer (felhasznált napok)
  remaining: integer (fennmaradó - LEHET NEGATÍV!)

  // Negatív egyenleg kezelése
  allowNegativeBalance: boolean (számított - Organization + Employment alapján)
  negativeBalanceLimit: integer (max mennyit mehet mínuszba)
  negativeBalanceCurrent: integer (aktuális negatív egyenleg, ha van)

  // Következő évi korrekció
  carryForwardDebt: integer (ha év végén negatív, jövő évben levonásra kerül)
}
```

**Validációs szabályok:**

```javascript
function validateLeaveRequest(leaveRequest, entitlement, employment, organization) {
  const requestedDays = leaveRequest.workDaysRequested;
  const currentRemaining = entitlement.remaining;
  const afterRequest = currentRemaining - requestedDays;

  // 1. Ha negatív lesz az egyenleg
  if (afterRequest < 0) {
    // 1.1 Megengedett-e negatív egyenleg?
    if (!organization.allowNegativeLeaveBalance) {
      throw new ValidationError(
        `Nincs elegendő szabadság. Fennmaradó: ${currentRemaining} nap, igényelt: ${requestedDays} nap.`
      );
    }

    // 1.2 Meghaladja-e a negatív limitet?
    const negativeAmount = Math.abs(afterRequest);
    if (negativeAmount > organization.maxNegativeLeaveDays) {
      throw new ValidationError(
        `Maximum ${organization.maxNegativeLeaveDays} nap negatív egyenleg engedélyezett. ` +
        `Igénylés esetén: -${negativeAmount} nap lenne.`
      );
    }

    // 1.3 Jogviszony-kritériumok ellenőrzése
    const restrictions = organization.negativeLeaveRestrictions;

    if (restrictions.excludeProbation && employment.probationEndDate > new Date()) {
      throw new ValidationError('Próbaidő alatt negatív egyenleg nem engedélyezett.');
    }

    if (restrictions.allowOnlyForPermanentEmployee &&
        employment.contractDuration === 'FIXED_TERM') {
      throw new ValidationError('Határozott idejű jogviszony esetén negatív egyenleg nem engedélyezett.');
    }

    const seniorityMonths = calculateSeniorityMonths(employment);
    if (seniorityMonths < restrictions.minimumSeniorityMonths) {
      throw new ValidationError(
        `Minimum ${restrictions.minimumSeniorityMonths} hónap jogviszony szükséges negatív egyenleghez.`
      );
    }

    // 1.4 Magasabb szintű jóváhagyás szükséges
    leaveRequest.requiresHigherApproval = true;
    leaveRequest.approvalLevel = organization.negativeLeaveRequiresApproval;
    leaveRequest.negativeBalanceWarning =
      `FIGYELEM: Ez az igénylés negatív egyenleget eredményez (-${negativeAmount} nap). ` +
      `A következő évi szabadságból automatikusan levonásra kerül.`;
  }

  return true; // Validáció sikeres (de lehet figyelmeztető üzenet)
}
```

**Jóváhagyási workflow (negatív egyenleg esetén):**

```
LeaveRequest {
  status: SUBMITTED
  requiresHigherApproval: true
  approvalLevel: MANAGER_AND_HR
  negativeBalanceWarning: "FIGYELEM: -3 nap egyenleg..."
}

Workflow:
1. Közvetlen vezető jóváhagyja → status = MANAGER_APPROVED
2. HR ügyintéző jóváhagyja → status = APPROVED
3. LeaveRecord generálása
4. LeaveEntitlement.remaining = -3
5. LeaveEntitlement.carryForwardDebt = 3 (következő évre)
```

**Év végi elszámolás:**

```javascript
// Év végi szabadság-zárás (2026. december 31.)
function yearEndLeaveSettlement(employment, year) {
  const currentEntitlement = getLeaveEntitlement(employment, year);

  if (currentEntitlement.remaining < 0) {
    // Negatív egyenleg → következő évre átvisszük tartozásként
    const nextYearEntitlement = createLeaveEntitlement(employment, year + 1);

    nextYearEntitlement.baseEntitlement = 20; // 2027-es alap
    nextYearEntitlement.supplementaryEntitlement = 5; // pótszabadság
    nextYearEntitlement.carryForwardDebt = Math.abs(currentEntitlement.remaining); // pl. 3 nap tartozás
    nextYearEntitlement.totalAvailable = 20 + 5 - 3 = 22; // ← Csökkentett jogosultság!

    // Figyelmeztetés a munkavállalónak
    sendNotification(employment.personId, {
      title: "Szabadság-tartozás 2027-re",
      message: `2026-os negatív egyenleg (${currentEntitlement.remaining} nap) levonásra került a 2027-es jogosultságból.`
    });
  }
}
```

**Jogviszony megszűnéskor:**

```javascript
// Ha a munkavállaló kilép NEGATÍV egyenleggel
function handleTerminationWithNegativeLeave(employment, terminationDate) {
  const currentYear = terminationDate.getFullYear();
  const entitlement = getLeaveEntitlement(employment, currentYear);

  if (entitlement.remaining < 0) {
    // PROBLÉMA: Előre vette a szabadságot, de már nem dolgozza le

    // 1. Számítás: Hány nap tartozás maradt?
    const negativeDays = Math.abs(entitlement.remaining);

    // 2. Pénzben visszakövetelés
    const dailyWage = calculateDailyWage(employment);
    const debtAmount = negativeDays * dailyWage;

    // 3. CompensationElement vagy OneTimePayment (NEGATÍV)
    createOneTimePayment({
      employmentId: employment.id,
      paymentType: 'LEAVE_DEBT_DEDUCTION',
      amount: -debtAmount,  // NEGATÍV = levonás
      paymentDate: terminationDate,
      description: `Elővételezett szabadság visszakövetelése (${negativeDays} nap)`,
      legalBasis: 'Mt. 167. § - elszámolás',
      status: 'PLANNED'
    });

    // 4. Figyelmeztetés
    console.warn(`Negatív szabadság-egyenleg kilépéskor: ${employment.id}, ${negativeDays} nap, ${debtAmount} Ft`);
  }
}
```

**Reporting & Monitoring:**

```sql
-- Negatív egyenlegű munkavállalók riport
SELECT
  p.familyName,
  p.givenNames,
  e.employmentNumber,
  le.year,
  le.totalAvailable,
  le.used,
  le.remaining,
  le.negativeBalanceCurrent
FROM leave_entitlement le
JOIN employment e ON le.employmentId = e.id
JOIN person p ON e.personId = p.id
WHERE le.remaining < 0
  AND e.status = 'ACTIVE'
ORDER BY le.remaining ASC;

-- Eredmény:
-- Nagy Péter    EMP-123   2026   20   23   -3   -3
-- Kiss János    EMP-456   2026   25   28   -3   -3
```

**GDPR megfelelés:**

- **Jogalap:** GDPR 6(1)(c) - jogi kötelezettség (munkaviszony adminisztráció)
- **Megőrzési idő:** Jogviszony megszűnése + 3 év (munkaügyi per elévülési idő)

**Ajánlás:**

1. **MVP (konzervatív):**
   - `allowNegativeLeaveBalance = false` (alapértelmezett)
   - Csak a tárgyévi jogosultság használható fel

2. **Extended (rugalmas):**
   - `allowNegativeLeaveBalance = true` (konfigurálható)
   - Max -5 nap negatív egyenleg
   - Magasabb jóváhagyási szint (MANAGER_AND_HR)
   - Év végi automatikus elszámolás
   - Kilépéskor automatikus visszakövetelés

3. **Strategic:**
   - Prediktív analitika: "Ki veszi rendszeresen előre a szabadságot?"
   - Kockázatelemzés: "Mekkora a potenciális pénzügyi kockázat?"

**Konklúzió:** Extended verzió ajánlott, ha a szervezet munkavállalóbarát és alacsony fluktuációjú. MVP biztonságosabb, ha magas a fluktuáció vagy rövid jogviszonyok jellemzőek.

---

#### 18.1.7. Szabadság-megváltás jogviszony megszűnéskor

**Probléma:** Ki nem vett szabadság pénzben történő megváltása. Automatikus generálás vagy manuális?

**Javaslat: Automatikus számítás + manuális jóváhagyás (ajánlott)**

**Jogszabályi háttér:**

```
Mt. 122. § (4): "Ha a munkaviszony megszűnik, a ki nem adott szabadságot pénzben meg kell váltani."
→ KÖTELEZŐ megváltani
→ Távolléti díj alapján számítva (nem alapbér!)

Kit./Kttv./Púétv./Eszjtv.: Hasonló rendelkezések
```

**Implementáció – OffboardingCase kapcsolat:**

```
OffboardingCase {
  id: UUID PK
  employmentId: UUID FK
  terminationDate: date (2026-06-30)
  lastWorkingDay: date (2026-06-30)

  // Szabadság-elszámolás
  unusedLeaveDays: integer (ki nem vett szabadság napok - számított)
  unusedLeavePayoutAmount: decimal (megváltás összege Ft - számított)
  unusedLeavePayoutApproved: boolean
  unusedLeavePayoutApprovedBy: UUID FK (HR vezető)
  unusedLeavePayoutApprovedAt: datetime

  // Kapcsolódó CompensationElement
  unusedLeaveCompensationId: UUID FK → CompensationElement/OneTimePayment

  status: enum (initiated, clearance, final_settlement, completed)
}
```

**Automatikus számítás (év közbeni kilépés):**

```javascript
function calculateUnusedLeaveOnTermination(employment, terminationDate) {
  const year = terminationDate.getFullYear();
  const entitlement = getLeaveEntitlement(employment, year);

  // 1. Időarányos jogosultság (ha év közben lép ki)
  const monthsWorked = calculateMonthsWorked(employment.startDate, terminationDate, year);
  const fullYearEntitlement = entitlement.baseEntitlement + entitlement.supplementaryEntitlement;
  const proportionalEntitlement = Math.round((fullYearEntitlement / 12) * monthsWorked);

  // 2. Felhasznált napok
  const usedDays = entitlement.used;

  // 3. Fennmaradó ki nem vett napok
  const unusedDays = proportionalEntitlement - usedDays;

  // 4. Ha NEGATÍV (túl sokat vett ki) → visszakövetelés
  // 5. Ha POZITÍV → megváltás

  return {
    proportionalEntitlement: proportionalEntitlement,
    usedDays: usedDays,
    unusedDays: unusedDays,
    isDebt: unusedDays < 0  // true ha tartozás
  };
}

function calculateLeavePayoutAmount(employment, unusedDays) {
  // Mt. 122. § (4): TÁVOLLÉTI DÍJ alapján
  // Távolléti díj = az előző 4 hónap átlagkeresete

  const averageWage = calculateAverageWage(employment, 4); // Előző 4 hónap átlaga
  const dailyWage = averageWage / 21; // Átlagos munkanapok száma havonta

  const payoutAmount = unusedDays * dailyWage;

  return {
    unusedDays: unusedDays,
    averageWage: averageWage,
    dailyWage: dailyWage,
    totalPayoutAmount: payoutAmount
  };
}
```

**Példa számítás:**

```
Kilépő munkavállaló:
- Jogviszony kezdete: 2025-01-01
- Kilépés: 2026-06-30 (6 hónap 2026-ban)
- Éves jogosultság: 25 nap (20 alap + 5 pót)
- Felhasznált 2026-ban: 8 nap

Számítás:
1. Időarányos jogosultság: (25 / 12) × 6 = 12,5 → 13 nap
2. Felhasznált: 8 nap
3. Fennmaradó: 13 - 8 = 5 nap megváltandó

4. Átlagkereset (előző 4 hónap): 350.000 Ft/hó
5. Napi átlagkereset: 350.000 / 21 = 16.667 Ft/nap
6. Megváltás összege: 5 nap × 16.667 Ft = 83.333 Ft

→ OneTimePayment generálása 83.333 Ft-ról
```

**Automatikus generálás workflow:**

```
1. Offboarding indítása (OffboardingCase):
   - status: initiated
   - terminationDate: 2026-06-30

2. Rendszer automatikusan számol:
   - unusedLeaveDays = 5
   - unusedLeavePayoutAmount = 83.333 Ft
   - status: clearance (elszámolás folyamatban)

3. HR áttekinti és jóváhagyja:
   - Ellenőrzi a számítást
   - unusedLeavePayoutApproved = true
   - unusedLeavePayoutApprovedBy = hr-user-001
   - unusedLeavePayoutApprovedAt = 2026-06-15

4. Automatikus CompensationElement/OneTimePayment generálás:
   createOneTimePayment({
     employmentId: employment.id,
     paymentType: 'UNUSED_LEAVE_PAYOUT',
     amount: 83333,
     description: `Ki nem vett szabadság megváltása (5 nap)`,
     legalBasis: 'Mt. 122. § (4)',
     paymentDate: terminationDate,  // Utolsó munkanapon
     status: 'APPROVED'
   });

5. Bérszámfejtés feldolgozza:
   - Utolsó bérszámfejtésben szerepel
   - Adóköteles jövedelem
   - status: PAID

6. Offboarding lezárása:
   - status: completed
```

**Manuális felülírás (kivételes esetek):**

```
OffboardingCase {
  ...
  // Ha a HR módosítja az automatikus számítást
  manualOverride: boolean
  manualUnusedLeaveDays: integer (felülbírált napok száma)
  manualPayoutAmount: decimal (felülbírált összeg)
  manualOverrideReason: text
  manualOverrideBy: UUID FK
  manualOverrideAt: datetime
}

Példa kivételes eset:
- Munkavállaló és munkáltató megállapodtak: 3 napot természetben kivesz (május),
  maradék 2 napot megváltják
- manualOverride = true
- manualUnusedLeaveDays = 2 (nem 5!)
- manualOverrideReason = "Megállapodás szerint 3 napot természetben kivett május 20-24."
```

**Jogszabályi megfelelés:**

| Jogviszony | Megváltás szabálya | Díjazás alapja |
|---|---|---|
| **Mt.** | Mt. 122. § (4) - kötelező | Távolléti díj (előző 4 hónap átlaga) |
| **Kit.** | Kit. 129. § - kötelező | Illetmény + rendszeres pótlékok |
| **Kttv.** | Kttv. 155. § - kötelező | Illetmény |
| **Púétv.** | Púétv. 92. § - kötelező | Pedagógus illetmény |
| **Eszjtv.** | Eszjtv. 10. § - kötelező | Egészségügyi dolgozó illetménye |
| **Kjt.** | Kjt. 66. § - kötelező | Illetmény |

**GDPR megfelelés:**

- **Jogalap:** GDPR 6(1)(c) - jogi kötelezettség (Mt. elszámolási kötelezettség)
- **Megőrzési idő:** Jogviszony megszűnése + 8 év (számviteli bizonylat - Szt.)

**Riportok:**

```sql
-- Folyamatban lévő kilépések, szabadság-megváltás státusza
SELECT
  p.familyName,
  p.givenNames,
  oc.terminationDate,
  oc.unusedLeaveDays,
  oc.unusedLeavePayoutAmount,
  oc.unusedLeavePayoutApproved,
  oc.status
FROM offboarding_case oc
JOIN employment e ON oc.employmentId = e.id
JOIN person p ON e.personId = p.id
WHERE oc.status IN ('clearance', 'final_settlement')
ORDER BY oc.terminationDate ASC;
```

**Ajánlás:**

1. **MVP:**
   - Automatikus számítás (unusedDays, payoutAmount)
   - Manuális jóváhagyás HR által
   - Manuális OneTimePayment létrehozás

2. **Extended (ajánlott):**
   - Automatikus számítás + automatikus CompensationElement generálás
   - HR jóváhagyás kötelező
   - Manuális felülírás lehetősége (kivételes esetek)
   - Integrálva az OffboardingCase workflow-ba

3. **Strategic:**
   - Automatikus bérszámfejtés-integráció (azonnal megjelenik az utolsó bérszámfejtésben)
   - Prediktív analitika: "Mekkora lesz a megváltási kötelezettség az év végén?"
   - Cash-flow tervezés

**Konklúzió:** Extended verzió ajánlott. Automatizálás csökkenti a hibalehetőséget, de HR jóváhagyás megmarad (compliance + audit).

---

#### 18.1.8. Koronavírus / járványügyi speciális szabályok

**Probléma:** Karantén, járványügyi szabadság, COVID-igazolás. Önálló AbsenceType vagy általános keresőképtelenség?

**Javaslat: Önálló AbsenceType (ajánlott) - rugalmasság és reporting igény miatt**

**Indokok:**

1. ✅ **Különböző díjazás:** Karantén ≠ betegszabadság ≠ keresőképtelenség
2. ✅ **Reporting igény:** Járványügyi statisztika (KSH, NEAK)
3. ✅ **Jogszabályi változékonyság:** Járvány alatt gyakran változnak a szabályok
4. ✅ **GDPR compliance:** Elkülönített kezelés (járványügyi adat vs. diagnózis)

**Megvalósítás:**

```
AbsenceType (törzsadat) bővítése:

Új kategóriák:
- QUARANTINE_ORDERED           // Hatósági karantén (fizetett)
- QUARANTINE_VOLUNTARY         // Önkéntes karantén (fizetetlen, ha nincs más jogcím)
- ISOLATION_COVID_POSITIVE     // COVID+ izoláció (betegszabadság vagy táppénz)
- VACCINATION_LEAVE            // Oltás utáni szabadnap (fizetett - Mt. 126/A. §)
- CHILD_QUARANTINE             // Gyermek karanténja miatt otthon (fizetett - Mt. 127/A. §)
- EPIDEMIC_EMERGENCY           // Járványügyi veszélyhelyzet miatti távollét
```

**LeaveType / AbsenceType példák:**

```
1. Hatósági karantén:
   LeaveType {
     code: "QUARANTINE_ORDERED"
     name: "Hatósági karantén"
     category: SPECIAL_PAID
     isPaid: true
     payRate: 1.0  (100% távolléti díj)
     payBase: ABSENCE_PAY
     deductsFromEntitlement: false
     requiresDocumentation: true  (hatósági határozat)
     legalBasis: "Kjt. 2020. évi LVIII. tv. 247. § (COVID-különleges szabály)"
   }

2. COVID-oltás utáni szabadnap:
   LeaveType {
     code: "VACCINATION_LEAVE"
     name: "Védőoltás utáni szabadnap"
     category: SPECIAL_PAID
     isPaid: true
     payRate: 1.0
     payBase: ABSENCE_PAY
     maxDaysPerOccurrence: 2  (oltásonként max 2 nap)
     requiresDocumentation: true  (oltási igazolás)
     legalBasis: "Mt. 126/A. § (2021-es módosítás)"
   }

3. Gyermek karanténja miatt:
   LeaveType {
     code: "CHILD_QUARANTINE"
     name: "Gyermek karanténja miatti távollét"
     category: SPECIAL_PAID
     isPaid: true
     payRate: 1.0
     payBase: ABSENCE_PAY
     maxDaysPerYear: 60  (gyermekenként, 12 év alatt)
     requiresDocumentation: true  (hatósági határozat)
     legalBasis: "Mt. 127/A. § (CSED/GYED jogosultaknak)"
   }
```

**AbsenceEvent példa:**

```
AbsenceEvent {
  employmentId: emp-123
  absenceType: QUARANTINE_ORDERED
  startDate: 2024-11-10
  endDate: 2024-11-17  (7 nap)

  // Járványügyi specifikus mezők
  diseaseType: "COVID-19"
  confirmedCase: true
  officialOrderNumber: "ÁNTSZ-2024-12345"  (hatósági határozat száma)
  healthAuthorityNotified: true

  // Dokumentumok
  officialDocumentId: doc-456  (hatósági határozat szkennelve)
  medicalCertificateId: null  (nem kell külön orvosi)

  // Díjazás
  paymentResponsibility: EMPLOYER
  payRate: 1.0

  status: APPROVED
}
```

**Jogszabályi háttér (2020-2023 COVID-szabályok):**

| Jogcím | Jogszabály | Díjazás | Időtartam |
|---|---|---|---|
| **Hatósági karantén** | Kjt. 2020. LVIII. tv. 247. § | 100% távolléti díj | Hatóság által elrendelt |
| **COVID-oltás utáni** | Mt. 126/A. § (2021) | 100% távolléti díj | Max 2 nap/oltás |
| **Gyermek karanténja** | Mt. 127/A. § (2020) | 100% távolléti díj | Max 60 nap/év, 12 év alatti gyermekenként |
| **Távmunka kötelezés** | Kvtv. veszélyhelyzeti szabályok | Normál bér (nem távollét) | Veszélyhelyzet ideje |

**Reporting igények:**

```sql
-- COVID-19 kapcsolatos távollétek statisztikája (2024. nov.)
SELECT
  at.name AS absence_type,
  COUNT(*) AS case_count,
  SUM(DATEDIFF(ae.endDate, ae.startDate) + 1) AS total_days,
  AVG(DATEDIFF(ae.endDate, ae.startDate) + 1) AS avg_duration
FROM absence_event ae
JOIN absence_type at ON ae.absenceTypeId = at.id
WHERE ae.diseaseType = 'COVID-19'
  AND ae.startDate >= '2024-11-01'
  AND ae.startDate < '2024-12-01'
GROUP BY at.id
ORDER BY case_count DESC;

-- Eredmény:
-- Hatósági karantén          15 eset    105 nap    7.0 nap átlag
-- COVID+ izoláció            8 eset     64 nap     8.0 nap átlag
-- Oltás utáni szabadnap      23 eset    35 nap     1.5 nap átlag
```

**GDPR megfelelés:**

⚠️ **KRITIKUS:** COVID-státusz = egészségügyi adat (GDPR 9. cikk)

```
TILOS megjeleníteni vezetőnek:
❌ "Kiss János COVID-pozitív"
❌ "Nagy Péter karanténban COVID miatt"

MEGENGEDETT:
✅ "Kiss János távol (beteg)"
✅ "Nagy Péter távol (járványügyi indok)" - ha általános kategória
✅ Statisztika szinten (aggregált, név nélkül): "15 COVID-eset novemberben"

LeaveVisibilityLevel:
- Vezető: CATEGORY szint ("beteg" / "járványügyi")
- HR/TB admin: FULL_TYPE ("COVID+ izoláció")
- Diagnózis (PCR+): SZIGORÚAN titkos (csak TB-admin)
```

**AbsenceEvent GDPR-mezők:**

```
AbsenceEvent {
  ...
  // Egészségügyi adatok (GDPR 9. cikk - különleges kategória)
  diseaseType: string (pl. "COVID-19", "Influenza", "Norovirus")
  gdprCategory: SPECIAL_CATEGORY_HEALTH
  restrictedAccess: true  (csak TB-admin, HR-vezető)

  // Hozzáférési napló
  accessLog: [
    { userId: "hr-001", accessedAt: "2024-11-11 10:23", reason: "TB-bejelentés" },
    { userId: "tb-admin-002", accessedAt: "2024-11-12 14:05", reason: "NEAK jelentés" }
  ]
}
```

**Flexibilitás jövőbeli járványokra:**

```
DiseaseType (törzsadat - opcionális Extended verzió):

DiseaseType {
  id: UUID PK
  code: string (COVID-19, INFLUENZA, MONKEYPOX, etc.)
  name: string
  category: enum (viral, bacterial, parasitic, other)

  // Különleges szabályok
  requiresOfficialQuarantine: boolean
  quarantineDurationDays: integer (default)
  healthAuthorityNotification: boolean

  // Díjazás
  defaultPayRate: decimal
  defaultPaymentResponsibility: enum (employer, social_insurance)

  isActive: boolean
  validFrom: date (járvány kezdete)
  validTo: date (járvány vége / szabályok hatályon kívül)
}

→ Ha új járvány jön (pl. H5N1 madárinfluenza 2027):
   - Új DiseaseType létrehozása
   - Új AbsenceType generálása
   - Szabályok konfigurálása
   - NINCS kódmódosítás!
```

**Alternatív megközelítés (NEM ajánlott):**

```
AbsenceType: SICK_LEAVE (általános betegszabadság)
AbsenceEvent.notes: "COVID-19 karantén"

PROBLÉMÁK:
❌ Reporting nehézkes (szöveges mező elemzése)
❌ Nem különböztethető meg hatósági karantén vs. önkéntes
❌ Díjazási szabályok nem automatizálhatók
❌ GDPR audit trail hiányos
```

**Ajánlás:**

1. **MVP (kis szervezet, alacsony járványügyi kockázat):**
   - Általános AbsenceType: EPIDEMIC_EMERGENCY
   - Dokumentáció: hatósági határozat feltöltése
   - Manuális díjazás-kezelés

2. **Extended (ajánlott - közepes/nagy szervezet):**
   - Önálló AbsenceType-ok minden járványügyi jogcímhez
   - Automatikus díjazás-számítás
   - GDPR-compliant hozzáférés-szabályozás
   - Reporting dashboard (statisztika, trendek)

3. **Strategic (multinacionális, egészségügy):**
   - DiseaseType törzsadat (dinamikus járványkezelés)
   - Automatikus NEAK/egészségügyi hatóság bejelentés
   - Kontakt-követés (ki volt kapcsolatban kivel - munkahelyi expozíció)

**Konklúzió:** Extended verzió ajánlott. A COVID-19 megmutatta, hogy rugalmas, részletezett járványügyi távollét-kezelés üzleti és compliance szükséglet.

---

### 18.2. Összefoglaló döntési mátrix

| Kérdés | Rövid válasz | Implementáció | Prioritás |
|---|---|---|---|
| **1. Szabadságterv** | LeavePlan entitás (Extended) vagy LeaveRequest.status (MVP) | LeavePlan + LeavePlanItem entitások | MVP (status), Extended (entitás) |
| **2. Helyettesítés** | LeaveRequest mezők (MVP) vagy Substitution entitás (Extended) | substituteEmploymentId + Substitution workflow | MVP (mezők), Extended (workflow) |
| **3. Csapatos nézet** | Többszintű láthatóság (CATEGORY szint ajánlott) | OrgUnit.managerLeaveVisibility konfiguráció | MVP |
| **4. TB-ellátások** | HRMS scope-ban marad, AbsenceEvent (MVP) vagy SocialBenefitClaim (Extended) | SocialBenefitClaim + NEAK API integráció | MVP (AbsenceEvent), Extended (entitás) |
| **5. Külföldi munkavégzés** | Assignment.leaveRegulation + automatikus számítás | MOST_FAVORABLE logika, országspecifikus szabályok | Extended |
| **6. Negatív egyenleg** | Konfiguráció alapján engedélyezhető | Organization.allowNegativeLeaveBalance + validációk | Extended |
| **7. Szabadság-megváltás** | Automatikus számítás + HR jóváhagyás | Automatikus OneTimePayment generálás | Extended (ajánlott) |
| **8. Járványügyi szabályok** | Önálló AbsenceType-ok | QUARANTINE, VACCINATION_LEAVE, stb. | Extended (ajánlott) |
