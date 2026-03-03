# Learning (Képzés és Fejlesztés) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer
> **Verzió:** 0.1 – első tervezet
> **Utolsó frissítés:** 2025. február
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md, employment-entity.md, organization-entity.md, position-entity.md, performance-review-entity.md, document-entity.md, qualification-entity.md

---

## 1. Az entitás célja és hatóköre

A képzési és fejlesztési entitások biztosítják a szervezeti tanulás, készségfejlesztés, kötelező képzések és egyéni fejlesztési tervek kezelését. Lefedi mind a jogi kötelezettségekből fakadó képzéseket (munkavédelem, adatvédelem, compliance), mind az önkéntes fejlesztési programokat.

### Hatókör

A képzési entitások lefedik:
- **Képzési katalógus**: belső és külső képzések, kurzusok nyilvántartása
- **Képzési munkamenetek**: konkrét képzési alkalmak, időpontok, helyszínek
- **Képzési beiratkozás**: munkatársak jelentkezése és kijelölése képzésekre
- **Részvétel nyilvántartás**: jelenléti ívek, teljesítések, eredmények
- **Egyéni fejlesztési tervek (IDP)**: célok, kompetenciafejlesztés, karrier tervezés
- **Tanúsítványok és licenszek**: lejáró képesítések nyomon követése, megújítások
- **Képzési költségvetés**: képzési keret, költségek elszámolása
- **Kötelező képzések compliance**: határidők, teljesítettség, emlékeztetők

### Foglalkoztatási típus specifikus követelmények

#### Minden jogviszony (Mt., Kjt., Kttv., Kit., Púétv., Eszjtv., Küt.)
- **Munkavédelmi oktatás**: 1993. évi XCIII. törvény a munkavédelemről
  - Belépéskor kötelező
  - Évente ismétlő oktatás
  - Munkaköri változáskor
  - Jegyzőkönyv vezetése kötelező
- **Adatvédelmi (GDPR) oktatás**: belépéskor és változáskor
- **Tűzvédelmi oktatás**: belépéskor és rendszeresen

#### Közszféra (Kjt., Kit., Kttv.)
- **Köztisztviselői továbbképzés**: Kjt. 48-50. § alapján kötelező továbbképzési rendszer
  - Legalább 80 óra/5 év
  - Képzési terv készítése
  - Teljesítés nyilvántartása
- **Kormánytisztviselői továbbképzés**: Kttv. 108-109. § alapján kötelező
  - Legalább 80 óra/5 év
  - Vezetői továbbképzés kötelező
- **Közszolgálati szakvizsga**: Kit. 54-57. §
  - Kinevezéstől számított 1 éven belül kötelező
  - 3 vizsgaalkalom (közigazgatási ismeretek, szakmai ismeretek, nyelvvizsga)

#### Egészségügy (Eszjtv.)
- **Folyamatos orvosképzés**: Eszjtv. 63. § szerinti kötelező továbbképzési pont (kreditpont) rendszer
  - Évente minimum kreditmegszerzési kötelezettség
  - 7 évenként recertifikáció

#### Pedagógusok (Púétv.)
- **Továbbképzési kötelezettség**: Púétv. 63. § alapján 7 évenként legalább 120 óra
- **Minősítési rendszer**: Púétv. 62. § szerinti Pedagógus I-II fokozatok (képzések kötelezőek)

## 2. Entitásstruktúra

### 2.1 TrainingCatalog (Képzési katalógus)

Elérhető képzések, kurzusok, programok központi jegyzéke.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `training_code` | String | igen | Egyedi képzési kód (pl. "WHS-001", "GDPR-101") | - |
| `training_name` | String | igen | Képzés neve | - |
| `training_description` | Text | igen | Részletes leírás | - |
| `training_category` | Enum | igen | mandatory_compliance/safety/technical/soft_skills/leadership/language/professional | Kategória |
| `compliance_type` | Enum | nem | work_safety/data_protection/fire_safety/anti_corruption/ethics/other | Ha mandatory_compliance |
| `is_mandatory` | Boolean | igen | Kötelező-e | Jogszabály vagy munkáltatói döntés |
| `legal_basis` | String | nem | Jogszabályi hivatkozás (pl. "Mvt. 54. §", "Kjt. 48. §") | - |
| `target_audience` | JSON | nem | Célcsoport szűrők (employment_type, org_unit, position) | - |
| `delivery_method` | Enum | igen | classroom/online/blended/on_the_job/external | Képzési forma |
| `provider_type` | Enum | igen | internal/external | Belső vagy külső szolgáltató |
| `external_provider_name` | String | nem | Külső szolgáltató neve | - |
| `duration_hours` | Decimal | igen | Képzés időtartama órában | - |
| `max_participants` | Integer | nem | Maximum résztvevők száma | - |
| `cost_per_participant` | Decimal | nem | Költség/fő (Ft) | - |
| `language` | String | igen | Képzés nyelve (hu/en/de...) | - |
| `learning_objectives` | Text | nem | Tanulási célok | - |
| `prerequisites` | Text | nem | Előfeltételek | - |
| `certification_issued` | Boolean | igen | Ad-e tanúsítványt | - |
| `certification_validity_years` | Integer | nem | Tanúsítvány érvényessége (év) | - |
| `recurrence_rule` | String | nem | Ismétlődési szabály (pl. "évente", "5 évente 80 óra") | Kjt. 48. §, Kttv. 108. § |
| `requires_approval` | Boolean | igen | Igényel-e jóváhagyást | - |
| `is_active` | Boolean | igen | Aktív-e a katalógusban | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.2 TrainingSession (Képzési munkamenet)

Konkrét képzési alkalmak (időpont, helyszín, oktató).

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `session_code` | String | igen | Egyedi munkamenet kód (pl. "WHS-001-2026-01") | - |
| `training_catalog_id` | UUID | igen | Melyik képzés | TrainingCatalog |
| `session_name` | String | nem | Munkamenet neve (ha eltér a katalógustól) | - |
| `start_date` | DateTime | igen | Kezdés időpontja | - |
| `end_date` | DateTime | igen | Befejezés időpontja | - |
| `location_type` | Enum | igen | on_site/remote/external_venue | - |
| `work_location_id` | UUID | nem | Helyszín (ha on_site) | organization-entity.md |
| `external_location` | String | nem | Külső helyszín címe | - |
| `online_meeting_link` | String | nem | Online link (Teams, Zoom) | - |
| `instructor_internal_id` | UUID | nem | Belső oktató | person-entity.md |
| `instructor_external_name` | String | nem | Külső oktató neve | - |
| `max_participants` | Integer | nem | Maximum résztvevők (ha eltér a katalógustól) | - |
| `enrolled_count` | Integer | igen | Beiratkozott résztvevők száma | Számított mező |
| `waitlist_count` | Integer | igen | Várólistán lévők száma | Számított mező |
| `status` | Enum | igen | planned/open_for_enrollment/full/in_progress/completed/cancelled | - |
| `cancellation_reason` | Text | nem | Lemondás oka | - |
| `actual_cost` | Decimal | nem | Tényleges költség | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.3 TrainingEnrollment (Képzési beiratkozás)

Munkatársak jelentkezése vagy kijelölése képzésre.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `training_session_id` | UUID | igen | Melyik munkamenetre | TrainingSession |
| `employment_id` | UUID | igen | Melyik munkatárs | employment-entity.md |
| `enrollment_type` | Enum | igen | self_enrolled/manager_assigned/hr_assigned/mandatory | Hogyan került a listára |
| `enrollment_date` | DateTime | igen | Beiratkozás időpontja | - |
| `enrolled_by_id` | UUID | nem | Ki íratta be (ha assigned) | person-entity.md |
| `status` | Enum | igen | enrolled/waitlisted/confirmed/attended/no_show/cancelled/exempted | Státusz |
| `waitlist_position` | Integer | nem | Sorszám a várólistán | - |
| `confirmed_date` | DateTime | nem | Megerősítés időpontja | - |
| `cancellation_date` | DateTime | nem | Lemondás időpontja | - |
| `cancellation_reason` | Text | nem | Lemondás indoka | - |
| `exemption_reason` | Text | nem | Mentesség indoka (pl. már rendelkezik a képesítéssel) | - |
| `cost_charged` | Decimal | nem | Elszámolt költség (ha van költségmegosztás) | - |
| `cost_center` | String | nem | Költséghely | - |
| `manager_approved` | Boolean | nem | Vezető jóváhagyta-e | - |
| `manager_approved_by_id` | UUID | nem | Jóváhagyó vezető | person-entity.md |
| `manager_approved_date` | DateTime | nem | Jóváhagyás időpontja | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.4 TrainingAttendance (Képzési részvétel)

Jelenléti ív, teljesítés, eredmények.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `training_enrollment_id` | UUID | igen | Melyik beiratkozáshoz | TrainingEnrollment |
| `attendance_date` | Date | igen | Részvétel dátuma (ha többnapos képzés, naponta 1 rekord) | - |
| `attended` | Boolean | igen | Jelen volt-e | - |
| `attendance_percentage` | Decimal | nem | Részvételi arány (%) | - |
| `check_in_time` | DateTime | nem | Érkezés időpontja | - |
| `check_out_time` | DateTime | nem | Távozás időpontja | - |
| `recorded_by_id` | UUID | igen | Ki rögzítette (oktató, HR) | person-entity.md |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.5 TrainingAssessment (Képzési értékelés)

Képzés végi értékelés, vizsga, teszt eredmények.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `training_enrollment_id` | UUID | igen | Melyik beiratkozáshoz | TrainingEnrollment |
| `assessment_type` | Enum | igen | exam/quiz/practical/presentation/peer_review/self_assessment | Értékelés típusa |
| `assessment_date` | Date | igen | Értékelés dátuma | - |
| `score` | Decimal | nem | Pontszám | - |
| `max_score` | Decimal | nem | Maximum pontszám | - |
| `percentage` | Decimal | nem | Százalék (%) | - |
| `pass_threshold` | Decimal | nem | Átmenő küszöb (%) | - |
| `pass_fail_status` | Enum | nem | passed/failed/pending | - |
| `grade` | String | nem | Osztályzat (pl. "jeles", "A", "5") | - |
| `assessor_id` | UUID | nem | Értékelő | person-entity.md |
| `feedback` | Text | nem | Visszajelzés | - |
| `retry_number` | Integer | igen | Hányadik próbálkozás (1, 2, 3...) | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.6 TrainingCompletion (Képzési teljesítés)

Képzés végleges teljesítése, tanúsítvány kiadás.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `training_enrollment_id` | UUID | igen | Melyik beiratkozáshoz | TrainingEnrollment |
| `training_catalog_id` | UUID | igen | Melyik képzés (denormalizált) | TrainingCatalog |
| `employment_id` | UUID | igen | Melyik munkatárs (denormalizált) | employment-entity.md |
| `completion_date` | Date | igen | Teljesítés dátuma | - |
| `completion_status` | Enum | igen | completed/failed/exempted | - |
| `total_hours_attended` | Decimal | nem | Összesen részvett órák | - |
| `certificate_issued` | Boolean | igen | Tanúsítvány kiállítva-e | - |
| `certificate_number` | String | nem | Tanúsítvány száma | - |
| `certificate_issue_date` | Date | nem | Tanúsítvány kiállítás dátuma | - |
| `certificate_valid_until` | Date | nem | Tanúsítvány érvényessége | - |
| `certificate_document_id` | UUID | nem | Tanúsítvány dokumentum | document-entity.md |
| `cpd_credits` | Decimal | nem | CPD/CE kreditpontok (folyamatos szakmai fejlődés) | Eszjtv. 63. § |
| `recorded_by_id` | UUID | igen | Ki rögzítette | person-entity.md |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.7 TrainingEvaluation (Képzés visszajelzés)

Résztvevők értékelése a képzésről (course evaluation, feedback).

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `training_enrollment_id` | UUID | igen | Melyik beiratkozáshoz | TrainingEnrollment |
| `evaluation_date` | DateTime | igen | Értékelés kitöltés időpontja | - |
| `overall_rating` | Integer | nem | Összesített értékelés (1-5) | - |
| `content_rating` | Integer | nem | Tartalom értékelése (1-5) | - |
| `instructor_rating` | Integer | nem | Oktató értékelése (1-5) | - |
| `relevance_rating` | Integer | nem | Releváncia értékelése (1-5) | - |
| `would_recommend` | Boolean | nem | Ajánlaná-e másoknak | - |
| `comments` | Text | nem | Szöveges visszajelzés | - |
| `suggestions` | Text | nem | Fejlesztési javaslatok | - |
| `is_anonymous` | Boolean | igen | Névtelen kitöltés-e | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |

### 2.8 IndividualDevelopmentPlan (Egyéni fejlesztési terv - IDP)

Munkatársak egyéni fejlesztési céljai, kompetencia gap-ek, karrierfejlesztés.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `employment_id` | UUID | igen | Melyik munkatárs | employment-entity.md |
| `plan_year` | Integer | igen | Tervezési év | - |
| `status` | Enum | igen | draft/active/completed/cancelled | - |
| `created_date` | Date | igen | Létrehozás dátuma | - |
| `approved_by_manager_id` | UUID | nem | Jóváhagyó vezető | person-entity.md |
| `approved_date` | Date | nem | Jóváhagyás dátuma | - |
| `review_date` | Date | nem | Felülvizsgálat dátuma (féléves, éves) | - |
| `career_aspiration` | Text | nem | Karriertörekvések | - |
| `strengths` | Text | nem | Erősségek | - |
| `development_areas` | Text | nem | Fejlesztendő területek | - |
| `notes` | Text | nem | Általános megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.9 DevelopmentGoal (Fejlesztési cél)

IDP-n belüli konkrét fejlesztési célok.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `idp_id` | UUID | igen | Melyik IDP-hez | IndividualDevelopmentPlan |
| `goal_type` | Enum | igen | competency/skill/certification/role_preparation/behavior | Cél típusa |
| `competency_name` | String | nem | Kompetencia neve (pl. "Vezetés", "Projektmenedzsment") | - |
| `goal_description` | Text | igen | Cél leírása | - |
| `success_criteria` | Text | nem | Sikerességi kritériumok | - |
| `target_completion_date` | Date | nem | Tervezett teljesítés dátuma | - |
| `priority` | Enum | igen | high/medium/low | Prioritás |
| `current_proficiency_level` | Integer | nem | Jelenlegi szint (1-5) | - |
| `target_proficiency_level` | Integer | nem | Cél szint (1-5) | - |
| `status` | Enum | igen | not_started/in_progress/completed/cancelled | - |
| `completion_date` | Date | nem | Tényleges teljesítés dátuma | - |
| `outcome` | Text | nem | Eredmény leírása | - |
| `related_training_catalog_ids` | JSON | nem | Kapcsolódó képzések | TrainingCatalog UUID-k |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.10 MandatoryTrainingAssignment (Kötelező képzés kijelölés)

Kötelező képzések automatikus kijelölése.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `employment_id` | UUID | igen | Melyik munkatárs | employment-entity.md |
| `training_catalog_id` | UUID | igen | Melyik képzés | TrainingCatalog |
| `assignment_reason` | Enum | igen | onboarding/annual_refresh/regulation_change/role_change/certification_expiry | Kijelölés oka |
| `assignment_date` | Date | igen | Kijelölés dátuma | - |
| `due_date` | Date | igen | Teljesítési határidő | Jogszabály vagy munkáltatói szabályzat |
| `status` | Enum | igen | assigned/in_progress/completed/overdue/exempted | - |
| `completion_date` | Date | nem | Teljesítés dátuma | - |
| `training_completion_id` | UUID | nem | Kapcsolódó teljesítés | TrainingCompletion |
| `exemption_reason` | Text | nem | Mentesség indoka | - |
| `exemption_approved_by_id` | UUID | nem | Mentesség jóváhagyó | person-entity.md |
| `reminder_sent_count` | Integer | igen | Hány emlékeztető lett küldve | - |
| `last_reminder_sent` | DateTime | nem | Utolsó emlékeztető időpontja | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.11 TrainingBudget (Képzési költségvetés)

Képzési költségkeretek szervezeti egységenként / személyenként.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `budget_year` | Integer | igen | Költségvetési év | - |
| `organization_unit_id` | UUID | nem | Szervezeti egység (ha egységre szól) | organization-entity.md |
| `employment_id` | UUID | nem | Munkatárs (ha egyéni keretre szól) | employment-entity.md |
| `budget_type` | Enum | igen | organizational/individual | - |
| `allocated_amount` | Decimal | igen | Kiosztott összeg (Ft) | - |
| `spent_amount` | Decimal | igen | Elköltött összeg (Ft) | Számított mező |
| `remaining_amount` | Decimal | igen | Maradvány (Ft) | Számított mező |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

## 3. GDPR kategorizáció és adatkezelés

### Képzési adatok kezelése

#### Jogalap
- **GDPR Art. 6(1)(b)**: szerződés teljesítése (munkaviszony keretében biztosított képzések)
- **GDPR Art. 6(1)(c)**: jogi kötelezettség teljesítése (kötelező képzések)
- **GDPR Art. 88**: munkavállalói adatok kezelése foglalkoztatás keretében

#### Adatkategóriák
- **Általános személyes adat**: név, munkakör, szervezeti egység, képzési részvétel
- **Teljesítményadat**: értékelések, eredmények, kompetencia szintek (IDP)
- **Nem különleges adat kategória**: képzési adatok nem tartoznak GDPR Art. 9 alá

#### Adatmegőrzési idők
- **Kötelező képzések**: munkaviszony megszűnése + 5 év (munkavédelmi oktatás: munkaviszony + 50 év, ha munkabaleseti kockázat)
- **Önkéntes képzések**: munkaviszony megszűnése + 3 év
- **Tanúsítványok**: véglegesen (igazolási kötelezettség miatt)
- **IDP**: munkaviszony megszűnése + 3 év

### Hozzáférési jogok
- **Munkatárs**: saját képzési előzmények, IDP, beiratkozások
- **Vezető**: beosztottak képzési adatai, IDP-k, kötelező képzések státusza
- **HR**: teljes hozzáférés
- **Oktató**: saját képzéseinek résztvevői listája, értékelések

## 4. Hozzáférési szintek

| Entitás | HR Admin | Manager | Employee | Instructor |
|---|---|---|---|---|
| TrainingCatalog | CRUD | Read | Read | Read |
| TrainingSession | CRUD | Read | Read | Read/Update (saját) |
| TrainingEnrollment | CRUD | CRUD (team) | Create/Read (saját) | Read (saját session) |
| TrainingAttendance | CRUD | Read (team) | Read (saját) | CRUD (saját session) |
| TrainingAssessment | CRUD | Read (team) | Read (saját) | CRUD (saját session) |
| TrainingCompletion | CRUD | Read (team) | Read (saját) | Create (saját session) |
| TrainingEvaluation | Read | Read (aggregált) | CRUD (saját) | Read (aggregált) |
| IndividualDevelopmentPlan | CRUD | CRUD (team) | CRUD (saját) | - |
| DevelopmentGoal | CRUD | CRUD (team) | CRUD (saját) | - |
| MandatoryTrainingAssignment | CRUD | Read (team) | Read (saját) | - |
| TrainingBudget | CRUD | Read (saját unit) | Read (saját) | - |

## 5. Validációs szabályok

### TrainingCatalog
- **Kötelező képzéseknél**: `is_mandatory=true` → `legal_basis` kitöltése ajánlott
- **Tanúsítvány érvényesség**: `certification_issued=true` → `certification_validity_years` megadása

### TrainingSession
- **Időtartam ellenőrzés**: `end_date >= start_date`
- **Kapacitás ellenőrzés**: `enrolled_count <= max_participants`

### TrainingEnrollment
- **Duplicate ellenőrzés**: egy munkatárs nem iratkozhat be többször ugyanarra a TrainingSession-re
- **Kapacitás ellenőrzés**: ha `enrolled_count >= max_participants` → status=waitlisted
- **Kötelező képzés jóváhagyás**: `is_mandatory=true` esetén manager_approved nem szükséges

### TrainingCompletion
- **Attendance feltétel**: csak akkor completed, ha TrainingAttendance létezik és attended=true
- **Certificate érvényesség számítás**: `certificate_valid_until = completion_date + certification_validity_years`

### MandatoryTrainingAssignment
- **Lejárat ellenőrzés**: ha `due_date < current_date` és `status != completed` → status=overdue
- **Emlékeztető szabályok**:
  - 30 nappal határidő előtt: 1. emlékeztető
  - 14 nappal határidő előtt: 2. emlékeztető
  - 7 nappal határidő előtt: 3. emlékeztető
  - Lejárat napján: 4. emlékeztető
  - Lejárat után 7 naponként: ismétlő emlékeztetők

### IndividualDevelopmentPlan
- **Éves ciklus**: egy munkatársnak egy évre csak 1 aktív IDP-je lehet
- **Jóváhagyási folyamat**: status=active csak manager approval után

## 6. Integrációs pontok

### Külső rendszerek
- **LMS (Learning Management System)**:
  - Moodle, Coursera, LinkedIn Learning, Udemy Business
  - SCORM/xAPI integráció (képzési tartalmak, eredmények szinkronizálása)
- **Webinar platformok**: Teams, Zoom, Webex
- **Assessment platformok**: online vizsgarendszerek, skill assessment tools
- **Közigazgatási Továbbképzési Centrum (KTK)**: Kjt./Kttv. továbbképzési kreditekhez
- **Magyar Orvosi Kamara (MOK)**: Eszjtv. kreditpontok elismertetése
- **E-learning platformok**: saját vagy külső e-learning tartalmak

### Belső entitások
- **Employment**: képzések munkatársakhoz rendelése
- **Person**: belső oktatók
- **Organization**: képzések szervezeti egységekhez
- **Position**: pozíció-specifikus képzési követelmények
- **Performance Review**: IDP és teljesítményértékelés kapcsolat (fejlesztési területek)
- **Document**: tanúsítványok, jelenléti ívek, képzési anyagok
- **Qualification**: TrainingCompletion → ProfessionalLicense (ha jogosultság megszerzése)
- **Compensation**: képzési költségtérítés, ösztöndíjak

## 7. Kapcsolódó entitások

- **employment-entity.md**: Employment (munkatársak)
- **person-entity.md**: Person (oktatók, jóváhagyók)
- **organization-entity.md**: OrgUnit, WorkLocation (képzési helyszínek)
- **position-entity.md**: Position (pozíció-specifikus képzési követelmények)
- **performance-review-entity.md**: PerformanceReview (fejlesztési területek → IDP)
- **document-entity.md**: Document (tanúsítványok, képzési anyagok)
- **qualification-entity.md**: ProfessionalLicense (képzésből származó jogosultságok)

## 8. Üzleti folyamatok

### 8.1 Kötelező képzési folyamat

1. **Új belépő**: onboarding során MandatoryTrainingAssignment automatikus létrehozása
   - Munkavédelmi oktatás (due_date = start_date)
   - Adatvédelmi oktatás (due_date = start_date + 7 nap)
   - Tűzvédelmi oktatás (due_date = start_date + 14 nap)
2. **Emlékeztetők**: automatikus emailek a határidők előtt
3. **Beiratkozás**: HR vagy munkatárs beíratkozik egy TrainingSession-re
4. **Részvétel**: TrainingAttendance rögzítése (jelenléti ív)
5. **Teljesítés**: TrainingCompletion létrehozása
6. **MandatoryTrainingAssignment lezárása**: status=completed

### 8.2 Önkéntes képzési folyamat

1. **Katalógus böngészés**: munkatárs megtekinti a TrainingCatalog-ot
2. **Jelentkezés**: TrainingEnrollment létrehozása (enrollment_type=self_enrolled)
3. **Jóváhagyás**: vezető jóváhagyja (ha requires_approval=true)
4. **Beiratkozás megerősítés**: status=confirmed
5. **Részvétel**: TrainingAttendance
6. **Értékelés**: TrainingEvaluation kitöltése (course feedback)
7. **Teljesítés**: TrainingCompletion
8. **Költségelszámolás**: TrainingBudget spent_amount frissítése

### 8.3 IDP folyamat

1. **IDP létrehozás**: munkatárs vagy vezető kezdeményezi (draft status)
2. **Célok meghatározás**: DevelopmentGoal entitások hozzáadása
3. **Képzési javaslatok**: kapcsolódó TrainingCatalog elemek kiválasztása
4. **Jóváhagyás**: vezető jóváhagyja → status=active
5. **Félévközi review**: review_date-kor felülvizsgálat, célok aktualizálása
6. **Képzések teljesítése**: DevelopmentGoal status=completed
7. **Éves értékelés**: performance review során IDP eredmények értékelése

### 8.4 Közszolgálati továbbképzés (Kjt. 80 óra/5 év)

1. **5 éves ciklus számítás**: kinevezéstől / előző ciklus végétől
2. **Képzési terv**: elvárt 80 óra elosztása 5 évre (pl. 16 óra/év)
3. **Beiratkozások**: TrainingEnrollment
4. **Óra aggregálás**: TrainingCompletion.total_hours_attended összegzése
5. **Compliance monitoring**: 5 év végén >= 80 óra ellenőrzés
6. **Reporting**: köztisztviselői továbbképzési jelentés

## 9. Reporting és analitika

### Compliance reporting
- **Kötelező képzések teljesítettség**: MandatoryTrainingAssignment status=completed arány
- **Lejárt kötelezettségek**: status=overdue lista (munkatárs, képzés, határidő)
- **Tanúsítványok lejárata**: certificate_valid_until < 90 nap (megújítási figyelmeztetés)
- **Kjt./Kttv. továbbképzési kötelezettség**: 5 éves ciklusonkénti teljesítés
- **Eszjtv. kreditpontok**: éves minimum kreditmegszerzés

### Képzési aktivitás
- **Képzések száma**: TrainingSession count (év, negyedév, hónap)
- **Résztvevők száma**: TrainingEnrollment count
- **Részvételi arány**: attended / enrolled %
- **No-show rate**: no_show / enrolled %
- **Képzési napok/fő**: átlagos képzési napok munkatársanként
- **Belső vs külső képzések aránya**: provider_type szerinti bontás

### Költségek
- **Képzési költségek összesen**: TrainingSession.actual_cost vagy TrainingEnrollment.cost_charged
- **Költség/fő**: total_cost / participant_count
- **Költségvetési kihasználtság**: TrainingBudget spent_amount / allocated_amount %
- **ROI (Return on Investment)**: képzési költség vs teljesítménynövekedés (performance-review kapcsolat)

### Képzési hatékonyság
- **Átlagos értékelés**: TrainingEvaluation.overall_rating átlaga képzésenként
- **Completion rate**: TrainingCompletion / TrainingEnrollment %
- **Pass rate**: pass_fail_status=passed / total assessments %
- **Knowledge retention**: időbeli értékelések (3 hó, 6 hó után)

### IDP és fejlesztés
- **IDP kitöltöttség**: active IDP / total employees %
- **Fejlesztési célok teljesítése**: DevelopmentGoal status=completed %
- **Kompetencia növekedés**: target_proficiency_level vs current_proficiency_level delta

## 10. Nyitott kérdések és jövőbeli fejlesztések

### Technológiai kérdések
1. **LMS platform választás**: saját fejlesztés vs külső LMS (Moodle, TalentLMS, stb.)?
2. **E-learning tartalmak**: saját fejlesztés vagy vásárolt tartalmak (LinkedIn Learning, Udemy)?
3. **SCORM/xAPI integráció**: e-learning standardok támogatása?
4. **Mobil alkalmazás**: oktatási tartalmak mobilon, offline mód?

### Funkcionális fejlesztések
1. **Skill matrix**: kompetencia mátrix pozíciónként, gap analysis
2. **Career pathing**: karriérútvonalak meghatározása, szükséges képzések
3. **Mentoring program**: mentor-mentee párosítás, találkozók nyomon követése
4. **Gamification**: pontok, jelvények, ranglisták a tanulási aktivitásért
5. **Social learning**: fórumok, peer-to-peer tanulás, knowledge sharing

### Compliance és jogi
1. **Kjt./Kttv. továbbképzési rendszer változások**: jogszabály változások követése
2. **Eszjtv. kreditpont rendszer frissítések**: MOK előírások
3. **Munkavédelmi oktatás auditálás**: jegyzőkönyvek digitális kezelése
4. **ISO 10015 (Quality management - Guidelines for training)**: megfelelés

### Integrációk
1. **KTK (Közigazgatási Továbbképzési Centrum)**: köztisztviselői képzések elismertetése
2. **MOK (Magyar Orvosi Kamara)**: egészségügyi kreditpontok
3. **Coursera / LinkedIn Learning API**: külső képzési platformok
4. **Teams / Zoom**: automatikus meeting létrehozás képzésekhez
5. **HRIS integráció**: pozíció változás → kötelező képzések újraértékelése

### AI és automatizálás
1. **Személyre szabott képzési javaslatok**: ML alapú ajánlás (IDP, teljesítmény, pozíció alapján)
2. **Chatbot**: képzési katalógus keresés, beiratkozási segítség
3. **Automatic skill tagging**: képzések automatikus címkézése kompetenciákkal
4. **Predictive analytics**: ki fog elhagyni egy képzést? (no-show prediction)
