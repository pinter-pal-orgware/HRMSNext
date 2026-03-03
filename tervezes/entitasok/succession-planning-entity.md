# SuccessionPlanning (Utódlástervezés és Tehetségmenedzsment) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer
> **Verzió:** 0.1 – első tervezet
> **Utolsó frissítés:** 2025. február
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md, employment-entity.md, organization-entity.md, position-entity.md, performance-review-entity.md, learning-entity.md, compensation-entity.md

---

## 1. Az entitás célja és hatóköre

Az utódlástervezési és tehetségmenedzsment entitások biztosítják a szervezet kritikus pozícióinak folyamatos betöltését, a kulcstehetségek azonosítását, fejlesztését és megtartását, valamint a vezetői utánpótlás tervezését.

### Hatókör

Az utódlástervezési entitások lefedik:
- **Kulcspozíciók azonosítása**: stratégiai fontosságú, nehezen betölthető pozíciók
- **Utódjelöltek kiválasztása**: ready now / 1-2 év / 3+ év kategóriák
- **Tehetségek (talent pool) menedzsmentje**: magas potenciálú munkatársak azonosítása
- **9-box grid értékelés**: teljesítmény vs. potenciál mátrix
- **Fejlesztési tervek**: utódjelöltek célzott fejlesztése
- **Kockázatmenedzsment**: betöltetlen kulcspozíciók kockázatának kezelése
- **Tudásátadás**: kritikus tudás dokumentálása és átadása
- **Vezetői utánpótlás (leadership pipeline)**: vezetői szintek közötti előrelépés tervezése

### Foglalkoztatási típus specifikus követelmények

#### Közszféra (Kit., Kttv., Kjt.)
- **Helyettesítési szabályok**: Kit. 49. §, Kttv. 44. § - vezető helyettesítésének kijelölése kötelező
- **Kinevezési feltételek**: Kit. 54-57. § - közszolgálati szakvizsga, nyelvvizsga követelmények
- **Besorolási kategóriák**: Kit. 35-40. § - álláshely-alapú utódlástervezés
- **Pályáztatási kötelezettség**: minden vezetői pozíció betöltése nyilvános pályázattal (közigállás.hu)

#### Magánszektor (Mt.)
- **Nincs kötelező utódlástervezés**: üzleti döntés alapú
- **Versenytilalmi megállapodás**: Mt. 228. § - kulcsemberek távozása esetén
- **Tudásmegőrzés**: üzleti titok védelme (Ptk. 2:47. §)

#### Nagyvállalati best practice
- **Kritikus pozíciók**: CEO, CFO, CTO, divízióvezetők
- **Tehetségmenedzsment programok**: HiPo (High Potential), LEAP (Leadership Excellence Acceleration Program)
- **9-box grid**: teljesítmény-potenciál mátrix évente

## 2. Entitásstruktúra

### 2.1 KeyPosition (Kulcspozíció)

Stratégiai fontosságú pozíciók, amelyekre utódlástervezés szükséges.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `position_id` | UUID | igen | Melyik pozíció | position-entity.md |
| `organization_unit_id` | UUID | igen | Szervezeti egység | organization-entity.md |
| `key_position_category` | Enum | igen | executive/senior_management/middle_management/technical_expert/critical_operational | - |
| `business_criticality` | Enum | igen | critical/high/medium | Üzleti hatás |
| `vacancy_risk` | Enum | igen | imminent/high/medium/low | Betöltetlen pozíció kockázata |
| `replacement_difficulty` | Enum | igen | very_difficult/difficult/moderate/easy | Betöltési nehézség |
| `replacement_cost_estimate` | Decimal | nem | Becsült helyettesítési költség (Ft) | Toborzás + betanítás |
| `knowledge_transfer_required` | Boolean | igen | Tudásátadás szükséges-e | - |
| `estimated_vacancy_impact` | Text | nem | Betöltetlen pozíció hatása az üzletre | - |
| `succession_plan_status` | Enum | igen | no_plan/in_progress/plan_ready/under_review | - |
| `incumbent_employment_id` | UUID | nem | Jelenlegi pozícióbetöltő | employment-entity.md |
| `incumbent_tenure_years` | Decimal | számított | Jelenlegi betöltő pozícióban eltöltött idő (év) | - |
| `incumbent_retirement_eligibility_date` | Date | nem | Nyugdíjjogosultság várható dátuma | - |
| `incumbent_flight_risk` | Enum | nem | high/medium/low | Távozási kockázat |
| `number_of_successors_required` | Integer | igen | Hány utódjelölt szükséges (általában 2-3) | Best practice |
| `current_successors_count` | Integer | számított | Jelenleg hány jelölt van | - |
| `backup_appointed` | Boolean | igen | Azonnali helyettes kijelölve-e | Kit. 49. § (közszféra kötelező) |
| `backup_employment_id` | UUID | nem | Azonnali helyettes | employment-entity.md |
| `last_review_date` | Date | nem | Utolsó felülvizsgálat dátuma | - |
| `next_review_date` | Date | nem | Következő felülvizsgálat (általában éves) | - |
| `reviewed_by_id` | UUID | nem | Felülvizsgáló vezető | person-entity.md |
| `approved_by_id` | UUID | nem | Jóváhagyó (HR igazgató, CEO) | person-entity.md |
| `approved_date` | Date | nem | Jóváhagyás dátuma | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.2 SuccessionPlan (Utódlási terv)

Egy kulcspozícióhoz tartozó utódlási terv (jelöltek + fejlesztési akciók).

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `key_position_id` | UUID | igen | Melyik kulcspozícióhoz | KeyPosition |
| `plan_name` | String | igen | Terv neve (pl. "CFO Succession Plan 2026") | - |
| `plan_year` | Integer | igen | Tervezési év | - |
| `status` | Enum | igen | draft/active/under_review/approved/archived | - |
| `planning_horizon_years` | Integer | igen | Tervezési horizont (általában 3-5 év) | - |
| `succession_trigger` | Enum | nem | retirement/promotion/resignation/termination/emergency/planned_rotation | Mi váltja ki |
| `anticipated_vacancy_date` | Date | nem | Várható megüresedés dátuma | - |
| `emergency_successor_id` | UUID | nem | Vészhelyzeti utód (ready now) | SuccessorCandidate |
| `primary_successor_id` | UUID | nem | Elsődleges utód | SuccessorCandidate |
| `total_successors_count` | Integer | számított | Összes jelölt száma | - |
| `knowledge_transfer_plan` | Text | nem | Tudásátadási terv | - |
| `knowledge_transfer_status` | Enum | nem | not_started/in_progress/completed | - |
| `critical_competencies` | JSON | nem | Kritikus kompetenciák a pozícióhoz | - |
| `development_budget` | Decimal | nem | Fejlesztési költségkeret (Ft) | - |
| `development_budget_spent` | Decimal | nem | Elköltött összeg | - |
| `risk_assessment` | Text | nem | Kockázatértékelés | - |
| `mitigation_actions` | Text | nem | Kockázatcsökkentő akciók | - |
| `reviewed_by_id` | UUID | nem | Felülvizsgáló | person-entity.md |
| `reviewed_date` | Date | nem | Felülvizsgálat dátuma | - |
| `approved_by_id` | UUID | nem | Jóváhagyó | person-entity.md |
| `approved_date` | Date | nem | Jóváhagyás dátuma | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.3 SuccessorCandidate (Utódjelölt)

Egy kulcspozícióhoz jelölt utód.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `succession_plan_id` | UUID | igen | Melyik utódlási tervhez | SuccessionPlan |
| `key_position_id` | UUID | igen | Melyik kulcspozícióhoz (denormalizált) | KeyPosition |
| `candidate_employment_id` | UUID | igen | Jelölt munkatárs | employment-entity.md |
| `candidate_current_position_id` | UUID | igen | Jelölt jelenlegi pozíciója | position-entity.md |
| `readiness_level` | Enum | igen | ready_now/ready_1_2_years/ready_3_plus_years/not_ready | Készenléti szint |
| `readiness_assessment_date` | Date | igen | Készenléti értékelés dátuma | - |
| `successor_rank` | Integer | igen | Sorrend (1=elsődleges, 2=másodlagos, stb.) | - |
| `nomination_date` | Date | igen | Jelölés dátuma | - |
| `nominated_by_id` | UUID | igen | Ki jelölte | person-entity.md |
| `nomination_reason` | Text | nem | Jelölés indoka | - |
| `current_performance_rating` | Enum | nem | outstanding/exceeds/meets/needs_improvement/unsatisfactory | performance-review-entity.md |
| `potential_rating` | Enum | nem | high/medium/low | 9-box grid |
| `nine_box_category` | Enum | számított | star/high_potential/solid_performer/inconsistent/underperformer/talent_risk/high_professional/core_player/low_contributor | 9-box grid |
| `strengths` | Text | nem | Erősségek | - |
| `development_needs` | Text | nem | Fejlesztendő területek | - |
| `critical_experiences_needed` | JSON | nem | Hiányzó kritikus tapasztalatok | - |
| `development_plan_id` | UUID | nem | Egyéni fejlesztési terv | learning-entity.md IndividualDevelopmentPlan |
| `development_actions_count` | Integer | számított | Fejlesztési akciók száma | - |
| `development_progress` | Integer | nem | Fejlesztési terv haladás (%) | - |
| `mobility_willingness` | Enum | nem | willing_relocate/willing_travel/location_bound | Mobilitási hajlandóság |
| `flight_risk` | Enum | nem | high/medium/low | Távozási kockázat |
| `retention_actions` | Text | nem | Megtartási akciók | - |
| `interim_assignments` | JSON | nem | Ideiglenes megbízások (stretch assignments) | - |
| `mentoring_provided` | Boolean | nem | Mentorálás biztosított-e | - |
| `mentor_id` | UUID | nem | Mentor személye | person-entity.md |
| `status` | Enum | igen | active/on_hold/promoted/withdrawn/no_longer_suitable | - |
| `status_change_date` | Date | nem | Státuszváltozás dátuma | - |
| `status_change_reason` | Text | nem | Státuszváltozás oka | - |
| `confidentiality_level` | Enum | igen | highly_confidential/confidential/restricted/open | Ki ismerheti |
| `candidate_aware` | Boolean | igen | Jelölt tud-e a jelölésről | - |
| `last_assessment_date` | Date | nem | Utolsó értékelés dátuma | - |
| `next_assessment_date` | Date | nem | Következő értékelés | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.4 TalentPool (Tehetség pool)

Tehetségek csoportosítása (HiPo, szakértők, jövő vezetői).

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `pool_name` | String | igen | Pool neve (pl. "High Potential 2026", "Future Leaders") | - |
| `pool_code` | String | igen | Pool kód (pl. "HIPO-2026") | - |
| `pool_type` | Enum | igen | high_potential/future_leaders/technical_experts/diverse_talent/early_career/other | - |
| `pool_description` | Text | nem | Leírás | - |
| `selection_criteria` | Text | igen | Beválogatási kritériumok | - |
| `entry_requirements` | JSON | nem | Strukturált belépési feltételek | - |
| `pool_year` | Integer | igen | Évfolyam | - |
| `max_members` | Integer | nem | Maximum létszám | - |
| `current_members_count` | Integer | számított | Jelenlegi létszám | - |
| `program_duration_months` | Integer | nem | Program időtartama (hónap) | - |
| `program_start_date` | Date | nem | Program kezdete | - |
| `program_end_date` | Date | nem | Program vége | - |
| `development_activities` | JSON | nem | Fejlesztési tevékenységek (képzések, projektek, mentorálás) | - |
| `budget_allocated` | Decimal | nem | Kiosztott költségkeret (Ft) | - |
| `budget_spent` | Decimal | nem | Elköltött összeg | - |
| `sponsor_id` | UUID | nem | Szponzor vezető (általában C-level) | person-entity.md |
| `program_manager_id` | UUID | nem | Program felelős (HR) | person-entity.md |
| `status` | Enum | igen | planned/active/completed/cancelled | - |
| `success_metrics` | JSON | nem | Siker mutatók (retention rate, promotion rate) | - |
| `graduation_criteria` | Text | nem | Kikerülési feltételek | - |
| `is_confidential` | Boolean | igen | Bizalmas-e a tagsági lista | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.5 TalentPoolMembership (Tehetség pool tagság)

Munkatársak tagsága tehetség pool-okban.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `talent_pool_id` | UUID | igen | Melyik pool | TalentPool |
| `employment_id` | UUID | igen | Melyik munkatárs | employment-entity.md |
| `entry_date` | Date | igen | Belépés dátuma | - |
| `exit_date` | Date | nem | Kilépés dátuma | - |
| `nomination_source` | Enum | igen | manager_nomination/hr_nomination/self_nomination/assessment_center/performance_review | Honnan került be |
| `nominated_by_id` | UUID | igen | Ki jelölte | person-entity.md |
| `nomination_reason` | Text | nem | Jelölés indoka | - |
| `selection_assessment_date` | Date | nem | Felvételi értékelés dátuma | - |
| `selection_assessment_score` | Decimal | nem | Felvételi pontszám | - |
| `status` | Enum | igen | active/graduated/exited/suspended | - |
| `exit_reason` | Enum | nem | completed_program/promoted/resigned/performance_issues/personal_reasons/other | Kilépési ok |
| `development_plan_id` | UUID | nem | Egyéni fejlesztési terv | learning-entity.md IndividualDevelopmentPlan |
| `mentor_assigned_id` | UUID | nem | Kijelölt mentor | person-entity.md |
| `sponsor_assigned_id` | UUID | nem | Kijelölt szponzor (senior leader) | person-entity.md |
| `assignments_completed` | Integer | nem | Teljesített stretch assignments száma | - |
| `trainings_completed` | Integer | nem | Teljesített képzések száma | - |
| `progress_rating` | Enum | nem | excellent/good/satisfactory/needs_improvement | Haladás értékelés |
| `mid_program_review_date` | Date | nem | Félidős értékelés dátuma | - |
| `final_review_date` | Date | nem | Záró értékelés dátuma | - |
| `graduation_date` | Date | nem | Kikerülés dátuma | - |
| `post_program_position_id` | UUID | nem | Program utáni pozíció | position-entity.md |
| `promoted_after_program` | Boolean | nem | Előléptetés a program után | - |
| `retention_12_months` | Boolean | nem | Bent maradt-e 12 hónappal a program után | Siker mutató |
| `member_aware` | Boolean | igen | Tag tudja-e, hogy a pool-ban van | - |
| `confidentiality_agreement_signed` | Boolean | nem | Titoktartási megállapodás aláírva | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.6 TalentAssessment (Tehetség értékelés - 9-box grid)

Teljesítmény-potenciál mátrix értékelés.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `employment_id` | UUID | igen | Értékelt munkatárs | employment-entity.md |
| `assessment_year` | Integer | igen | Értékelési év | - |
| `assessment_date` | Date | igen | Értékelés dátuma | - |
| `assessment_type` | Enum | igen | annual_talent_review/succession_planning/talent_pool_selection/promotion_review | - |
| `performance_rating` | Enum | igen | outstanding/exceeds_expectations/meets_expectations/needs_improvement/unsatisfactory | Y tengely |
| `performance_score` | Integer | nem | Teljesítmény pontszám (1-5) | - |
| `potential_rating` | Enum | igen | high/medium/low | X tengely |
| `potential_score` | Integer | nem | Potenciál pontszám (1-3) | - |
| `nine_box_category` | Enum | számított | star/high_potential/solid_performer/inconsistent/underperformer/talent_risk/high_professional/core_player/low_contributor | Mátrix cella |
| `readiness_for_next_level` | Enum | nem | ready_now/ready_1_2_years/ready_3_plus_years/not_ready | - |
| `leadership_potential` | Enum | nem | executive/senior_management/middle_management/individual_contributor | Vezetői szint potenciál |
| `technical_potential` | Enum | nem | expert/advanced/intermediate/beginner | Szakértői szint potenciál |
| `mobility_potential` | Enum | nem | high/medium/low | Mobilitási potenciál |
| `key_strengths` | Text | nem | Kulcserősségek | - |
| `development_areas` | Text | nem | Fejlesztendő területek | - |
| `critical_experiences_needed` | JSON | nem | Hiányzó kritikus tapasztalatok | - |
| `derailers` | Text | nem | Karrier kockázatok (pl. mikromenedzselés, konfliktuskezelés) | - |
| `recommended_actions` | Text | nem | Javasolt intézkedések | - |
| `succession_candidate_for` | JSON | nem | Melyik pozíciókra jelölt (position_id lista) | - |
| `talent_pool_recommended` | JSON | nem | Melyik talent pool-ba ajánlott | - |
| `retention_risk` | Enum | nem | high/medium/low | Távozási kockázat |
| `retention_actions` | Text | nem | Megtartási akciók | - |
| `compensation_adjustment_recommended` | Boolean | nem | Béremelés javasolt-e | - |
| `compensation_adjustment_amount` | Decimal | nem | Javasolt emelés összege (Ft vagy %) | - |
| `calibration_session_date` | Date | nem | Kalibrációs meeting dátuma | - |
| `calibration_session_attendees` | JSON | nem | Kalibrációs meeting résztvevői | - |
| `assessor_id` | UUID | igen | Értékelő vezető | person-entity.md |
| `reviewed_by_id` | UUID | nem | Felülvizsgáló (magasabb vezető) | person-entity.md |
| `approved_by_id` | UUID | nem | Jóváhagyó (HR igazgató) | person-entity.md |
| `approved_date` | Date | nem | Jóváhagyás dátuma | - |
| `confidentiality_level` | Enum | igen | highly_confidential/confidential | - |
| `employee_feedback_provided` | Boolean | nem | Visszajelzés adva-e a munkatársnak | - |
| `employee_feedback_date` | Date | nem | Visszajelzés dátuma | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.7 DevelopmentAction (Fejlesztési akció utódjelölteknek)

Utódjelöltek specifikus fejlesztési akcióinak nyomon követése.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `successor_candidate_id` | UUID | igen | Melyik utódjelölthöz | SuccessorCandidate |
| `employment_id` | UUID | igen | Munkatárs (denormalizált) | employment-entity.md |
| `action_type` | Enum | igen | training/stretch_assignment/job_rotation/mentoring/coaching/shadowing/special_project/international_assignment/other | - |
| `action_name` | String | igen | Akció megnevezése | - |
| `action_description` | Text | igen | Részletes leírás | - |
| `development_objective` | Text | igen | Fejlesztési cél | - |
| `competency_addressed` | String | nem | Fejlesztendő kompetencia | - |
| `priority` | Enum | igen | critical/high/medium/low | - |
| `planned_start_date` | Date | igen | Tervezett kezdés | - |
| `planned_end_date` | Date | igen | Tervezett befejezés | - |
| `actual_start_date` | Date | nem | Tényleges kezdés | - |
| `actual_end_date` | Date | nem | Tényleges befejezés | - |
| `status` | Enum | igen | planned/in_progress/completed/cancelled/on_hold | - |
| `completion_percentage` | Integer | nem | Haladás (%) | - |
| `assigned_by_id` | UUID | igen | Ki rendelte el | person-entity.md |
| `supervisor_id` | UUID | nem | Felelős vezető/mentor | person-entity.md |
| `training_catalog_id` | UUID | nem | Kapcsolódó képzés (ha training) | learning-entity.md TrainingCatalog |
| `stretch_assignment_position_id` | UUID | nem | Ideiglenes pozíció (ha stretch assignment) | position-entity.md |
| `rotation_department_id` | UUID | nem | Rotációs egység (ha job rotation) | organization-entity.md |
| `cost` | Decimal | nem | Költség (Ft) | - |
| `outcome` | Text | nem | Eredmény leírása | - |
| `effectiveness_rating` | Enum | nem | very_effective/effective/somewhat_effective/not_effective | - |
| `lessons_learned` | Text | nem | Tanulságok | - |
| `follow_up_required` | Boolean | nem | Követés szükséges-e | - |
| `follow_up_date` | Date | nem | Követés dátuma | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.8 KnowledgeTransferPlan (Tudásátadási terv)

Kritikus tudás átadásának tervezése és követése.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `succession_plan_id` | UUID | nem | Kapcsolódó utódlási terv | SuccessionPlan |
| `key_position_id` | UUID | igen | Melyik kulcspozíció | KeyPosition |
| `source_employment_id` | UUID | igen | Tudást átadó (incumbent) | employment-entity.md |
| `target_employment_id` | UUID | nem | Tudást átvevő (successor) | employment-entity.md |
| `plan_name` | String | igen | Terv neve | - |
| `knowledge_category` | Enum | igen | technical/business_process/relationships/institutional_knowledge/tools_systems/strategic | Tudás típusa |
| `critical_knowledge_areas` | JSON | igen | Kritikus tudásterületek listája | - |
| `knowledge_documentation_required` | Boolean | igen | Dokumentálás szükséges-e | - |
| `documentation_status` | Enum | nem | not_started/in_progress/completed | - |
| `documentation_location` | String | nem | Dokumentációk helye (wiki, SharePoint) | - |
| `transfer_method` | Enum | igen | shadowing/mentoring/documentation/training/job_overlap/video_recording/knowledge_base | - |
| `planned_start_date` | Date | igen | Tervezett kezdés | - |
| `planned_completion_date` | Date | igen | Tervezett befejezés | - |
| `actual_start_date` | Date | nem | Tényleges kezdés | - |
| `actual_completion_date` | Date | nem | Tényleges befejezés | - |
| `overlap_period_weeks` | Integer | nem | Átfedési időszak (hét) - mindketten jelen vannak | - |
| `status` | Enum | igen | planned/in_progress/completed/cancelled | - |
| `completion_percentage` | Integer | nem | Haladás (%) | - |
| `key_stakeholders` | JSON | nem | Kulcs érintettek (ügyfelek, partnerek) bemutatása | - |
| `stakeholder_introductions_completed` | Boolean | nem | Bemutatkozások megtörténtek-e | - |
| `effectiveness_assessment` | Text | nem | Hatékonyság értékelése | - |
| `gaps_identified` | Text | nem | Azonosított hiányosságok | - |
| `mitigation_actions` | Text | nem | Kockázatcsökkentő intézkedések | - |
| `responsible_person_id` | UUID | nem | Felelős személy (általában HR) | person-entity.md |
| `reviewed_by_id` | UUID | nem | Felülvizsgáló | person-entity.md |
| `reviewed_date` | Date | nem | Felülvizsgálat dátuma | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

## 3. GDPR kategorizáció és adatkezelés

### Utódlástervezési adatok kezelése

#### Jogalap
- **GDPR Art. 6(1)(b)**: szerződés teljesítése (munkaviszony keretében)
- **GDPR Art. 6(1)(f)**: jogos érdek (tehetségfejlesztés, üzletmenet folytonosság)
- **GDPR Art. 88**: munkavállalói adatok kezelése foglalkoztatás keretében

#### Adatkategóriák
- **Általános személyes adat**: teljesítmény értékelés, potenciál értékelés, fejlesztési igények
- **Különösen bizalmas üzleti adat**: utódlási tervek, tehetség pool tagság, 9-box besorolás
- **NEM különleges adat**: nem tartozik GDPR Art. 9 alá (nem egészségügyi, nem érzékeny)

#### Adatmegőrzési idők
- **Aktív utódlási tervek**: amíg aktuális + 3 év
- **Archivált tervek**: terv lezárása + 5 év
- **Tehetség értékelések**: munkaviszony megszűnése + 3 év
- **Fejlesztési akciók**: teljesítés + 3 év
- **Tudásátadási tervek**: végrehajtás + 3 év

### Bizalmassági szintek
- **Highly Confidential (szigorúan bizalmas)**: CEO utódlási terv, C-level utódlási tervek, HiPo listák
- **Confidential (bizalmas)**: vezetői utódlási tervek, 9-box értékelések
- **Restricted (korlátozott)**: fejlesztési akciók
- **Open (nyílt)**: publikus tehetségprogramok (ha nem bizalmas jellegűek)

### Hozzáférési jogok
- **Munkatárs**: saját fejlesztési terv, saját talent pool tagság (ha member_aware=true)
- **Közvetlen vezető**: beosztottak tehetség értékelése, fejlesztési akciók
- **Senior management**: divízió/egység utódlási tervei
- **C-level**: teljes utódlási portfólió
- **HR**: teljes hozzáférés (adminisztráció, koordináció)
- **HR Analytics**: aggregált riportok (személyazonosítás nélkül)

## 4. Hozzáférési szintek

| Entitás | HR Admin | C-Level | Senior Manager | Manager | Employee |
|---|---|---|---|---|---|
| KeyPosition | CRUD | Read (all) | Read (division) | Read (department) | - |
| SuccessionPlan | CRUD | Read (all) | Read (division) | Read (department) | - |
| SuccessorCandidate | CRUD | Read (all) | Read (division) | Read (department) | Read (saját, ha aware) |
| TalentPool | CRUD | Read | Read | - | - |
| TalentPoolMembership | CRUD | Read | Read (division) | - | Read (saját, ha aware) |
| TalentAssessment | CRUD | Read (all) | Read (division) | Read/Create (team) | - |
| DevelopmentAction | CRUD | Read | Read (division) | CRUD (team) | Read (saját) |
| KnowledgeTransferPlan | CRUD | Read | Read (division) | Read (department) | Read (ha érintett) |

## 5. Validációs szabályok

### KeyPosition
- **Utódjelöltek minimuma**: number_of_successors_required általában 2-3 (best practice)
- **Backup kötelező közszférában**: Kit./Kttv. alkalmazottnál backup_appointed=true kötelező
- **Éves felülvizsgálat**: next_review_date <= current_date + 12 hónap

### SuccessionPlan
- **Emergency successor kötelező**: critical kulcspozícióknál emergency_successor_id kötelező
- **Anticipált dátum validáció**: anticipated_vacancy_date figyelmeztető, ha < 6 hónap

### SuccessorCandidate
- **Readiness szintek**: readiness_level progression tracking (3+ years → 1-2 years → ready now)
- **Duplicate jelölés tiltása**: egy employment_id nem jelölhető többször ugyanarra a key_position_id-re (kivéve archived)
- **Confidentiality enforcement**: confidentiality_level=highly_confidential esetén hozzáférés korlátozás

### TalentAssessment
- **9-box calculation**: nine_box_category automatic számítás performance_rating × potential_rating alapján
- **Éves értékelés**: assessment_year = current_year, évente egyszer
- **Calibration required**: nagy szervezetekben calibration_session_date kötelező

### DevelopmentAction
- **Határidő ellenőrzés**: planned_end_date >= planned_start_date
- **Lejárt akciók**: status != completed és actual_end_date < current_date → figyelmeztetés

### KnowledgeTransferPlan
- **Overlap period**: overlap_period_weeks >= 4 hét (ajánlott minimum)
- **Completion tracking**: completion_percentage 0-100

## 6. Integrációs pontok

### Külső rendszerek
- **Assessment tools**: SHL, Hogan, Korn Ferry Leadership Architect
- **9-box grid tools**: SuccessFactors, Workday Talent Management
- **Learning platforms**: leadership development programok (learning-entity.md integráció)
- **Performance management**: teljesítményértékelés importálása (performance-review-entity.md)

### Belső entitások
- **Employment**: utódjelöltek, jelenlegi pozíciók
- **Position**: kulcspozíciók, utódlásra tervezett pozíciók
- **Organization**: szervezeti szintű utódlástervezés
- **PerformanceReview**: teljesítmény értékelés → TalentAssessment
- **Learning**: fejlesztési akciók → TrainingCatalog, IndividualDevelopmentPlan
- **Compensation**: megtartási bónuszok, potenciál alapú bérezés
- **Recruitment**: külső vs belső jelölt döntés kulcspozíciókra

## 7. Kapcsolódó entitások

- **employment-entity.md**: Employment (utódjelöltek, incumbent, backup)
- **person-entity.md**: Person (mentorok, szponzorok, értékelők)
- **position-entity.md**: Position (kulcspozíciók, stretch assignments)
- **organization-entity.md**: OrgUnit (szervezeti szintű utódlástervezés)
- **performance-review-entity.md**: PerformanceReview (teljesítmény értékelés)
- **learning-entity.md**: IndividualDevelopmentPlan, TrainingCatalog (fejlesztési akciók)
- **compensation-entity.md**: Compensation (megtartási bónuszok, equity grants)

## 8. Üzleti folyamatok

### 8.1 Éves tehetségmenedzsment ciklus (Talent Review)

1. **Q1 - Teljesítményértékelés**: PerformanceReview lezárása
2. **Q2 - Tehetség értékelés**: TalentAssessment létrehozása (9-box grid)
3. **Q2 - Kalibrációs meeting**: vezetői kalibrációs ülések (calibration_session_date)
4. **Q3 - Utódlási tervek felülvizsgálata**: SuccessionPlan frissítése
5. **Q3 - Fejlesztési tervek**: DevelopmentAction-ök tervezése
6. **Q4 - Kompenzációs döntések**: megtartási bónuszok, equity grants
7. **Q4 - Következő év tervezése**: következő év talent pool-ok, programok

### 8.2 Kulcspozíció utódlástervezése

1. **Kulcspozíció azonosítás**: KeyPosition létrehozása
   - business_criticality, vacancy_risk, replacement_difficulty értékelése
2. **Kockázatfelmérés**: incumbent_flight_risk, incumbent_retirement_eligibility_date
3. **Utódlási terv készítése**: SuccessionPlan létrehozása
4. **Utódjelöltek azonosítása**: SuccessorCandidate entitások
   - readiness_level beállítása (ready now, 1-2 év, 3+ év)
   - successor_rank (1=elsődleges, 2=másodlagos, 3=harmadlagos)
5. **Fejlesztési akciók tervezése**: DevelopmentAction-ök
   - stretch assignments, job rotations, mentoring
6. **Tudásátadási terv**: KnowledgeTransferPlan (kritikus tudás dokumentálása)
7. **Felülvizsgálat**: negyedéves/féléves SuccessorCandidate readiness review

### 8.3 HiPo (High Potential) program

1. **Jelölés**: TalentAssessment alapján potential_rating=high
2. **Talent Pool létrehozása**: TalentPool (pool_type=high_potential)
3. **Beválogatás**: TalentPoolMembership létrehozása
   - nomination_source, selection_assessment
4. **Program indítás**: program_start_date, development_activities meghatározása
5. **Mentorálás**: mentor_assigned_id kijelölése (senior leader)
6. **Fejlesztési akciók**: stretch assignments, leadership training, international assignment
7. **Haladás követés**: progress_rating, mid_program_review_date
8. **Záró értékelés**: final_review_date, graduation_date
9. **Program utáni follow-up**: retention_12_months, promoted_after_program tracking

### 8.4 Vészhelyzeti utódlás (Emergency Succession)

1. **Váratlan távozás**: CEO/kulcsvezető azonnali távozása
2. **Emergency successor aktiválás**: SuccessionPlan.emergency_successor_id
3. **Ideiglenes kinevezés**: Employment új pozíció (interim CEO)
4. **Tudásátadás gyorsított**: KnowledgeTransferPlan express végrehajtás
5. **Kommunikáció**: stakeholder_introductions gyorsított
6. **Állandó utód keresése**:
   - Belső jelölt (SuccessorCandidate ready now) vs
   - Külső toborzás (recruitment-entity.md)

### 8.5 Tudásátadási folyamat

1. **Tudásátadási terv készítése**: KnowledgeTransferPlan
2. **Kritikus tudás azonosítása**: critical_knowledge_areas
3. **Dokumentálás**: knowledge_documentation_required=true
   - Folyamatok, kapcsolatok, stratégiai döntések dokumentálása
4. **Shadowing időszak**: transfer_method=shadowing, overlap_period_weeks
5. **Stakeholder bemutatás**: key_stakeholders bemutatkozás
6. **Tudástranszfer**: mentoring, job overlap, knowledge base
7. **Értékelés**: effectiveness_assessment, gaps_identified
8. **Lezárás**: status=completed

## 9. Reporting és analitika

### Utódlási kockázat jelentések
- **Nincs utód kulcspozíciók**: KeyPosition ahol current_successors_count = 0
- **Készenléti gap**: ready now utód hiánya kritikus pozícióknál
- **Várható nyugdíjazások**: incumbent_retirement_eligibility_date < 24 hónap
- **Távozási kockázat**: incumbent_flight_risk = high és nincs ready now utód

### Tehetség pipeline jelentések
- **9-box megoszlás**: muntatársak megoszlása a 9 kategóriában
- **HiPo arány**: high_potential / total employees %
- **Utódlási lefedettség**: kulcspozíciók hány %-ának van legalább 2 jelöltje
- **Tehetség pool sikeresség**: retention_12_months, promoted_after_program %

### Fejlesztési aktivitás
- **Fejlesztési akciók száma**: DevelopmentAction count utódjelöltenként
- **Fejlesztési költségek**: DevelopmentAction.cost összesítése
- **Akció completion rate**: completed / total DevelopmentAction %
- **Stretch assignment effectiveness**: effectiveness_rating megoszlás

### Diverzitás és inklúzió
- **Nemek aránya**: utódjelöltek gender megoszlása
- **Generációs megoszlás**: utódjelöltek életkor szerinti bontás
- **Külső vs belső mobilitás**: belső előléptetés vs külső toborzás arány

## 10. Nyitott kérdések és jövőbeli fejlesztések

### Technológiai kérdések
1. **9-box grid vizualizáció**: interaktív drag-and-drop 9-box canvas
2. **Org chart integráció**: vizuális utódlási lánc megjelenítés org chart-on
3. **Mobil alkalmazás**: vezetői dashboard mobilon (HiPo listák, jelöltek)
4. **AI/ML integráció**: prediktív flight risk modell, successor matching algoritmus

### Funkcionális fejlesztések
1. **Career pathing**: karriérútvonalak (Junior → Mid → Senior → Lead → Manager)
2. **Skill gap analysis**: pozíció kompetencia követelmények vs jelölt kompetenciák
3. **Scenario planning**: "mi lenne ha" szimulációk (pl. 3 kulcsember távozik)
4. **External talent benchmarking**: külső piac vs belső jelöltek összehasonlítása
5. **Succession planning for technical experts**: nem csak vezetői, hanem kritikus szakértői pozíciók

### Compliance és jogi
1. **Esélyegyenlőség monitoring**: GDPR-kompatibilis diverzitás tracking
2. **Data privacy**: ki láthatja a tehetség értékeléseket (employee aware, confidentiality)
3. **Non-discrimination**: objektív kiválasztási kritériumok biztosítása

### Integrációk
1. **Assessment center eredmények**: SHL, Hogan, Korn Ferry integráció
2. **LinkedIn**: külső jelöltek azonosítása, benchmarking
3. **Workday / SuccessFactors**: enterprise talent management platformok
4. **Survey tools**: employee engagement, stay interview eredmények

### AI és automatizálás
1. **Flight risk prediction**: ML alapú távozási kockázat előrejelzés
2. **Successor matching**: AI alapú jelölt ajánlás (kompetencia matching)
3. **Development recommendation engine**: személyre szabott fejlesztési javaslatok
4. **Sentiment analysis**: stay interview, engagement survey szöveganalízis

---

**Megjegyzések:**
- Az utódlástervezés **szigorúan bizalmas** – a munkatársak nem feltétlenül tudják, hogy jelöltek
- **"Candidate aware"** mező kritikus: etikai kérdés, hogy tájékoztatjuk-e a jelöltet
- **9-box grid**: évente egyszer, kalibrációs ülésekkel (elkerülni a szubjektivitást)
- **HiPo programok**: általában 12-24 hónapos, structured development activities
- **Backup appointment**: közszférában (Kit., Kttv.) kötelező helyettes kijelölés
- **Retention risk**: utódjelöltek megtartása kritikus – kompenzáció, development, visibility
- **Knowledge transfer**: kritikus, különösen founder/long-tenure vezetők távozásakor
- **Emergency succession**: vészhelyzeti terv minden C-level pozícióra
