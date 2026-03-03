# Payroll domén – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer – magyar jogszabályi környezet  
> **Verzió:** 0.1 – első tervezet  
> **Utolsó frissítés:** 2025. február  
> **Kapcsolódó dokumentum:** hrms-er-osszefoglalo.md, employment-entity.md, compensation-entity.md, timetracking-entity.md

---

## 1. A domén célja és hatóköre

A **Payroll domén** a munkabér kiszámításának, elszámolásának, kifizetésének és bevallásának teljes havi ciklusát kezeli. Ez a domén integrálja a Compensation (rendszeres bérelemek, juttatások, egyszeri kifizetések) és a TimeTracking (munkaidő, túlóra, ügyeleti díj) domének adatait, és ezekből állítja elő az egyéni bérszámfejtési lapot, a banki átutalási megbízást, valamint a NAV és egyéb hatóságok felé teljesítendő bevallásokat.

A domén **8 entitásból** áll:

| Entitás | Típus | Leírás |
|---|---|---|
| **PayrollCycle** | Törzsadat | Havi bérszámfejtési időszak definiálása |
| **PayrollRun** | Operatív | Egy bérszámfejtési futtatás (cikluson belül több is lehetséges) |
| **PayrollRecord** | Operatív | Egyéni bérszámfejtési lap – aggregációs gyök |
| **PayrollItem** | Operatív | Egy bérszámfejtési sor (jövedelem, levonás, járulék) |
| **TaxCalculation** | Operatív | Adó- és járulékszámítási részletek |
| **PayslipDocument** | Operatív | Generált bérpapír / elszámolólap |
| **PaymentOrder** | Operatív | Banki átutalási megbízás |
| **PayrollCorrection** | Operatív | Visszamenőleges korrekció |

**Tervezési alapelvek:**

- **Employment-kötöttség:** Minden `PayrollRecord` egy konkrét jogviszonyhoz (Employment) kötődik. Többes jogviszony esetén minden jogviszony önálló bérszámfejtési rekordot kap – a különböző jogviszony-típusokra eltérő adó- és járulékszabályok vonatkoznak.
- **Immutabilitás a zárolás után:** A `PayrollRun` zárolása után az érintett `PayrollRecord` és `PayrollItem` rekordok nem módosíthatók. Visszamenőleges javítás kizárólag `PayrollCorrection` rekord létrehozásával lehetséges, amely egy új `PayrollRun`-ban fut el.
- **Számítási transzparencia:** Minden `PayrollItem` tárolja a számítási alapot, a mértéket és az eredményt – a bérszámfejtési lap bármely sorához visszakövethető, hogy miből, hogyan jött ki az összeg.
- **Jogviszony-specifikus adózás:** A magyar jog alapján a jogviszony típusa (Mt., Kjt., Kttv., Púétv., megbízási, vállalkozói, EKHO) alapvetően meghatározza az alkalmazandó járulék- és adószabályokat. A `TaxCalculation` entitás ezt a jogviszony-specifikus számítási logikát dokumentálja.
- **NAV-kompatibilitás:** A bevallási adatstruktúra a 08-as havi adó- és járulékbevallás, a T1041 biztosítotti bejelentés és a NAV Online Számla rendszer követelményeihez igazodik.

---

## 2. PayrollCycle entitás

### 2.1. Az entitás célja

A **PayrollCycle** egy havi bérszámfejtési időszakot definiál: mikor kezdődik és végződik a bérszámfejtési időszak, mik a kulcsdátumok (adatszolgáltatási határidő, kifizetési nap, bevallási határidő), és milyen státuszban van a ciklus. Szervezetenként eltérő kifizetési napok és határidők tárolhatók.

### 2.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `organizationId` | UUID FK | igen | Melyik szervezethez tartozik | Multi-tenancy |
| `year` | integer | igen | Év | |
| `month` | integer | igen | Hónap (1–12) | |
| `periodStart` | date | igen | Bérszámfejtési időszak kezdete (általában az adott hónap 1-je) | Mt. 157. § – a munkabér esedékességének alapja |
| `periodEnd` | date | igen | Bérszámfejtési időszak vége (általában az adott hónap utolsó napja) | |
| `dataCollectionDeadline` | date | igen | Munkaidő-adatok és változások leadási határideje (TimeTracking zárolás) | Szervezeti döntés; általában a hónap utolsó előtti munkanapja |
| `paymentDate` | date | igen | Tervezett kifizetési nap | Mt. 157. § (1) – a munkabért havonta utólag, legkésőbb a tárgyhónapot követő hónap 10. napjáig kell kifizetni |
| `actualPaymentDate` | date | nem | Tényleges kifizetési nap | |
| `navDeclarationDeadline` | date | igen | NAV 08-as bevallás benyújtási határideje | Art. 50. § – a havi adó- és járulékbevallás határideje: a tárgyhónapot követő hónap 12-e |
| `status` | enum | igen | `OPEN` / `DATA_COLLECTION` / `CALCULATION_IN_PROGRESS` / `LOCKED` / `PAYMENT_ORDERED` / `CLOSED` / `ARCHIVED` | |
| `lockedAt` | datetime | nem | Bérszámfejtés zárolásának időpontja | Zárolás után módosítás csak korrekcióval lehetséges |
| `lockedBy` | UUID FK | nem | Zárolást végző felhasználó | |
| `isYearEnd` | boolean | igen | Év végi elszámolási ciklus-e (éves adójóváírás-elszámolás, éves bérösszesítő) | Szja tv. 46. § – az éves adóbevallás alapadatai |
| `centralParameterSnapshotId` | UUID FK | igen | A számítás során használt minimálbér, járulékmértékek pillanatképe | Évközi jogszabályváltozások kezelése; visszamenőleg rekonstruálható számítás |
| `notes` | text | nem | Belső megjegyzések (pl. „Rendkívüli bérelem: pedagógusbéremelés 2025.01.01.") | |
| `createdAt` | datetime | igen | | Audit |
| `createdBy` | UUID FK | igen | | Audit |
| `updatedAt` | datetime | igen | | Audit |
| `updatedBy` | UUID FK | igen | | Audit |

**Állapotgép:**
```
OPEN → DATA_COLLECTION → CALCULATION_IN_PROGRESS → LOCKED → PAYMENT_ORDERED → CLOSED → ARCHIVED
                                    ↑__________________________|
                           (újraszámítás lehetséges zárolás előtt)
```

**Megjegyzések:**
- A `centralParameterSnapshotId` kulcsfontosságú az auditálhatósághoz: ha az év során minimálbér-változás vagy járulékmérték-módosítás történik, a korábbi hónapok bérszámfejtése az akkori paraméterekkel volt helyes. A pillanatkép megőrzése lehetővé teszi a rekonstrukciót hatósági ellenőrzés esetén.
- `LOCKED` állapotban az összes `PayrollRecord` immutábilis. A zárolás után felfedezett hiba korrekcióval kezelendő – nem a ciklus újranyitásával.

---

## 3. PayrollRun entitás

### 3.1. Az entitás célja

A **PayrollRun** egy konkrét bérszámfejtési futtatást jelöl egy `PayrollCycle`-on belül. Egy ciklusban több futtatás is lehetséges: az első futtatás előzetes kalkuláció, az utolsó a végleges, zárolásra kerülő run. Visszamenőleges korrekciós futtatás is önálló `PayrollRun`-ként jelenik meg.

### 3.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `cycleId` | UUID FK | igen | Melyik bérszámfejtési ciklushoz tartozik | |
| `runType` | enum | igen | `PRELIMINARY` / `FINAL` / `CORRECTION` / `SUPPLEMENTARY` | `CORRECTION`: visszamenőleges hiba javítása; `SUPPLEMENTARY`: pótlólagos kifizetés (pl. késve érkező prémium) |
| `runNumber` | integer | igen | Futtatás sorszáma a cikluson belül (1, 2, 3...) | |
| `status` | enum | igen | `PENDING` / `RUNNING` / `COMPLETED` / `FAILED` / `LOCKED` | |
| `triggeredBy` | UUID FK | igen | Futtatást indító felhasználó | Audit |
| `triggeredAt` | datetime | igen | Futtatás indításának időpontja | Audit |
| `completedAt` | datetime | nem | Futtatás befejezésének időpontja | |
| `lockedAt` | datetime | nem | Futtatás zárolásának időpontja | |
| `lockedBy` | UUID FK | nem | Zárolást végző személy | |
| `affectedEmploymentIds` | UUID[] | nem | null = minden jogviszony; lista = csak megjelölt jogviszonyok (részleges futtatás) | Részleges újraszámítás, pl. csak az újonnan felvett munkavállalókra |
| `correctionForCycleId` | UUID FK | nem | Ha `runType = CORRECTION`: melyik korábbi ciklus hibáját javítja | |
| `errorLog` | JSON | nem | Számítási hibák és figyelmeztetések listája | |
| `recordCount` | integer | nem | Feldolgozott `PayrollRecord` rekordok száma | |
| `totalGrossPay` | decimal | nem | Összes bruttó bér (ellenőrző összeg) | |
| `totalNetPay` | decimal | nem | Összes nettó bér (ellenőrző összeg) | |
| `totalEmployerCost` | decimal | nem | Összes munkáltatói bérköltség (ellenőrző összeg) | |
| `notes` | text | nem | Belső megjegyzés | |

**Megjegyzések:**
- A `FAILED` státuszú futtatás nem eredményez `PayrollRecord` rekordokat – a hiba javítása után új futtatás indítandó.
- `SUPPLEMENTARY` futtatás esetén (pótlólagos kifizetés) az érintett munkavállalók bérszámfejtési lapja kiegészül, de az eredeti `FINAL` futtatás rekordjai nem módosulnak – a különbözet külön `PayrollRecord`-ban jelenik meg.

---

## 4. PayrollRecord entitás

### 4.1. Az entitás célja

A **PayrollRecord** egy munkavállaló egy `PayrollRun`-ban szereplő teljes egyéni bérszámfejtési lapját reprezentálja. Ez a domén aggregációs gyöke: minden részlet (tételek, adó- és járulékszámítás, bérpapír, átutalási sor) erre a rekordra épül. A rekord tárolja a számítás szempontjából releváns pillanatkép-adatokat is (milyen besorolással, milyen munkaidővel, milyen jogviszony-típussal rendelkezett a munkavállaló az adott hónapban).

### 4.2. Azonosítók és kapcsolatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `runId` | UUID FK | igen | Melyik futtatáshoz tartozik | |
| `cycleId` | UUID FK | igen | Melyik bérszámfejtési ciklushoz tartozik (denormalizált gyorsítás) | |
| `employmentId` | UUID FK | igen | Melyik jogviszonyhoz tartozik | Jogviszony-specifikus adózás alapja |
| `personId` | UUID FK | igen | Melyik természetes személyhez tartozik (denormalizált) | NAV bevallásokhoz szükséges |

### 4.3. Pillanatkép-adatok (snapshot)

Ezek a mezők az adott hónapban érvényes állapotot rögzítik – a bérszámfejtési rekord utólag is rekonstruálható legyen, még ha az Employment adatai azóta változtak is.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `snapshotEmploymentType` | enum | igen | Jogviszony típusa a számítás pillanatában | Meghatározza az alkalmazandó adó- és járulékszabályokat |
| `snapshotPositionId` | UUID FK | igen | Betöltött pozíció a hónapban | Munkakör rögzítése bérszámfejtési dokumentációhoz |
| `snapshotOrgUnitId` | UUID FK | igen | Szervezeti egység a hónapban | Költséghely-allokációhoz |
| `snapshotWeeklyHours` | decimal | igen | Heti munkaidő a hónapban | Arányos bérszámítás alapja részmunkaidőnél |
| `snapshotIsPensioner` | boolean | igen | Nyugdíjas státusz a hónapban | Tbj. 6. § – nyugdíjas nem fizet TB járulékot |
| `snapshotIsDisabled` | boolean | igen | Megváltozott munkaképesség a hónapban | Szocho tv. – munkáltatói szocho kedvezmény |
| `snapshotSalaryGrade` | string | nem | Besorolási fokozat a hónapban (közszféra) | Kjt., Kttv., Eszjtv., Púétv. besorolási táblák |
| `snapshotTaxExemptionDeclaration` | enum | nem | Az adott hónapban érvényes adónyilatkozat típusa | Szja tv. 48. § – munkavállaló adónyilatkozata az adóelőleg-alap meghatározásához |
| `calendarDaysInPeriod` | integer | igen | Naptári napok száma a hónapban | Arányos számítás alapja (pl. belépő/kilépő munkavállalóknál) |
| `workingDaysInPeriod` | integer | igen | Munkanapok száma a hónapban (munkaidőnaptár szerint) | |
| `actualWorkedDays` | decimal | igen | Ténylegesen munkában töltött napok (távollétek levonva) | Arányos bér alapja |
| `actualWorkedHours` | decimal | igen | Ténylegesen ledolgozott órák | Órabéres számításhoz; túlóra-alap |

### 4.4. Összesített béradatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `grossPay` | decimal | igen | Bruttó jövedelem összesen (minden jövedelemelem összege) | Szja tv. 24. § – összevont adóalap |
| `taxableIncome` | decimal | igen | Adóköteles jövedelem (adómentes tételek levonva a bruttóból) | Szja tv. 7. § – jövedelem fogalma; adómentes tételek: Szja tv. 1. sz. melléklet |
| `superGrossBase` | decimal | nem | Szuperbruttó alap (ha alkalmazandó – Magyarországon 2011 óta nem, de visszamenőleges adatoknál releváns lehet) | |
| `personalIncomeTax` | decimal | igen | SZJA előleg összege | Szja tv. 47-48. § – 15%-os kulcs |
| `personalIncomeTaxBase` | decimal | igen | SZJA adóalap (adókedvezmények levonása előtt) | |
| `employeeSocialContribution` | decimal | igen | Munkavállalói TB járulék | Tbj. 24. § – 18,5% (2020-tól) |
| `employeeSocialContributionBase` | decimal | igen | Munkavállalói TB járulékalap | Tbj. 27. § – az alap és a maximum összege |
| `employerSocialContribution` | decimal | igen | Munkáltatói szociális hozzájárulási adó (szocho) | Szocho tv. 2. § – 13% (2022-től) |
| `employerSocialContributionBase` | decimal | igen | Szocho adóalap | |
| `totalTaxDeductions` | decimal | igen | Összes levonás az adóelőlegből (adókedvezmények) | |
| `netPay` | decimal | igen | Nettó kifizetendő összeg (bruttó − munkavállalói járulékok − SZJA + adókedvezmények) | |
| `totalEmployerCost` | decimal | igen | Teljes munkáltatói bérköltség (bruttó + munkáltatói járulékok és adók) | Tervezési, cost center allokációs célra |
| `payableAmount` | decimal | igen | Ténylegesen utalandó összeg (nettó − egyéb levonások pl. előleg-visszavonás, foglalás) | Mt. 161. § – a munkabér letiltásának szabályai |

### 4.5. Adókedvezmények összesítője

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `familyTaxAllowance` | decimal | nem | Igénybevett családi adókedvezmény | Szja tv. 29/A. § – eltartottanként és jogcímenként eltérő összegek |
| `familyTaxAllowanceBase` | decimal | nem | Családi kedvezmény alapja (eltartottak száma × havi összeg) | |
| `personalAllowance` | decimal | nem | Személyi kedvezmény (fogyatékossági, súlyos fogyatékossági) | Szja tv. 29/B. § |
| `firstMarriageAllowance` | decimal | nem | Első házasok kedvezménye | Szja tv. 29/C. § – havi 5 000 Ft adókedvezmény, max. 24 hónapig |
| `under25Exemption` | decimal | nem | 25 év alattiak SZJA-mentessége | Szja tv. 29/D. § – az átlagos kereseti korlát figyelembevételével |
| `under30MothersExemption` | decimal | nem | 30 év alatti anyák SZJA-mentessége | Szja tv. 29/E. § (2023-tól) |
| `ekhoTax` | decimal | nem | EKHO mértéke (ha alkalmazandó) | 2005. évi CXX. tv. – egyszerűsített közteherviselési hozzájárulás |
| `totalAllowances` | decimal | igen | Összes érvényesített adókedvezmény összege | |

### 4.6. Státusz és folyamat

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `status` | enum | igen | `PENDING` / `CALCULATED` / `REVIEWED` / `APPROVED` / `LOCKED` / `PAID` / `CORRECTED` | |
| `isLocked` | boolean | igen | Immutábilis-e a rekord | Zárolás után csak korrekcióval módosítható |
| `isCorrected` | boolean | igen | Van-e hozzá korrekciós rekord | |
| `correctionId` | UUID FK | nem | Ha korrigálva lett, melyik `PayrollCorrection` rekord javítja | |
| `approvedBy` | UUID FK | nem | Jóváhagyó (általában a bérszámfejtő vagy vezető bérszámfejtő) | |
| `approvedAt` | datetime | nem | Jóváhagyás időpontja | |
| `notes` | text | nem | Bérszámfejtői megjegyzések (pl. belépő munkavállaló, részleges hónap) | |
| `createdAt` | datetime | igen | | Audit |
| `updatedAt` | datetime | igen | | Audit |

---

## 5. PayrollItem entitás

### 5.1. Az entitás célja

A **PayrollItem** egyetlen bérszámfejtési sort képvisel: egy jövedelemelemet, levonást, munkáltatói járulékot vagy egyéb tételt. Minden sor tartalmazza a számítás alapját, a mértéket és az eredményt – a bérszámfejtési lap bármely sorához visszakövethető a számítási logika. A `PayrollItem` közvetlenül kapcsolódik a forrás-entitáshoz (pl. `CompensationElement`, `LeaveRecord`, `OvertimeRecord`), ezzel megvalósítva a teljes nyomonkövethetőséget.

### 5.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `recordId` | UUID FK | igen | Melyik `PayrollRecord`-hoz tartozik | |
| `compensationTypeId` | UUID FK | igen | Bérelem típusa (`CompensationType` törzsadatból) | A számítási szabály és adójelleg forrása |
| `itemCategory` | enum | igen | `GROSS_EARNING` / `BENEFIT_IN_KIND` / `BENEFIT_TAX_FREE` / `EMPLOYEE_DEDUCTION` / `EMPLOYEE_CONTRIBUTION` / `EMPLOYER_CONTRIBUTION` / `EMPLOYER_TAX` / `NET_DEDUCTION` | Bérszámfejtési lap szekciójának meghatározásához |
| `description` | string | igen | Megjelenő sor-megnevezés (pl. „Alapbér", „Éjszakai pótlék 25%", „TB járulék 18,5%") | Bérpapíron megjelenő szöveg |
| `calculationBase` | decimal | nem | Számítás alapja (pl. alapbér, napi átlagbér) | |
| `calculationRate` | decimal | nem | Alkalmazott mérték (pl. 0.25 = 25% pótlék; 0.185 = 18,5% TB) | |
| `quantity` | decimal | nem | Mennyiség (pl. ledolgozott munkanapok, túlóraórák száma) | |
| `unitPrice` | decimal | nem | Egységár (pl. napi bér = havi bér / munkanapok) | |
| `amount` | decimal | igen | Tétel összege (pozitív = jövedelem/munkáltatói teher; negatív = levonás) | |
| `isTaxable` | boolean | igen | SZJA-köteles-e | Szja tv. 1. sz. melléklet – adómentességi lista |
| `isSocialContributionBase` | boolean | igen | TB járulékalap része-e | Tbj. 27. § – a járulékalap és kivételek |
| `isSzochoBase` | boolean | igen | Szocho alap része-e | Szocho tv. 4. § – a szocho alap és kivételek |
| `affectsAveragePay` | boolean | igen | Az átlagkereset-számítás alapjába tartozik-e | Mt. 151. § – átlagkereset-számítás (szabadságpótlék, végkielégítés alapja) |
| `sourceEntityType` | enum | nem | `COMPENSATION_ELEMENT` / `ONE_TIME_PAYMENT` / `BENEFIT_ENTITLEMENT` / `LEAVE_RECORD` / `OVERTIME_RECORD` / `ONCALL_RECORD` / `MANUAL` | |
| `sourceEntityId` | UUID | nem | A forrás entitás azonosítója (pl. `OvertimeRecord.id`) | Teljes visszakövethetőség |
| `sortOrder` | integer | igen | Megjelenítési sorrend a bérszámfejtési lapon | |
| `createdAt` | datetime | igen | | Audit |

**Megjegyzések:**
- A `NET_DEDUCTION` kategória a nettó bérből levont tételeket jelöli: előleg-visszavonás (Mt. 161. §), bírósági letiltás/foglalás, önkéntes levonások (pl. munkáltatói kölcsön törlesztése).
- Mt. 161. § alapján a bírósági letiltás maximuma: a nettó munkabér 33%-a általánosan, 50%-a tartásdíj esetén. A rendszernek automatikusan ellenőriznie kell, hogy a `NET_DEDUCTION` tételek összege nem haladja-e meg a jogszabályi korlátot.
- `MANUAL` `sourceEntityType`: kivételesen kézzel rögzített tételek, amelyek automatikusan nem generálódnak (pl. ad hoc jutalom, bérkorrekció) – ezek kötelezően indoklást igényelnek.

---

## 6. TaxCalculation entitás

### 6.1. Az entitás célja

A **TaxCalculation** egy `PayrollRecord`-hoz tartozó részletes adó- és járulékszámítási dokumentáció. Míg a `PayrollRecord` összesített összegeket tárol (SZJA összesen, TB összesen stb.), a `TaxCalculation` az egyes számítási lépéseket, korlátokat, kedvezmény-érvényesítési sorrendeket és az éves göngyölített értékeket rögzíti – ez szükséges a hatósági ellenőrzéshez és az éves adóelszámoláshoz.

### 6.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `recordId` | UUID FK | igen | Melyik `PayrollRecord`-hoz tartozik | |
| `calculationDate` | datetime | igen | Számítás végrehajtásának időpontja | |
| `parameterSnapshotId` | UUID FK | igen | Használt CentralParameter pillanatkép azonosítója | Visszamenőleges rekonstrukcióhoz |

**Göngyölített éves adatok (kumulatív, az év elejétől):**

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `ytdGrossIncome` | decimal | igen | Éves göngyölített bruttó jövedelem (year-to-date) | Szja tv. 48. § – az adóelőleg-alap meghatározása az éves jövedelem becslésén alapul |
| `ytdTaxableIncome` | decimal | igen | Éves göngyölített adóköteles jövedelem | |
| `ytdPersonalIncomeTaxWithheld` | decimal | igen | Éves göngyölített levont SZJA előleg | Szja tv. 47. § |
| `ytdFamilyAllowanceUsed` | decimal | nem | Éves göngyölített igénybevett családi kedvezmény | Szja tv. 29/A. § – az éves korlát figyelembevételéhez |
| `ytdUnder25ExemptionUsed` | decimal | nem | Éves göngyölített 25 év alattiak mentessége | Szja tv. 29/D. § – éves átlagkereset-korlát figyelembevételéhez |
| `ytdSocialContributionBase` | decimal | igen | Éves göngyölített TB járulékalap | Tbj. 27. § – havi és éves maximumok ellenőrzéséhez |
| `ytdEmployeeSocialContribution` | decimal | igen | Éves göngyölített levont munkavállalói TB járulék | |
| `ytdEmployerSocialContribution` | decimal | igen | Éves göngyölített munkáltatói szocho | |

**Havi részletes számítás:**

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `monthlyTaxBase` | decimal | igen | Havi adóalap (összevont adóalap rész) | Szja tv. 29. § |
| `monthlyTaxBeforeAllowances` | decimal | igen | Kedvezmények érvényesítése előtti SZJA (adóalap × 15%) | Szja tv. 45. § |
| `allowanceApplicationOrder` | JSON | nem | Kedvezmények érvényesítési sorrendje és összegei (Mt.: 29/D, 29/E, 29/C, 29/B, 29/A) | Szja tv. – a kedvezmények érvényesítési sorrendje kötött |
| `taxAfterAllowances` | decimal | igen | Kedvezmények utáni SZJA előleg | |
| `socialContributionCap` | decimal | nem | Alkalmazott TB járulékmaximum (ha adott esetben releváns) | Tbj. 28. § – a biztosított TB járulékfizetési felső határa egyes járuléktípusoknál |
| `szochoAllowance` | decimal | nem | Munkáltatói szocho kedvezmény (pl. megváltozott munkaképességű, GYED-ről visszatérő, 25 év alatti) | Szocho tv. 10-17. § – kedvezményes szocho mértékek |
| `szochoAllowanceType` | enum | nem | `DISABLED_WORKER` / `RETURNING_FROM_GYED` / `UNDER25` / `KARRIER_HID` / `NONE` | Szocho tv. vonatkozó paragrafusai |
| `ekhoBase` | decimal | nem | EKHO alapja (ha alkalmazandó) | 2005. évi CXX. tv. 3. § |
| `ekhoEmployeeRate` | decimal | nem | Alkalmazott munkavállalói EKHO kulcs | 2005. évi CXX. tv. 5. § |
| `ekhoEmployerRate` | decimal | nem | Alkalmazott munkáltatói EKHO kulcs | 2005. évi CXX. tv. 5. § |
| `isPartialMonth` | boolean | igen | Részleges hónap-e (belépő/kilépő munkavállaló) | |
| `partialMonthRatio` | decimal | nem | Arányosítási szorzó (ha `isPartialMonth = true`) | |
| `calculationNotes` | text | nem | Bérszámfejtői megjegyzés speciális számítási esetekhez | |
| `calculationLog` | JSON | nem | Lépésenkénti számítási napló (debug és audit célra) | |

**Megjegyzések:**
- Az `allowanceApplicationOrder` JSON a Szja tv. által meghatározott kötött érvényesítési sorrendet dokumentálja: (1) 25 év alattiak mentessége / 30 év alatti anyák mentessége, (2) első házasok kedvezménye, (3) személyi kedvezmény, (4) családi adókedvezmény. Ha a kedvezmény a havi adót meghaladja, az átvihető a következő hónapra – ennek nyomonkövetése az `ytd` mezőkkel valósítható meg.
- A `szochoAllowanceType = RETURNING_FROM_GYED`: a GYED-ről, GYES-ről visszatérő munkavállaló foglalkoztatása esetén 2 évig a munkáltató szocho mentességet élvez (Szocho tv. 12. §). Ezt a `Employment.snapshotIsReturningFromGyed` jelzőből kell vezérelni.

---

## 7. PayslipDocument entitás

### 7.1. Az entitás célja

A **PayslipDocument** a munkavállaló részére kiállított bérpapírt (elszámolólapot) reprezentálja. A Mt. 155. § (1) bekezdése kötelezővé teszi, hogy a munkáltató a munkabér kifizetésekor írásbeli elszámolást adjon a munkavállaló részére – ez a bérszámfejtési lap. A `PayslipDocument` tárolja a generált dokumentum metaadatait és hivatkozás-azonosítóját.

### 7.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `recordId` | UUID FK | igen | Melyik `PayrollRecord`-hoz tartozik | |
| `documentType` | enum | igen | `MONTHLY_PAYSLIP` / `ANNUAL_SUMMARY` / `CORRECTION_PAYSLIP` / `SUPPLEMENTARY_PAYSLIP` | Mt. 155. § (1); Szja tv. 46. § – éves igazolás |
| `generatedAt` | datetime | igen | Generálás időpontja | |
| `generatedBy` | UUID FK | igen | Generálást végző felhasználó vagy rendszer | |
| `documentReference` | string | igen | Dokumentum egyedi hivatkozási azonosítója (iktatószám) | |
| `storageLocation` | string | nem | A Document doménbe mutató hivatkozás (pl. `Document.id`) | Mt. 155. § – az elszámolólapot a munkáltatónak meg kell őriznie |
| `deliveryMethod` | enum | nem | `ELECTRONIC` / `PAPER` / `PORTAL` | Mt. 155. § – elektronikus kézbesítés a munkavállaló beleegyezésével lehetséges |
| `deliveredAt` | datetime | nem | Kézbesítés időpontja | |
| `deliveryConfirmedAt` | datetime | nem | Kézhezvétel visszaigazolásának időpontja | Mt. 155. § (1) – a kifizetéssel egyidejűleg átadandó |
| `employeeAcknowledgedAt` | datetime | nem | Online portálon való megnyitás / letöltés időpontja | |
| `isElectronicConsentGiven` | boolean | nem | Adott-e a munkavállaló hozzájárulást elektronikus kézbesítéshez | Mt. 155. § – hozzájárulás szükséges az e-kézbesítéshez |
| `locale` | string | nem | Bérpapír nyelvezete (ISO 639-1, pl. `hu`, `en`) | Külföldi munkavállalóknál releváns |

**Megjegyzések:**
- Az éves adóigazolás (Szja tv. 46. §) szintén `PayslipDocument`-ként kezelendő, `documentType = ANNUAL_SUMMARY` típussal. Ez a dokumentum az adóbevallás alapdokumentuma, amelyet a munkáltatónak január 31-ig kell kiadnia.
- A `storageLocation` a Document doménre mutat: a bérpapír hosszú távú megőrzése ott történik, ahol az összes személyi irathoz kapcsolódó dokumentum archiválása zajlik.

---

## 8. PaymentOrder entitás

### 8.1. Az entitás célja

A **PaymentOrder** egy `PayrollRun`-hoz tartozó banki átutalási megbízás-rekordot képvisel. Egy futtatáshoz több `PaymentOrder` tartozhat: munkavállalónként, ha az utalás egyéni tételenként megy (ritka), vagy összesítve bankszámlánként és kifizetési napokként. Ezen kívül különálló `PaymentOrder` keletkezik a NAV felé fizetendő adók és járulékok, a SZÉP-kártya-feltöltések és egyéb kifizetések tekintetében.

### 8.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `runId` | UUID FK | igen | Melyik futtatáshoz tartozik | |
| `cycleId` | UUID FK | igen | Melyik bérszámfejtési ciklushoz tartozik | |
| `orderType` | enum | igen | `EMPLOYEE_NET_PAY` / `TAX_AUTHORITY_PAYMENT` / `SOCIAL_INSURANCE_PAYMENT` / `SZEP_CARD_TRANSFER` / `OTHER_BENEFIT` | |
| `paymentDate` | date | igen | Tervezett utalási nap | Mt. 157. § (1) – legkésőbb a tárgyhónapot követő 10-ig |
| `actualPaymentDate` | date | nem | Tényleges utalási nap | |
| `sourceBankAccount` | string | igen | Munkáltató bankszámlaszáma | |
| `targetBankAccount` | string | igen | Kedvezményezett bankszámlaszáma | Mt. 158. § – munkavállaló által megjelölt bankszámla |
| `targetName` | string | igen | Kedvezményezett neve | |
| `amount` | decimal | igen | Utalandó összeg | |
| `currency` | string(3) | igen | Pénznem (ISO 4217, általában `HUF`) | |
| `paymentReference` | string | nem | Közlemény mező tartalma (pl. „2025-01 bér", NAV azonosítószám) | |
| `status` | enum | igen | `DRAFT` / `SUBMITTED` / `CONFIRMED` / `REJECTED` / `CANCELLED` | |
| `submittedAt` | datetime | nem | Banki rendszerbe küldés időpontja | |
| `confirmedAt` | datetime | nem | Banki visszaigazolás időpontja | |
| `bankTransactionId` | string | nem | Bank által adott tranzakcióazonosító | Visszakereshetőség, banki kivonat egyeztetéséhez |
| `rejectionReason` | string | nem | Ha `status = REJECTED`: elutasítás oka | |
| `payrollRecordIds` | UUID[] | nem | Érintett `PayrollRecord` rekordok listája (ha egyéni utalás) | |
| `navPaymentType` | enum | nem | Ha `orderType = TAX_AUTHORITY_PAYMENT`: NAV adónem-kód | Art. – az adónem azonosításához |

**Megjegyzések:**
- A Mt. 158. §-a alapján a munkabért a munkavállaló által megjelölt bankszámlára kell utalni. Ha a munkavállalónak több bankszámlája van, és megosztja a kifizetést (pl. 70% egyik, 30% másik számlára), minden számlára külön `PaymentOrder` keletkezik.
- NAV felé fizetendő összegek (SZJA előleg, TB járulék, szocho): a vállalkozás saját bankszámlájáról az adott hónapra meghatározott NAV számlaszámokra utalt összegek. Ezek a `TAX_AUTHORITY_PAYMENT` és `SOCIAL_INSURANCE_PAYMENT` típusú `PaymentOrder`-ek.

---

## 9. PayrollCorrection entitás

### 9.1. Az entitás célja

A **PayrollCorrection** visszamenőleges bérszámfejtési hibák vagy változások kezelésére szolgál. Egy már `LOCKED` állapotú `PayrollRecord` nem módosítható – az eltérés korrigálása mindig egy új `PayrollRun`-ban, egy `CORRECTION` típusú futtatáson belül zajlik, amelyhez ez az entitás biztosítja a korrektív összegek és indoklás dokumentálását. Jellemző esetek: utólag érkező táppénzbejelentés, visszamenőleges besorolásmódosítás (Kjt. fokozatlépés), téves juttatás visszavonása.

### 9.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `correctionRunId` | UUID FK | igen | Melyik korrekciós `PayrollRun`-ban fut | |
| `originalRecordId` | UUID FK | igen | Melyik eredeti `PayrollRecord`-ot korrigálja | |
| `originalCycleId` | UUID FK | igen | Melyik eredeti hónap hibáját javítja | |
| `correctionType` | enum | igen | `GROSS_PAY_CORRECTION` / `BENEFIT_CORRECTION` / `TAX_CORRECTION` / `CONTRIBUTION_CORRECTION` / `DEDUCTION_CORRECTION` / `FULL_RECALCULATION` | |
| `correctionReason` | enum | igen | `SICK_LEAVE_LATE_NOTIFICATION` / `RETROACTIVE_SALARY_CHANGE` / `GRADING_CORRECTION` / `PAYROLL_ERROR` / `REGULATORY_CHANGE` / `OTHER` | |
| `correctionReasonDetail` | text | igen | Részletes indoklás | Art. 57. § – az önellenőrzés indokát dokumentálni kell |
| `originalGrossPay` | decimal | igen | Eredeti bruttó bér | |
| `correctedGrossPay` | decimal | igen | Helyes bruttó bér | |
| `differencePay` | decimal | igen | Eltérés összege (corrected − original; pozitív = pótlólagos kifizetés; negatív = visszavonás) | |
| `originalTax` | decimal | igen | Eredeti SZJA összeg | |
| `correctedTax` | decimal | igen | Helyes SZJA összeg | |
| `differenceTax` | decimal | igen | SZJA eltérés | |
| `originalContributions` | decimal | igen | Eredeti munkavállalói járulékok összege | |
| `correctedContributions` | decimal | igen | Helyes munkavállalói járulékok összege | |
| `differenceContributions` | decimal | igen | Járulék eltérés | |
| `requiresNavCorrection` | boolean | igen | Szükséges-e a 08-as bevallás önellenőrzése | Art. 54. § – önellenőrzési kötelezettség |
| `navCorrectionStatus` | enum | nem | `NOT_REQUIRED` / `PENDING` / `SUBMITTED` / `ACCEPTED` | |
| `navCorrectionSubmittedAt` | datetime | nem | NAV önellenőrzés benyújtásának időpontja | Art. 54. § (5) – az önellenőrzés benyújtásának határideje |
| `navCorrectionReference` | string | nem | NAV által kiadott befogadási azonosító | |
| `selfRevisionSurcharge` | decimal | nem | Önellenőrzési pótlék összege (ha fizetendő) | Art. 54. § (3) – önellenőrzési pótlék = késedelmi pótlék 75%-a |
| `approvedBy` | UUID FK | igen | Korrekcióját jóváhagyó személy (általában vezető bérszámfejtő) | |
| `approvedAt` | datetime | igen | | |
| `createdAt` | datetime | igen | | Audit |
| `createdBy` | UUID FK | igen | | Audit |

**Megjegyzések:**
- `RETROACTIVE_SALARY_CHANGE` típusú korrekció: a Kjt. és Kttv. jogviszonyoknál a fokozatlépés, besorolásmódosítás visszamenőleges hatályú. Ilyenkor a különbözetet az aktuális hónapban kell kifizetni, és a korábbi hónapok 08-as bevallásait önellenőrzéssel kell javítani.
- `REGULATORY_CHANGE`: az Art. 57. § (1) bekezdése alapján, ha jogszabályváltozás miatt az előző időszakra megállapított adó összege változik, az adózó önellenőrzéssel köteles helyesbíteni. Ez különösen minimálbér-változásnál és SZJA-kulcs változásakor releváns.

---

## 10. Historikus adatkezelés

| Entitás | Kezelési mód | Indok |
|---|---|---|
| **PayrollCycle** | Státuszváltások naplózva; `CLOSED` után nem törölhető | Art. 78. § – az adó megállapításához való jog elévülése 5 év; ennyi ideig a bevallási alap rekonstruálható kell legyen |
| **PayrollRun** | `LOCKED` után immutábilis; `errorLog` megőrzendő | Hatósági ellenőrzés esetén a számítás folyamata rekonstruálható kell legyen |
| **PayrollRecord** | `LOCKED` után immutábilis; korrekció új rekordban | Art. 54. § – önellenőrzés kötelező; az eredeti és a korrigált rekord mindkettő megőrzendő |
| **PayrollItem** | Nem módosítható zárolás után | Bérszámfejtési lap hitelessége |
| **TaxCalculation** | Immutábilis; `calculationLog` JSON teljes lépéssorozattal | NAV ellenőrzés esetén a számítás lépései visszakövethetők |
| **PayslipDocument** | Nem törölhető | Mt. 155. § – munkáltató megőrzési kötelezettsége |
| **PaymentOrder** | `CONFIRMED` után csak olvasható | Számviteli bizonylat; banki kivonat egyeztetéséhez |
| **PayrollCorrection** | Teljes auditnyom; az eredeti rekord és a korrekció mindkettő megőrzendő | Art. 54. §; könyvvizsgálati szükséglet |

**Megőrzési idők:**

| Adat | Megőrzési idő | Jogalap |
|---|---|---|
| Bérszámfejtési lapok (PayrollRecord, PayrollItem) | Jogviszony megszűnése + 50 év | 1997. évi LXXXI. tv. (Tny.) – a nyugdíj megállapításához szükséges kereset-adatok |
| Elszámoló lapok (PayslipDocument) | Jogviszony megszűnése + 5 év (Mt.) + 50 év (Tny.) | Irányadó a hosszabb; a nyugdíjigénylés alapdokumentumai |
| Adóbevallási alapadatok (TaxCalculation) | 5 év | Art. 78. § – az adó megállapításához való jog elévülése |
| Banki átutalási megbízások (PaymentOrder) | 8 év | Szt. 169. § – számviteli bizonylatok megőrzési ideje |
| Önellenőrzési iratok (PayrollCorrection) | 5 év a korrekció időpontjától | Art. 78. § |
| CentralParameter pillanatképek | Az érintett bérszámfejtési adatok megőrzési idejéig | Számítás rekonstruálhatóság |

---

## 11. GDPR adatkezelési kategorizáció

| Kategória | Érintett property-k | Jogalap | Megjegyzés |
|---|---|---|---|
| **Bérszámfejtési adatok** | `PayrollRecord.grossPay`, `netPay`, `payableAmount` | Jogi kötelezettség (6(1)c) – Mt. 155. §; Art. | A munkabér megállapítása és kifizetése jogszabályi kötelezettség |
| **Adó- és járulékadatok** | `TaxCalculation.*` | Jogi kötelezettség (6(1)c) – Szja tv., Tbj., Szocho tv., Art. | NAV bevalláshoz szükséges |
| **Adókedvezmény adatok** | `familyTaxAllowance`, `personalAllowance`, `under25Exemption` stb. | Jogi kötelezettség (6(1)c) – Szja tv. | A munkavállaló nyilatkozata alapján kezelendő; a nyilatkozat megőrzendő |
| **Bankszámla adatok (kifizetési)** | `PaymentOrder.targetBankAccount` | Szerződés (6(1)b) + Jogi kötelezettség (6(1)c) | Mt. 158. § – kötelező bankszámlára utalni |
| **Különleges adatból eredő kedvezmény** | `snapshotIsDisabled`, `szochoAllowanceType = DISABLED_WORKER` | Jogi kötelezettség (9(2)b) | A fogyatékossági kedvezmény érvényesítéséhez szükséges, de a bérszámfejtési adatban csak a kedvezmény összege és jogcíme szerepeljen, ne az orvosi diagnózis |
| **Audit adatok** | `approvedBy`, `approvedAt`, `lockedBy`, `lockedAt` | Jogos érdek (6(1)f) + Elszámoltathatóság (GDPR 5(2)) | |

**Adattakarékosság a bérszámfejtési adatokban:**
A `PayrollItem.description` és a `TaxCalculation.calculationNotes` mezők nem tartalmazhatnak személyes egészségügyi vagy egyéb különleges kategóriájú adatot. A megváltozott munkaképesség kedvezménye pl. `szochoAllowanceType = DISABLED_WORKER` kóddal rögzítendő – az orvosi diagnózis nem kerül a bérszámfejtési rekordba (az a Person entitásnál tárolódik külön jogalapon).

**Hozzáférés-korlátozás:**
A bérszámfejtési adatok különösen érzékeny személyes adatok: a munkavállaló pontos jövedelmét, adózási helyzetét és bankszámláját tartalmazzák. A „need-to-know" elvnek szigorúan érvényesülnie kell – a közvetlen vezető alapvetően **nem** jogosult a beosztottjai konkrét bérszámfejtési adataira, csak az általa jóváhagyáshoz szükséges aggregált költségadatokra.

---

## 12. Hozzáférési szintek (javasolt)

| Szerep | Olvasási jog | Írási jog | Megjegyzés |
|---|---|---|---|
| **Munkavállaló** | Saját `PayslipDocument` és `PayrollRecord` összesített adatok | Nincs | GDPR 15. cikk – hozzáférési jog; Mt. 155. § – elszámoló lap kiadása kötelező |
| **Bérszámfejtő** | Az általa kezelt szervezeti egység összes `PayrollRecord`, `PayrollItem`, `TaxCalculation` | `PayrollRun` indítás, `PayrollItem` rögzítés zárolás előtt, `PayrollCorrection` létrehozás | Feladatellátáshoz szükséges teljes hozzáférés |
| **Vezető bérszámfejtő** | Szervezet teljes bérszámfejtési adata | `PayrollCycle` kezelés, `PayrollRun` zárolás, `PayrollCorrection` jóváhagyás | Magas szintű jogosultság – az összes bérszámfejtési lépés jóváhagyója |
| **Pénzügyi vezető** | Összesített bérköltség-adatok (`totalGrossPay`, `totalEmployerCost`), `PaymentOrder` | `PaymentOrder` jóváhagyás | Csak aggregált és kifizetési adatok; egyéni bérszámfejtési laphoz nincs hozzáférés |
| **Közvetlen vezető** | Saját szervezeti egységre vonatkozó összesített bérköltség | Nincs | Nem látja az egyéni béreket; csak cost center szintű aggregátumokat |
| **HR ügyintéző** | Strukturális adatok (beosztás, munkaidő); `PayrollRecord` státusz | Nincs a bérszámfejtési adatra | Elkülönítés: a HR nem fér hozzá a konkrét béradatokhoz, hacsak bérszámfejtői jogkört is ellát |
| **Könyvvizsgáló (auditált hozzáférés)** | Csak olvasás, teljes Payroll domén, meghatározott időszakra | Nincs | Ideiglenes, audit-naplózott hozzáférés; zárt auditálási munkamenetben |
| **Rendszeradminisztrátor** | Technikai mezők | Technikai mezők | Bérszámfejtési értékekhez nincs hozzáférés |

---

## 13. Validációs szabályok

| Szabály | Validáció | Megjegyzés |
|---|---|---|
| `PayrollCycle.paymentDate` legkésőbb a tárgyhónapot követő 10. nap | Kötelező figyelmeztetés | Mt. 157. § (1) – törvényi határidő; nem hiba, de eltérés esetén figyelmeztessen |
| `PayrollRecord.netPay` ≥ 0 | Kötelező | Negatív nettó bér nem keletkezhet (ha a levonások meghaladnák a nettót, a rendszer jelezze) |
| `PayrollRecord.netPay` ≥ törvényi minimálbér × `snapshotWeeklyHours / 40` | Kötelező figyelmeztetés | Mt. 153. § – minimálbér-kötelezettség; részmunkaidőnél arányos; vendéglátóipari kivételek |
| `NET_DEDUCTION` tételek összege ≤ nettó bér × 33% (általános) / 50% (tartásdíj) | Kötelező | Mt. 161. § – bírósági letiltás korlátai |
| `TaxCalculation.ytdPersonalIncomeTaxWithheld` összhangban van a `PayrollItem` SZJA-sorokkal | Kötelező | Belső konzisztencia-ellenőrzés |
| `PayrollItem.amount` összege (GROSS_EARNING) = `PayrollRecord.grossPay` | Kötelező | Mérlegfőösszeg egyezőség |
| `PayrollRecord.totalEmployerCost` = `grossPay` + `employerSocialContribution` + egyéb munkáltatói terhek | Kötelező | |
| `PayrollCorrection.requiresNavCorrection = true` esetén `navCorrectionStatus` kötelezően kitöltendő | Kötelező | Art. 54. § – önellenőrzési kötelezettség nyomonkövetése |
| `under25Exemption` csak 25. életévét be nem töltött munkavállalónál | Kötelező | Szja tv. 29/D. § – életkori feltétel; születési dátum a Person entitásból |
| `ekhoTax` csak `EKHO` jogviszony-típusnál | Kötelező | 2005. évi CXX. tv. – az EKHO alkalmazhatóságának feltételei |
| `PaymentOrder.amount` = érintett `PayrollRecord.payableAmount` összege | Kötelező | Belső konzisztencia |
| `snapshotIsPensioner = true` esetén `employeeSocialContribution = 0` | Kötelező | Tbj. 6. § – nyugdíjas foglalkoztatott nem fizet TB járulékot |

---

## 14. Integrációs pontok

### 14.1. Belső domének

| Forrás domén | Irány | Érintett adatok | Leírás |
|---|---|---|---|
| **Employment** | bejövő | `employmentType`, `weeklyHours`, `isPensioner`, `isDisabled` | Jogviszony-specifikus adójellemzők, pillanatkép-adatok |
| **Compensation (CompensationElement)** | bejövő | Rendszeres bérelemek, pótlékok, juttatások | Havi bérelemek automatikus `PayrollItem` generálása |
| **Compensation (BenefitEntitlement)** | bejövő | Cafeteria, SZÉP-kártya juttatások | Béren kívüli juttatások elszámolása |
| **Compensation (OneTimePayment)** | bejövő | Jutalom, prémium, végkielégítés, jubileumi jutalom | Egyszeri kifizetési tételek |
| **TimeTracking (DailyTimeRecord)** | bejövő | Ledolgozott napok, órák, hiányzások | Arányos bérszámítás alapja, távolléti levonások |
| **TimeTracking (OvertimeRecord)** | bejövő | Elrendelt, teljesített, kompenzált túlóraórák | Túlóradíj `PayrollItem` generálásához |
| **TimeTracking (OnCallRecord)** | bejövő | Ügyeleti, készenléti idő | Ügyeleti/készenléti díj |
| **Leave (LeaveRecord)** | bejövő | Szabadság, betegszabadság, táppénz | Távolléti díj, táppénz-kiegészítés számításához |
| **Person** | bejövő | Születési dátum, TAJ, adóazonosító | Adókedvezmény jogosultság (25 alatti, 30 alatti anya); NAV bevallás |
| **Document** | kimenő | `PayslipDocument.storageLocation` | Bérpapírok archiválása a Document doménben |
| **PerformanceReview** | bejövő | `overallRating`, `regulatoryOutcome` | Teljesítményprémium jogalapja |

### 14.2. Külső rendszerek

| Külső rendszer | Irány | Érintett adatok | Cél |
|---|---|---|---|
| **NAV (08-as havi bevallás)** | kimenő | SZJA előleg, TB járulék, szocho összesítők adózónként | Art. 50. § – havi kötelező adóbevallás; tárgyhónapot követő 12-éig |
| **NAV (T1041 biztosítotti bejelentés)** | kimenő | TAJ, jogviszony kezdete/vége, jogviszony típusa | Tbj. – biztosítotti jogviszony bejelentése/lezárása |
| **NAV (önellenőrzés)** | kimenő | `PayrollCorrection` adatai | Art. 54. § – visszamenőleges hiba javítása |
| **MÁK (Kincstár)** | kimenő | Közszféra bérszámfejtési adatok | Kincstáron keresztüli bérfolyósítás (közszféra kötelező) |
| **NEAK** | kimenő | TAJ, táppénz-alapadatok | Egészségbiztosítási ellátások igénylése |
| **Banki rendszer** | kimenő | `PaymentOrder` adatai (SEPA XML, CIB, OTP formátumok) | Nettó bér és NAV-befizetés utalása |
| **SZÉP-kártya szolgáltatók** (OTP, MKB, K&H) | kimenő | `BenefitEntitlement` SZÉP-kártya tételek | SZÉP-kártya-feltöltési megbízások |
| **KSH (OSAP)** | kimenő | Aggregált bérstatisztika (anonimizált) | Statisztikai adatszolgáltatás |

---

## 15. Kapcsolódó entitások (hivatkozás)

```
PayrollCycle 1 ──── N PayrollRun            (ciklus → futtatások)

PayrollRun 1 ──── N PayrollRecord           (futtatás → egyéni lapok)
PayrollRun 1 ──── N PaymentOrder            (futtatás → átutalási megbízások)

PayrollRecord 1 ──── N PayrollItem          (lap → bérszámfejtési sorok)
PayrollRecord 1 ──── 1 TaxCalculation       (lap → adószámítás részletei)
PayrollRecord 1 ──── N PayslipDocument      (lap → bérpapírok: havi + éves)
PayrollRecord 1 ──── N PayrollCorrection    (lap ← korrekciók)

PayrollItem N ──── 1 CompensationType       (sor → bérelem típus)

Employment 1 ──── N PayrollRecord           (jogviszony → bérszámfejtési lapok)
Person 1 ──── N PayrollRecord               (személy → bérszámfejtési lapok, minden jogviszonyon)

PayrollCorrection N ──── 1 PayrollRecord    (korrekció → eredeti lap)
PayrollCorrection N ──── 1 PayrollRun       (korrekció → korrekciós futtatás)
```

---

## 16. Nyitott kérdések

1. **CentralParameter entitás szükségessége:** A bérszámfejtési számítások paraméterei (minimálbér, garantált bérminimum, SZJA-kulcs 15%, TB 18,5%, szocho 13%, adókedvezmény-összegek, stb.) jogszabályban rögzített, de évente/félévenként változó értékek. Ezeket `CentralParameter` törzsadat-entitásban kellene tárolni, pillanatkép-mechanizmussal (`centralParameterSnapshotId`). Ennek az entitásnak a tervezése a Payroll domén előfeltétele – ha nincs meg, az összes adószámítás „beégetett" konstansokkal dolgozik, ami jogszabályváltozásnál hibaforrás.

2. **MÁK integráció mélysége:** A közszféra bérszámfejtése (Kjt., Kttv., Kit., Púétv., Eszjtv.) az MÁK Kincstáron keresztül zajlik – a MÁK saját bérszámfejtő rendszert (Forrás) üzemeltet, amellyel integrálni kell. Ez befolyásolja, hogy a Payroll domén önálló számítást végez-e közszféra jogviszonynál, vagy az HRMS csak adatszolgáltatóként funkcionál a MÁK felé.

3. **Cafeteria és SZÉP-kártya elszámolása:** A cafeteria rendszer komplex: a SZÉP-kártya béren kívüli juttatás (adómentesen adható összeghatárig), az összeghatár feletti rész adóköteles természetbeni juttatás. Az elszámolás éves keretből havi lebontásban működik. Kérdés: a cafeteria éves kerettárgyalás és a havi SZÉP-kártya-feltöltés teljesen a Payroll doménben kezelendő-e, vagy a Compensation domén `BenefitEntitlement` entitása a gazdája, és a Payroll csak az elszámolási oldalt látja?

4. **Párhuzamos jogviszonyok nettó bér letiltási korlátja:** Ha ugyanannak a személynek két jogviszonya van ugyanannál a munkáltatónál, a Mt. 161. § szerinti bírósági letiltási korlát a teljes nettó jövedelemre vonatkozik – nem jogviszonyanként. A rendszernek képesnek kell lennie a Person szintű összesítésre a letiltási korlát ellenőrzésekor. Ez a `PayrollRecord`-ok Person-szintű összesítését igényli a validációban.

5. **Visszamenőleges besorolásmódosítás automatizálása:** A Kjt. és Kttv. fokozatlépések visszamenőleges hatályúak (pl. 2025. január 1-jével jár, de csak márciusban kerül rögzítésre). A különbözet számítása januártól márciusig terjedő korrekciókat igényel – ez 3 `PayrollCorrection` + 3 korrekciós `PayrollRun` + 3 NAV önellenőrzés. Kérdés: ezt a folyamatot automatizálja-e a rendszer (kaszkád korrekció), vagy manuálisan kell minden hónapot külön kezelni?

6. **Éves adóelszámolás (munkáltatói adómegállapítás):** A Szja tv. 44. §-a lehetővé teszi, hogy a munkavállaló a munkáltatóhoz nyújtsa be az éves adóelszámolását. Ilyenkor a munkáltató az éves adóbevallást elkészíti és benyújtja. Ez az éves `PayrollCycle` (`isYearEnd = true`) keretein belül kezelhető, de önálló workflow-t (munkavállalói nyilatkozat begyűjtése, ellenőrzés, benyújtás) igényel – ennek az entitásmodell-igénye még nem vizsgált.

7. **EKHO kezelése:** Az egyszerűsített közteherviselési hozzájárulás (2005. évi CXX. tv.) sajátos adózási mód, amely helyettesíti az SZJA-t és a járulékokat meghatározott tevékenységekre (pl. sportoló, előadóművész, edző). Ha az HRMS érintett szektort (sport, kultúra) is kiszolgál, az EKHO teljes számítási logikáját (pl. EKHO-s és nem EKHO-s jövedelem elválasztása, adókulcs, bevallási sajátosságok) ki kell dolgozni.

8. **Külföldi munkavállalók és kettős adóztatás:** Más EU-tagállam vagy harmadik ország állampolgára esetén a kettős adóztatás elkerüléséről szóló egyezmények befolyásolják az SZJA és TB kezelését. A `TaxCalculation` entitásba bekerülhet egy `appliedTaxTreaty` mező, de a tényleges implementáció (melyik egyezmény, milyen feltétellel) komplex és jogász által validálandó.
