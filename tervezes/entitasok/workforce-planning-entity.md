# WorkforcePlanning (Munkaerő-tervezés) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer
> **Verzió:** 0.1 – első tervezet
> **Utolsó frissítés:** 2025. február
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md, employment-entity.md, organization-entity.md, position-entity.md, compensation-entity.md, succession-planning-entity.md, recruitment-entity.md, offboarding-entity.md

---

## 1. Az entitás célja és hatóköre

A munkaerő-tervezési entitások biztosítják a szervezet stratégiai és operatív létszámtervezését, kapacitásmenedzsmentjét, költségvetés-tervezését, skill gap analízist és demográfiai előrejelzéseket a megfelelő létszám és kompetencia-mix biztosítása érdekében.

### Hatókör

A munkaerő-tervezési entitások lefedik:
- **Létszámtervezés (headcount planning)**: pozíció-alapú létszámterv éves/többéves horizonton
- **Demográfiai elemzés és előrejelzés**: nyugdíjazások, fluktuáció, öregedés előrejelzése
- **Kapacitástervezés**: munkaerő-kapacitás vs. üzleti igények (FTE, munkaórák)
- **Költségvetés-tervezés**: személyi jellegű költségek tervezése
- **Skill gap analízis**: kompetencia hiányok azonosítása, fejlesztési igények
- **Scenario planning**: "mi lenne ha" szimulációk (növekedés, költségcsökkentés, átszervezés)
- **Szervezeti átstrukturálás**: reorg tervezés, létszámleépítés, bővítés
- **Toborzási igények előrejelzése**: recruitment pipeline tervezés
- **Fluktuációs előrejelzés**: várható távozások becslése

### Foglalkoztatási típus specifikus követelmények

#### Közszféra (Kjt., Kttv., Kit., Púétv., Eszjtv.)
- **Költségvetési létszám**: éves költségvetési törvényben meghatározott létszámkeret
  - Kjt./Kttv./Kit. intézmények: engedélyezett létszám (fő)
  - Púétv. oktatási intézmények: tanulólétszám alapú normatív finanszírozás
  - Eszjtv. egészségügy: ágyszám/szakma alapú létszámnorma
- **Létszámkeret túllépés tilalma**: közszférában engedély nélkül nem léphető túl
- **Költségvetési év**: január 1 - december 31 (rigid tervezési ciklus)
- **MÁK (Magyar Államkincstár) jelentés**: havi létszám és bérköltség jelentés
- **Áthúzódó pozíciók**: költségvetési év közötti pozíció változások korlátozottak

#### Magánszektor (Mt.)
- **Nincs kötelező workforce planning**: üzleti döntés alapú
- **Rugalmas létszám**: dinamikus növekedés/csökkentés lehetséges
- **Kontingens munkavállalók**: contractors, temps rugalmas kezelése
- **Agile workforce planning**: negyedéves/havi tervezési ciklusok

#### Nagyvállalati best practice
- **3-5 éves stratégiai workforce plan**: hosszú távú létszám és kompetencia tervezés
- **Éves operational plan**: részletes éves létszámterv költségvetéssel
- **Negyedéves review**: forecast frissítés, headcount tracking
- **FTE (Full-Time Equivalent) alapú tervezés**: részmunkaidő, contractors egységes kezelése

## 2. Entitásstruktúra

### 2.1 WorkforcePlan (Munkaerő-terv)

Átfogó munkaerő-terv (stratégiai vagy operatív) szervezeti/divíziós szinten.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `plan_name` | String | igen | Terv neve (pl. "2026 Annual Workforce Plan", "2026-2028 Strategic Plan") | - |
| `plan_code` | String | igen | Terv kód (pl. "WFP-2026") | - |
| `plan_type` | Enum | igen | strategic/operational/budget/scenario | Terv típusa |
| `plan_scope` | Enum | igen | organization_wide/division/department/business_unit | Hatókör |
| `organization_unit_id` | UUID | nem | Szervezeti egység (ha nem organization_wide) | organization-entity.md |
| `planning_horizon` | Enum | igen | annual/quarterly/multi_year | Tervezési horizont |
| `plan_year_start` | Integer | igen | Kezdő év | - |
| `plan_year_end` | Integer | igen | Záró év | - |
| `fiscal_year_start_month` | Integer | igen | Költségvetési év kezdő hónapja (1-12) | - |
| `baseline_headcount` | Integer | nem | Kiinduló létszám (plan start dátum) | - |
| `baseline_fte` | Decimal | nem | Kiinduló FTE | - |
| `target_headcount` | Integer | nem | Célzott létszám (plan end dátum) | - |
| `target_fte` | Decimal | nem | Célzott FTE | - |
| `planned_hires` | Integer | nem | Tervezett felvételek száma | - |
| `planned_terminations` | Integer | nem | Tervezett kilépések/leépítések száma | - |
| `natural_attrition_rate` | Decimal | nem | Becsült természetes fluktuáció (%) | Historical data |
| `budget_allocated` | Decimal | nem | Kiosztott költségvetés (Ft) | - |
| `budget_spent` | Decimal | számított | Elköltött összeg | - |
| `status` | Enum | igen | draft/under_review/approved/active/closed/archived | - |
| `submitted_date` | Date | nem | Benyújtás dátuma | - |
| `submitted_by_id` | UUID | nem | Benyújtó | person-entity.md |
| `approved_date` | Date | nem | Jóváhagyás dátuma | - |
| `approved_by_id` | UUID | nem | Jóváhagyó (CFO, CEO) | person-entity.md |
| `review_cycle` | Enum | nem | monthly/quarterly/semi_annual/annual | Felülvizsgálati gyakoriság |
| `last_review_date` | Date | nem | Utolsó felülvizsgálat | - |
| `next_review_date` | Date | nem | Következő felülvizsgálat | - |
| `assumptions` | Text | nem | Tervezési feltételezések | - |
| `risks` | Text | nem | Kockázatok | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.2 HeadcountPlan (Létszámterv)

Részletes létszámterv pozíciónként/szervezeti egységenként.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `workforce_plan_id` | UUID | igen | Melyik munkaerő-tervhez | WorkforcePlan |
| `organization_unit_id` | UUID | igen | Szervezeti egység | organization-entity.md |
| `position_id` | UUID | nem | Konkrét pozíció (ha pozíció-specifikus terv) | position-entity.md |
| `job_family` | String | nem | Munkakör család (pl. "Engineering", "Sales") | - |
| `job_level` | Enum | nem | entry/mid/senior/lead/manager/director/executive | Szint |
| `employment_type` | Enum | nem | mt/kjt/kttv/kit/puetv/eszjtv/kut/contractor/temp | Jogviszony típus |
| `plan_period_start` | Date | igen | Tervezési időszak kezdete | - |
| `plan_period_end` | Date | igen | Tervezési időszak vége | - |
| `baseline_headcount` | Integer | igen | Kiinduló létszám (időszak elején) | - |
| `planned_new_hires` | Integer | nem | Tervezett új felvételek | - |
| `planned_internal_transfers_in` | Integer | nem | Beérkező belső áthelye zések | - |
| `planned_internal_transfers_out` | Integer | nem | Kimenő belső áthelyezések | - |
| `planned_promotions` | Integer | nem | Tervezett előléptetések (ha szint változás) | - |
| `planned_terminations` | Integer | nem | Tervezett kilépések (resignation, retirement) | - |
| `planned_layoffs` | Integer | nem | Tervezett leépítések (employer-initiated) | - |
| `target_headcount` | Integer | igen | Célzott létszám (időszak végén) | - |
| `actual_headcount` | Integer | számított | Tényleges létszám (real-time, Employment count) | - |
| `variance` | Integer | számított | Eltérés (actual - target) | - |
| `variance_percentage` | Decimal | számított | Eltérés % | - |
| `fte_baseline` | Decimal | nem | Kiinduló FTE (részmunkaidő figyelembevételével) | - |
| `fte_target` | Decimal | nem | Célzott FTE | - |
| `fte_actual` | Decimal | számított | Tényleges FTE | - |
| `status` | Enum | igen | draft/approved/active/completed | - |
| `is_budget_approved` | Boolean | igen | Költségvetés jóváhagyva-e | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.3 HeadcountForecast (Létszám-előrejelzés)

Dinamikus létszám-előrejelzés historikus trendek és predikciók alapján.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `workforce_plan_id` | UUID | nem | Kapcsolódó munkaerő-terv | WorkforcePlan |
| `organization_unit_id` | UUID | igen | Szervezeti egység | organization-entity.md |
| `forecast_date` | Date | igen | Előrejelzés dátuma (snapshot) | - |
| `forecast_horizon_months` | Integer | igen | Előrejelzési horizont (hónap, pl. 12, 24, 36) | - |
| `forecast_method` | Enum | igen | historical_trend/regression/machine_learning/manual | Módszer |
| `current_headcount` | Integer | igen | Jelenlegi létszám | - |
| `forecasted_headcount` | Integer | igen | Előrejelzett létszám (horizon végén) | - |
| `forecasted_hires` | Integer | nem | Előrejelzett felvételek száma | - |
| `forecasted_terminations` | Integer | nem | Előrejelzett kilépések száma | - |
| `forecasted_attrition_rate` | Decimal | nem | Előrejelzett fluktuációs ráta (%) | - |
| `forecasted_retirement_count` | Integer | nem | Előrejelzett nyugdíjazások | DemographicAnalysis |
| `confidence_level` | Enum | nem | high/medium/low | Előrejelzés megbízhatósága |
| `confidence_interval_lower` | Integer | nem | Alsó konfidencia intervallum | - |
| `confidence_interval_upper` | Integer | nem | Felső konfidencia intervallum | - |
| `assumptions` | Text | nem | Előrejelzési feltételezések | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |

### 2.4 SkillRequirement (Kompetencia követelmény)

Szervezeti/pozíció szintű kompetencia és skill követelmények.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `workforce_plan_id` | UUID | nem | Kapcsolódó munkaerő-terv | WorkforcePlan |
| `organization_unit_id` | UUID | nem | Szervezeti egység | organization-entity.md |
| `position_id` | UUID | nem | Pozíció (ha pozíció-specifikus) | position-entity.md |
| `skill_category` | Enum | igen | technical/functional/leadership/soft_skills/language/certification | Kategória |
| `skill_name` | String | igen | Kompetencia/skill neve (pl. "Python", "Project Management", "Vezetői kompetencia") | - |
| `skill_description` | Text | nem | Részletes leírás | - |
| `proficiency_level_required` | Enum | igen | beginner/intermediate/advanced/expert | Szükséges szint |
| `priority` | Enum | igen | critical/high/medium/low | Prioritás |
| `required_headcount` | Integer | nem | Hány főnél szükséges ez a skill | - |
| `current_headcount_with_skill` | Integer | számított | Jelenleg hány fő rendelkezik (SkillGap alapján) | - |
| `gap_headcount` | Integer | számított | Hiányzó létszám (required - current) | - |
| `target_date` | Date | nem | Célzott teljesítési dátum | - |
| `source_strategy` | Enum | nem | hire/train/external_contractor/outsource | Beszerzési stratégia |
| `estimated_training_cost_per_person` | Decimal | nem | Becsült képzési költség/fő (Ft) | - |
| `estimated_hire_cost_per_person` | Decimal | nem | Becsült toborzási költség/fő (Ft) | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.5 SkillGap (Skill gap analízis)

Munkatársak skill hiányainak azonosítása.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `skill_requirement_id` | UUID | igen | Melyik skill követelményhez | SkillRequirement |
| `employment_id` | UUID | nem | Munkatárs (ha egyéni gap) | employment-entity.md |
| `organization_unit_id` | UUID | nem | Szervezeti egység (ha team gap) | organization-entity.md |
| `gap_type` | Enum | igen | individual/team/organizational | Gap szintje |
| `current_proficiency_level` | Enum | nem | none/beginner/intermediate/advanced/expert | Jelenlegi szint |
| `required_proficiency_level` | Enum | igen | beginner/intermediate/advanced/expert | Szükséges szint |
| `proficiency_gap` | Integer | számított | Gap mértéke (numerikusan, 1-4) | - |
| `gap_severity` | Enum | igen | critical/high/medium/low | Súlyosság |
| `impact_on_business` | Text | nem | Üzleti hatás leírása | - |
| `remediation_plan` | Enum | nem | training/mentoring/hire/contractor/no_action | Megoldási terv |
| `training_plan_id` | UUID | nem | Kapcsolódó képzési terv | learning-entity.md IndividualDevelopmentPlan |
| `target_close_date` | Date | nem | Célzott gap lezárási dátum | - |
| `actual_close_date` | Date | nem | Tényleges lezárás dátuma | - |
| `status` | Enum | igen | open/in_progress/closed/no_action | Státusz |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.6 CapacityPlan (Kapacitásterv)

Munkaerő-kapacitás tervezése FTE és munkaórák alapján.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `workforce_plan_id` | UUID | igen | Melyik munkaerő-tervhez | WorkforcePlan |
| `organization_unit_id` | UUID | igen | Szervezeti egység | organization-entity.md |
| `plan_period_start` | Date | igen | Tervezési időszak kezdete | - |
| `plan_period_end` | Date | igen | Tervezési időszak vége | - |
| `total_available_fte` | Decimal | igen | Rendelkezésre álló FTE (headcount × work hours) | - |
| `total_available_hours` | Decimal | számított | Rendelkezésre álló munkaórák | - |
| `planned_productive_hours` | Decimal | igen | Tervezett produktív órák (projects, operations) | - |
| `planned_non_productive_hours` | Decimal | nem | Nem produktív órák (training, meetings, admin) | - |
| `planned_absence_hours` | Decimal | nem | Tervezett távollét (szabadság, betegség) | - |
| `capacity_utilization_target` | Decimal | nem | Célzott kapacitás kihasználtság (%) | - |
| `capacity_utilization_actual` | Decimal | számított | Tényleges kihasználtság (%) | - |
| `capacity_surplus_deficit` | Decimal | számított | Kapacitás többlet/hiány (órák) | - |
| `demand_forecast_hours` | Decimal | nem | Előrejelzett kereslet (órák, projektek alapján) | - |
| `demand_source` | String | nem | Kereslet forrása (pl. "Sales pipeline", "Project backlog") | - |
| `hiring_need_fte` | Decimal | számított | Felvételi igény (deficit alapján) | - |
| `status` | Enum | igen | draft/approved/active/completed | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.7 WorkforceBudget (Munkaerő-költségvetés)

Személyi jellegű költségek tervezése.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `workforce_plan_id` | UUID | igen | Melyik munkaerő-tervhez | WorkforcePlan |
| `organization_unit_id` | UUID | igen | Szervezeti egység | organization-entity.md |
| `budget_year` | Integer | igen | Költségvetési év | - |
| `budget_period_start` | Date | igen | Költségvetési időszak kezdete | - |
| `budget_period_end` | Date | igen | Költségvetési időszak vége | - |
| `budget_category` | Enum | igen | salary/bonus/benefits/payroll_taxes/training/recruitment/contingent_labor/other | Kategória |
| `budgeted_amount` | Decimal | igen | Tervezett költség (Ft) | - |
| `actual_amount` | Decimal | számított | Tényleges költség (real-time, payroll alapján) | - |
| `variance_amount` | Decimal | számított | Eltérés összege (actual - budgeted) | - |
| `variance_percentage` | Decimal | számított | Eltérés % | - |
| `forecast_amount` | Decimal | nem | Előrejelzett éves költség (év közben) | - |
| `headcount_assumption` | Integer | nem | Létszám feltételezés (budget alapja) | - |
| `average_salary_assumption` | Decimal | nem | Átlagbér feltételezés (Ft) | - |
| `salary_increase_rate` | Decimal | nem | Béremelési ráta (%) | - |
| `benefit_cost_per_employee` | Decimal | nem | Juttatás költsége/fő (Ft) | - |
| `payroll_tax_rate` | Decimal | nem | Bérjárulék ráta (%) | Tbj. |
| `status` | Enum | igen | draft/approved/active/closed | - |
| `approved_by_id` | UUID | nem | Jóváhagyó (CFO) | person-entity.md |
| `approved_date` | Date | nem | Jóváhagyás dátuma | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.8 ScenarioPlan (Scenario tervezés)

"Mi lenne ha" szimulációk (növekedés, költségcsökkentés, átszervezés).

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `workforce_plan_id` | UUID | nem | Kapcsolódó munkaerő-terv (base plan) | WorkforcePlan |
| `scenario_name` | String | igen | Szcenárió neve (pl. "Growth 20%", "Cost Reduction 15%", "Merger Scenario") | - |
| `scenario_type` | Enum | igen | growth/cost_reduction/merger_acquisition/restructuring/market_downturn/best_case/worst_case | Típus |
| `scenario_description` | Text | igen | Részletes leírás | - |
| `planning_horizon_months` | Integer | igen | Tervezési horizont (hónap) | - |
| `baseline_headcount` | Integer | igen | Kiinduló létszám | - |
| `scenario_headcount_change` | Integer | nem | Létszám változás (+/-) | - |
| `scenario_target_headcount` | Integer | számított | Célzott létszám scenario-ban | - |
| `scenario_budget_change_percentage` | Decimal | nem | Költségvetés változás (%) | - |
| `scenario_budget_change_amount` | Decimal | nem | Költségvetés változás (Ft) | - |
| `assumptions` | Text | igen | Scenario feltételezések | - |
| `key_actions` | JSON | nem | Kulcs akciók (hiring freeze, layoffs, salary freeze) | - |
| `expected_benefits` | Text | nem | Várható előnyök | - |
| `expected_risks` | Text | nem | Várható kockázatok | - |
| `probability` | Enum | nem | very_likely/likely/possible/unlikely | Bekövetkezési valószínűség |
| `impact_assessment` | Text | nem | Hatáselemzés | - |
| `status` | Enum | igen | draft/under_review/approved/implemented/archived | - |
| `reviewed_by_id` | UUID | nem | Felülvizsgáló | person-entity.md |
| `reviewed_date` | Date | nem | Felülvizsgálat dátuma | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.9 DemographicAnalysis (Demográfiai elemzés)

Munkaerő demográfiai elemzése és előrejelzések.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `analysis_date` | Date | igen | Elemzés dátuma (snapshot) | - |
| `organization_unit_id` | UUID | nem | Szervezeti egység (ha nem organization_wide) | organization-entity.md |
| `analysis_type` | Enum | igen | age_distribution/tenure_distribution/retirement_forecast/gender_diversity/generation_mix | Elemzés típusa |
| `total_headcount` | Integer | igen | Összes létszám (elemzés alapja) | - |
| `age_distribution` | JSON | nem | Életkor megoszlás (age_bracket: headcount) | GDPR-compliant aggregált |
| `average_age` | Decimal | nem | Átlagéletkor | - |
| `median_age` | Decimal | nem | Medián életkor | - |
| `retirement_eligible_count` | Integer | nem | Nyugdíjkorhatárt elérők száma | - |
| `retirement_eligible_percentage` | Decimal | nem | Nyugdíjkorhatárt elérők aránya (%) | - |
| `projected_retirements_12_months` | Integer | nem | Várható nyugdíjazások 12 hónapon belül | - |
| `projected_retirements_24_months` | Integer | nem | Várható nyugdíjazások 24 hónapon belül | - |
| `projected_retirements_60_months` | Integer | nem | Várható nyugdíjazások 60 hónapon belül | - |
| `tenure_distribution` | JSON | nem | Szolgálati idő megoszlás (tenure_bracket: headcount) | - |
| `average_tenure_years` | Decimal | nem | Átlagos szolgálati idő (év) | - |
| `gender_distribution` | JSON | nem | Nemek megoszlása (aggregált, GDPR) | - |
| `gender_pay_gap_percentage` | Decimal | nem | Bérkülönbség nemek között (%) | Transzparencia |
| `generation_distribution` | JSON | nem | Generációs megoszlás (Gen Z, Millennial, Gen X, Boomer) | - |
| `diversity_metrics` | JSON | nem | Diverzitási mutatók | - |
| `succession_risk_score` | Decimal | nem | Utódlási kockázati pontszám (0-100) | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |

### 2.10 RestructuringPlan (Átszervezési terv)

Szervezeti átstrukturálás, reorg, létszámleépítés tervezése.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `plan_name` | String | igen | Átszervezési terv neve (pl. "2026 Divisional Restructuring") | - |
| `restructuring_type` | Enum | igen | organizational_redesign/cost_reduction/merger_integration/divestiture/expansion | Típus |
| `affected_organization_units` | JSON | igen | Érintett szervezeti egységek (org_unit_id lista) | organization-entity.md |
| `plan_start_date` | Date | igen | Terv indítása | - |
| `plan_completion_date` | Date | igen | Tervezett befejezés | - |
| `current_headcount` | Integer | igen | Jelenlegi létszám (affected units) | - |
| `target_headcount` | Integer | igen | Célzott létszám | - |
| `headcount_reduction` | Integer | számított | Létszámcsökkentés (current - target) | - |
| `positions_eliminated` | Integer | nem | Megszüntetett pozíciók száma | - |
| `positions_created` | Integer | nem | Új pozíciók száma | - |
| `positions_modified` | Integer | nem | Módosított pozíciók száma | - |
| `affected_employees_count` | Integer | nem | Érintett munkatársak száma | - |
| `layoff_count` | Integer | nem | Elbocsátások száma | - |
| `voluntary_severance_count` | Integer | nem | Önkéntes távozási csomag (VSP) | - |
| `internal_redeployment_count` | Integer | nem | Belső áthelyezések száma | - |
| `estimated_severance_cost` | Decimal | nem | Becsült végkielégítési költség (Ft) | Mt. 77. §, Kjt. 37. § |
| `estimated_restructuring_cost` | Decimal | nem | Becsült teljes átszervezési költség (Ft) | - |
| `expected_annual_savings` | Decimal | nem | Várható éves megtakarítás (Ft) | - |
| `payback_period_months` | Integer | számított | Megtérülési idő (hónap) | - |
| `communication_plan` | Text | nem | Kommunikációs terv | - |
| `employee_support_programs` | Text | nem | Munkatársak támogatása (outplacement, retraining) | - |
| `legal_compliance_notes` | Text | nem | Jogi megfelelés (Mt., Kjt. collective dismissal rules) | - |
| `union_consultation_required` | Boolean | nem | Szakszervezeti egyeztetés szükséges-e | Mt. 63. § (tömeges leépítés) |
| `union_consultation_completed` | Boolean | nem | Egyeztetés megtörtént-e | - |
| `status` | Enum | igen | draft/under_review/approved/in_implementation/completed/cancelled | - |
| `approved_by_id` | UUID | nem | Jóváhagyó (CEO, Board) | person-entity.md |
| `approved_date` | Date | nem | Jóváhagyás dátuma | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

## 3. GDPR kategorizáció és adatkezelés

### Workforce planning adatok kezelése

#### Jogalap
- **GDPR Art. 6(1)(f)**: jogos érdek (üzleti tervezés, munkaerő optimalizálás)
- **GDPR Art. 6(1)(b)**: szerződés teljesítése (foglalkoztatási tervezés)
- **GDPR Art. 88**: munkavállalói adatok kezelése foglalkoztatás keretében

#### Adatkategóriák
- **Aggregált, nem személyes adat**: headcount, FTE, költségvetés (szervezeti szinten)
- **Általános személyes adat**: egyéni skill gap, training plan
- **Demográfiai adat (különös védelem)**: életkor, nem (CSAK aggregált formában, GDPR Art. 9)
- **Nem különleges adat**: workforce planning döntően aggregált, nem személyes

#### Adatmegőrzési idők
- **WorkforcePlan**: terv lezárása + 5 év (historikus trendek, audit)
- **HeadcountPlan**: terv lezárása + 5 év
- **SkillGap (egyéni)**: gap lezárása + 3 év
- **DemographicAnalysis**: elemzés dátuma + 5 év (aggregált, név nélküli)
- **RestructuringPlan**: végrehajtás + 8 év (jogi, pénzügyi audit)

#### GDPR megfelelés
- **Anonymizáció**: demográfiai adatok (életkor, nem) CSAK aggregált, név nélküli formában
- **Minimalizáció**: csak szükséges adatok (életkor helyett "retirement eligible" flag)
- **Hozzáférés korlátozás**: workforce planning adatok leadership/HR/Finance szintű hozzáférés

### Hozzáférési jogok
- **Munkatárs**: saját skill gap, training plan
- **Manager**: csapat headcount plan, skill gap (aggregált vagy egyéni ha fejlesztési terv)
- **HR**: teljes hozzáférés (workforce planning, skill gap, demográfiai elemzés)
- **Finance/CFO**: workforce budget, headcount plan, restructuring cost
- **C-level**: teljes hozzáférés (stratégiai döntések)

## 4. Hozzáférési szintek

| Entitás | HR Admin | C-Level | Finance | Manager | Employee |
|---|---|---|---|---|---|
| WorkforcePlan | CRUD | CRUD | Read | Read (department) | - |
| HeadcountPlan | CRUD | CRUD | Read | Read (team) | - |
| HeadcountForecast | CRUD | Read | Read | Read (team) | - |
| SkillRequirement | CRUD | Read | - | Read (team) | - |
| SkillGap | CRUD | Read | - | CRUD (team) | Read (saját) |
| CapacityPlan | CRUD | Read | Read | Read (team) | - |
| WorkforceBudget | CRUD | CRUD | CRUD | Read (department budget) | - |
| ScenarioPlan | CRUD | CRUD | Read | - | - |
| DemographicAnalysis | CRUD | Read | - | Read (aggregált) | - |
| RestructuringPlan | CRUD | CRUD | Read | Read (ha érintett) | - |

## 5. Validációs szabályok

### WorkforcePlan
- **Plan years consistency**: plan_year_end >= plan_year_start
- **Headcount balance**: target_headcount = baseline_headcount + planned_hires - planned_terminations
- **Approval workflow**: status=approved csak approved_by_id és approved_date kitöltése után

### HeadcountPlan
- **Headcount calculation**: target_headcount = baseline + new_hires + transfers_in - transfers_out - terminations - layoffs
- **Variance alerting**: variance_percentage > 10% → figyelmeztetés
- **FTE consistency**: fte_target <= target_headcount (FTE nem lehet több mint headcount)

### HeadcountForecast
- **Forecast horizon validation**: forecast_horizon_months <= 60 (max 5 év)
- **Confidence interval**: confidence_interval_lower <= forecasted_headcount <= confidence_interval_upper

### SkillGap
- **Gap severity calculation**: proficiency_gap >= 2 → severity=critical/high
- **Closure tracking**: status=closed csak actual_close_date kitöltése után

### CapacityPlan
- **Capacity utilization**: capacity_utilization = planned_productive_hours / total_available_hours × 100
- **Surplus/deficit**: capacity_surplus_deficit = total_available_hours - demand_forecast_hours
- **Hiring need**: hiring_need_fte = capacity_surplus_deficit (ha negatív) / standard_work_hours_per_fte

### WorkforceBudget
- **Variance tracking**: variance_percentage = (actual_amount - budgeted_amount) / budgeted_amount × 100
- **Overspend alerting**: variance_percentage > 10% → figyelmeztetés

### RestructuringPlan
- **Headcount reduction**: headcount_reduction = current_headcount - target_headcount
- **Payback period**: payback_period_months = estimated_restructuring_cost / (expected_annual_savings / 12)
- **Collective dismissal**: layoff_count >= 10 (vagy 10% workforce) → union_consultation_required=true (Mt. 63. §)

## 6. Integrációs pontok

### Külső rendszerek
- **ERP / Financial Planning (SAP, Oracle)**: költségvetési integráció, pénzügyi tervezés
- **HRIS / Talent Management (Workday, SuccessFactors)**: headcount adatok, skill inventory
- **BI / Analytics tools (Tableau, Power BI)**: workforce analytics dashboards
- **Payroll system**: actual salary cost, headcount data
- **Project Management (Jira, MS Project)**: project demand forecast, capacity planning

### Belső entitások
- **Employment**: actual headcount, FTE calculation (real-time)
- **Position**: open positions, planned positions
- **Organization**: org structure, headcount distribution
- **Compensation**: salary budget, average salary
- **Recruitment**: hiring pipeline, time-to-fill forecast
- **Offboarding**: termination forecast, attrition rate
- **SuccessionPlanning**: retirement forecast, successor readiness
- **Learning**: training budget, skill development plans
- **PerformanceReview**: skill assessments, competency ratings

## 7. Kapcsolódó entitások

- **employment-entity.md**: Employment (actual headcount, FTE)
- **person-entity.md**: Person (approvers, planners)
- **organization-entity.md**: OrgUnit (organizational planning)
- **position-entity.md**: Position (position-based planning)
- **compensation-entity.md**: Compensation (salary budget, cost planning)
- **recruitment-entity.md**: JobRequisition (hiring plan execution)
- **offboarding-entity.md**: OffboardingCase (attrition data)
- **succession-planning-entity.md**: SuccessionPlan (retirement forecast)
- **learning-entity.md**: IndividualDevelopmentPlan (skill development)
- **performance-review-entity.md**: PerformanceReview (skill ratings)

## 8. Üzleti folyamatok

### 8.1 Éves workforce planning ciklus

1. **Q3-Q4 (előző év)**: stratégiai tervezés indítása
   - **Üzleti stratégia review**: C-level meghatározza growth/cost targets
   - **WorkforcePlan létrehozása**: HR workforce plan draft (plan_year=következő év)
2. **Q4**: részletes tervezés
   - **HeadcountPlan készítés**: szervezeti egységenként, pozíciónként
   - **SkillRequirement**: kritikus skill-ek azonosítása
   - **CapacityPlan**: kapacitás vs. demand elemzés
   - **WorkforceBudget**: költségvetés tervezés (salary, benefits, training, recruitment)
3. **Q4**: jóváhagyási folyamat
   - **Manager review**: department managers review és feedback
   - **Finance review**: CFO budget review
   - **C-level approval**: CEO/Board jóváhagyás
   - WorkforcePlan status=approved
4. **Q1 (tárgy év)**: végrehajtás indítása
   - **Recruitment activation**: JobRequisition létrehozása hiring plan alapján
   - **Training activation**: képzési programok indítása (SkillGap alapján)
5. **Q1-Q4**: monitoring és adjustment
   - **Negyedéves review**: HeadcountPlan actual vs. target tracking
   - **Forecast frissítés**: HeadcountForecast update (attrition, market changes)
   - **Budget tracking**: WorkforceBudget variance analysis

### 8.2 Skill gap analízis és fejlesztési terv

1. **Skill követelmények meghatározása**: SkillRequirement létrehozása
   - Pozíció-alapú: position_id kapcsolat
   - Stratégiai: organization-wide skill needs
2. **Skill inventory**: munkatársak skill-jeinek felmérése
   - PerformanceReview alapján
   - Self-assessment
   - Manager assessment
3. **SkillGap analízis**: gap azonosítás
   - gap_type=individual (egyéni)
   - gap_type=team (csapat szintű)
   - gap_type=organizational (szervezeti)
4. **Remediation tervezés**: remediation_plan kiválasztása
   - training: IndividualDevelopmentPlan létrehozása (learning-entity.md)
   - hire: JobRequisition létrehozása (recruitment-entity.md)
   - contractor: külső resource bevonása
5. **Végrehajtás tracking**: status=in_progress → closed

### 8.3 Demográfiai elemzés és nyugdíjazási terv

1. **Demográfiai snapshot**: DemographicAnalysis létrehozása (évente)
   - age_distribution, tenure_distribution
   - retirement_eligible_count
2. **Nyugdíjazási előrejelzés**:
   - projected_retirements_12_months, _24_months, _60_months
3. **Succession planning aktiválás**:
   - retirement_eligible pozíciók → SuccessionPlan (succession-planning-entity.md)
   - Utódjelöltek azonosítása
4. **Replacement planning**:
   - HeadcountPlan adjustment (planned_terminations=retirement count)
   - JobRequisition tervezése (ha nincs belső utód)

### 8.4 Scenario planning (növekedés/költségcsökkentés)

1. **Base plan**: WorkforcePlan (approved, status=active)
2. **Scenario létrehozása**: ScenarioPlan
   - scenario_type=growth: headcount_change=+20%
   - scenario_type=cost_reduction: headcount_change=-15%
3. **Scenario modellezés**:
   - Headcount impact: scenario_target_headcount
   - Budget impact: scenario_budget_change_amount
   - Key actions: hiring freeze, layoffs, salary freeze
4. **What-if analysis**: assumptions, risks, benefits
5. **Decision**: C-level döntés scenario implementálásról
6. **Implementation**: ScenarioPlan status=implemented
   - Base WorkforcePlan frissítése
   - HeadcountPlan adjustment
   - RestructuringPlan (ha layoffs)

### 8.5 Átszervezés (restructuring)

1. **Restructuring decision**: C-level döntés átszervezésről
2. **RestructuringPlan létrehozása**:
   - restructuring_type=cost_reduction/organizational_redesign
   - affected_organization_units, positions_eliminated, layoff_count
3. **Impact analysis**:
   - Affected employees
   - Severance cost: estimated_severance_cost
   - Expected savings: expected_annual_savings
4. **Legal compliance**:
   - Collective dismissal: union_consultation_required (Mt. 63. §)
   - Szakszervezeti egyeztetés
5. **Communication plan**: employee communication strategy
6. **Implementation**:
   - OffboardingCase létrehozása (affected employees)
   - Position updates (eliminated, modified)
   - OrgUnit updates (organizational redesign)
7. **Completion**: RestructuringPlan status=completed

## 9. Reporting és analitika

### Workforce metrics
- **Headcount trend**: havi/negyedéves headcount változás (actual vs. plan)
- **FTE trend**: FTE változás (part-time, contractors figyelembevételével)
- **Turnover forecast**: attrition rate előrejelzés
- **Retirement forecast**: 12/24/60 hónapos nyugdíjazási előrejelzés
- **Span of control**: manager / direct reports arány

### Budget és költségek
- **Salary cost trend**: actual vs. budgeted salary cost
- **Cost per employee**: total workforce cost / headcount
- **Budget variance**: WorkforceBudget variance_percentage kategóriánként
- **Recruitment cost**: hiring cost vs. budget
- **Training cost**: training budget utilization

### Skill és kompetencia
- **Skill coverage**: required skills teljesítettség (%)
- **Skill gap count**: open skill gaps severity szerint
- **Skill gap closure rate**: closed gaps / total gaps %
- **Critical skill shortage**: critical severity skill gaps lista

### Capacity és produktivitás
- **Capacity utilization**: capacity_utilization_actual trend
- **Capacity surplus/deficit**: headcount vs. demand gap
- **Hiring need forecast**: hiring_need_fte előrejelzés

### Demográfia és diverzitás
- **Age distribution**: munkavállalói életkor megoszlás (aggregált)
- **Tenure distribution**: szolgálati idő megoszlás
- **Gender diversity**: nemek aránya (leadership szinteken)
- **Generation mix**: generációs megoszlás (Gen Z, Millennial, Gen X, Boomer)

## 10. Nyitott kérdések és jövőbeli fejlesztések

### Technológiai kérdések
1. **Predictive analytics**: ML-alapú headcount forecast (attrition prediction, hiring need)
2. **Real-time dashboards**: workforce analytics real-time (Power BI, Tableau integráció)
3. **What-if scenario tools**: interaktív scenario planning calculator
4. **API integrációk**: HRIS, ERP, Payroll adatok automatikus szinkronizálása

### Funkcionális fejlesztések
1. **Contractor/gig worker planning**: kontingens munkaerő tervezése (FTE konverzió)
2. **Skills taxonomy**: standardizált skill library (O*NET, LinkedIn Skills)
3. **Competency framework integráció**: skill requirement ↔ pozíció competency mapping
4. **Project-based capacity planning**: projektek kapacitásigényének integrálása
5. **Workforce segmentation**: különböző workforce szegmensek (HiPo, core, low performer) kezelése

### Compliance és jogi
1. **Collective bargaining**: szakszervezeti egyeztetés restructuring esetén (Mt. 63. §)
2. **GDPR demográfiai adatok**: anonymizált demográfiai riportok (életkor, nem)
3. **Pay equity analysis**: bérkülönbség nemek/etnikumok között (transparency directive)

### Integrációk
1. **ERP / SAP**: pénzügyi tervezés, költségvetés integráció
2. **Talent Management (Workday, SuccessFactors)**: skill inventory, performance data
3. **Project Management (Jira, MS Project)**: project demand, capacity allocation
4. **Survey tools**: employee sentiment → attrition forecast

### AI és automatizálás
1. **Attrition prediction**: ML modell turnover forecast (flight risk alapján)
2. **Skill gap recommendation engine**: AI javaslatok skill development stratégiára
3. **Workforce optimization**: AI-alapú headcount optimalizálás (cost vs. capacity)
4. **Scenario simulation**: Monte Carlo szimuláció workforce planning-hez

---

**Megjegyzések:**
- **Workforce planning kritikus**: stratégiai üzleti tervezés alapja
- **Headcount vs. FTE**: FTE figyelembe veszi részmunkaidőt, contractors-t
- **Skill gap analysis**: kompetencia alapú workforce planning (nem csak headcount)
- **Scenario planning**: "mi lenne ha" szimulációk (growth, cost reduction, M&A)
- **Demográfiai elemzés**: nyugdíjazás, fluktuáció, generációs váltás előrejelzése
- **Budget integration**: workforce cost = legnagyobb költség (60-80% operating cost)
- **GDPR demográfiai adatok**: CSAK aggregált, név nélküli (életkor, nem)
- **Közszféra**: költségvetési létszám rigid, magánszektor: agile planning
- **Real-time tracking**: actual headcount vs. plan continuous monitoring
- **Succession planning kapcsolat**: retirement forecast → succession planning
