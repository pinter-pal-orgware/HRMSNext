# PerformanceReview domén – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer – magyar jogszabályi környezet  
> **Verzió:** 0.1 – első tervezet  
> **Utolsó frissítés:** 2025. február  
> **Kapcsolódó dokumentum:** hrms-er-osszefoglalo.md, employment-entity.md

---

## 1. A domén célja és hatóköre

A **PerformanceReview domén** a munkavállalói teljesítményértékelési folyamat teljes életciklusát kezeli: a szervezeti szintű ciklus-tervezéstől az egyéni értékelési folyamaton és célkitűzésen át a jogszabályi minősítésig és annak bér- és jogviszony-következményeiig.

A domén **8 entitásból** áll:

| Entitás | Típus | Leírás |
|---|---|---|
| **EvaluationCycle** | Törzsadat | Szervezeti szintű értékelési időablak |
| **EvaluationTemplate** | Törzsadat | Jogviszony-típusonkénti értékelési struktúra |
| **EvaluationCriteria** | Törzsadat | Értékelési szempontok hierarchiája |
| **PerformanceReview** | Operatív | Egyéni értékelési folyamat – aggregációs gyök |
| **ReviewGoal** | Operatív | Célkitűzések és teljesítmény-elvárások |
| **CriteriaRating** | Operatív | Szempontonkénti értékelések |
| **ReviewComment** | Operatív | Szöveges visszajelzések és megjegyzések |
| **CalibrationSession** | Operatív | Szervezeti kalibrációs ülések |

**Tervezési alapelvek:**

- **Employment-kötöttség:** Az értékelés mindig egy konkrét jogviszonyhoz (Employment) kötődik, nem a Person entitáshoz. Többes jogviszony esetén minden jogviszony önálló értékelési folyamatot indít – ahogy a Kjt. szerinti közalkalmazotti minősítés és egy párhuzamos Mt. szerinti jogviszony értékelése egymástól teljesen független.
- **Jogviszony-specifikus sablonok:** A Kttv., Kit., Kjt., Púétv., Eszjtv. jogviszonyok mindegyike eltérő értékelési módszertant, skálát és jogkövetkezményt ír elő. Az EvaluationTemplate entitás ezt a sokféleséget egységes struktúrában kezeli.
- **Jogszabályi vs. szervezeti értékelés szétválasztása:** Az `isRegulatory` jelző egyértelműen megkülönbözteti a kötelező jogszabályi minősítést (pl. Kjt. 66. §) a szervezet által önállóan bevezetett teljesítménymenedzsment-rendszertől.
- **Auditálhatóság:** Az értékelési folyamat minden érdemi mozzanata (célkitűzés, önértékelés, vezető értékelés, kalibráció, jóváhagyás, vitatás) külön időbélyeggel és felelős személlyel kerül rögzítésre.
- **GDPR és Mt. 10. §:** A teljesítményértékelési adatok a munkavállaló munkavégzésével kapcsolatos adatnak minősülnek; kezelésük jogalapja jogi kötelezettség (közszféra) vagy jogos érdek/szerződés (versenyszektor). Az értékelési indoklások nem tartalmazhatnak jogszabály által tiltott alapon (pl. nem, egészségi állapot) hozott megítélést.

---

## 2. EvaluationCycle entitás

### 2.1. Az entitás célja

Az **EvaluationCycle** egy szervezet-szintű értékelési időablakot definiál. Meghatározza, hogy mikor, milyen jogviszony-típusokra, milyen ütemezéssel zajlik az adott értékelési kör. Egy ciklus több párhuzamos PerformanceReview folyamatot fog össze – ez teszi lehetővé a szervezeti szintű kalibrációt és aggregált riportolást.

### 2.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `organizationId` | UUID FK | igen | Melyik szervezethez tartozik a ciklus | Technikai – multi-tenancy |
| `name` | string | igen | Ciklus neve (pl. „2025. évi Kttv. teljesítményértékelés") | Szervezeti döntés |
| `cycleType` | enum | igen | `ANNUAL` / `SEMI_ANNUAL` / `QUARTERLY` / `AD_HOC` / `REGULATORY` | Kttv. 130. § – éves kötelezettség; Kjt. 66. § – 4 évente; Eszjtv. – 3 évente |
| `referencePeriodStart` | date | igen | Az értékelt időszak kezdete | A minősítési/értékelési időszak azonosításához |
| `referencePeriodEnd` | date | igen | Az értékelt időszak vége | |
| `goalSettingDeadline` | date | nem | Célkitűzések rögzítési határideje | Kttv. 130. § (1) – a teljesítménykövetelményt az időszak elején kell kitűzni |
| `selfAssessmentDeadline` | date | nem | Önértékelés benyújtási határideje | Szervezeti döntés; Kttv. nem írja elő kötelezően, de bevált gyakorlat |
| `managerReviewDeadline` | date | igen | Vezető értékelés rögzítési határideje | Kttv. 131. §; Kjt. 66. § – a minősítőnek meghatározott határidőn belül kell elvégezni |
| `calibrationDeadline` | date | nem | Kalibrációs ülés határideje | Szervezeti döntés; kötelező jogviszony-típusonkénti eloszlási szabály esetén szükséges |
| `status` | enum | igen | `DRAFT` / `GOAL_SETTING` / `IN_PROGRESS` / `CALIBRATION` / `COMPLETED` / `ARCHIVED` | Folyamat-vezérlés |
| `applicableEmploymentTypes` | enum[] | igen | Érintett jogviszony-típusok | Kttv., Kit., Kjt., Púétv., Eszjtv., Mt. – különböző kötelezettségek |
| `isRegulatory` | boolean | igen | Jogszabályi kötelezettségből ered-e | Kttv. 130. §; Kjt. 66. §; Eszjtv. 84. § |
| `regulatoryReference` | string | nem | Pontosított jogszabályi hivatkozás (pl. „Kttv. 130-132. §§") | |
| `forcedDistribution` | JSON | nem | Kötelező eloszlási arányok kategóriánként (pl. max 20% „Kiváló") | Kttv. 131. § (3) – az értékelések meghatározott arányban adhatók ki |
| `createdAt` | datetime | igen | Rekord létrehozásának időpontja | Audit |
| `createdBy` | UUID FK | igen | Létrehozó felhasználó | Audit |
| `updatedAt` | datetime | igen | Utolsó módosítás időpontja | Audit |
| `updatedBy` | UUID FK | igen | Utolsó módosító | Audit |

**Megjegyzések:**
- A `forcedDistribution` a Kttv. azon szabályát tükrözi, hogy az „5-ös" (kiváló) értékelés az adott szervezeti egységen belül az értékeltek legfeljebb 25%-ának adható ki. Ez szervezeti szinten kezelendő korlát, nem egyéni értékelési adat.
- Az `ARCHIVED` státusz nem törlést jelent: a lezárt értékelési ciklusok adatai megőrzendők (Kttv. 228. §, Kjt. 85. § alapján a személyi iratok megőrzési ideje).

**Állapotgép:**
```
DRAFT → GOAL_SETTING → IN_PROGRESS → [CALIBRATION →] COMPLETED → ARCHIVED
                                              ↑
                              (CALIBRATION opcionális, csak ha
                               hasCalibration = true a sablonban)
```

---

## 3. EvaluationTemplate entitás

### 3.1. Az entitás célja

Az **EvaluationTemplate** jogviszony-típusonként meghatározza az értékelési folyamat struktúráját, módszertanát, skáláját és kötelező elemeit. Ez az entitás veszi fel a magyar jogszabályi sokféleséget: a Kttv. szerinti 5-fokozatú pontozós értékelés teljesen más struktúrát igényel, mint a Kjt. 4-fokozatú minősítése vagy a Púétv. fokozathoz kötött minősítési eljárása.

### 3.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `organizationId` | UUID FK | nem | null = rendszer-szintű (alapértelmezett) sablon; kitöltve = szervezet-specifikus testreszabás | Multi-tenancy |
| `name` | string | igen | Sablon neve (pl. „Kttv. éves teljesítményértékelés v2") | |
| `employmentType` | enum | igen | Melyik jogviszony-típusra alkalmazható | Kttv., Kit., Kjt., Púétv., Eszjtv., Mt. eltérő szabályai |
| `evaluationMethod` | enum | igen | `RATING_SCALE` / `COMPETENCY_BASED` / `MBO` / `360_DEGREE` / `REGULATORY_CHECKLIST` | Kttv. 130. § – teljesítménykövetelmény-alapú; Kjt. 66. § – komplex minősítés |
| `ratingScale` | enum | igen | `FOUR_POINT` / `FIVE_POINT` / `PERCENTAGE` / `PASS_FAIL` / `REGULATORY` | Kjt.: 4 fokozat; Kttv.: 5 fokozat; Mt.: szabad |
| `ratingLabels` | JSON | igen | Fokozatok szöveges megnevezései, pl. `{"1":"Nem megfelelő","2":"Megfelelő","3":"Jó","4":"Kiváló"}` | Kttv. 131. §; Kjt. 66. § (4) bek. |
| `hasGoals` | boolean | igen | Tartalmaz-e célkitűzés (MBO) modult | Kttv. 130. § (1) – kötelező teljesítménykövetelmény-kitűzés |
| `hasSelfAssessment` | boolean | igen | Van-e önértékelési szakasz | Szervezeti döntés; Kttv. nem írja elő, de bevált gyakorlat |
| `hasManagerAssessment` | boolean | igen | Van-e közvetlen vezető értékelési szakasz | Kttv. 131. §; Kjt. 66. § – a minősítő a közvetlen vezető |
| `hasSecondLevelApproval` | boolean | igen | Szükséges-e másodfokú (pl. vezető-felettes) jóváhagyás | Kttv. 131. § (4) – a teljesítményértékelést a munkáltatói jogkör gyakorlója hagyja jóvá |
| `hasCalibration` | boolean | igen | Van-e kalibrációs szakasz (cross-team összehasonlítás) | Kttv. 131. § (3) – eloszlási korlát betartatásához szükséges |
| `weightingMethod` | enum | nem | `EQUAL_WEIGHT` / `CUSTOM_WEIGHT` / `CRITERIA_WEIGHTED` | Szervezeti döntés |
| `minPassingScore` | decimal | nem | Minimális elfogadható összpontszám | Kttv. 132. § (1) – „nem megfelelő" értékelés következményei |
| `regulatoryBasis` | string | nem | Pontosított jogszabályi alap (pl. „Kttv. 130-132. §§, 63/2011. Korm. r.") | |
| `legalConsequences` | JSON | nem | Értékelési fokozatokhoz rendelt jogkövetkezmények leírása | Kttv. 132. §; Kjt. 66. § (7) – fizetési fokozat hatás |
| `isActive` | boolean | igen | Aktív sablon-e (soft delete) | |
| `validFrom` | date | igen | Sablon érvényességének kezdete | Jogszabályváltozás-követés |
| `validTo` | date | nem | Sablon érvényességének vége (null = jelenleg érvényes) | |
| `createdAt` | datetime | igen | | Audit |
| `createdBy` | UUID FK | igen | | Audit |

**Megjegyzések:**
- A `legalConsequences` JSON mező azért szükséges, mert a jogkövetkezmények jogviszony-típusonként eltérők: Kjt. esetén a „Nem megfelelő" minősítés a következő fizetési fokozatba lépést megakadályozza (Kjt. 66. § (7)); Kttv. esetén két egymást követő „nem megfelelő" értékelés felmentési ok (Kttv. 232. § (1) e) pont); Mt. esetén az értékelési eredménynek önmagában nincs kötelező jogkövetkezménye.
- A `ratingLabels` JSON struktúrája: `{"pontszám_vagy_kód": "megnevezés", ...}` – pl. Kttv.-nél `{"1":"Nem megfelelő","2":"Kevéssé megfelelő","3":"Megfelelő","4":"Jó","5":"Kiváló"}`.

---

## 4. EvaluationCriteria entitás

### 4.1. Az entitás célja

Az **EvaluationCriteria** egy adott sablon értékelési szempontjainak hierarchikus rendszerét írja le. A hierarchia kétszintű: kategória (pl. „Szakmai kompetenciák") → alkritérium (pl. „Jogszabályi ismeretek"). Ez lehetővé teszi a súlyozott összpontszám számítását és a szempontonkénti részletes visszajelzést.

### 4.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `templateId` | UUID FK | igen | Melyik sablonhoz tartozik | |
| `parentCriteriaId` | UUID FK | nem | Self-referencia: null = kategória szint; kitöltve = alkritérium | |
| `code` | string | igen | Egyedi kód a sablonon belül (pl. „COMP-01", „GOAL-QUALITY") | Riportoláshoz, szabályok hivatkozásához |
| `name` | string | igen | Szempont neve (pl. „Szakmai felkészültség", „Feladatok határidőre teljesítése") | Kttv. 130. § – a teljesítménykövetelmények konkrét tartalma |
| `description` | text | nem | Részletes leírás, értékelési útmutató | |
| `criteriaType` | enum | igen | `COMPETENCY` / `GOAL` / `BEHAVIORAL` / `REGULATORY_REQUIREMENT` | `REGULATORY_REQUIREMENT`: jogszabály által előírt kötelező elem |
| `weight` | decimal | nem | Súly százalékban (az azonos szintű kritériumok súlyainak összege 100%) | |
| `minScore` | decimal | nem | Minimálisan adható pontszám (általában a skála minimuma) | |
| `maxScore` | decimal | nem | Maximálisan adható pontszám | |
| `isMandatory` | boolean | igen | Kötelezően értékelendő-e, vagy elhagyható | Kttv. – egyes kompetenciacsoportok kötelezőek |
| `applicablePositionCategories` | string[] | nem | null = minden pozícióra érvényes; kitöltve = csak megjelölt pozíciócsoportokra | Kttv.: vezető és nem vezető eltérő kritériumrendszer |
| `sortOrder` | integer | igen | Megjelenítési sorrend | |
| `createdAt` | datetime | igen | | Audit |

**Megjegyzések:**
- A `REGULATORY_REQUIREMENT` típusú kritériumok kötelező értékelési elemek, amelyeket jogszabály ír elő – pl. Kttv. esetén a „jogszabály-ismereti felmérés" eredménye, Kjt. esetén a kötelező továbbképzési kötelezettség teljesítése (Kjt. 64. §).
- Súlyozásnál: a gyökér kategóriák súlyainak összege = 100%; az alkritériumoknál az adott kategórián belüli súlyok összege = 100%. Az összpontszám számítása: `Σ (kategória_súlya * Σ(alkritérium_súlya * pontszám))`.

---

## 5. PerformanceReview entitás

### 5.1. Az entitás célja

A **PerformanceReview** egy munkavállaló egy EvaluationCycle-on belüli teljes értékelési folyamatának aggregációs gyöke. Az Employment-hez kötött: ha ugyanaz a személy két jogviszonnyal rendelkezik, mindkét jogviszonyra külön PerformanceReview jön létre. Az entitás tárolja a folyamat összes mérföldkövét (önértékelés benyújtása, vezető értékelés, kalibráció, jóváhagyás, vitatás) és a végső értékelési eredményt – beleértve a jogszabályi minősítési fokozatot és a besorolási következményt.

### 5.2. Azonosítók és kapcsolatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `employmentId` | UUID FK | igen | Melyik jogviszonyhoz tartozik az értékelés | Kulcsfontosságú: a közszféra jogviszony-típusa határozza meg az alkalmazandó szabályrendszert |
| `cycleId` | UUID FK | igen | Melyik értékelési ciklus keretében zajlik | |
| `templateId` | UUID FK | igen | Melyik sablon alapján zajlik | Az `Employment.employmentType` + `EvaluationCycle` párosa alapján automatikusan kiválasztható |
| `revieweePositionId` | UUID FK | igen | Az értékelt által az értékelési időszakban betöltött pozíció | Jogszabályi minősítésnél a munkakör rögzítése kötelező |
| `primaryReviewerId` | UUID FK | igen | Közvetlen értékelő (általában a közvetlen vezető) | Kttv. 131. §; Kjt. 66. § – a minősítő a munkáltatói jogkör gyakorlója vagy a közvetlen vezető |
| `secondaryReviewerId` | UUID FK | nem | Másodfokú jóváhagyó (vezető-felettes, osztályvezető) | Kttv. 131. § (4) – munkáltatói jogkör gyakorlója hagyja jóvá |
| `hrReviewerId` | UUID FK | nem | HR felülvizsgáló (folyamatminőség ellenőrzés) | Szervezeti döntés |

### 5.3. Folyamat-státusz és mérföldkövek

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `status` | enum | igen | Értékelési folyamat aktuális állapota (ld. állapotgép) | |
| `goalSettingCompletedAt` | datetime | nem | Célkitűzés lezárásának időpontja | Kttv. 130. § (1) – az időszak eleji kitűzés dátuma rögzítendő |
| `selfAssessmentSubmittedAt` | datetime | nem | Önértékelés benyújtásának időpontja | Bizonyítás: a munkavállaló kapott lehetőséget önértékelésre |
| `managerReviewCompletedAt` | datetime | nem | Vezető értékelés rögzítésének időpontja | |
| `reviewSharedWithEmployeeAt` | datetime | nem | Az értékelés munkavállalóval való közlésének időpontja | Kttv. 131. § (5) – az értékelt aláírásával igazolja a megismerést; Kjt. 66. § (6) |
| `employeeAcknowledgedAt` | datetime | nem | Munkavállaló általi megismerés visszaigazolása | Kttv. 131. § (5); Kjt. 66. § (6) – az aláírás megtagadása esetén tanúk szükségesek |
| `calibrationSessionId` | UUID FK | nem | Ha kalibrációs ülésen módosítottak, melyiken | |
| `approvedAt` | datetime | nem | Végleges jóváhagyás időpontja | |
| `approvedBy` | UUID FK | nem | Jóváhagyó személye | Kttv. 131. § (4) |

**Állapotgép:**
```
NOT_STARTED
    │
    ▼ (ciklus megnyílik, célkitűzési határidő elérhető)
GOAL_SETTING
    │
    ▼ (célkitűzések rögzítve és zárolva)
SELF_ASSESSMENT ──────────────────────────── (opcionális, ha hasSelfAssessment = false → átugorja)
    │
    ▼
MANAGER_REVIEW
    │
    ▼
CALIBRATION ──────────────────────────────── (opcionális, ha hasCalibration = false → átugorja)
    │
    ▼
PENDING_APPROVAL ─────────────────────────── (opcionális, ha hasSecondLevelApproval = false → átugorja)
    │
    ▼
COMPLETED ◄──────────────────────────────── (munkavállaló megismerte, aláírta)
    │
    ├──► DISPUTED ──► (COMPLETED, ha vitatás lezárult)
    │                  (vizsgálati folyamat indul: Kttv. 232. §, Kjt. jogorvoslat)
    └──► WITHDRAWN ─► (ARCHIVED) (indokolt visszavonás: pl. jogviszony megszűnt az értékelés előtt)
```

### 5.4. Értékelési eredmények

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `selfAssessmentScore` | decimal | nem | Önértékelés összpontszáma | Összehasonlítási célra; nem befolyásolja a végleges eredményt |
| `managerScore` | decimal | nem | Vezető által adott összpontszám (kalibráció előtt) | |
| `calibratedScore` | decimal | nem | Kalibráció utáni végleges összpontszám | Kttv. 131. § (3) – eloszlási korlát figyelembevételével |
| `overallRating` | decimal | nem | Végleges összesített értékelés | |
| `overallRatingLabel` | string | nem | Fokozat szöveges megnevezése (pl. „Jó", „Megfelelő") | Kttv. 131. §; Kjt. 66. § (4) |
| `selfAssessmentText` | text | nem | Munkavállaló szöveges önértékelése | GDPR: a munkavállaló saját maga által adott adat; különleges adatot nem tartalmazhat |
| `managerSummaryText` | text | nem | Vezető összefoglaló értékelő szövege | Mt. 10. § – csak a munkavégzéssel összefüggő körülmények rögzíthetők |
| `developmentPlanText` | text | nem | Fejlesztési terv, következő időszak elvárásai | Kjt. 66. § (5) – a minősítési lapon rögzítendő |
| `strengthsText` | text | nem | Erősségek szöveges összefoglalója | Szervezeti döntés |
| `developmentAreasText` | text | nem | Fejlesztendő területek szöveges összefoglalója | Szervezeti döntés |

### 5.5. Jogszabályi minősítési adatok

Ezek a mezők csak akkor relevánsak, ha `isRegulatory = true` (a sablon vagy a ciklus szintjén).

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `isRegulatory` | boolean | igen | Jogszabályi kötelezettségből eredő minősítés-e | Kttv. 130. §; Kjt. 66. §; Eszjtv. 84. § |
| `regulatoryOutcome` | enum | nem | `EXCELLENT` / `GOOD` / `SATISFACTORY` / `UNSATISFACTORY` / `SUITABLE` / `UNSUITABLE` | Kttv.: 5 fokozat; Kjt.: 4 fokozat (Kiváló/Jó/Megfelelő/Nem megfelelő); Eszjtv.: Kiváló/Alkalmas/Alkalmatlan |
| `regulatoryOutcomeLabel` | string | nem | Jogszabály szerinti fokozat eredeti megnevezése | A `regulatoryOutcome` enum-ból számított, sablonból vett szöveg |
| `legalConsequenceApplied` | string | nem | A konkrétan alkalmazott jogkövetkezmény leírása | Kttv. 132. §; Kjt. 66. § (7) – besorolási hatás rögzítése |
| `nextMandatoryReviewDate` | date | nem | Következő kötelező értékelés / minősítés időpontja | Kttv.: évente; Kjt.: 4 évente; Eszjtv.: 3 évente; Púétv. minősítés: fokozattól függ |
| `regulatoryDeadline` | date | nem | Jogszabályi határidő (ha az értékelés kötelező és határidős) | |
| `previousRegulatoryOutcome` | enum | nem | Előző jogszabályi minősítési fokozat | Kttv. 232. § – két egymást követő „nem megfelelő" következménye |
| `consecutiveUnsatisfactoryCount` | integer | nem | Egymást követő „nem megfelelő" / „alkalmatlan" minősítések száma | Kttv. 232. § (1) e) – felmentési ok keletkezéséhez szükséges |

**Megjegyzések:**
- A `consecutiveUnsatisfactoryCount` automatikusan számítható az Employment azonos jogviszonyán korábbi PerformanceReview rekordokból, de denormalizált tárolása gyorsítja a jogkövetkezmény-értékelést és csökkenti a figyelmetlenségből eredő hibák kockázatát.
- A `legalConsequenceApplied` mező szabad szöveges: pl. „Kjt. 66. § (7) alapján a következő fizetési fokozatba lépés megtagadva; hatályos: 2025-07-01" – ez a személyi irat részévé válik.

### 5.6. Vitatási adatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `disputedAt` | datetime | nem | Vitatás bejelentésének időpontja | Kttv. 232. §; Kjt. jogorvoslati szabályok – a munkavállaló jogosult jogorvoslatra |
| `disputeReason` | text | nem | Vitatás indoklása | |
| `disputeResolution` | text | nem | Vitatás lezárásának szövege | |
| `disputeResolvedAt` | datetime | nem | Vitatás lezárásának időpontja | |
| `disputeResolvedBy` | UUID FK | nem | Vitatást lezáró személy | |
| `finalOutcomeAfterDispute` | enum | nem | Végleges eredmény a vitatás lezárása után | |

### 5.7. Audit mezők

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `createdAt` | datetime | igen | Rekord létrehozásának időpontja | |
| `createdBy` | UUID FK | igen | Létrehozó felhasználó | |
| `updatedAt` | datetime | igen | Utolsó módosítás időpontja | |
| `updatedBy` | UUID FK | igen | Utolsó módosító | |
| `completedAt` | datetime | nem | Az értékelési folyamat lezárásának időpontja | |
| `dataSource` | enum | nem | `MANUAL` / `INTEGRATION` / `MIGRATION` / `SELF_SERVICE` | |

---

## 6. ReviewGoal entitás

### 6.1. Az entitás célja

A **ReviewGoal** egy adott PerformanceReview keretében kitűzött célkitűzést vagy teljesítmény-elvárást tárol. A Kttv. 130. § (1) bekezdése kötelezővé teszi az értékelési időszak elején a teljesítménykövetelmények írásban való rögzítését. Az MBO (Management by Objectives) módszertan esetén a célok SMART-kritériumok szerint írhatók le és mérhető mutatókkal kísérhetők.

### 6.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `reviewId` | UUID FK | igen | Melyik értékelési folyamathoz tartozik | |
| `criteriaId` | UUID FK | nem | Ha sablonbeli célkritériumhoz kapcsolódik | |
| `title` | string | igen | Célkitűzés rövid megnevezése | Kttv. 130. § (1) – írásban rögzítendő teljesítménykövetelmény |
| `description` | text | nem | Részletes leírás, elvárások kifejtése | |
| `goalType` | enum | igen | `PERFORMANCE` / `DEVELOPMENT` / `ORGANIZATIONAL` / `MANDATORY_REGULATORY` | `MANDATORY_REGULATORY`: pl. kötelező továbbképzési cél |
| `weight` | decimal | nem | A cél súlya az összértékelésben (összesen 100%-ra kell adniuk) | |
| `targetValue` | string | nem | Célérték (pl. „95% határidőre teljesítés", „3 projekt lezárva") | SMART-célokhoz |
| `measurementMethod` | string | nem | Mérési módszer leírása | |
| `targetDate` | date | nem | Célhoz rendelt határidő (ha az egész értékelési időszaktól eltér) | |
| `status` | enum | igen | `DRAFT` / `AGREED` / `MODIFIED` / `ACHIEVED` / `PARTIALLY_ACHIEVED` / `NOT_ACHIEVED` / `CANCELLED` | |
| `selfAssessmentNote` | text | nem | Munkavállaló önértékelő megjegyzése a célhoz | |
| `selfAssessmentScore` | decimal | nem | Munkavállaló által saját magának adott pontszám erre a célra | |
| `managerAssessmentNote` | text | nem | Vezető értékelő megjegyzése a célhoz | Mt. 10. § – csak munkavégzéssel összefüggő tartalom |
| `managerScore` | decimal | nem | Vezető által adott pontszám erre a célra | |
| `finalScore` | decimal | nem | Végleges (esetleg kalibrált) pontszám | |
| `agreedAt` | datetime | nem | Célkitűzés kölcsönös elfogadásának időpontja | Kttv. 130. § (1) – az időszak elejéhez kötve |
| `agreedBy` | UUID FK | nem | Célkitűzést jóváhagyó vezető | |
| `modifiedAt` | datetime | nem | Utolsó célmódosítás időpontja | Kttv. 130. § (2) – a célok az időszak alatt módosíthatók, ha a körülmények változnak |
| `modificationReason` | text | nem | Célmódosítás indoka | |
| `sortOrder` | integer | igen | Megjelenítési sorrend | |
| `createdAt` | datetime | igen | | Audit |
| `createdBy` | UUID FK | igen | | Audit |

**Megjegyzések:**
- A `MODIFIED` státusz és a `modificationReason` a Kttv. 130. § (2) bekezdésének felel meg, amely lehetővé teszi a teljesítménykövetelmény módosítását, ha az értékelési időszakban olyan körülmény merül fel, amely az eredeti célkitűzést szükségtelenné vagy teljesíthetetlenné teszi. A módosítás mindkét fél aláírásával válik érvényessé – ezt az `agreedAt` / `agreedBy` mezők rögzítik.
- `MANDATORY_REGULATORY` típusú célok automatikusan generálhatók: pl. ha az Employment adataiból kiderül, hogy a közalkalmazottnak kötelező továbbképzési kötelezettsége áll fenn (Kjt. 64. §), a rendszer automatikusan kitűz egy megfelelő célt.

---

## 7. CriteriaRating entitás

### 7.1. Az entitás célja

A **CriteriaRating** az `EvaluationCriteria` szintjén rögzíti az adott értékelési folyamat szempontonkénti részletes értékelési eredményét. Különválasztja az önértékelési és a vezető általi értékelést, és tárolja a kalibrált végleges pontszámot is.

### 7.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `reviewId` | UUID FK | igen | Melyik értékelési folyamathoz tartozik | |
| `criteriaId` | UUID FK | igen | Melyik értékelési szemponthoz tartozik | |
| `raterRole` | enum | igen | `SELF` / `MANAGER` / `CALIBRATED` / `SECOND_LEVEL` | Az értékelő szerepköre |
| `raterId` | UUID FK | nem | Az értékelő személye (null, ha `SELF` és `revieweeId` már ismert) | |
| `score` | decimal | igen | Adott pontszám | A sablon `ratingScale` szerint értelmezendő |
| `scoreLabel` | string | nem | Pontszámhoz tartozó szöveges fokozat (pl. „Megfelelő") | A sablonból számított, megjelenítési célú |
| `comment` | text | nem | Indoklás, szöveges megjegyzés ehhez a szemponthoz | Mt. 10. § – csak munkavégzéssel összefüggő tartalom |
| `ratedAt` | datetime | igen | Az értékelés kitöltésének időpontja | Audit |
| `createdAt` | datetime | igen | | Audit |

**Megjegyzések:**
- Egy adott `(reviewId, criteriaId)` párhoz több `CriteriaRating` rekord is tartozhat, különböző `raterRole` értékekkel – ez lehetővé teszi az önértékelés vs. vezető értékelés vs. kalibrált érték összehasonlítását riportszinten.
- `CALIBRATED` típusú rekord kalibrációs ülésen jön létre, és ez felülírja a `MANAGER` értékelést a végeredmény számításánál.

---

## 8. ReviewComment entitás

### 8.1. Az entitás célja

A **ReviewComment** az értékelési folyamat bármely pontján keletkező szöveges megjegyzéseket tárolja. Ez elkülönül a `CriteriaRating.comment` mezőtől: az önálló megjegyzés nem kapcsolódik konkrét értékelési szemponthoz, hanem az egész folyamathoz, annak egy fázisához, vagy egy célkitűzéshez fűzött észrevétel.

### 8.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `reviewId` | UUID FK | igen | Melyik értékelési folyamathoz tartozik | |
| `goalId` | UUID FK | nem | Ha egy konkrét célkitűzéshez kapcsolódik | |
| `authorId` | UUID FK | igen | Megjegyzés szerzője | |
| `authorRole` | enum | igen | `REVIEWEE` / `PRIMARY_REVIEWER` / `SECONDARY_REVIEWER` / `HR` / `CALIBRATION_FACILITATOR` | |
| `reviewPhase` | enum | igen | `GOAL_SETTING` / `SELF_ASSESSMENT` / `MANAGER_REVIEW` / `CALIBRATION` / `APPROVAL` / `DISPUTE` | Melyik fázisban keletkezett |
| `commentText` | text | igen | Megjegyzés szövege | Mt. 10. § – csak munkavégzéssel összefüggő tartalom; GDPR 5(1)c adattakarékosság |
| `isVisibleToEmployee` | boolean | igen | Látja-e a munkavállaló ezt a megjegyzést | HR belső megjegyzések esetén `false`; munkavállaló-jogok: Kttv. 131. § (5) |
| `isVisibleToManager` | boolean | igen | Látja-e a közvetlen vezető | Kalibrációs facilitátor belső feljegyzései esetén releváns |
| `createdAt` | datetime | igen | | Audit |
| `updatedAt` | datetime | nem | | Audit (megjegyzés javítva lett-e) |

**Megjegyzések:**
- Az `isVisibleToEmployee = false` megjegyzések kezelése GDPR szempontból különös figyelmet igényel: a munkavállaló az Art. 15. cikk (hozzáférési jog) alapján adatközlést kérhet minden róla tárolt adatból. A nem látható megjegyzések csak akkor tarthatók vissza, ha konkrét kivétel (pl. mások jogainak védelme, belső ellenőrzés) alkalmazható.

---

## 9. CalibrationSession entitás

### 9.1. Az entitás célja

A **CalibrationSession** egy szervezeti szintű kalibrációs ülés adatait rögzíti, amelyen az értékelők összehangolják értékelési eredményeiket – különösen a kényszerelosztásos rendszerekben (Kttv. 131. § (3)). A kalibráció során egyes értékelések módosulhatnak; ez a módosítás visszakövethetően rögzítendő.

### 9.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `cycleId` | UUID FK | igen | Melyik értékelési ciklushoz tartozik | |
| `organizationUnitId` | UUID FK | nem | Melyik szervezeti egység kalibrációja (null = szervezet egészére) | |
| `facilitatorId` | UUID FK | igen | Kalibrációt vezető személy (általában HR) | |
| `scheduledAt` | datetime | igen | Tervezett időpont | |
| `heldAt` | datetime | nem | Tényleges megtartás időpontja | |
| `status` | enum | igen | `SCHEDULED` / `IN_PROGRESS` / `COMPLETED` / `CANCELLED` | |
| `participantIds` | UUID[] | igen | Részt vevő értékelők listája | |
| `reviewIds` | UUID[] | igen | Ezen az ülésen tárgyalt PerformanceReview rekordok | |
| `summaryText` | text | nem | Az ülés összefoglalója, döntések | |
| `distributionSnapshot` | JSON | nem | Az értékelési eloszlás az ülés előtt és után (audit célra) | Kttv. 131. § (3) – eloszlási korlát teljesülésének igazolása |
| `createdAt` | datetime | igen | | Audit |
| `createdBy` | UUID FK | igen | | Audit |

---

## 10. Historikus adatkezelés

A PerformanceReview domén entitásai **lezárt rekordként** kezelendők: az értékelési folyamat befejezése után a végleges adatok nem módosíthatók, csak olvashatók. Ez a megközelítés biztosítja a jogi bizonyítóerőt.

| Entitás | Historikus kezelés módja | Indok |
|---|---|---|
| **EvaluationCycle** | Státuszváltások naplózása; `COMPLETED` után csak archiválható | Kttv. 131. § – a lezárt ciklus adatai a személyi irat részei |
| **EvaluationTemplate** | `validFrom` / `validTo` párral verziózott; új jogszabályhoz új verzió | Jogszabályváltozás esetén a korábbi értékelések az akkori szabályok szerint értelmezendők |
| **PerformanceReview** | Minden állapotváltás időbélyeggel rögzítve; `COMPLETED` után immutábilis | Kjt. 85. §; Kttv. 228. § – személyi iratok megőrzési kötelezettsége |
| **ReviewGoal** | Módosítások `modifiedAt` + `modificationReason` mezőkkel naplózva | Kttv. 130. § (2) – célmódosítás igazolása |
| **CriteriaRating** | Minden `raterRole`-hoz külön rekord; felülírás helyett új rekord keletkezik | Kalibráció előtti és utáni értékelés összehasonlíthatósága |

**Megőrzési idők:**

| Adat | Megőrzési idő | Jogalap |
|---|---|---|
| Jogszabályi minősítés (Kttv., Kjt.) | Jogviszony megszűnése + 50 év | Kttv. 228. §; Kjt. 85. § – minősítési lap a személyi irat elidegeníthetetlen része |
| Szervezeti értékelés (Mt.) | Jogviszony megszűnése + 5 év | Art. elévülési idő; esetleges munkaügyi per |
| Kalibrációs dokumentumok | Jogviszony megszűnése + 5 év | Bizonyítási szükséglet |
| Vitatási iratok | Lezárás + 5 év (jogerő esetén + 10 év) | Pp. elévülési szabályok |

---

## 11. GDPR adatkezelési kategorizáció

| Kategória | Érintett property-k | Jogalap | Megjegyzés |
|---|---|---|---|
| **Teljesítményértékelési adatok** | `overallRating`, `overallRatingLabel`, `managerSummaryText`, `CriteriaRating.*` | Jogi kötelezettség (6(1)c) – közszféra; Jogos érdek (6(1)f) – versenyszektor | Az Mt. 10. § alapján csak a munkavégzéssel összefüggő adatok kezelhetők |
| **Célkitűzések** | `ReviewGoal.title`, `description`, `targetValue` | Szerződés (6(1)b) + Jogi kötelezettség (6(1)c) | Kttv. 130. § – teljesítménykövetelmény kötelező eleme |
| **Fejlesztési terv** | `developmentPlanText`, `developmentAreasText` | Jogos érdek (6(1)f) | Adattakarékossági elv: csak a fejlesztési szükséglethez szükséges tartalom |
| **Jogszabályi minősítés** | `regulatoryOutcome`, `legalConsequenceApplied`, `consecutiveUnsatisfactoryCount` | Jogi kötelezettség (6(1)c) | Kttv. 131-132. §; személyi irat kötelező eleme |
| **Vitatási adatok** | `disputeReason`, `disputeResolution` | Jogi kötelezettség (6(1)c) + Jogos érdek (6(1)f) | Jogorvoslati eljárás dokumentálása |
| **Audit adatok** | `createdAt`, `createdBy`, `updatedAt`, `updatedBy`, mérföldkő-időbélyegek | Jogos érdek (6(1)f) + Elszámoltathatóság (GDPR 5(2)) | |

**Különleges adatokra vonatkozó figyelmeztetés:**
A teljesítményértékelési szövegek (pl. `managerSummaryText`, `ReviewComment.commentText`) **nem tartalmazhatnak** különleges kategóriájú személyes adatot (GDPR 9. cikk): egészségi állapotra, fogyatékosságra, vallásra, politikai meggyőződésre, szakszervezeti tagságra, szexuális orientációra vonatkozó megállapításokat. Ha az értékelés eredménye a munkavállaló egészségi állapotából következő teljesítményromlással kapcsolatos, a dokumentáció csak a tényleges munkavégzési eredményt rögzítheti, az okot nem – hacsak az érintett nem járult hozzá (GDPR 9(2)a) vagy jogi kötelezettség (9(2)b) nem áll fenn.

**Automatizált döntéshozatal (GDPR 22. cikk):**
Ha az értékelési rendszer automatikusan generál béremelési javaslatot, bónuszdöntést vagy felmentési előterjesztést az összpontszám alapján, az érintett hozzájárulása vagy a jogi kötelezettség fennállása szükséges, és tájékoztatást kell adni az automatizált döntéshozatal logikájáról.

---

## 12. Hozzáférési szintek (javasolt)

| Szerep | Olvasási jog | Írási jog | Megjegyzés |
|---|---|---|---|
| **Értékelt munkavállaló** | Saját értékelés (kivéve `isVisibleToEmployee = false` megjegyzések); saját célkitűzések | Önértékelés, saját célkitűzési javaslatok | GDPR 15. cikk – hozzáférési jog az összes róla tárolt adathoz; kivételek szűken értelmezendők |
| **Közvetlen vezető** | Saját beosztottak értékelései a saját ciklusban | Értékelési pontszámok, megjegyzések – az általa értékelt személyekre | A más csapatok értékeléseihez kalibrációs ülésen kívül nincs hozzáférés |
| **Kalibrációs facilitátor (HR)** | Az érintett ciklus összes értékelése a kalibrációs egységen belül | Kalibrált pontszámok, `distributionSnapshot` | Kizárólag a kalibráció időtartama alatt |
| **HR ügyintéző** | Az általa kezelt szervezeti egység összes értékelése (minden ciklus) | Ciklus-adminisztráció, sablon-kezelés | Személyi iratok kezeléséhez szükséges hozzáférés |
| **Másodszintű jóváhagyó** | Az érintett értékelések (jóváhagyásra váró státuszban) | Jóváhagyás / visszaküldés | Kttv. 131. § (4) |
| **Érintett közvetlen felettes felettese** | Szervezeti összesített statisztikák (anonimizált) | Nincs | Szervezeti döntéshozatal támogatása |
| **Rendszeradminisztrátor** | Technikai mezők | Technikai mezők | Érdemi értékelési adathoz nem fér hozzá |

---

## 13. Validációs szabályok

| Property / Szabály | Validáció | Megjegyzés |
|---|---|---|
| `EvaluationCycle.referencePeriodEnd > referencePeriodStart` | Kötelező | Negatív vagy nulla hosszú értékelési időszak nem megengedett |
| `EvaluationCycle.managerReviewDeadline ≤ referencePeriodEnd + 90 nap` | Ajánlott (warning) | Kttv. 131. § – a minősítést az értékelt időszakot követően „haladéktalanul" el kell végezni |
| `EvaluationTemplate.ratingLabels` JSON teljes lefedettség | Kötelező | A sablon skálájának minden fokozatához tartoznia kell szöveges megnevezésnek |
| `EvaluationCriteria.weight` összege szintenkénti 100% | Kötelező | Ha `weightingMethod = CUSTOM_WEIGHT` vagy `CRITERIA_WEIGHTED` |
| `PerformanceReview: employmentId → employmentType` illeszkedés az `EvaluationTemplate.employmentType`-hoz | Kötelező | Elkerüli a rossz sablon alkalmazását |
| `PerformanceReview.regulatoryOutcome` csak ha `isRegulatory = true` | Kötelező | |
| `PerformanceReview.consecutiveUnsatisfactoryCount` nem negatív egész | Kötelező | |
| `ReviewGoal.weight` összege az egy értékeléshez tartozó célokon belül 100% | Ajánlott (warning) | Ha `weightingMethod = MBO` |
| `ReviewGoal.agreedAt` ≤ `EvaluationCycle.goalSettingDeadline` | Kötelező | Kttv. 130. § (1) – a célkitűzés az értékelési időszak elején rögzítendő |
| `CriteriaRating.score` a sablon `[minScore, maxScore]` tartományán belül | Kötelező | |
| `CalibrationSession.distributionSnapshot` tartalmazza az eloszlási korlátok teljesülésének igazolását | Ajánlott | Kttv. 131. § (3) compliance |

---

## 14. Integrációs pontok

| Rendszer / Folyamat | Irány | Érintett property-k | Cél |
|---|---|---|---|
| **Employment** | bejövő | `Employment.employmentType`, `seniorityDate`, `orgUnitId` | Jogviszony-típus alapján sablon kiválasztása; senioritás-alapú automatikus ciklus-indítás |
| **Compensation (CompensationElement)** | kimenő | `regulatoryOutcome`, `legalConsequenceApplied` | Kjt. fizetési fokozat módosítás; Kttv. illetményeltérítés alapja; teljesítményprémium számítása |
| **Compensation (OneTimePayment)** | kimenő | `overallRatingLabel`, `completedAt` | Teljesítményprémium, jutalom kifizetési jogalap rögzítése |
| **Document** | kimenő | `PerformanceReview.id`, `regulatoryOutcome`, `employeeAcknowledgedAt` | Minősítési lap / értékelési dokumentum generálása; személyi iratba csatolás |
| **Qualification (TrainingRecord)** | kétirányú | `ReviewGoal` (fejlesztési célok) | Kötelező továbbképzés teljesítése mint ReviewGoal; teljesítés visszajelzése a Qualification doménből |
| **Notification / Workflow** | kimenő | Határidők, státuszváltások | Célkitűzési határidő emlékeztetők; önértékelés megnyitása; vezető értékelési felszólítás; kalibrációs ülés értesítések |
| **Személyi irat / irattár** | kimenő | `PerformanceReview.*` (lezárt rekord) | Kjt. 85. §; Kttv. 228. § – a minősítési lap a személyi irat elidegeníthetetlen része |
| **KSH / OSAP** | kimenő | Aggregált értékelési eloszlás (anonimizált) | Statisztikai adatszolgáltatás köztisztviselőkre vonatkozóan |

---

## 15. Kapcsolódó entitások (hivatkozás)

```
EvaluationCycle 1 ──── N PerformanceReview   (egy ciklus → sok értékelés)
EvaluationCycle 1 ──── N CalibrationSession  (egy ciklus → több kalibrációs ülés)

EvaluationTemplate 1 ──── N EvaluationCriteria (sablon → kritériumhierarchia)
EvaluationTemplate 1 ──── N PerformanceReview  (sablon → értékelések)

EvaluationCriteria 1 ──── N EvaluationCriteria (self-ref: kategória → alkritérium)
EvaluationCriteria 1 ──── N CriteriaRating     (kritérium → értékelések)
EvaluationCriteria 1 ──── N ReviewGoal         (ha célkritériumhoz kötött cél)

PerformanceReview 1 ──── N ReviewGoal          (értékelés → célkitűzések)
PerformanceReview 1 ──── N CriteriaRating      (értékelés → szempontonkénti értékelések)
PerformanceReview 1 ──── N ReviewComment       (értékelés → megjegyzések)
PerformanceReview N ──── 1 CalibrationSession  (értékelés ← kalibrációs ülés)

Employment 1 ──── N PerformanceReview          (jogviszony → értékelések időrendben)
Person 1 ──── N PerformanceReview              (Person → primaryReviewerId-n keresztül: értékelőként)
```

---

## 16. Nyitott kérdések

1. **360 fokos értékelés hatóköre:** A Kttv. és Kjt. nem ír elő 360 fokos értékelést – a felfelé irányuló (munkavállaló értékeli a vezetőt) és horizontális (kolléga értékeli a kollégát) visszajelzés bevezetése szervezeti döntés. Ha mégis bekerül, a `CriteriaRating.raterRole` bővítésén túl anonimizálási kérdések merülnek fel (GDPR: az értékelő kilétének védelme vs. munkavállaló hozzáférési joga).

2. **Calibration módosítás dokumentálása:** Ha kalibrációs ülésen az értékelés módosul, a módosítás indokolása kötelező-e dokumentálni a munkavállaló számára? A Kttv. 131. § (5) alapján a munkavállaló a végleges értékelést kézhez kapja – de a kalibráció belső folyamata dokumentálásának részletessége nincs jogszabályban szabályozva.

3. **Értékelési fellebbezési fórum:** A Kttv. 232. § és Kjt. jogorvoslati szabályai a fellebbezési fórumot (munkáltatói döntés → bíróság) meghatározzák, de az HRMS-ben a `DISPUTED` státuszú értékelések kezelési folyamata (ki dönt, milyen határidővel) szervezeti szabályzatban definiálandó, és ez befolyásolja a `CalibrationSession` vagy egy esetleges `DisputeResolutionProcess` entitás szükségességét.

4. **Automatizált jogkövetkezmény-kezelés:** A `consecutiveUnsatisfactoryCount` és a `nextMandatoryReviewDate` számítása automatizálható – de a Kttv. 232. § (1) e) szerinti felmentési előterjesztés automatikus generálása jogi kockázatot hordoz (GDPR 22. cikk – automatizált döntéshozatal). Ajánlott: a rendszer csak figyelmeztessen, a döntést emberi felülvizsgálat hozza meg.

5. **Púétv. pedagógus-minősítés sajátossága:** A Púétv. szerinti minősítési eljárást nem a munkáltató, hanem az Oktatási Hivatal (OH) folytatja le – ez egy külső szervezet. Az HRMS hatóköre itt korlátozott: a minősítési kérelmet rögzíti, az eredményt importálja, de maga az eljárás külső rendszerben zajlik. Ez önálló `ExternalEvaluation` alentitást vagy integrációs endpointot indokolhat.

6. **Egyidejű jogviszony kezelése:** Ha a munkavállaló Kjt. és Mt. jogviszonnyal is rendelkezik ugyanannál a munkáltatónál, az értékelések szigorúan elkülönítendők. A sablonválasztás és a riportolás szintjén biztosítani kell, hogy a két értékelés ne „keveredjen" – különösen, ha az egyik jogszabályi minősítés, a másik szabad szervezeti értékelés.

7. **Próbaidős értékelés:** Az Mt. 45. § szerinti próbaidő alatt a munkáltató szabad felmondási jogot gyakorolhat; a Kttv. 48. § alapján próbaidő alatti minősítés is lehetséges. Kérdés, hogy a próbaidős értékelés ugyanabba a PerformanceReview entitásba kerüljön-e (más `cycleType = AD_HOC`), vagy önálló `ProbationaryReview` entitás indokolt.

8. **Célkitűzés-módosítás jogszabályi keretek között:** A Kttv. 130. § (2) lehetővé teszi a teljesítménykövetelmény módosítását, de csak „az érintett körülmény felmerülésétől számított rövid időn belül" és „mindkét fél aláírásával". Az HRMS-ben a célmódosítási workflow (jóváhagyás, határidő) implementálása szükséges – ez befolyásolja a `ReviewGoal.MODIFIED` státusz kezelési logikáját.
