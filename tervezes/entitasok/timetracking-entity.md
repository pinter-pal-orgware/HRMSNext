# TimeTracking (Munkaidő-nyilvántartás) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer  
> **Verzió:** 0.1 – első tervezet  
> **Utolsó frissítés:** 2025. február  
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md, employment-entity.md, compensation-entity.md, organization-entity.md, position-entity.md, qualification-entity.md, leave-entity.md

---

## 1. Az entitás célja és hatóköre

A **TimeTracking** entitáscsalád a foglalkoztatási jogviszonyhoz (Employment) kapcsolódó munkaidő-nyilvántartást valósítja meg: a tervezett (beosztás szerinti) és a tényleges munkaidőt, a rendkívüli munkaidőt, az ügyeletet, a készenlétet és a pihenőidőket. A magyar munkajog az Mt. 134. § alapján **kötelezővé** teszi a munkaidő-nyilvántartást, amelynek tartalmát és formáját jogszabály határozza meg.

**Miért kritikus a TimeTracking entitás?**

| Felhasználási terület | Jogszabály | Hatás |
|---|---|---|---|
| **Nyilvántartási kötelezettség** | Mt. 134. § | A munkáltató köteles nyilvántartani a rendes és rendkívüli munkaidőt, készenlétet, szabadságot |
| **Rendkívüli munkaidő korlátok** | Mt. 109. § | Éves max. 250 óra (KSZ-szel max. 300 óra); napi max. +8 óra; heti max. 48 óra (átlagban) |
| **Munkaidőkeret** | Mt. 93–94. § | 4–6 hónapos keret; az egyenlőtlen beosztás elszámolása |
| **Pihenőidő garanciák** | Mt. 104–106. § | Napi 11 óra; heti 48 óra (kéthetente 40 óra) |
| **Pótlék-számítás** | Mt. 139–146. § | Éjszakai, műszak, túlóra, ügyelet, készenlét pótlékok |
| **Bérszámfejtés** | Mt.; Szja tv.; Tbj. | A munkaidő-adatokból számítódnak a pótlékok és juttatások |
| **Munkaügyi ellenőrzés** | 1996. évi LXXV. tv. | Munkaügyi hatóság ellenőrizheti a nyilvántartást; bírságalap |
| **Munkavédelem** | Mvt. | Pihenőidő betartása, éjszakai munka korlátozása |

**Tervezési alapelvek:**

- **Napi szintű nyilvántartás:** A munkaidő-nyilvántartás alapegysége a **nap**. Minden munkanapra egy `DailyTimeRecord` rekord keletkezik, amely tartalmazza a beosztás szerinti és tényleges munkaidőt, a távolléteket és a rendkívüli munkaidőt.
- **Beosztás és tény szétválasztása:** A **WorkSchedule** (munkaidő-beosztás / műszakbeosztás) és a **DailyTimeRecord** (tényleges nyilvántartás) két különálló réteg. A beosztás a terv, a nyilvántartás a tény.
- **Automatikus pótlék-kalkuláció:** A rendszer a DailyTimeRecord adatokból automatikusan számolja a pótlékra jogosító órákat (éjszakai, műszak, túlóra, ügyelet, készenlét).
- **Employment-hez kötött:** A munkaidő-nyilvántartás a jogviszonyhoz tartozik (Employment), nem a személyhez.
- **Jogszabályi korlátok betartatása:** A rendszer validálja és figyelmeztet a munkaidő-korlátok (napi, heti, éves maximum; pihenőidő minimum) megsértésére.

---

## 2. Entitásstruktúra – áttekintés

```
Employment 1 ──── 1 WorkScheduleAssignment      (munkarend hozzárendelés)
WorkScheduleAssignment ──── WorkScheduleTemplate  (munkarend sablon)

Employment 1 ──── N ShiftAssignment              (műszakbeosztás – ha egyenlőtlen munkarend)
ShiftAssignment ──── ShiftTemplate                (műszak sablon)

Employment 1 ──── N DailyTimeRecord               (napi munkaidő-nyilvántartás)
Employment 1 ──── N OvertimeRecord                 (rendkívüli munkaidő nyilvántartás)
Employment 1 ──── N OnCallRecord                   (ügyelet / készenlét nyilvántartás)
Employment 1 ──── N TimeFramePeriod                (munkaidőkeret periódusok)

DailyTimeRecord ──── LeaveRecord                   (távollét-összekapcsolás a Leave entitásból)
```

---

## 3. WorkScheduleTemplate (Munkarend sablon – törzsadat)

### 3.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `organizationId` | UUID (FK) | igen | Melyik szervezethez tartozik a sablon | Technikai |
| `code` | string | igen | Munkarend kódja | Belső szabályzat |
| `name` | string | igen | Megnevezés (pl. „Általános nappali 8–16:30", „Kétműszakos 6–14/14–22", „Kötetlen") | – |
| `scheduleType` | enum | igen | Munkarend típusa (lásd 3.2.) | Mt. 97. § |
| `isFlexible` | boolean | igen | Rugalmas munkaidő-e (törzsidő + peremidő) | Mt. 96. § (2) |
| `coreTimeStart` | time | nem | Törzsidő kezdete (rugalmas munkarendnél) | – |
| `coreTimeEnd` | time | nem | Törzsidő vége | – |
| `flexTimeStart` | time | nem | Peremidő legkorábbi kezdete | – |
| `flexTimeEnd` | time | nem | Peremidő legkésőbbi vége | – |
| `defaultStartTime` | time | nem | Alapértelmezett munkakezdés | – |
| `defaultEndTime` | time | nem | Alapértelmezett munkavégzés | – |
| `defaultBreakMinutes` | integer | nem | Alapértelmezett munkaközi szünet (percben) | Mt. 103. § |
| `dailyHours` | decimal | igen | Napi munkaidő (órában) | Mt. 92. § |
| `weeklyHours` | decimal | igen | Heti munkaidő (órában) | Mt. 92. § |
| `workDays` | array[enum] | nem | Munkaszüneti napok beosztása (pl. [MON, TUE, WED, THU, FRI]) | – |
| `applicableEmploymentTypes` | array[enum] | nem | Mely jogviszony-típusoknál használható | – |
| `isActive` | boolean | igen | Aktív-e | – |

### 3.2. Munkarend típusok (scheduleType)

| Kód | Megnevezés | Leírás | Jellemző jellemzők | Jogszabály |
|---|---|---|---|---|
| `GENERAL` | Általános munkarend | Hétfő–péntek, fix munkaidő (pl. 8:00–16:30) | Napi 8 óra, heti 40 óra | Mt. 97. § (1) |
| `IRREGULAR` | Egyenlőtlen munkaidő-beosztás | A heti munkaidő egyenlőtlenül oszlik el a munkanapok között | Munkaidőkeretben elszámolva | Mt. 97. § (2) |
| `SHIFT_2` | Kétműszakos | Két műszak váltakozása (pl. délelőtti + délutáni) | Műszakpótlék | Mt. 97. § |
| `SHIFT_3` | Háromműszakos | Három műszak váltakozása (délelőtti + délutáni + éjszakai) | Műszak + éjszakai pótlék | Mt. 97. § |
| `SHIFT_CONTINUOUS` | Folyamatos (megszakítás nélküli) | 0–24, hét minden napja (pl. kórház, tűzoltóság) | Munkaszüneti napi pótlék is | Mt. 97. § (4) |
| `UNBOUND` | Kötetlen munkarend | A munkavállaló maga osztja be a munkaidejét | Nem jár pótlék (Mt. 96. §) | Mt. 96. § |
| `SPLIT` | Osztott munkaidő | A napi munkaidő két részre osztott (pl. reggeli + esti) | Speciális szünetek | Mt. 99. § (7) |
| `PEDAGOGICAL` | Pedagógus munkarend | Kötött + kötetlen rész; tanítási óraszám + felkészülés | Púétv. 88. § | Púétv. 88. § |
| `ON_CALL_BASED` | Ügyeleti alapú | Ügyeleti beosztás szerint (pl. orvos) | Ügyeleti díj | Eszjtv. |

---

## 4. ShiftTemplate (Műszak sablon – törzsadat)

### 4.1. Property-k

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `id` | UUID | igen | PK |
| `organizationId` | UUID (FK) | igen | Szervezet |
| `code` | string | igen | Műszak kód (pl. „D" = délelőtti, „É" = éjszakai) |
| `name` | string | igen | Megnevezés (pl. „Délelőtti műszak 6:00–14:00") |
| `startTime` | time | igen | Műszak kezdete |
| `endTime` | time | igen | Műszak vége |
| `breakMinutes` | integer | igen | Munkaközi szünet (percben) |
| `netHours` | decimal | számított | Nettó munkaidő (szünet nélkül) |
| `grossHours` | decimal | számított | Bruttó munkaidő (szünettel) |
| `isNightShift` | boolean | számított | Éjszakai műszaknak minősül-e (Mt. 89. §: 22:00–06:00 között legalább 1 óra) |
| `isAfternoonShift` | boolean | számított | Délutáni műszaknak minősül-e (Mt. 141. §) |
| `nightHours` | decimal | számított | Éjszakai időszakra eső órák száma (22:00–06:00) |
| `color` | string | nem | Megjelenítési szín (naptárban, beosztásban) |
| `isActive` | boolean | igen | Aktív-e |

---

## 5. WorkScheduleAssignment (Munkarend hozzárendelés)

### 5.1. Property-k

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `id` | UUID | igen | PK |
| `employmentId` | UUID (FK) | igen | Hivatkozás az Employment-re |
| `workScheduleTemplateId` | UUID (FK) | igen | Hivatkozás a munkarend sablonra |
| `validFrom` | date | igen | Érvényesség kezdete |
| `validTo` | date | nem | Érvényesség vége (null = aktuális) |
| `customStartTime` | time | nem | Egyéni munkakezdés (ha eltér a sablontól) |
| `customEndTime` | time | nem | Egyéni munkavégzés |
| `customDailyHours` | decimal | nem | Egyéni napi munkaidő (részmunkaidő) |
| `customWeeklyHours` | decimal | nem | Egyéni heti munkaidő |
| `createdAt` | datetime | igen | Létrehozás |
| `createdBy` | string | igen | Létrehozó |
| `updatedAt` | datetime | igen | Módosítás |
| `updatedBy` | string | igen | Módosító |

---

## 6. ShiftAssignment (Műszakbeosztás)

Az egyenlőtlen munkarendű és műszakos dolgozók konkrét napi beosztása.

### 6.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `employmentId` | UUID (FK) | igen | Hivatkozás az Employment-re | Technikai |
| `date` | date | igen | Beosztás napja | – |
| `shiftTemplateId` | UUID (FK) | nem | Hivatkozás a műszak sablonra (ha sablon-alapú) | – |
| `plannedStartTime` | time | igen | Tervezett műszakkezdés | – |
| `plannedEndTime` | time | igen | Tervezett műszakvégzés | – |
| `plannedBreakMinutes` | integer | igen | Tervezett szünet | Mt. 103. § |
| `plannedNetHours` | decimal | számított | Tervezett nettó munkaidő | – |
| `isWorkDay` | boolean | igen | Beosztás szerinti munkanap-e (false = pihenőnap) | – |
| `isPublicHolidayWork` | boolean | számított | Munkaszüneti napon történő munkavégzés-e | Mt. 102. § |
| `status` | enum | igen | `PLANNED`, `PUBLISHED`, `MODIFIED`, `CANCELLED` | – |
| `publishedAt` | datetime | nem | Beosztás közlésének időpontja | Mt. 97. § (4) – legalább 7 nappal korábban |
| `publishedBy` | string | nem | Közlő személy | – |
| `modificationReason` | string | nem | Módosítás indoka (ha MODIFIED) | Mt. 97. § (5) – rendkívüli ok |
| `createdAt` | datetime | igen | Létrehozás | – |
| `createdBy` | string | igen | Létrehozó | – |
| `updatedAt` | datetime | igen | Módosítás | – |
| `updatedBy` | string | igen | Módosító | – |

**Megjegyzések:**
- Az Mt. 97. § (4) szerint a munkaidő-beosztást **legalább 7 nappal korábban, legalább egy hétre** előre írásban kell közölni. A rendszernek ezt a `publishedAt` mezővel kell dokumentálnia.
- A beosztás módosítására az Mt. 97. § (5) alapján csak **rendkívüli esetben** kerülhet sor – a `modificationReason` kötelezővé válik módosításkor.

---

## 7. DailyTimeRecord (Napi munkaidő-nyilvántartás)

Ez az entitás az Mt. 134. § szerinti **kötelező munkaidő-nyilvántartás** alapegysége.

### 7.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `employmentId` | UUID (FK) | igen | Hivatkozás az Employment-re | Technikai |
| `date` | date | igen | Nyilvántartott nap | Mt. 134. § |
| `dayType` | enum | igen | Nap típusa: `WORKDAY`, `REST_DAY`, `PUBLIC_HOLIDAY`, `TRANSFERRED_WORKDAY`, `TRANSFERRED_REST_DAY` | WorkCalendar-ból |
| `shiftAssignmentId` | UUID (FK) | nem | Hivatkozás a műszakbeosztásra (ha van) | – |
| `leaveRecordId` | UUID (FK) | nem | Hivatkozás a távolléti rekordra (ha távollét) | Leave entitás |

#### 7.1.1. Beosztás szerinti munkaidő

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `scheduledStartTime` | time | nem | Beosztás szerinti munkakezdés |
| `scheduledEndTime` | time | nem | Beosztás szerinti munkavégzés |
| `scheduledBreakMinutes` | integer | nem | Beosztás szerinti szünet |
| `scheduledNetHours` | decimal | nem | Beosztás szerinti nettó munkaidő |

#### 7.1.2. Tényleges munkaidő

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `actualStartTime` | time | nem | Tényleges munkakezdés | Mt. 134. § |
| `actualEndTime` | time | nem | Tényleges munkavégzés | Mt. 134. § |
| `actualBreakMinutes` | integer | nem | Tényleges szünet | Mt. 103. § |
| `actualNetHours` | decimal | számított | Tényleges nettó munkaidő (actualEnd – actualStart – break) | Mt. 134. § |

#### 7.1.3. Munkaidő-elemzés (számított / pótlék-kalkulációhoz)

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `regularHours` | decimal | számított | Rendes munkaidő (beosztás szerinti) | – |
| `overtimeHours` | decimal | számított | Rendkívüli munkaidő (tényleges – beosztás szerinti, ha pozitív) | Mt. 107. § |
| `nightHours` | decimal | számított | Éjszakai időszakra eső órák (22:00–06:00 között) | Mt. 89. § |
| `afternoonShiftHours` | decimal | számított | Délutáni műszakban ledolgozott órák | Mt. 141. § |
| `nightShiftHours` | decimal | számított | Éjszakai műszakban ledolgozott órák | Mt. 141–142. § |
| `publicHolidayHours` | decimal | számított | Munkaszüneti napon ledolgozott órák | Mt. 143. § (5) |
| `restDayHours` | decimal | számított | Pihenőnapon ledolgozott órák | Mt. 143. § |
| `standbyHours` | decimal | nem | Készenléti órák (nem aktív munkavégzés) | Mt. 110. § |
| `onCallHours` | decimal | nem | Ügyeleti órák | Mt. 110. § |
| `onCallActiveHours` | decimal | nem | Ügyeleten belüli tényleges munkavégzés | Eszjtv.; Mt. |
| `idleHours` | decimal | nem | Állásidő (munkáltató működési körébe eső ok) | Mt. 146. § |

#### 7.1.4. Távollét

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `absenceType` | enum | nem | Távollét típusa (ha a nap nem munkavégzés) – a LeaveRecord-ból |
| `absenceHours` | decimal | nem | Távollét óraszáma |
| `isFullDayAbsence` | boolean | nem | Egész napos távollét-e |

#### 7.1.5. Státusz és audit

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `status` | enum | igen | `OPEN` (rögzítendő), `DRAFT`, `SUBMITTED`, `APPROVED`, `LOCKED` |
| `approvedBy` | string | nem | Jóváhagyó |
| `approvedAt` | datetime | nem | Jóváhagyás időpontja |
| `lockedAt` | datetime | nem | Zárolás időpontja (bérszámfejtés után) |
| `notes` | string | nem | Megjegyzés |
| `createdAt` | datetime | igen | Létrehozás |
| `createdBy` | string | igen | Létrehozó |
| `updatedAt` | datetime | igen | Módosítás |
| `updatedBy` | string | igen | Módosító |

---

## 8. OvertimeRecord (Rendkívüli munkaidő nyilvántartás)

A rendkívüli munkaidő (túlóra) kiemelt jogszabályi figyelmet kap, ezért érdemes önálló nyilvántartást vezetni, bár a DailyTimeRecord-ból is levezethető.

### 8.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `employmentId` | UUID (FK) | igen | Hivatkozás az Employment-re | Technikai |
| `dailyTimeRecordId` | UUID (FK) | igen | Hivatkozás a napi nyilvántartásra | Technikai |
| `date` | date | igen | Dátum | – |
| `overtimeType` | enum | igen | Rendkívüli munkaidő típusa (lásd 8.2.) | Mt. 107–109. § |
| `hours` | decimal | igen | Túlóra óraszám | – |
| `reason` | string | nem | Elrendelés indoka | Mt. 107. § – a munkáltató rendelheti el |
| `orderedBy` | string | nem | Elrendelő személy | – |
| `isEmployeeInitiated` | boolean | nem | Munkavállaló kezdeményezte-e (önként vállalt túlóra) | Mt. 109. § (2) |
| `compensationType` | enum | igen | Ellentételezés módja: `PAY` (pótlékos díjazás), `TIME_OFF` (szabadidő), `COMBINED` | Mt. 143. § |
| `compensationCompleted` | boolean | nem | Az ellentételezés megtörtént-e | – |
| `timeOffGrantedDate` | date | nem | Szabadidő kiadásának dátuma (ha TIME_OFF) | Mt. 143. § (6) – a túlmunka ellentételezése |
| `supplementRate` | decimal | számított | Alkalmazandó pótlékszázalék | Mt. 143. § |
| `createdAt` | datetime | igen | Létrehozás | – |
| `createdBy` | string | igen | Létrehozó | – |
| `updatedAt` | datetime | igen | Módosítás | – |
| `updatedBy` | string | igen | Módosító | – |

### 8.2. Rendkívüli munkaidő típusok (overtimeType)

| Kód | Megnevezés | Pótlék mértéke | Jogszabály |
|---|---|---|---|
| `OVERTIME_WORKDAY` | Munkanapon elrendelt túlóra | 50% | Mt. 143. § (2) |
| `OVERTIME_REST_DAY` | Pihenőnapon (heti pihenőnap) elrendelt | 100% (vagy 50% + szabadnap) | Mt. 143. § (3)–(4) |
| `OVERTIME_PUBLIC_HOLIDAY` | Munkaszüneti napon elrendelt | 100% | Mt. 143. § (5) |
| `OVERTIME_VOLUNTARY` | Önként vállalt túlóra (Mt. 109. § (2)) | Megegyezés, de min. 50% | Mt. 109. § (2) |
| `OVERTIME_FRAME_EXCESS` | Munkaidőkereten felüli többlet (a keret végén számított) | 50% vagy 100% (a túllépés napjától függ) | Mt. 143. § |

**Megjegyzések:**
- A rendkívüli munkaidő **éves korlátja** 250 óra (kollektív szerződéssel max. 300 óra). Önként vállalt túlóra esetén további max. 150 óra. Mt. 109. § (1)–(2).
- A pihenőnapon elrendelt munkavégzés esetén a munkáltató választhat: 100% pótlékot fizet, VAGY 50% pótlék + másik pihenőnapot ad. Mt. 143. § (4).
- A kötetlen munkarendben (Mt. 96. §) a rendkívüli munkaidő szabályai **nem alkalmazandók** – nincs túlórapótlék.

---

## 9. OnCallRecord (Ügyelet / Készenlét nyilvántartás)

### 9.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `employmentId` | UUID (FK) | igen | Hivatkozás az Employment-re | Technikai |
| `date` | date | igen | Dátum | – |
| `onCallType` | enum | igen | Típus: `ON_CALL` (ügyelet – munkahelyen), `STANDBY` (készenlét – munkavégzési helyen kívül) | Mt. 110–112. § |
| `startTime` | time | igen | Ügyelet/készenlét kezdete | – |
| `endTime` | time | igen | Vége | – |
| `totalHours` | decimal | számított | Összes óra | – |
| `activeWorkHours` | decimal | nem | Tényleges munkavégzés órái (ügyeleten belül) | Eszjtv. – az aktív ügyeleti idő külön elszámolás |
| `passiveHours` | decimal | számított | Passzív (várakozási) idő | – |
| `isVoluntary` | boolean | nem | Önként vállalt többletmunka keretében-e (Eszjtv.) | Eszjtv. – önként vállalt többletmunka |
| `compensationType` | enum | igen | Ellentételezés módja: `PAY`, `TIME_OFF`, `COMBINED` | Mt. 144. § |
| `payRate` | decimal | számított | Díjazás mértéke (ügyelet: 40%; készenlét: 20%) | Mt. 144. § |
| `eszjtv_onCallCategory` | enum | nem | Eszjtv. szerinti ügyeleti kategória (ha eü. jogviszony) | 528/2020. Korm. r.; 3/2023. OKFŐ utasítás |
| `notes` | string | nem | Megjegyzés | – |
| `createdAt` | datetime | igen | Létrehozás | – |
| `createdBy` | string | igen | Létrehozó | – |
| `updatedAt` | datetime | igen | Módosítás | – |
| `updatedBy` | string | igen | Módosító | – |

**Megjegyzések:**
- **Ügyelet** (Mt. 110. §): a munkavállaló a munkáltató által meghatározott helyen tartózkodik a beosztás szerinti munkaidején kívül. Díjazás: az alapbér 40%-a.
- **Készenlét** (Mt. 110. §): a munkavállaló a munkáltató által meghatározott időben elérhető, de a tartózkodási helyét maga határozza meg. Díjazás: az alapbér 20%-a.
- Az **Eszjtv.** szerinti ügyeleti rendszer ennél jelentősen bonyolultabb: a beosztás szerinti munkaidő keretében végzett ügyelet és az önként vállalt többletmunka keretében végzett ügyelet eltérő díjazási szabályok alá esik (3/2023. OKFŐ utasítás 7–12. pont).
- A havi ügyeleti/készenléti órák összesítése a Compensation entitás felé megy (pótlék-számítás input).

---

## 10. TimeFramePeriod (Munkaidőkeret periódus)

### 10.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `employmentId` | UUID (FK) | igen | Hivatkozás az Employment-re | Technikai |
| `startDate` | date | igen | Keret kezdete | Mt. 93. § |
| `endDate` | date | igen | Keret vége | Mt. 93. § |
| `durationMonths` | integer | számított | Keret hossza (hónapban) | Mt. 94. § – max. 4 hó; KSZ-szel max. 6 hó |
| `totalWorkDays` | integer | számított | A keretbe eső munkanapok száma (WorkCalendar alapján) | – |
| `totalScheduledHours` | decimal | számított | A keretben beosztandó összes munkaidő (totalWorkDays × dailyHours) | – |
| `totalActualHours` | decimal | számított | A keretben ténylegesen ledolgozott órák (DailyTimeRecord-okból) | – |
| `overtimeHours` | decimal | számított | Keret végén kalkulált rendkívüli munkaidő (totalActual – totalScheduled, ha pozitív) | Mt. 143. § |
| `underTimeHours` | decimal | számított | Keret végén kalkulált alulteljesítés (totalScheduled – totalActual, ha pozitív) | – |
| `status` | enum | igen | `OPEN` (folyamatban), `CLOSED` (lezárt), `SETTLED` (elszámolt) | – |
| `settledAt` | datetime | nem | Elszámolás időpontja | – |
| `settledBy` | string | nem | Elszámolást végző | – |
| `createdAt` | datetime | igen | Létrehozás | – |
| `createdBy` | string | igen | Létrehozó | – |

**Megjegyzések:**
- A **munkaidőkeret** (Mt. 93–94. §) lehetővé teszi, hogy a munkáltató a munkaidőt egyenlőtlenül ossza be: egyes napokon több, más napokon kevesebb munkaidővel. A rendkívüli munkaidő **a keret végén** számítandó, nem naponta.
- A keret maximális hossza **4 hónap** (kollektív szerződéssel **6 hónap**; idénymunkánál, megszakítás nélküli üzemnél **12 hónap**). Mt. 94. §.
- A keret lezárásakor a `totalActualHours` – `totalScheduledHours` pozitív különbözete rendkívüli munkaidőnek minősül → OvertimeRecord keletkezik `OVERTIME_FRAME_EXCESS` típussal.
- A negatív különbözet (alulteljesítés) az **állásidő** szabályai szerint kezelendő, ha a munkáltató nem biztosított elegendő munkát (Mt. 146. §).

---

## 11. Munkaidő-korlátok és pihenőidő szabályok

### 11.1. Mt. munkaidő-korlátok

| Korlát | Érték | Jogszabály | HRMS validáció |
|---|---|---|---|
| Napi munkaidő maximum | 12 óra (munkaidőkeretben) | Mt. 99. § (2) | Figyelmeztetés, ha `actualNetHours` > 12 |
| Heti munkaidő maximum | 48 óra (rendkívüli munkaidővel együtt, referencia-időszak átlagában) | Mt. 99. § (3); EU 2003/88/EK irányelv | Figyelmeztetés a heti átlag kiszámítása alapján |
| Rendkívüli munkaidő – éves max. | 250 óra (KSZ-szel 300 óra) | Mt. 109. § (1) | Kumulatív számláló; figyelmeztetés 200/240 óránál |
| Önként vállalt túlóra – éves max. | További 150 óra (az éves kereten felül) | Mt. 109. § (2) | Külön számláló |
| Napi pihenőidő | Min. 11 óra két munkanap között (egyenlőtlen beosztásnál min. 8 óra) | Mt. 104. § | Figyelmeztetés, ha az előző napi `actualEndTime` és a következő napi `actualStartTime` között < 11 óra |
| Heti pihenőidő | Min. 48 óra összefüggő pihenőidő hetente (vagy kéthetente min. 40 óra) | Mt. 105–106. § | Figyelmeztetés |
| Éjszakai munkavégzés korlátozás | Napi max. 8 óra (referencia-időszak átlagában) | Mt. 99. § (4) | Figyelmeztetés |

### 11.2. Személyi korlátozások

| Korlátozás | Érintett személyek | Jogszabály | Forrás |
|---|---|---|---|
| Éjszakai munka tilalom | Várandós nő (a megállapítástól); kisgyermekes szülő (3 éves korig) – kivéve, ha hozzájárul | Mt. 113. § (2) | Person.dependents; AbsenceEvent (várandósság) |
| Rendkívüli munkaidő korlátozás | Várandós nő; kisgyermekes szülő (3 éves korig); egyedülálló szülő (gyermek 3 éves koráig) | Mt. 113. § (3) | Person.dependents |
| Készenlét korlátozás | Várandós nő – teljes tilalom | Mt. 113. § (2) | – |
| Fiatal munkavállaló | Napi max. 8 óra; éjszakai munka és rendkívüli munkaidő tilalom; min. 12 óra pihenőidő | Mt. 114. § | Person.birthDate |

---

## 12. Pótlékszámítás – a TimeTracking és Compensation közötti interfész

A DailyTimeRecord számított mezőiből a Compensation modul pótlékot számít. Az összekötés logikája:

### 12.1. Havi összesítés a pótlék-számításhoz

| Pótlék típus | Input (DailyTimeRecord-ból) | Compensation elem | Számítás | Jogszabály |
|---|---|---|---|---|
| Éjszakai pótlék | `nightHours` összesítés (havi) | `MT_NIGHT_SUPPLEMENT` | nightHours × (alapbér/174) × 15% | Mt. 142. § |
| Délutáni műszakpótlék | `afternoonShiftHours` összesítés | `MT_SHIFT_SUPPLEMENT` | hours × (alapbér/174) × 30% | Mt. 141. § |
| Éjszakai műszakpótlék | `nightShiftHours` összesítés | `MT_SHIFT_SUPPLEMENT` | hours × (alapbér/174) × 15% | Mt. 141. § |
| Túlóra – munkanap | `OVERTIME_WORKDAY` hours összesítés | `MT_OVERTIME_SUPPLEMENT` | hours × (alapbér/174) × 50% | Mt. 143. § (2) |
| Túlóra – pihenőnap | `OVERTIME_REST_DAY` hours összesítés | `MT_OVERTIME_SUPPLEMENT` | hours × (alapbér/174) × 100% | Mt. 143. § (3) |
| Túlóra – munkaszüneti nap | `OVERTIME_PUBLIC_HOLIDAY` hours összesítés | `MT_HOLIDAY_SUPPLEMENT` | hours × (alapbér/174) × 100% | Mt. 143. § (5) |
| Ügyeleti díj | `onCallHours` összesítés | `MT_ON_CALL_PAY` | hours × (alapbér/174) × 40% | Mt. 144. § |
| Készenléti díj | `standbyHours` összesítés | `MT_STANDBY_PAY` | hours × (alapbér/174) × 20% | Mt. 144. § |
| Állásidő díjazás | `idleHours` összesítés | (alapbér) | hours × (alapbér/174) × 100% | Mt. 146. § (1) |

**Megjegyzések:**
- A 174-es osztó a havi átlagos munkaórák száma (éves 2088 / 12 = 174).
- Az Eszjtv. szerinti ügyeleti díjszámítás ennél összetettebb (lásd compensation-entity.md 5.4. fejezet).
- A kötetlen munkarendben (Mt. 96. §) az éjszakai, műszak- és túlórapótlék **nem jár**.

---

## 13. Pedagógus munkaidő sajátosságai (Púétv.)

A pedagógus munkaideje eltér a hagyományos munkaidő-nyilvántartástól:

| Időkategória | Leírás | Nyilvántartás módja | Jogszabály |
|---|---|---|---|
| **Kötött munkaidő – tanítási óra** | Heti kötelező tanítási óraszám (munkakörtől függ: óvónő 32, tanár 22–26) | Órarend + tényleges megtartás | Púétv. 88. § (3) |
| **Kötött munkaidő – nem tanítási** | Neveléssel-oktatással összefüggő, de nem tanóra (fogadóóra, felügyelet, értekezlet) | A munkáltató határozza meg | Púétv. 88. § (3) |
| **Kötetlen munkaidő** | Felkészülés, értékelés, önképzés – a pedagógus maga osztja be | Nem kell nyilvántartani (a pedagógus diszkrecionális) | Púétv. 88. § (3) |

### 13.1. Pedagógus napi nyilvántartás – kiegészítő mezők

| Property | Típus | Leírás |
|---|---|---|
| `puetv_teachingHours` | decimal | Az adott napon megtartott tanítási órák száma |
| `puetv_boundNonTeachingHours` | decimal | Kötött, de nem tanítási idő |
| `puetv_unboundHours` | decimal | Kötetlen munkaidő (tájékoztató, nem kötelező rögzítés) |
| `puetv_substitutionHours` | decimal | Helyettesítési órák (többlettanítási óradíj alapja) |

---

## 14. Egészségügyi ügyelet sajátosságai (Eszjtv.)

| Ügyeleti forma | Leírás | Díjazás | Jogszabály |
|---|---|---|---|
| Beosztás szerinti ügyelet | A havi beosztás részeként, a havi keret terhére | Ügyeleti óradíj | 528/2020. Korm. r.; 3/2023. OKFŐ utasítás 7–8. pont |
| Önként vállalt többletmunka – ügyelet | Az Eszjtv. szerinti hozzájáruló nyilatkozat alapján, a havi kereten felül | Ügyeleti óradíj + 20% | 3/2023. OKFŐ utasítás 11. pont |
| Beosztás szerinti készenlét | A havi beosztás részeként | Készenléti díj | Eszjtv. |
| Ügyeleti aktív / passzív idő | Az ügyeleten belüli tényleges munkavégzés vs. várakozás | Az aktív idő pótlékszámítás szempontjából eltérhet | 3/2023. OKFŐ utasítás |

### 14.1. Eszjtv. ügyeleti kiegészítő mezők (OnCallRecord-on)

| Property | Típus | Leírás |
|---|---|---|
| `eszjtv_onCallCategory` | enum | Ügyeleti kategória (beosztás szerinti / önként vállalt többletmunka) |
| `eszjtv_monthlyOnCallQuota` | decimal | Havi ügyeleti keret (órában) |
| `eszjtv_voluntaryConsentDate` | date | Önként vállalt többletmunka hozzájáruló nyilatkozat dátuma |
| `eszjtv_voluntaryConsentValidUntil` | date | Nyilatkozat érvényessége |

---

## 15. Üzleti szabályok és validációk

| Szabály | Leírás | Jogszabály | Súlyosság |
|---|---|---|---|
| Napi munkaidő max. 12 óra | `actualNetHours` ≤ 12 (munkaidőkeretben) | Mt. 99. § (2) | Hiba |
| Napi pihenőidő min. 11 óra | Két egymást követő munkanap között min. 11 óra (egyenlőtlen: min. 8 óra) | Mt. 104. § | Figyelmeztetés |
| Heti pihenőidő min. 48 óra | Hetente min. 48 óra összefüggő (kéthetente min. 40 óra) | Mt. 105–106. § | Figyelmeztetés |
| Éves rendkívüli munkaidő max. 250 óra | Kumulatív OvertimeRecord.hours ≤ 250 (KSZ: 300) | Mt. 109. § (1) | Hiba 250 felett / Figyelmeztetés 200-nál |
| Önként vállalt túlóra max. 150 óra | Kumulatív (isEmployeeInitiated = true) hours ≤ 150 | Mt. 109. § (2) | Hiba |
| Munkaidőkeret max. hossz | `durationMonths` ≤ 4 (KSZ: 6; idénymunka/megszakítás nélküli: 12) | Mt. 94. § | Hiba |
| Beosztás közlési határidő | `ShiftAssignment.publishedAt` ≥ beosztott nap – 7 nap | Mt. 97. § (4) | Figyelmeztetés |
| Munkaközi szünet kötelező | 6 óra felett min. 20 perc; 9 óra felett min. 25 perc szünet | Mt. 103. § (1) | Figyelmeztetés |
| Éjszakai munka tilalom – várandós | Ha Person védelmi státusz `PREGNANCY` → éjszakai beosztás nem engedélyezhető | Mt. 113. § (2) | Hiba |
| Éjszakai munka tilalom – kisgyermekes | Ha Person.dependents-ben 3 év alatti gyermek → éjszakai beosztás csak hozzájárulással | Mt. 113. § (2) | Figyelmeztetés |
| Fiatal munkavállaló korlátok | < 18 év: max. napi 8 óra; éjszakai és rendkívüli tilalom; min. 12 óra pihenőidő | Mt. 114. § | Hiba |
| Zárolás bérszámfejtés után | `LOCKED` státuszú DailyTimeRecord nem módosítható (csak korrekciós folyamattal) | Belső szabályzat | Hiba |

---

## 16. Automatikus események és figyelmeztetések

| Esemény | Kiváltó feltétel | Akció |
|---|---|---|
| **Rendkívüli munkaidő kumulatív figyelmeztetés** | Éves kumulatív OvertimeRecord eléri a 200 / 240 / 250 órát | Figyelmeztetés HR-nek és vezetőnek |
| **Pihenőidő megsértés** | DailyTimeRecord-ok közötti pihenőidő < 11 óra (vagy 8 óra egyenlőtlen beosztásnál) | Azonnali figyelmeztetés |
| **Heti munkaidő-túllépés** | Heti `actualNetHours` + `overtimeHours` > 48 (referencia-időszak átlagában) | Figyelmeztetés |
| **Munkaidőkeret lejárta** | TimeFramePeriod.endDate elérése | Keret-elszámolás indítása; OvertimeRecord generálás |
| **Beosztás közzétételi határidő** | ShiftAssignment dátuma – 7 nap, és `publishedAt` még null | Figyelmeztetés: a beosztás közlése esedékes |
| **Zárolási határidő** | Bérszámfejtési ciklus zárása → DailyTimeRecord-ok zárolása | Automatikus státuszváltás: APPROVED → LOCKED |
| **Éjszakai munka – várandós/kisgyermekes** | ShiftAssignment.isNightShift = true és Employment védelmi státusz aktív | Beosztás létrehozásakor figyelmeztetés/tiltás |
| **Hiányzó napi nyilvántartás** | Munkanap (WorkCalendar) + aktív Employment, de nincs DailyTimeRecord | Figyelmeztetés (heti szinten) |
| **Igazolatlan távollét** | DailyTimeRecord: nincs actualStartTime, nincs leaveRecordId, munkanap | Azonnali riasztás |

---

## 17. GDPR adatkezelési kategorizáció

| Kategória | Property-k | Jogalap | Megőrzési idő |
|---|---|---|---|
| **Munkaidő-nyilvántartás** | DailyTimeRecord (összes) | Jogi kötelezettség (6(1)c) – Mt. 134. § | Jogviszony megszűnése + 3 év (munkaügyi elévülés) |
| **Munkaidő-beosztás** | ShiftAssignment (összes) | Jogi kötelezettség (6(1)c) – Mt. 97. § | Jogviszony megszűnése + 3 év |
| **Rendkívüli munkaidő** | OvertimeRecord (összes) | Jogi kötelezettség (6(1)c) – Mt. 134. § | Jogviszony megszűnése + 3 év |
| **Ügyelet / készenlét** | OnCallRecord (összes) | Jogi kötelezettség (6(1)c) – Mt. 134. §; Eszjtv. | Jogviszony megszűnése + 3 év (eü.: + 5 év) |
| **Munkaidőkeret** | TimeFramePeriod (összes) | Jogi kötelezettség (6(1)c) – Mt. 93–94. § | Jogviszony megszűnése + 3 év |
| **Jóváhagyási adatok** | approvedBy, approvedAt | Elszámoltathatóság (GDPR 5(2)) | Az érintett nyilvántartás megőrzési idejével megegyező |

**Megjegyzések:**
- A **3 éves** megőrzési idő a munkaügyi per (Mt. 287. §) elévülési idejéből fakad.
- A munkaidő-nyilvántartás a **munkaügyi ellenőrzés** (1996. évi LXXV. tv.) tárgyát képezi: a munkaügyi hatóság jogosult megtekinteni, másolatot kérni. A rendszernek képesnek kell lennie a hatósági adatszolgáltatásra.
- A **tényleges belépés/kilépés időpontja** (ha beléptető rendszer integráció van) biometrikus adat lehet (GDPR 9. cikk) → külön jogalap szükséges.

---

## 18. Hozzáférési szintek

| Szerep | WorkSchedule / ShiftAssignment | DailyTimeRecord | OvertimeRecord | OnCallRecord |
|---|---|---|---|---|
| **Érintett munkavállaló** | Saját beosztás olvasás | Saját: olvasás, rögzítés (ha manuális) | Saját: olvasás | Saját: olvasás |
| **Közvetlen vezető** | Beosztottak: beosztás készítés/módosítás | Beosztottak: olvasás, jóváhagyás | Beosztottak: elrendelés, olvasás | Beosztottak: beosztás, olvasás |
| **HR ügyintéző** | Teljes olvasás/írás a hatáskörben | Teljes olvasás/írás | Teljes olvasás/írás | Teljes olvasás/írás |
| **Bérszámfejtő** | Olvasás | Olvasás (jóváhagyott/zárolt) | Olvasás | Olvasás |
| **Munkavédelmi felelős** | Olvasás (éjszakai munka, pihenőidő) | Olvasás (pihenőidő megsértés) | Olvasás (korlátok figyelés) | Olvasás |
| **Rendszeradminisztrátor** | Törzsadat karbantartás (WorkScheduleTemplate, ShiftTemplate) | – | – | – |

---

## 19. Integrációs pontok

| Külső rendszer | Irány | Érintett adatok | Cél |
|---|---|---|---|
| **Beléptető rendszer (access control)** | bejövő | Belépés/kilépés időpontja → actualStartTime/actualEndTime | Automatikus munkaidő-rögzítés |
| **Bérszámfejtő modul (payroll)** | kimenő | Havi összesítés: rendes órák, túlóra órák típusonként, ügyeleti/készenléti órák, éjszakai/műszak órák, állásidő, távollét-típusok | Pótlékszámítás; távolléti díj |
| **Leave modul** | kétirányú | LeaveRecord → DailyTimeRecord.absenceType, absenceHours | Távollét megjelenítése a napi nyilvántartásban |
| **NAV (munkaügyi ellenőrzés)** | kimenő | Munkaidő-nyilvántartás exportálása (hatósági kérésre) | Mt. 134. § |
| **Munkavédelmi rendszer** | kimenő | Éjszakai/műszakos munkaidő adatok, pihenőidő megsértések | Mvt. szerinti nyilvántartás |
| **Önkiszolgáló portál** | kétirányú | Munkaidő rögzítés (ha manuális); beosztás megtekintése; túlóra jóváhagyás kérés | Munkavállaló és vezető felület |
| **Projektkövető rendszer** | bejövő | Projektre fordított munkaidő (ha projektalapú elszámolás) | Projekt-munkaidő ↔ DailyTimeRecord összevetés |
| **KIR / KRÉTA (köznevelés)** | kétirányú | Pedagógus órarend, helyettesítés adatok | Tanítási órák nyilvántartása |
| **MÁK (kincstár)** | kimenő | Költségvetési szervek munkaidő-adatai | Kincstári bérszámfejtés |

---

## 20. Kapcsolódó entitások – teljes kép

```
Employment 1 ──── 1 WorkScheduleAssignment         (munkarend hozzárendelés)
WorkScheduleAssignment ──── WorkScheduleTemplate     (munkarend sablon törzsadat)

Employment 1 ──── N ShiftAssignment                  (műszakbeosztás – napi bontás)
ShiftAssignment ──── ShiftTemplate                    (műszak sablon törzsadat)

Employment 1 ──── N DailyTimeRecord                   (napi munkaidő-nyilvántartás)
Employment 1 ──── N OvertimeRecord                     (rendkívüli munkaidő)
Employment 1 ──── N OnCallRecord                       (ügyelet / készenlét)
Employment 1 ──── N TimeFramePeriod                    (munkaidőkeret periódus)

DailyTimeRecord ──── ShiftAssignment                  (beosztás–tény összekapcsolás)
DailyTimeRecord ──── LeaveRecord                       (távollét–napi rekord összekapcsolás)
OvertimeRecord ──── DailyTimeRecord                    (túlóra–napi rekord)

WorkCalendar                                            (ünnepnapok, áthelyezések)
Person.birthDate → validáció (fiatal munkavállaló)
Person.dependents → validáció (kisgyermekes korlátozások)
Employment.protectionStatus → validáció (várandós korlátozások)
Position.specialWorkConditions → validáció (veszélyes munkakörülmények)
```

---

## 21. Nyitott kérdések

1. **Munkaidő-rögzítés módja:** A napi munkaidő rögzítése manuális (munkavállaló / vezető rögzíti a portálon), beléptető rendszerből automatikus, vagy vegyes? A kötetlen munkarendű és a távmunkás dolgozóknál hogyan történik a rögzítés?
2. **Beléptető rendszer integráció mélysége:** Ha van beléptető rendszer, az HRMS importálja a nyers belépés/kilépés időpontokat (és azokból számol), vagy az access control rendszer már feldolgozott adatot küld?
3. **Munkaidő-kerekítés:** A tényleges munkaidő kerekítendő-e (pl. 15 perces egységekre)? Ha igen, milyen szabállyal?
4. **Projektalapú munkaidő:** Szükséges-e a DailyTimeRecord-on belüli projektre/feladatra bontás (timesheet jellegű nyilvántartás), vagy az a projektkövető rendszer feladata?
5. **Többmunkáltatós ügyelet:** Az egészségügyben előfordul, hogy az orvos több intézménynél is ügyeletet lát el. Ez több Employment → több OnCallRecord, vagy kell külső ügyeleti nyilvántartás?
6. **Home office / távmunka munkaideje:** A távmunkás (Mt. 196. §) és a home office-ban dolgozó munkaidejének nyilvántartása hogyan történik? A kötetlen munkarendű távmunkás esetén a nyilvántartási kötelezettség korlátozottabb.
7. **Történeti módosítás:** Mi történik, ha utólag kiderül, hogy egy korábbi hónap munkaidő-adata hibás volt (pl. betegszabadság visszamenőleges rögzítése)? Korrekciós DailyTimeRecord, vagy az eredeti felülírása audit trail-lel?
8. **Bérszámfejtési zárolás rugalmassága:** A `LOCKED` státuszú rekordok feloldása milyen jogosultságokkal és workflow-val lehetséges (pl. bérszámfejtési korrekció esetén)?
9. **Pihenőnap-nyilvántartás:** A pihenőnapokat is aktívan nyilván kell tartani DailyTimeRecord-ként (dayType = REST_DAY), vagy csak a munkanapokat?
