# Recruitment (Toborzás és Munkába Állás) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer
> **Verzió:** 0.1 – első tervezet
> **Utolsó frissítés:** 2025. február
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md, employment-entity.md, organization-entity.md, position-entity.md, document-entity.md, qualification-entity.md

---

## 1. Az entitás célja és hatóköre

A toborzási és munkába állási entitások biztosítják az álláshirdetések, jelentkezések, jelöltek, interjúk, ajánlatok és munkába állási folyamatok kezelését. Különös figyelmet fordítunk a közszféra specifikus követelményeire, különösen a közigállás.hu kötelező közzétételi szabályokra (Kit., Kttv., Kjt.), valamint a munkajogi kötelezettségekre (Mt.).

### Hatókör

A toborzási entitások lefedik:
- **Álláshirdetések kezelése**: belső és külső álláshelyek publikálása, közigállás integráció
- **Jelöltek és jelentkezések**: pályázók adatainak kezelése, jelentkezési folyamat nyomon követése
- **Kiválasztási folyamat**: interjúk, tesztek, értékelések dokumentálása
- **Ajánlat menedzsment**: munkavégzésre irányuló jogviszony ajánlatának kezelése
- **Munkába állás (onboarding)**: új belépők feladatlistája, dokumentumgyűjtés, betanítás
- **GDPR compliance**: pályázói adatok kezelése, hozzájárulás, törlési kötelezettségek

### Foglalkoztatási típus specifikus követelmények

#### Közszféra (Kit., Kttv., Kjt.)
- **Közigállás.hu integráció**: Kit. 45. §, Kttv. 102. §, Kjt. 45/A. § alapján KÖTELEZŐ a közigállás.hu rendszeren keresztüli hirdetés
- **Nyilvános pályáztatás**: minden közszolgálati/kormánytisztviselői/közalkalmazotti jogviszony pályázat kötelező
- **Várólista kezelés**: sikertelen pályázók 6 hónapig tárolhatók, újrahirdetés esetén tájékoztatás
- **Felmentési időszak tájékoztatás**: kötelező feltüntetni a hirdetésben
- **Összeférhetetlenségi nyilatkozat**: pályázóknak nyilatkozniuk kell
- **Erkölcsi bizonyítvány**: Kit./Kttv./Kjt. alapján kötelező
- **Vagyonnyilatkozat tételi kötelezettség**: egyes pozíciók esetén

#### Magánszektor (Mt.)
- **Hirdetési szabadság**: nincs kötelező platform
- **Egyenlő bánásmód**: Ebktv. alapján diszkriminációmentes hirdetés
- **Adatvédelmi tájékoztatás**: GDPR szerinti információk megadása
- **Próbaidő**: Mt. 37. § alapján maximum 3 hónap (vezető 6 hónap)

#### Egyéb közszféra (Púétv., Eszjtv.)
- **Püspöki kinevezés**: Púétv. alapján püspök nevezi ki (pályáztatás nem kötelező)
- **Eszjtv. munkáltatók**: Kit.-hez hasonló pályáztatási kötelezettség lehet az alapító okirat szerint

## 2. Entitásstruktúra

### 2.1 JobRequisition (Álláshely igény)

Üzleti igény egy új munkavállaló felvételére. Jóváhagyási folyamat eredményeként születik JobPosting.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `requisition_number` | String | igen | Egyedi azonosító (pl. REQ-2026-001) | Belső szabályzat |
| `organization_unit_id` | UUID | igen | Melyik szervezeti egységhez tartozik | organization-entity.md |
| `position_id` | UUID | nem | Melyik pozícióra (ha meglévő pozíció újratöltése) | position-entity.md |
| `employment_type` | Enum | igen | mt/kjt/kttv/kit/puetv/eszjtv/kut | Jogviszony típusa |
| `job_title` | String | igen | Betöltendő munkakör | - |
| `headcount` | Integer | igen | Hány fő felvétele szükséges | - |
| `justification` | Text | igen | Indoklás (új pozíció/helyettesítés/kapacitásbővítés) | Belső jóváhagyás |
| `employment_start_date` | Date | igen | Tervezett munkakezdés | - |
| `is_replacement` | Boolean | igen | Helyettesítés-e | - |
| `replaced_employment_id` | UUID | nem | Kit helyettesít (ha is_replacement=true) | employment-entity.md |
| `status` | Enum | igen | draft/pending_approval/approved/rejected/cancelled | Jóváhagyási folyamat |
| `requested_by_id` | UUID | igen | Ki kérte (vezető) | person-entity.md |
| `requested_date` | DateTime | igen | Kérés dátuma | - |
| `approved_by_id` | UUID | nem | Ki hagyta jóvá | person-entity.md |
| `approved_date` | DateTime | nem | Jóváhagyás dátuma | - |
| `rejection_reason` | Text | nem | Elutasítás indoka | - |
| `budget_code` | String | nem | Költségvetési kód | Controlling/pénzügy |
| `salary_range_min` | Decimal | nem | Fizetési sáv alsó határ | compensation-entity.md |
| `salary_range_max` | Decimal | nem | Fizetési sáv felső határ | compensation-entity.md |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.2 JobPosting (Álláshirdetés)

Publikált álláshirdetés belső vagy külső platformokon.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `posting_number` | String | igen | Egyedi hirdetési szám (pl. JOB-2026-001) | - |
| `job_requisition_id` | UUID | igen | Melyik igényhez kapcsolódik | JobRequisition |
| `employment_type` | Enum | igen | mt/kjt/kttv/kit/puetv/eszjtv/kut | Jogviszony típusa |
| `job_title` | String | igen | Hirdetett munkakör megnevezése | - |
| `job_description` | Text | igen | Feladatok részletes leírása | - |
| `requirements` | Text | igen | Elvárások (végzettség, tapasztalat, kompetenciák) | Kit. 45. §, Kttv. 102. § |
| `benefits` | Text | nem | Juttatások felsorolása | - |
| `salary_range_display` | String | nem | Megjelenített fizetési sáv (pl. "bruttó 500-700 eFt") | Ebktv. alapján ajánlott |
| `location` | String | igen | Munkavégzés helye | Mt. 45. §, Kit. 45. § |
| `employment_level` | String | nem | Foglalkoztatás mértéke (teljes/részmunkaidő %-ban) | Mt. 85. § |
| `posting_type` | Enum | igen | internal/external/both | - |
| `is_public_sector` | Boolean | igen | Közszféra-e (kozigallas kötelezettség) | Kit., Kttv., Kjt. |
| `kozigallas_id` | String | nem | Közigállás.hu azonosító (ha is_public_sector=true) | Kit. 45. §, Kttv. 102. §, Kjt. 45/A. § |
| `kozigallas_published_date` | DateTime | nem | Közigállás.hu közzététel időpontja | Közigállás API |
| `application_deadline` | DateTime | igen | Jelentkezési határidő | Kit. 45. § (min 8 nap) |
| `posting_start_date` | DateTime | igen | Hirdetés kezdete | - |
| `posting_end_date` | DateTime | nem | Hirdetés vége (ha lezárva) | - |
| `status` | Enum | igen | draft/active/paused/closed/filled/cancelled | - |
| `requires_security_clearance` | Boolean | igen | Biztonsági ellenőrzés szükséges-e | Kit., Kttv. |
| `requires_ethical_certificate` | Boolean | igen | Erkölcsi bizonyítvány szükséges-e | Kit. 50. §, Kttv. 107. § |
| `requires_wealth_declaration` | Boolean | igen | Vagyonnyilatkozat szükséges-e | Kit. 131-135. § |
| `application_instructions` | Text | igen | Jelentkezési útmutató | - |
| `contact_person_id` | UUID | nem | Kapcsolattartó személy | person-entity.md |
| `external_urls` | JSON | nem | Külső platformok URL-jei (profession.hu, LinkedIn, stb.) | - |
| `view_count` | Integer | igen | Megtekintések száma | Analitika |
| `application_count` | Integer | igen | Beérkezett jelentkezések száma | Analitika |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.3 Candidate (Pályázó/Jelölt)

Pályázó személye és profilja. Elkülönül a Person entitástól, mivel nem minden pályázó válik munkatárssá.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `person_id` | UUID | nem | Person entitás (ha már munkatárs vagy volt munkatárs) | person-entity.md |
| `external_candidate_id` | String | nem | Külső rendszer azonosító (ha ATS integráció) | - |
| `first_name` | String | igen | Keresztnév | GDPR Art. 6(1)(b) + Art. 88 |
| `last_name` | String | igen | Vezetéknév | GDPR Art. 6(1)(b) + Art. 88 |
| `email` | String | igen | Email cím | GDPR Art. 6(1)(b) |
| `phone` | String | nem | Telefonszám | GDPR Art. 6(1)(b) |
| `address` | String | nem | Lakcím | GDPR Art. 6(1)(b) + Mt. 7. § |
| `birth_date` | Date | nem | Születési dátum | GDPR Art. 6(1)(b) + Mt. 8. § |
| `cv_document_id` | UUID | nem | Önéletrajz dokumentum | document-entity.md |
| `cover_letter_document_id` | UUID | nem | Motivációs levél | document-entity.md |
| `linkedin_url` | String | nem | LinkedIn profil | - |
| `source` | Enum | igen | website/linkedin/profession_hu/referral/kozigallas/other | Forrás tracking |
| `referrer_employee_id` | UUID | nem | Ajánló munkatárs (ha source=referral) | person-entity.md |
| `gdpr_consent_given` | Boolean | igen | GDPR hozzájárulás megadva-e | GDPR Art. 7 |
| `gdpr_consent_date` | DateTime | nem | Hozzájárulás dátuma | GDPR Art. 7 |
| `gdpr_consent_text` | Text | nem | Hozzájárulási nyilatkozat szövege | GDPR Art. 7(1) |
| `data_retention_until` | Date | igen | Meddig tárolható az adat | GDPR Art. 5(1)(e) |
| `status` | Enum | igen | active/hired/rejected/withdrawn/blacklisted | - |
| `blacklist_reason` | Text | nem | Feketelistázás oka (ha status=blacklisted) | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | nem | Létrehozó (lehet automatikus webformból) | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | nem | Módosító | Audit trail |

### 2.4 Application (Jelentkezés)

Egy pályázó jelentkezése egy konkrét álláshirdetésre.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `application_number` | String | igen | Egyedi jelentkezési szám (pl. APP-2026-001234) | - |
| `job_posting_id` | UUID | igen | Melyik álláshirdetésre | JobPosting |
| `candidate_id` | UUID | igen | Melyik pályázó | Candidate |
| `application_date` | DateTime | igen | Jelentkezés időpontja | - |
| `current_stage` | Enum | igen | new/screening/interview/assessment/offer/hired/rejected/withdrawn | Recruitment pipeline |
| `stage_updated_date` | DateTime | igen | Aktuális szakasz dátuma | - |
| `overall_rating` | Integer | nem | Összesített értékelés (1-5) | - |
| `screening_result` | Enum | nem | passed/failed/pending | Szűrés eredménye |
| `screening_notes` | Text | nem | Szűrési megjegyzések | - |
| `screened_by_id` | UUID | nem | Ki végezte a szűrést | person-entity.md |
| `screened_date` | DateTime | nem | Szűrés dátuma | - |
| `rejection_reason` | Enum | nem | underqualified/overqualified/cultural_fit/other | - |
| `rejection_notes` | Text | nem | Elutasítás részletes indoklás | - |
| `rejected_by_id` | UUID | nem | Ki utasította el | person-entity.md |
| `rejected_date` | DateTime | nem | Elutasítás dátuma | - |
| `withdrawn_reason` | Text | nem | Visszalépés indoka (pályázó oldaláról) | - |
| `withdrawn_date` | DateTime | nem | Visszalépés dátuma | - |
| `hired_date` | DateTime | nem | Felvétel dátuma | - |
| `resulting_employment_id` | UUID | nem | Létrejött Employment entitás (ha hired) | employment-entity.md |
| `attachments` | JSON | nem | Csatolt dokumentumok ID-i | document-entity.md |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.5 Interview (Interjú)

Kiválasztási interjúk és értékelések.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `application_id` | UUID | igen | Melyik jelentkezéshez | Application |
| `interview_type` | Enum | igen | phone_screening/technical/hr/panel/final | Interjú típusa |
| `interview_round` | Integer | igen | Hanyadik kör (1, 2, 3...) | - |
| `scheduled_date` | DateTime | igen | Tervezett időpont | - |
| `duration_minutes` | Integer | igen | Tervezett időtartam (perc) | - |
| `location` | String | nem | Helyszín (vagy "online") | - |
| `meeting_link` | String | nem | Online meeting link (Teams, Zoom) | - |
| `status` | Enum | igen | scheduled/completed/cancelled/no_show | - |
| `actual_start_time` | DateTime | nem | Tényleges kezdés | - |
| `actual_end_time` | DateTime | nem | Tényleges befejezés | - |
| `interviewer_ids` | JSON | igen | Interjúztatók ID-i (Person UUID-k) | person-entity.md |
| `overall_rating` | Integer | nem | Összesített értékelés (1-5) | - |
| `recommendation` | Enum | nem | strong_yes/yes/maybe/no/strong_no | - |
| `notes` | Text | nem | Általános megjegyzések | - |
| `cancellation_reason` | Text | nem | Lemondás oka | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.6 InterviewEvaluation (Interjú értékelés)

Interjúztatók egyéni értékelései.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `interview_id` | UUID | igen | Melyik interjúhoz | Interview |
| `interviewer_id` | UUID | igen | Melyik interjúztató | person-entity.md |
| `criteria_name` | String | igen | Értékelési szempont neve (pl. "Szakmai tudás", "Kommunikáció") | - |
| `rating` | Integer | igen | Értékelés (1-5) | - |
| `comments` | Text | nem | Megjegyzések az adott szempontra | - |
| `created_at` | DateTime | igen | Értékelés időpontja | Audit trail |

### 2.7 Assessment (Felmérés/Teszt)

Különböző értékelő eszközök (képességteszt, személyiségteszt, szakmai feladat).

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `application_id` | UUID | igen | Melyik jelentkezéshez | Application |
| `assessment_type` | Enum | igen | cognitive/personality/technical/case_study/presentation | Teszt típusa |
| `assessment_name` | String | igen | Teszt megnevezése | - |
| `provider` | String | nem | Szolgáltató (pl. "SHL", "Psytech", belső) | - |
| `external_assessment_id` | String | nem | Külső rendszer azonosító | - |
| `assigned_date` | DateTime | igen | Kijelölés dátuma | - |
| `due_date` | DateTime | nem | Határidő | - |
| `completed_date` | DateTime | nem | Teljesítés dátuma | - |
| `status` | Enum | igen | assigned/in_progress/completed/expired | - |
| `score` | Decimal | nem | Pontszám (ha numerikus) | - |
| `max_score` | Decimal | nem | Maximális pontszám | - |
| `percentile` | Integer | nem | Percentilis (összehasonlító norm) | - |
| `result_summary` | Text | nem | Eredmény összefoglaló | - |
| `result_document_id` | UUID | nem | Részletes eredmény dokumentum | document-entity.md |
| `pass_fail` | Enum | nem | passed/failed/not_applicable | - |
| `evaluated_by_id` | UUID | nem | Ki értékelte (ha manuális) | person-entity.md |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.8 JobOffer (Állásjánlat)

Hivatalos munkavégzésre irányuló jogviszony ajánlat.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `offer_number` | String | igen | Egyedi ajánlat szám (pl. OFFER-2026-001) | - |
| `application_id` | UUID | igen | Melyik jelentkezéshez | Application |
| `candidate_id` | UUID | igen | Melyik jelölt | Candidate |
| `employment_type` | Enum | igen | mt/kjt/kttv/kit/puetv/eszjtv/kut | Jogviszony típusa |
| `job_title` | String | igen | Felajánlott munkakör | - |
| `organization_unit_id` | UUID | igen | Szervezeti egység | organization-entity.md |
| `position_id` | UUID | nem | Pozíció (ha van) | position-entity.md |
| `start_date` | Date | igen | Munkakezdés dátuma | Mt. 45. §, Kit. 47. § |
| `employment_level` | Decimal | igen | Foglalkoztatás mértéke (1.0 = teljes munkaidő) | Mt. 85. § |
| `probation_period_months` | Integer | nem | Próbaidő hónapban (Mt: max 3/6, Kjt: max 3) | Mt. 37. §, Kjt. 41. § |
| `base_salary` | Decimal | igen | Alapbér (bruttó Ft/hó) | compensation-entity.md |
| `salary_supplement` | Decimal | nem | Pótlékok összesen | compensation-entity.md |
| `benefits_summary` | Text | nem | Juttatások összefoglalója | - |
| `kjt_salary_grade` | String | nem | Kjt. besorolás (ha kjt) | Kjt. 2. melléklet |
| `kttv_classification` | String | nem | Kttv. besorolás (ha kttv) | Kttv. 152-157. § |
| `kit_classification` | String | nem | Kit. besorolás (ha kit) | Kit. 85-86. § |
| `offer_template_id` | UUID | nem | Ajánlat sablon | document-entity.md |
| `offer_document_id` | UUID | nem | Generált ajánlat dokumentum | document-entity.md |
| `offer_date` | Date | igen | Ajánlat kiállítás dátuma | - |
| `expiration_date` | Date | igen | Ajánlat lejárat dátuma | - |
| `sent_date` | DateTime | nem | Ajánlat elküldés időpontja | - |
| `sent_method` | Enum | nem | email/postal/in_person | - |
| `status` | Enum | igen | draft/sent/accepted/rejected/expired/withdrawn | - |
| `candidate_response_date` | DateTime | nem | Jelölt válaszának dátuma | - |
| `rejection_reason` | Text | nem | Elutasítás oka (jelölt részéről) | - |
| `approved_by_id` | UUID | nem | Ki hagyta jóvá az ajánlatot | person-entity.md |
| `approved_date` | DateTime | nem | Jóváhagyás dátuma | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.9 OnboardingTask (Munkába állási feladat)

Új belépők feladatlistája (HR, IT, vezető, munkatárs feladatok).

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `onboarding_plan_id` | UUID | igen | Melyik onboarding tervhez | OnboardingPlan |
| `task_template_id` | UUID | nem | Sablon feladat (ha van) | OnboardingTaskTemplate |
| `task_name` | String | igen | Feladat neve | - |
| `task_description` | Text | nem | Feladat leírása | - |
| `task_category` | Enum | igen | hr_admin/it_setup/training/documentation/equipment/access | Feladat típusa |
| `assigned_to_id` | UUID | nem | Kire van kiosztva (HR, IT, vezető) | person-entity.md |
| `assigned_role` | Enum | nem | hr/it/manager/buddy/new_hire | Szerep |
| `due_date` | Date | nem | Határidő | - |
| `due_offset_days` | Integer | nem | Határidő offsetje munkakezdéstől (pl. -5, 0, +7) | - |
| `status` | Enum | igen | not_started/in_progress/completed/skipped | - |
| `completed_date` | DateTime | nem | Teljesítés dátuma | - |
| `completed_by_id` | UUID | nem | Ki teljesítette | person-entity.md |
| `completion_notes` | Text | nem | Teljesítési megjegyzések | - |
| `related_document_id` | UUID | nem | Kapcsolódó dokumentum | document-entity.md |
| `is_mandatory` | Boolean | igen | Kötelező-e | - |
| `sort_order` | Integer | igen | Sorrend | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.10 OnboardingPlan (Munkába állási terv)

Egy új belépő teljes onboarding folyamata.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `employment_id` | UUID | igen | Melyik Employment-hez | employment-entity.md |
| `job_offer_id` | UUID | nem | Melyik ajánlatból indult | JobOffer |
| `template_id` | UUID | nem | Használt sablon | OnboardingTemplate |
| `start_date` | Date | igen | Munkakezdés dátuma | - |
| `planned_end_date` | Date | nem | Tervezett onboarding lezárás | - |
| `actual_end_date` | Date | nem | Tényleges lezárás | - |
| `status` | Enum | igen | not_started/in_progress/completed/cancelled | - |
| `assigned_hr_id` | UUID | igen | Felelős HR munkatárs | person-entity.md |
| `manager_id` | UUID | igen | Közvetlen vezető | person-entity.md |
| `buddy_id` | UUID | nem | Mentor/buddy kolléga | person-entity.md |
| `completion_percentage` | Integer | igen | Készültségi fok (%) | Számított mező |
| `notes` | Text | nem | Általános megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.11 OnboardingTaskTemplate (Munkába állási feladat sablon)

Újrafelhasználható feladat sablonok különböző pozíciótípusokhoz.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `template_name` | String | igen | Sablon neve | - |
| `task_category` | Enum | igen | hr_admin/it_setup/training/documentation/equipment/access | - |
| `task_description` | Text | igen | Feladat leírása | - |
| `assigned_role` | Enum | igen | hr/it/manager/buddy/new_hire | - |
| `due_offset_days` | Integer | igen | Határidő offsetje munkakezdéstől | - |
| `is_mandatory` | Boolean | igen | Kötelező-e | - |
| `employment_type_filter` | JSON | nem | Melyik jogviszony típusokra (mt/kjt/kttv/kit...) | - |
| `position_category_filter` | JSON | nem | Melyik pozíció kategóriákra | - |
| `is_active` | Boolean | igen | Aktív sablon-e | - |
| `sort_order` | Integer | igen | Sorrend | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

## 3. GDPR kategorizáció és adatkezelés

### Candidate és Application entitások adatkezelése

#### Jogalap
- **GDPR Art. 6(1)(b)**: szerződéskötést megelőző lépések (munkaszerződés előkészítése)
- **GDPR Art. 88**: munkavállalói adatok kezelése foglalkoztatás keretében
- **GDPR Art. 7**: hozzájáruláson alapuló adatkezelés (ha túlmutat a szükségesen)

#### Különleges adatkategóriák
- **Életrajzban szereplő érzékeny adatok**: ha a pályázó önkéntesen megad (pl. egészségügyi állapot, fogyatékosság)
  - Külön hozzájárulás szükséges (GDPR Art. 9(2)(a))
  - Javasolt: strukturált űrlap helyett fájlfeltöltés, ne kényszerítsük az érzékeny adatok megadását

#### Adatmegőrzési idők
- **Sikeres pályázók**: Employment létrejöttével átkerülnek a Person entitásba
  - Candidate és Application adatok archiválhatók vagy törölhetők
- **Sikertelen pályázók**:
  - **Alapértelmezett**: 6 hónap a jelentkezés lezárásától (data_retention_until)
  - **Hozzájárulással**: maximum 1 év (újrahirdetések esetén értesítés céljából)
  - **Közszféra várólista**: 6 hónap (Kit./Kttv./Kjt. alapján újrahirdetésnél előnyös lehet)
  - **Feketelistázás**: legfeljebb 3 év (visszaélés, csalás esetén)
- **Törlési kötelezettség**: data_retention_until lejártával automatikus törlés vagy anonimizálás

#### Hozzáférési jogok
- **Pályázó**: saját adatokhoz való hozzáférés, helyesbítés, törlés kérése
- **HR munkatársak**: teljes hozzáférés aktív pályázatokhoz
- **Hiring manager**: olvasási jog a saját pozíciójához tartozó pályázatokhoz
- **Interjúztatók**: korlátozott hozzáférés (csak interjúhoz szükséges adatok)

### JobPosting - közigállás.hu integráció

- **Közigállás.hu API**: Kit., Kttv., Kjt. alapján kötelező közzététel
- **Személyes adat**: nincs (álláskövetelmények, nem személyes adatok)

## 4. Hozzáférési szintek

| Entitás | HR Admin | Hiring Manager | Interjúztató | Pályázó (önmaga) |
|---|---|---|---|---|
| JobRequisition | CRUD | Read (saját egység), Create | - | - |
| JobPosting | CRUD | Read (saját pozíciók) | Read | Read (publikus) |
| Candidate | CRUD | Read (jelentkezett pozíciókhoz) | Read (interjúzott jelöltek) | Read/Update (saját) |
| Application | CRUD | Read/Update (saját pozíciókhoz) | Read (saját interjúzott) | Read (saját) |
| Interview | CRUD | Read/Update (saját pozíciókhoz) | Read/Update (saját interjúk) | Read (saját) |
| InterviewEvaluation | Read | Read (saját pozíciókhoz) | CRUD (saját értékelések) | - |
| Assessment | CRUD | Read (saját pozíciókhoz) | Read | Read/Update (saját) |
| JobOffer | CRUD | Read (saját pozíciókhoz) | - | Read (saját) |
| OnboardingPlan | CRUD | Read/Update (saját beosztottak) | Read | Read (saját) |
| OnboardingTask | CRUD | Read/Update (assigned_to) | - | Read/Update (ha assigned=new_hire) |

## 5. Validációs szabályok

### JobPosting
- **Közigállás kötelezettség**: `is_public_sector=true` esetén kötelező a `kozigallas_id`
- **Jelentkezési határidő**: Kit. 45. § alapján minimum 8 nap a közzétételhez
- **Ebktv. compliance**: hirdetés nem tartalmazhat diszkriminatív elemeket

### Application
- **Határidő ellenőrzés**: application_date <= job_posting.application_deadline
- **Duplicate check**: egy Candidate maximum egyszer jelentkezhet ugyanarra a JobPosting-ra

### Interview
- **Időpont ütközés**: ugyanaz az interviewer_id nem lehet egyszerre több interjún
- **Értékelés kötelezettség**: status=completed esetén overall_rating és recommendation kitöltése ajánlott

### JobOffer
- **Lejárat ellenőrzés**: expiration_date > offer_date
- **Próbaidő limit**:
  - Mt.: maximum 3 hónap (vezető 6 hónap)
  - Kjt.: maximum 3 hónap
  - Kit./Kttv.: maximum 3 hónap
- **Besorolás kötelezettség**:
  - `employment_type=kjt` → `kjt_salary_grade` kötelező
  - `employment_type=kttv` → `kttv_classification` kötelező
  - `employment_type=kit` → `kit_classification` kötelező

### OnboardingTask
- **due_date számítás**: `onboarding_plan.start_date + due_offset_days`
- **Kötelező feladatok**: `is_mandatory=true` feladatok completion szükséges az onboarding lezárásához

## 6. Integrációs pontok

### Külső rendszerek
- **Közigállás.hu API**:
  - JobPosting publikálása (Kit., Kttv., Kjt. esetén kötelező)
  - Státusz szinkronizáció
  - Jelentkezések fogadása (ha közigállás oldalról érkezik)
- **ATS (Applicant Tracking System)**:
  - profession.hu, LinkedIn Recruiter, egyéb álláshirdetési platformok
  - Kétirányú szinkronizáció: hirdetés publikálás, jelentkezések importálása
- **Email / SMS értesítések**:
  - Pályázók értesítése (jelentkezés visszaigazolás, interjú meghívó, ajánlat, elutasítás)
  - HR és Hiring Manager értesítések (új jelentkezés, interjú emlékeztető)
- **Naptár integráció**:
  - Interview időpontok szinkronizálása (Outlook, Google Calendar)
- **Assessment platformok**:
  - SHL, Psytech, Cubiks, egyéb online tesztek
  - API integráció vagy manuális eredmény feltöltés

### Belső entitások
- **Person**: Candidate → Person konverzió felvételkor
- **Employment**: JobOffer accepted → Employment létrehozása
- **Position**: JobRequisition → Position kapcsolat
- **Organization**: JobPosting → OrgUnit kapcsolat
- **Compensation**: JobOffer → CompensationElement létrehozása
- **Document**: CV, motivációs levél, ajánlat dokumentumok, onboarding dokumentumok

## 7. Kapcsolódó entitások

- **person-entity.md**: Person (felvett munkatárs), EmergencyContact
- **employment-entity.md**: Employment (JobOffer-ből létrejött jogviszony)
- **position-entity.md**: Position, PositionHistory
- **organization-entity.md**: OrgUnit, WorkLocation
- **compensation-entity.md**: CompensationElement, BenefitEntitlement
- **document-entity.md**: Document (CV, ajánlat, szerződés tervezet, onboarding dokumentumok)
- **qualification-entity.md**: Education, Language, ProfessionalExam (jelölt kvalifikációi)

## 8. Üzleti folyamatok

### 8.1 Tipikus recruitment flow (magánszektor - Mt.)

1. **Igény keletkezése**: JobRequisition létrehozása (vezető kezdeményezi)
2. **Jóváhagyás**: JobRequisition status=approved
3. **Hirdetés létrehozása**: JobPosting creation, publication external platforms
4. **Jelentkezések érkezése**: Application entitások létrehozása
5. **Szűrés**: HR screening (Application.screening_result)
6. **Interjúk**:
   - 1. kör: HR interjú (Interview type=hr)
   - 2. kör: Szakmai interjú (Interview type=technical, type=panel)
   - 3. kör: Vezetői interjú (Interview type=final)
7. **Assessmentek**: képességteszt, személyiségteszt (ha szükséges)
8. **Ajánlat**: JobOffer creation, approval, sending
9. **Elfogadás**: JobOffer status=accepted → Employment létrehozása
10. **Onboarding**: OnboardingPlan és OnboardingTask-ok létrehozása

### 8.2 Közszféra recruitment flow (Kit./Kttv./Kjt.)

1. **Igény**: JobRequisition jóváhagyása
2. **Hirdetés**: JobPosting creation
3. **Közigállás.hu publikálás**: KÖTELEZŐ (API call)
4. **Minimum 8 napos pályázati határidő**: Kit. 45. § compliance
5. **Formális pályázati csomag**:
   - Önéletrajz
   - Motivációs levél
   - Erkölcsi bizonyítvány (Kit. 50. §, Kttv. 107. §)
   - Végzettség igazoló dokumentumok
   - Összeférhetetlenségi nyilatkozat
6. **Pályázatok feldolgozása**: Application screening
7. **Interjú bizottság**: Interview (panel típus, több interjúztató)
8. **Döntés**: formális jegyzőkönyv
9. **Ajánlat**: JobOffer (kinevezési okirat előkészítése)
10. **Várólista**: sikertelen pályázók 6 hónapig megőrzése
11. **Onboarding**: OnboardingPlan (közszolgálati eskü, jognyilatkozatok)

### 8.3 Onboarding folyamat

**HR feladatok**:
- Személyi anyag összeállítása (Mt. 10. §, Kit./Kttv./Kjt. személyi anyag szabályok)
- Szerződés aláírása
- TAJ, adószám, bankszámlaszám gyűjtése
- Adatlapok kitöltése (bérszámfejtéshez)
- Munkaköri leírás átadása
- Munkavédelmi oktatás szervezése

**IT feladatok**:
- Email fiók létrehozása
- Hozzáférések beállítása (rendszerek, mappák)
- Eszközök kiadása (laptop, telefon, asztali gép)
- Szoftverek telepítése

**Vezető feladatok**:
- Fogadás első napon
- Csapatbemutatás
- Célok meghatározása (első 30/60/90 nap)
- Mentor/buddy kijelölése

**Munkatárs feladatok**:
- Kötelező tájékoztatók elolvasása (GDPR, compliance, biztonság)
- Nyilatkozatok aláírása
- Képzések elvégzése (e-learning)

## 9. Reporting és analitika

### Recruitment metrikák
- **Time to Fill**: JobRequisition.approved_date → JobOffer.accepted_date
- **Time to Hire**: JobPosting.posting_start_date → JobOffer.accepted_date
- **Cost per Hire**: recruitment costs / hires
- **Source effectiveness**: Candidate.source alapján konverziós arányok
- **Application funnel**: stage-ek közötti konverzió (Application.current_stage)
- **Offer acceptance rate**: sent offers / accepted offers
- **Quality of Hire**: új belépők első évi teljesítménye (performance-review-entity.md kapcsolat)

### Onboarding metrikák
- **Onboarding completion rate**: OnboardingPlan.completion_percentage
- **Time to productivity**: új belépő mennyi idő alatt éri el a teljes produktivitást
- **New hire retention**: első 90 nap/6 hónap/1 év retenciós ráta

### Compliance reporting
- **Közigállás.hu reporting**: Kit./Kttv./Kjt. hirdetések száma, válaszidő
- **GDPR compliance**: Candidate adatok törlése data_retention_until alapján
- **Ebktv. compliance**: diszkriminációmentes hirdetések auditálása

## 10. Nyitott kérdések és jövőbeli fejlesztések

### Technológiai kérdések
1. **ATS vs saját fejlesztés**: külső ATS integráció vagy teljes saját fejlesztésű recruitment rendszer?
2. **Assessment platformok**: mely assessment eszközök integrációja prioritás?
3. **Videó interjú integráció**: HireVue, Spark Hire, vagy egyéb platform?
4. **AI-powered screening**: CV parsing, automatikus előszűrés gépi tanulással?

### Funkcionális fejlesztések
1. **Referral program**: munkatársi ajánlási bónusz kezelése
2. **Talent pool**: korábbi pályázók, passzív jelöltek adatbázisa
3. **Employer branding**: karrieroldal CMS, tartalom menedzsment
4. **Candidate experience**: jelöltek számára saját portál (jelentkezés státusza, interjú feedback)

### Compliance és jogi
1. **Közigállás.hu API változások**: API specifikáció frissítések követése
2. **Ebktv. auditálás**: automatikus hirdetés-screening diszkrimináció ellen
3. **Adatvédelmi hatásvizsgálat (DPIA)**: GDPR Art. 35 szerinti vizsgálat szükséges-e?

### Integrációk
1. **LinkedIn Recruiter**: API integráció jelöltek importálásához
2. **profession.hu, cvonline.hu**: álláshirdetési API-k
3. **Email marketing**: pályázói kommunikáció automatizálása
4. **SMS értesítések**: interjú emlékeztetők
5. **E-aláírás**: ajánlat és szerződés digitális aláírása (pl. Küt. elektronikus aláírás)

### Folyamat optimalizáció
1. **Recruitment automation**: automatikus interjúidőpont egyeztetés, AI chatbot pályázóknak
2. **Collaborative hiring**: hiring team együttműködési funkciók, megosztott jegyzetek
3. **Structured interviews**: sablon kérdéssorok, értékelési rubrikák
4. **Diversity hiring**: inkluzív toborzási metrikák, célok követése
