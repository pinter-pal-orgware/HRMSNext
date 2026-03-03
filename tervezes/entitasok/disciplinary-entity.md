# Disciplinary (Fegyelmi és Panaszkezelés) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer
> **Verzió:** 0.1 – első tervezet
> **Utolsó frissítés:** 2025. február
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md, employment-entity.md, organization-entity.md, position-entity.md, performance-review-entity.md, document-entity.md, timetracking-entity.md, compensation-entity.md

---

## 1. Az entitás célja és hatóköre

A fegyelmi és panaszkezelési entitások biztosítják a munkajogi kötelezettségszegések, fegyelmi eljárások, panaszok és kivizsgálások nyomon követését. Különös figyelmet fordítunk a közszféra specifikus fegyelmi eljárásaira (Kjt., Kit., Kttv.), valamint a magánszféra munkajogi szabályaira (Mt.).

### Hatókör

A fegyelmi és panaszkezelési entitások lefedik:
- **Fegyelmi ügyek**: kötelezettségszegések, fegyelmi eljárások, szankciók
- **Panaszkezelés**: munkatársak panaszai, bejelentések (pl. zaklatás, diszkrimináció, munkakörülmények)
- **Kivizsgálások**: tényfeltáró eljárások, bizonyítékok gyűjtése, tanúmeghallgatások
- **Fegyelmi büntetések**: írásbeli figyelmeztetés, megrovás, pénzbírság, felfüggesztés, jogviszony megszüntetés
- **Fellebbezések**: jogorvoslati eljárások
- **Etikai szabályok megsértése**: közszféra etikai kódex, összeférhetetlenség
- **Munkahelyi visszaélések bejelentése (whistleblowing)**: védett bejelentő rendszer

### Foglalkoztatási típus specifikus követelmények

#### Magánszektor (Mt.)
- **Munkáltatói intézkedési jog**: Mt. 52-54. § alapján munkáltatói intézkedés
- **Azonnali hatályú felmondás**: Mt. 78. § alapján súlyos kötelezettségszegés esetén
- **Munkaügyi vita**: Munkaügyi Bíróság hatásköre
- **Írásbeli figyelmeztetés**: dokumentálási kötelezettség
- **Rendes felmondás**: Mt. 65-66. § alapján gyakran ismétlődő kötelezettségszegés

#### Köztisztviselők (Kjt.)
- **Fegyelmi eljárás**: Kjt. 77-87. § alapján KÖTELEZŐ formális eljárás
- **Fegyelmi vétségek**: Kjt. 77. § alapján meghatározott körülmények
  - Kötelességszegés szándékosan vagy gondatlanságból
  - Köztisztviselői jogviszonyból eredő kötelezettség megszegése
- **Fegyelmi büntetések**: Kjt. 78. §
  1. Megrovás
  2. Illetménycsökkentés (max 1 év, max 30%)
  3. Besorolás visszaminősítése
  4. Felmentés fegyelmi úton
- **Fegyelmi eljárás határideje**: Kjt. 82. § - tudomásszerzéstől 6 hónap, elkövetéstől 1 év
- **Fegyelmi bizottság**: Kjt. 80. § alapján létrehozandó
- **Védekezési jog**: meghallgatás kötelező, védő meghatalmazása

#### Kormánytisztviselők (Kttv.)
- **Fegyelmi eljárás**: Kttv. 134-145. § részletes szabályozás
- **Fegyelmi vétségek**: Kttv. 134. § alapján
  - Szolgálati kötelezettség megszegése
  - Etikai normák megsértése
  - Összeférhetetlenség
- **Fegyelmi büntetések**: Kttv. 135. §
  1. Figyelmeztetés
  2. Megrovás
  3. Illetmény 3-50%-ának elvonása (max 1 év)
  4. Besorolás visszaminősítése
  5. Felmentés fegyelmi úton
- **Fegyelmi eljárás határideje**: Kttv. 137. § - tudomásszerzéstől 6 hónap, elkövetéstől 3 év
- **Fegyelmi tanács**: Kttv. 139. § háromtagú fegyelmi tanács

#### Közszolgálati tisztviselők (Kit.)
- **Fegyelmi eljárás**: Kit. 123-132. § szabályozás
- **Fegyelmi vétségek**: Kit. 123. § alapján
- **Fegyelmi büntetések**: Kit. 124. §
  1. Megrovás
  2. Pénzbírság (1 havi illetmény max 50%-a)
  3. várakozási idő meghosszabbítása (max 1 év)
  4. Besorolás visszaminősítése
  5. Felmentés fegyelmi úton
- **Fegyelmi eljárás határideje**: Kit. 127. § - tudomásszerzéstől 6 hónap, elkövetéstől 3 év
- **Fegyelmi bizottság**: Kit. 126. § legalább háromtagú

#### Egyéb közszféra (Púétv., Eszjtv., Küt.)
- **Pedagógusok**: Púétv. 72. § fegyelmi eljárás (megrovás, megintés, eltiltás, felmentés)
- **Egészségügyi dolgozók**: Eszjtv. szakmai kamarai fegyelmi eljárás (MOK, MKEK)
- **Ügyészek**: Üszt. fegyelmi szabályok
- **Bírák**: Bszi. fegyelmi szabályok (Országos Bírói Tanács hatásköre)

## 2. Entitásstruktúra

### 2.1 DisciplinaryCase (Fegyelmi ügy)

Fegyelmi eljárás nyilvántartása.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `case_number` | String | igen | Egyedi ügyszám (pl. "DISC-2026-001") | - |
| `employment_id` | UUID | igen | Melyik munkatárs ellen | employment-entity.md |
| `employment_type` | Enum | igen | mt/kjt/kttv/kit/puetv/eszjtv/kut | Jogviszony típusa (eljárási szabályok) |
| `case_type` | Enum | igen | disciplinary/grievance/investigation/ethics_violation | Ügy típusa |
| `incident_date` | Date | nem | Feltételezett esemény dátuma | - |
| `reported_date` | Date | igen | Bejelentés dátuma | - |
| `reported_by_id` | UUID | nem | Ki jelentette be (lehet névtelen) | person-entity.md |
| `is_anonymous_report` | Boolean | igen | Névtelen bejelentés-e | Whistleblowing |
| `incident_description` | Text | igen | Esemény leírása | - |
| `alleged_violation` | Text | igen | Feltételezett szabálysértés, jogszabályhely | Mt., Kjt., Kttv., Kit. |
| `severity` | Enum | igen | minor/moderate/serious/critical | Súlyosság |
| `case_status` | Enum | igen | reported/under_review/investigation/hearing/decision/closed/appealed | Státusz |
| `assigned_investigator_id` | UUID | nem | Felelős kivizsgáló | person-entity.md |
| `assigned_date` | DateTime | nem | Kijelölés időpontja | - |
| `investigation_start_date` | Date | nem | Vizsgálat kezdete | - |
| `investigation_end_date` | Date | nem | Vizsgálat lezárása | - |
| `hearing_required` | Boolean | igen | Meghallgatás szükséges-e | Kjt. 81. §, Kttv. 138. §, Kit. 128. § |
| `statute_of_limitations_date` | Date | nem | Elévülési határidő | Kjt. 82. §, Kttv. 137. §, Kit. 127. § |
| `decision_date` | Date | nem | Határozat dátuma | - |
| `decision_maker_id` | UUID | nem | Határozatot hozó (vezető, bizottság elnök) | person-entity.md |
| `decision_outcome` | Enum | nem | no_violation/violation_confirmed/dismissed | Eredmény |
| `decision_summary` | Text | nem | Határozat indokolása | - |
| `decision_document_id` | UUID | nem | Határozat dokumentum | document-entity.md |
| `closure_date` | Date | nem | Ügy lezárása | - |
| `closure_reason` | Text | nem | Lezárás oka (ha dismissed) | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.2 Investigation (Vizsgálat)

Tényfeltáró eljárás részletei.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `disciplinary_case_id` | UUID | igen | Melyik fegyelmi ügy | DisciplinaryCase |
| `investigation_type` | Enum | igen | internal/external/third_party | Belső vagy külső vizsgálat |
| `lead_investigator_id` | UUID | igen | Fő vizsgáló | person-entity.md |
| `investigation_team_ids` | JSON | nem | Vizsgálati team (Person UUID-k) | person-entity.md |
| `start_date` | Date | igen | Vizsgálat kezdete | - |
| `planned_end_date` | Date | nem | Tervezett befejezés | - |
| `actual_end_date` | Date | nem | Tényleges befejezés | - |
| `status` | Enum | igen | initiated/in_progress/completed/suspended | - |
| `suspension_reason` | Text | nem | Felfüggesztés oka | - |
| `findings` | Text | nem | Megállapítások | - |
| `evidence_summary` | Text | nem | Bizonyítékok összefoglalása | - |
| `conclusion` | Enum | nem | violation_confirmed/violation_not_confirmed/inconclusive | Következtetés |
| `recommendation` | Text | nem | Javaslat (fegyelmi büntetés, intézkedés) | - |
| `investigation_report_id` | UUID | nem | Vizsgálati jelentés | document-entity.md |
| `confidentiality_level` | Enum | igen | public/internal/confidential/strictly_confidential | Bizalmasság |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.3 Evidence (Bizonyíték)

Vizsgálat során gyűjtött bizonyítékok.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `investigation_id` | UUID | igen | Melyik vizsgálathoz | Investigation |
| `evidence_type` | Enum | igen | document/email/chat/photo/video/audio/witness_statement/physical | Bizonyíték típusa |
| `evidence_description` | Text | igen | Leírás | - |
| `collected_date` | Date | igen | Gyűjtés dátuma | - |
| `collected_by_id` | UUID | igen | Ki gyűjtötte | person-entity.md |
| `document_id` | UUID | nem | Kapcsolódó dokumentum | document-entity.md |
| `chain_of_custody` | Text | nem | Őrzési lánc (ki, mikor kezelte) | - |
| `is_admissible` | Boolean | nem | Felhasználható-e (jogi értékelés) | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.4 Witness (Tanú)

Tanúmeghallgatások nyilvántartása.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `investigation_id` | UUID | igen | Melyik vizsgálathoz | Investigation |
| `witness_person_id` | UUID | nem | Tanú személye (ha munkatárs) | person-entity.md |
| `witness_external_name` | String | nem | Tanú neve (ha külső) | - |
| `witness_role` | Enum | igen | eyewitness/character_witness/expert | Tanú típusa |
| `interview_date` | DateTime | nem | Meghallgatás időpontja | - |
| `interviewed_by_id` | UUID | nem | Ki hallgatta meg | person-entity.md |
| `interview_location` | String | nem | Helyszín | - |
| `statement_summary` | Text | nem | Vallomás összefoglalása | - |
| `statement_document_id` | UUID | nem | Írásos vallomás | document-entity.md |
| `is_statement_signed` | Boolean | nem | Aláírta-e a vallomást | - |
| `credibility_assessment` | Text | nem | Hitelesség értékelése | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.5 DisciplinaryHearing (Fegyelmi meghallgatás)

Fegyelmi eljárás során tartott meghallgatások.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `disciplinary_case_id` | UUID | igen | Melyik fegyelmi ügy | DisciplinaryCase |
| `hearing_date` | DateTime | igen | Meghallgatás időpontja | - |
| `hearing_type` | Enum | igen | preliminary/main_hearing/appeal_hearing | Típus |
| `location` | String | igen | Helyszín | - |
| `presiding_officer_id` | UUID | igen | Eljáró személy (vezető, bizottsági elnök) | person-entity.md |
| `panel_members_ids` | JSON | nem | Fegyelmi bizottság tagjai (Kjt./Kttv./Kit.) | person-entity.md UUID-k |
| `employee_present` | Boolean | igen | Munkatárs jelen volt-e | Kjt. 81. §, Kttv. 138. § |
| `employee_representative_id` | UUID | nem | Munkatárs képviselője (védő, szakszervezet) | person-entity.md |
| `employee_statement` | Text | nem | Munkatárs védekezése | - |
| `witnesses_heard_ids` | JSON | nem | Meghallgatott tanúk | Witness UUID-k |
| `evidence_presented_ids` | JSON | nem | Bemutatott bizonyítékok | Evidence UUID-k |
| `minutes_document_id` | UUID | nem | Jegyzőkönyv | document-entity.md |
| `is_minutes_signed` | Boolean | nem | Jegyzőkönyv aláírva-e | - |
| `outcome_summary` | Text | nem | Meghallgatás eredménye | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.6 DisciplinaryAction (Fegyelmi büntetés)

Kiszabott fegyelmi szankció.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `disciplinary_case_id` | UUID | igen | Melyik fegyelmi ügy | DisciplinaryCase |
| `employment_id` | UUID | igen | Melyik munkatárs | employment-entity.md |
| `employment_type` | Enum | igen | mt/kjt/kttv/kit/puetv/eszjtv/kut | Jogviszony típusa |
| `action_type` | Enum | igen | verbal_warning/written_warning/reprimand/salary_reduction/suspension/demotion/termination | Szankció típusa |
| `kjt_action_type` | Enum | nem | reprimand/salary_reduction/demotion/termination | Kjt. 78. § |
| `kttv_action_type` | Enum | nem | warning/reprimand/salary_reduction/demotion/termination | Kttv. 135. § |
| `kit_action_type` | Enum | nem | reprimand/fine/waiting_period_extension/demotion/termination | Kit. 124. § |
| `legal_basis` | String | igen | Jogszabályi hivatkozás | Mt., Kjt., Kttv., Kit. |
| `action_date` | Date | igen | Határozat kelte | - |
| `effective_date` | Date | igen | Hatálybalépés dátuma | - |
| `decision_maker_id` | UUID | igen | Határozatot hozó | person-entity.md |
| `description` | Text | igen | Részletes indokolás | - |
| `salary_reduction_percentage` | Decimal | nem | Illetménycsökkentés mértéke (%) | Kjt. 78. § max 30%, Kttv. 135. § max 50% |
| `salary_reduction_duration_months` | Integer | nem | Illetménycsökkentés időtartama (hónap) | Kjt./Kttv./Kit. max 12 hónap |
| `fine_amount` | Decimal | nem | Pénzbírság összege (Ft) | Kit. 124. § max 1 havi illetmény 50%-a |
| `suspension_start_date` | Date | nem | Felfüggesztés kezdete | - |
| `suspension_end_date` | Date | nem | Felfüggesztés vége | - |
| `suspension_paid` | Boolean | nem | Fizetett felfüggesztés-e | Mt. 52. § |
| `termination_type` | Enum | nem | immediate/notice_period | Azonnali vagy felmondási idős |
| `termination_notice_days` | Integer | nem | Felmondási idő (nap) | Mt. 66. §, Kjt. 63. § |
| `demotion_from_position_id` | UUID | nem | Korábbi pozíció | position-entity.md |
| `demotion_to_position_id` | UUID | nem | Új pozíció | position-entity.md |
| `action_document_id` | UUID | nem | Határozat dokumentum | document-entity.md |
| `is_appealable` | Boolean | igen | Fellebbezthető-e | - |
| `appeal_deadline` | Date | nem | Fellebbezési határidő | Mt., Kjt., Kttv., Kit. |
| `status` | Enum | igen | pending/active/completed/appealed/overturned | Státusz |
| `completion_date` | Date | nem | Teljesítés dátuma | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | igen | Létrehozó | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | igen | Módosító | Audit trail |

### 2.7 Appeal (Fellebbezés)

Jogorvoslati eljárások.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `disciplinary_action_id` | UUID | igen | Melyik fegyelmi büntetés ellen | DisciplinaryAction |
| `employment_id` | UUID | igen | Ki fellebbez | employment-entity.md |
| `appeal_date` | Date | igen | Fellebbezés beadás dátuma | - |
| `appeal_grounds` | Text | igen | Fellebbezési indokok | - |
| `appeal_document_id` | UUID | nem | Fellebbezés dokumentuma | document-entity.md |
| `review_body` | Enum | igen | hr_manager/senior_management/disciplinary_board/labor_court/administrative_court | Felülvizsgáló szerv |
| `review_body_contact_id` | UUID | nem | Felülvizsgáló személy | person-entity.md |
| `hearing_date` | DateTime | nem | Tárgyalás időpontja | - |
| `decision_date` | Date | nem | Határozat dátuma | - |
| `decision_outcome` | Enum | nem | upheld/overturned/modified/remanded | Eredmény |
| `decision_summary` | Text | nem | Határozat indokolása | - |
| `modified_action_id` | UUID | nem | Módosított fegyelmi büntetés (ha modified) | DisciplinaryAction |
| `decision_document_id` | UUID | nem | Határozat dokumentum | document-entity.md |
| `status` | Enum | igen | submitted/under_review/hearing_scheduled/decided/closed | Státusz |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.8 GrievanceCase (Panaszügy)

Munkatársak panaszai, bejelentései (pl. zaklatás, diszkrimináció, munkahelyi konfliktus).

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `case_number` | String | igen | Egyedi ügyszám (pl. "GRIEV-2026-001") | - |
| `complainant_employment_id` | UUID | nem | Panaszos (ha nem névtelen) | employment-entity.md |
| `is_anonymous` | Boolean | igen | Névtelen bejelentés-e | Whistleblowing |
| `grievance_type` | Enum | igen | harassment/discrimination/bullying/unsafe_conditions/pay_dispute/workload/other | Panasz típusa |
| `grievance_category` | Enum | nem | workplace_environment/management/colleague/policy/compensation | Kategória |
| `against_person_id` | UUID | nem | Ki ellen (ha személy ellen) | person-entity.md |
| `against_org_unit_id` | UUID | nem | Melyik szervezeti egység ellen | organization-entity.md |
| `incident_date` | Date | nem | Esemény dátuma | - |
| `reported_date` | Date | igen | Bejelentés dátuma | - |
| `description` | Text | igen | Panasz leírása | - |
| `desired_outcome` | Text | nem | Kívánt megoldás | - |
| `severity` | Enum | igen | low/medium/high/critical | Súlyosság |
| `status` | Enum | igen | submitted/acknowledged/investigating/resolved/closed/escalated | Státusz |
| `assigned_to_id` | UUID | nem | Felelős ügyintéző (HR, compliance officer) | person-entity.md |
| `assigned_date` | DateTime | nem | Kijelölés időpontja | - |
| `acknowledgement_date` | Date | nem | Visszaigazolás dátuma (munkatárs felé) | - |
| `investigation_required` | Boolean | igen | Szükséges-e vizsgálat | - |
| `investigation_id` | UUID | nem | Kapcsolódó vizsgálat | Investigation |
| `resolution_date` | Date | nem | Megoldás dátuma | - |
| `resolution_summary` | Text | nem | Megoldás leírása | - |
| `resolution_document_id` | UUID | nem | Megoldás dokumentum | document-entity.md |
| `complainant_satisfied` | Boolean | nem | Panaszos elégedett-e a megoldással | - |
| `follow_up_required` | Boolean | nem | Követés szükséges-e | - |
| `follow_up_date` | Date | nem | Követés dátuma | - |
| `confidentiality_level` | Enum | igen | internal/confidential/strictly_confidential | Bizalmasság |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `created_by_id` | UUID | nem | Létrehozó (lehet a panaszos vagy HR) | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |
| `updated_by_id` | UUID | nem | Módosító | Audit trail |

### 2.9 DisciplinaryHistory (Fegyelmi előzmények)

Munkatársak fegyelmi nyilvántartása (összesített nézet).

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `employment_id` | UUID | igen | Melyik munkatárs | employment-entity.md |
| `disciplinary_action_id` | UUID | igen | Melyik fegyelmi büntetés | DisciplinaryAction |
| `action_type` | Enum | igen | verbal_warning/written_warning/reprimand/salary_reduction/suspension/demotion/termination | Szankció típusa |
| `action_date` | Date | igen | Határozat kelte | - |
| `expiry_date` | Date | nem | Nyilvántartásból törlés dátuma | Kjt. 87. § (3 év), Mt. gyakorlat |
| `is_active` | Boolean | igen | Aktív-e (figyelembe vehető-e újabb eljárásban) | - |
| `was_appealed` | Boolean | igen | Fellebbezték-e | - |
| `appeal_outcome` | Enum | nem | upheld/overturned/modified | Fellebbezés eredménye |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

### 2.10 EthicsViolation (Etikai szabálysértés)

Közszféra specifikus etikai szabályok megsértése.

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | Elsődleges kulcs | - |
| `employment_id` | UUID | igen | Melyik munkatárs | employment-entity.md |
| `violation_type` | Enum | igen | conflict_of_interest/gift_acceptance/misuse_of_position/confidentiality_breach/political_activity | Etikai szabálysértés típusa |
| `legal_basis` | String | igen | Jogszabályi hivatkozás | Kit. 77-84. §, Kttv. 84-95. §, Kjt. 22-29. § |
| `reported_date` | Date | igen | Bejelentés dátuma | - |
| `incident_date` | Date | nem | Esemény dátuma | - |
| `description` | Text | igen | Leírás | - |
| `disciplinary_case_id` | UUID | nem | Kapcsolódó fegyelmi ügy | DisciplinaryCase |
| `status` | Enum | igen | reported/under_review/confirmed/dismissed | Státusz |
| `resolution_date` | Date | nem | Megoldás dátuma | - |
| `notes` | Text | nem | Megjegyzések | - |
| `created_at` | DateTime | igen | Létrehozás időpontja | Audit trail |
| `updated_at` | DateTime | igen | Módosítás időpontja | Audit trail |

## 3. GDPR kategorizáció és adatkezelés

### Fegyelmi és panaszkezelési adatok

#### Jogalap
- **GDPR Art. 6(1)(c)**: jogi kötelezettség teljesítése (Mt., Kjt., Kttv., Kit. fegyelmi eljárások)
- **GDPR Art. 6(1)(f)**: jogos érdek (munkahelyi panaszok kivizsgálása, munkáltatói védelem)
- **GDPR Art. 88**: munkavállalói adatok kezelése foglalkoztatás keretében

#### Különleges adatkategóriák
- **GDPR Art. 9**: ha a panasz egészségügyi állapotra, etnikai hovatartozásra, szexuális irányultságra vonatkozik
  - Külön hozzájárulás vagy jogi kötelezettség szükséges
  - Például: zaklatási panasz esetén szexuális vonatkozású információk
- **Büntetőjogi adatok**: GDPR Art. 10 - ha fegyelmi ügy bűncselekménnyel kapcsolatos
  - Csak jogi kötelezettség vagy hivatali hatáskör alapján

#### Adatmegőrzési idők
- **Fegyelmi ügyek (Mt.)**: munkaviszony megszűnése + 3 év
- **Fegyelmi ügyek (Kjt.)**: Kjt. 87. § alapján 3 év után törölhető a nyilvántartásból (de archívum továbbra is megőrizhető)
- **Fegyelmi ügyek (Kttv., Kit.)**: hasonlóan 3 év aktív nyilvántartás, utána archívum
- **Panaszok (GrievanceCase)**: megoldástól + 5 év (bizonyítási kötelezettség miatt)
- **Zaklatási, diszkriminációs ügyek**: megoldástól + 8 év (Ebktv. alapján hosszabb megőrzés indokolt)
- **Felmentéshez vezető fegyelmi ügyek**: véglegesen (igazolási kötelezettség)

#### Hozzáférési jogok korlátai
- **Bizalmasság**: Investigation, GrievanceCase confidentiality_level alapján korlátozott hozzáférés
- **Névtelen bejelentők védelme**: is_anonymous=true esetén bejelentő személye nem hozzáférhető
- **Tanúk védelme**: Witness adatok csak jogosultak számára
- **Harmadik fél adatai**: panaszban érintett személyek adatvédelme

### Whistleblowing - Bejelentővédelmi szabályok

- **EU Whistleblowing Directive (2019/1937)**: 2021. december 17. hatálybalépés
- **Magyar jogszabály**: 2023. évi CLXXXIII. törvény a visszaélések bejelentéséről
  - 50+ fős szervezetek kötelező bejelentési csatornája
  - Névtelen bejelentések fogadása
  - Bejelentő védelme (tilos hátrányos megkülönböztetés, megtorlás)
  - Bizalmas kezelés
  - 7 napon belül visszaigazolás, 3 hónapon belül érdemi válasz

## 4. Hozzáférési szintek

| Entitás | HR Admin | HR Manager | Manager | Employee | Legal Counsel |
|---|---|---|---|---|---|
| DisciplinaryCase | CRUD | CRUD | Read (team) | Read (saját) | CRUD |
| Investigation | CRUD | CRUD | Read (ha assigned) | Read (ha érintett) | CRUD |
| Evidence | CRUD | CRUD | - | - | Read |
| Witness | CRUD | CRUD | - | - | Read |
| DisciplinaryHearing | CRUD | CRUD | Read (ha panel member) | Read (saját) | CRUD |
| DisciplinaryAction | CRUD | CRUD | Read (team) | Read (saját) | CRUD |
| Appeal | CRUD | CRUD | Read (team) | CRUD (saját) | CRUD |
| GrievanceCase | CRUD | CRUD | Read (ha against/assigned) | Create/Read (saját) | CRUD |
| DisciplinaryHistory | Read | Read | Read (team, summary) | Read (saját) | Read |
| EthicsViolation | CRUD | CRUD | - | Read (saját) | CRUD |

**Confidentiality levels**:
- **Strictly confidential**: csak HR Manager, Legal Counsel
- **Confidential**: + assigned investigator, panel members
- **Internal**: + direct manager (ha szükséges)

## 5. Validációs szabályok

### DisciplinaryCase
- **Elévülési határidő**:
  - Mt.: gyakorlat szerint ésszerű határidőn belül
  - Kjt. 82. §: tudomásszerzéstől 6 hónap, elkövetéstől 1 év
  - Kttv. 137. §: tudomásszerzéstől 6 hónap, elkövetéstől 3 év
  - Kit. 127. §: tudomásszerzéstől 6 hónap, elkövetéstől 3 év
- **Meghallgatás kötelező**: Kjt./Kttv./Kit. esetén hearing_required=true
- **Döntési határidő**: ésszerű időn belül (általában 30-60 nap)

### DisciplinaryAction
- **Illetménycsökkentés limitekKjt. 78. §: maximum 30%, maximum 1 év
  - Kttv. 135. §: 3-50% elvonás, maximum 1 év
  - Kit. 124. §: pénzbírság max 1 havi illetmény 50%-a
- **Fellebbezési határidő**:
  - Mt.: általában 15 nap kézhezvételtől
  - Kjt. 86. §: 15 nap közléstől
  - Kttv. 144. §: 15 nap közléstől
  - Kit. 131. §: 15 nap közléstől
- **Jogviszony típus megfelelés**: employment_type alapján megfelelő action_type választása

### Appeal
- **Határidő ellenőrzés**: appeal_date <= disciplinary_action.appeal_deadline
- **Felülvizsgálati fórum jogszerűsége**:
  - Mt.: munkaügyi bíróság
  - Kjt./Kttv./Kit.: felettes szerv, végső soron közigazgatási bíróság

### GrievanceCase
- **Visszaigazolási kötelezettség**: acknowledgement_date <= reported_date + 7 nap (whistleblowing directive)
- **Érdemi válasz határidő**: resolution_date <= reported_date + 3 hónap (whistleblowing directive)

### DisciplinaryHistory
- **Nyilvántartás törlése**: expiry_date után is_active=false
  - Kjt. 87. §: 3 év után törölhető
  - Mt.: 3 év gyakorlati időtartam
- **Recidíva ellenőrzés**: korábbi active fegyelmi büntetések száma befolyásolja az új szankció súlyosságát

## 6. Integrációs pontok

### Külső rendszerek
- **Munkaügyi Bíróság**: fellebbezések, munkaügyi perek (case management integráció)
- **Rendőrség, Ügyészség**: ha fegyelmi ügy bűncselekmény gyanúját veti fel
- **Közigazgatási Bíróság**: Kjt./Kttv./Kit. fellebbezések
- **Adatvédelmi Hatóság (NAIH)**: ha GDPR panasz
- **Egyenlő Bánásmód Hatóság**: diszkriminációs panaszok
- **Munkavédelmi Hatóság**: munkahelyi balesetek, munkavédelmi szabálysértések
- **Whistleblowing platform**: külső vagy belső bejelentő rendszer (compliance hotline)

### Belső entitások
- **Employment**: fegyelmi ügyek munkatársakhoz
- **Person**: vizsgálók, tanúk, fegyelmi bizottsági tagok
- **Organization**: szervezeti egységek (hierarchia, felelősségi körök)
- **Document**: határozatok, jegyzőkönyvek, bizonyítékok, fellebbezések
- **Performance Review**: ismétlődő teljesítményproblémák → fegyelmi eljárás
- **TimeTracking**: jelenléti adatok (hiányzások, késések bizonyítékaként)
- **Compensation**: illetménycsökkentés, pénzbírság → bérelszámolás
- **Leave**: felfüggesztés alatt szabadság kezelése

## 7. Kapcsolódó entitások

- **employment-entity.md**: Employment (érintett munkatársak)
- **person-entity.md**: Person (vizsgálók, tanúk, bizottsági tagok, bejelentők)
- **organization-entity.md**: OrgUnit (szervezeti felelősségi körök)
- **position-entity.md**: Position (besorolás visszaminősítés esetén)
- **compensation-entity.md**: CompensationElement (illetménycsökkentés, pénzbírság)
- **document-entity.md**: Document (határozatok, jegyzőkönyvek, bizonyítékok)
- **performance-review-entity.md**: PerformanceReview (teljesítményproblémák kapcsolata)
- **timetracking-entity.md**: DailyTimeRecord (késések, hiányzások)

## 8. Üzleti folyamatok

### 8.1 Fegyelmi eljárás (Mt. - magánszektor)

1. **Bejelentés**: vezető vagy HR tudomást szerez kötelezettségszegésről
2. **DisciplinaryCase létrehozás**: case_type=disciplinary, status=reported
3. **Előzetes vizsgálat**: Investigation létrehozása, tények tisztázása
4. **Bizonyítékok gyűjtése**: Evidence, Witness entitások
5. **Meghallgatás**: informális vagy formális (nagyobb ügyek)
6. **Döntés**: DisciplinaryAction kiszabása
   - Szóbeli figyelmeztetés (nincs dokumentálva)
   - Írásbeli figyelmeztetés
   - Felfüggesztés
   - Rendes felmondás (Mt. 65-66. §)
   - Azonnali hatályú felmondás (Mt. 78. §)
7. **Közlés**: munkatárssal határozat kézbesítése
8. **Fellebbezés**: ha munkatárs fellebbez (Appeal), munkaügyi bíróság
9. **Végrehajtás**: DisciplinaryAction status=active → completed

### 8.2 Fegyelmi eljárás (Kjt./Kttv./Kit. - közszféra)

1. **Kötelezettségszegés tudomásszerzése**: bejelentés vagy hivatalból
2. **Fegyelmi eljárás megindítása**: DisciplinaryCase, employment_type=kjt/kttv/kit
3. **Fegyelmi vizsgálat**: Investigation, lead_investigator kijelölése
4. **Bizonyítékok gyűjtése**: Evidence, Witness
5. **Fegyelmi bizottság összehívása**: Kjt. 80. §, Kttv. 139. §, Kit. 126. §
   - Legalább 3 tagú bizottság
6. **Érintett meghallgatása**: DisciplinaryHearing
   - Védekezési jog biztosítása
   - Védő meghatalmazása lehetséges
   - Jegyzőkönyv készítése kötelező
7. **Fegyelmi határozat**: DisciplinaryAction
   - Kjt. 78. §: megrovás, illetménycsökkentés, besorolás visszaminősítése, felmentés
   - Kttv. 135. §: figyelmeztetés, megrovás, illetményelvonás, besorolás visszaminősítése, felmentés
   - Kit. 124. §: megrovás, pénzbírság, várakozási idő hosszabbítása, besorolás visszaminősítése, felmentés
8. **Határozat közlése**: írásban, indokolással
9. **Fellebbezés**: Appeal (15 napon belül)
   - Felettes szerv felülvizsgálata
   - Közigazgatási bíróság
10. **Nyilvántartás**: DisciplinaryHistory (3 év után törölhető Kjt. 87. §)

### 8.3 Panaszkezelési folyamat (GrievanceCase)

1. **Panasz beérkezése**: munkatárs bejelentése (GrievanceCase creation)
   - Lehet névtelen (is_anonymous=true)
2. **Visszaigazolás**: acknowledgement_date rögzítése (7 napon belül)
3. **Kijelölés**: assigned_to_id (HR, compliance officer)
4. **Értékelés**: severity, investigation_required meghatározása
5. **Vizsgálat** (ha szükséges): Investigation létrehozása
6. **Megoldási javaslat**: resolution_summary összeállítása
7. **Intézkedések**: GrievanceCase resolution_date, kapcsolódó DisciplinaryCase (ha indokolt)
8. **Visszajelzés panaszosnak**: complainant_satisfied ellenőrzés
9. **Follow-up**: follow_up_date-kor utánkövetés
10. **Lezárás**: status=closed

### 8.4 Whistleblowing folyamat

1. **Bejelentés**: GrievanceCase vagy DisciplinaryCase (is_anonymous_report=true)
   - Dedikált bejelentő csatorna (hotline, webform, email)
2. **7 napos visszaigazolás**: bejelentő értesítése (ha nem teljesen névtelen)
3. **Vizsgálat**: Investigation (confidentiality_level=strictly_confidential)
4. **Bejelentő védelme**: tilos megtorlás, hátrányos megkülönböztetés
5. **3 hónapos érdemi válasz**: investigation eredménye, intézkedések
6. **Dokumentálás**: teljes audit trail megőrzése

## 9. Reporting és analitika

### Fegyelmi statisztikák
- **Fegyelmi ügyek száma**: DisciplinaryCase count (év, negyedév, hónap)
- **Jogviszony típus bontás**: employment_type szerinti megoszlás
- **Súlyosság szerinti bontás**: severity (minor/moderate/serious/critical)
- **Szankció típusok megoszlása**: action_type statisztika
- **Elévült ügyek**: statute_of_limitations_date túllépése

### Panaszkezelési metrikák
- **Panaszok száma**: GrievanceCase count
- **Panasz típus megoszlás**: grievance_type bontás (harassment, discrimination, bullying...)
- **Átlagos megoldási idő**: resolution_date - reported_date
- **Visszaigazolási megfelelés**: acknowledgement_date <= reported_date + 7 nap %
- **Elégedettség**: complainant_satisfied arány

### Compliance monitoring
- **Fegyelmi eljárási határidők betartása**: Kjt./Kttv./Kit. 6 hónapos határidő
- **Meghallgatási kötelezettség teljesítése**: hearing_required=true esetén DisciplinaryHearing létezik-e
- **Fellebbezési arány**: Appeal / DisciplinaryAction %
- **Fellebbezési sikeresség**: appeal_outcome=overturned / total appeals %
- **Whistleblowing compliance**: 7 napos visszaigazolás, 3 hónapos válasz határidők

### Kockázati elemzés
- **Visszatérő problémák**: gyakori grievance_type, incident típusok
- **Hotspot területek**: melyik org_unit-ban több fegyelmi ügy / panasz
- **Vezetői felelősség**: melyik vezető alatt több fegyelmi probléma
- **Trend analízis**: növekvő vagy csökkenő fegyelmi ügyek száma

### Jogsértési mintázatok
- **Leggyakoribb kötelezettségszegések**: alleged_violation kategorizálás
- **Recidíva arány**: DisciplinaryHistory alapján ismétlődő szabálysértők
- **Zaklatási / diszkriminációs esetek**: GrievanceCase grievance_type=harassment/discrimination

## 10. Nyitott kérdések és jövőbeli fejlesztések

### Technológiai kérdések
1. **Case management platform**: külső fegyelmi / panaszkezelő szoftver vs saját fejlesztés?
2. **Whistleblowing platform**: harmadik féltől vásárolt platform (EQS Integrity Line, NAVEX Global) vs belső?
3. **Titkosított kommunikáció**: bejelentők számára biztonságos csatorna (end-to-end encryption)?
4. **Mobil app**: panaszok mobil bejelentése?

### Funkcionális fejlesztések
1. **Automatikus workflow**: státusz változások alapján értesítések, emlékeztetők
2. **Template library**: fegyelmi határozat sablonok jogviszony típusonként
3. **Risk scoring**: automatikus súlyossági értékelés (severity suggestion)
4. **Pattern recognition**: ismétlődő problémák, kockázati területek azonosítása AI-val
5. **Mediáció támogatás**: konfliktuskezelési folyamat támogatása fegyelmi eljárás előtt

### Compliance és jogi
1. **EU Whistleblowing Directive változások**: jogszabály frissítések követése
2. **Munkaügyi Bíróság case law**: friss ítéletek, precedensek figyelése
3. **GDPR auditálás**: fegyelmi adatok kezelésének felülvizsgálata
4. **Ebktv. compliance**: diszkriminációs panaszok kezelése, reporting

### Integrációk
1. **Munkaügyi Bíróság elektronikus ügyintézés**: e-peremlékeztetők1. **Compliance hotline szolgáltatók**: NAVEX Global, EthicsPoint integráció
3. **Legal case management**: ügyvédi irodai rendszerekkel integráció (nagyobb ügyek esetén)
4. **HR Analytics platform**: fegyelmi adatok összevont elemzése workforce analytics-szel

### Folyamat optimalizáció
1. **Alternate Dispute Resolution (ADR)**: mediáció, egyeztetés fegyelmi eljárás előtt
2. **Early warning system**: teljesítményproblémák korai jelzése (performance review integráció)
3. **Manager training**: vezetői képzések a fegyelmi eljárások helyes kezeléséről
4. **Transparency**: aggregált fegyelmi statisztikák publikálása (anonim módon)
