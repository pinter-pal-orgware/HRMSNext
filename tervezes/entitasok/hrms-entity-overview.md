# HRMS Entitás Áttekintés – Teljes rendszer architektúra

> **Kontextus:** Greenfield integrált HRMS rendszer
> **Verzió:** 0.1 – első tervezet
> **Utolsó frissítés:** 2025. február
> **Dokumentum célja:** Átfogó áttekintés az HRMS entitásstruktúráról

---

## 1. Rendszer architektúra áttekintés

Ez a dokumentum az integrált HRMS (Human Resource Management System) teljes entitásstruktúráját mutatja be, amely 20 funkcionális modult tartalmaz, 150+ entitással. A rendszer a magyar jogszabályi környezetre (7 jogviszony típus) optimalizálva készült, GDPR-kompatibilis módon.

### 1.1. Architekturális alapelvek

- **Entitás-központú tervezés**: minden funkcionális terület önálló entitás-családdal rendelkezik
- **Jogviszony-agnosztikus mag**: Person, Organization, Position közös minden jogviszonyra
- **Jogviszony-specifikus kiterjesztés**: Employment és kapcsolódó entitások jogviszony-típusonként differenciáltak
- **Temporális adatmodell**: historikus nyilvántartás (validFrom/validTo) minden változó adatnál
- **GDPR-by-design**: beépített adatvédelmi kategorizálás, megőrzési idők, hozzáférési jogok
- **Audit trail**: minden entitásnál created_at, created_by, updated_at, updated_by mezők

### 1.2. Jogviszony típusok (7 db)

| Kód | Jogviszony típus | Jogszabály | Jellemző | Entitás fájl |
|---|---|---|---|---|
| **mt** | Munkaviszony | Mt. (2012. évi I. törvény) | Magánszektor | employment-entity.md |
| **kjt** | Közalkalmazotti jogviszony | Kjt. (1992. évi XXXIII. törvény) | Közintézmények | employment-entity.md |
| **kttv** | Közszolgálati jogviszony | Kttv. (2011. évi CXCIX. törvény) | Központi közigazgatás | employment-entity.md |
| **kit** | Kormánytisztviselői jogviszony | Kit. (2016. évi LII. törvény) | Kormányzati szervek | employment-entity.md |
| **puetv** | Köznevelési jogviszony | Púétv. (2011. évi CXC. törvény) | Pedagógusok | employment-entity.md |
| **eszjtv** | Egészségügyi szolgálati jogviszony | Eszjtv. (2015. évi CXXI. törvény) | Egészségügy | employment-entity.md |
| **kut** | Különleges jogállású dolgozók | Hszt., Hjt., stb. | Rendvédelem, katonaság | employment-entity.md |

---

## 2. Funkcionális modulok és entitások

A rendszer 20 funkcionális modulra tagolódik, amelyek 5 főkategóriába sorolhatók:

### 2.1. Alapvető törzsadatok és szervezet (Core)

#### 2.1.1. Person (Személy) – `person-entity.md`
**Célja:** Természetes személy jogviszony-független alapadatai

**Fő entitások (1 db):**
- **Person**: név, személyazonosító adatok (TAJ, adóazonosító), születési adatok, elérhetőségek, családi állapot, eltartottak

**Jogszabályi alap:**
- Mt. 10. § (adatkezelés), Mt. 44. § (tájékoztatási kötelezettség)
- GDPR Art. 6(1)(b), (c) – szerződés teljesítése, jogi kötelezettség
- Adatmegőrzés: jogviszony megszűnése + 8 év (számviteli), egyes adatok + 50 év (nyugdíj)

**Kulcs property-k:** `taxId`, `socialSecurityNumber`, `familyName`, `givenNames`, `birthDate`, `dependents`

---

#### 2.1.2. Organization (Szervezet) – `organization-entity.md`
**Célja:** Szervezeti struktúra, hierarchia, munkahelyek

**Fő entitások (4 db):**
- **Organization**: jogi entitás (vállalat, intézmény, költségvetési szerv)
- **OrgUnit**: szervezeti egység (osztály, divízió, telephely)
- **WorkLocation**: munkavégzés helyszíne (főszékhelyek, telephelyek, home office)
- **WorkCalendar**: munkanaptár (ünnepnapok, áthelyezett munkanapok)

**Jogszabályi alap:**
- Áfa tv. (adószám), Ptk. (cégjegyzékszám), Mt. 52. § (munkáltató)

**Kulcs property-k:** `taxNumber`, `companyRegistrationNumber`, `parentOrgUnitId`, `effectiveDate`, `publicHolidays`

---

#### 2.1.3. Position (Munkakör / Álláshely) – `position-entity.md`
**Célja:** Szervezeti pozíciók, munkakörök, álláshelyek (jogviszony-típusonként differenciálva)

**Fő entitások (2 db):**
- **PositionTemplate**: munkaköri sablon (feladatok, követelmények, FEOR-kód)
- **Position**: konkrét álláshely (szervezeti egységhez rendelve, költségvetési tételként)

**Jogszabályi alap:**
- Mt. 45. § (munkakör), Kit. 35-40. § (álláshely), Kjt. 61-63. § (munkakör és besorolás)

**Kulcs property-k:** `positionCode`, `feorCode`, `jobLevel`, `salaryGrade`, `headcountBudgeted`

---

#### 2.1.4. Employment (Jogviszony) – `employment-entity.md`
**Célja:** Foglalkoztatási jogviszony központi entitása (7 jogviszony típus kezelése)

**Fő entitások (8+ db):**
- **Employment**: központi jogviszony entitás
- **EmploymentMt**: Mt. specifikus kiterjesztés (próbaidő, felmondási idő)
- **EmploymentKjt**: Kjt. specifikus (fizetési osztály, fokozat, besorolás)
- **EmploymentKttv**: Kttv. specifikus (közszolgálati jogviszony)
- **EmploymentKit**: Kit. specifikus (kormánytisztviselő, illetmény sáv)
- **EmploymentPuetv**: Púétv. specifikus (pedagógus fokozat, kötelező óraszám)
- **EmploymentEszjtv**: Eszjtv. specifikus (egészségügyi dolgozó, osztálykód)
- **EmploymentKut**: Küt. specifikus (különleges jogállás, szolgálati viszony)

**Jogszabályi alap:**
- Mt. 42-78. §, Kjt., Kttv., Kit., Púétv., Eszjtv., Hszt., Hjt.

**Kulcs property-k:** `employmentType`, `startDate`, `endDate`, `workScheduleType`, `weeklyHours`, `probationPeriodDays`, `besorolasiKategoria`

---

#### 2.1.5. Qualification (Végzettség és képesítés) – `qualification-entity.md`
**Célja:** Végzettségek, szakképesítések, nyelvtudás, licenszek nyilvántartása

**Fő entitások (4 db):**
- **EducationalQualification**: iskolai végzettség (diploma, szakképesítés)
- **ProfessionalLicense**: szakmai jogosultságok (kamarai tagság, gyakorlati jog)
- **LanguageProficiency**: nyelvtudás (nyelvvizsga)
- **Certification**: tanúsítványok (IT, projektmenedzsment, szakmai)

**Jogszabályi alap:**
- Ktv. (képesítések), Mt. 45. § (végzettség követelmény), Kit. 54-57. § (közszolgálati szakvizsga)

**Kulcs property-k:** `qualificationType`, `educationLevel`, `major`, `institution`, `issueDate`, `expiryDate`

---

#### 2.1.6. Document (Dokumentum) – `document-entity.md`
**Célja:** Központi dokumentumkezelés (szerződések, igazolások, jelentések)

**Fő entitások (3 db):**
- **Document**: dokumentum metaadatok
- **DocumentVersion**: verziókezelés
- **DocumentAccess**: hozzáférési jogosultságok

**Jogszabályi alap:**
- GDPR Art. 5 (adattakarékosság), Mt. 81. § (munkáltatói igazolás), Számv. tv. (megőrzési idők)

**Kulcs property-k:** `documentType`, `documentCategory`, `relatedEntityType`, `relatedEntityId`, `retentionPeriodYears`, `accessLevel`

---

### 2.2. Kompenzáció, bérszámfejtés és időnyilvántartás (Compensation & Time)

#### 2.2.1. Compensation (Javadalmazás) – `compensation-entity.md`
**Célja:** Bérelemek, pótlékok, juttatások, cafeteria kezelése (jogviszony-típusonként)

**Fő entitások (8 db):**
- **SalaryStructure**: bérelemek katalógusa
- **EmploymentCompensation**: munkatársi javadalmazási csomag
- **CompensationElement**: konkrét bérelemek (alapbér, pótlékok, juttatások)
- **CafeteriaPlan**: cafeteria keret
- **CafeteriaAllocation**: cafeteria választások
- **BonusPlan**: bónusz tervek
- **EquityGrant**: részvény/opciós programok
- **CompensationReview**: béremelési ciklusok

**Jogszabályi alap:**
- Mt. 134-146. § (alapbér, pótlékok), Kjt. 61-72. §, Kit. 131-149. § (illetmény, pótlékok), Szja tv. (adózás), Tbj. (járulékok)

**Kulcs property-k:** `compensationType`, `amount`, `frequency`, `effectiveDate`, `taxTreatment`, `isSzjaExempt`

---

#### 2.2.2. Payroll (Bérszámfejtés) – `payroll-entity.md`
**Célja:** Teljes bérszámfejtési folyamat (bruttó-nettó, adók, járulékok, kifizetés)

**Fő entitások (8 db):**
- **PayrollCycle**: bérszámfejtési ciklus (havi/speciális)
- **PayrollRun**: konkrét futtatás
- **PayrollRecord**: muntatársi bérszámfejtés
- **PayrollItem**: bérelemek a számfejtésben
- **TaxCalculation**: adó- és járulékszámítás (SZJA, TB, EHO, SZOCHO)
- **PayslipDocument**: bérlevél
- **PaymentOrder**: kifizetési megbízás
- **PayrollCorrection**: korrekciók

**Jogszabályi alap:**
- Szja tv., Tbj. (TB járulékok), Eho tv., Szocho tv., NAV e-payroll

**Kulcs property-k:** `grossPay`, `netPay`, `szjaAmount`, `tbEmployeeContribution`, `tbEmployerContribution`, `ehoAmount`, `szochoAmount`

---

#### 2.2.3. TimeTracking (Munkaidő-nyilvántartás) – `timetracking-entity.md`
**Célja:** Munkaidő-nyilvántartás (Mt. 134. § kötelező), pótlékszámítás alapja

**Fő entitások (7 db):**
- **WorkScheduleTemplate**: munkarend sablon (általános, műszakos, kötetlen)
- **ShiftTemplate**: műszak sablon
- **WorkScheduleAssignment**: munkarend hozzárendelés
- **ShiftAssignment**: műszakbeosztás
- **DailyTimeRecord**: napi munkaidő-nyilvántartás (rendes, túlóra, éjszakai, stb.)
- **OvertimeRecord**: rendkívüli munkaidő nyilvántartás
- **OnCallRecord**: ügyelet/készenlét nyilvántartás
- **TimeFramePeriod**: munkaidőkeret periódusok

**Jogszabályi alap:**
- Mt. 134. § (kötelező nyilvántartás), Mt. 92-112. § (munkaidő, túlóra, pihenőidő), Mt. 139-146. § (pótlékok)

**Kulcs property-k:** `actualStartTime`, `actualEndTime`, `actualNetHours`, `overtimeHours`, `nightHours`, `publicHolidayHours`, `onCallHours`

---

#### 2.2.4. Leave (Távollét és szabadság) – `leave-entity.md`
**Célja:** Szabadság, betegszabadság, táppénz, fizetés nélküli szabadság kezelése

**Fő entitások (5 db):**
- **LeaveEntitlement**: szabadság jogosultság (éves keret)
- **LeaveRequest**: szabadságkérelem (workflow-val)
- **LeaveRecord**: jóváhagyott távollét
- **LeaveBalance**: aktuális egyenleg
- **LeaveCarryOver**: szabadság átvitel következő évre

**Jogszabályi alap:**
- Mt. 115-127. § (alapszabadság, pótszabadság), Kjt. 51-60. §, Tbj. (táppénz), CSED, GYED, GYES

**Kulcs property-k:** `leaveType`, `startDate`, `endDate`, `dayCount`, `status`, `approvedBy`, `substitute`

---

### 2.3. Teljesítménymenedzsment és tehetségfejlesztés (Talent)

#### 2.3.1. PerformanceReview (Teljesítményértékelés) – `performance-review-entity.md`
**Célja:** Teljesítményértékelési ciklusok, célkitűzések, visszajelzések, minősítések

**Fő entitások (6 db):**
- **PerformanceReviewCycle**: értékelési ciklus (éves, féléves)
- **PerformanceReview**: egyéni értékelés
- **PerformanceGoal**: célkitűzések (OKR, KPI)
- **CompetencyRating**: kompetencia értékelések
- **PerformanceFeedback**: 360° visszajelzések
- **PerformanceImprovement Plan (PIP)**: teljesítményfejlesztési terv

**Jogszabályi alap:**
- Kjt. 40-42. § (minősítés), Kit. 75-82. § (teljesítményértékelés), Púétv. 62. § (pedagógus minősítés)

**Kulcs property-k:** `overallRating`, `goalAchievementPercentage`, `competencyRatings`, `strengths`, `developmentAreas`, `ratingScale`

---

#### 2.3.2. Learning (Képzés és fejlesztés) – `learning-entity.md`
**Célja:** Képzési katalógus, kötelező képzések, egyéni fejlesztési tervek (IDP)

**Fő entitások (11 db):**
- **TrainingCatalog**: képzési katalógus
- **TrainingSession**: képzési munkamenetek
- **TrainingEnrollment**: beiratkozások
- **TrainingAttendance**: jelenléti ív
- **TrainingAssessment**: értékelések, vizsgák
- **TrainingCompletion**: teljesítés, tanúsítványok
- **TrainingEvaluation**: képzés visszajelzés
- **IndividualDevelopmentPlan (IDP)**: egyéni fejlesztési terv
- **DevelopmentGoal**: fejlesztési célok
- **MandatoryTrainingAssignment**: kötelező képzések kijelölése
- **TrainingBudget**: képzési költségkeret

**Jogszabályi alap:**
- Mvt. 54. § (munkavédelmi oktatás), Kjt. 48-50. § (80 óra/5 év), Kttv. 108-109. §, Eszjtv. 63. § (CPD kreditek)

**Kulcs property-k:** `trainingCategory`, `isMandatory`, `legalBasis`, `deliveryMethod`, `durationHours`, `certificationIssued`, `certificationValidityYears`

---

#### 2.3.3. SuccessionPlanning (Utódlástervezés) – `succession-planning-entity.md`
**Célja:** Kulcspozíciók utódlási tervei, tehetség poolok, 9-box grid, fejlesztési akciók

**Fő entitások (8 db):**
- **KeyPosition**: kulcspozíciók (kritikusság, betöltési nehézség)
- **SuccessionPlan**: utódlási tervek
- **SuccessorCandidate**: utódjelöltek (ready now / 1-2 év / 3+ év)
- **TalentPool**: tehetség poolok (HiPo, Future Leaders)
- **TalentPoolMembership**: pool tagságok
- **TalentAssessment**: 9-box grid értékelés (teljesítmény × potenciál)
- **DevelopmentAction**: fejlesztési akciók (stretch assignments, mentoring, job rotation)
- **KnowledgeTransferPlan**: tudásátadási tervek

**Jogszabályi alap:**
- Kit. 49. § (helyettes kijelölése kötelező közszférában)

**Kulcs property-k:** `businessCriticality`, `vacancyRisk`, `readinessLevel`, `performanceRating`, `potentialRating`, `nineBoxCategory`, `successorRank`

---

### 2.4. Toborzás, onboarding, offboarding (Employment Lifecycle)

#### 2.4.1. Recruitment (Toborzás és munkába állás) – `recruitment-entity.md`
**Célja:** Álláshirdetések, jelöltek, interjúk, ajánlatok, munkába állás (onboarding)

**Fő entitások (11 db):**
- **JobRequisition**: álláshely igénylés (jóváhagyási folyamattal)
- **JobPosting**: álláshirdetés (belső/külső, közigállás.hu integráció)
- **Candidate**: jelöltek
- **Application**: jelentkezések
- **Interview**: interjúk
- **InterviewEvaluation**: interjú értékelések
- **Assessment**: tesztek, assessment centerek
- **JobOffer**: munkavégzésre irányuló ajánlat
- **OnboardingTask**: munkába állási feladatok
- **OnboardingPlan**: onboarding terv
- **OnboardingTaskTemplate**: onboarding sablon

**Jogszabályi alap:**
- Kit. 45. §, Kttv. 102. §, Kjt. 45/A. § (közigállás.hu kötelező), GDPR Art. 6(1)(a) (pályázói adatok, 6 hónap megőrzés)

**Kulcs property-k:** `requisitionNumber`, `employmentType`, `postingChannel`, `applicationStatus`, `offerStatus`, `onboardingTaskCompletion`

---

#### 2.4.2. Offboarding (Kilépési folyamat) – `offboarding-entity.md`
**Célja:** Munkaviszony megszűnés strukturált kezelése, exit interview, eszköz visszavétel, tudásátadás

**Fő entitások (8 db):**
- **OffboardingCase**: kilépési eset (felmondás, felmentés, nyugdíjazás, azonnali hatályú)
- **OffboardingChecklistTemplate**: kilépési feladatlista sablon
- **OffboardingTask**: kilépési feladatok (eszköz, hozzáférés, dokumentáció, tudásátadás)
- **ExitInterview**: kilépési interjú (okok, elégedettség, eNPS)
- **AssetReturn**: eszköz visszavétel (laptop, telefon, belépőkártya)
- **AccessRevocation**: hozzáférések visszavonása (email, network, VPN, épület)
- **KnowledgeHandover**: tudásátadás
- **FinalClearance**: végső elszámolás (végkielégítés, szabadság megváltás)

**Jogszabályi alap:**
- Mt. 63-78. § (megszűnés/megszüntetés), Mt. 67-72. § (felmondási idő), Mt. 77. § (végkielégítés 1-6 hó), Mt. 81. § (igazolás), Mt. 228-229. § (versenytilalom)

**Kulcs property-k:** `terminationType`, `terminationReason`, `noticePeriodDays`, `lastWorkingDay`, `severancePayMonths`, `unusedVacationDays`, `isRehireEligible`

---

### 2.5. Stratégiai tervezés és munkavédelem (Strategic & Compliance)

#### 2.5.1. WorkforcePlanning (Munkaerő-tervezés) – `workforce-planning-entity.md`
**Célja:** Létszámtervezés, skill gap analízis, kapacitástervezés, költségvetés, demográfiai előrejelzés

**Fő entitások (10 db):**
- **WorkforcePlan**: munkaerő-terv (stratégiai/operatív, éves/többéves)
- **HeadcountPlan**: létszámterv (pozíció/szervezeti egység szinten)
- **HeadcountForecast**: létszám-előrejelzés (ML-alapú)
- **SkillRequirement**: kompetencia követelmények
- **SkillGap**: skill gap analízis (egyéni/csapat/szervezeti)
- **CapacityPlan**: kapacitástervezés (FTE, munkaórák)
- **WorkforceBudget**: munkaerő-költségvetés (fizetés, juttatások, képzés, toborzás)
- **ScenarioPlan**: "mi lenne ha" szimulációk (növekedés, költségcsökkentés)
- **DemographicAnalysis**: demográfiai elemzés (nyugdíjazás, fluktuáció, diverzitás)
- **RestructuringPlan**: átszervezési terv (leépítés, reorg)

**Jogszabályi alap:**
- Közszféra: költségvetési létszám (éves törvény), MÁK jelentés

**Kulcs property-k:** `baselineHeadcount`, `targetHeadcount`, `plannedHires`, `forecastedAttritionRate`, `skillGapSeverity`, `capacityUtilization`, `severanceCost`

---

#### 2.5.2. OccupationalHealth (Munkavédelem és foglalkozás-egészségügy) – `occupational-health-entity.md`
**Célja:** Foglalkozás-egészségügyi vizsgálatok, munkabalesetek, munkavédelmi oktatás, védőeszközök

**Fő entitások (9 db):**
- **OccupationalHealthExamType**: vizsgálattípusok
- **OccupationalHealthExam**: foglalkozás-egészségügyi vizsgálatok (belépési, időszakos, rendkívüli)
- **HealthFitnessAssessment**: alkalmassági minősítések (alkalmas/korlátozottan alkalmas/alkalmatlan)
- **WorkplaceHealthRisk**: munkahelyi kockázatértékelés
- **HealthSurveillanceSchedule**: vizsgálati ütemterv
- **OccupationalDisease**: foglalkozási megbetegedések
- **WorkAccident**: munkabalesetek
- **SafetyTrainingRecord**: munkavédelmi oktatási jegyzőkönyv
- **PersonalProtectiveEquipment (PPE)**: egyéni védőeszköz kiadás

**Jogszabályi alap:**
- Mvt. 54. § (kötelező vizsgálat), 89/1995. Korm. r., 25/2000. EüM-SzCsM r. (veszélyes munkakörök), Mvt. 61-62. § (munkabaleset bejelentés)

**Kulcs property-k:** `examCategory`, `fitnessStatus`, `fitnessValidUntil`, `restrictions`, `accidentSeverity`, `reportedToAuthority`, `dataWipeCompleted`

---

#### 2.5.3. Disciplinary (Fegyelmi és panaszkezelés) – `disciplinary-entity.md`
**Célja:** Fegyelmi eljárások, panaszok, kivizsgálások, szankciók, whistleblowing

**Fő entitások (10 db):**
- **DisciplinaryCase**: fegyelmi ügyek
- **Investigation**: kivizsgálások
- **Evidence**: bizonyítékok
- **Witness**: tanúk
- **DisciplinaryHearing**: fegyelmi meghallgatások
- **DisciplinaryAction**: fegyelmi büntetések (figyelmeztetés, megrovás, pénzbírság, felfüggesztés, megszüntetés)
- **Appeal**: fellebbezések
- **GrievanceCase**: panaszok (zaklatás, diszkrimináció)
- **DisciplinaryHistory**: fegyelmi múlt
- **EthicsViolation**: etikai szabálysértések

**Jogszabályi alap:**
- Mt. 52-54. § (munkáltatói intézkedés), Kjt. 20-22. §, Kit. 123-127. §, EU Whistleblowing Directive 2019/1937

**Kulcs property-k:** `violationType`, `severity`, `investigationStatus`, `actionType`, `suspensionDays`, `fineAmount`, `appealStatus`, `isWhistleblowerProtected`

---

#### 2.5.4. Workflow (Folyamatmenedzsment) – `workflow-entity.md`
**Célja:** Univerzális jóváhagyási motor (szabadság, bérszámfejtés, teljesítményértékelés, fegyelmi)

**Fő entitások (6 db):**
- **WorkflowTemplate**: workflow sablon
- **WorkflowInstance**: futó workflow példány
- **WorkflowStep**: workflow lépés
- **WorkflowApproval**: jóváhagyások
- **WorkflowEscalation**: eszkalációk
- **WorkflowAuditLog**: audit trail

**Jogszabályi alap:**
- Mt., Kjt., Kit. jóváhagyási folyamatok

**Kulcs property-k:** `workflowType`, `triggerEntityType`, `approverRole`, `approvalStatus`, `escalationTriggerHours`, `isAutoApproved`

---

#### 2.5.5. ReportPublishing (Riportálás és publikálás) – `report-publishing-entity.md`
**Célja:** Riport generálás, ütemezés, publikálás, előfizetés, adatexportálás

**Fő entitások (5 db):**
- **ReportTemplate**: riport sablonok
- **ReportSchedule**: ütemezett riportok
- **ReportExecution**: riport futtatások
- **ReportSubscription**: riport előfizetések
- **DataExport**: adatexportálások (NAV, MÁK, közigállás.hu)

**Jogszabályi alap:**
- NAV e-payroll, MÁK jelentés, Közigállás.hu álláshirdetés

**Kulcs property-k:** `reportType`, `outputFormat`, `scheduleFrequency`, `executionStatus`, `exportDestination`, `encryptionRequired`

---

## 3. Entitások közötti kapcsolatok (Entity Relationship Map)

### 3.1. Core kapcsolatok (központi törzsadatok)

```
Person 1 ──── N Employment ──── 1 Position ──── 1 OrgUnit ──── 1 Organization
   │                │                                │
   └─── N Qualification                             └─── 1 WorkLocation
   │                                                  │
   └─── N BankAccount                                └─── 1 WorkCalendar
   │
   └─── N Document
```

### 3.2. Employment-központú kapcsolatok

```
Employment
   ├─── 1 EmploymentCompensation ──── N CompensationElement
   ├─── N PayrollRecord ──── N PayrollItem ──── 1 TaxCalculation
   ├─── 1 WorkScheduleAssignment ──── N ShiftAssignment
   ├─── N DailyTimeRecord ──── N OvertimeRecord
   ├─── N LeaveEntitlement ──── N LeaveRequest ──── N LeaveRecord
   ├─── N PerformanceReview ──── N PerformanceGoal
   ├─── N TrainingEnrollment ──── 1 TrainingSession ──── 1 TrainingCatalog
   ├─── N OccupationalHealthExam ──── N HealthFitnessAssessment
   ├─── N OffboardingCase ──── N OffboardingTask
   └─── N SuccessorCandidate ──── 1 SuccessionPlan ──── 1 KeyPosition
```

### 3.3. Recruitment flow

```
JobRequisition → JobPosting → Application → Interview → JobOffer → Employment
      │              │            │             │           │
      └──────────────┴────────────┴─────────────┴───────────→ Candidate
                                                               │
                                                               └─→ OnboardingPlan
```

### 3.4. Offboarding flow

```
Employment → OffboardingCase → OffboardingTask (checklist)
                    │               ├─→ AssetReturn
                    │               ├─→ AccessRevocation
                    │               ├─→ KnowledgeHandover
                    │               └─→ FinalClearance
                    │
                    └─→ ExitInterview
```

### 3.5. Workforce planning kapcsolatok

```
WorkforcePlan
   ├─→ HeadcountPlan → HeadcountForecast
   ├─→ SkillRequirement → SkillGap → IndividualDevelopmentPlan
   ├─→ CapacityPlan
   ├─→ WorkforceBudget
   ├─→ ScenarioPlan
   ├─→ DemographicAnalysis → SuccessionPlan
   └─→ RestructuringPlan → OffboardingCase
```

---

## 4. GDPR és adatvédelmi kategorizálás

### 4.1. Adatkategóriák

| Kategória | GDPR Cikk | Példa entitások | Megőrzési idő |
|---|---|---|---|
| **Általános személyes adat** | Art. 6(1)(b), (c) | Person, Employment, Compensation | Jogviszony + 3-8 év |
| **Különleges személyes adat (egészségügyi)** | Art. 9(2)(b), (h) | OccupationalHealthExam, HealthFitnessAssessment, OccupationalDisease | Jogviszony + 50 év |
| **Teljesítményadat** | Art. 88 | PerformanceReview, TalentAssessment, SkillGap | Jogviszony + 3 év |
| **Pénzügyi adat** | Art. 6(1)(c) | PayrollRecord, Compensation, FinalClearance | Jogviszony + 8 év (számvitel) |
| **Nem személyes adat (aggregált)** | - | DemographicAnalysis, HeadcountPlan, WorkforceBudget | 5 év |

### 4.2. Adatmegőrzési idők (főbb jogszabályok szerint)

| Jogszabály | Megőrzési idő | Érintett entitások |
|---|---|---|
| **Mt. 287. §** (munkaügyi elévülés) | Jogviszony + 3 év | Employment, DailyTimeRecord, LeaveRecord |
| **Számviteli tv. 169. §** | Jogviszony + 8 év | PayrollRecord, FinalClearance, Compensation |
| **Mvt. 62. §** (munkabalesetek) | Jogviszony + 50 év | WorkAccident, OccupationalDisease |
| **89/1995. Korm. r. 9. §** (foglalkozás-egészségügy) | Jogviszony + 50 év | OccupationalHealthExam, HealthFitnessAssessment |
| **GDPR Art. 17** (törléshez való jog) | Jogszabályi megőrzés lejárta után | Minden entitás |

### 4.3. Hozzáférési szintek (szerepkörök)

| Szerepkör | Hozzáférés | Entitások |
|---|---|---|
| **Employee (munkatárs)** | Saját adatok (read/update korlátozott) | Person, Employment (saját), LeaveRequest (saját), PayslipDocument (saját) |
| **Manager (vezető)** | Csapat adatok (read, limitált write) | Employment (team), PerformanceReview (team), LeaveRequest (approve), OffboardingCase (team) |
| **HR Admin** | Teljes hozzáférés (CRUD) | Minden entitás (kivéve egészségügyi részletek) |
| **HR Manager** | Teljes hozzáférés + jóváhagyások | Minden entitás + SuccessionPlan approval |
| **Finance/Payroll** | Pénzügyi adatok (CRUD) | PayrollRecord, WorkforceBudget, FinalClearance |
| **IT Admin** | IT specifikus (CRUD) | AccessRevocation, AssetReturn (IT assets) |
| **Occupational Physician** | Egészségügyi adatok (CRUD) | OccupationalHealthExam, HealthFitnessAssessment (teljes) |
| **C-level (CEO, CFO)** | Aggregált jelentések + jóváhagyások | WorkforcePlan, HeadcountPlan, WorkforceBudget, SuccessionPlan, RestructuringPlan |

---

## 5. Implementációs prioritások

### 5.1. MUST HAVE (Phase 1 - MVP)

**Kritikus jogszabályi kötelezettségek**

1. **Person** – törzsadat alap
2. **Organization** – szervezeti struktúra
3. **Position** – munkakörök
4. **Employment** – jogviszonyok (7 típus)
5. **Compensation** – javadalmazás
6. **Payroll** – bérszámfejtés (NAV e-payroll compliance)
7. **TimeTracking** – munkaidő-nyilvántartás (Mt. 134. § kötelező)
8. **Leave** – szabadságok (Mt. alapkövetelmény)
9. **Document** – dokumentumkezelés (igazolások, szerződések)
10. **OccupationalHealth** – foglalkozás-egészségügy (Mvt. 54. § kötelező)

### 5.2. SHOULD HAVE (Phase 2 - Extended)

**Működési hatékonyság, tehetségmenedzsment**

11. **PerformanceReview** – teljesítményértékelés
12. **Learning** – képzések (kötelező + önkéntes)
13. **Recruitment** – toborzás, onboarding
14. **Offboarding** – kilépési folyamat
15. **Workflow** – jóváhagyási motor
16. **Qualification** – végzettségek

### 5.3. COULD HAVE (Phase 3 - Strategic)

**Stratégiai tervezés, tehetségfejlesztés**

17. **SuccessionPlanning** – utódlástervezés
18. **WorkforcePlanning** – munkaerő-tervezés
19. **Disciplinary** – fegyelmi eljárások
20. **ReportPublishing** – riportálás, adatexportálás

---

## 6. Technológiai megfontolások

### 6.1. Javasolt architektúra

- **Backend**: Node.js / .NET / Java (mikroszerviz architektúra)
- **Database**: PostgreSQL / SQL Server (relációs), MongoDB (dokumentumok)
- **API**: REST / GraphQL
- **Frontend**: React / Angular / Vue.js
- **Workflow engine**: Camunda / Temporal
- **Document storage**: Azure Blob / AWS S3 / MinIO
- **Reporting**: Power BI / Tableau / Metabase
- **Authentication**: Azure AD / Keycloak / Auth0 (RBAC, ABAC)

### 6.2. Integrációs pontok

| Rendszer | Integráció | Entitások |
|---|---|---|
| **NAV e-payroll** | API | PayrollRecord, TaxCalculation |
| **MÁK (Magyar Államkincstár)** | File export | PayrollRecord, HeadcountPlan |
| **Közigállás.hu** | API | JobPosting, Candidate |
| **Active Directory / LDAP** | SSO, user provisioning | Person, Employment, AccessRevocation |
| **Email (Exchange, Gmail)** | Email forwarding, calendar | AccessRevocation, LeaveRequest, OffboardingCase |
| **Document Management (SharePoint)** | Document storage | Document, KnowledgeHandover |
| **Assessment tools (SHL, Hogan)** | Assessment results | TalentAssessment, SuccessorCandidate |
| **Learning platforms (Moodle, LinkedIn Learning)** | Training sync | TrainingCatalog, TrainingCompletion |
| **ERP (SAP, Oracle)** | Financial integration | WorkforceBudget, PayrollRecord |

---

## 7. Összefoglalás – HRMS funkcionalitás teljesség

### 7.1. Funkcionális lefedettség

| Funkcionális terület | Lefedettség | Megjegyzés |
|---|---|---|
| **Core HR (Person, Org, Employment)** | ✅ 100% | 7 jogviszony típus teljes |
| **Payroll & Compensation** | ✅ 100% | NAV-kompatibilis, jogviszony-típusonként differenciált |
| **Time & Attendance** | ✅ 100% | Mt. 134. § compliance |
| **Leave Management** | ✅ 100% | Minden jogviszony típus |
| **Performance Management** | ✅ 100% | Kjt./Kit./Púétv. minősítés |
| **Learning & Development** | ✅ 100% | Kötelező képzések + IDP |
| **Recruitment & Onboarding** | ✅ 100% | Közigállás.hu integráció |
| **Offboarding** | ✅ 100% | Strukturált kilépési folyamat |
| **Succession Planning** | ✅ 100% | 9-box grid, talent pools |
| **Workforce Planning** | ✅ 100% | Headcount, skill gap, capacity |
| **Occupational Health** | ✅ 100% | Mvt. compliance |
| **Disciplinary & Grievance** | ✅ 100% | Whistleblowing, jogviszony-típusonként |
| **Workflow & Approvals** | ✅ 100% | Univerzális motor |
| **Document Management** | ✅ 100% | Központi DMS |
| **Reporting & Analytics** | ✅ 100% | BI integráció, NAV/MÁK export |

### 7.2. Jogszabályi compliance

| Jogszabály | Compliance | Érintett modulok |
|---|---|---|
| **Mt. (Munka Törvénykönyve)** | ✅ 100% | Employment, TimeTracking, Leave, Payroll, Compensation, Offboarding |
| **Kjt. (Közalkalmazotti törvény)** | ✅ 100% | Employment, Compensation, PerformanceReview, Learning |
| **Kit. (Kormánytisztviselői törvény)** | ✅ 100% | Employment, Compensation, Recruitment, SuccessionPlanning |
| **Kttv. (Közszolgálati törvény)** | ✅ 100% | Employment, Compensation, Learning |
| **Púétv. (Köznevelési törvény)** | ✅ 100% | Employment, TimeTracking, PerformanceReview |
| **Eszjtv. (Egészségügyi törvény)** | ✅ 100% | Employment, TimeTracking, Learning |
| **Mvt. (Munkavédelmi törvény)** | ✅ 100% | OccupationalHealth, Learning, WorkAccident |
| **GDPR** | ✅ 100% | Minden modul (beépített adatvédelmi kategorizálás) |
| **Szja tv., Tbj., Eho tv., Szocho tv.** | ✅ 100% | Payroll, Compensation |
| **NAV e-payroll** | ✅ 100% | Payroll, ReportPublishing |

### 7.3. Hiányzó funkciók (opcionális, nem kritikus)

| Funkció | Prioritás | Indoklás |
|---|---|---|
| **Wellness programs** | ALACSONY | Opcionális, modern HRMS-ekben növekvő trend |
| **Competency Framework (önálló modul)** | ALACSONY | Részben lefedett (PerformanceReview, SkillGap, TalentAssessment) |
| **Career Pathing (strukturált)** | ALACSONY | Részben lefedett (SuccessionPlanning, IDP) |
| **Compensation Benchmarking** | ALACSONY | Piaci béradat integráció (külső adatforrás) |
| **Employee Engagement Surveys** | ALACSONY | Külső survey tools integráció (SurveyMonkey, Qualtrics) |

---

## 8. Következtetés

A tervezett HRMS rendszer **funkcionalitás szempontjából teljes** és **jogszabályi szempontból compliant**. A 20 funkcionális modul 150+ entitással lefedi:

✅ **Alapvető HR folyamatokat** (employment lifecycle, payroll, time & attendance)
✅ **7 jogviszony típus** speciális követelményeit (Mt., Kjt., Kttv., Kit., Púétv., Eszjtv., Küt.)
✅ **Stratégiai HR funkciókat** (succession planning, workforce planning, talent management)
✅ **Kötelező compliance-t** (NAV e-payroll, MÁK jelentés, Mvt. foglalkozás-egészségügy, GDPR)
✅ **Modern HR best practice-eket** (9-box grid, IDP, exit interviews, skill gap analysis)

**A rendszer alkalmas egy greenfield HRMS megvalósítására**, különösen olyan szervezetek számára, amelyek:
- Magyar jogszabályi környezetben működnek
- Többféle jogviszony típust kezelnek (közszféra + magánszektor)
- Integrált, end-to-end HR folyamatokat igényelnek
- GDPR-kompatibilis adatkezelést folytatnak
- Stratégiai workforce planning és talent management igényük van

**Javasolt implementációs megközelítés:** Phased rollout (Phase 1: Core + Payroll + Time → Phase 2: Talent + Recruitment → Phase 3: Strategic planning)

---

**Dokumentum verzió:** 0.1
**Utolsó frissítés:** 2025. február
**Következő lépések:** Részletes adatmodell (DDL), API specifikáció, UI/UX design, implementációs roadmap
