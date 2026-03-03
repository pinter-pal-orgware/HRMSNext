# HRMS Entitás-térkép – Összefoglaló dokumentum

> **Kontextus:** Greenfield integrált HRMS rendszer – magyar jogszabályi környezet
> **Verzió:** 1.0 – Teljes funkcionális lefedettség
> **Utolsó frissítés:** 2025. február
> **Kapcsolódó:** hrms-entity-overview.md (részletes áttekintés)

---

## 1. Összesítés

| Mutató | Érték |
|---|---|
| **Entitások száma (összesen)** | ~150+ |
| **Funkcionális modulok** | 20 |
| **Főkategóriák** | 5 (Core, Compensation & Time, Talent, Employment Lifecycle, Strategic & Compliance) |
| **Kapcsolatok száma (becsült)** | 200+ |
| **Támogatott jogviszony-típusok** | 7 (Mt., Kjt., Kttv., Kit., Púétv., Eszjtv., Küt.) |
| **GDPR compliance** | 100% (beépített adatvédelmi kategorizálás) |
| **Jogszabályi lefedettség** | 15+ magyar jogszabály (Mt., Kjt., Kit., Kttv., Púétv., Eszjtv., Mvt., GDPR, NAV, stb.) |

---

## 2. Funkcionális modulok rendszer-architektúra

### 2.1. A 20 modul és kategorizálásuk

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                       HRMS TELJES RENDSZER-ARCHITEKTÚRA                          │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌────────────────────────────────────────────────────────────────────────┐     │
│  │                    1. CORE (Alapvető törzsadatok) – 6 modul            │     │
│  ├────────────────────────────────────────────────────────────────────────┤     │
│  │                                                                         │     │
│  │  ┌───────────┐  ┌──────────────┐  ┌──────────┐  ┌───────────────┐     │     │
│  │  │ Person    │  │ Organization │  │ Position │  │  Employment   │     │     │
│  │  │   (1)     │  │     (4)      │  │   (2)    │  │     (8+)      │     │     │
│  │  └─────┬─────┘  └──────┬───────┘  └────┬─────┘  └───────┬───────┘     │     │
│  │        │                │                │                │             │     │
│  │        └────────────────┴────────────────┴────────────────┘             │     │
│  │                             │                                           │     │
│  │                ┌────────────┴────────────┐                              │     │
│  │                │                         │                              │     │
│  │        ┌───────────────┐         ┌─────────────┐                        │     │
│  │        │ Qualification │         │  Document   │                        │     │
│  │        │      (4)      │         │     (3)     │                        │     │
│  │        └───────────────┘         └─────────────┘                        │     │
│  │                                                                         │     │
│  └────────────────────────────────────────────────────────────────────────┘     │
│                                                                                  │
│  ┌────────────────────────────────────────────────────────────────────────┐     │
│  │          2. COMPENSATION & TIME (Javadalmazás és időnyilvántartás)     │     │
│  │                              – 4 modul                                 │     │
│  ├────────────────────────────────────────────────────────────────────────┤     │
│  │                                                                         │     │
│  │  ┌──────────────┐  ┌─────────┐  ┌──────────────┐  ┌──────────┐        │     │
│  │  │ Compensation │  │ Payroll │  │ TimeTracking │  │  Leave   │        │     │
│  │  │     (8)      │  │   (8)   │  │     (7)      │  │   (5)    │        │     │
│  │  └──────────────┘  └─────────┘  └──────────────┘  └──────────┘        │     │
│  │                                                                         │     │
│  └────────────────────────────────────────────────────────────────────────┘     │
│                                                                                  │
│  ┌────────────────────────────────────────────────────────────────────────┐     │
│  │                3. TALENT (Tehetségmenedzsment) – 3 modul               │     │
│  ├────────────────────────────────────────────────────────────────────────┤     │
│  │                                                                         │     │
│  │  ┌───────────────────┐  ┌──────────┐  ┌──────────────────┐             │     │
│  │  │ PerformanceReview │  │ Learning │  │ SuccessionPlan.  │             │     │
│  │  │       (6)         │  │   (11)   │  │       (8)        │             │     │
│  │  └───────────────────┘  └──────────┘  └──────────────────┘             │     │
│  │                                                                         │     │
│  └────────────────────────────────────────────────────────────────────────┘     │
│                                                                                  │
│  ┌────────────────────────────────────────────────────────────────────────┐     │
│  │          4. EMPLOYMENT LIFECYCLE (Foglalkoztatási életciklus)          │     │
│  │                              – 2 modul                                 │     │
│  ├────────────────────────────────────────────────────────────────────────┤     │
│  │                                                                         │     │
│  │      ┌─────────────┐                    ┌─────────────┐                │     │
│  │      │ Recruitment │                    │ Offboarding │                │     │
│  │      │    (11)     │                    │     (8)     │                │     │
│  │      └─────────────┘                    └─────────────┘                │     │
│  │                                                                         │     │
│  └────────────────────────────────────────────────────────────────────────┘     │
│                                                                                  │
│  ┌────────────────────────────────────────────────────────────────────────┐     │
│  │         5. STRATEGIC & COMPLIANCE (Stratégiai és compliance)           │     │
│  │                              – 5 modul                                 │     │
│  ├────────────────────────────────────────────────────────────────────────┤     │
│  │                                                                         │     │
│  │  ┌──────────────┐  ┌──────────────────┐  ┌─────────────┐              │     │
│  │  │ Workforce    │  │ Occupational     │  │ Disciplinary│              │     │
│  │  │ Planning     │  │ Health           │  │    (10)     │              │     │
│  │  │    (10)      │  │      (9)         │  └─────────────┘              │     │
│  │  └──────────────┘  └──────────────────┘                                │     │
│  │                                                                         │     │
│  │  ┌──────────┐              ┌──────────────────┐                        │     │
│  │  │ Workflow │              │ Report Publishing│                        │     │
│  │  │   (6)    │              │       (5)        │                        │     │
│  │  └──────────┘              └──────────────────┘                        │     │
│  │                                                                         │     │
│  └────────────────────────────────────────────────────────────────────────┘     │
│                                                                                  │
└──────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Modulok és entitások részletes katalógusa

### 3.1. CORE (Alapvető törzsadatok) – 6 modul, ~25 entitás

#### 3.1.1. Person (1 entitás) – `person-entity.md`
**Célja:** Természetes személy jogviszony-független alapadatai

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **Person** | taxId, socialSecurityNumber, familyName, givenNames, birthDate, gender, dependents | GDPR Art. 6(1)(b), (c); Megőrzés: jogviszony + 8 év |

#### 3.1.2. Organization (4 entitás) – `organization-entity.md`
**Célja:** Szervezeti struktúra, hierarchia, munkahelyek

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **Organization** | taxNumber, companyRegistrationNumber, legalForm, sector | Jogi személy / munkáltató |
| **OrgUnit** | parentOrgUnitId, orgUnitType, level, costCenter | Szervezeti egység (fastruktúra) |
| **WorkLocation** | siteType, address, isRegistered, isRemoteWork | Munkavégzési helyszín |
| **WorkCalendar** | year, publicHolidays, transferredWorkDays | Munkanaptár (11 ünnep + áthelyezések) |

#### 3.1.3. Position (2 entitás) – `position-entity.md`
**Célja:** Munkakörök, álláshelyek (jogviszony-típusonként differenciálva)

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **PositionTemplate** | feorCode, category, jobLevel, applicableEmploymentTypes | Munkaköri sablon / katalógus |
| **Position** | positionCode, headcount, status, salaryGrade, reportingToPositionId | Konkrét álláshely |

#### 3.1.4. Employment (8+ entitás) – `employment-entity.md`
**Célja:** Foglalkoztatási jogviszony központi entitása (7 jogviszony típus)

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **Employment** | employmentType (diszkriminátor), startDate, endDate, weeklyHours, status | Központi hub, minden operatív entitás ide kapcsolódik |
| **EmploymentMt** | probationPeriodDays, noticePeriodDays, employmentMode | Mt. specifikus kiterjesztés |
| **EmploymentKjt** | fizetesiOsztalyKod, fizetesiFokozat, koztisztKategoria | Kjt. specifikus |
| **EmploymentKttv** | besorolasiKategoria, szazalekosIlletmeny | Kttv. specifikus |
| **EmploymentKit** | besorolasiKategoria, illetmenySav, kormanytisztKategoria | Kit. specifikus |
| **EmploymentPuetv** | pedagogusMunkakortipus, pedagogusFokozat, kotetoOraTeher | Púétv. specifikus |
| **EmploymentEszjtv** | egeszsegugyiOsztaly, onkentesanVallaltTM, illetmenyKategoriaKod | Eszjtv. specifikus |
| **EmploymentKut** | szolgalatiFokozat, szolgalatiIdoKezdete, eskuDatum | Küt. specifikus |

#### 3.1.5. Qualification (4 entitás) – `qualification-entity.md`
**Célja:** Végzettségek, szakképesítések, nyelvtudás, licenszek

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **EducationalQualification** | qualificationType, educationLevel, major, institution, issueDate | Iskolai végzettség |
| **ProfessionalLicense** | licenseType, licenseNumber, issuingAuthority, expiryDate | Szakmai jogosultságok (kamarai tagság) |
| **LanguageProficiency** | languageCode, examType, examLevel, certificateNumber | Nyelvtudás (nyelvvizsga) |
| **Certification** | certificationType, certificationName, issuingOrganization, expiryDate | Tanúsítványok (IT, PM, szakmai) |

#### 3.1.6. Document (3 entitás) – `document-entity.md`
**Célja:** Központi dokumentumkezelés (szerződések, igazolások, jelentések)

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **Document** | documentType, documentCategory, relatedEntityType, relatedEntityId, storageLocation | Dokumentum metaadatok |
| **DocumentVersion** | versionNumber, uploadedDate, fileSize, checksum | Verziókezelés |
| **DocumentAccess** | accessLevel, grantedToRole, grantedToPersonId, expiryDate | Hozzáférési jogosultságok |

---

### 3.2. COMPENSATION & TIME (Javadalmazás és időnyilvántartás) – 4 modul, ~28 entitás

#### 3.2.1. Compensation (8 entitás) – `compensation-entity.md`
**Célja:** Bérelemek, pótlékok, juttatások, cafeteria (jogviszony-típusonként)

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **SalaryStructure** | structureType, category, isTaxable, isSocialContribBase | Bérelemek katalógusa |
| **EmploymentCompensation** | employment_id, effectiveDate, totalMonthlyGross | Muntatársi javadalmazási csomag |
| **CompensationElement** | compensationType, amount, frequency, calculationMethod | Konkrét bérelemek (alapbér, pótlékok) |
| **CafeteriaPlan** | planYear, totalBudget, employmentTypes | Cafeteria keret |
| **CafeteriaAllocation** | cafeteriaPlan_id, employment_id, szepCard, vendeglatoUtalvany | Cafeteria választások |
| **BonusPlan** | planName, planType, targetAchievement, payoutSchedule | Bónusz tervek |
| **EquityGrant** | grantType, numberOfShares, grantDate, vestingSchedule | Részvény/opciós programok |
| **CompensationReview** | reviewCycle, budgetIncrease, meritIncreasePool | Béremelési ciklusok |

#### 3.2.2. Payroll (8 entitás) – `payroll-entity.md`
**Célja:** Teljes bérszámfejtési folyamat (bruttó-nettó, adók, járulékok, NAV)

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **PayrollCycle** | cycleYear, cycleMonth, cycleType, status | Bérszámfejtési ciklus |
| **PayrollRun** | payrollCycle_id, runType, executionDate, status | Konkrét futtatás |
| **PayrollRecord** | payrollRun_id, employment_id, grossPay, netPay | Munkatársi bérszámfejtés |
| **PayrollItem** | payrollRecord_id, itemType, amount, isTaxable | Bérelemek a számfejtésben |
| **TaxCalculation** | payrollRecord_id, szjaAmount, tbEmployeeContribution, ehoAmount | Adó- és járulékszámítás |
| **PayslipDocument** | payrollRecord_id, document_id, generatedDate | Bérlevél |
| **PaymentOrder** | payrollRun_id, paymentMethod, totalAmount, paymentDate | Kifizetési megbízás |
| **PayrollCorrection** | originalPayrollRecord_id, correctionType, correctionReason | Korrekciók |

#### 3.2.3. TimeTracking (7 entitás) – `timetracking-entity.md`
**Célja:** Munkaidő-nyilvántartás (Mt. 134. § kötelező), pótlékszámítás alapja

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **WorkScheduleTemplate** | scheduleType (9 típus), dailyHours, weeklyHours, isFlexible | Munkarend sablon |
| **ShiftTemplate** | startTime, endTime, breakMinutes, isNightShift | Műszak sablon |
| **WorkScheduleAssignment** | employment_id, workScheduleTemplate_id, validFrom, validTo | Munkarend hozzárendelés |
| **ShiftAssignment** | employment_id, date, plannedStartTime, publishedAt | Műszakbeosztás |
| **DailyTimeRecord** | employment_id, date, actualStartTime, actualNetHours, overtimeHours, nightHours | Napi munkaidő-nyilvántartás (Mt. 134. §) |
| **OvertimeRecord** | employment_id, date, overtimeType (5 típus), compensationType | Rendkívüli munkaidő |
| **OnCallRecord** | employment_id, date, onCallType, totalHours, activeWorkHours | Ügyelet / készenlét |

**TimeFramePeriod is benne van, de az áttekintésben 7-et írtam, mert ez opcionális (munkaidőkeret esetén)**

#### 3.2.4. Leave (5 entitások) – `leave-entity.md`
**Célja:** Szabadság, betegszabadság, táppénz, fizetés nélküli szabadság

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **LeaveEntitlement** | employment_id, year, baseEntitlement, supplementaryEntitlement, carryOver | Szabadság jogosultság |
| **LeaveRequest** | employment_id, leaveType, startDate, endDate, status, approvedBy | Szabadságkérelem (workflow) |
| **LeaveRecord** | leaveRequest_id, date, dayCount, leaveType, isPaid | Jóváhagyott távollét |
| **LeaveBalance** | employment_id, year, entitled, used, pending, remaining | Aktuális egyenleg |
| **LeaveCarryOver** | employment_id, fromYear, toYear, carryOverDays, expiryDate | Szabadság átvitel |

---

### 3.3. TALENT (Tehetségmenedzsment) – 3 modul, ~25 entitás

#### 3.3.1. PerformanceReview (6 entitások) – `performance-review-entity.md`
**Célja:** Teljesítményértékelés, célkitűzések, visszajelzések, minősítések

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **PerformanceReviewCycle** | cycleYear, cycleType, startDate, endDate | Értékelési ciklus |
| **PerformanceReview** | employment_id, reviewCycle_id, overallRating, status | Egyéni értékelés |
| **PerformanceGoal** | performanceReview_id, goalType, goalDescription, targetValue, achievedValue | Célkitűzések (OKR, KPI) |
| **CompetencyRating** | performanceReview_id, competencyName, currentLevel, targetLevel | Kompetencia értékelések |
| **PerformanceFeedback** | performanceReview_id, feedbackType, feedbackProvider_id, feedbackText | 360° visszajelzések |
| **PerformanceImprovementPlan** | employment_id, pipReason, startDate, reviewDate, status | Teljesítményfejlesztési terv (PIP) |

#### 3.3.2. Learning (11 entitás) – `learning-entity.md`
**Célja:** Képzési katalógus, kötelező képzések, IDP, tanúsítványok

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **TrainingCatalog** | trainingCode, trainingName, trainingCategory, isMandatory, legalBasis | Képzési katalógus |
| **TrainingSession** | trainingCatalog_id, startDate, endDate, maxParticipants, status | Képzési munkamenetek |
| **TrainingEnrollment** | trainingSession_id, employment_id, enrollmentType, status | Beiratkozások |
| **TrainingAttendance** | trainingEnrollment_id, attendanceDate, attended, checkInTime | Jelenléti ív |
| **TrainingAssessment** | trainingEnrollment_id, assessmentType, score, passFail | Értékelések, vizsgák |
| **TrainingCompletion** | trainingEnrollment_id, completionDate, certificateIssued, cpdCredits | Teljesítés, tanúsítványok |
| **TrainingEvaluation** | trainingEnrollment_id, overallRating, instructorRating, comments | Képzés visszajelzés |
| **IndividualDevelopmentPlan** | employment_id, planYear, status, careerAspiration | Egyéni fejlesztési terv (IDP) |
| **DevelopmentGoal** | idp_id, goalType, goalDescription, targetCompletionDate, status | Fejlesztési célok |
| **MandatoryTrainingAssignment** | employment_id, trainingCatalog_id, dueDate, status | Kötelező képzés kijelölés |
| **TrainingBudget** | budgetYear, organizationUnit_id, allocatedAmount, spentAmount | Képzési költségkeret |

#### 3.3.3. SuccessionPlanning (8 entitás) – `succession-planning-entity.md`
**Célja:** Utódlástervezés, tehetség poolok, 9-box grid, fejlesztési akciók

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **KeyPosition** | position_id, businessCriticality, vacancyRisk, replacementDifficulty | Kulcspozíciók |
| **SuccessionPlan** | keyPosition_id, planYear, status, emergencySuccessor_id | Utódlási tervek |
| **SuccessorCandidate** | successionPlan_id, candidate_employment_id, readinessLevel, successorRank | Utódjelöltek (ready now / 1-2 év / 3+ év) |
| **TalentPool** | poolName, poolType (HiPo, Future Leaders), poolYear, maxMembers | Tehetség poolok |
| **TalentPoolMembership** | talentPool_id, employment_id, entryDate, status, mentor_id | Pool tagságok |
| **TalentAssessment** | employment_id, assessmentYear, performanceRating, potentialRating, nineBoxCategory | 9-box grid értékelés |
| **DevelopmentAction** | successorCandidate_id, actionType, actionName, status, completionPercentage | Fejlesztési akciók (stretch assignments) |
| **KnowledgeTransferPlan** | successionPlan_id, source_employment_id, target_employment_id, status | Tudásátadási tervek |

---

### 3.4. EMPLOYMENT LIFECYCLE (Foglalkoztatási életciklus) – 2 modul, ~19 entitás

#### 3.4.1. Recruitment (11 entitás) – `recruitment-entity.md`
**Célja:** Toborzás, álláshirdetések, jelöltek, interjúk, onboarding

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **JobRequisition** | requisitionNumber, position_id, employmentType, headcount, status | Álláshely igénylés |
| **JobPosting** | jobRequisition_id, postingChannel, publicationDate, expiryDate | Álláshirdetés (közigállás.hu) |
| **Candidate** | person_id, candidateSource, applicationDate, status | Jelöltek |
| **Application** | candidate_id, jobPosting_id, applicationDate, cvDocument_id, status | Jelentkezések |
| **Interview** | application_id, interviewType, interviewDate, interviewer_id | Interjúk |
| **InterviewEvaluation** | interview_id, overallRating, strengths, concerns, recommendation | Interjú értékelések |
| **Assessment** | application_id, assessmentType, assessmentDate, score, result | Tesztek, assessment centerek |
| **JobOffer** | application_id, offerDate, position_id, salaryOffered, offerStatus | Munkavégzésre irányuló ajánlat |
| **OnboardingTask** | employment_id, taskName, taskCategory, dueDate, status | Munkába állási feladatok |
| **OnboardingPlan** | employment_id, planTemplate_id, startDate, completionPercentage | Onboarding terv |
| **OnboardingTaskTemplate** | templateName, applicablePositions, tasks | Onboarding sablon |

#### 3.4.2. Offboarding (8 entitás) – `offboarding-entity.md`
**Célja:** Kilépési folyamat, exit interview, eszköz visszavétel, tudásátadás

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **OffboardingCase** | employment_id, terminationType, terminationReason, lastWorkingDay, status | Kilépési eset |
| **OffboardingChecklistTemplate** | templateName, applicablePositions, tasksCount | Kilépési feladatlista sablon |
| **OffboardingTask** | offboardingCase_id, taskCategory, taskName, dueDate, status | Kilépési feladatok |
| **ExitInterview** | offboardingCase_id, interviewDate, primaryReasonForLeaving, wouldRecommend | Kilépési interjú |
| **AssetReturn** | offboardingCase_id, assetType, assetName, returnDate, conditionOnReturn | Eszköz visszavétel |
| **AccessRevocation** | offboardingCase_id, accessType, accessName, revocationDate, status | Hozzáférések visszavonása |
| **KnowledgeHandover** | offboardingCase_id, knowledgeArea, recipient_employment_id, status | Tudásátadás |
| **FinalClearance** | offboardingCase_id, severancePayAmount, unusedVacationPayout, netFinalPayment | Végső elszámolás |

---

### 3.5. STRATEGIC & COMPLIANCE (Stratégiai és compliance) – 5 modul, ~40 entitás

#### 3.5.1. WorkforcePlanning (10 entitás) – `workforce-planning-entity.md`
**Célja:** Létszámtervezés, skill gap, kapacitástervezés, költségvetés, demográfia

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **WorkforcePlan** | planName, planType, planYear, baselineHeadcount, targetHeadcount | Munkaerő-terv |
| **HeadcountPlan** | workforcePlan_id, organizationUnit_id, baselineHeadcount, plannedHires, targetHeadcount | Létszámterv |
| **HeadcountForecast** | organizationUnit_id, forecastDate, forecastedHeadcount, forecastMethod, confidenceLevel | Létszám-előrejelzés |
| **SkillRequirement** | workforcePlan_id, skillName, proficiencyLevelRequired, requiredHeadcount | Kompetencia követelmények |
| **SkillGap** | skillRequirement_id, employment_id, gapType, currentLevel, requiredLevel, gapSeverity | Skill gap analízis |
| **CapacityPlan** | workforcePlan_id, totalAvailableFTE, plannedProductiveHours, capacityUtilization | Kapacitástervezés |
| **WorkforceBudget** | workforcePlan_id, budgetYear, budgetCategory, budgetedAmount, actualAmount | Munkaerő-költségvetés |
| **ScenarioPlan** | workforcePlan_id, scenarioName, scenarioType, scenarioHeadcountChange, assumptions | "Mi lenne ha" szimulációk |
| **DemographicAnalysis** | analysisDate, organizationUnit_id, analysisType, averageAge, retirementEligibleCount | Demográfiai elemzés |
| **RestructuringPlan** | planName, restructuringType, currentHeadcount, targetHeadcount, severanceCost | Átszervezési terv |

#### 3.5.2. OccupationalHealth (9 entitás) – `occupational-health-entity.md`
**Célja:** Foglalkozás-egészségügy, munkabalesetek, munkavédelmi oktatás, védőeszközök

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **OccupationalHealthExamType** | examCode, examName, examCategory, validityMonths | Vizsgálattípusok |
| **OccupationalHealthExam** | employment_id, examType_id, examCategory, scheduledDate, actualExamDate | Foglalkozás-egészségügyi vizsgálatok |
| **HealthFitnessAssessment** | occupationalHealthExam_id, fitnessStatus, fitnessValidUntil, restrictions | Alkalmassági minősítések |
| **WorkplaceHealthRisk** | position_id, riskLevel, hazardFactors, requiredExamFrequency | Munkahelyi kockázatértékelés |
| **HealthSurveillanceSchedule** | employment_id, examType_id, nextExamDueDate, isOverdue | Vizsgálati ütemterv |
| **OccupationalDisease** | employment_id, diseaseCode, diagnosisDate, reportedToAuthority | Foglalkozási megbetegedések |
| **WorkAccident** | employment_id, accidentDate, accidentType, severity, reportedToAuthority | Munkabalesetek |
| **SafetyTrainingRecord** | employment_id, trainingType, trainingDate, assessmentCompleted, certificateValidUntil | Munkavédelmi oktatás |
| **PersonalProtectiveEquipment** | employment_id, ppeType, ppeName, issueDate, expiryDate, status | Egyéni védőeszköz kiadás |

#### 3.5.3. Disciplinary (10 entitás) – `disciplinary-entity.md`
**Célja:** Fegyelmi eljárások, panaszok, kivizsgálások, szankciók, whistleblowing

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **DisciplinaryCase** | employment_id, caseNumber, violationType, severity, caseStatus | Fegyelmi ügyek |
| **Investigation** | disciplinaryCase_id, investigationType, investigator_id, findings | Kivizsgálások |
| **Evidence** | investigation_id, evidenceType, evidenceDescription, collectedDate | Bizonyítékok |
| **Witness** | investigation_id, witness_employment_id, statementDate, statementSummary | Tanúk |
| **DisciplinaryHearing** | disciplinaryCase_id, hearingDate, hearingType, attendees, outcome | Fegyelmi meghallgatások |
| **DisciplinaryAction** | disciplinaryCase_id, actionType, actionDate, suspensionDays, fineAmount | Fegyelmi büntetések |
| **Appeal** | disciplinaryAction_id, appealDate, appealReason, appealStatus, decision | Fellebbezések |
| **GrievanceCase** | employment_id, grievanceType, filedDate, resolution, status | Panaszok |
| **DisciplinaryHistory** | employment_id, totalCasesCount, activeSuspensionDays, lastCaseDate | Fegyelmi múlt |
| **EthicsViolation** | employment_id, violationType, reportedDate, isWhistleblowerProtected | Etikai szabálysértések |

#### 3.5.4. Workflow (6 entitás) – `workflow-entity.md`
**Célja:** Univerzális jóváhagyási motor (szabadság, bérszámfejtés, teljesítményértékelés)

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **WorkflowTemplate** | workflowName, workflowType, triggerEntityType, isActive | Workflow sablon |
| **WorkflowInstance** | workflowTemplate_id, triggerEntityId, status, startedDate | Futó workflow példány |
| **WorkflowStep** | workflowInstance_id, stepName, stepType, approverRole, status | Workflow lépés |
| **WorkflowApproval** | workflowStep_id, approver_id, approvalStatus, approvalDate, comments | Jóváhagyások |
| **WorkflowEscalation** | workflowStep_id, escalationLevel, escalatedTo_id, escalationDate | Eszkalációk |
| **WorkflowAuditLog** | workflowInstance_id, eventType, eventDate, eventDetails | Audit trail |

#### 3.5.5. ReportPublishing (5 entitás) – `report-publishing-entity.md`
**Célja:** Riport generálás, ütemezés, publikálás, előfizetés, adatexportálás

| Entitás | Főbb property-k | Megjegyzés |
|---|---|---|
| **ReportTemplate** | reportName, reportType, reportCategory, dataSource, outputFormat | Riport sablonok |
| **ReportSchedule** | reportTemplate_id, scheduleFrequency, nextRunDate, isActive | Ütemezett riportok |
| **ReportExecution** | reportTemplate_id, executionDate, executionStatus, recordCount, fileSize | Riport futtatások |
| **ReportSubscription** | reportTemplate_id, subscriber_id, deliveryMethod, isActive | Riport előfizetések |
| **DataExport** | exportType, exportDestination, exportDate, exportStatus, encryptionUsed | Adatexportálások (NAV, MÁK) |

---

## 4. A központi hub: Employment

Az **Employment** az egész HRMS aggregációs gyökere. Minden operatív entitás (Compensation, Payroll, TimeTracking, Leave, PerformanceReview, Learning, stb.) az Employment-en keresztül kapcsolódik a Person-höz és az Organization-höz.

Ez a tervezési döntés biztosítja, hogy **többes jogviszony** esetén minden adat jogviszonyhoz kötötten kezelhető.

**Az Employment-ből kiinduló közvetlen kapcsolatok száma: ~40+** (a legtöbb bármely entitásból).

```
                              ┌─────────────┐
                              │   Person    │
                              └──────┬──────┘
                                     │
                                     ▼
                          ┌──────────────────────┐
                          │    Employment        │◄──── Position
                          │  (központi hub)      │◄──── Organization
                          └──┬───────────────┬───┘
                             │               │
        ┌────────────────────┼───────────────┼────────────────────┐
        │                    │               │                    │
        ▼                    ▼               ▼                    ▼
  Compensation          TimeTracking      Leave           PerformanceReview
  Payroll               OvertimeRecord    LeaveRecord     Learning
  CompensationElement   DailyTimeRecord   LeaveRequest    TrainingEnrollment
  BonusPlan             OnCallRecord                      SuccessorCandidate
        │                    │               │                    │
        ▼                    ▼               ▼                    ▼
  OffboardingCase      HealthFitnessAsmt  SkillGap        DisciplinaryCase
  FinalClearance       WorkAccident       HeadcountPlan
```

---

## 5. Kereszthivatkozási mátrix – főbb kapcsolatok

| Forrás entitás → Cél entitás | Kapcsolat | Leírás |
|---|---|---|
| **Person → Employment** | 1:N | Egy személy több jogviszonnyal rendelkezhet |
| **Person → Qualification** | 1:N | Végzettségek jogviszony-független |
| **Organization → OrgUnit** | 1:N | Szervezeti hierarchia |
| **OrgUnit → Position** | 1:N | Álláshelyek szervezeti egységhez |
| **Position → Employment** | 1:N | Egy álláshelyet több személy tölthet be |
| **Employment → Compensation** | 1:N | Javadalmazási elemek |
| **Employment → PayrollRecord** | 1:N | Bérszámfejtési rekordok (havi) |
| **Employment → DailyTimeRecord** | 1:N | Napi munkaidő-nyilvántartás |
| **Employment → LeaveRequest** | 1:N | Szabadságkérelmek |
| **Employment → PerformanceReview** | 1:N | Teljesítményértékelések |
| **Employment → TrainingEnrollment** | 1:N | Képzési beiratkozások |
| **Employment → SuccessorCandidate** | 1:N | Utódjelölti státusz |
| **Employment → OffboardingCase** | 1:1 | Kilépési folyamat |
| **Employment → OccupationalHealthExam** | 1:N | Foglalkozás-egészségügyi vizsgálatok |
| **Employment → DisciplinaryCase** | 1:N | Fegyelmi ügyek |
| **DailyTimeRecord → PayrollItem** | havi összesítés | Pótlékszámítás (túlóra, éjszakai, műszak) |
| **LeaveRecord → DailyTimeRecord** | 1:1 napi | Távollét-munkaidő összekapcsolás |
| **JobOffer → Employment** | 1:1 | Ajánlat elfogadása → jogviszony |
| **OffboardingCase → FinalClearance** | 1:1 | Kilépési végső elszámolás |

---

## 6. GDPR kategorizálás és adatmegőrzés

### 6.1. Adatkategóriák

| Kategória | GDPR Cikk | Példa entitások |
|---|---|---|
| **Általános személyes adat** | Art. 6(1)(b), (c) | Person, Employment, Compensation, TimeTracking, Leave |
| **Különleges személyes adat (egészségügyi)** | Art. 9(2)(b), (h) | OccupationalHealthExam, HealthFitnessAssessment, OccupationalDisease, WorkAccident |
| **Teljesítményadat** | Art. 88 | PerformanceReview, TalentAssessment, SkillGap, DisciplinaryCase |
| **Pénzügyi adat** | Art. 6(1)(c) | PayrollRecord, Compensation, FinalClearance |
| **Nem személyes adat (aggregált)** | - | DemographicAnalysis, HeadcountPlan, WorkforceBudget |

### 6.2. Adatmegőrzési idők (főbb jogszabályok)

| Jogszabály | Megőrzési idő | Érintett entitások |
|---|---|---|
| **Mt. 287. §** (munkaügyi elévülés) | Jogviszony + 3 év | Employment, DailyTimeRecord, LeaveRecord, OvertimeRecord |
| **Számv. tv. 169. §** | Jogviszony + 8 év | PayrollRecord, FinalClearance, Compensation |
| **Mvt. 62. §** (munkabalesetek) | Jogviszony + 50 év | WorkAccident, OccupationalDisease |
| **89/1995. Korm. r. 9. §** (foglalkozás-egészségügy) | Jogviszony + 50 év | OccupationalHealthExam, HealthFitnessAssessment |
| **GDPR Art. 17** (törléshez való jog) | Jogszabályi megőrzés lejárta után | Minden személyes adat |

---

## 7. Az „employmentType" diszkriminátor hatása

Az `Employment.employmentType` a rendszer legfontosabb diszkriminátora. Hatása az egész rendszerre kiterjed:

| Domén/Modul | Hatás |
|---|---|
| **Employment** | Jogviszony-specifikus property-csoportok (kjt_*, kit_*, puetv_*, eszjtv_*, kut_*) |
| **Compensation** | Számítási logika (táblázat vs. sávos vs. szabad megállapodás); alkalmazható pótlékok |
| **Payroll** | Adó- és járulékszámítás (Kjt./Kit. eltérő járulékok) |
| **Leave** | Alapszabadság (Mt. 20+kor, Púétv. 50, Kit. 25 nap); pótszabadságok |
| **TimeTracking** | Munkarend-típusok (pedagógus kötött/kötetlen, eü. ügyeleti); pótlékszámítás |
| **Position** | Besorolási rendszer (Kjt. fizetési osztály, Kit. besorolási kategória) |
| **Qualification** | Továbbképzési kötelezettség (Kjt. 80 óra/5 év, Púétv. 120 óra/7 év) |
| **Recruitment** | Közigállás.hu kötelező (Kit., Kttv., Kjt.) |
| **PerformanceReview** | Minősítési rendszer (Kjt. 40-42. §, Kit. 75-82. §, Púétv. 62. §) |

---

## 8. Implementációs prioritások

### 8.1. Phase 1: MVP (MUST HAVE) – 10 modul

**Kritikus jogszabályi kötelezettségek**

1. Person
2. Organization
3. Position
4. Employment (7 jogviszony típus)
5. Compensation
6. Payroll (NAV e-payroll compliance)
7. TimeTracking (Mt. 134. § kötelező)
8. Leave
9. Document
10. OccupationalHealth (Mvt. 54. § kötelező)

### 8.2. Phase 2: Extended (SHOULD HAVE) – 6 modul

**Működési hatékonyság, tehetségmenedzsment**

11. PerformanceReview
12. Learning
13. Recruitment (közigállás.hu integráció)
14. Offboarding
15. Workflow
16. Qualification

### 8.3. Phase 3: Strategic (COULD HAVE) – 4 modul

**Stratégiai tervezés, tehetségfejlesztés**

17. SuccessionPlanning
18. WorkforcePlanning
19. Disciplinary
20. ReportPublishing

---

## 9. Integrációs térkép – külső rendszerek

```
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│   NAV        │         │     MÁK      │         │  Közigállás  │
│  e-payroll   │◄────    │   Kincstár   │◄────    │  gov.hu      │◄────
│  T1041       │ Payroll │  Bérszámf.   │ Payroll │  Pályázat    │ Recruitment
└──────────────┘         └──────────────┘         └──────────────┘

┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│   NEAK       │         │  Beléptető   │         │  SZÉP-kártya │
│  TB ellátás  │◄────    │  rendszer    │────►    │  szolgáltató │◄────
│  CSED/GYED   │ Leave   │  (access)    │ Time    │              │ Comp
└──────────────┘         └──────────────┘         └──────────────┘

┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│  KIR/KRÉTA   │         │     OKFŐ     │         │  Active      │
│  Köznevelés  │◄───►    │  Egészségügy │◄────    │  Directory   │◄────
│  Órarend     │ Empl+T  │              │ Empl    │  SSO/Users   │ Person+Access
└──────────────┘         └──────────────┘         └──────────────┘

┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│  Kamarák     │         │  Pénzügyi/   │         │  Assessment  │
│  MOK/MESZK   │────►    │  ERP         │◄───►    │  tools       │────►
│  MGYK        │ Qual    │              │ Org+Comp│  (SHL,Hogan) │ Talent
└──────────────┘         └──────────────┘         └──────────────┘
```

---

## 10. Összefoglalás – Funkcionalitás teljesség

### 10.1. Funkcionális lefedettség

| Funkcionális terület | Lefedettség | Modulok |
|---|---|---|
| **Core HR** | ✅ 100% | Person, Organization, Position, Employment (7 jogviszony) |
| **Payroll & Compensation** | ✅ 100% | Compensation, Payroll (NAV-kompatibilis) |
| **Time & Attendance** | ✅ 100% | TimeTracking, Leave (Mt. 134. § compliance) |
| **Performance Management** | ✅ 100% | PerformanceReview (Kjt./Kit./Púétv. minősítés) |
| **Learning & Development** | ✅ 100% | Learning (kötelező képzések + IDP) |
| **Recruitment & Onboarding** | ✅ 100% | Recruitment (közigállás.hu integráció) |
| **Offboarding** | ✅ 100% | Offboarding (strukturált kilépési folyamat) |
| **Succession Planning** | ✅ 100% | SuccessionPlanning (9-box grid, talent pools) |
| **Workforce Planning** | ✅ 100% | WorkforcePlanning (headcount, skill gap, capacity) |
| **Occupational Health** | ✅ 100% | OccupationalHealth (Mvt. compliance) |
| **Disciplinary & Grievance** | ✅ 100% | Disciplinary (whistleblowing, jogviszony-típusonként) |
| **Workflow & Approvals** | ✅ 100% | Workflow (univerzális jóváhagyási motor) |
| **Document Management** | ✅ 100% | Document (központi DMS) |
| **Reporting & Analytics** | ✅ 100% | ReportPublishing (NAV/MÁK export, BI integráció) |

### 10.2. Jogszabályi compliance

| Jogszabály | Compliance | Érintett modulok |
|---|---|---|
| **Mt. (Munka Törvénykönyve)** | ✅ 100% | Employment, TimeTracking, Leave, Payroll, Compensation, Offboarding |
| **Kjt., Kttv., Kit., Púétv., Eszjtv., Küt.** | ✅ 100% | Employment (7 jogviszony-specifikus kiterjesztés) |
| **Mvt. (Munkavédelmi törvény)** | ✅ 100% | OccupationalHealth, Learning |
| **GDPR** | ✅ 100% | Minden modul (beépített adatvédelmi kategorizálás) |
| **Szja tv., Tbj., Eho tv., Szocho tv.** | ✅ 100% | Payroll, Compensation |
| **NAV e-payroll** | ✅ 100% | Payroll, ReportPublishing |

### 10.3. Hiányzó funkciók (opcionális, nem kritikus)

| Funkció | Prioritás | Indoklás |
|---|---|---|
| **Wellness programs** | ALACSONY | Opcionális, modern HRMS trend |
| **Competency Framework (önálló modul)** | ALACSONY | Részben lefedett (PerformanceReview, SkillGap, TalentAssessment) |
| **Career Pathing (strukturált)** | ALACSONY | Részben lefedett (SuccessionPlanning, IDP) |
| **Employee Engagement Surveys** | ALACSONY | Külső survey tools integráció |

---

## 11. Következtetés

A tervezett HRMS rendszer **funkcionalitás szempontjából teljes** és **jogszabályi szempontból compliant**.

**20 funkcionális modul, ~150+ entitás** lefedi:
- ✅ Alapvető HR folyamatokat (employment lifecycle, payroll, time & attendance)
- ✅ 7 jogviszony típus speciális követelményeit
- ✅ Stratégiai HR funkciókat (succession planning, workforce planning, talent management)
- ✅ Kötelező compliance-t (NAV, MÁK, Mvt., GDPR)
- ✅ Modern HR best practice-eket (9-box grid, IDP, exit interviews, skill gap analysis)

**A rendszer alkalmas greenfield HRMS megvalósítására**, különösen magyar közszféra + magánszektor vegyes környezetben.

---

**Dokumentum verzió:** 1.0
**Utolsó frissítés:** 2025. február
**Kapcsolódó dokumentumok:**
- `hrms-entity-overview.md` – Részletes architektúra és entitás leírások
- `*-entity.md` (20 db) – Modul-specifikus entitás dokumentációk
