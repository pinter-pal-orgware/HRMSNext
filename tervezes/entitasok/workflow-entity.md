# Workflow / Case Management domén – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer – magyar jogszabályi környezet  
> **Verzió:** 0.1 – első tervezet  
> **Utolsó frissítés:** 2025. február  
> **Kapcsolódó dokumentum:** hrms-er-osszefoglalo.md, minden operatív domén

---

## 1. A domén célja és hatóköre

A **Workflow / Case Management domén** egy keresztmetszeti (cross-cutting) bounded context: nem tartozik egyetlen üzleti domén alá sem, hanem az összes többi HRMS modul jóváhagyási, feladatkezelési és ügykezelési igényét egységes motoron keresztül elégíti ki.

### 1.1. A probléma, amit megold

A naiv megközelítésben minden domén saját jóváhagyási logikát implementál:

```
LeaveRequest.status:    REQUESTED → APPROVED / REJECTED
PayrollRecord.status:   CALCULATED → REVIEWED → APPROVED → LOCKED
PerformanceReview.status: ... → PENDING_APPROVAL → COMPLETED
PayrollCorrection.approvedBy / approvedAt ...
```

Ez hat különböző jóváhagyás-implementációt, hat különböző értesítési megoldást, hat különböző határidő-kezelőt és hat különböző auditnyomot jelent – amelyek mindegyike külön fejlesztendő, tesztelendő és karbantartandó. A cross-cutting BC ezt az ismétlést szünteti meg.

### 1.2. Mit nyújt a többi domén számára

- **Egységes jóváhagyási motor:** bármely entitás bármely állapotváltása irányítható ezen keresztül
- **Feladatlista (inbox):** minden felhasználónak egyetlen helyen jelenik meg, mit kell tennie – legyen az szabadságkérelem jóváhagyása, bérszámfejtés ellenőrzése vagy minősítési lap aláírása
- **Határidő- és SLA-kezelés:** esedékességi figyelmeztetések, eszkalációk egységesen
- **Auditnyom:** minden jóváhagyási döntés, megjegyzés és határidő-módosítás egyetlen helyen rögzítve
- **Delegálás és helyettesítés:** vezető távolléte esetén automatikus átirányítás
- **Értesítési alap:** a notifikációs rendszer egyetlen forrásból dolgozik

### 1.3. Amit nem tesz ez a domén

- **Nem tartalmaz üzleti logikát.** Azt, hogy egy szabadságkérelem elfogadható-e (pl. van-e elegendő szabadságkeret), a Leave domén dönti el – a Workflow csak a döntési folyamatot vezérli.
- **Nem helyettesíti a forrás entitás állapotgépét.** A `LeaveRequest.status` változása a Leave doménben történik; a Workflow az ehhez vezető emberi döntési folyamatot kezeli.
- **Nem tárolja a tárgyi adatot.** A kérelem részletei (hány nap, milyen típusú szabadság) a forrás entitásban maradnak.

### 1.4. A domén entitásai

| Entitás | Típus | Leírás |
|---|---|---|
| **WorkflowDefinition** | Törzsadat | Egy folyamattípus sablona (lépések, átmenetek, szabályok) |
| **StepDefinition** | Törzsadat | Egy lépés definíciója a sablonon belül |
| **TransitionDefinition** | Törzsadat | Engedélyezett átmenetek lépések között |
| **WorkflowInstance** | Operatív | Egy konkrét, futó folyamat – aggregációs gyök |
| **WorkflowTask** | Operatív | Egy adott lépéshez tartozó emberi feladat |
| **WorkflowEvent** | Operatív | A folyamat életciklusa során keletkező esemény (döntés, megjegyzés, átmenet) |
| **EscalationRule** | Törzsadat | Eszkalációs szabály: mikor, kinek, hogyan |
| **DelegationRule** | Operatív | Helyettesítési/delegálási beállítás |

---

## 2. WorkflowDefinition entitás

### 2.1. Az entitás célja

A **WorkflowDefinition** egy folyamattípus újrafelhasználható sablona. Meghatározza a lépések sorát, az engedélyezett átmeneteket, az SLA-határidőket és az eszkalációs szabályokat. A sablon nem tartalmaz konkrét személyeket – a feladat-hozzárendelés a futó instance szintjén, dinamikusan történik (pl. „a kérelmező közvetlen vezetője" feloldása az Employment adataiból).

Egy HRMS-ben tipikusan előforduló `WorkflowDefinition` rekordok:

| Kód | Leírás | Forrás domén |
|---|---|---|
| `LEAVE_APPROVAL` | Szabadságkérelem jóváhagyása | Leave |
| `LEAVE_PLAN_APPROVAL` | Éves szabadságterv jóváhagyása | Leave |
| `EMPLOYMENT_CHANGE` | Jogviszony-módosítás (áthelyezés, fizetésemelés) | Employment |
| `EMPLOYMENT_TERMINATION` | Jogviszony megszüntetés | Employment |
| `PAYROLL_APPROVAL` | Bérszámfejtés ellenőrzés és zárolás | Payroll |
| `PAYROLL_CORRECTION_APPROVAL` | Visszamenőleges korrekció jóváhagyása | Payroll |
| `PERFORMANCE_REVIEW_APPROVAL` | Teljesítményértékelés jóváhagyása | PerformanceReview |
| `GOAL_SETTING_APPROVAL` | Célkitűzés kölcsönös elfogadása | PerformanceReview |
| `QUALIFICATION_VERIFICATION` | Végzettség/képesítés hitelesítése | Qualification |
| `DISCIPLINARY_PROCESS` | Fegyelmi eljárás (többlépéses) | DisciplinaryAction |
| `DOCUMENT_SIGNING` | Munkaszerződés / kinevezés aláírás | Document |
| `OVERTIME_APPROVAL` | Rendkívüli munkavégzés elrendelése | TimeTracking |

### 2.2. Property-k

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `organizationId` | UUID FK | nem | null = rendszer-szintű sablon; kitöltve = szervezet-specifikus testreszabás | |
| `code` | string | igen | Egyedi kód (pl. `LEAVE_APPROVAL`) | Forrás domén általi hivatkozáshoz |
| `name` | string | igen | Emberi neve (pl. „Szabadságkérelem jóváhagyása") | |
| `description` | text | nem | A folyamat leírása | |
| `subjectEntityType` | enum | igen | Melyik domén entitáshoz kapcsolódik: `LEAVE_REQUEST` / `EMPLOYMENT` / `PAYROLL_RECORD` / `PERFORMANCE_REVIEW` / `DOCUMENT` / stb. | A `WorkflowInstance.subjectEntityId` típusának meghatározásához |
| `triggerType` | enum | igen | `MANUAL` / `AUTOMATIC_ON_STATUS_CHANGE` / `SCHEDULED` / `EVENT_DRIVEN` | `AUTOMATIC`: pl. minden új `LeaveRequest` automatikusan indít egy `LEAVE_APPROVAL` instance-t |
| `triggerCondition` | JSON | nem | Ha `triggerType = AUTOMATIC_ON_STATUS_CHANGE`: melyik entitás, melyik státusz-átmenetére | pl. `{"entity": "LeaveRequest", "fromStatus": null, "toStatus": "SUBMITTED"}` |
| `completionBehavior` | enum | igen | `APPROVE_SUBJECT` / `REJECT_SUBJECT` / `NOTIFY_ONLY` / `CUSTOM` | Mit csinál a folyamat lezárásakor a forrás entitással |
| `approvalBehavior` | enum | igen | `ALL_MUST_APPROVE` / `ANY_ONE_APPROVES` / `MAJORITY` / `SEQUENTIAL` | Több párhuzamos jóváhagyó esetén a döntési szabály |
| `allowRejectionAtAnyStep` | boolean | igen | Bármely lépésben visszautasítható-e a kérelem | |
| `allowWithdrawalByInitiator` | boolean | igen | A kérelmező visszavonhatja-e az aktív folyamatot | |
| `isSlaEnforced` | boolean | igen | Aktív-e az SLA-határidő és az eszkaláció | |
| `defaultSlaHours` | integer | nem | Alapértelmezett SLA az összes lépésre (órában), ha `isSlaEnforced = true` | |
| `applicableEmploymentTypes` | enum[] | nem | null = minden jogviszony-típusra; kitöltve = csak megjelöltekre | pl. Kjt.-specifikus jóváhagyási folyamatok |
| `requiresComment` | enum | igen | `NEVER` / `ON_REJECTION` / `ON_APPROVAL` / `ALWAYS` | Kötelező-e szöveges indoklás |
| `isParallelizable` | boolean | igen | Indítható-e ugyanarra az alanyra párhuzamosan több instance | pl. szabadságkérelemnél általában nem; fegyelmi eljárásnál sem |
| `maxParallelInstances` | integer | nem | Ha `isParallelizable = true`: max hány párhuzamos instance | |
| `isActive` | boolean | igen | Aktív sablon-e | |
| `validFrom` | date | igen | | |
| `validTo` | date | nem | | |
| `version` | integer | igen | Sablon verziószáma (1, 2, 3...) | Változásnál új verzió; a futó instance-ok a létrehozáskor érvényes verziót használják |
| `createdAt` | datetime | igen | | Audit |
| `createdBy` | UUID FK | igen | | Audit |

---

## 3. StepDefinition entitás

### 3.1. Az entitás célja

A **StepDefinition** egy `WorkflowDefinition`-on belüli egyetlen lépés sémáját írja le: ki hajtja végre, milyen típusú feladat, mennyi ideje van rá, és mi történik, ha nem reagál. A lépések sorrendje a `TransitionDefinition` entitásokon keresztül érvényesül – nem egy szekvenciális index alapján, hanem explicit átmeneti szabályokkal (ez lehetővé teszi az elágazó és párhuzamos folyamatokat).

### 3.2. Property-k

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `definitionId` | UUID FK | igen | Melyik `WorkflowDefinition`-hoz tartozik | |
| `code` | string | igen | Lépés kódja (pl. `MANAGER_APPROVAL`, `HR_REVIEW`, `SECOND_LEVEL_APPROVAL`) | |
| `name` | string | igen | Lépés neve (pl. „Közvetlen vezető jóváhagyása") | Feladatlistán megjelenő szöveg |
| `stepType` | enum | igen | `APPROVAL` / `REVIEW` / `INFORMATION` / `SIGNING` / `DATA_ENTRY` / `SYSTEM_ACTION` | `SYSTEM_ACTION`: emberi beavatkozás nélkül, automatikusan teljesül (pl. értesítés küldése) |
| `assigneeType` | enum | igen | `DIRECT_MANAGER` / `SECOND_LEVEL_MANAGER` / `HR_OFFICER` / `PAYROLL_OFFICER` / `SPECIFIC_ROLE` / `SPECIFIC_PERSON` / `INITIATOR` / `DYNAMIC` | `DYNAMIC`: a `assigneeResolver` meghatározza futásidőben |
| `assigneeRoleCode` | string | nem | Ha `assigneeType = SPECIFIC_ROLE`: melyik rendszerszerep | |
| `assigneePersonId` | UUID FK | nem | Ha `assigneeType = SPECIFIC_PERSON`: ki konkrétan | Ritkán használt; inkább szerepkör-alapú hozzárendelés javasolt |
| `assigneeResolver` | string | nem | Ha `assigneeType = DYNAMIC`: a feloldó logika kódja (pl. `ORG_UNIT_HEAD`, `COST_CENTER_APPROVER`) | A tényleges feloldás az Employment + Organization adatokból |
| `isStartStep` | boolean | igen | Ez-e a folyamat kezdő lépése | |
| `isEndStep` | boolean | igen | Ez-e terminális lépés (a folyamat itt ér véget) | |
| `slaHours` | integer | nem | Lépésre vonatkozó SLA (órában); felülírja a definition szintű alapértelmezést | |
| `reminderAfterHours` | integer | nem | Emlékeztető küldése X óra tétlenség után | |
| `escalationAfterHours` | integer | nem | Eszkaláció indítása X óra tétlenség után | |
| `escalationRuleId` | UUID FK | nem | Melyik `EscalationRule` alkalmazandó | |
| `allowedActions` | enum[] | igen | Ebben a lépésben milyen döntések hozhatók: `APPROVE` / `REJECT` / `RETURN_FOR_CORRECTION` / `DELEGATE` / `REQUEST_INFO` / `SKIP` | |
| `requiresComment` | enum | nem | Ha eltér a `WorkflowDefinition` szintű beállítástól: `NEVER` / `ON_REJECTION` / `ON_APPROVAL` / `ALWAYS` | |
| `notifyOnEntry` | boolean | igen | Értesítés küldendő-e a hozzárendelt személynek, amikor a feladat hozzá kerül | |
| `notifyOnReminder` | boolean | igen | Emlékeztető értesítés | |
| `isMandatory` | boolean | igen | Átugorható-e ez a lépés (pl. ha a jóváhagyó maga a kérelmező) | |
| `skipCondition` | JSON | nem | Ha `isMandatory = false`: milyen feltétel esetén ugorja át automatikusan | pl. `{"if": "initiator_is_assignee"}` |
| `sortOrder` | integer | igen | Vizuális megjelenítési sorrend | |

**Megjegyzések:**
- Az `assigneeType = DIRECT_MANAGER` feloldása futásidőben történik: a `WorkflowInstance.subjectEmploymentId` → `Employment.orgUnitId` → `OrgUnit` felelős vezető → `Employment` → `Person`. Ez a feloldási lánc az Organization és Employment domének adatain alapul; a Workflow domén csak a feloldás eredményét (egy konkrét `personId`) tárolja a `WorkflowTask`-ban.
- `SKIP` akció: pl. ha az automatikus jóváhagyási szabály (pl. 1 napos szabadság = automatikus elfogadás) teljesül, a lépés emberi beavatkozás nélkül befejezhető.

---

## 4. TransitionDefinition entitás

### 4.1. Az entitás célja

A **TransitionDefinition** két `StepDefinition` közötti engedélyezett átmenetet definiál, és azt, hogy melyik esemény (akció) váltja ki. Ez teszi lehetővé az elágazó folyamatokat: pl. ha a közvetlen vezető elutasítja, más lépés következik, mint ha jóváhagyja.

### 4.2. Property-k

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `definitionId` | UUID FK | igen | Melyik `WorkflowDefinition`-hoz tartozik | |
| `fromStepId` | UUID FK | igen | Kiinduló lépés | |
| `toStepId` | UUID FK | nem | Célállomás lépés; null = folyamat lezárása | |
| `triggerAction` | enum | igen | `APPROVE` / `REJECT` / `RETURN_FOR_CORRECTION` / `ESCALATE` / `TIMEOUT` / `WITHDRAW` / `SYSTEM` | Melyik esemény váltja ki ezt az átmenetet |
| `condition` | JSON | nem | Feltételes átmenet (pl. csak akkor, ha az összeg meghaladja a határt): `{"field": "subject.amount", "op": "gt", "value": 500000}` | Feltétel nélkül az átmenet feltétel nélkül aktív |
| `subjectStatusChange` | enum | nem | Ha az átmenet bekövetkezik, a forrás entitás állapotát erre változtatja | pl. `APPROVED`, `REJECTED`, `RETURNED` – ez aktivál egy eseményt a forrás doménben |
| `name` | string | nem | Átmenet neve (pl. „Jóváhagyva → HR ellenőrzés", „Elutasítva → Lezárás") | |

**Megjegyzések:**
- A `subjectStatusChange` a Workflow domén egyetlen pontja, ahol a forrás entitásba visszaír. Az állapotváltást esemény formájában közli (domain event), a forrás domén saját logikája hajtja végre a tényleges módosítást.
- Feltételes átmenet példa: a `LEAVE_APPROVAL` folyamatnál, ha a kérelmezett napok száma ≤ 1, a vezető jóváhagyása után nem szükséges HR-ellenőrzési lépés – `condition: {"field": "subject.requestedDays", "op": "lte", "value": 1}`.

---

## 5. WorkflowInstance entitás

### 5.1. Az entitás célja

A **WorkflowInstance** egy konkrét, élő vagy lezárt folyamatot reprezentál. Ez a domén aggregációs gyöke: minden futó feladat, esemény, megjegyzés és döntés erre a rekordra épül. Egy `WorkflowInstance` pontosan egy forrás entitáshoz (subject) kapcsolódik, de egy forrás entitáshoz az életciklusa során több instance is tartozhat (pl. visszaküldés és újraindítás esetén).

### 5.2. Azonosítók és kapcsolatok

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `definitionId` | UUID FK | igen | Melyik sablon alapján futtatják | A létrehozáskor érvényes verziót „fagyasztja" |
| `definitionVersion` | integer | igen | A sablon verziószáma a létrehozás pillanatában | Ha a sablon frissül, a futó instance nem változik |
| `subjectEntityType` | enum | igen | A tárgyi entitás típusa (pl. `LEAVE_REQUEST`) | |
| `subjectEntityId` | UUID | igen | A tárgyi entitás azonosítója | Nincs FK kényszer – a Workflow domén nem ismeri a forrás táblák struktúráját |
| `initiatorId` | UUID FK | igen | A folyamatot kezdeményező személy | |
| `initiatorEmploymentId` | UUID FK | nem | Melyik jogviszony kontextusában indult | |

### 5.3. Állapot és lépések

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `status` | enum | igen | `PENDING` / `IN_PROGRESS` / `APPROVED` / `REJECTED` / `RETURNED` / `WITHDRAWN` / `CANCELLED` / `EXPIRED` | |
| `currentStepId` | UUID FK | nem | Az éppen aktív lépés; null ha a folyamat lezárult | |
| `currentAssigneeId` | UUID FK | nem | Az éppen aktuálisan felelős személy | Denormalizált gyors lekérdezéshez (feladatlista) |
| `currentAssigneeRole` | enum | nem | Az aktuális felelős szerepköre | |

**Állapotgép:**
```
PENDING
  │ (instance létrehozva, első lépés hozzárendelve)
  ▼
IN_PROGRESS ◄──────────────────────────────────────────────┐
  │                                                         │
  ├──► [APPROVE minden szükséges lépésben] ──► APPROVED     │
  │                                                         │
  ├──► [REJECT bármely lépésben, ha engedélyezett] ──► REJECTED
  │                                                         │
  ├──► [RETURN_FOR_CORRECTION] ──► RETURNED                 │
  │         └── kérelmező javít, visszaküldi ───────────────┘
  │
  ├──► [WITHDRAW kérelmező által] ──► WITHDRAWN
  │
  ├──► [Adminisztrátori megszüntetés] ──► CANCELLED
  │
  └──► [SLA timeout, eszkaláció sikertelen] ──► EXPIRED
```

### 5.4. Határidők és SLA

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `dueDate` | datetime | nem | A teljes folyamat esedékessége | Jogszabályi határidőből számítható (pl. Mt. szabadságkérelem) |
| `slaDeadline` | datetime | nem | Az aktuális lépés SLA-határideje | Lépésváltáskor frissül |
| `slaBreached` | boolean | igen | Megsértette-e az SLA-t valamelyik lépés | Riportoláshoz; szervezeti teljesítményindikátor |
| `slaBreachCount` | integer | igen | Hányszor sértette meg az SLA-t a folyamat során | |
| `reminderSentAt` | datetime | nem | Utolsó emlékeztető kiküldésének időpontja | |
| `escalatedAt` | datetime | nem | Utolsó eszkaláció időpontja | |
| `escalatedToId` | UUID FK | nem | Aktuális eszkalációs személy | |

### 5.5. Eredmény és lezárás

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `outcome` | enum | nem | `APPROVED` / `REJECTED` / `RETURNED` / `WITHDRAWN` / `CANCELLED` / `EXPIRED` | Lezáráskor kitöltve |
| `outcomeReason` | text | nem | A döntés szöveges indoklása (ha kötelező) | |
| `outcomeByPersonId` | UUID FK | nem | Ki hozta a végső döntést | |
| `startedAt` | datetime | igen | A folyamat indulásának időpontja | |
| `completedAt` | datetime | nem | A folyamat lezárásának időpontja | |
| `totalDurationMinutes` | integer | nem | Teljes átfutási idő percben (riportoláshoz) | Számított mező |

### 5.6. Technikai mezők

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `priority` | enum | nem | `LOW` / `NORMAL` / `HIGH` / `URGENT` | Feladatlistán való rendezéshez |
| `tags` | string[] | nem | Szabad szöveges címkék (pl. „sürgős", „Kjt.", „2025-Q1") | Szűréshez és csoportosításhoz |
| `externalReference` | string | nem | Külső iktatószám vagy hivatkozás (pl. papíralapú irat azonosítója) | |
| `contextSnapshot` | JSON | nem | A döntéshez releváns forrás-adat pillanatkép az induláskor | pl. a kérelmezett napok száma, a munkavállaló neve, a szabadságegyenleg – olvashatóság nélküli forrás-entitás hozzáférés esetén |
| `createdAt` | datetime | igen | | Audit |
| `createdBy` | UUID FK | igen | | Audit |
| `updatedAt` | datetime | igen | | Audit |

**Megjegyzések:**
- A `subjectEntityId` szándékosan nem FK: a Workflow domén nem tartalmaz referenciális kényszert a forrás doménekre. Ez az egyetlen módja annak, hogy a Workflow valóban cross-cutting legyen – nem függ egyetlen forrás domén adatbázis-sémájától sem. A konzisztenciát az alkalmazás réteg biztosítja.
- A `contextSnapshot` JSON különösen értékes auditálási szempontból: ha a szabadság-egyenleg az azóta megváltozott, a döntéshozatal pillanatában fennállt adatok visszakereshetők. Tartalmaz emberi olvasásra szánt kulcsadatokat (nem a teljes entitást).

---

## 6. WorkflowTask entitás

### 6.1. Az entitás célja

A **WorkflowTask** egy konkrét emberi feladatot képvisel: egy adott személy egy adott lépésben elvégzendő tennivalóját. Ez jelenik meg a felhasználó „beérkező feladatok" listájában (inbox). Egy `WorkflowInstance` életciklusa során több `WorkflowTask` keletkezik – lépésenként egy, plusz eszkaláció esetén pótlólagos feladatok.

### 6.2. Property-k

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `instanceId` | UUID FK | igen | Melyik `WorkflowInstance`-hoz tartozik | |
| `stepDefinitionId` | UUID FK | igen | Melyik `StepDefinition` alapján keletkezett | |
| `assigneeId` | UUID FK | igen | A feladat elvégzésére kijelölt személy | A `StepDefinition.assigneeType` sablon alapján, futásidőben feloldva |
| `assignedAt` | datetime | igen | Hozzárendelés időpontja | |
| `dueAt` | datetime | nem | A feladat esedékessége | `assignedAt` + `StepDefinition.slaHours` |
| `status` | enum | igen | `PENDING` / `IN_PROGRESS` / `COMPLETED` / `DELEGATED` / `ESCALATED` / `CANCELLED` | |
| `action` | enum | nem | Ha `status = COMPLETED`: melyik döntés született: `APPROVE` / `REJECT` / `RETURN_FOR_CORRECTION` / `REQUEST_INFO` / `SKIP` | |
| `actionTakenAt` | datetime | nem | Döntés időpontja | |
| `comment` | text | nem | A döntéshez fűzött megjegyzés | |
| `delegatedToId` | UUID FK | nem | Ha `status = DELEGATED`: kinek adta tovább | |
| `delegatedAt` | datetime | nem | Delegálás időpontja | |
| `delegationReason` | text | nem | Delegálás indoka | |
| `isEscalated` | boolean | igen | Eszkalálva lett-e ez a feladat | |
| `escalatedAt` | datetime | nem | Eszkaláció időpontja | |
| `escalatedToId` | UUID FK | nem | Eszkaláció célszemélye | |
| `reminderCount` | integer | igen | Kiküldött emlékeztetők száma (0-tól indul) | |
| `lastReminderAt` | datetime | nem | Utolsó emlékeztető időpontja | |
| `viewedAt` | datetime | nem | Mikor nyitotta meg először az assignee | Olvasási visszaigazoláshoz |
| `createdAt` | datetime | igen | | Audit |

**Megjegyzések:**
- **Delegálás vs. eszkaláció:** a delegálás a jelenlegi felelős saját döntése (a feladatot átadja valakinek); az eszkaláció automatikus, SLA-túllépéskor triggereli a rendszer az `EscalationRule` alapján.
- A `comment` mező tartalma a döntésnaplóba (`WorkflowEvent`) is átmásolódik – a `WorkflowTask` lezárása után a feladat törölhető lenne, de az esemény megmarad az audit-célokra.
- Egy lépéshez egyszerre több `WorkflowTask` is aktív lehet, ha az `approvalBehavior = ANY_ONE_APPROVES` és több potenciális jóváhagyó van (pl. bármely elérhető HR ügyintéző jóváhagyhat).

---

## 7. WorkflowEvent entitás

### 7.1. Az entitás célja

A **WorkflowEvent** az immutábilis auditnapló: a folyamat életciklusa során keletkező összes eseményt rögzíti időrendben. Nem módosítható, nem törölhető – a jogi bizonyítóerőt ez adja a folyamat dokumentációjának. Minden döntés, megjegyzés, delegálás, eszkaláció, határidő-módosítás és állapotváltás itt jelenik meg.

### 7.2. Property-k

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `instanceId` | UUID FK | igen | Melyik `WorkflowInstance`-hoz tartozik | |
| `taskId` | UUID FK | nem | Ha konkrét feladathoz kapcsolódik | |
| `eventType` | enum | igen | Esemény típusa (ld. lent) | |
| `actorId` | UUID FK | nem | Az eseményt kiváltó személy; null ha rendszer-esemény | |
| `actorRole` | enum | nem | Az aktor szerepköre az adott lépésben | |
| `stepCode` | string | nem | Melyik lépésben keletkezett | Denormalizált a könnyű olvasáshoz |
| `fromStatus` | enum | nem | Az instance állapota az esemény előtt | |
| `toStatus` | enum | nem | Az instance állapota az esemény után | |
| `comment` | text | nem | Szöveges tartalom (megjegyzés, indoklás, visszajelzés) | |
| `metadata` | JSON | nem | Esemény-specifikus adatok (pl. delegálás esetén az új felelős neve, eszkaláció esetén a késés percben) | |
| `occurredAt` | datetime | igen | Az esemény bekövetkezésének időpontja | Immutábilis |
| `ipAddress` | string | nem | Az aktor IP-címe (emberi akció esetén) | Biztonsági audithoz |

**Eseménytípusok (`eventType` enum értékei):**

| Érték | Leírás |
|---|---|
| `INSTANCE_CREATED` | Folyamat létrehozva |
| `TASK_ASSIGNED` | Feladat hozzárendelve |
| `TASK_VIEWED` | Feladat megtekintve |
| `APPROVED` | Jóváhagyva |
| `REJECTED` | Elutasítva |
| `RETURNED_FOR_CORRECTION` | Visszaküldve javításra |
| `INFO_REQUESTED` | Információ kérve |
| `INFO_PROVIDED` | Információ megadva |
| `COMMENT_ADDED` | Megjegyzés hozzáfűzve |
| `DELEGATED` | Feladat delegálva |
| `ESCALATED` | Eszkalálva |
| `REMINDER_SENT` | Emlékeztető kiküldve |
| `SLA_BREACHED` | SLA határidő meghaladva |
| `STEP_SKIPPED` | Lépés automatikusan átugorva |
| `WITHDRAWN` | Kérelmező visszavonta |
| `CANCELLED` | Adminisztrátor megszüntette |
| `EXPIRED` | Lejárt (eszkaláció sem segített) |
| `INSTANCE_COMPLETED` | Folyamat sikeresen lezárult |
| `DUE_DATE_CHANGED` | Esedékesség módosítva |
| `PRIORITY_CHANGED` | Prioritás módosítva |

---

## 8. EscalationRule entitás

### 8.1. Az entitás célja

Az **EscalationRule** meghatározza, hogy SLA-túllépés esetén mit tegyen a rendszer: kit értesítsen, kinek adja át a feladatot, és ha az eszkaláció sem eredményes, mi történjen. Egy `StepDefinition`-höz tartozik, de újrafelhasználható: több lépés hivatkozhat ugyanarra az eszkalációs szabályra.

### 8.2. Property-k

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `code` | string | igen | Egyedi kód (pl. `MANAGER_ESCALATION_48H`) | |
| `name` | string | igen | Emberi neve | |
| `level1AfterHours` | integer | igen | Hány óra tétlenség után lép be az 1. szintű eszkaláció | |
| `level1Action` | enum | igen | `SEND_REMINDER` / `NOTIFY_SUPERVISOR` / `AUTO_REASSIGN` / `AUTO_APPROVE` / `AUTO_REJECT` | |
| `level1TargetType` | enum | nem | `DIRECT_SUPERVISOR` / `DEPARTMENT_HEAD` / `HR_OFFICER` / `SPECIFIC_ROLE` | Az eszkaláció célszemélyének típusa |
| `level2AfterHours` | integer | nem | Ha az 1. szint sem oldotta meg: 2. szintű eszkaláció | |
| `level2Action` | enum | nem | | |
| `level2TargetType` | enum | nem | | |
| `finalAction` | enum | nem | Ha minden szint sikertelen: `EXPIRE_INSTANCE` / `AUTO_APPROVE` / `AUTO_REJECT` / `ALERT_ADMIN` | |
| `notifyInitiatorOnEscalation` | boolean | igen | Értesítse-e a kérelmezőt az eszkalációról | |
| `createdAt` | datetime | igen | | Audit |

**Megjegyzések:**
- `AUTO_APPROVE`: óvatosan használandó – csak ott alkalmazható, ahol a határidő lejárta üzletileg elfogadási vélelemet kelt (pl. egyszerű 1 napos szabadságkérelem: ha a vezető 48 óra alatt nem reagál, automatikusan jóváhagyódik). Jogszabályi folyamatoknál (Kttv., Kjt.) ez általában nem alkalmazható.
- `AUTO_REJECT`: általában nem célszerű emberi döntés helyettesítésére, de fegyelmi eljárás esetén indokolt lehet, ha a vizsgálati határidő lejárt és senki nem határozott.

---

## 9. DelegationRule entitás

### 9.1. Az entitás célja

A **DelegationRule** egy személy által beállított helyettesítési / delegálási szabályt tárol: szabadság, betegség vagy más ok miatt az adott időszakban a feladatait ki látja el helyette. Ez nem azonos a `WorkflowTask` szintű manuális delegálással: a `DelegationRule` proaktív, előre beállított, automatikusan érvényesülő szabály.

### 9.2. Property-k

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `delegatorId` | UUID FK | igen | Az a személy, aki helyett mások jóváhagynak | |
| `delegateId` | UUID FK | igen | Az a személy, aki a feladatokat átveszi | |
| `validFrom` | datetime | igen | Delegálás kezdete | |
| `validTo` | datetime | nem | Delegálás vége; null = visszavonásig érvényes | |
| `scope` | enum | igen | `ALL_WORKFLOWS` / `SPECIFIC_WORKFLOW_TYPES` | |
| `applicableWorkflowCodes` | string[] | nem | Ha `scope = SPECIFIC_WORKFLOW_TYPES`: melyekre vonatkozik | pl. csak `LEAVE_APPROVAL`, `PAYROLL_APPROVAL` |
| `isActive` | boolean | igen | Aktív-e | |
| `reason` | string | nem | Delegálás indoka (pl. „Éves szabadság 2025.08.01-08.15.") | |
| `createdAt` | datetime | igen | | Audit |
| `createdBy` | UUID FK | igen | | Audit |

**Megjegyzések:**
- Ha a delegált személy maga is delegált (láncdelegálás), a rendszer csak egy szintig kövesse – ennél mélyebb lánc jogbiztonsági kérdéseket vet fel.
- Adminisztrátori felülvizsgálat szükséges: a `DelegationRule` létrehozásakor érdemes automatikusan értesíteni a HR-t (különösen közszféra esetén, ahol a jóváhagyási jogkör kiosztása jogszabályi keretek közt mozog – Kttv. 80. §, Mt. 20. §).

---

## 10. Historikus adatkezelés

| Entitás | Kezelési mód | Indok |
|---|---|---|
| **WorkflowDefinition** | `validFrom` / `validTo` + verziózás; futó instance-ok a létrehozáskori verziót rögzítik | Sablon-változás nem módosíthatja a már futó folyamatokat |
| **WorkflowInstance** | `COMPLETED` / `REJECTED` / `CANCELLED` után csak olvasható | Jogi bizonyítóerő (Mt. 286. § – munkaügyi igény elévülése 3 év; közszféra: hosszabb) |
| **WorkflowTask** | Lezárás után csak olvasható | Auditnyom |
| **WorkflowEvent** | Soha nem törölhető, nem módosítható – append-only | Az egyetlen hiteles auditnapló; teljes döntési nyomvonal |
| **EscalationRule** | Soft delete; verziózás nem szükséges (lépés-szintű kötés) | |
| **DelegationRule** | `validTo` kitöltésével inaktiválható; nem törölhető | Visszamenőleges rekonstrukcióhoz szükséges |

**Megőrzési idők:**

| Adat | Megőrzési idő | Jogalap |
|---|---|---|
| `WorkflowInstance` + `WorkflowEvent` (jogviszony-kapcsolódó) | Jogviszony megszűnése + 5 év | Mt. 286. § – munkaügyi igény elévülése; közszféra esetén a személyi irat megőrzési ideje alkalmazandó |
| `WorkflowInstance` (bérszámfejtési jóváhagyás) | Az érintett `PayrollRecord` megőrzési ideje (50 év) | Tny. – a döntési folyamat az elszámolás részének tekintendő |
| `WorkflowInstance` (fegyelmi) | Lezárás + 5 év (jogerő esetén + 10 év) | Pp. elévülési szabályok |
| `WorkflowEvent` | Ugyanaz, mint a `WorkflowInstance` | |
| `DelegationRule` | Érvényesség vége + 5 év | Visszamenőleges rekonstrukcióhoz |

---

## 11. GDPR adatkezelési kategorizáció

| Kategória | Érintett property-k | Jogalap | Megjegyzés |
|---|---|---|---|
| **Döntési adatok** | `WorkflowTask.action`, `comment`, `actionTakenAt` | Jogi kötelezettség (6(1)c) + Jogos érdek (6(1)f) | A jóváhagyási folyamat dokumentálása munkaügyi jogviták esetén bizonyítékként szolgál |
| **Kommunikációs adatok** | `WorkflowEvent.comment` | Jogi kötelezettség (6(1)c) + Jogos érdek (6(1)f) | Mt. 10. § – csak munkavégzéssel összefüggő tartalom; személyes vélemény és különleges adat nem rögzíthető |
| **Hozzárendelési adatok** | `WorkflowTask.assigneeId`, `delegatedToId`, `escalatedToId` | Jogi kötelezettség (6(1)c) | Szükséges a felelősségi körök dokumentálásához |
| **Technikai auditadatok** | `ipAddress`, `viewedAt`, `occurredAt` | Jogos érdek (6(1)f) + Elszámoltathatóság (GDPR 5(2)) | |
| **Delegálási adatok** | `DelegationRule.*` | Jogos érdek (6(1)f) | Szükséges a helyettesítési jogkör igazolásához |

**Kiemelten kezelt szituációk:**
- Ha a `WorkflowEvent.comment` különleges kategóriájú adatra utal (pl. „betegszabadsághoz kapcsolódó" szabadságkérelem indoklásában az egészségi állapotra való hivatkozás), azt nem szabad rögzíteni – a megjegyzés tartalmát a kitöltő felelőssége tartani a jogszabályi keretek között, de a rendszer szintjén figyelmeztetést érdemes megjeleníteni.
- Az érintett (munkavállaló) GDPR 15. cikk szerinti hozzáférési joga kiterjed a saját kérelmeihez kapcsolódó `WorkflowEvent`-ekre is – beleértve a jóváhagyó által fűzött megjegyzéseket.

---

## 12. Hozzáférési szintek (javasolt)

| Szerep | Olvasási jog | Írási jog | Megjegyzés |
|---|---|---|---|
| **Kérelmező munkavállaló** | Saját `WorkflowInstance`-ok és azok `WorkflowEvent`-jei | `WITHDRAW` akció; `INFO_PROVIDED` akció | Nem látja más munkavállalók folyamatait |
| **Közvetlen vezető** | A hozzá rendelt `WorkflowTask`-ok; az érintett beosztott folyamatainak összesítői | `APPROVE` / `REJECT` / `RETURN_FOR_CORRECTION` / `REQUEST_INFO` / `DELEGATE` a hozzá rendelt feladatokon | Nem látja más szervezeti egységek folyamatait |
| **HR ügyintéző** | Az általa kezelt szervezeti egység összes folyamata | `CANCEL` adminisztrátori okokból; sablon-kezelés | |
| **Bérszámfejtő** | Payroll-típusú folyamatok | Payroll-jóváhagyási akciók | |
| **Rendszeradminisztrátor** | Összes folyamat (audit) | `WorkflowDefinition` kezelés; `CANCEL` | |
| **Bármely felhasználó** | Saját `DelegationRule`-ok | Saját `DelegationRule` létrehozása és módosítása | Hatókör-korlátok érvényesek |

---

## 13. Validációs szabályok

| Szabály | Validáció | Megjegyzés |
|---|---|---|
| `WorkflowDefinition`: minden lépésből elérhető legalább egy terminális lépés | Kötelező | Végtelen ciklus megelőzése |
| `WorkflowDefinition`: pontosan egy `isStartStep = true` lépés | Kötelező | |
| `StepDefinition.weight` (párhuzamos lépések esetén) összege 100% | Kötelező | Ha `approvalBehavior = MAJORITY` |
| `WorkflowInstance`: nem indítható, ha `isParallelizable = false` és már van aktív instance ugyanarra az alanyra | Kötelező | |
| `WorkflowTask.dueAt ≤ WorkflowInstance.dueDate` | Kötelező warning | Lépés SLA nem lépheti túl a folyamat határidejét |
| `DelegationRule`: `delegatorId ≠ delegateId` | Kötelező | Önmagának nem delegálhat |
| `DelegationRule`: nincs aktív láncdelegálás (delegateId maga is delegátor ugyanabban az időszakban) | Kötelező warning | |
| `WorkflowEvent`: soha nem módosítható és nem törölhető | Kötelező | Append-only napló |
| `WorkflowTask.comment` kötelező, ha `StepDefinition.requiresComment = ON_REJECTION` és `action = REJECT` | Kötelező | |
| `WorkflowInstance.subjectEntityType` illeszkedik `WorkflowDefinition.subjectEntityType`-hoz | Kötelező | |

---

## 14. Integrációs pontok

### 14.1. A domén mint szolgáltató – más domének általi használat

Minden forrás domén két ponton integrálja a Workflow motort:

**Indítás:** amikor a forrás entitás állapota olyan pontra ér, hogy emberi jóváhagyás szükséges, a forrás domén egy `WorkflowInstance`-t hoz létre a Workflow doménben.

```
Leave domén:
  LeaveRequest.status → SUBMITTED
  → WorkflowInstance létrehozása (definitionCode: LEAVE_APPROVAL, subjectEntityId: leaveRequest.id)

Payroll domén:
  PayrollRun.status → COMPLETED
  → WorkflowInstance létrehozása (definitionCode: PAYROLL_APPROVAL, subjectEntityId: payrollRun.id)
```

**Visszajelzés:** amikor a `WorkflowInstance` lezárul, a Workflow domén egy domain eventet publikál, amelyre a forrás domén feliratkozik, és végrehajtja a tényleges állapotváltást.

```
WorkflowInstance.status → APPROVED
→ domain event: WorkflowApproved { instanceId, subjectEntityType, subjectEntityId, outcome }
→ Leave domén: LeaveRequest.status → APPROVED
→ Payroll domén: PayrollRun.status → LOCKED
```

### 14.2. Forrás doménenként

| Forrás domén | Trigger | Workflow kód | Visszajelzés hatása |
|---|---|---|---|
| **Leave** | `LeaveRequest` benyújtva | `LEAVE_APPROVAL` | `LeaveRequest.status → APPROVED / REJECTED / RETURNED` |
| **Leave** | Éves szabadságterv | `LEAVE_PLAN_APPROVAL` | `LeavePlan.status → APPROVED` |
| **Employment** | Jogviszony-módosítás | `EMPLOYMENT_CHANGE` | `Employment` módosítás érvénybe lép |
| **Employment** | Jogviszony megszüntetés | `EMPLOYMENT_TERMINATION` | `Employment.status → TERMINATED` |
| **Compensation** | Fizetésemelési javaslat | `SALARY_CHANGE_APPROVAL` | `CompensationElement` aktiválva |
| **Payroll** | Bérszámfejtési futtatás | `PAYROLL_APPROVAL` | `PayrollRun.status → LOCKED` |
| **Payroll** | Korrekció | `PAYROLL_CORRECTION_APPROVAL` | `PayrollCorrection.status → APPROVED` |
| **PerformanceReview** | Célkitűzés kölcsönös elfogadás | `GOAL_SETTING_APPROVAL` | `ReviewGoal.status → AGREED` |
| **PerformanceReview** | Értékelés jóváhagyása | `PERFORMANCE_REVIEW_APPROVAL` | `PerformanceReview.status → COMPLETED` |
| **TimeTracking** | Rendkívüli munkavégzés elrendelése | `OVERTIME_APPROVAL` | `OvertimeRecord` létrehozva |
| **Document** | Munkaszerződés aláírás | `DOCUMENT_SIGNING` | `Document.status → SIGNED` |
| **Qualification** | Végzettség hitelesítés | `QUALIFICATION_VERIFICATION` | `Qualification.isVerified → true` |

### 14.3. Kimenő integráció

| Cél | Esemény | Tartalom |
|---|---|---|
| **Értesítési rendszer** | Minden `WorkflowEvent` | Feladatkiosztás, emlékeztető, eszkaláció, lezárás – email / push / in-app |
| **Naptár integráció** | `WorkflowInstance` esedékesség | Jóváhagyási határidő naptárba kerül (Outlook, Google Calendar) |
| **Riportolás / BI** | `WorkflowInstance` lezárva | Átfutási idő, SLA-teljesítés, eszkalációs arány |

---

## 15. Kapcsolódó entitások (hivatkozás)

```
WorkflowDefinition 1 ──── N StepDefinition          (sablon → lépés-definíciók)
WorkflowDefinition 1 ──── N TransitionDefinition     (sablon → átmenet-definíciók)
WorkflowDefinition 1 ──── N WorkflowInstance         (sablon → futó instance-ok)

StepDefinition N ──── 1 EscalationRule               (lépés → eszkalációs szabály)
StepDefinition 1 ──── N WorkflowTask                 (lépés-sablon → konkrét feladatok)
StepDefinition 1 ──── N TransitionDefinition (from)  (lépés → kimenő átmenetek)
StepDefinition 1 ──── N TransitionDefinition (to)    (lépés → bejövő átmenetek)

WorkflowInstance 1 ──── N WorkflowTask               (instance → feladatok)
WorkflowInstance 1 ──── N WorkflowEvent              (instance → auditnapló)

WorkflowTask 1 ──── N WorkflowEvent                  (feladat → kapcsolódó esemény)

Person 1 ──── N WorkflowTask (assignee)              (személy → hozzárendelt feladatok)
Person 1 ──── N WorkflowTask (delegatedTo)           (személy → delegált feladatok)
Person 1 ──── N DelegationRule (delegator)           (személy → saját delegálásai)
Person 1 ──── N DelegationRule (delegate)            (személy → mások által rá delegált)
```

---

## 16. Nyitott kérdések

1. **Folyamaton belüli párhuzamos ágak:** A jelenlegi modell szekvenciális és elágazó folyamatokat kezel (`TransitionDefinition` feltételekkel), de párhuzamos végrehajtást (pl. „egyszerre kell a közvetlen vezető ÉS a HR jóváhagyása, és mindkettő jóváhagyása után mehet tovább") nem modellez közvetlenül. Szükséges-e `AND-split` / `AND-join` minta? Ez egy önálló `WorkflowGateway` entitást igényelne – BPMN-szerű modellezés felé mutató irány.

2. **Workflow vs. egyszerű státuszgép határa:** Nem minden döntési pont igényel teljes workflow-t. Egy 1 napnál rövidebb szabadságkérelem automatikusan jóváhagyható vezető nélkül – ez már a `StepDefinition.skipCondition`-nel kezelhető, de hol a határ? Javasolt döntési szabály: ha a döntés visszafordíthatatlan (jogviszony-megszüntetés, bérszámfejtés zárolás), teljes workflow szükséges; ha visszavonható és alacsony kockázatú (rövid szabadság, munkaidő-korrekció), elég a közvetlen státuszváltás jóváhagyói értesítéssel.

3. **Közszféra jóváhagyási jogkör delegálhatósága:** A Kttv. 10. §, Mt. 20. § és a vonatkozó közszolgálati jogszabályok korlátozzák, hogy a munkáltatói jogkör-gyakorló milyen döntéseket delegálhat és kire. A `DelegationRule` jelenlegi modellje nem ellenőrzi, hogy az adott `workflowCode`-hoz tartozó döntés jogszabályilag delegálható-e. Ez üzleti szabályként implementálandó – de a metaadat (delegálható-e egy adott `WorkflowDefinition`?) a sablonba kerülhet: `isDelegatable: boolean`.

4. **Döntési határidő jogszabályi kezelése:** Egyes folyamatokra a jogszabály explicit határidőt szab: Mt. 134. § (4) – a szabadságkérelemre vonatkozó döntés határidejét törvény nem szabályozza explicit módon, de a „nem mondta le" helyzet jogi következményekkel jár. Kttv. 48. § (2) – a kinevezésről szóló döntés határideje. Ezeket a `WorkflowDefinition.defaultSlaHours` mezőbe kell kódolni, de külön jelzés szükséges, hogy a határidő jogszabályi-e (`isSlaLegallyMandated: boolean`), mert az `AUTO_REJECT` eszkalációs akció jogszabályi határidő esetén nem alkalmazható.

5. **Több szintű jóváhagyás és összeghatárok:** A `TransitionDefinition.condition` JSON-alapú feltételes logikája elegendő-e összetett szabályokhoz? Pl.: „ha az igényelt összeg > 500 000 Ft, pénzügyi vezető is jóváhagyja; ha > 2 000 000 Ft, ügyvezető is szükséges". Ez dinamikus lépés-hozzáadást igényelne (a sablon statikus lépései nem írják le), ami a jelenlegi modellből hiányzik. Megoldási irányok: (a) több sablon (kis összeg / nagy összeg), (b) dinamikus lépésgeneráló logika az instance létrehozásakor, (c) feltételes `StepDefinition.skipCondition`.

6. **Notifikációs tartalom kezelése:** A `WorkflowEvent` triggerel értesítést, de az értesítés szövegét (tárgy, törzs, csatorna) hol tároljuk? Egy `NotificationTemplate` entitás szükséges, amely `WorkflowDefinition` + `eventType` + `locale` kombinációhoz rendelési szövegsablont. Ez egy újabb cross-cutting concern – valószínűleg önálló `Notification` domén indokolt.

7. **Archiválás és törlés:** Az `ARCHIVED` státuszú instance-ok hosszú megőrzési ideje (akár 50 év) adatmennyiség-problémát vet fel. Cold storage stratégia szükséges: aktív adatbázisból archiváló store-ba migrálás, de a `WorkflowEvent` lekérdezhetőségének megőrzésével. Ez infrastruktúra-döntés, de az entitásmodellben az `archivedAt` és `archiveLocation` mezőkkel előkészíthető.

8. **Auditabilité vs. teljesítmény:** A `WorkflowEvent` append-only napló hosszú futások alatt nagy táblává nőhet. Indexelési stratégia szükséges: `instanceId` + `occurredAt` partícionálás; a „legutóbbi esemény" lekérdezése ne igényeljen teljes tábla-scanelést. Ez a `WorkflowInstance.currentStepId` és `currentAssigneeId` denormalizált mezők tartásával részben kezelt – de a részletes teljesítmény-tesztelés az implementáció korai fázisában szükséges.
