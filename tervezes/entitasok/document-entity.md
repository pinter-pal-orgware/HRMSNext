# Document (Dokumentum) entitás – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer  
> **Verzió:** 0.1 – első tervezet  
> **Utolsó frissítés:** 2025. február  
> **Kapcsolódó dokumentumok:** erintett-torvenyek.md, person-entity.md, employment-entity.md, compensation-entity.md, organization-entity.md, position-entity.md, qualification-entity.md, leave-entity.md, timetracking-entity.md

---

## 1. Az entitás célja és hatóköre

A **Document** entitáscsalád az HRMS-ben keletkező, tárolt és kezelt dokumentumokat nyilvántartja: a jogviszony létesítésétől a megszűnésig keletkező iratokat, a munkáltató által kötelezően kiállítandó dokumentumokat, a munkavállaló által benyújtott okiratokat és a belső szabályzatokat. A magyar munkajog **számos ponton** előírja írásbeli formát, dokumentumkiadási kötelezettséget és iratőrzési szabályokat.

**Miért kritikus a Document entitás?**

| Felhasználási terület | Jogszabály | Hatás |
|---|---|---|---|
| **Munkaszerződés / kinevezés** | Mt. 44–46. §; Kjt. 21–23. §; Púétv.; Eszjtv.; Kit.; Kttv. | A jogviszony írásbeli alapdokumentuma |
| **Munkaszerződés módosítás** | Mt. 58. § | Minden módosítás írásban |
| **Munkaköri leírás** | SZMSZ; belső szabályzat | A munkakör részletes feladatai |
| **Bérjegyzék (payslip)** | Mt. 155. § (2) | Havonta, írásban kiadandó |
| **Munkáltatói igazolás** | Mt. 80. § (2) | Jogviszony megszűnésekor kötelezően kiadandó |
| **Működési bizonyítvány** | Mt. 81. § | Munkavállalói kérésre kiadandó |
| **Kinevezési okirat** | Kjt. 21. §; Púétv.; Kit.; Kttv. | Közszférás jogviszony alapdokumentuma |
| **Felmentési/felmondási okirat** | Mt. 64–86. §; jogállási törvények | Megszüntetés írásbeli közlése |
| **Személyi anyag** | Kjt. 83/A. §; Kit.; Kttv. | Közszféra: személyi anyag vezetése kötelező |
| **Iratőrzés** | Lt. (1995. évi LXVI. tv.); GDPR; ágazati szabályok | Megőrzési kötelezettségek |
| **Okirat-bemutatás** | Mt. 10. § | Munkavállaló bemutatja, munkáltató megtekinti (másolat NEM!) |

**Tervezési alapelvek:**

- **Metaadat-központú modell:** A Document entitás elsősorban a dokumentum **metaadatait** (típus, keletkezés, érvényesség, aláírók, jogszabályi hivatkozás) tárolja. A tényleges fájl (PDF, DOCX, scan) opcionálisan csatolható, de a metaadat a kötelező mag.
- **Polimorf kapcsolódás:** Egy dokumentum többféle entitáshoz kapcsolódhat: Employment-hez (munkaszerződés), Person-höz (személyes okirat), Organization-höz (szabályzat), Position-höz (munkaköri leírás). Ezt a `relatedEntityType` + `relatedEntityId` polimorf FK mintával oldjuk meg.
- **Életciklus-kezelés:** A dokumentumnak státusza van (piszkozat → jóváhagyva → aláírva → kiadva → archiválva → megsemmisítve), és a státuszátmenetek szabályozottak.
- **GDPR és iratőrzés:** Minden dokumentumtípushoz megőrzési idő tartozik, amely a kapcsolódó entitás megőrzési szabályaiból és a levéltári törvényből együttesen adódik.
- **Mt. 10. § tisztelete:** A munkavállaló által bemutatott személyes okiratokról (diploma, személyi igazolvány) a rendszer **csak metaadatot** rögzít; digitális másolat tárolásához külön GDPR jogalap szükséges.

---

## 2. Entitásstruktúra – áttekintés

```
DocumentType (törzsadat) ──── N Document (dokumentum rekord)
Document 1 ──── N DocumentVersion              (verziók)
Document 1 ──── N DocumentSignature            (aláírások)
Document 1 ──── N DocumentDelivery             (kézbesítések)
Document ──── polimorf kapcsolat ──── Employment / Person / Organization / Position / ...

DocumentTemplate (sablonok) ──── DocumentType
```

---

## 3. DocumentType (Dokumentum típus – törzsadat)

### 3.1. Property-k

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `id` | UUID | igen | PK |
| `code` | string | igen | Egyedi kód (pl. `EMPLOYMENT_CONTRACT`, `PAYSLIP`, `TERMINATION_NOTICE`) |
| `name` | string | igen | Megnevezés |
| `category` | enum | igen | Kategória (lásd 3.2.) |
| `subcategory` | enum | nem | Alkategória (lásd 3.3.) |
| `applicableEmploymentTypes` | array[enum] | nem | Mely jogviszony-típusoknál releváns (null = mind) |
| `relatedEntityType` | enum | igen | Melyik entitáshoz kapcsolódik elsődlegesen: `EMPLOYMENT`, `PERSON`, `ORGANIZATION`, `POSITION`, `ORG_UNIT` |
| `isLegallyRequired` | boolean | igen | Jogszabály által kötelezően előírt dokumentum-e |
| `legalBasis` | string | nem | Jogszabályi hivatkozás |
| `requiresSignature` | boolean | igen | Aláírás szükséges-e |
| `signatureType` | enum | nem | Aláírás típusa: `WET_INK` (saját kezű), `ELECTRONIC_QUALIFIED` (minősített e-aláírás), `ELECTRONIC_ADVANCED` (fokozott), `EITHER` |
| `requiresDeliveryProof` | boolean | igen | Kézbesítés igazolása szükséges-e |
| `retentionPeriod` | string | nem | Megőrzési idő szabálya (pl. "jogviszony + 5 év"; "50 év"; "maradandó érték") |
| `retentionLegalBasis` | string | nem | Megőrzés jogszabályi alapja |
| `isConfidential` | boolean | igen | Bizalmas dokumentum-e (hozzáférés korlátozott) |
| `templateId` | UUID (FK) | nem | Dokumentumsablon (ha van előre definiált forma) |
| `isActive` | boolean | igen | Aktív típus-e |

### 3.2. Dokumentum kategóriák (category)

| Kód | Megnevezés | Leírás |
|---|---|---|
| `EMPLOYMENT_ESTABLISHMENT` | Jogviszony létesítés | Munkaszerződés, kinevezés, módosítások |
| `EMPLOYMENT_MODIFICATION` | Jogviszony módosítás | Munkaszerződés módosítás, átsorolás, kinevezés módosítás |
| `EMPLOYMENT_TERMINATION` | Jogviszony megszüntetés/megszűnés | Felmondás, felmentés, közös megegyezés, igazolások |
| `COMPENSATION` | Javadalmazás | Bérjegyzék, bérmegállapító határozat, illetmény-megállapítás |
| `POSITION` | Munkakör | Munkaköri leírás, kinevezés mellékletei |
| `QUALIFICATION_PROOF` | Végzettség igazolása | Oklevél/bizonyítvány bemutatásának metaadatai, elismerési határozat |
| `LEAVE` | Szabadság / távollét | Keresőképtelenségi igazolás, szülési szabadság bejelentés |
| `TIME` | Munkaidő | Munkaidő-nyilvántartás havi zárása, rendkívüli munkaidő elrendelés |
| `ORGANIZATIONAL` | Szervezeti | SZMSZ, belső szabályzatok, szervezeti ábra, közszolgálati szabályzat |
| `REGULATORY` | Jogszabályi / hatósági | Munkaügyi hatósági határozat, DPIA, adatkezelési tájékoztató |
| `PERSONAL_ADMIN` | Személyügyi adminisztráció | Személyi anyag borítólap, nyilatkozatok, meghatalmazás |
| `HEALTH_SAFETY` | Munkavédelem / munkaegészségügy | Munkaköri alkalmassági vélemény, baleseti jegyzőkönyv |
| `TRAINING` | Továbbképzés | Tanulmányi szerződés, képzési igazolás, tanúsítvány |
| `DISCIPLINARY` | Fegyelmi | Figyelmeztető levél, fegyelmi határozat |
| `EXTERNAL_CORRESPONDENCE` | Külső levelezés | NAV, NEAK, bíróság felé/felől |

### 3.3. Alkategóriák – a legfontosabb dokumentum-típusok részletezése

#### 3.3.1. Jogviszony létesítés (EMPLOYMENT_ESTABLISHMENT)

| Kód | Megnevezés | Jogviszony | Jogszabály | Aláírás | Megjegyzés |
|---|---|---|---|---|---|
| `EMPLOYMENT_CONTRACT` | Munkaszerződés | Mt. | Mt. 44–46. § | Mindkét fél | Kötelező tartalom: munkakör, munkavégzés helye, alapbér |
| `CONTRACT_ANNEX` | Munkaszerződés melléklet | Mt. | Mt. | Mindkét fél | Munkaköri leírás, tájékoztatás |
| `EMPLOYER_INFORMATION` | Munkáltatói tájékoztató | Mt. | Mt. 46. § | Munkáltató | 15 napon belül kiadandó: munkarend, bérfizetés napja stb. |
| `APPOINTMENT_DECREE` | Kinevezési okirat | Kjt. | Kjt. 21–23. § | Munkáltató + közalkalmazott | Kötelező tartalom: munkakör, besorolás, illetmény, munkaidő |
| `PUETV_APPOINTMENT` | Kinevezés (köznevelési) | Púétv. | Púétv. 24–26. § | Munkáltató + foglalkoztatott | Pedagógus-munkakör, fokozat, illetmény |
| `ESZJTV_SERVICE_CONTRACT` | Eü. szolgálati munkaszerződés | Eszjtv. | Eszjtv. 2–3. § | Mindkét fél | Speciális tartalom: kötelező 3 hó próbaidő |
| `GOV_APPOINTMENT` | Kinevezés (kormányzati/közszolgálati) | Kit./Kttv./Küt. | Kit. 38–42. §; Kttv. | Munkáltató + tisztviselő | Álláshely, besorolás, illetmény |
| `OATH_DECLARATION` | Esküokmány | Kit./Kttv./Küt. | Kit. 38. § | Tisztviselő | Eskütétel dokumentálása |
| `PROBATION_EVALUATION` | Próbaidő értékelés | Minden | Mt.; Eszjtv. | Munkáltató | Eszjtv.: kötelező 3 hó próbaidő értékelés |
| `GDPR_CONSENT` | GDPR adatkezelési tájékoztató és nyilatkozat | Minden | GDPR 13–14. cikk | Munkavállaló (tudomásulvétel) | Belépéskor kiadandó |

#### 3.3.2. Jogviszony módosítás (EMPLOYMENT_MODIFICATION)

| Kód | Megnevezés | Jogszabály | Megjegyzés |
|---|---|---|---|
| `CONTRACT_AMENDMENT` | Munkaszerződés módosítás | Mt. 58. § | Közös megegyezéssel; kötelezően írásban |
| `APPOINTMENT_AMENDMENT` | Kinevezés módosítás | Kjt.; Púétv.; Kit.; Kttv. | Minden módosítandó elem |
| `RECLASSIFICATION_NOTICE` | Átsorolási értesítés | Kjt. 64. §; Eszjtv. | Fokozatlépés, fizetési osztály változás |
| `POSITION_CHANGE_NOTICE` | Munkakör-változás értesítés | Mt.; minden jogviszony | Belső áthelyezés, előléptetés |
| `SALARY_CHANGE_NOTICE` | Illetmény/bér változás értesítés | Mt.; minden jogviszony | Béremelés, illetménytáblázat-módosítás |
| `WORK_SCHEDULE_CHANGE` | Munkarend módosítás | Mt. 97. § | Munkaidőbeosztás változás közlése |
| `REMOTE_WORK_AGREEMENT` | Távmunka megállapodás | Mt. 196–197/A. § | 2022-es módosítás: írásbeli megállapodás |
| `TRANSFER_NOTICE` | Áthelyezési értesítés | Kit.; Kttv. | Más szervezeti egységbe |

#### 3.3.3. Jogviszony megszüntetés/megszűnés (EMPLOYMENT_TERMINATION)

| Kód | Megnevezés | Jogszabály | Megjegyzés |
|---|---|---|---|
| `TERMINATION_NOTICE_EMPLOYER` | Munkáltatói felmondás | Mt. 66–68. § | Írásban, indoklással; kézbesítés igazolással |
| `TERMINATION_NOTICE_EMPLOYEE` | Munkavállalói felmondás | Mt. 67. § | Írásban; indoklás nem kötelező (Mt.), de Kjt./Kit. – lemondás indoklás nélkül |
| `MUTUAL_TERMINATION` | Közös megegyezés | Mt. 64. § (1) a) | Mindkét fél aláírása |
| `IMMEDIATE_TERMINATION` | Azonnali hatályú felmondás / rendkívüli felmentés | Mt. 78–79. §; Kjt.; Kit. | Indoklás kötelező |
| `DISMISSAL_DECREE` | Felmentési okirat (közszféra) | Kjt.; Púétv.; Kit.; Kttv. | Indoklás + felmentési idő + végkielégítés |
| `EMPLOYER_CERTIFICATE` | Munkáltatói igazolás | Mt. 80. § (2) | **Kötelezően kiadandó** a jogviszony megszűnésekor: jogviszony időtartama, munkakör, bér, levonások |
| `WORK_CERTIFICATE` | Működési bizonyítvány | Mt. 81. § | Kérésre kiadandó: feladatok leírása, értékelés |
| `LEAVE_SETTLEMENT` | Szabadság-elszámolás | Mt. 125. § (2) | Ki nem adott szabadság pénzbeli megváltása |
| `TAX_CERTIFICATE` | Adóigazolás (M30) | Szja tv. | Éves jövedelemigazolás |
| `TB_CERTIFICATE` | TB igazolás | Tbj. | TB jogviszony igazolás |
| `UNEMPLOYMENT_FORM` | Álláskeresési nyilvántartásba vételhez szükséges igazolás | Flt. | Munkaügyi szervnek |

#### 3.3.4. Javadalmazás (COMPENSATION)

| Kód | Megnevezés | Jogszabály | Megjegyzés |
|---|---|---|---|
| `PAYSLIP` | Bérjegyzék | Mt. 155. § (2) | **Havonta, írásban** kiadandó; tartalom: bruttó, levonások, nettó, jogcímek |
| `SALARY_DETERMINATION` | Illetmény-megállapító határozat (közszféra) | Kjt.; Kit.; Kttv.; Púétv. | Besorolás, illetmény összege, pótlékok |
| `BENEFIT_STATEMENT` | Cafeteria/juttatás nyilatkozat | Szja tv. 71. § | Éves cafeteria-választás dokumentálása |
| `WITHHOLDING_ORDER` | Letiltási határozat (bírósági/hatósági) | Vht. (1994. évi LIII. tv.) | Munkabérből történő levonás jogalapja |
| `STUDY_CONTRACT` | Tanulmányi szerződés | Mt. 229. § | Képzési költség + röghöz kötés |

#### 3.3.5. Személyügyi adminisztráció (PERSONAL_ADMIN)

| Kód | Megnevezés | Jogszabály | Megjegyzés |
|---|---|---|---|
| `PERSONAL_FILE_COVER` | Személyi anyag borítólap | Kjt. 83/A. §; Kit.; Kttv. | Közszférában kötelező személyi anyag |
| `EMPLOYEE_DECLARATION` | Munkavállalói nyilatkozat | Mt.; Szja tv. | Adókedvezmény, családi kedvezmény igénylés |
| `TAX_ADVANCE_DECLARATION` | Adóelőleg-nyilatkozat | Szja tv. 48. § | Éves, módosítható |
| `ASSET_DECLARATION` | Vagyonnyilatkozat | Kit. 157–160. §; Kttv. | Közszféra – kötelező egyes pozíciókban |
| `INCOMPATIBILITY_DECLARATION` | Összeférhetetlenségi nyilatkozat | Eszjtv. 4. §; Kit.; Kttv. | Éves nyilatkozat további jogviszonyokról |
| `CONFIDENTIALITY_AGREEMENT` | Titoktartási nyilatkozat | Mt. 8. § | Üzleti titok, személyes adatok |
| `POWER_OF_ATTORNEY` | Meghatalmazás | Ptk. | Munkáltatói jogkör gyakorlás átruházása |
| `CONSENT_FORM` | Hozzájáruló nyilatkozat | GDPR; Mt. | Adatkezelési hozzájárulás (ha nem törvényi jogalap) |
| `VOLUNTARY_OVERTIME_CONSENT` | Önként vállalt többletmunka hozzájárulás | Mt. 109. § (2); Eszjtv. | Írásbeli nyilatkozat |

#### 3.3.6. Munkavédelem és munkaegészségügy (HEALTH_SAFETY)

| Kód | Megnevezés | Jogszabály | Megjegyzés |
|---|---|---|---|
| `OCCUPATIONAL_HEALTH_OPINION` | Munkaköri alkalmassági vélemény | 33/1998. NM r. | Alkalmas / nem alkalmas / feltételesen alkalmas |
| `ACCIDENT_REPORT` | Munkabaleseti jegyzőkönyv | Mvt. 64. §; 5/1993. MüM r. | Súlyos baleset: 24 órán belül bejelentendő |
| `RISK_ASSESSMENT` | Munkavédelmi kockázatértékelés | Mvt. 54. § | Munkakör szintű |
| `SAFETY_INSTRUCTION_RECORD` | Munkavédelmi oktatás igazolás | Mvt. 55. § | Belépéskor + évente + változáskor |
| `PPE_ISSUANCE_RECORD` | Egyéni védőeszköz kiadás igazolás | Mvt. 56. § | Védőeszköz átadás-átvétel |
| `OCCUPATIONAL_DISEASE_REPORT` | Foglalkozási megbetegedés jelentés | Mvt. 65. § | ÁNTSZ felé |

#### 3.3.7. Szervezeti dokumentumok (ORGANIZATIONAL)

| Kód | Megnevezés | Jogszabály | Megjegyzés |
|---|---|---|---|
| `ORGANIZATIONAL_RULES` | Szervezeti és Működési Szabályzat (SZMSZ) | Áht.; Ptk.; belső szabályzat | Szervezeti struktúra, hatáskörök |
| `COLLECTIVE_AGREEMENT` | Kollektív szerződés | Mt. 276–284. § | Munkáltató + szakszervezet |
| `WORK_RULES` | Munkáltatói szabályzat / Belső szabályzat | Mt. | Munkarend, szabadság kiadás rendje |
| `DATA_PROTECTION_POLICY` | Adatvédelmi szabályzat | GDPR; Info tv. | Kötelező DPO kijelölés esetén |
| `DPIA_REPORT` | Adatvédelmi hatásvizsgálat | GDPR 35. cikk | Magas kockázatú adatkezeléshez |
| `EQUAL_OPPORTUNITY_PLAN` | Esélyegyenlőségi terv | Ebktv. 63. § | 50+ fős cégek esetén kötelező |
| `PUBLIC_DATA_REPORT` | Közérdekű adatok közzététele | Info tv. III. fejezet | Közszférás szervezetek |

---

## 4. Document (Dokumentum rekord)

### 4.1. Alapadatok

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `documentTypeId` | UUID (FK) | igen | Dokumentum típus | Technikai |
| `documentNumber` | string | nem | Iktatószám / belső azonosító | Iratkezelési szabályzat |
| `title` | string | igen | Dokumentum címe / megnevezése | – |
| `description` | string | nem | Rövid leírás | – |

### 4.2. Polimorf kapcsolódás

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `relatedEntityType` | enum | igen | Kapcsolódó entitás típusa: `EMPLOYMENT`, `PERSON`, `ORGANIZATION`, `ORG_UNIT`, `POSITION`, `COMPENSATION_ELEMENT`, `LEAVE_REQUEST`, `ABSENCE_EVENT`, `TRAINING_RECORD`, `OVERTIME_RECORD`, `NONE` (önálló dokumentum) |
| `relatedEntityId` | UUID | feltételes | Kapcsolódó entitás azonosítója (null ha `NONE`) |
| `secondaryRelatedEntityType` | enum | nem | Másodlagos kapcsolódás (pl. munkaszerződés → Employment + Person) |
| `secondaryRelatedEntityId` | UUID | nem | Másodlagos entitás azonosítója |

### 4.3. Időbeliség és érvényesség

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `documentDate` | date | igen | Dokumentum kelte (kiállítás dátuma) | – |
| `effectiveDate` | date | nem | Hatályba lépés dátuma (ha eltér a keltétől) | Mt. – pl. munkaszerződés módosítás: „hatályos ... naptól" |
| `expiryDate` | date | nem | Érvényesség vége (ha lejárattal rendelkezik) | – |
| `supersededByDocumentId` | UUID (FK) | nem | Melyik dokumentum váltotta fel (módosítás láncolat) | – |
| `supersedesDocumentId` | UUID (FK) | nem | Melyik dokumentumot váltja fel | – |

### 4.4. Tartalom és fájlkezelés

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `hasDigitalContent` | boolean | igen | Van-e digitális tartalom (fájl) csatolva |
| `contentType` | enum | nem | Fájl típus: `PDF`, `DOCX`, `IMAGE_SCAN`, `XML`, `OTHER` |
| `fileReference` | string | nem | Fájl hivatkozás (DMS azonosító vagy fájlrendszer útvonal) |
| `fileSize` | integer | nem | Fájl méret (byte) |
| `fileHash` | string | nem | Fájl hash (integritás ellenőrzés – SHA-256) |
| `isOriginalDigital` | boolean | nem | Eredetileg digitálisan keletkezett-e (vs. scan) |
| `physicalLocation` | string | nem | Fizikai irat tárolási helye (ha papír alapú) |
| `physicalArchiveBox` | string | nem | Irattári doboz / polc azonosító | 

### 4.5. Hozzáférés és bizalmasság

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `confidentialityLevel` | enum | igen | Bizalmassági szint: `PUBLIC` (nyilvános – közérdekű adat), `INTERNAL` (belső), `CONFIDENTIAL` (bizalmas), `STRICTLY_CONFIDENTIAL` (szigorúan bizalmas) |
| `accessRestriction` | string | nem | Hozzáférési korlátozás leírása (pl. „csak HR vezető és bérszámfejtő") |
| `containsPersonalData` | boolean | igen | Tartalmaz-e személyes adatot (GDPR releváns) |
| `containsSpecialCategoryData` | boolean | nem | Tartalmaz-e különleges adatot (GDPR 9. cikk – egészségügyi, szakszervezeti) |
| `gdprDataCategories` | array[string] | nem | GDPR adatkategóriák listája (pl. ["egészségügyi", "pénzügyi"]) |

### 4.6. Életciklus

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `status` | enum | igen | Dokumentum státusza (lásd 4.7.) |
| `retentionEndDate` | date | nem | Megőrzési idő vége (számított vagy manuális) |
| `disposalMethod` | enum | nem | Megsemmisítés módja: `SHRED` (iratmegsemmisítő), `DIGITAL_DELETE` (digitális törlés), `ARCHIVE_TRANSFER` (levéltárba átadás) |
| `disposedAt` | datetime | nem | Megsemmisítés/átadás dátuma |
| `disposedBy` | string | nem | Megsemmisítő/átadó személy |

### 4.7. Dokumentum státuszok

| Kód | Megnevezés | Leírás |
|---|---|---|
| `DRAFT` | Piszkozat | Szerkesztés alatt, nem végleges |
| `PENDING_REVIEW` | Felülvizsgálatra vár | Jóváhagyási folyamatban |
| `APPROVED` | Jóváhagyva | Tartalom véglegesítve, aláírásra vár |
| `SIGNED` | Aláírva | Minden szükséges aláírás megtörtént |
| `DELIVERED` | Kézbesítve | Az érintett(ek)nek átadva |
| `ACTIVE` | Hatályos | Érvényes, hatályban lévő dokumentum |
| `SUPERSEDED` | Felváltva | Újabb dokumentum lépett a helyébe |
| `EXPIRED` | Lejárt | Az érvényességi idő letelt |
| `ARCHIVED` | Irattárazva | Aktív használatból kivonva, megőrzés alatt |
| `PENDING_DISPOSAL` | Megsemmisítésre vár | Megőrzési idő lejárt, selejtezési eljárás alatt |
| `DISPOSED` | Megsemmisítve | Fizikailag és/vagy digitálisan megsemmisítve |

### 4.8. Audit mezők

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `createdAt` | datetime | igen | Rekord létrehozása |
| `createdBy` | string | igen | Létrehozó |
| `updatedAt` | datetime | igen | Utolsó módosítás |
| `updatedBy` | string | igen | Módosító |
| `version` | integer | igen | Aktuális verzió sorszáma |

---

## 5. DocumentVersion (Dokumentum verzió)

A dokumentum több verziót is tartalmazhat (pl. munkaszerződés-tervezet → végleges → aláírt scan). Minden verzió önálló fájlt tartalmazhat.

### 5.1. Property-k

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `id` | UUID | igen | PK |
| `documentId` | UUID (FK) | igen | Hivatkozás a dokumentumra |
| `versionNumber` | integer | igen | Verzió sorszáma (1, 2, 3...) |
| `versionLabel` | string | nem | Verzió címke (pl. „Tervezet v2", „Aláírt végleges") |
| `fileReference` | string | nem | Fájl hivatkozás |
| `contentType` | enum | nem | Fájl típus |
| `fileSize` | integer | nem | Fájl méret |
| `fileHash` | string | nem | SHA-256 hash |
| `changeDescription` | string | nem | Változás leírása |
| `createdAt` | datetime | igen | Létrehozás |
| `createdBy` | string | igen | Létrehozó |

---

## 6. DocumentSignature (Aláírás)

### 6.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `documentId` | UUID (FK) | igen | Hivatkozás a dokumentumra | Technikai |
| `signerRole` | enum | igen | Aláíró szerepe: `EMPLOYER` (munkáltató), `EMPLOYEE` (munkavállaló), `WITNESS` (tanú), `APPROVER` (jóváhagyó), `HR_OFFICER` (HR ügyintéző), `LEGAL_OFFICER` (jogtanácsos), `UNION_REP` (szakszervezeti képviselő) | – |
| `signerPersonId` | UUID (FK) | nem | Aláíró személy (Person entitás, ha a rendszerben nyilvántartott) | – |
| `signerName` | string | igen | Aláíró neve | – |
| `signerTitle` | string | nem | Aláíró beosztása | – |
| `signatureType` | enum | igen | Aláírás típusa: `WET_INK`, `ELECTRONIC_QUALIFIED`, `ELECTRONIC_ADVANCED`, `DIGITAL_SIMPLE` | eIDAS rendelet; 2001. évi XXXV. tv. |
| `signatureDate` | datetime | nem | Aláírás dátuma | – |
| `signatureStatus` | enum | igen | `PENDING` (aláírásra vár), `SIGNED` (aláírva), `REFUSED` (megtagadta), `EXPIRED` (határidő lejárt) | – |
| `signatureReference` | string | nem | E-aláírás referencia / tanúsítvány hivatkozás | – |
| `signatureDeadline` | datetime | nem | Aláírás határideje | – |
| `refusalReason` | string | nem | Megtagadás indoka (ha REFUSED) | – |

**Megjegyzések:**
- A **munkaszerződés** érvényességéhez mindkét fél aláírása szükséges (Mt. 44. §).
- A **kinevezési okiratot** a munkáltató állítja ki, a közalkalmazott/köztisztviselő aláírásával veszi tudomásul (Kjt. 21. §; Kit. 38. §).
- A **felmondás/felmentés** egyoldalú jognyilatkozat: csak a nyilatkozó fél aláírása szükséges, de a **kézbesítés igazolása** kötelező (Mt. 24. §).
- A **minősített e-aláírás** (eIDAS rendelet) a saját kezű aláírással egyenértékű – a rendszer támogassa mindkettőt.

---

## 7. DocumentDelivery (Kézbesítés)

Számos munkajogi dokumentum érvényességéhez a kézbesítés igazolása szükséges.

### 7.1. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Forrás |
|---|---|---|---|---|
| `id` | UUID | igen | PK | Technikai |
| `documentId` | UUID (FK) | igen | Hivatkozás a dokumentumra | Technikai |
| `recipientPersonId` | UUID (FK) | nem | Címzett (Person, ha a rendszerben) | – |
| `recipientName` | string | igen | Címzett neve | – |
| `deliveryMethod` | enum | igen | Kézbesítés módja (lásd 7.2.) | – |
| `deliveryDate` | datetime | nem | Kézbesítés dátuma | – |
| `deliveryStatus` | enum | igen | `PENDING`, `DELIVERED`, `REFUSED`, `UNDELIVERABLE`, `UNKNOWN` | – |
| `receiptConfirmation` | boolean | nem | Átvétel igazolva-e | – |
| `receiptDate` | datetime | nem | Átvétel igazolásának dátuma | – |
| `trackingNumber` | string | nem | Postai tértivevény száma / kézbesítési azonosító | – |
| `notes` | string | nem | Megjegyzés (pl. „tanú jelenlétében átadva") | – |

### 7.2. Kézbesítési módok (deliveryMethod)

| Kód | Megnevezés | Leírás | Bizonyító erő |
|---|---|---|---|
| `IN_PERSON` | Személyes átadás | Aláírással igazolt átvétel | Erős – érintett aláírása |
| `IN_PERSON_WITNESS` | Személyes átadás tanú jelenlétében | Ha a címzett megtagadja az átvételt | Erős – tanú igazolás |
| `REGISTERED_MAIL` | Tértivevényes postai levél | Klasszikus jogi kézbesítés | Erős – postai igazolás |
| `ELECTRONIC_SIGNED` | E-mailben, e-aláírással | Minősített e-aláírással ellátott dokumentum | Erős (ha minősített) |
| `ELECTRONIC_PORTAL` | Önkiszolgáló portálon elérhetővé tétel | A munkavállaló elektronikusan veszi át (bérjegyzék) | Közepes – megnyitás igazolása |
| `ELECTRONIC_EMAIL` | E-mailben, egyszerű | Nem ajánlott jogi dokumentumokhoz | Gyenge |
| `INTERNAL_MAIL` | Belső posta / irat | Szervezeten belüli kézbesítés | Közepes |

**Megjegyzések:**
- Az Mt. 24. § szerint a jognyilatkozatot **közöltnek kell tekinteni**, ha az a címzett tudomására jutott. A kézbesítés igazolása a jognyilatkozat hatályosulásának feltétele.
- A **felmondás** kézbesítésénél különös figyelmet kell fordítani a „közlés" időpontjára, mert ettől indul a felmondási idő (Mt. 68. §).
- Ha a címzett **megtagadja az átvételt**, az Mt. 24. § (2) szerint a jognyilatkozatot közöltnek kell tekinteni, ha a közlés a cím jogszabályoknak megfelelő módon megtörtént. A tanú jelenlétében történő átadás kísérlet ezt dokumentálja.

---

## 8. DocumentTemplate (Dokumentumsablon)

### 8.1. Property-k

| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `id` | UUID | igen | PK |
| `documentTypeId` | UUID (FK) | igen | Dokumentum típus |
| `organizationId` | UUID (FK) | igen | Szervezet (a sablon szervezet-specifikus) |
| `name` | string | igen | Sablon megnevezése |
| `templateVersion` | string | igen | Sablon verzió (pl. „2025.01") |
| `templateFormat` | enum | igen | Formátum: `DOCX`, `PDF_FORM`, `HTML`, `XML` |
| `templateFileReference` | string | igen | Sablon fájl hivatkozás |
| `placeholders` | JSON | nem | Mezőhelyettesítők listája (pl. `{{employee_name}}`, `{{start_date}}`, `{{base_salary}}`) |
| `applicableFromDate` | date | igen | Sablon érvényessége kezdete |
| `applicableToDate` | date | nem | Sablon érvényessége vége |
| `isDefault` | boolean | igen | Alapértelmezett sablon-e az adott típushoz |
| `legalReviewedAt` | datetime | nem | Utolsó jogi felülvizsgálat dátuma |
| `legalReviewedBy` | string | nem | Felülvizsgáló jogtanácsos |
| `isActive` | boolean | igen | Aktív-e |

**Megjegyzések:**
- A dokumentumsablonok **szervezet-specifikusak**: egy önkormányzat más munkaszerződés-sablont használ, mint egy versenyszférás Kft.
- A sablonokban **mezőhelyettesítők** (placeholders) szerepelnek, amelyeket a rendszer a Person, Employment, Compensation stb. entitásokból tölt ki.
- A sablonok **jogi felülvizsgálata** évente vagy jogszabályváltozáskor szükséges – a `legalReviewedAt` mező ezt dokumentálja.

---

## 9. Személyi anyag – közszférás sajátosság

### 9.1. Szabályozás

A közszférában (Kjt. 83/A. §; Kit.; Kttv.) a személyi anyag vezetése **kötelező**. A személyi anyag tartalmazza a foglalkoztatotti jogviszonnyal összefüggő összes dokumentumot, és az érintettnek betekintési joga van.

### 9.2. Személyi anyag tartalma (tipikus)

| Iratcsoport | Dokumentum típusok | Jogszabály |
|---|---|---|
| **I. csoport – Személyi adatok** | Személyi adatlap, okmánymásolatok metaadatai, fénykép (ha van) | Kjt. 83/A. § |
| **II. csoport – Jogviszony létesítés** | Kinevezési okirat, esküokmány, GDPR nyilatkozat | Kjt. 21. §; Kit. 38. § |
| **III. csoport – Jogviszony módosítások** | Kinevezés módosítások, átsorolások, munkakör-változások | Kjt.; Kit.; Kttv. |
| **IV. csoport – Végzettségek, képesítések** | Végzettség-ellenőrzés metaadatai, szakvizsga, nyelvvizsga | Kjt. 61. §; Kit. |
| **V. csoport – Teljesítményértékelés, minősítés** | Teljesítményértékelő lap, pedagógus-minősítés eredménye | Kit.; Kttv.; Púétv. |
| **VI. csoport – Javadalmazás** | Illetmény-megállapítás, pótlékok, juttatások | Kjt.; Kit.; Kttv. |
| **VII. csoport – Fegyelmi, kártérítés** | Fegyelmi határozat, kártérítési határozat | Kjt.; Kit.; Kttv. |
| **VIII. csoport – Szabadság, távollét** | Szülési szabadság igazolás, GYED/GYES határozat | Mt.; Tbj. |
| **IX. csoport – Továbbképzés** | Továbbképzési igazolások, tanulmányi szerződés | Kjt.; Kit.; Kttv.; Púétv. |
| **X. csoport – Jogviszony megszüntetés** | Felmentési okirat, munkáltatói igazolás | Kjt.; Kit.; Kttv. |

### 9.3. Modellezés az HRMS-ben

A személyi anyag nem önálló entitás, hanem a **Document entitás szűrt nézete**: minden `relatedEntityType` = `EMPLOYMENT` dokumentum, amelynek `Employment.employmentType` közszférás jogviszony. A rendszer automatikusan képes generálni a személyi anyag tartalomjegyzéket az iratcsoportok szerinti rendezéssel.

---

## 10. Dokumentum-generálás

### 10.1. Automatikusan generálható dokumentumok

| Dokumentum | Kiváltó esemény | Input entitások | Sablon |
|---|---|---|---|
| **Munkaszerződés / kinevezés** | Employment létrehozása | Person + Employment + Position + Compensation | EMPLOYMENT_CONTRACT / APPOINTMENT_DECREE sablon |
| **Munkaszerződés módosítás** | Employment módosítása (EmploymentHistory) | Person + Employment (régi + új értékek) | CONTRACT_AMENDMENT sablon |
| **Átsorolási értesítés** | Kjt./Eszjtv. fokozatlépés (automatikus) | Employment (régi + új fokozat/illetmény) | RECLASSIFICATION_NOTICE sablon |
| **Bérjegyzék** | Havi bérszámfejtés lezárása | Employment + Compensation + TimeTracking + Leave összesítés | PAYSLIP sablon |
| **Munkáltatói igazolás** | Employment terminationDate beállítása | Person + Employment + Compensation (utolsó bér) + Leave (maradék) | EMPLOYER_CERTIFICATE sablon |
| **Szabadság-elszámolás** | Jogviszony megszűnés | LeaveEntitlement (maradék) + Compensation (távolléti díj) | LEAVE_SETTLEMENT sablon |
| **Illetmény-megállapítás** | Compensation változás | Employment + CompensationElement (új) | SALARY_DETERMINATION sablon |

### 10.2. Sablon mezőhelyettesítők (példa: munkaszerződés)

```
{{employer_name}}           ← Organization.name
{{employer_tax_number}}     ← Organization.taxNumber
{{employer_address}}        ← Organization.headquarterAddress
{{employer_rep_name}}       ← OrgUnit.employerRightsPersonId → Person.name
{{employer_rep_title}}      ← Position.positionTitle (a jogkör-gyakorlóé)
{{employee_name}}           ← Person.familyName + givenNames
{{employee_birth_date}}     ← Person.birthDate
{{employee_birth_place}}    ← Person.birthPlace
{{employee_mother_name}}    ← Person.motherBirthName
{{employee_address}}        ← Person.permanentAddress
{{employee_tax_id}}         ← Person.taxId
{{employee_taj}}            ← Person.socialSecurityNumber
{{position_title}}          ← Position.positionTitle
{{work_place}}              ← Employment.workPlace
{{start_date}}              ← Employment.startDate
{{contract_duration}}       ← Employment.contractDuration
{{probation_end_date}}      ← Employment.probationEndDate
{{weekly_hours}}            ← Employment.weeklyHours
{{base_salary}}             ← CompensationElement (BASE_SALARY).amount
{{work_schedule}}           ← WorkScheduleTemplate.name
{{collective_agreement}}    ← Organization.collectiveBargainingAgreement
{{document_date}}           ← Document.documentDate
```

---

## 11. GDPR adatkezelési kategorizáció

| Kategória | Dokumentum típusok | Jogalap | Megőrzési idő |
|---|---|---|---|
| **Jogviszony okiratok** | Munkaszerződés, kinevezés, módosítások | Jogi kötelezettség (6(1)c) + Szerződés (6(1)b) | Jogviszony + 5 év (Art. elévülés); közszféra: jogszabály szerinti |
| **Megszüntetési okiratok** | Felmondás, felmentés, igazolások | Jogi kötelezettség (6(1)c) | Jogviszony + 5 év; peres ügy: lezárás + 5 év |
| **Bérjegyzék** | Payslip | Jogi kötelezettség (6(1)c) – Mt. 155. § | Jogviszony + 8 év (Szt.) |
| **Munkavédelmi dokumentumok** | Baleseti jegyzőkönyv, alkalmassági vélemény | Jogi kötelezettség (6(1)c) – Mvt. | Baleseti: 5 év; alkalmassági: jogviszony + 5 év; foglalkozási betegség: 40 év |
| **Személyi nyilatkozatok** | Adóelőleg, vagyonnyilatkozat, összeférhetetlenségi | Jogi kötelezettség (6(1)c) | Érvényesség + 5 év |
| **Szervezeti dokumentumok** | SZMSZ, kollektív szerződés, szabályzatok | Nem személyes adat | Hatályvesztés + 5 év; maradandó értékű: levéltárba |
| **Egészségügyi tartalmú** | Munkaköri alkalmassági, foglalkozási betegség | GDPR 9. cikk – különleges adat | Fokozott védelem; foglalkozási betegség: 40 év |

### 11.1. Iratőrzési szabályok összefoglalása

| Irat típus | Minimális megőrzési idő | Jogszabály |
|---|---|---|
| Munkaszerződés / kinevezés | Jogviszony + 50 év (ha nyugdíjjogosultsághoz szükséges) | Tbj.; Lt. |
| Bérszámfejtési bizonylatok | 8 év | Szt. 169. § |
| Adóbevallás, adóigazolás | 5 év (elévülés) | Art. |
| TB bejelentés, igazolás | 5 év | Tbj. |
| Munkabaleseti jegyzőkönyv | 5 év (súlyos: 10 év) | 5/1993. MüM r. |
| Foglalkozási betegség dokumentáció | 40 év | Mvt.; NNK ajánlás |
| Munkaköri alkalmassági vélemény | Jogviszony + 5 év (expozíció esetén: 40 év) | 33/1998. NM r. |
| Munkaidő-nyilvántartás | 3 év (elévülés) | Mt. 287. § |
| Vagyonnyilatkozat | A jogszabályban meghatározott ideig | Kit.; Kttv. |
| Személyi anyag (közszféra) | 75 év a jogviszony megszűnésétől (levéltári törvény) | Lt. 12–13. § |
| Maradandó értékű iratok | Korlátlan (levéltárba átadandó) | Lt. 9. § |

---

## 12. Hozzáférési szintek

| Szerep | Document (általános) | Bérjegyzék | Személyi anyag | Szervezeti dok. | Munkavédelmi |
|---|---|---|---|---|---|
| **Érintett munkavállaló** | Saját jogviszony dokumentumai: olvasás | Saját: olvasás, letöltés | Saját: betekintés (Kjt. 83/A. §) | Hatályos belső szabályzatok: olvasás | Saját alkalmassági: olvasás |
| **Közvetlen vezető** | Beosztottak: korlátozott olvasás (nem bér, nem személyi anyag) | **Nincs** | **Nincs** (kivéve: ha a vezető a munkáltatói jogkör gyakorlója) | Olvasás | Beosztottak baleseti: olvasás |
| **HR ügyintéző** | Teljes olvasás/írás a hatáskörben | Olvasás | Teljes olvasás/írás | Olvasás/írás | Olvasás/írás |
| **Bérszámfejtő** | Bér-releváns dokumentumok olvasás | Teljes olvasás/írás (generálás) | Nem | – | – |
| **Jogtanácsos** | Jogi dokumentumok: teljes | Olvasás | Olvasás | Teljes olvasás/írás | Olvasás |
| **Munkavédelmi felelős** | – | – | – | Munkavédelmi szabályzat: olvasás/írás | Teljes olvasás/írás |
| **DPO** | GDPR-releváns dokumentumok: olvasás | – | – | Adatvédelmi: teljes | – |
| **Rendszeradminisztrátor** | Törzsadat karbantartás (DocumentType, DocumentTemplate) | – | – | – | – |

---

## 13. Integrációs pontok

| Külső rendszer | Irány | Érintett adatok | Cél |
|---|---|---|---|
| **Dokumentumkezelő rendszer (DMS)** | kétirányú | Fájlok tárolása, verziózása, keresése | A Document entitás metaadatot tárol, a DMS a fájlt |
| **E-aláírás szolgáltató** | kétirányú | Aláírás kérés → aláírt dokumentum visszavétel | DocumentSignature.signatureReference |
| **Bérszámfejtő modul (payroll)** | bejövő | Bérjegyzék adatok → PAYSLIP dokumentum generálás | Havi automatikus generálás |
| **Önkiszolgáló portál** | kimenő | Bérjegyzék kézbesítés, munkaszerződés letöltés, nyilatkozat kitöltés | ELECTRONIC_PORTAL kézbesítés |
| **NAV** | kimenő | Munkáltatói igazolás, adóigazolás exportálása | Hatósági adatszolgáltatás |
| **Levéltár** | kimenő | Maradandó értékű iratok átadása | Lt. 9. § – levéltári átadás |
| **Nyomtató / postázás** | kimenő | Papír alapú dokumentumok nyomtatása, borítékozása | Tértivevényes kézbesítés |
| **OCR rendszer** | bejövő | Bescannelt papír dokumentumok digitalizálása | Régi iratok archiválása |

---

## 14. Üzleti szabályok és validációk

| Szabály | Leírás | Jogszabály |
|---|---|---|
| Munkaszerződés kötelező tartalmi elemek | A EMPLOYMENT_CONTRACT típusú dokumentum generálásakor ellenőrzés: van-e munkakör, munkavégzési hely, alapbér megadva | Mt. 45. § (1) |
| Aláírás teljessége | A dokumentum `SIGNED` státuszba csak akkor kerülhet, ha minden szükséges DocumentSignature `SIGNED` állapotban van | Mt. 44. § – mindkét fél aláírása |
| Kézbesítés igazolása | Felmondás/felmentés típusú dokumentum `ACTIVE` státuszba csak kézbesítés-igazolás (DocumentDelivery.deliveryStatus = `DELIVERED`) után kerülhet | Mt. 24. § |
| Bérjegyzék havi generálás | Minden aktív Employment-hez havonta kell PAYSLIP típusú dokumentumot generálni | Mt. 155. § (2) |
| Munkáltatói igazolás kötelező kiadás | Employment termination → automatikus EMPLOYER_CERTIFICATE dokumentum generálás | Mt. 80. § (2) |
| Sablonverzió érvényesség | Dokumentum generáláskor az adott dátumhoz érvényes sablon-verziót kell használni | Jogi megfelelőség |
| Megőrzési idő betartása | Dokumentum `DISPOSED` státuszba csak `retentionEndDate` lejárta után kerülhet | GDPR; Lt.; Szt. |
| Személyes adatot tartalmazó dokumentum törlése | Ha a megőrzési idő lejárt és nincs aktív jogalap → törlési kötelezettség | GDPR 17. cikk |
| Felváltott dokumentum archiválása | Ha `supersededByDocumentId` kitöltött → automatikus `SUPERSEDED` státusz + archiválás | Változáskezelés |

---

## 15. Automatikus események és figyelmeztetések

| Esemény | Kiváltó feltétel | Akció |
|---|---|---|
| **Munkaszerződés/kinevezés generálás** | Employment létrehozása (`ACTIVE` státuszba lépés) | Dokumentum sablon kiválasztása, mezők kitöltése, aláírási workflow indítása |
| **Bérjegyzék generálás** | Havi bérszámfejtés lezárása | PAYSLIP automatikus generálás + portálon elérhetővé tétel |
| **Átsorolási értesítés** | EmploymentHistory rekord (RECLASSIFICATION changeReason) | RECLASSIFICATION_NOTICE generálás |
| **Munkáltatói igazolás** | Employment.status → `TERMINATED` | EMPLOYER_CERTIFICATE automatikus generálás |
| **Megőrzési idő lejárat** | Document.retentionEndDate elérése | Értesítés: selejtezési eljárás indítása szükséges |
| **Aláírási emlékeztető** | DocumentSignature.signatureStatus = `PENDING` és signatureDeadline közelít | Emlékeztető az aláírónak |
| **Sablon felülvizsgálat** | DocumentTemplate.legalReviewedAt + 12 hónap | Emlékeztető a jogtanácsosnak |
| **Munkavédelmi oktatás lejárat** | SAFETY_INSTRUCTION_RECORD + 12 hónap | Emlékeztető a munkavédelmi felelősnek |

---

## 16. Kapcsolódó entitások – teljes kép

```
DocumentType 1 ──── N Document                  (típus → dokumentumok)
DocumentTemplate 1 ──── N DocumentType           (sablon → típusok)

Document 1 ──── N DocumentVersion                (verziók)
Document 1 ──── N DocumentSignature              (aláírások)
Document 1 ──── N DocumentDelivery               (kézbesítések)
Document ──── Document (supersededBy)             (felváltási láncolat – self-ref)

Document ──── Employment (polimorf FK)            (jogviszony-dokumentumok)
Document ──── Person (polimorf FK)                (személyes okiratok metaadatai)
Document ──── Organization (polimorf FK)          (szervezeti dokumentumok)
Document ──── Position (polimorf FK)              (munkaköri leírás)
Document ──── OrgUnit (polimorf FK)               (szervezeti egység szabályzatok)
Document ──── CompensationElement (polimorf FK)   (bérmegállapítás)
Document ──── LeaveRequest (polimorf FK)          (szabadság-dokumentumok)
Document ──── AbsenceEvent (polimorf FK)          (keresőképtelenségi igazolás)
Document ──── TrainingRecord (polimorf FK)        (képzési igazolás, tanulmányi szerződés)
Document ──── OvertimeRecord (polimorf FK)        (rendkívüli munkaidő elrendelés)
```

---

## 17. Nyitott kérdések

1. **DMS integráció vs. beépített fájlkezelés:** Az HRMS saját fájltárolást használjon (blob storage), vagy külső DMS-re (pl. SharePoint, Alfresco, M-Files) delegálja a fájlkezelést? Az előbbi egyszerűbb, az utóbbi skálázhatóbb és jobb kereshetőséget biztosít.
2. **E-aláírás integráció:** Milyen e-aláírás szolgáltatóval integráljon a rendszer (pl. Microsec, NetLock, DocuSign)? A minősített e-aláírás követelmény a közszférában?
3. **Papír-digitál kettősség:** A jogszabályok sok helyen „írásbeli" formát követelnek meg, amely a bírói gyakorlat szerint papír alapú. A 2023-as Mt. módosítás (elektronikus dokumentum) mennyiben oldotta fel ezt? A rendszernek kell-e támogatnia a papír és digitális példány párhuzamos kezelését?
4. **OCR és automatikus adatkinyerés:** A bescannelt dokumentumokból (pl. régi papír alapú munkaszerződések) automatikus adatkinyerés (OCR + NLP) szükséges-e a migrációhoz?
5. **Dokumentum kereshetőség:** Szükséges-e teljes szöveges keresés a dokumentumok tartalmában (full-text search), vagy elegendő a metaadat-alapú keresés?
6. **Többnyelvű sablonok:** Külföldi munkavállalók esetén kétnyelvű (magyar + angol/eredeti nyelv) munkaszerződés sablon szükséges?
7. **Levéltári átadás:** A közszférás szervezetek (költségvetési szervek) iratainak levéltári átadása (Lt. 12. §) automatikus exportot igényel? Milyen formátumban (PDF/A, XML)?
8. **Dokumentum-workflow motor:** Az aláírási és jóváhagyási folyamat beépített workflow legyen, vagy külső BPM rendszerre (pl. Camunda, Flowable) támaszkodjon?
