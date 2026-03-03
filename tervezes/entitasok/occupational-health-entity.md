# OccupationalHealth (Munkavédelem és Foglalkozás-egészségügy) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer
> **Verzió:** 0.1 – első tervezet
> **Utolsó frissítés:** 2025. február
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md, employment-entity.md, organization-entity.md, position-entity.md, learning-entity.md, document-entity.md, timetracking-entity.md

---

## 1. Az entitás célja és hatóköre

A munkavédelmi és foglalkozás-egészségügyi entitások biztosítják a munkavédelmi jogszabályi kötelezettségek teljesítését, a munkahelyi egészség és biztonság fenntartását, valamint a foglalkozás-egészségügyi alkalmassági vizsgálatok nyilvántartását.

### Hatókör

A munkavédelmi entitások lefedik:
- **Foglalkozás-egészségügyi vizsgálatok**: belépési, időszakos, rendkívüli és soron kívüli vizsgálatok
- **Alkalmassági minősítések**: alkalmas, korlátozottan alkalmas, alkalmatlan
- **Munkahelyi kockázatértékelés**: munkakörökh

öz kapcsolódó egészségügyi kockázatok
- **Vizsgálati ütemezés**: kötelező vizsgálatok automatikus ütemezése, emlékeztetők
- **Foglalkozási megbetegedések**: munkaviszonnyal összefüggő megbetegedések nyilvántartása
- **Munkabalesetek**: munkahelyi balesetek dokumentálása, kivizsgálása
- **Munkavédelmi oktatások**: munkavédelmi, tűzvédelmi, elsősegély oktatási jegyzőkönyvek
- **Védőeszköz kezelés**: egyéni védőeszközök kiadása, nyilvántartása

### Foglalkoztatási típus specifikus követelmények

#### Minden jogviszony (Mt., Kjt., Kttv., Kit., Púétv., Eszjtv., Küt.)
- **Foglalkozás-egészségügyi vizsgálat**: 1993. évi XCIII. törvény a munkavédelemről (Mvt.)
  - Munkakezdés előtt kötelező (Mvt. 54. §)
  - Időszakos vizsgálat munkakör-specifikus gyakorisággal
  - Munkakör változáskor
  - Munkavállaló kérésére (egészségügyi probléma esetén)
  - 89/1995. (VII. 14.) Korm. rendelet a foglalkozás-egészségügyi szolgálatról
  - 20/2025. (VI. 26.) BM rendelet (mellkas röntgen szabályozás)
- **Munkavédelmi oktatás**: Mvt. 54. §
  - Belépéskor kötelező (általános és munkakör-specifikus)
  - Évente ismétlő oktatás
  - Munkaköri változáskor
  - Jegyzőkönyv vezetése kötelező
- **Munkabaleset jelentés**: 1993. évi XCIII. törvény 61-62. §
  - Bejelentési kötelezettség 8 órán belül (súlyos/halálos baleset)
  - Kivizsgálás dokumentálása
  - Munkabiztonsági Felügyelet értesítése

#### Veszélyes munkakörök (Mvt. 54. §, 25/2000. (IX. 30.) EüM-SzCsM rendelet)
- **Fokozott gyakorisággal ismétlődő vizsgálatok**:
  - Ionizáló sugárzás: 6 hónaponként
  - Vegyi anyagoknak kitett: 12 hónaponként
  - Éjszakai munkavégzés: évente kötelező
  - Zajos munkakörnyezet: évente
- **Különleges vizsgálatok**: audiometria, spirometria, toxikológiai vizsgálatok

#### Egészségügy (Eszjtv.)
- **Egészségügyi dolgozók védőoltásai**: Hepatitis B, influenza, COVID-19 ajánlott
- **Vérrel terjedő kórokozók elleni védelem**: foglalkozási expozíció dokumentálása
- **Radiológiai munkakörök**: sugáregészségügyi nyilvántartás

#### Köznevelés (Púétv.)
- **Pedagógusok alkalmassági vizsgálata**: pályaalkalmassági és munkaköri alkalmassági vizsgálat
- **Közoktatási törvény szerinti egészségügyi alkalmassági vizsgálat**

#### Fegyveres testületek, rendvédelem (Hszt., Hjt., Rtv.)
- **Pályaalkalmassági vizsgálat**: fizikai, pszichológiai és orvosi vizsgálatok
- **Rendszeres egészségügyi felülvizsgálat**: 1-3 évenként szolgálati ágonként
- **1/2025. (I. 13.) HM rendelet**: munkára való alkalmassági vizsgálat

## 2. Entitásstruktúra

### 2.1 OccupationalHealthExamType (Foglalkozás-egészségügyi vizsgálattípus törzsadat)

Vizsgálattípusok katalógusa (belépési, időszakos, rendkívüli).

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `exam_code` | String | igen | Vizsgálat kód (pl. "ENTRY-GENERAL", "PERIODIC-OFFICE") | - |
| `exam_name` | String | igen | Vizsgálat neve | - |
| `exam_category` | Enum | igen | pre_employment/periodic/extraordinary/on_demand/return_to_work | Mvt. 54. § |
| `legal_basis` | String | nem | Jogszabályi hivatkozás | Mvt. 54. §, 89/1995. Korm. r. |
| `description` | Text | nem | Leírás | - |
| `required_tests` | JSON | nem | Kötelező vizsgálati elemek (vizelet, vér, EKG, audiometria, spirometria, rtg, stb.) | - |
| `estimated_duration_minutes` | Integer | nem | Becsült időtartam (perc) | - |
| `validity_months` | Integer | nem | Érvényesség hónapokban (időszakos vizsgálatoknál) | - |
| `applicable_risk_factors` | JSON | nem | Alkalmazandó kockázati tényezők (zaj, vegyi anyag, ionizáló sugárzás) | 25/2000. EüM-SzCsM r. |
| `is_active` | Boolean | igen | Aktív-e | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.2 OccupationalHealthExam (Foglalkozás-egészségügyi vizsgálat)

Konkrét vizsgálati események.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `employment_id` | UUID | igen | Melyik munkatárs | employment-entity.md |
| `exam_type_id` | UUID | igen | Vizsgálattípus | OccupationalHealthExamType |
| `exam_category` | Enum | igen | pre_employment/periodic/extraordinary/on_demand/return_to_work/fitness_review | Mvt. 54. § |
| `exam_reason` | Enum | nem | new_hire/job_change/periodic_schedule/health_concern/accident_followup/regulatory_requirement | - |
| `scheduled_date` | Date | igen | Tervezett vizsgálat dátuma | - |
| `actual_exam_date` | Date | nem | Tényleges vizsgálat dátuma | - |
| `exam_location` | String | nem | Vizsgálat helyszíne | - |
| `occupational_physician_name` | String | nem | Foglalkozás-egészségügyi orvos neve | - |
| `occupational_physician_license` | String | nem | Orvos pecsétszáma/jogosítványa | - |
| `service_provider_name` | String | nem | Foglalkozás-egészségügyi szolgáltató neve | 89/1995. Korm. r. |
| `tests_performed` | JSON | nem | Elvégzett vizsgálatok listája | - |
| `status` | Enum | igen | scheduled/completed/cancelled/no_show/pending_results | - |
| `cancellation_reason` | Text | nem | Lemondás oka | - |
| `cost` | Decimal | nem | Vizsgálat költsége (Ft) | - |
| `invoice_number` | String | nem | Számla száma | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.3 HealthFitnessAssessment (Alkalmassági minősítés)

Az üzemorvos által kiadott alkalmassági minősítés (alkalmas/korlátozottan alkalmas/alkalmatlan).

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `occupational_health_exam_id` | UUID | igen | Melyik vizsgálathoz | OccupationalHealthExam |
| `employment_id` | UUID | igen | Melyik munkatárs (denormalizált) | employment-entity.md |
| `position_id` | UUID | nem | Melyik munkakörre szól | position-entity.md |
| `assessment_date` | Date | igen | Minősítés dátuma | - |
| `fitness_status` | Enum | igen | fit/fit_with_restrictions/temporarily_unfit/permanently_unfit | Mvt. 54. § (4) |
| `fitness_valid_from` | Date | igen | Érvényesség kezdete | - |
| `fitness_valid_until` | Date | nem | Érvényesség vége (időszakos vizsgálatoknál) | - |
| `restrictions` | Text | nem | Korlátozások leírása (pl. "nehéz fizikai munka nem végezhető") | - |
| `recommended_restrictions` | JSON | nem | Strukturált korlátozások (no_night_shift, no_heavy_lifting, reduced_hours) | - |
| `follow_up_required` | Boolean | igen | Szükséges-e utánkövetés | - |
| `follow_up_date` | Date | nem | Utánkövetés javasolt időpontja | - |
| `occupational_physician_name` | String | igen | Aláíró orvos neve | - |
| `occupational_physician_license` | String | igen | Orvos pecsétszáma | - |
| `certificate_number` | String | nem | Igazolás száma | - |
| `certificate_document_id` | UUID | nem | Alkalmassági igazolás dokumentum | document-entity.md |
| `is_current` | Boolean | igen | Aktuális minősítés-e | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.4 WorkplaceHealthRisk (Munkahelyi egészségügyi kockázat)

Munkakörök/pozíciók egészségügyi kockázati besorolása.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `position_id` | UUID | igen | Melyik munkakör/pozíció | position-entity.md |
| `organization_unit_id` | UUID | nem | Szervezeti egység | organization-entity.md |
| `risk_assessment_date` | Date | igen | Kockázatértékelés dátuma | Mvt. 54. § (1) |
| `risk_level` | Enum | igen | low/medium/high/very_high | - |
| `hazard_factors` | JSON | igen | Veszélyforrások (noise, chemicals, radiation, biological, ergonomic, psychosocial) | 25/2000. EüM-SzCsM r. |
| `noise_exposure_db` | Integer | nem | Zajterhelés (dB) | 25/2000. EüM-SzCsM r. 2. § |
| `chemical_exposure` | JSON | nem | Vegyi anyagok expozíció | - |
| `radiation_exposure` | Boolean | nem | Ionizáló sugárzás-e | - |
| `biological_hazards` | JSON | nem | Biológiai veszélyforrások | - |
| `ergonomic_hazards` | JSON | nem | Ergonómiai kockázatok (nehéz fizikai munka, ismétlődő mozdulatok) | - |
| `psychosocial_hazards` | JSON | nem | Pszichoszociális kockázatok (stressz, éjszakai munka) | - |
| `required_exam_frequency_months` | Integer | igen | Kötelező vizsgálati gyakoriság (hónap) | 25/2000. EüM-SzCsM r. |
| `required_exam_type_id` | UUID | nem | Kötelező vizsgálattípus | OccupationalHealthExamType |
| `required_ppe` | JSON | nem | Kötelező egyéni védőeszközök | Mvt. 14. § |
| `control_measures` | Text | nem | Megelőző intézkedések | - |
| `reviewed_by_id` | UUID | nem | Kockázatértékelést készítő | person-entity.md |
| `approved_by_id` | UUID | nem | Jóváhagyó (munkavédelmi felelős) | person-entity.md |
| `next_review_date` | Date | nem | Következő felülvizsgálat dátuma | Mvt. 54. § |
| `is_active` | Boolean | igen | Aktív-e | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.5 HealthSurveillanceSchedule (Vizsgálati ütemterv)

Munkatársak egyéni vizsgálati ütemezése.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `employment_id` | UUID | igen | Melyik munkatárs | employment-entity.md |
| `position_id` | UUID | igen | Melyik munkakör | position-entity.md |
| `workplace_health_risk_id` | UUID | nem | Kapcsolódó kockázatértékelés | WorkplaceHealthRisk |
| `exam_type_id` | UUID | igen | Vizsgálattípus | OccupationalHealthExamType |
| `frequency_months` | Integer | igen | Gyakorisá

g hónapokban | - |
| `next_exam_due_date` | Date | igen | Következő vizsgálat esedékessége | - |
| `last_exam_date` | Date | nem | Utolsó vizsgálat dátuma | - |
| `last_exam_id` | UUID | nem | Utolsó vizsgálat | OccupationalHealthExam |
| `status` | Enum | igen | active/suspended/completed/cancelled | - |
| `suspension_reason` | Text | nem | Felfüggesztés oka (pl. szülési szabadság) | - |
| `reminder_sent_count` | Integer | igen | Hány emlékeztető lett küldve | - |
| `last_reminder_sent` | DateTime | nem | Utolsó emlékeztető időpontja | - |
| `is_overdue` | Boolean | számított | Lejárt-e (next_exam_due_date < current_date) | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.6 OccupationalDisease (Foglalkozási megbetegedés)

Munkaviszonnyal összefüggő megbetegedések nyilvántartása.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `employment_id` | UUID | igen | Érintett munkatárs | employment-entity.md |
| `disease_code` | String | nem | BNO kód (Betegségek Nemzetközi Osztályozása) | - |
| `disease_name` | String | igen | Megbetegedés megnevezése | - |
| `disease_category` | Enum | igen | respiratory/musculoskeletal/skin/hearing_loss/mental_health/cancer/other | - |
| `suspected_occupational_cause` | Text | igen | Feltételezett foglalkozási ok | - |
| `exposure_factors` | JSON | nem | Expozíciós tényezők (vegyi anyag, zaj, sugárzás) | - |
| `onset_date` | Date | nem | Tünetek kezdete | - |
| `diagnosis_date` | Date | igen | Diagnosztizálás dátuma | - |
| `reported_to_authority` | Boolean | igen | Bejelentve a hatóságnak | Mvt. 62. § |
| `authority_report_date` | Date | nem | Bejelentés dátuma | - |
| `authority_case_number` | String | nem | Hatósági ügyszám | - |
| `occupational_disease_recognized` | Boolean | nem | Foglalkozási megbetegedésként elismerve | - |
| `recognition_date` | Date | nem | Elismerés dátuma | - |
| `severity` | Enum | nem | mild/moderate/severe/permanent_disability | - |
| `work_restriction_required` | Boolean | nem | Munkavégzési korlátozás szükséges-e | - |
| `job_change_required` | Boolean | nem | Munkakör váltás szükséges-e | - |
| `compensation_claimed` | Boolean | nem | Kártérítési igény | - |
| `physician_name` | String | nem | Diagnosztizáló orvos | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.7 WorkAccident (Munkabaleseti esemény)

Munkahelyi balesetek dokumentálása és kivizsgálása.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `accident_number` | String | igen | Baleseti nyilvántartási szám | Mvt. 61. § |
| `employment_id` | UUID | igen | Sérült munkatárs | employment-entity.md |
| `accident_date` | DateTime | igen | Baleset időpontja | Mvt. 61. § |
| `accident_location` | String | igen | Baleset helyszíne | - |
| `organization_unit_id` | UUID | nem | Szervezeti egység | organization-entity.md |
| `position_id` | UUID | nem | Munkakör | position-entity.md |
| `accident_type` | Enum | igen | injury/near_miss/property_damage/occupational_disease_related | - |
| `severity` | Enum | igen | minor/serious/fatal/no_injury | Mvt. 61. § (súlyos/halálos baleset különleges bejelentés) |
| `injury_type` | Enum | nem | cut/fracture/burn/crush/amputation/poisoning/electric_shock/fall/other | - |
| `body_part_injured` | JSON | nem | Sérült testrész(ek) | - |
| `accident_description` | Text | igen | Baleset körülményeinek leírása | Mvt. 62. § |
| `root_cause` | Text | nem | Kiváltó ok | - |
| `contributing_factors` | JSON | nem | Közreható tényezők | - |
| `witnesses` | JSON | nem | Tanúk listája (név, employment_id) | - |
| `immediate_action_taken` | Text | nem | Azonnali intézkedés | - |
| `first_aid_provided` | Boolean | igen | Elsősegély nyújtva-e | - |
| `medical_treatment_required` | Boolean | igen | Orvosi ellátás szükséges volt-e | - |
| `hospital_name` | String | nem | Kórház neve (ha kórházi ellátás) | - |
| `days_lost` | Integer | nem | Munkanapok kiesése | - |
| `return_to_work_date` | Date | nem | Munkába állás dátuma | - |
| `reported_to_authority` | Boolean | igen | Bejelentve a hatóságnak | Mvt. 61. § (8 órán belül súlyos/halálos) |
| `authority_report_date` | DateTime | nem | Bejelentés időpontja | - |
| `authority_case_number` | String | nem | Hatósági ügyszám | - |
| `investigation_status` | Enum | igen | pending/in_progress/completed/closed | - |
| `investigation_report_id` | UUID | nem | Kivizsgálási jelentés | document-entity.md |
| `corrective_actions` | Text | nem | Korrekciós intézkedések | - |
| `responsible_person_id` | UUID | nem | Felelős személy | person-entity.md |
| `preventive_measures` | Text | nem | Megelőző intézkedések | - |
| `compensation_claimed` | Boolean | nem | Kártérítési igény | - |
| `insurance_claim_number` | String | nem | Biztosítási kárszám | - |
| `investigation_completed_date` | Date | nem | Kivizsgálás lezárásának dátuma | - |
| `investigator_id` | UUID | nem | Kivizsgáló személy | person-entity.md |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.8 SafetyTrainingRecord (Munkavédelmi oktatási jegyzőkönyv)

Munkavédelmi, tűzvédelmi, elsősegély oktatások jegyzőkönyve (integráció Learning entitással).

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `training_completion_id` | UUID | nem | Kapcsolódó képzési teljesítés | learning-entity.md TrainingCompletion |
| `employment_id` | UUID | igen | Melyik munkatárs | employment-entity.md |
| `training_type` | Enum | igen | general_safety/job_specific_safety/fire_safety/first_aid/ergonomics/chemical_safety/electrical_safety | Mvt. 54. § |
| `training_category` | Enum | igen | initial/annual_refresher/job_change/extraordinary | Mvt. 54. § (3) |
| `training_date` | Date | igen | Oktatás dátuma | - |
| `training_duration_hours` | Decimal | igen | Oktatás időtartama (óra) | - |
| `training_content` | Text | nem | Oktatás tartalma | - |
| `instructor_name` | String | igen | Oktató neve | - |
| `instructor_qualification` | String | nem | Oktató képesítése | - |
| `training_location` | String | nem | Oktatás helyszíne | - |
| `assessment_completed` | Boolean | igen | Számonkérés megtörtént-e | Mvt. 54. § (3) |
| `assessment_result` | Enum | nem | passed/failed | - |
| `certificate_issued` | Boolean | igen | Igazolás kiállítva-e | - |
| `certificate_number` | String | nem | Igazolás száma | - |
| `certificate_valid_until` | Date | nem | Érvényesség (általában 1 év) | - |
| `protocol_document_id` | UUID | nem | Jegyzőkönyv dokumentum | document-entity.md |
| `next_training_due_date` | Date | nem | Következő oktatás esedékessége | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.9 PersonalProtectiveEquipment (Egyéni védőeszköz kiadás)

Egyéni védőeszközök (sisak, védőszemüveg, kesztyű, védőruha) kiadásának nyilvántartása.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `employment_id` | UUID | igen | Melyik munkatárs | employment-entity.md |
| `position_id` | UUID | nem | Munkakör | position-entity.md |
| `ppe_type` | Enum | igen | helmet/safety_glasses/gloves/respirator/ear_protection/safety_shoes/protective_clothing/fall_protection/other | Mvt. 14. § |
| `ppe_name` | String | igen | Védőeszköz megnevezése | - |
| `ppe_standard` | String | nem | Szabvány (pl. EN 166, EN 388) | - |
| `manufacturer` | String | nem | Gyártó | - |
| `model` | String | nem | Típus | - |
| `serial_number` | String | nem | Gyári szám | - |
| `size` | String | nem | Méret | - |
| `issue_date` | Date | igen | Kiadás dátuma | - |
| `expiry_date` | Date | nem | Lejárati idő | - |
| `replacement_due_date` | Date | nem | Csere esedékessége | - |
| `quantity_issued` | Integer | igen | Kiadott mennyiség | - |
| `cost_per_unit` | Decimal | nem | Egységár (Ft) | - |
| `issued_by_id` | UUID | nem | Kiadó személy | person-entity.md |
| `acknowledgement_signed` | Boolean | igen | Átvételi elismervény aláírva | Mvt. 14. § |
| `acknowledgement_document_id` | UUID | nem | Átvételi elismervény | document-entity.md |
| `status` | Enum | igen | in_use/returned/damaged/lost/expired | - |
| `return_date` | Date | nem | Visszaadás dátuma | - |
| `return_condition` | Enum | nem | good/fair/poor/damaged | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

## 3. GDPR kategorizáció és adatkezelés

### Foglalkozás-egészségügyi adatok kezelése

#### Jogalap
- **GDPR Art. 6(1)(c)**: jogi kötelezettség teljesítése (Mvt. 54. §)
- **GDPR Art. 9(2)(b)**: foglalkoztatással kapcsolatos egészségügyi adatok kezelése (munkajog)
- **GDPR Art. 9(2)(h)**: megelőző célú foglalkozás-orvostan

#### Adatkategóriák
- **Különleges személyes adat (egészségügyi)**: foglalkozás-egészségügyi vizsgálat eredménye, alkalmassági minősítés, foglalkozási megbetegedés, munkabaleseti sérülés
- **Általános személyes adat**: vizsgálati időpontok, oktatási jegyzőkönyvek, védőeszköz kiadás

#### Adatmegőrzési idők
- **Foglalkozás-egészségügyi vizsgálatok**: munkaviszony megszűnése + 50 év (89/1995. Korm. r. 9. §)
- **Munkabaleseti jegyzőkönyvek**: munkaviszony megszűnése + 50 év (Mvt. 62. §)
- **Foglalkozási megbetegedések**: munkaviszony megszűnése + 50 év
- **Munkavédelmi oktatási jegyzőkönyvek**: munkaviszony megszűnése + 5 év
- **Védőeszköz kiadási nyilvántartás**: visszaadás/lejárat + 3 év

### Hozzáférési jogok
- **Munkatárs**: saját alkalmassági minősítés, vizsgálati időpontok, oktatási jegyzőkönyvek
- **Vezető**: **NINCS** hozzáférése beosztottak egészségügyi vizsgálati eredményeihez (csak alkalmassági státusz: alkalmas/alkalmatlan)
- **HR**: alkalmassági státusz, vizsgálati ütemezés (egészségügyi részletek NEM)
- **Munkavédelmi felelős**: kockázatértékelés, baleseti jegyzőkönyvek, oktatási nyilvántartás
- **Foglalkozás-egészségügyi szolgálat**: teljes hozzáférés egészségügyi adatokhoz

## 4. Hozzáférési szintek

| Entitás | HR Admin | Manager | Employee | Safety Officer | Occupational Physician |
|---|---|---|---|---|---|
| OccupationalHealthExamType | CRUD | - | - | Read | Read |
| OccupationalHealthExam | Create/Read (scheduling) | - | Read (saját) | Read | CRUD |
| HealthFitnessAssessment | Read (fitness status only) | Read (fitness status only) | Read (saját) | Read (fitness status only) | CRUD |
| WorkplaceHealthRisk | Read | Read (team positions) | Read (saját pozíció) | CRUD | Read |
| HealthSurveillanceSchedule | CRUD | - | Read (saját) | Read | Read |
| OccupationalDisease | Read | - | Read (saját) | CRUD | CRUD |
| WorkAccident | Read | Read (team) | Read (saját) | CRUD | Read |
| SafetyTrainingRecord | CRUD | Read (team) | Read (saját) | CRUD | - |
| PersonalProtectiveEquipment | CRUD | Read (team) | Read (saját) | CRUD | - |

## 5. Validációs szabályok

### OccupationalHealthExam
- **Belépési vizsgálat kötelező**: új Employment létrehozásakor automatic scheduling
- **Vizsgálat a munkakezdés előtt**: pre_employment exam actual_exam_date <= employment start_date

### HealthFitnessAssessment
- **Alkalmatlan minősítés**: fitness_status=permanently_unfit esetén Employment status=suspended/terminated trigger
- **Érvényesség lejárat**: fitness_valid_until < current_date → új vizsgálat ütemezése
- **Korlátozás enforcement**: fit_with_restrictions esetén timetracking/shift assignment validáció

### HealthSurveillanceSchedule
- **Lejárt vizsgálat**: next_exam_due_date < current_date és status=active → is_overdue=true
- **Emlékeztetők**:
  - 60 nappal esedékesség előtt: 1. emlékeztető
  - 30 nappal esedékesség előtt: 2. emlékeztető
  - 14 nappal esedékesség előtt: 3. emlékeztető
  - Lejárat napján: 4. emlékeztető
  - Lejárat után 7 naponként: ismétlő emlékeztetők

### WorkAccident
- **Súlyos/halálos baleset bejelentés**: severity=serious/fatal → reported_to_authority kötelező, authority_report_date <= accident_date + 8 óra
- **Kivizsgálás kötelező**: minden baleset esetén investigation_status tracking

### SafetyTrainingRecord
- **Belépési oktatás kötelező**: training_category=initial, training_date <= employment start_date
- **Éves ismétlés**: training_category=annual_refresher, certificate_valid_until 1 év
- **Számonkérés kötelező**: assessment_completed=true (Mvt. 54. §)

## 6. Integrációs pontok

### Külső rendszerek
- **Foglalkozás-egészségügyi szolgáltató rendszere**:
  - Vizsgálati időpontok szinkronizálása
  - Alkalmassági minősítések automatikus importálása
- **Munkavédelmi hatóság (OMMF)**:
  - Munkabaleseti bejelentés elektronikus úton
  - Foglalkozási megbetegedés bejelentés
- **ÁNTSZ (Állami Népegészségügyi és Tisztiorvosi Szolgálat)**:
  - Veszélyes anyagokkal kapcsolatos expozíciós nyilvántartás
- **Biztosító társaságok**:
  - Munkabaleseti kárigények
  - Foglalkozási megbetegedés biztosítási igények

### Belső entitások
- **Employment**: alkalmassági státusz validáció, munkakezdés feltétele
- **Position**: kockázatértékelés pozíciónként, kötelező vizsgálatok meghatározása
- **Learning**: munkavédelmi oktatások (SafetyTrainingRecord ↔ TrainingCompletion)
- **TimeTracking**: munkaidő-korlátozások (éjszakai munka tilalom alkalmatlansági korlátozás esetén)
- **Document**: vizsgálati jegyzőkönyvek, baleseti jelentések, oktatási anyagok
- **Leave**: táppénz, baleset utáni betegszabadság
- **Compensation**: baleseti pótlék, foglalkozási megbetegedés miatti kompenzáció

## 7. Kapcsolódó entitások

- **employment-entity.md**: Employment (foglalkozás-egészségügyi vizsgálatok munkatársakhoz)
- **person-entity.md**: Person (egészségügyi adatok személyhez, munkavédelmi felelősök)
- **position-entity.md**: Position (kockázatértékelés munkakörönként)
- **organization-entity.md**: OrgUnit, WorkLocation (baleset helyszínek, kockázatértékelés)
- **learning-entity.md**: TrainingCompletion (munkavédelmi oktatások)
- **document-entity.md**: Document (jegyzőkönyvek, igazolások)
- **timetracking-entity.md**: DailyTimeRecord (éjszakai munka tiltás, korlátozások)
- **leave-entity.md**: LeaveRecord (táppénz, baleset utáni betegszabadság)

## 8. Üzleti folyamatok

### 8.1 Belépéskori foglalkozás-egészségügyi vizsgálat

1. **Új Employment létrehozása** → automatikus OccupationalHealthExam scheduling (exam_category=pre_employment)
2. **Vizsgálati időpont egyeztetés**: HR egyezteti a szolgáltatóval
3. **Vizsgálat elvégzése**: status=completed
4. **Alkalmassági minősítés kiadása**: HealthFitnessAssessment létrehozása
   - fitness_status=fit → Employment status=active engedélyezett
   - fitness_status=fit_with_restrictions → korlátozások rögzítése, TimeTracking/ShiftAssignment validáció
   - fitness_status=permanently_unfit → Employment nem indítható
5. **Következő időszakos vizsgálat ütemezése**: HealthSurveillanceSchedule létrehozása

### 8.2 Időszakos foglalkozás-egészségügyi vizsgálat

1. **Automatikus ütemezés**: HealthSurveillanceSchedule next_exam_due_date alapján
2. **Emlékeztetők**: 60/30/14 nappal esedékesség előtt
3. **Vizsgálat ütemezése**: OccupationalHealthExam scheduling
4. **Vizsgálat elvégzése**: status=completed
5. **Alkalmassági minősítés frissítése**: új HealthFitnessAssessment (is_current=true, korábbi is_current=false)
6. **HealthSurveillanceSchedule frissítése**: last_exam_date, next_exam_due_date kalkuláció

### 8.3 Munkabaleseti folyamat

1. **Baleset történik**: WorkAccident létrehozása (accident_date, severity, description)
2. **Azonnali intézkedések**: first_aid_provided, immediate_action_taken rögzítése
3. **Hatósági bejelentés** (ha severity=serious/fatal): reported_to_authority=true, authority_report_date <= 8 óra
4. **Kivizsgálás indítása**: investigation_status=in_progress, investigator_id kijelölése
5. **Kivizsgálási jelentés készítése**: root_cause, contributing_factors, corrective_actions
6. **Megelőző intézkedések**: preventive_measures implementálása
7. **Lezárás**: investigation_status=completed, investigation_completed_date

### 8.4 Munkavédelmi oktatási folyamat

1. **Oktatás ütemezése**: TrainingSession létrehozása (Learning modul)
2. **Oktatás megtartása**: TrainingAttendance
3. **Számonkérés**: TrainingAssessment (assessment_result=passed/failed)
4. **Jegyzőkönyv rögzítése**: SafetyTrainingRecord létrehozása
   - training_completion_id kapcsolat
   - assessment_completed=true, certificate_issued=true
   - certificate_valid_until = training_date + 1 év
5. **Következő oktatás ütemezése**: next_training_due_date = training_date + 1 év

### 8.5 Kockázatértékelési folyamat

1. **Új Position létrehozása** vagy meglévő Position felülvizsgálata
2. **Kockázatértékelés készítése**: WorkplaceHealthRisk létrehozása
   - hazard_factors azonosítása
   - risk_level meghatározása
   - required_exam_frequency_months beállítása
3. **Megelőző intézkedések**: control_measures, required_ppe meghatározása
4. **Jóváhagyás**: approved_by_id (munkavédelmi felelős)
5. **Employment-ekhez kapcsolódó vizsgálati ütemezés frissítése**: HealthSurveillanceSchedule frequency_months módosítása

## 9. Reporting és analitika

### Compliance reporting
- **Lejárt alkalmassági minősítések**: fitness_valid_until < current_date lista
- **Esedékes vizsgálatok**: HealthSurveillanceSchedule is_overdue=true
- **Munkabaleseti statisztikák**: balesetek száma, súlyossági megoszlás, balesetmentes napok
- **Foglalkozási megbetegedések**: betegségtípus, expozíciós tényező szerinti bontás
- **Munkavédelmi oktatási teljesítettség**: certificate_valid_until érvényes igazolások aránya

### Munkabaleseti mutatók
- **Baleseti gyakorisági mutató (Incident Rate)**: balesetek száma / munkaórák × 200,000
- **Súlyossági mutató**: munkanapok kiesése / balesetek száma
- **LTIFR (Lost Time Injury Frequency Rate)**: munkanapkiesést okozó balesetek / munkaórák × 1,000,000
- **Balesetmentes napok száma**: utolsó baleset óta eltelt napok

### Egészségügyi surveillance
- **Alkalmassági státusz megoszlás**: fit / fit_with_restrictions / unfit arány
- **Korlátozások gyakorisága**: recommended_restrictions típusonkénti bontás
- **Vizsgálati költségek**: OccupationalHealthExam cost összesítése évente/negyedévente
- **Vizsgálati megfelelés**: scheduled vs completed exam arány

## 10. Nyitott kérdések és jövőbeli fejlesztések

### Technológiai kérdések
1. **Foglalkozás-egészségügyi szolgáltató integráció mélysége**: API integráció vagy manuális adatrögzítés?
2. **Biometrikus hozzáférés-szabályozás**: egészségügyi adatok szigorú hozzáférési audit trail
3. **Mobil alkalmazás**: védőeszköz kiadás mobilon, QR kód alapú nyilvántartás
4. **IoT integráció**: zajmérők, légszennyezettség-mérők automatikus adatrögzítése

### Funkcionális fejlesztések
1. **Ergonómiai kockázatértékelés**: részletes ergonómiai modulok (ülőmunka, nehéz fizikai munka)
2. **Pszichoszociális kockázatértékelés**: stressz, munkahelyi bántalmazás, kiégés felmérése
3. **Védőoltási nyilvántartás**: Hepatitis B, influenza, COVID-19 oltások követése (különösen egészségügy)
4. **Audiometria/spirometria trend analysis**: hallásromlás, tüdőfunkció változás időbeli követése
5. **Sugárvédelmi dozimetria**: ionizáló sugárzásnak kitett munkakörök dózis nyilvántartása

### Compliance és jogi
1. **OMMF (Országos Munkavédelmi és Munkaügyi Főfelügyelet) elektronikus bejelentés**: automatizálás
2. **ÁNTSZ kémiai anyag expozíciós nyilvántartás**: veszélyes anyagokkal dolgozók külön nyilvántartása
3. **EU OSH (Occupational Safety and Health) direktívák**: naprakészség biztosítása
4. **ISO 45001 (Occupational Health and Safety Management)**: megfelelés

### Integrációk
1. **Biztosító társaságok API**: munkabaleseti kárigények automatikus továbbítása
2. **OMMF / ÁNTSZ elektronikus bejelentés**: kötelező bejelentések automatizálása
3. **Kórházi információs rendszerek**: baleset utáni kórházi kezelés adatai
4. **ERP rendszerek**: védőeszköz beszerzés, költségkövetés

### AI és automatizálás
1. **Balesetmegelőzési predikció**: ML alapú kockázatelemzés (mely munkakörökben várható baleset)
2. **Kockázatértékelési asszisztens**: AI alapú hazard identification
3. **Chatbot**: munkavédelmi tanácsadás, GYIK (pl. "Milyen védőeszközt kell viselnem?")
4. **Automatikus compliance alert**: lejáró vizsgálatok, oktatások, védőeszközök

---

**Megjegyzések:**
- A foglalkozás-egészségügyi adatok **különleges személyes adatok** (GDPR Art. 9), ezért fokozott védelem szükséges
- Az üzemorvos szakmai titokhoz kötött – a HRMS **nem tárolhatja** a teljes orvosi dokumentációt, csak az alkalmassági minősítést
- A munkáltatónak **nincs joga** megismerni a pontos diagnózist, csak az alkalmassági státuszt
- 50 éves megőrzési idő különösen kritikus – az adatarchívás és jogszabály-változások követése elengedhetetlen
