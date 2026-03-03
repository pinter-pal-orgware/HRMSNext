# Compensation (Javadalmazás) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer  
> **Verzió:** 0.1 – első tervezet  
> **Utolsó frissítés:** 2025. február  
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md, employment-entity.md

---

## 1. Az entitás célja és hatóköre

A **Compensation** entitáscsalád a foglalkoztatási jogviszonyhoz (Employment) kapcsolódó javadalmazási elemeket tartalmazza: az illetmény/munkabér megállapításának alapját, a pótlékokat, juttatásokat, jutalmakat és egyéb díjazási elemeket. A Compensation nem a tényleges bérszámfejtés (payroll) entitása – az a havi elszámolás szintje –, hanem a **javadalmazási jogosultságok és paraméterek** nyilvántartása, amelyek alapján a bérszámfejtés történik.

**Tervezési alapelvek:**

- **Elemi építőkövek:** A javadalmazás nem egyetlen összeg, hanem **elemekből** áll össze (alapilletmény + pótlékok + juttatások + jutalmak + levonások). Minden elem önálló rekord saját jogalappal, érvényességgel és számítási logikával.
- **Jogviszony-típus vezérelt:** Az alkalmazható javadalmazási elemek köre és számítási módja a jogviszony típusától függ (Mt. → szabad megállapodás; Kjt./Púétv./Eszjtv. → illetménytáblázat + szabályozott pótlékok; Kit./Kttv. → sávos illetmény).
- **Temporális modell:** Minden javadalmazási elem érvényességi időszakkal rendelkezik. Egy adott hónapban az érvényes elemek összessége adja a bruttó javadalmazást.
- **Elválasztás az Employment-től:** Az Employment tartalmazza a besorolási adatokat (fokozat, kategória), amelyekből a Compensation elemek **levezethetők**. Az Employment a „miért", a Compensation a „mennyi és mettől meddig".

---

## 2. Entitásstruktúra – áttekintés

```
Employment 1 ──── N CompensationElement    (javadalmazási elemek)
Employment 1 ──── N BenefitEntitlement     (béren kívüli juttatás-jogosultságok)
Employment 1 ──── N OneTimePayment         (egyszeri kifizetések: jutalom, jubileum, végkielégítés)

CompensationElement ──── 1 CompensationType (elem típusa, számítási logika)
```

A három alenentitás szétválasztásának indoka:

| Alentitás | Jelleg | Példák |
|---|---|---|
| **CompensationElement** | Rendszeres, havi szintű, az illetmény/bér részét képezi | Alapilletmény, vezetői pótlék, nyelvpótlék, műszakpótlék, illetménykiegészítés |
| **BenefitEntitlement** | Béren kívüli juttatás-jogosultság, nem része az illetménynek | Cafeteria keret, étkezési hozzájárulás, utazási támogatás, lakhatási támogatás |
| **OneTimePayment** | Eseti, nem rendszeres kifizetés | Jubileumi jutalom, végkielégítés, prémium, céljuttatás, köznevelési foglalkoztatotti jutalom |

---

## 3. CompensationElement (Rendszeres javadalmazási elem)

### 3.1. Azonosítók és alapadatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | Technikai |
| `employmentId` | UUID (FK) | igen | Hivatkozás az Employment entitásra | Technikai |
| `compensationTypeId` | UUID (FK) | igen | Hivatkozás a CompensationType törzsadatra | Technikai |
| `validFrom` | date | igen | Érvényesség kezdete | – |
| `validTo` | date | nem | Érvényesség vége (null = határozatlan) | – |
| `status` | enum | igen | Állapot: `ACTIVE`, `SUSPENDED`, `ENDED` | – |

### 3.2. Összeg és számítási mód

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `calculationMethod` | enum | igen | Számítási mód (lásd 3.3.) | – |
| `amount` | decimal | feltételes | Fix összeg (Ft/hó) – ha `calculationMethod` = `FIXED` | – |
| `percentage` | decimal | feltételes | Százalékos mérték – ha `calculationMethod` = `PERCENTAGE` | – |
| `percentageBase` | enum | feltételes | A százalék alapja (BASE_SALARY, GUARANTEED_SALARY, MINIMUM_WAGE, GUARANTEED_WAGE_MINIMUM) | – |
| `hourlyRate` | decimal | feltételes | Óradíj – ha `calculationMethod` = `HOURLY` | – |
| `calculatedMonthlyAmount` | decimal | nem | Számított havi összeg (a rendszer tölti, tájékoztató jellegű) | – |
| `currency` | string(3) | igen | Pénznem (ISO 4217, alapértelmezés: `HUF`) | – |

### 3.3. Számítási módok (calculationMethod)

| Kód | Leírás | Tipikus használat |
|---|---|---|
| `FIXED` | Fix havi összeg | Alapilletmény (Kjt., Púétv., Eszjtv.); fix pótlék |
| `PERCENTAGE` | Százalékos, valamilyen bázisérték alapján | Vezetői pótlék (az alapilletmény %-a); illetménykiegészítés |
| `HOURLY` | Óradíj alapú | Többlettanítási óradíj; rendkívüli munkaidő díjazása |
| `TABLE_LOOKUP` | Táblázatból kikeresett érték (besorolási adatok alapján) | Kjt. garantált illetmény; Púétv. illetménytáblázat; Eszjtv. illetmény |
| `FORMULA` | Számított képlet alapján | Műszakpótlék (óradíj × pótlékszázalék × ledolgozott órák); ügyeleti díj |
| `DAILY_RATE` | Napi díj alapú | Napidíj (kiküldetés) |

### 3.4. Jogszabályi hivatkozás és indoklás

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `legalBasis` | string | nem | Jogszabályi hivatkozás (pl. "Kjt. 70. § (1) a)"; "Mt. 143. §") |
| `legalReference` | string | nem | Részletes jogszabályi szöveghivatkozás |
| `justification` | string | nem | Munkáltatói indoklás (különösen sávos illetménynél, eltérítésnél) |
| `approvedBy` | string | nem | Jóváhagyó személy |
| `approvedAt` | datetime | nem | Jóváhagyás dátuma |

### 3.5. Audit mezők

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `createdAt` | datetime | igen | Rekord létrehozása |
| `createdBy` | string | igen | Létrehozó |
| `updatedAt` | datetime | igen | Utolsó módosítás |
| `updatedBy` | string | igen | Módosító |
| `previousElementId` | UUID | nem | Előző elem azonosítója (változáskövetés láncolt listával) |

---

## 4. CompensationType (Javadalmazási elem típus – törzsadat)

A rendszerben előre definiált javadalmazási elem típusok, amelyek meghatározzák az elem viselkedését.

### 4.1. Property-k

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `id` | UUID | igen | PK |
| `code` | string | igen | Egyedi kód (pl. `BASE_SALARY`, `KJT_LEADER_ALLOWANCE`, `PUETV_SUBJECT_ALLOWANCE`) |
| `name` | string | igen | Megnevezés (pl. "Alapilletmény", "Vezetői pótlék", "Tantárgyi illetménynövekedés") |
| `category` | enum | igen | Kategória (lásd 4.2.) |
| `applicableEmploymentTypes` | array[enum] | igen | Mely jogviszony-típusoknál alkalmazható |
| `isTaxable` | boolean | igen | Adóköteles-e (Szja) |
| `isSocialContribBase` | boolean | igen | Szocho/járulékalap része-e |
| `isPartOfBaseSalary` | boolean | igen | Az alapilletmény/bér részét képezi-e (szabadságszámítás, végkielégítés alapja) |
| `isRecurring` | boolean | igen | Rendszeres (havonta járó) vagy eseti |
| `defaultCalculationMethod` | enum | igen | Alapértelmezett számítási mód |
| `minValue` | decimal | nem | Jogszabályi minimum (ha van) |
| `maxValue` | decimal | nem | Jogszabályi maximum (ha van) |
| `legalBasis` | string | nem | Jogszabályi alap |
| `isActive` | boolean | igen | Aktív törzsadat-e |

### 4.2. Kategóriák

| Kód | Megnevezés | Leírás |
|---|---|---|
| `BASE` | Alapilletmény / alapbér | A jogviszony alapjául szolgáló díjazás |
| `SALARY_SUPPLEMENT` | Illetménykiegészítés | Végzettséghez, munkakörcsoporthoz kötött kiegészítés (Kjt., Kttv.) |
| `SALARY_DEVIATION` | Illetményeltérítés | Teljesítményértékelés alapján (Púétv. ±20%) |
| `ALLOWANCE_ROLE` | Munkakör / megbízás alapú pótlék | Vezetői pótlék, osztályfőnöki pótlék, intézményvezetői pótlék |
| `ALLOWANCE_QUALIFICATION` | Képesítés alapú pótlék | Nyelvpótlék, képesítési pótlék, mesterfokozat, tantárgyi növekedés |
| `ALLOWANCE_WORKING_CONDITION` | Munkakörülmény alapú pótlék | Műszakpótlék, éjszakai pótlék, készenléti pótlék, ügyeleti díj, veszélyességi pótlék |
| `ALLOWANCE_OVERTIME` | Rendkívüli munkaidő díjazása | Túlóra pótlék, többlettanítási óradíj, önként vállalt többletmunka díja |
| `ALLOWANCE_OTHER` | Egyéb rendszeres pótlék | Esélyteremtési illetményrész, nemzetiségi pótlék, differenciált pótlék |

---

## 5. Javadalmazási elem katalógus jogviszony-típusonként

### 5.1. Mt. – Munkaviszony (versenyszféra)

| Elem kód | Megnevezés | Számítás | Jogszabály |
|---|---|---|---|
| `MT_BASE_SALARY` | Alapbér (személyi alapbér) | Fix összeg – a munkaszerződésben megállapított; min. a minimálbér vagy garantált bérminimum | Mt. 45. § (1); Korm. rendelet a minimálbérről |
| `MT_OVERTIME_SUPPLEMENT` | Túlóra pótlék | 50% (alapeset) vagy 100% (pihenőnap, munkaszüneti nap) az alapbér arányos részéhez | Mt. 143. § |
| `MT_HOLIDAY_SUPPLEMENT` | Munkaszüneti napi pótlék | 100% pótlék rendkívüli munkaidőben | Mt. 143. § (5) – 2025-ös egyértelműsítés |
| `MT_NIGHT_SUPPLEMENT` | Éjszakai pótlék | 15% az alapbér arányos részéhez | Mt. 142. § |
| `MT_SHIFT_SUPPLEMENT` | Műszakpótlék | Délutáni: 30%; éjszakai: 15% | Mt. 141. § |
| `MT_STANDBY_PAY` | Készenlét díjazása | 20% az alapbér arányos részéhez | Mt. 144. § |
| `MT_ON_CALL_PAY` | Ügyelet díjazása | 40% az alapbér arányos részéhez | Mt. 144. § |

**Megjegyzések:**
- Az Mt.-ben a pótlékok **diszpozitívak**: kollektív szerződés vagy a felek megállapodása eltérhet (de a munkavállaló hátrányára csak kollektív szerződés).
- Az alapbér szabad megállapodás kérdése, de nem lehet kevesebb a minimálbérnél (szakképzettséget nem igénylő munkakör) vagy a garantált bérminimumal (szakképzettséget igénylő munkakör).

---

### 5.2. Kjt. – Közalkalmazotti jogviszony

| Elem kód | Megnevezés | Számítás | Jogszabály |
|---|---|---|---|
| `KJT_GUARANTEED_SALARY` | Garantált illetmény | Táblázatból: fizetési osztály (A–J) × fokozat (1–14) | Kjt. 66. § + illetménytáblázat (évente Korm. rendelettel módosított) |
| `KJT_SALARY_SUPPLEMENT` | Illetménykiegészítés | Ágazati vhr. szerint, általában a pótlékalap %-a vagy fix összeg | Kjt. 66. § + ágazati vhr.-ek (pl. 257/2000. Korm. r. – szociális) |
| `KJT_LEADER_ALLOWANCE` | Vezetői pótlék | Magasabb vezető: a pótlékalap 200–300%-a; vezető: 100–200%-a | Kjt. 70. § (1) |
| `KJT_TITLE_ALLOWANCE` | Címpótlék | Ágazat-specifikus | Kjt. 71. § |
| `KJT_DANGEROUS_WORK` | Veszélyességi pótlék | A pótlékalap egyes %-a | Ágazati vhr.-ek |
| `KJT_OVERTIME` | Rendkívüli munkaidő díjazása | Mt. szabályai szerint (Kjt. → Mt.-re utal) | Kjt. → Mt. 143. § |
| `KJT_NIGHT_SUPPLEMENT` | Éjszakai pótlék | Mt. szabályai szerint | Kjt. → Mt. 142. § |
| `KJT_SHIFT_SUPPLEMENT` | Műszakpótlék | Mt. szabályai szerint | Kjt. → Mt. 141. § |

**Megjegyzések:**
- A Kjt.-ben az illetmény táblázatos: a fizetési osztály és fokozat **egyértelműen meghatározza** a garantált illetményt. A munkáltató ennél magasabb illetményt állapíthat meg, de alacsonyabbat nem.
- A **pótlékalap** összege évente kormányrendelettel kerül megállapításra – a rendszernek ezt központi paraméterként kell kezelnie.

---

### 5.3. Púétv. – Köznevelési foglalkoztatotti jogviszony

| Elem kód | Megnevezés | Számítás | Jogszabály |
|---|---|---|---|
| `PUETV_BASE_SALARY` | Illetmény (pedagógus-illetménytáblázat) | Táblázatból: fokozat × fizetési kategória; évente Korm. rendelet | Púétv. 98. § + éves Korm. rendelet (pl. 453/2024.) |
| `PUETV_SALARY_DEVIATION` | Illetményeltérítés | ±20% a munkáltató döntése alapján (teljesítményértékelés) – 2025.09.01-től | Púétv. 99. § |
| `PUETV_OPPORTUNITY_ALLOWANCE` | Esélyteremtési illetményrész | Fix összeg – hátrányos helyzetű tanulókat nevelő intézmények pedagógusai | Púétv. 100. § |
| `PUETV_MASTER_DEGREE_INCREASE` | Mesterfokozat utáni illetménynövekedés | Fix összeg vagy % | Púétv. 101. § |
| `PUETV_SUBJECT_INCREASE` | Tantárgy utáni illetménynövekedés | Egyes hiányszakmák, nehézségi kategória alapján | Púétv. 101. § |
| `PUETV_CAREER_START_INCREASE` | Pályakezdés utáni illetménynövekedés | Pályakezdő pedagógusnak járó kiegészítés | Púétv. |
| `PUETV_EXTRA_TEACHING_HOURLY` | Többlettanítási óradíj | Havi illetmény / havi kötelező óraszám | Púétv. 104. § |
| `PUETV_EXTRA_TEACHING_COMBINED` | Többlettanítási óradíj (összevont csoport) | A fenti 50%-a | Púétv. 104. § |
| `PUETV_LEADER_ALLOWANCE` | Intézményvezetői pótlék | Az illetmény %-a, intézménymérettől függően | Púétv. 106. § |
| `PUETV_HEAD_TEACHER` | Osztályfőnöki pótlék | Fix összeg vagy az illetmény %-a | Púétv. 106. § |
| `PUETV_MENTOR_ALLOWANCE` | Mentori pótlék (gyakornok mentorálása) | Fix összeg | Púétv. |
| `PUETV_NATIONALITY_ALLOWANCE` | Nemzetiségi pótlék | Differenciált pótlék nemzetiségi oktatásban | Púétv. 107. § |
| `PUETV_INCENTIVE_SUPPLEMENT` | Ösztönzési keresetkiegészítés | Munkáltató döntése, éves költségvetés | Púétv. |

**Megjegyzések:**
- A pedagógus illetmény **kötött**: a táblázatból adódó összeg + jogszabályban meghatározott növekedések. Az illetményeltérítés (±20%) 2025 szeptemberétől alkalmazható először.
- A **2025-ös béremelés** (453/2024. Korm. r.) fokozatonként eltérő mértékű, ezt a táblázatkeresés logikának kell tükröznie.

---

### 5.4. Eszjtv. – Egészségügyi szolgálati jogviszony

| Elem kód | Megnevezés | Számítás | Jogszabály |
|---|---|---|---|
| `ESZJTV_BASE_SALARY` | Illetmény (fizetési fokozat alapján) | Táblázatból: munkakör-kategória × fizetési fokozat | Eszjtv. 8–9. § + Korm. rendelet |
| `ESZJTV_ON_CALL_PAY` | Ügyeleti díj | Ügyeleti óradíj × ledolgozott ügyeleti órák (beosztás szerinti keret terhére) | 528/2020. Korm. r.; 3/2023. OKFŐ utasítás 7–8. pont |
| `ESZJTV_ON_CALL_VOLUNTARY` | Ügyeleti díj (önként vállalt többletmunka) | Ügyeleti díj + 20% többlet | 3/2023. OKFŐ utasítás 11. pont |
| `ESZJTV_STANDBY_PAY` | Készenléti díj | A rendkívüli munkavégzés díjazás mértéke szerint | Eszjtv.; 528/2020. Korm. r. |
| `ESZJTV_VOLUNTARY_OVERTIME` | Önként vállalt többletmunka díja (nem ügyelet) | Rendkívüli munkavégzés pótléka + 50% | 3/2023. OKFŐ utasítás 12. pont |
| `ESZJTV_QUALIFICATION_ALLOWANCE` | Képesítési pótlék | További szakképesítés / doktori fokozat, ha a munkaidő min. 25%-ában hasznosul | 3/2023. OKFŐ utasítás 13–16. pont |
| `ESZJTV_LEADER_ALLOWANCE` | Vezetői juttatás | Intézménymérettől függő felső határ (50–150 eFt) | 3/2023. OKFŐ utasítás 17. pont |
| `ESZJTV_SHIFT_SUPPLEMENT` | Műszakpótlék | Mt. alapján + Eszjtv. sajátosságok | Eszjtv. → Mt. 141. § |
| `ESZJTV_NIGHT_SUPPLEMENT` | Éjszakai pótlék | Mt. alapján | Eszjtv. → Mt. 142. § |
| `ESZJTV_INFECTIOUS_ALLOWANCE` | Fertőzésveszélyességi pótlék | Ágazati szabályok | Ágazati rendeletek |

**Megjegyzések:**
- Az egészségügyi bérrendszer **2021-es bevezetésekor** háromfázisú béremelés történt. A jelenlegi illetménytáblázatot a végrehajtási rendelet tartalmazza.
- Az ügyeleti díjazás rendszere különösen komplex: a beosztás szerinti keret terhére végzett ügyelet és az önként vállalt többletmunka keretében végzett ügyelet eltérő díjszabással működik.

---

### 5.5. Kit./Kttv./Küt. – Kormányzati és közszolgálat

| Elem kód | Megnevezés | Számítás | Jogszabály |
|---|---|---|---|
| `GOV_BASE_SALARY` | Illetmény (sávon belül megállapított) | A besorolási kategória illetménysávjának alsó és felső határa között a munkáltató állapítja meg | Kit. 114–118. §; Kttv. 132–133. §; Küt. |
| `GOV_SALARY_SUPPLEMENT` | Illetménykiegészítés | Felsőfokú végzettség: az alapilletmény bizonyos %-a | Kttv. 134. § |
| `GOV_LANGUAGE_ALLOWANCE` | Nyelvpótlék | Komplex felsőfokú: az illetményalap 100%-a; egyéb fokozatok: 50–70% | Kttv. 141. § |
| `GOV_PERFORMANCE_BONUS` | Teljesítményértékeléshez kötött jutalom | Éves, a teljesítményértékelés eredménye alapján | Kit.; Kttv. |
| `GOV_LEADER_ALLOWANCE` | Vezetői illetménypótlék | Vezetői szinttől függően | Kit.; Kttv.; Küt. |
| `GOV_OVERTIME` | Rendkívüli munkavégzés díjazása | Mt. szabályai szerint, eltérésekkel | Kit. → Mt.; Kttv. → Mt. |
| `GOV_STANDBY_PAY` | Készenléti/ügyeleti díj | Mt. szabályai szerint | Kit.; Kttv. |

---

## 6. BenefitEntitlement (Béren kívüli juttatás-jogosultság)

### 6.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `employmentId` | UUID (FK) | igen | Hivatkozás az Employment-re | Technikai |
| `benefitTypeId` | UUID (FK) | igen | Juttatás típusa (törzsadat) | Technikai |
| `annualBudget` | decimal | feltételes | Éves keret (Ft) | Szja tv. 71. § – béren kívüli juttatások éves kerete |
| `monthlyBudget` | decimal | feltételes | Havi keret (Ft) | – |
| `validFrom` | date | igen | Érvényesség kezdete | – |
| `validTo` | date | nem | Érvényesség vége | – |
| `usedAmount` | decimal | nem | Felhasznált összeg (számított mező) | – |
| `remainingAmount` | decimal | nem | Fennmaradó összeg (számított mező) | – |
| `status` | enum | igen | `ACTIVE`, `SUSPENDED`, `ENDED` | – |
| `createdAt` | datetime | igen | Létrehozás | – |
| `createdBy` | string | igen | Létrehozó | – |
| `updatedAt` | datetime | igen | Módosítás | – |
| `updatedBy` | string | igen | Módosító | – |

### 6.2. Tipikus juttatás-típusok (BenefitType törzsadat)

| Kód | Megnevezés | Adózás | Jogszabály |
|---|---|---|---|
| `CAFETERIA_SZEP_SZALLASHELY` | SZÉP-kártya szálláshely alszámla | Kedvezményes adózás (éves limitig) | Szja tv. 71. § |
| `CAFETERIA_SZEP_VENDEGLATAS` | SZÉP-kártya vendéglátás alszámla | Kedvezményes adózás | Szja tv. 71. § |
| `CAFETERIA_SZEP_SZABADIDO` | SZÉP-kártya szabadidő alszámla | Kedvezményes adózás | Szja tv. 71. § |
| `CAFETERIA_OTHER` | Egyéb cafeteria elem | Egyes meghatározott juttatás – kedvezményes | Szja tv. 71. § |
| `MEAL_ALLOWANCE` | Étkezési hozzájárulás (nem SZÉP) | Adóköteles juttatás (Szja tv. 70. §) felett | Szja tv. 70–71. § |
| `COMMUTING_ALLOWANCE` | Utazási költségtérítés | Adómentes (39/2010. Korm. r.) | Mt. 51. § (2); 39/2010. Korm. r. |
| `HOUSING_SUPPORT` | Lakhatási támogatás | Adómentes limitig | Szja tv. 1. sz. melléklet |
| `VOLUNTARY_PENSION` | Önkéntes nyugdíjpénztári hozzájárulás | Kedvezményes | Szja tv. 71. § |
| `HEALTH_FUND` | Egészségpénztári hozzájárulás | Kedvezményes | Szja tv. 71. § |
| `LIFE_INSURANCE` | Munkáltatói életbiztosítás | Kedvezményes limitig | Szja tv. 71. § |
| `EDUCATION_SUPPORT` | Képzési hozzájárulás | Adómentes feltételekkel | Szja tv. 1. sz. melléklet |
| `CHILD_CARE_SUPPORT` | Bölcsődei/óvodai hozzájárulás | Adómentes feltételekkel | Szja tv. 1. sz. melléklet |

**Megjegyzések:**
- A **cafeteria** keretösszeg és annak allokációja az egyes alszámlákra a munkáltató belső szabályzata szerint történik; a közszférában jellemzően Korm. rendelet vagy belső szabályzat határozza meg.
- Az **utazási költségtérítés** (munkába járás) a munkáltató kötelező térítése (Mt. 51. § (2) + 39/2010. Korm. rendelet) – nem opcionális juttatás, hanem jogszabályi kötelezettség. A mértéke napidíjhoz vagy bérletárhoz kötött.
- Az adózási szabályok évente változhatnak – a BenefitType törzsadatban az adózási paramétereket éves szinten kell karbantartani.

---

## 7. OneTimePayment (Egyszeri kifizetés)

### 7.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `employmentId` | UUID (FK) | igen | Hivatkozás az Employment-re | Technikai |
| `paymentType` | enum | igen | Kifizetés típusa (lásd 7.2.) | – |
| `amount` | decimal | igen | Összeg (Ft, bruttó) | – |
| `paymentDate` | date | igen | Kifizetés dátuma / esedékesség | – |
| `reason` | string | nem | Indoklás | – |
| `legalBasis` | string | nem | Jogszabályi alap | – |
| `calculationDetails` | JSON | nem | Számítás részletei (pl. hány év × milyen összeg) | – |
| `status` | enum | igen | `PLANNED`, `APPROVED`, `PAID`, `CANCELLED` | – |
| `approvedBy` | string | nem | Jóváhagyó | – |
| `approvedAt` | datetime | nem | Jóváhagyás dátuma | – |
| `paidAt` | datetime | nem | Tényleges kifizetés dátuma | – |
| `createdAt` | datetime | igen | Létrehozás | – |
| `createdBy` | string | igen | Létrehozó | – |
| `updatedAt` | datetime | igen | Módosítás | – |
| `updatedBy` | string | igen | Módosító | – |

### 7.2. Kifizetés-típusok

| Kód | Megnevezés | Számítás | Jogszabály |
|---|---|---|---|
| `KJT_JUBILEE_25` | Jubileumi jutalom – 25 év (Kjt.) | 2 havi illetmény | Kjt. 78. § (1) |
| `KJT_JUBILEE_30` | Jubileumi jutalom – 30 év (Kjt.) | 3 havi illetmény | Kjt. 78. § (2) |
| `KJT_JUBILEE_35` | Jubileumi jutalom – 35 év (Kjt.) | 5 havi illetmény | Kjt. 78. § (3) |
| `KJT_JUBILEE_40` | Jubileumi jutalom – 40 év (Kjt.) | 5 havi illetmény | Kjt. 78. § (4) |
| `PUETV_SERVICE_AWARD` | Köznevelési foglalkoztatotti jutalom | A Púétv. szerinti mérték, szakmai gyakorlati idő alapján | Púétv. 108. § |
| `SEVERANCE_PAY` | Végkielégítés | Mt. 77. § táblázat: 1–6 havi távolléti díj a jogviszony hosszától függően | Mt. 77. §; Kjt.; Púétv. |
| `TERMINATION_COMPENSATION` | Felmentési/felmondási időre járó távolléti díj (munkavégzés alóli felmentés idejére) | Távolléti díj × felmentett napok száma | Mt. 70. § (3) |
| `BONUS` | Jutalom / prémium | Munkáltatói döntés; vagy teljesítményértékeléshez kötött | Mt.; Kjt.; Kit.; Kttv. |
| `TARGET_BONUS` | Céljuttatás | Meghatározott feladat teljesítéséhez kötött | Kjt.; Kit.; Kttv. |
| `PUETV_TEACHING_BONUS` | Köznevelési foglalkoztatotti jutalom (Púétv.) | Éves, teljesítményértékeléshez is köthető | Púétv. |
| `GOV_PERFORMANCE_BONUS` | Teljesítményjutalom (közszolgálat) | Teljesítményértékelés eredménye alapján | Kit.; Kttv. |
| `SIGNING_BONUS` | Belépési bónusz / letelepedési támogatás | Egyedi megállapodás; Eszjtv. – visszatérítendő/vissza nem térítendő támogatás | Eszjtv. 3. § |
| `RELOCATION_SUPPORT` | Átköltözési támogatás | Egyedi megállapodás | Eszjtv.; Mt. |

---

## 8. Minimálbér és garantált bérminimum kezelése

### 8.1. Központi paraméterek (éves szinten karbantartandó)

| Paraméter | Leírás | Frissítési ciklus |
|---|---|---|
| `MINIMUM_WAGE` | Minimálbér (Ft/hó, teljes munkaidő) | Évente (Korm. rendelet, általában nov–dec-ben jelenik meg a következő évre) |
| `GUARANTEED_WAGE_MINIMUM` | Garantált bérminimum (Ft/hó, teljes munkaidő) | Évente |
| `MINIMUM_WAGE_HOURLY` | Minimálbér órabére | Számított: havi / 174 óra |
| `GUARANTEED_WAGE_MINIMUM_HOURLY` | Garantált bérminimum órabére | Számított |
| `KJT_SUPPLEMENT_BASE` | Kjt. pótlékalap | Évente (Korm. rendelet) |
| `PUETV_SALARY_TABLE` | Púétv. illetménytáblázat | Évente (Korm. rendelet) |
| `ESZJTV_SALARY_TABLE` | Eszjtv. illetménytáblázat | Korm. rendelettel módosítva |

### 8.2. Validációs szabályok

| Szabály | Leírás | Jogszabály |
|---|---|---|
| Minimálbér-ellenőrzés | Mt.-s munkaviszonyban az alapbér nem lehet kevesebb a minimálbérnél (szakképzettséget nem igénylő munkakör) vagy garantált bérminimumal (szakképzettséget igénylő) – részmunkaidőnél arányosítva | Mt. 153. § |
| Kjt. garantált illetmény | A közalkalmazott illetménye nem lehet kevesebb a fizetési osztály × fokozat szerinti garantált illetménynél | Kjt. 66. § |
| Púétv. illetmény-minimum | A pedagógus illetménye nem lehet kevesebb az illetménytáblázat szerinti összegnél; az eltérítés max. –20% | Púétv. 98–99. § |
| Eszjtv. illetmény-minimum | A fizetési fokozat szerinti illetmény garantált | Eszjtv. 8–9. § |
| Kit./Kttv. sávminimum | Az illetmény nem lehet kevesebb a besorolási kategória illetménysávjának alsó határánál | Kit. 114. § |

---

## 9. Historikus adatkezelés

### 9.1. CompensationElement historikussága

Minden CompensationElement **érvényességi időszakkal** rendelkezik (`validFrom`, `validTo`). Amikor egy elem megváltozik (pl. béremelés, új pótlék összeg), a korábbi elem lezáródik (`validTo` = változás napja – 1) és új elem jön létre (`validFrom` = változás napja). Így az adott időszakra vonatkozó javadalmazás **bármikor rekonstruálható**.

### 9.2. Tipikus változási események

| Esemény | Hatás a Compensation-re | Jogszabály |
|---|---|---|
| Éves béremelés (versenyszféra) | `MT_BASE_SALARY` lezárása + új elem magasabb összeggel | Munkáltatói döntés |
| Éves illetménytáblázat-módosítás (Kjt./Púétv./Eszjtv.) | `*_BASE_SALARY` újraszámítása az új táblázat alapján | Korm. rendelet |
| Fokozatlépés (Kjt., Eszjtv.) | Automatikus: új fizetési fokozat → új garantált illetmény → új CompensationElement | Kjt. 62. §; Eszjtv. 8. § |
| Pedagógus-fokozatváltás (Púétv.) | Minősítés eredménye → új fokozat → új illetménytáblázat-sor | Púétv. 96–97. § |
| Illetményeltérítés megállapítása (Púétv.) | Új `PUETV_SALARY_DEVIATION` elem szept. 1.–aug. 31. érvényességgel | Púétv. 99. § |
| Vezetői megbízás | Új `*_LEADER_ALLOWANCE` elem | Minden jogállási tv. |
| Teljes → részmunkaidő váltás | Összes arányosítható elem újraszámítása | Mt. 92. § |
| Minimálbér-emelés | Érintett `MT_BASE_SALARY` elemek felülvizsgálata, szükség esetén emelése | Mt. 153. § |

---

## 10. GDPR adatkezelési kategorizáció

| Kategória | Property-k | Jogalap | Megőrzési idő |
|---|---|---|---|
| **Illetmény/bér adatok** | CompensationElement (összes) | Szerződés (6(1)b) + Jogi kötelezettség (6(1)c) | Jogviszony megszűnése + 8 év (Szt. – számviteli bizonylat) |
| **Béren kívüli juttatások** | BenefitEntitlement (összes) | Szerződés (6(1)b) + Jogi kötelezettség (6(1)c – adózás) | Jogviszony megszűnése + 8 év |
| **Egyszeri kifizetések** | OneTimePayment (összes) | Jogi kötelezettség (6(1)c) + Szerződés (6(1)b) | Jogviszony megszűnése + 8 év |
| **Jóváhagyási adatok** | approvedBy, approvedAt | Elszámoltathatóság (GDPR 5(2)) | Az érintett kifizetés megőrzési idejével megegyező |
| **Központi paraméterek** | Minimálbér, táblázatok, pótlékalap | Nem személyes adat | Korlátlan (referencia) |

**Megjegyzések:**
- A **8 éves** megőrzési idő a számviteli törvényből (Szt. 169. §) fakad: a számviteli bizonylatokat a bizonylat keltétől számított 8 évig kell megőrizni.
- A **közszférában** a javadalmazási adatok egy része **közérdekű adat** (vezetők illetménye, juttatásai) → Info tv. szerinti közzétételi kötelezettség.
- A bérjegyzék (payslip) adatai a Compensation entitásból és a bérszámfejtés eredményéből állnak össze – az HRMS és a payroll modul közötti interfész kérdés.

---

## 11. Hozzáférési szintek

| Szerep | CompensationElement | BenefitEntitlement | OneTimePayment |
|---|---|---|---|
| **Érintett munkavállaló** | Saját: olvasás (összeg, típus, érvényesség) | Saját: olvasás + keret felhasználás | Saját: olvasás |
| **Közvetlen vezető** | **Nincs hozzáférés** (kivéve: ha a szervezet bérezési politikája megengedi, és DPIA készült) | Nincs | Jóváhagyás (jutalom) – összeg nélkül? |
| **HR ügyintéző** | Teljes olvasás/írás a hatásköri szervezeti egységen belül | Teljes olvasás/írás | Teljes olvasás/írás |
| **Bérszámfejtő** | Teljes olvasás; módosítás jóváhagyás után | Teljes olvasás | Teljes olvasás |
| **Pénzügyi vezető** | Aggregált/statisztikai hozzáférés | Aggregált | Aggregált |
| **Rendszeradminisztrátor** | Központi paraméterek karbantartása (minimálbér, táblázatok) | BenefitType törzsadat | PaymentType törzsadat |

---

## 12. Integrációs pontok

| Külső rendszer | Irány | Érintett adatok | Cél |
|---|---|---|---|
| **Bérszámfejtő modul (payroll)** | kimenő | Összes érvényes CompensationElement + BenefitEntitlement + esedékes OneTimePayment | Havi bérszámfejtés input |
| **NAV (08-as bevallás)** | kimenő (payroll-on keresztül) | Bruttó bér, járulékalapok, adóelőleg | Havi bevallás |
| **MÁK (kincstár)** | kétirányú | Illetmény, pótlékok (költségvetési szervek) | Költségvetési bérszámfejtés |
| **SZÉP-kártya szolgáltató** | kimenő | Cafeteria allokáció az alszámlákra | Feltöltési megbízás |
| **Önkiszolgáló portál** | bejövő | Cafeteria választás, SZÉP-kártya alszámla elosztás | Munkavállaló által kezdeményezett allokáció |
| **Kontrolling / BI** | kimenő | Aggregált javadalmazási adatok | Bérköltség-elemzés, tervezés |

---

## 13. Kapcsolódó entitások – teljes kép

```
Employment 1 ──── N CompensationElement        (rendszeres javadalmazási elemek)
Employment 1 ──── N BenefitEntitlement          (béren kívüli juttatások)
Employment 1 ──── N OneTimePayment              (egyszeri kifizetések)

CompensationElement N ──── 1 CompensationType   (elem típus törzsadat)
BenefitEntitlement  N ──── 1 BenefitType        (juttatás típus törzsadat)

CompensationType ──── CentralParameter          (minimálbér, pótlékalap, illetménytáblázat)
```

---

## 14. Nyitott kérdések

1. **Illetmény vs. Compensation elhelyezése:** Az Employment entitáson is szerepelnek besorolási adatok (pl. `kjt_guaranteedSalary`, `puetv_baseSalary`). Ezek redundánsak a CompensationElement-tel? Javaslat: az Employment tartalmazza a **besorolási alapot** (fokozat, kategória), a CompensationElement tartalmazza a **tényleges összeget és érvényességet**. A kettő közötti konzisztenciát validációval biztosítjuk.
2. **Természetbeni juttatások:** Az étkezés, szolgálati lakás, gépjárműhasználat, telefon stb. a BenefitEntitlement része, vagy önálló entitás?
3. **Levonások kezelése:** A munkabérből történő levonások (letiltás, tartásdíj, kártérítés, szakszervezeti tagdíj) a Compensation entitáscsaládba tartoznak, vagy külön Deduction entitás?
4. **Bértömeg-tervezés:** A rendszer támogassa-e a tervezési funkciókat (pl. "mi lenne ha" szcenáriók béremelésre, létszámváltozásra)?
5. **Visszamenőleges korrekció:** Hogyan kezeljük, ha utólag kiderül, hogy egy korábbi hónap javadalmazása hibás volt? A CompensationElement korrekciója vagy külön korrekciós rekord?
6. **Közfoglalkoztatotti bér:** A közfoglalkoztatottak bére eltér a minimálbértől (alacsonyabb) – szükséges-e külön CompensationType?
7. **Pedagógus béremelés automatizálása:** A 2025-ös és várhatóan éves béremelések (Korm. rendelet) automatikus végrehajtása a rendszerben: tömegesen kell-e tudni frissíteni a CompensationElement-eket, vagy egyénileg?
