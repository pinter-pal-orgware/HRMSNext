# Offboarding (Kilépési folyamat) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer
> **Verzió:** 0.1 – első tervezet
> **Utolsó frissítés:** 2025. február
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md, employment-entity.md, organization-entity.md, position-entity.md, document-entity.md, succession-planning-entity.md, compensation-entity.md, leave-entity.md

---

## 1. Az entitás célja és hatóköre

A kilépési (offboarding) entitások biztosítják a munkaviszony megszűnésének strukturált, jogszabályoknak megfelelő és hatékony kezelését, a szervezeti tudás megőrzését, az eszközök és hozzáférések visszavonását, valamint a kilépő munkatársak tapasztalatainak gyűjtését.

### Hatókör

A kilépési entitások lefedik:
- **Kilépési folyamat menedzsment**: jogviszony megszűnésének teljes folyamata
- **Kilépési feladatlista (checklist)**: standardizált kilépési lépések pozíciónként
- **Exit interview (kilépési interjú)**: kilépési okok feltárása, tapasztalatok gyűjtése
- **Eszközök visszavétele**: laptop, telefon, belépőkártya, céges autó, egyéb eszközök
- **Hozzáférések visszavonása**: IT rendszerek, épületek, dokumentumok, levelezés
- **Tudásátadás**: kritikus tudás dokumentálása, átadása utódnak vagy csapatnak
- **Végső elszámolás**: végkielégítés, fel nem vett szabadság, kölcsönök elszámolása
- **Dokumentumok kezelése**: munkaviszony megszüntetése, titoktartási nyilatkozat, versenytilalmi megállapodás
- **Alumni kapcsolattartás**: korábbi munkatársak adatbázisa, rehire lehetőség

### Foglalkoztatási típus specifikus követelmények

#### Minden jogviszony - Általános Mt. szabályok
- **Munkaviszony megszűnése/megszüntetése**: Mt. 63-78. §
  - Megszűnés: határozott idő lejárta, közös megegyezés, halál
  - Felmondás: munkáltató/munkavállaló kezdeményezésére
  - Azonnali hatályú felmondás: súlyos kötelezettségszegés
- **Felmondási idő**: Mt. 67-72. §
  - Általános: 30 nap
  - Munkáltatói felmondás: munkaviszony időtartamától függően 30-90 nap
  - Végkielégítés: 3 év feletti munkaviszony esetén (Mt. 77. §)
- **Igazolás kiadása**: Mt. 81. §
  - Munkáltató köteles igazolást kiadni a jogviszony időtartamáról
- **GDPR adattörlés**: GDPR Art. 17 - törléshez való jog
  - Munkaviszony megszűnése után jogszabályi megőrzési idők

#### Közszféra (Kjt., Kttv., Kit.)
- **Közalkalmazotti jogviszony megszűnése**: Kjt. 23-37. §
  - Felmentés: Kjt. 30. § (60 napos felmentési idő)
  - Végkielégítés: Kjt. 37. § (3 év feletti jogviszony, 1-6 havi illetmény)
- **Közszolgálati jogviszony megszűnése**: Kttv. 63-74. §
  - Felmentés: Kttv. 63. § (30 napos felmentési idő, kivéve vezető: 60 nap)
  - Végkielégítés: Kttv. 66. § (3 év feletti jogviszony)
- **Kormánytisztviselői jogviszony megszűnése**: Kit. 62-73. §
  - Felmentés: Kit. 63. § (60 napos felmentési idő)
  - Végkielégítés: Kit. 67. § (3 év feletti jogviszony, 2-8 havi illetmény)
- **Összeférhetetlenség, titoktartás**: Kit. 83-85. §, Kttv. 84-87. §
  - Titoktartási kötelezettség jogviszony megszűnése után is fennáll

#### Pedagógusok (Púétv.)
- **Pedagógus jogviszony megszűnése**: Púétv. 71-77. §
  - Felmentés: Púétv. 72. § (60 napos felmentési idő)
  - Végkielégítés: Púétv. 77. § (3 év feletti jogviszony)

#### Egészségügyi dolgozók (Eszjtv.)
- **Egészségügyi szolgálati jogviszony megszűnése**: Eszjtv. 35-42. §
  - Felmentés: Eszjtv. 36. § (30-60 napos felmentési idő)
  - Végkielégítés: Eszjtv. 42. § (3 év feletti jogviszony)

#### Versenytilalom és titoktartás
- **Versenytilalmi megállapodás**: Mt. 228-229. §
  - Munkaviszony megszűnése után is hatályos (max. 2 év)
  - Ellenérték fizetése kötelező (alapbér 1/3-a havonta)
- **Üzleti titok védelme**: Ptk. 2:47. §, 2018. évi LIV. törvény
  - Titoktartási kötelezettség jogviszony után is

## 2. Entitásstruktúra

### 2.1 OffboardingCase (Kilépési eset)

Egy munkatárs kilépési folyamatának központi nyilvántartása.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `case_number` | String | igen | Kilépési eset azonosító (pl. "OFF-2026-001") | - |
| `employment_id` | UUID | igen | Melyik munkatárs | employment-entity.md |
| `position_id` | UUID | igen | Pozíció (denormalizált) | position-entity.md |
| `organization_unit_id` | UUID | igen | Szervezeti egység (denormalizált) | organization-entity.md |
| `termination_type` | Enum | igen | resignation/termination_by_employer/mutual_agreement/retirement/end_of_fixed_term/death/immediate_dismissal | Mt. 63-78. § |
| `termination_reason` | Enum | nem | voluntary_resignation/better_opportunity/relocation/personal_reasons/performance_issues/redundancy/restructuring/retirement/health_reasons/other | Kategorizálás |
| `termination_reason_detail` | Text | nem | Részletes indoklás | - |
| `initiated_by` | Enum | igen | employee/employer/mutual | - |
| `resignation_date` | Date | nem | Felmondás/felmentés bejelentésének dátuma | Mt. 67. § |
| `resignation_document_id` | UUID | nem | Felmondó levél dokumentum | document-entity.md |
| `notice_period_days` | Integer | igen | Felmondási idő (nap) | Mt. 67-72. § |
| `notice_period_waived` | Boolean | nem | Felmondási idő elengedve-e | Mt. 67. § (4) |
| `last_working_day` | Date | igen | Utolsó munkanap | - |
| `official_termination_date` | Date | igen | Jogviszony megszűnésének hivatalos dátuma | Mt. 63. § |
| `is_regrettable_loss` | Boolean | nem | Sajnálatos távozás-e (HR értékelés: értékes munkatárs) | Retention analytics |
| `is_rehire_eligible` | Boolean | nem | Újrafoglalkoztatható-e | - |
| `rehire_eligibility_notes` | Text | nem | Újrafoglalkoztathatóság indoklása | - |
| `manager_id` | UUID | igen | Közvetlen vezető | person-entity.md |
| `hr_contact_id` | UUID | igen | Felelős HR kapcsolattartó | person-entity.md |
| `offboarding_checklist_template_id` | UUID | nem | Alkalmazott checklist sablon | OffboardingChecklistTemplate |
| `checklist_completion_percentage` | Integer | számított | Checklist teljesítettség (%) | - |
| `exit_interview_scheduled` | Boolean | igen | Kilépési interjú ütemezve-e | - |
| `exit_interview_completed` | Boolean | igen | Kilépési interjú megtörtént-e | - |
| `knowledge_transfer_required` | Boolean | igen | Tudásátadás szükséges-e | - |
| `knowledge_transfer_completed` | Boolean | igen | Tudásátadás megtörtént-e | - |
| `successor_identified` | Boolean | nem | Utód azonosítva-e | succession-planning-entity.md |
| `successor_employment_id` | UUID | nem | Utód munkatárs | employment-entity.md |
| `final_clearance_completed` | Boolean | igen | Végső elszámolás lezárva-e | - |
| `all_assets_returned` | Boolean | számított | Összes eszköz visszavéve-e | - |
| `all_access_revoked` | Boolean | számított | Összes hozzáférés visszavonva-e | - |
| `status` | Enum | igen | initiated/in_progress/pending_clearance/completed/cancelled | - |
| `completion_date` | Date | nem | Kilépési folyamat lezárásának dátuma | - |
| `alumni_network_opt_in` | Boolean | nem | Volt munkavállalói hálózatba opt-in | - |
| `reference_contact_consent` | Boolean | nem | Referencia adáshoz hozzájárulás | GDPR |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.2 OffboardingChecklistTemplate (Kilépési feladatlista sablon)

Pozíció/szervezeti egység specifikus kilépési feladatlisták sablonja.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `template_name` | String | igen | Sablon neve (pl. "Standard Office Employee", "IT Developer", "Sales Manager") | - |
| `template_code` | String | igen | Sablon kód | - |
| `description` | Text | nem | Leírás | - |
| `applicable_employment_types` | JSON | nem | Alkalmazható jogviszony típusok | - |
| `applicable_positions` | JSON | nem | Alkalmazható pozíciók (position_id lista) | - |
| `applicable_org_units` | JSON | nem | Alkalmazható szervezeti egységek | - |
| `is_default` | Boolean | igen | Alapértelmezett sablon-e | - |
| `tasks_count` | Integer | számított | Feladatok száma a sablonban | - |
| `estimated_completion_days` | Integer | nem | Becsült teljesítési idő (nap) | - |
| `is_active` | Boolean | igen | Aktív-e | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.3 OffboardingTask (Kilépési feladat)

Konkrét kilépési feladatok egy offboarding case-hez.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `offboarding_case_id` | UUID | igen | Melyik kilépési esethez | OffboardingCase |
| `task_category` | Enum | igen | asset_return/access_revocation/documentation/knowledge_transfer/hr_admin/it_admin/finance/facility/other | Kategória |
| `task_name` | String | igen | Feladat neve | - |
| `task_description` | Text | nem | Részletes leírás | - |
| `responsible_department` | Enum | igen | hr/it/finance/facility/manager/employee | Felelős osztály |
| `assigned_to_id` | UUID | nem | Felelős személy | person-entity.md |
| `due_date` | Date | igen | Határidő (általában last_working_day vagy előtte) | - |
| `due_date_offset_days` | Integer | nem | Határidő offset (last_working_day - X nap) | - |
| `priority` | Enum | igen | critical/high/medium/low | - |
| `is_mandatory` | Boolean | igen | Kötelező-e (blockoló a clearance-hez) | - |
| `sequence_order` | Integer | nem | Sorrend (ha van függőség) | - |
| `depends_on_task_id` | UUID | nem | Függőség (másik feladat completion-jétől függ) | - |
| `status` | Enum | igen | pending/in_progress/completed/cancelled/not_applicable | - |
| `started_date` | Date | nem | Kezdés dátuma | - |
| `completed_date` | Date | nem | Teljesítés dátuma | - |
| `completed_by_id` | UUID | nem | Ki teljesítette | person-entity.md |
| `completion_notes` | Text | nem | Teljesítési megjegyzések | - |
| `is_overdue` | Boolean | számított | Lejárt-e (due_date < current_date és status != completed) | - |
| `reminder_sent_count` | Integer | igen | Hány emlékeztető lett küldve | - |
| `last_reminder_sent` | DateTime | nem | Utolsó emlékeztető időpontja | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.4 ExitInterview (Kilépési interjú)

Kilépő munkatárssal készített strukturált interjú.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `offboarding_case_id` | UUID | igen | Melyik kilépési esethez | OffboardingCase |
| `employment_id` | UUID | igen | Kilépő munkatárs (denormalizált) | employment-entity.md |
| `interview_date` | Date | igen | Interjú dátuma | - |
| `interview_type` | Enum | igen | face_to_face/video_call/phone/survey_only | - |
| `interviewer_id` | UUID | nem | Interjúztató (általában HR) | person-entity.md |
| `interview_location` | String | nem | Helyszín | - |
| `interview_duration_minutes` | Integer | nem | Időtartam (perc) | - |
| `primary_reason_for_leaving` | Enum | nem | better_opportunity/career_growth/compensation/work_life_balance/relocation/management_issues/company_culture/commute/retirement/health/other | - |
| `secondary_reasons` | JSON | nem | Másodlagos okok (több is lehet) | - |
| `new_employer_industry` | String | nem | Új munkáltató iparága (ha applicable) | - |
| `new_role_title` | String | nem | Új pozíció címe | - |
| `compensation_factor` | Enum | nem | not_a_factor/minor_factor/major_factor/primary_reason | Bér szerepe |
| `compensation_increase_percentage` | Decimal | nem | Béremelés mértéke új helyen (%) | - |
| `would_recommend_company` | Enum | nem | definitely_yes/probably_yes/neutral/probably_no/definitely_no | eNPS |
| `likelihood_to_return` | Enum | nem | very_likely/likely/neutral/unlikely/very_unlikely | Boomerang hiring |
| `overall_satisfaction` | Integer | nem | Összesített elégedettség (1-10) | - |
| `satisfaction_job_role` | Integer | nem | Munkakör elégedettség (1-10) | - |
| `satisfaction_management` | Integer | nem | Vezetés elégedettség (1-10) | - |
| `satisfaction_team` | Integer | nem | Csapat elégedettség (1-10) | - |
| `satisfaction_compensation` | Integer | nem | Javadalmazás elégedettség (1-10) | - |
| `satisfaction_benefits` | Integer | nem | Juttatások elégedettség (1-10) | - |
| `satisfaction_work_life_balance` | Integer | nem | Work-life balance elégedettség (1-10) | - |
| `satisfaction_career_development` | Integer | nem | Karrierfejlődés elégedettség (1-10) | - |
| `satisfaction_company_culture` | Integer | nem | Vállalati kultúra elégedettség (1-10) | - |
| `liked_most` | Text | nem | Mi tetszett leginkább | - |
| `liked_least` | Text | nem | Mi tetszett legkevésbé | - |
| `suggestions_for_improvement` | Text | nem | Fejlesztési javaslatok | - |
| `manager_feedback` | Text | nem | Közvetlen vezetőről visszajelzés | - |
| `concerns_or_issues` | Text | nem | Felmerült problémák (zaklatás, diszkrimináció) | - |
| `is_confidential` | Boolean | igen | Bizalmas-e (név nélküli riportokban) | - |
| `consent_to_share_feedback` | Boolean | igen | Hozzájárulás visszajelzés megosztásához (manager) | - |
| `follow_up_required` | Boolean | nem | Követő intézkedés szükséges-e (pl. HR investigation) | - |
| `follow_up_notes` | Text | nem | Követő intézkedések | - |
| `interview_document_id` | UUID | nem | Interjú jegyzőkönyv | document-entity.md |
| `survey_response_id` | String | nem | Külső survey rendszer válasz ID | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.5 AssetReturn (Eszköz visszavétel)

Céges eszközök visszavételének nyilvántartása.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `offboarding_case_id` | UUID | igen | Melyik kilépési esethez | OffboardingCase |
| `employment_id` | UUID | igen | Munkatárs (denormalizált) | employment-entity.md |
| `asset_type` | Enum | igen | laptop/desktop/monitor/phone/tablet/access_card/vehicle/office_keys/credit_card/uniform/equipment/other | Eszköz típus |
| `asset_name` | String | igen | Eszköz megnevezése | - |
| `asset_description` | Text | nem | Részletes leírás | - |
| `asset_serial_number` | String | nem | Sorozatszám / eszköz azonosító | - |
| `asset_tag` | String | nem | Leltári szám | - |
| `asset_value` | Decimal | nem | Eszköz értéke (Ft) | - |
| `issued_date` | Date | nem | Kiadás dátuma | - |
| `return_due_date` | Date | igen | Visszaadás határideje (általában last_working_day) | - |
| `return_date` | Date | nem | Tényleges visszaadás dátuma | - |
| `return_location` | String | nem | Visszaadás helyszíne | - |
| `received_by_id` | UUID | nem | Ki vette át | person-entity.md |
| `condition_on_return` | Enum | nem | excellent/good/fair/damaged/lost | Állapot |
| `condition_notes` | Text | nem | Állapot megjegyzések | - |
| `requires_data_wipe` | Boolean | nem | Adattörlés szükséges-e (IT eszközök) | GDPR |
| `data_wipe_completed` | Boolean | nem | Adattörlés megtörtént-e | - |
| `data_wipe_date` | Date | nem | Adattörlés dátuma | - |
| `data_wipe_certificate_id` | UUID | nem | Adattörlési tanúsítvány | document-entity.md |
| `replacement_cost` | Decimal | nem | Pótlási költség (ha damaged/lost) | - |
| `deduction_from_final_pay` | Boolean | nem | Levonás végkielégítésből | - |
| `status` | Enum | igen | pending/returned/damaged/lost/not_applicable | - |
| `is_blocking_clearance` | Boolean | igen | Blokkolja-e a végső elszámolást | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.6 AccessRevocation (Hozzáférés visszavonás)

IT rendszerek, épületek, dokumentumok hozzáférésének visszavonása.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `offboarding_case_id` | UUID | igen | Melyik kilépési esethez | OffboardingCase |
| `employment_id` | UUID | igen | Munkatárs (denormalizált) | employment-entity.md |
| `access_type` | Enum | igen | email/network/application/database/vpn/building/room/cloud_service/shared_drive/other | Hozzáférés típus |
| `access_name` | String | igen | Rendszer/hely neve | - |
| `access_description` | Text | nem | Részletes leírás | - |
| `username` | String | nem | Felhasználónév / azonosító | - |
| `revocation_priority` | Enum | igen | immediate/before_last_day/on_last_day/after_last_day | Visszavonás sürgőssége |
| `revocation_due_date` | Date | igen | Visszavonás határideje | - |
| `revocation_date` | Date | nem | Tényleges visszavonás dátuma | - |
| `revoked_by_id` | UUID | nem | Ki vonta vissza (IT admin) | person-entity.md |
| `email_forwarding_required` | Boolean | nem | Email továbbítás szükséges-e (átmeneti) | - |
| `email_forwarding_to_id` | UUID | nem | Email továbbítás címzettje | person-entity.md |
| `email_forwarding_duration_days` | Integer | nem | Email továbbítás időtartama (nap) | - |
| `data_backup_required` | Boolean | nem | Adat backup szükséges-e (személyes drive, emails) | - |
| `data_backup_completed` | Boolean | nem | Adat backup megtörtént-e | - |
| `data_backup_location` | String | nem | Backup helye | - |
| `account_deletion_date` | Date | nem | Fiók törlésének dátuma (visszavonás után N nap) | GDPR |
| `status` | Enum | igen | pending/revoked/archived/not_applicable | - |
| `is_blocking_clearance` | Boolean | igen | Blokkolja-e a végső elszámolást | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.7 KnowledgeHandover (Tudásátadás - offboarding specifikus)

Kilépő munkatárs tudásának dokumentálása és átadása.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `offboarding_case_id` | UUID | igen | Melyik kilépési esethez | OffboardingCase |
| `employment_id` | UUID | igen | Kilépő munkatárs | employment-entity.md |
| `knowledge_transfer_plan_id` | UUID | nem | Kapcsolódó tudásátadási terv (ha kulcspozíció) | succession-planning-entity.md KnowledgeTransferPlan |
| `handover_type` | Enum | igen | documentation/training/shadowing/meeting/email/transition_period | Átadás módja |
| `knowledge_area` | String | igen | Tudásterület (pl. "Customer X relationship", "Budget process") | - |
| `knowledge_category` | Enum | igen | processes/relationships/technical/institutional/ongoing_projects/tools_systems | Kategória |
| `description` | Text | igen | Részletes leírás | - |
| `criticality` | Enum | igen | critical/high/medium/low | Kritikusság |
| `recipient_employment_id` | UUID | nem | Átvevő munkatárs (successor vagy team member) | employment-entity.md |
| `recipient_team` | String | nem | Átvevő csapat (ha nem konkrét személy) | - |
| `documentation_required` | Boolean | igen | Dokumentálás szükséges-e | - |
| `documentation_completed` | Boolean | nem | Dokumentálás megtörtént-e | - |
| `documentation_location` | String | nem | Dokumentáció helye (wiki, SharePoint) | - |
| `documentation_document_id` | UUID | nem | Dokumentum | document-entity.md |
| `handover_session_scheduled` | Boolean | nem | Átadó meeting ütemezve-e | - |
| `handover_session_date` | Date | nem | Meeting dátuma | - |
| `handover_session_attendees` | JSON | nem | Résztvevők | - |
| `handover_session_completed` | Boolean | nem | Meeting megtörtént-e | - |
| `status` | Enum | igen | pending/in_progress/completed/not_required | - |
| `completion_date` | Date | nem | Teljesítés dátuma | - |
| `effectiveness_rating` | Enum | nem | effective/partially_effective/ineffective | Utólagos értékelés |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.8 FinalClearance (Végső elszámolás)

Kilépő munkatárs végső pénzügyi és adminisztratív elszámolása.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `offboarding_case_id` | UUID | igen | Melyik kilépési esethez | OffboardingCase |
| `employment_id` | UUID | igen | Munkatárs (denormalizált) | employment-entity.md |
| `clearance_date` | Date | igen | Elszámolás dátuma | - |
| `final_pay_date` | Date | igen | Végső fizetés dátuma | - |
| `final_pay_period_start` | Date | igen | Utolsó fizetési időszak kezdete | - |
| `final_pay_period_end` | Date | igen | Utolsó fizetési időszak vége (last_working_day) | - |
| `base_salary_amount` | Decimal | igen | Alapbér végkielégítésben (Ft) | - |
| `unused_vacation_days` | Decimal | nem | Fel nem vett szabadság (nap) | Mt. 122. § |
| `unused_vacation_payout` | Decimal | nem | Szabadság megváltás (Ft) | Mt. 122. § |
| `severance_pay_eligible` | Boolean | igen | Végkielégítésre jogosult-e | Mt. 77. § (3 év feletti jogviszony) |
| `years_of_service` | Decimal | számított | Szolgálati idő (év) | - |
| `severance_pay_months` | Integer | nem | Végkielégítés hónapjai (1-6 hónap) | Mt. 77. §, Kjt. 37. §, Kit. 67. § |
| `severance_pay_amount` | Decimal | nem | Végkielégítés összege (Ft) | - |
| `prorated_bonus` | Decimal | nem | Arányosított bónusz (Ft) | - |
| `other_benefits_payout` | Decimal | nem | Egyéb juttatások kifizetése | - |
| `outstanding_loans` | Decimal | nem | Fennálló kölcsönök (Ft) | - |
| `outstanding_advances` | Decimal | nem | Előlegek (Ft) | - |
| `asset_replacement_cost` | Decimal | nem | Eszköz pótlási költség (Ft) | AssetReturn |
| `other_deductions` | Decimal | nem | Egyéb levonások (Ft) | - |
| `deductions_detail` | Text | nem | Levonások részletezése | - |
| `gross_final_payment` | Decimal | számított | Bruttó végső kifizetés (Ft) | - |
| `tax_deductions` | Decimal | nem | Adólevonások (Ft) | Szja tv. |
| `social_security_deductions` | Decimal | nem | TB járulékok (Ft) | Tbj. |
| `net_final_payment` | Decimal | számított | Nettó végső kifizetés (Ft) | - |
| `payment_method` | Enum | igen | bank_transfer/check/cash | - |
| `payment_bank_account` | String | nem | Bankszámlaszám | - |
| `payment_completed` | Boolean | igen | Kifizetés megtörtént-e | - |
| `payment_completion_date` | Date | nem | Kifizetés dátuma | - |
| `work_certificate_issued` | Boolean | igen | Munkáltatói igazolás kiadva-e | Mt. 81. § |
| `work_certificate_document_id` | UUID | nem | Munkáltatói igazolás | document-entity.md |
| `tax_certificate_issued` | Boolean | igen | Adóigazolás kiadva-e | - |
| `tax_certificate_document_id` | UUID | nem | Adóigazolás | document-entity.md |
| `non_compete_agreement_signed` | Boolean | nem | Versenytilalmi megállapodás aláírva | Mt. 228-229. § |
| `non_compete_document_id` | UUID | nem | Versenytilalmi megállapodás | document-entity.md |
| `confidentiality_agreement_signed` | Boolean | igen | Titoktartási nyilatkozat aláírva | - |
| `confidentiality_document_id` | UUID | nem | Titoktartási nyilatkozat | document-entity.md |
| `clearance_status` | Enum | igen | pending/in_progress/approved/completed | - |
| `approved_by_id` | UUID | nem | Jóváhagyó (HR Manager / Vezető) | person-entity.md |
| `approved_date` | Date | nem | Jóváhagyás dátuma | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

## 3. GDPR kategorizáció és adatkezelés

### Offboarding adatok kezelése

#### Jogalap
- **GDPR Art. 6(1)(b)**: szerződés teljesítése (munkaviszony lezárása)
- **GDPR Art. 6(1)(c)**: jogi kötelezettség (Mt. igazolás kiadás, végkielégítés)
- **GDPR Art. 6(1)(f)**: jogos érdek (exit interview, alumni network)

#### Adatkategóriák
- **Általános személyes adat**: kilépési okok, exit interview válaszok, eszköz visszavétel
- **Pénzügyi adat**: végkielégítés, szabadság megváltás, levonások
- **Különösen bizalmas**: exit interview részletes visszajelzések (zaklatás, diszkrimináció)

#### Adatmegőrzési idők
- **OffboardingCase**: jogviszony megszűnése + 3 év (Mt. 287. § elévülés)
- **ExitInterview**: jogviszony megszűnése + 3 év (aggregált riportok hosszabb ideig név nélkül)
- **FinalClearance**: jogviszony megszűnése + 8 év (Számviteli tv. 169. §)
- **AssetReturn**: eszköz visszaadás + 3 év
- **AccessRevocation**: hozzáférés visszavonás + 1 év (IT audit trail)
- **KnowledgeHandover**: tudásátadás + 5 év (szervezeti tudás)

#### Adattörlési kötelezettségek
- **Email fiók**: jogviszony megszűnése + 30-90 nap (email forwarding után törlés)
- **Active Directory account**: jogviszony megszűnése + 30 nap (archívum után törlés)
- **Személyes drive**: jogviszony megszűnése előtt backup, majd törlés
- **GDPR Art. 17**: törléshez való jog (jogszabályi megőrzési idők lejárta után)

### Hozzáférési jogok
- **Munkatárs**: saját offboarding case (feladatlista, exit interview időpont)
- **Közvetlen vezető**: beosztottak offboarding case-ei, exit interview összefoglalók (aggregált)
- **HR**: teljes hozzáférés (adminisztráció, exit interview, clearance)
- **IT**: AccessRevocation, AssetReturn (IT eszközök)
- **Finance**: FinalClearance (pénzügyi elszámolás)
- **Facility**: AssetReturn (belépőkártya, kulcsok), AccessRevocation (épület)

## 4. Hozzáférési szintek

| Entitás | HR Admin | Manager | IT Admin | Finance | Facility | Employee |
|---|---|---|---|---|---|---|
| OffboardingCase | CRUD | Read (team) | Read (ha IT task) | Read (ha Finance task) | Read (ha Facility task) | Read (saját) |
| OffboardingChecklistTemplate | CRUD | Read | Read | Read | Read | - |
| OffboardingTask | CRUD | CRUD (team tasks) | CRUD (IT tasks) | CRUD (Finance tasks) | CRUD (Facility tasks) | Read (saját) |
| ExitInterview | CRUD | Read (aggregált) | - | - | - | CRUD (saját) |
| AssetReturn | CRUD | Read (team) | CRUD (IT assets) | Read | CRUD (facility assets) | Read (saját) |
| AccessRevocation | CRUD | Read (team) | CRUD | - | CRUD (building) | Read (saját) |
| KnowledgeHandover | CRUD | CRUD (team) | - | - | - | CRUD (saját) |
| FinalClearance | CRUD | Read (team) | - | CRUD | - | Read (saját) |

## 5. Validációs szabályok

### OffboardingCase
- **Felmondási idő számítás**: notice_period_days automatikus számítás employment tenure alapján (Mt. 67-72. §)
- **Utolsó munkanap validáció**: last_working_day >= resignation_date + notice_period_days (ha nem waived)
- **Completion feltétel**: status=completed csak ha checklist_completion_percentage=100 és final_clearance_completed=true

### OffboardingTask
- **Due date calculation**: due_date = OffboardingCase.last_working_day + due_date_offset_days
- **Dependency check**: status=pending ha depends_on_task_id.status != completed
- **Overdue flagging**: is_overdue=true ha due_date < current_date és status != completed
- **Emlékeztetők**:
  - 7 nappal határidő előtt: 1. emlékeztető
  - 3 nappal határidő előtt: 2. emlékeztető
  - Határidő napján: 3. emlékeztető
  - Lejárat után naponként: ismétlő emlékeztetők

### ExitInterview
- **Interview timing**: interview_date ideálisan last_working_day - 7 nap (elegendő idő feedback-re reagálni)
- **Confidentiality**: is_confidential=true esetén manager/team feedback aggregált, név nélkül

### AssetReturn
- **Return due date**: return_due_date <= last_working_day
- **Blocking clearance**: is_blocking_clearance=true critical eszközöknél (laptop, phone, access card)
- **Data wipe kötelező**: requires_data_wipe=true minden IT eszköznél (laptop, phone, tablet)

### AccessRevocation
- **Revocation priority**:
  - immediate: kritikus rendszerek (admin accounts, production database)
  - before_last_day: általános rendszerek (email, network, VPN)
  - on_last_day: épület belépés
  - after_last_day: email forwarding setup után email account
- **Email forwarding duration**: email_forwarding_duration_days max. 90 nap

### FinalClearance
- **Végkielégítés jogosultság**: severance_pay_eligible=true ha years_of_service >= 3
- **Végkielégítés számítás** (Mt. 77. §):
  - 3 év: 1 havi alapbér
  - 5 év: 2 havi alapbér
  - 10 év: 3 havi alapbér
  - 15 év: 4 havi alapbér
  - 20 év: 5 havi alapbér
  - 25 év: 6 havi alapbér
- **Közszféra végkielégítés**: Kjt., Kit., Kttv. eltérő szabályok (2-8 havi illetmény)
- **Fel nem vett szabadság**: unused_vacation_payout = unused_vacation_days × (base_salary / 30)
- **Gross final payment**: base_salary + unused_vacation_payout + severance_pay + bonuses - deductions

## 6. Integrációs pontok

### Külső rendszerek
- **Active Directory / LDAP**: hozzáférések automatikus visszavonása
- **Email rendszer (Exchange, Gmail)**: email forwarding, fiók archiválás
- **VPN / Network Access**: hozzáférések visszavonása
- **Badge/Access Control rendszer**: belépőkártya deaktiválása
- **IT Asset Management**: eszközök visszavétel nyilvántartása
- **Survey tools (SurveyMonkey, Qualtrics)**: exit interview survey integráció
- **Payroll system**: végső fizetés számítása
- **Alumni platform**: volt munkavállalói hálózat

### Belső entitások
- **Employment**: jogviszony status=terminated, termination_date
- **Position**: pozíció status=vacant (ha nincs utód)
- **Succession Planning**: KnowledgeTransferPlan, successor információk
- **Leave**: unused_vacation_days számítás
- **Compensation**: final pay calculation
- **Document**: munkáltatói igazolás, adóigazolás, versenytilalmi megállapodás
- **TimeTracking**: utolsó munkanap munkaidő nyilvántartás
- **Performance Review**: utolsó értékelés (pro-rated éves értékelés)

## 7. Kapcsolódó entitások

- **employment-entity.md**: Employment (jogviszony megszűnés)
- **person-entity.md**: Person (HR contact, manager, IT admin)
- **position-entity.md**: Position (pozíció betöltetlen státusza)
- **organization-entity.md**: OrgUnit (szervezeti egység átszervezés)
- **document-entity.md**: Document (igazolások, megállapodások)
- **succession-planning-entity.md**: KnowledgeTransferPlan, SuccessorCandidate
- **compensation-entity.md**: Compensation (végkielégítés, szabadság megváltás)
- **leave-entity.md**: LeaveRecord (fel nem vett szabadság)
- **timetracking-entity.md**: DailyTimeRecord (utolsó munkanap)
- **performance-review-entity.md**: PerformanceReview (utolsó értékelés)

## 8. Üzleti folyamatok

### 8.1 Önkéntes felmondás (Employee Resignation)

1. **Felmondás bejelentése**: munkatárs benyújtja felmondó levelét
2. **OffboardingCase létrehozása**:
   - termination_type=resignation
   - resignation_date, notice_period_days, last_working_day számítás
   - Checklist template alkalmazása (position/org unit alapján)
3. **Manager értesítés**: közvetlen vezető email notification
4. **HR intake meeting**: HR interjú a kilépési okról (informális)
5. **OffboardingTask-ok generálása**: checklist alapján task-ok létrehozása
6. **Exit interview ütemezés**: ExitInterview scheduling (last_working_day - 7 nap)
7. **Utód azonosítás**: successor_employment_id (ha applicable, succession planning)
8. **Tudásátadás**: KnowledgeHandover task-ok
9. **Eszköz visszavétel**: AssetReturn task-ok (laptop, phone, access card)
10. **Hozzáférések visszavonása**: AccessRevocation task-ok (email, network, VPN)
11. **Végső elszámolás**: FinalClearance preparation
12. **Utolsó munkanap**: farewell, exit interview (ha nem korábban)
13. **Clearance lezárás**: FinalClearance approval, final payment
14. **Alumni opt-in**: alumni_network_opt_in (ha igen, adatok megőrzése contact céljából)

### 8.2 Munkáltatói felmondás (Employer Termination)

1. **Döntés**: leadership/HR döntés a felmondásról (performance, redundancy)
2. **Jogi review**: HR + legal counsel felülvizsgálat (indokolás, dokumentáció)
3. **OffboardingCase létrehozása**:
   - termination_type=termination_by_employer
   - termination_reason (performance_issues, redundancy, restructuring)
4. **Termination meeting**: HR + manager + munkatárs (felmondó levél átadása)
5. **Azonnali hozzáférés visszavonás** (bizonyos esetekben):
   - AccessRevocation immediate priority (admin accounts, critical systems)
   - Escorted exit (biztonsági kockázat esetén)
6. **Felmondási idő vagy felmentés**: notice_period_days vs. garden leave
7. **OffboardingTask-ok**: checklist alapján
8. **Exit interview (opcionális)**: nem kötelező munkáltatói felmondásnál
9. **Végkielégítés számítás**: FinalClearance.severance_pay (ha jogosult, 3+ év)
10. **Settlement agreement**: végkielégítési megállapodás (mutual release)
11. **Clearance lezárás**: final payment
12. **Rehire eligibility**: is_rehire_eligible=false (általában)

### 8.3 Nyugdíjazás (Retirement)

1. **Nyugdíjazási szándék bejelentése**: 3-6 hónappal korábban (best practice)
2. **OffboardingCase létrehozása**: termination_type=retirement
3. **Succession planning aktiválás**: successor azonosítás (ha kulcspozíció)
4. **Hosszú tudásátadási periódus**: KnowledgeHandover (4-12 hét overlap)
5. **Búcsúrendezvény**: retirement celebration
6. **Exit interview**: career retrospective, organizational feedback
7. **Alumni program**: nyugdíjas alumni network opt-in
8. **Clearance**: final payment, pension information
9. **Rehire eligibility**: is_rehire_eligible=true (consultant kapacitásban)

### 8.4 Azonnali hatályú felmondás (Immediate Dismissal)

1. **Súlyos kötelezettségszegés**: Mt. 78. § (theft, fraud, violence)
2. **OffboardingCase létrehozása**: termination_type=immediate_dismissal
3. **Azonnali hozzáférés visszavonás**: AccessRevocation immediate (összes rendszer)
4. **Azonnali eszköz visszavétel**: AssetReturn immediate (escorted exit)
5. **Investigating documentation**: munkáltatói intézkedés dokumentálása
6. **NO exit interview**: nem alkalmazható
7. **NO végkielégítés**: Mt. 78. § (súlyos kötelezettségszegés esetén)
8. **Legal review**: HR legal counsel review (munkaügyi per kockázat)
9. **Clearance**: final payment (only base salary, no severance)
10. **Rehire eligibility**: is_rehire_eligible=false

### 8.5 Email és adatok átmeneti megőrzése

1. **Email forwarding setup**: AccessRevocation.email_forwarding=true
   - Időtartam: 30-90 nap
   - Címzett: successor vagy manager
2. **Email auto-reply**: "X has left the company, please contact Y"
3. **Personal drive backup**: data_backup_required=true (manager approval)
4. **Email archívum**: email fiók archiválása (compliance, litigation hold)
5. **Fiók törlés**: account_deletion_date = last_working_day + 90 nap
6. **GDPR compliance**: személyes adatok törlése (kivéve jogszabályi megőrzés)

## 9. Reporting és analitika

### Turnover (fluktuáció) jelentések
- **Önkéntes távozási arány**: resignation / total employees % (évente)
- **Önkéntes vs. nem önkéntes**: resignation vs. termination_by_employer arány
- **Regrettable loss rate**: is_regrettable_loss=true / total turnover %
- **Nyugdíjazások**: retirement count és előrejelzés
- **Fluktuáció szervezeti egységenként**: organization_unit bontás
- **Fluktuáció pozíciónként**: position bontás

### Exit interview insights
- **Primary leaving reasons**: termination_reason megoszlás
- **eNPS (Employee Net Promoter Score)**: would_recommend_company alapján
- **Boomerang hire potential**: likelihood_to_return=very_likely/likely %
- **Satisfaction scores**: átlagos satisfaction rating kategóriánként
- **Compensation factor**: compensation_factor=primary_reason/major_factor %
- **Manager feedback trends**: manager_feedback sentiment analysis

### Offboarding efficiency
- **Átlagos offboarding időtartam**: resignation_date → completion_date napok
- **Checklist completion rate**: checklist_completion_percentage átlag
- **Lejárt feladatok**: OffboardingTask is_overdue=true count
- **Asset recovery rate**: AssetReturn status=returned / total %
- **Knowledge transfer effectiveness**: KnowledgeHandover effectiveness_rating

### Költségek
- **Végkielégítések összesen**: FinalClearance.severance_pay sum (évente)
- **Fel nem vett szabadság költsége**: unused_vacation_payout sum
- **Eszköz veszteségek**: AssetReturn replacement_cost sum (lost/damaged)
- **Átlagos final payment**: net_final_payment átlag

## 10. Nyitott kérdések és jövőbeli fejlesztések

### Technológiai kérdések
1. **IT integráció mélysége**: Active Directory, email, VPN automatikus hozzáférés visszavonás API-n keresztül?
2. **Asset Management integráció**: eszköz nyilvántartó rendszer (ServiceNow, Jira) integráció?
3. **Survey platform**: exit interview survey külső rendszerrel (SurveyMonkey, Qualtrics)?
4. **Alumni platform**: volt munkavállalói hálózat (LinkedIn csoport, egyedi platform)?

### Funkcionális fejlesztések
1. **Boomerang hiring program**: rehire-eligible alumni targeting, simplified onboarding
2. **Stay interview**: kilépés megelőzése (proaktív interjú értékes munkatársakkal)
3. **Counter-offer process**: ellenajanlatkezés folyamat (ha regrettable loss)
4. **Offboarding analytics dashboard**: valós idejű turnover dashboard vezetőknek
5. **Automated email forwarding**: automatikus email forwarding setup (IT integráció)

### Compliance és jogi
1. **GDPR right to erasure**: automatikus adattörlés jogszabályi megőrzési idők lejárta után
2. **Email retention policy**: mennyi ideig őrizzük az email archívumot (litigation hold)?
3. **Reference check protocol**: reference_contact_consent alapján automatikus referencia kezelés
4. **Non-compete enforcement**: versenytilalmi megállapodás monitoring (új munkáltató)

### Integrációk
1. **Active Directory API**: automatikus account disable/delete
2. **Badge system API**: belépőkártya deaktiválás
3. **Payroll system**: final pay automatikus számítás és integráció
4. **Background check vendors**: rehire background check (ha applicable)

### AI és automatizálás
1. **Exit interview sentiment analysis**: szöveges válaszok NLP analízise (concerns flagging)
2. **Flight risk prediction**: ML modell kilépési kockázat előrejelzésre (megelőző intézkedések)
3. **Retention recommendation engine**: AI javaslatok counter-offer esetén
4. **Chatbot**: offboarding FAQ, feladatlista tracking (munkatárs és manager)

---

**Megjegyzések:**
- **Exit interview kritikus**: a kilépési okok feltárása segít a retention javításában
- **Regrettable loss**: értékes munkatárs távozása → counter-offer, retention stratégia
- **Rehire eligibility**: "boomerang employee" trend növekszik, érdemes alumni hálózat
- **Knowledge transfer**: kritikus pozícióknál (succession planning kapcsolat)
- **Email forwarding**: 30-90 napos átmeneti időszak (ügyfelek, projektek átadása)
- **Asset recovery**: kritikus (laptop, phone) eszközök blokkolják a clearance-t
- **Final clearance**: pénzügyi elszámolás utolsó lépés (minden task completed után)
- **GDPR**: adattörlési kötelezettség vs. jogszabályi megőrzési idők egyensúlya
- **Confidentiality**: exit interview bizalmas, manager feedback aggregált
- **Alumni network**: opt-in alapú, GDPR-kompatibilis kapcsolattartás
