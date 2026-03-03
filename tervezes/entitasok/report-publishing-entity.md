# Report Publishing domén – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer – magyar jogszabályi környezet  
> **Verzió:** 0.1 – első tervezet  
> **Utolsó frissítés:** 2025. február  
> **Kapcsolódó dokumentum:** hrms-er-osszefoglalo.md, analytics-query-entity.md, document-entity.md

---

## 1. A domén célja és hatóköre

A **Report Publishing domén** az HRMS riport- és dokumentumgenerálási rétege. Nem azonos az Analytics / Query doménnel: míg az adatokat teszi lekérdezhetővé, ez a domén formázott, verziózott, auditálható outputokat állít elő – a nyers adatból emberi olvasásra alkalmas dokumentumot.

### 1.1. A két kimeneti kategória és szétválasztásuk indoka

A tervezési alapdöntés: az **analitikus riport** és a **nyomtatvány** azonos adatforrásból táplálkozhat, de teljesen különböző gyártási folyamatot és governance-t igényel.

| Szempont | Analitikus riport | Nyomtatvány / Official document |
|---|---|---|
| **Elsődleges cél** | Elemzés, döntéstámogatás, export | Aláírás, iktatás, jogi bizonyítás |
| **Tipikus formátum** | XLSX, CSV, PDF (táblázatos) | PDF (pixel-perfect), DOCX |
| **Tartalom** | Sok sor, szűrhető, aggregált | 1 személy / 1 ügy / 1 időpont |
| **Jogi státusz** | Belső adat | Okirat-értékű dokumentum |
| **Template kötöttség** | Rugalmas, paraméterezett | Kötött, jogszabálytól vezérelt |
| **Megőrzési kötelezettség** | Futtatási napló (1–5 év) | Személyi irat (akár 50 év) |
| **Verziókritikusság** | Alacsony (legújabb sablon) | Magas (melyik verzióval készült?) |
| **Példák** | Havi létszámriport, túlóra-összesítő | Munkaszerződés, igazolás, kinevezés |

Mindkét kategóriát ugyanaz az entitásmodell kezeli – a `ReportTemplate.category` mező különbözteti meg őket, de a pipeline és a governance azonos struktúrán fut.

### 1.2. Pipeline – Template → Render → Publish

```
Analytics / Query BC
  (Report DB snapshots, asOfDate query)
           │
           │  render model (JSON) összeállítása
           ▼
    ReportDataContract
  (séma-ellenőrzés, paraméter-validáció,
   security profile érvényesítése)
           │
           │  adatkötés
           ▼
  ReportTemplateVersion
  (HTML+CSS / DOCX tartalom + komponensek + branding)
           │
           │  render engine
           ▼
       ReportJob
  (aszinkron végrehajtás, státuszkezelés, hibakezelés)
           │
           │  output mentés
           ▼
      ReportOutput
  (generált fájl + checksum + audit)
           │
           │  archiválás
           ▼
    Document Management BC
  (végleges tárolás, Person/Employment/Case linkek)
```

### 1.3. A domén entitásai

| Entitás | Típus | Leírás |
|---|---|---|
| **ReportTemplate** | Törzsadat | Egy riport/nyomtatvány típusának logikai definíciója |
| **ReportTemplateVersion** | Törzsadat | A sablon verzionált tartalma és eszközei |
| **ReportDataContract** | Törzsadat | A sablon adatigényének stabil API-leírása |
| **ReportComponent** | Törzsadat | Újrafelhasználható UI-komponens (fejléc, aláírásmező stb.) |
| **BrandingProfile** | Törzsadat | Szervezet-specifikus arculati csomag |
| **ReportJob** | Operatív | Egy generálási kérés és végrehajtása |
| **ReportOutput** | Operatív | A kész generált dokumentum rekordja |

---

## 2. ReportTemplate entitás

### 2.1. Az entitás célja

A **ReportTemplate** egy riport vagy nyomtatvány típusának logikai definíciója – a „mi ez a dokumentum" fogalmi szintje. Nem tartalmaz tényleges tartalmat (az a `ReportTemplateVersion` feladata), csak a típus metaadatait, kategorizálását, hatályosságát és a hozzá tartozó verziók közötti navigációt. Ez az entitás stabil: ritkán változik; a változásokat az alá tartozó verziók hordozzák.

Tipikus `ReportTemplate` rekordok:

| Kód | Kategória | Leírás | Kötelező jogalap |
|---|---|---|---|
| `EMPLOYMENT_CERTIFICATE` | `PRINT_DOCUMENT` | Munkáltatói igazolás | Mt. 46. § – kérésre kötelező |
| `INCOME_CERTIFICATE` | `PRINT_DOCUMENT` | Jövedelemigazolás | Mt. 46. §; hiteligényléshez |
| `APPOINTMENT_DECREE` | `PRINT_DOCUMENT` | Kinevezési okirat (közszféra) | Kttv. 44. §; Kjt. 21. § |
| `EMPLOYMENT_CONTRACT` | `PRINT_DOCUMENT` | Munkaszerződés | Mt. 44. § (3) – kötelező írásban |
| `TERMINATION_NOTICE` | `PRINT_DOCUMENT` | Felmondólevél | Mt. 65. § (3) – kötelező írásban, indoklással |
| `LEAVE_BALANCE_STATEMENT` | `PRINT_DOCUMENT` | Szabadságegyenleg-igazolás | Mt. 122. §; jogviszony megszűnésekor |
| `DISCIPLINARY_DECISION` | `PRINT_DOCUMENT` | Fegyelmi határozat | Kttv. 166. §; Kjt. 47. § |
| `PERFORMANCE_REVIEW_FORM` | `PRINT_DOCUMENT` | Teljesítményértékelési/minősítési lap | Kttv. 131. §; Kjt. 66. § |
| `HEADCOUNT_REPORT` | `ANALYTICS_REPORT` | Létszámriport | Belső; KSH OSAP |
| `OVERTIME_SUMMARY` | `ANALYTICS_REPORT` | Túlóra-összesítő | Mt. 99. § (5) – nyilvántartási kötelezettség |
| `ABSENCE_REGISTER` | `ANALYTICS_REPORT` | Hiányzások nyilvántartása | Mt. 134. §; Tbj. |
| `PAYROLL_SUMMARY_REPORT` | `ANALYTICS_REPORT` | Bérköltség-összesítő | Belső; MÁK adatszolgáltatás |

### 2.2. Property-k

| Property | Típus | Kötelező | Leírás | Jogalap / Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `code` | string | igen | Egyedi kód (pl. `EMPLOYMENT_CERTIFICATE`) | `ReportJob` indításkor hivatkozási alap |
| `name` | string | igen | Emberi neve (pl. „Munkáltatói igazolás") | |
| `nameLocalizations` | JSON | nem | Név más nyelveken: `{"en": "Employment Certificate", "de": "..."}` | Multi-language support |
| `category` | enum | igen | `ANALYTICS_REPORT` / `PRINT_DOCUMENT` / `OFFICIAL_DECISION` | `OFFICIAL_DECISION`: workflow outcome alapján generált határozat |
| `subjectType` | enum | igen | `PERSON` / `EMPLOYMENT` / `ORG_UNIT` / `PAYROLL_CYCLE` / `WORKFLOW_INSTANCE` / `MULTI` | A `ReportJob` elsődleges alanyának típusa |
| `status` | enum | igen | `DRAFT` / `PUBLISHED` / `DEPRECATED` / `ARCHIVED` | |
| `publishedVersionId` | UUID FK | nem | Az aktuálisan hatályos `ReportTemplateVersion` | null = nincs kiadott verzió |
| `effectiveFrom` | date | nem | Ettől a dátumtól alkalmazható | Jogszabályváltozás-követés; pl. új Kttv. sablon 2025.01.01-től |
| `effectiveTo` | date | nem | Eddig a dátumig alkalmazható | |
| `dataContractId` | UUID FK | igen | Melyik `ReportDataContract`-ot használja | A sablon és az adatréteg közötti stabil API |
| `defaultLocale` | string | igen | Alapértelmezett nyelv (pl. `hu-HU`) | |
| `availableLocales` | string[] | igen | Elérhető nyelvek listája | |
| `defaultLayoutEngine` | enum | igen | `HTML_PDF` / `DOCX_PDF` / `XLSX` | |
| `regulatoryReference` | string | nem | Ha jogszabályi kötelezettségből ered: hivatkozás (pl. „Mt. 46. §") | |
| `retentionPolicyCode` | string | nem | Melyik megőrzési policy vonatkozik a generált outputra | Az output törlési/archiválási szabályát határozza meg |
| `requiresApprovalToPublish` | boolean | igen | Szükséges-e jóváhagyás új verzió publikálásához | Igen pl. munkaszerződés-sablonnál |
| `requiresSignature` | boolean | igen | Az output aláírást igényel-e | `true` esetén `ReportOutput.signingStatus` nyomonkövetendő |
| `allowPreview` | boolean | igen | Engedélyezett-e a tényleges generálás előtt preview | Bizalmas sablonok esetén `false` |
| `ownerTeam` | string | nem | Felelős szerkesztői csapat (pl. „Jogi és HR Adminisztráció") | |
| `tags` | string[] | nem | Szabad szöveges kategóriák | |
| `isActive` | boolean | igen | | |
| `createdAt` | datetime | igen | | Audit |
| `createdBy` | UUID FK | igen | | Audit |
| `updatedAt` | datetime | igen | | Audit |
| `updatedBy` | UUID FK | igen | | Audit |

**Állapotgép:**
```
DRAFT → PUBLISHED → DEPRECATED → ARCHIVED
          ↑
  (approval, ha requiresApprovalToPublish = true)
  
PUBLISHED: pontosan egy publishedVersionId aktív
DEPRECATED: az effectiveTo lejárt, de régi outputok még hivatkoznak rá
ARCHIVED: teljes visszavonás, új generálás nem indítható
```

---

## 3. ReportTemplateVersion entitás

### 3.1. Az entitás célja

A **ReportTemplateVersion** a sablon tényleges tartalmát és eszközeit tárolja verziónként. Egy `ReportTemplate`-hez sok verzió tartozhat; a `publishedVersionId` mindig pontosan az egyikre mutat. A verziók nem törölhetők – minden generált `ReportOutput` visszamutat arra a verzióra, amellyel készült, és ez a kapcsolat az audit- és a jogi bizonyítóerő alapja.

### 3.2. Property-k

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `templateId` | UUID FK | igen | Melyik `ReportTemplate`-hez tartozik | |
| `version` | string | igen | Verziócímke (pl. `1.0.0`, `2.1`, `2025-A`) | Szemantikus verziózás ajánlott |
| `status` | enum | igen | `DRAFT` / `IN_REVIEW` / `APPROVED` / `PUBLISHED` / `RETIRED` | |
| `locale` | string | igen | Melyik nyelvhez (pl. `hu-HU`, `en-US`) | Egy adott verzió egy adott nyelvhez tartozik |
| `layoutEngine` | enum | igen | `HTML_PDF` / `DOCX_PDF` / `XLSX` | Felülírhatja a template-szintű alapértelmezést |
| `contentRef` | string | igen | A sablon tartalmának tárolóbeli hivatkozása (blob store path) | A tényleges HTML/CSS/DOCX fájl nem az adatbázisban van |
| `contentChecksum` | string | igen | SHA-256 checksum a tartalom integritásának ellenőrzéséhez | Tampering detection |
| `assetRefs` | JSON | nem | Hivatkozott eszközök (képek, fontok) és azok blob-referenciái | `{"logo": "assets/logo_2025.png", "font": "assets/NotoSans.woff2"}` |
| `brandingProfileId` | UUID FK | nem | Ha szervezet-specifikus arculatot alkalmaz | null = rendszer-szintű branding |
| `usedComponentIds` | UUID[] | nem | Hivatkozott `ReportComponent` rekordok | Komponens-verziók rögzítése a konzisztens reprodukálhatósághoz |
| `changeLog` | text | nem | Mi változott ebben a verzióban | |
| `dataContractVersion` | string | nem | Ha a `ReportDataContract` sémája is változott: melyik verziót vár | Visszafelé-kompatibilitás kezeléshez |
| `previewOutputRef` | string | nem | Előzetes render eredménye (bemutató célra) | |
| `approvedBy` | UUID FK | nem | Ha `requiresApprovalToPublish = true`: jóváhagyó | |
| `approvedAt` | datetime | nem | | |
| `publishedAt` | datetime | nem | Élesítés időpontja | |
| `publishedBy` | UUID FK | nem | | |
| `retiredAt` | datetime | nem | Visszavonás időpontja | |
| `retiredBy` | UUID FK | nem | | |
| `createdAt` | datetime | igen | | Audit |
| `createdBy` | UUID FK | igen | | Audit |

**Megjegyzések:**
- A `contentRef` blob store-ra mutat (pl. S3, Azure Blob, MinIO), nem az adatbázisba tárolt BLOB. Ez lehetővé teszi a nagy DOCX/HTML fájlok hatékony tárolását és a verziókövetési rendszerrel (git) való párhuzamos kezelhetőséget.
- Az `usedComponentIds` rögzítése azt biztosítja, hogy ha egy `ReportComponent` verzióváltáson megy át, a korábbi `ReportTemplateVersion` pontosan ugyanazokat a komponenseket használja, mint a generáláskor. Ez nélkül a régi outputok „egyező verziójú sablonból, de más komponensből" készültek volna – ami auditálási problémát okoz.
- `RETIRED` ≠ `ARCHIVED` a template szintjén: a verzió visszavonható (nem adható ki belőle új output), de a meglévő outputok érvényességét nem érinti.

---

## 4. ReportDataContract entitás

### 4.1. Az entitás célja

A **ReportDataContract** a Report Publishing BC legfontosabb architektúrális eleme: a sablon és az adatréteg közötti stabil, verziózott API-leírás. A template-fejlesztő nem a Report DB tábláit ismeri – csak a DataContract által definiált render modelt (JSON struktúrát). Az adatbetöltő pipeline a DataContract alapján állítja össze ezt a JSON-t. Ez a szétválasztás biztosítja, hogy az Analytics BC belső sémájának változása ne törje el a sablonokat.

### 4.2. Property-k

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `code` | string | igen | Egyedi kód (pl. `CONTRACT_EMPLOYMENT_SINGLE`, `CONTRACT_HEADCOUNT_LIST`) | |
| `name` | string | igen | Emberi neve | |
| `description` | text | nem | Mit tartalmaz a render model, hogyan kell értelmezni | |
| `version` | string | igen | Séma verziója (pl. `1.0`, `1.1`, `2.0`) | Tört verziónál visszafelé-kompatibilis; egész verziónál törés |
| `renderModelSchema` | JSON | igen | JSON Schema a render model teljes struktúrájára | A template-fejlesztő ebből tudja, milyen változók állnak rendelkezésre |
| `requiredParameters` | JSON | igen | A `ReportJob` indításakor kötelezően megadandó paraméterek: `[{"name": "employmentId", "type": "UUID", "description": "..."}, {"name": "asOfDate", "type": "date"}]` | |
| `optionalParameters` | JSON | nem | Opcionális paraméterek és alapértelmezett értékeik | |
| `primaryDatasetCode` | string | igen | Elsődleges adatforrás az Analytics BC-ben (pl. `EMPLOYMENT_CURRENT`) | |
| `secondaryDatasetCodes` | string[] | nem | Kiegészítő adatforrások | |
| `accessedClassifications` | enum[] | igen | Adatosztályozások, amelyeket a render model tartalmaz: `PUBLIC` / `INTERNAL` / `PII` / `SPECIAL` / `FIN` | A `ReportJob` generálásakor ezek alapján ellenőrzendő a kérelmező jogosultsága |
| `accessedFields` | JSON | nem | Mezőszintű lista (dataset + fieldName) a szemantikus pontossághoz | |
| `rlsScope` | enum | nem | Milyen row-level security scope érvényesül: `ALL_ROWS` / `OWN_ORG_UNIT_AND_CHILDREN` / `DIRECT_REPORTS` / `SUBJECT_ONLY` | `SUBJECT_ONLY`: csak a `ReportJob` alanyára vonatkozó adatot tartalmaz |
| `supportsAsOfDate` | boolean | igen | Elfogad-e `asOfDate` paramétert (point-in-time lekérdezéshez) | |
| `maxOutputRows` | integer | nem | Ha listás riport: maximum sorok száma | |
| `isActive` | boolean | igen | | |
| `deprecatedAt` | datetime | nem | | |
| `replacedByContractId` | UUID FK | nem | Ha elavult: utód contract | |
| `createdAt` | datetime | igen | | Audit |
| `createdBy` | UUID FK | igen | | Audit |

**A render model struktúrájának elve:**

A `renderModelSchema` által leírt JSON mindig egységes burkolóstruktúrát követ:

```json
{
  "meta": {
    "generatedAt": "2025-03-15T10:30:00Z",
    "templateCode": "EMPLOYMENT_CERTIFICATE",
    "templateVersion": "2.1.0",
    "dataContractVersion": "1.1",
    "asOfDate": "2025-03-15",
    "requestedBy": "Kiss Éva (HR ügyintéző)",
    "locale": "hu-HU"
  },
  "branding": { "...": "BrandingProfile adatok" },
  "subject": { "...": "Az elsődleges alany adatai (person/employment/orgunit)" },
  "data": { "...": "A contract által definiált fő adatstruktúra" },
  "parameters": { "...": "A futtatáskor megadott paraméterek" }
}
```

A `meta` blokk minden sablonban kötelező – ez teszi lehetővé a generálási időpont, a sablonverzió és az adatkészítő személy megjelenítését a dokumentumon.

---

## 5. ReportComponent entitás

### 5.1. Az entitás célja

A **ReportComponent** újrafelhasználható, verzionált UI-komponensek könyvtára. Egy komponens önálló HTML+CSS partial (vagy DOCX részlet), amelyet több sablon is importálhat. A komponens-könyvtár teszi lehetővé a konzisztens kinézetet, a gyors sablon-fejlesztést és a rendszer-szintű PII-masking szabályok centralizált érvényesítését.

Alap komponens-készlet:

| Kód | Leírás | Masking |
|---|---|---|
| `HEADER` | Céges fejléc (logo, cégnév, dokumentum cím, iktatószám) | Nincs |
| `FOOTER` | Lábléc (oldalszám, generálási időpont, sablonverzió, QR-kód) | Nincs |
| `PERSON_BLOCK` | Személy alapadatai (név, TAJ, adóazonosító, cím) | TAJ, bankszámla maszkolt alapból |
| `EMPLOYMENT_BLOCK` | Jogviszony adatai (munkakör, szervezeti egység, kezdeti dátum) | Nincs |
| `SALARY_BLOCK` | Béradatok | Csak `FIN` jogosultsággal unmaskolt |
| `SIGNATURE_BLOCK` | Aláírási sáv (dátum, aláíró neve, beosztása, helységnév) | Nincs |
| `APPROVAL_STAMP` | Iktatási/jóváhagyási bélyegző blokk | Nincs |
| `TABLE` | Standard táblázatstílus (fejlécsor, adatsorok, összesítő sor) | Nincs |
| `WARNING_CONFIDENTIAL` | „Bizalmas – csak belső használatra" figyelmeztető blokk | Nincs |

### 5.2. Property-k

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `code` | string | igen | Egyedi kód (pl. `PERSON_BLOCK`) | Template-ekből `{{> PERSON_BLOCK}}` szintaxissal hivatkozható |
| `name` | string | igen | Emberi neve | |
| `description` | text | nem | Mit tartalmaz, mikor és hogyan használandó | |
| `version` | string | igen | Verziócímke | |
| `status` | enum | igen | `DRAFT` / `PUBLISHED` / `DEPRECATED` | |
| `contentRef` | string | igen | Blob store hivatkozás a komponens tartalmára | |
| `contentChecksum` | string | igen | SHA-256 checksum | |
| `expectedDataPaths` | JSON | nem | Melyik render model útvonalakra épít (pl. `["subject.person.fullName", "subject.person.taxId"]`) | Dokumentáció és lint-ellenőrzés célra |
| `maskingBehavior` | JSON | nem | Melyik adatmezőn milyen masking érvényesül alapból (pl. `{"subject.person.socialSecurityNumber": "PARTIAL_MASK"}`) | Centralizált PII-masking szabály |
| `supportedLayoutEngines` | enum[] | igen | Melyik layout engine-ekkel kompatibilis | |
| `changeLog` | text | nem | | |
| `publishedAt` | datetime | nem | | |
| `publishedBy` | UUID FK | nem | | |
| `createdAt` | datetime | igen | | Audit |
| `createdBy` | UUID FK | igen | | Audit |

**Megjegyzések:**
- A `maskingBehavior` a `ReportComponent` szintjén definiálja, hogy pl. a `PERSON_BLOCK` TAJ-számot mindig maszkolt formában jeleníti meg – kivéve, ha a `ReportJob` kérelmezőjének explicit `SPECIAL` jogosultsága van. Ez a masking szabály a komponensbe égetett, nem a sablonba: így nem lehet véletlenül „elfelejteni" maszkolt mezőt egy új sablonban.
- A `FOOTER` komponens kötelező eleme minden `PRINT_DOCUMENT` kategóriájú sablonnak: tartalmazza a generálási időpontot és a sablonverziót – ez az anti-tampering és az audit alapja.

---

## 6. BrandingProfile entitás

### 6.1. Az entitás célja

A **BrandingProfile** egy szervezet (vagy szervezeti egység) arculati csomagját írja le: logót, színeket, betűtípusokat, fejléc/lábléc szövegeket. A sablonok nem tartalmaznak konkrét arculati elemeket – csak tokenekre hivatkoznak (`{{brand.logo}}`, `{{brand.primaryColor}}`), a tényleges értékeket a `BrandingProfile` adja. Ez teszi lehetővé a multi-tenant működést: azonos sablon, eltérő arculattal különböző szervezetek számára.

### 6.2. Property-k

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `organizationId` | UUID FK | igen | Melyik szervezethez tartozik | |
| `name` | string | igen | Profil neve (pl. „OrgwareHU 2025 arculat") | |
| `logoRef` | string | nem | Logo blob store hivatkozás | |
| `logoAltText` | string | nem | Logo alternatív szöveg (akadálymentesség) | |
| `primaryColor` | string | nem | Elsődleges szín hex kódja (pl. `#1A3A5C`) | |
| `secondaryColor` | string | nem | Másodlagos szín | |
| `fontFamilyRef` | string | nem | Betűtípus blob store hivatkozás (WOFF2) | |
| `fontFamilyName` | string | nem | Betűtípus neve (pl. `Noto Sans`) | |
| `headerText` | JSON | igen | Fejléc szöveg nyelvekhez: `{"hu-HU": "OrgwareHU Kft.", "en-US": "OrgwareHU Ltd."}` | |
| `footerText` | JSON | nem | Lábléc szöveg nyelvekhez | |
| `companyAddressBlock` | JSON | nem | Cím, adószám, cégjegyzékszám – nyomtatványokon megjelenítendő | |
| `documentNumberPrefix` | string | nem | Iktatószám-előtag (pl. `ORGW/2025/`) | |
| `signatureOfficerTitle` | JSON | nem | Az aláíró beosztása különböző dokumentumtípusokon (pl. `{"APPOINTMENT_DECREE": "Igazgató"}`) | |
| `watermarkText` | string | nem | Ha kell vízjel (pl. „TERVEZET") | |
| `isDefault` | boolean | igen | Ez-e az alapértelmezett profil a szervezethez | |
| `isActive` | boolean | igen | | |
| `validFrom` | date | nem | | |
| `validTo` | date | nem | | |
| `createdAt` | datetime | igen | | Audit |
| `createdBy` | UUID FK | igen | | Audit |

---

## 7. ReportJob entitás

### 7.1. Az entitás célja

A **ReportJob** egy konkrét generálási kérést és annak végrehajtási állapotát tartalmazza. A generálás aszinkron folyamat: a kérelmező elküldi a `ReportJob`-ot, a render engine feldolgozza (esetleg sorban áll, ha terhelt), és a kész output egy `ReportOutput` rekordban jelenik meg. A `ReportJob` tárolja a kérés teljes paraméterezését – az output törlése esetén is rekonstruálható, mit akartak generálni és milyen adatokkal.

### 7.2. Azonosítók és kapcsolatok

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `templateId` | UUID FK | igen | Melyik `ReportTemplate`-ből | |
| `templateVersionId` | UUID FK | nem | null = a létrehozáskor aktuális `publishedVersionId` használandó; explicit = adott verzió | Visszamenőleges újrageneráláshoz szükséges |
| `dataContractId` | UUID FK | igen | Melyik `ReportDataContract` alapján töltődnek az adatok | Denormalizált a template-verzióváltások kezelésére |
| `requestedById` | UUID FK | igen | Ki kérte a generálást | |
| `requestedAt` | datetime | igen | | |
| `subjectEntityType` | enum | nem | A `ReportTemplate.subjectType`-ból örökölt; ha konkretizálható | |
| `subjectEntityId` | UUID | nem | Az alany entitás azonosítója (pl. `Employment.id`, `PayrollCycle.id`) | |

### 7.3. Paraméterek

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `parameters` | JSON | igen | A `ReportDataContract.requiredParameters` szerint megadott értékek + opcionálisak | |
| `asOfDate` | date | nem | Point-in-time lekérdezési dátum (ha a contract `supportsAsOfDate = true`) | Kiemelve a `parameters`-ből: kritikus auditálási adat |
| `locale` | string | igen | Generálandó dokumentum nyelve | |
| `brandingProfileId` | UUID FK | nem | null = szervezet alapértelmezett BrandingProfile | |
| `outputFormat` | enum | igen | `PDF` / `DOCX` / `XLSX` / `HTML` | |

### 7.4. Végrehajtás és státusz

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `status` | enum | igen | `QUEUED` / `RUNNING` / `SUCCEEDED` / `FAILED` / `CANCELLED` | |
| `queuedAt` | datetime | nem | Feldolgozási sorba kerülés időpontja | |
| `startedAt` | datetime | nem | Render elkezdésének időpontja | |
| `finishedAt` | datetime | nem | Befejezés időpontja | |
| `durationMs` | integer | nem | Render futásidő milliszekundumban | |
| `renderEngineVersion` | string | nem | A render engine verziója (reprodukálhatósághoz) | |
| `rowCount` | integer | nem | Ha listás riport: hány sor szerepel az outputban | |
| `errorCode` | string | nem | Ha `status = FAILED`: hibakód | |
| `errorMessage` | text | nem | Részletes hibaüzenet | |
| `retryCount` | integer | igen | Automatikus újrapróbálkozások száma (0-tól) | |
| `maxRetries` | integer | igen | Maximum automatikus újrapróbálkozások | |
| `isPreview` | boolean | igen | Előnézeti generálás-e (nem menti végleges outputként) | Preview esetén a `ReportOutput` `isPreview = true`-val keletkezik |

### 7.5. Jogosultság-ellenőrzés

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `permissionCheckStatus` | enum | igen | `PENDING` / `PASSED` / `FAILED` | Automatikusan értékeli a kérelmező jogosultságait a `DataContract.accessedClassifications` alapján |
| `permissionCheckDetails` | JSON | nem | Miért sikertelen a jogosultság-ellenőrzés | |
| `appliedMasking` | JSON | nem | Melyik mezőkön érvényesült masking a generált outputban | Auditálási célra |
| `appliedRlsScope` | JSON | nem | Érvényesített row-level security scope | |
| `createdAt` | datetime | igen | | Audit |

**Állapotgép:**
```
QUEUED → RUNNING → SUCCEEDED → [ReportOutput létrejön]
                 ↘ FAILED    → [retry, ha retryCount < maxRetries]
                              → [véglegesen FAILED, ha limit elérve]
         ↑
CANCELLED (kérelmező visszavonta, mielőtt RUNNING lett)
```

---

## 8. ReportOutput entitás

### 8.1. Az entitás célja

A **ReportOutput** a kész generált dokumentum rekordja. Tárolja a fájl tárolóbeli hivatkozását, integritás-checksumát, az adatosztályozást, a megőrzési policy-t és az aláírási státuszt. Ez az entitás a Document Management BC felé mutat: az output archiválásakor a Document BC veszi át a hosszú távú tárolást, és linkeli a releváns személyekhez, jogviszonyokhoz vagy ügyekhez.

### 8.2. Property-k

| Property | Típus | Kötelező | Leírás | Megjegyzés |
|---|---|---|---|---|
| `id` | UUID | igen | Rendszer belső azonosító (PK) | |
| `jobId` | UUID FK | igen | Melyik `ReportJob` eredménye | |
| `templateId` | UUID FK | igen | Denormalizált – gyors lekérdezhetőséghez | |
| `templateVersionId` | UUID FK | igen | **Pontosan** melyik verzióval készült | Jogi bizonyítóerő: az output és a sablon közötti kapcsolat visszavonhatatlan |
| `dataContractId` | UUID FK | igen | Melyik contract alapján töltődtek az adatok | |
| `generatedAt` | datetime | igen | Generálás befejezésének időpontja | |
| `format` | enum | igen | `PDF` / `DOCX` / `XLSX` / `HTML` | |
| `fileSizeBytes` | bigint | nem | | |
| `contentRef` | string | igen | Blob store hivatkozás a generált fájlra | |
| `contentHash` | string | igen | SHA-256 checksum a tartalom integritásának ellenőrzéséhez | Anti-tampering; e-aláírásnál kötelező |
| `renderModelSnapshot` | JSON | nem | A generáláshoz felhasznált render model pillanatképe | Ezzel rekonstruálható, milyen adatokból készült a dokumentum – akkor is, ha a forrásadatok azóta változtak |
| `locale` | string | igen | A dokumentum nyelve | |
| `subjectEntityType` | enum | nem | Az alany típusa (denormalizált) | |
| `subjectEntityId` | UUID | nem | Az alany azonosítója (denormalizált) | |
| `classification` | enum | igen | Az output adatosztályozása: `INTERNAL` / `CONFIDENTIAL` / `RESTRICTED` | A legmagasabb osztályozású mező alapján automatikusan meghatározva |
| `containsPiiData` | boolean | igen | | |
| `containsSpecialData` | boolean | igen | | |
| `containsFinData` | boolean | igen | | |
| `isPreview` | boolean | igen | Előnézeti output-e (nem archiválandó) | Preview outputok automatikusan törölhetők X óra után |
| `documentManagementRef` | UUID FK | nem | Ha átkerült a Document BC-be: ott kapott azonosítója | null = még nem archiválva |
| `archivedAt` | datetime | nem | Document BC-be kerülés időpontja | |
| `retentionPolicyCode` | string | nem | Megőrzési policy kódja (a `ReportTemplate`-ből örökölt) | |
| `deleteAt` | datetime | nem | Ha időkorlátozott megőrzés: törlési időpont | |
| `signingStatus` | enum | nem | `NOT_REQUIRED` / `PENDING` / `SIGNED` / `REJECTED` | Ha `ReportTemplate.requiresSignature = true` |
| `signedAt` | datetime | nem | Aláírás időpontja | |
| `signedById` | UUID FK | nem | Aláíró személy | |
| `signatureRef` | string | nem | E-aláírás blob hivatkozása (ha digitális aláírás) | |
| `qrCodeData` | string | nem | QR-kódba kódolt verifikációs adat (dokumentumazonosító + checksum) | Anti-tampering és gyors hitelességellenőrzés |
| `downloadCount` | integer | igen | Hányszor töltötték le | |
| `lastDownloadedAt` | datetime | nem | Utolsó letöltés időpontja | |
| `lastDownloadedById` | UUID FK | nem | | |
| `createdAt` | datetime | igen | | Audit |

**Megjegyzések:**
- A `renderModelSnapshot` a legfontosabb auditálási elem: ha a munkavállaló adatai azóta megváltoztak, de az igazolást a korábbi állapotból generálták, pontosan rekonstruálható, hogy a dokumentum abban a pillanatban helyes volt-e. Ez munkaügyi viták esetén bizonyítékértékű.
- A `contentHash` + `qrCodeData` kombináció teszi lehetővé, hogy egy printelt papíralapú dokumentumot visszaellenőrizzünk: a QR-kód tartalmazza a hash-t, a rendszerben tárolt hash-sel összevetve igazolható a dokumentum hitelessége.
- `isPreview = true` outputok nem kerülnek a Document BC-be és nem keletkezik hozzájuk `DataAccessLog` export-rekord – de a `ReportJob` és a `ReportOutput` rekord megmarad auditálási célra.

---

## 9. Historikus adatkezelés

| Entitás | Kezelési mód | Indok |
|---|---|---|
| **ReportTemplate** | `DEPRECATED` / `ARCHIVED` státusszal inaktiválható; nem törölhető | Régi outputok hivatkoznak rá |
| **ReportTemplateVersion** | Soha nem törölhető | Minden `ReportOutput` visszamutat egy konkrét verzióra – a kapcsolat a jogi bizonyítóerő alapja |
| **ReportDataContract** | Verziózott; elavulás `deprecatedAt`-tel; nem törölhető | Template-ek hivatkoznak rá |
| **ReportComponent** | `DEPRECATED` státusszal inaktiválható; nem törölhető ha aktív template-ek hivatkoznak rá | |
| **BrandingProfile** | `validTo`-val inaktiválható; nem törölhető | Korábbi outputok arculatának rekonstruálhatósága |
| **ReportJob** | Nem módosítható; nem törölhető | Generálási kérés auditnyoma |
| **ReportOutput** | Nem módosítható; törlés csak lejárt `deleteAt` esetén | Jogi dokumentumok esetén a Document BC veszi át a megőrzést |

**Megőrzési idők:**

| Output típus | Megőrzési idő | Jogalap |
|---|---|---|
| **Munkajogi nyomtatványok** (munkaszerződés, felmondás, kinevezés) | Jogviszony megszűnése + 50 év | Tny. – kereset-adatok; Kttv. 228. §; Kjt. 85. § |
| **Igazolások** (munkáltatói, jövedelmi) | 5 év | Mt. 286. § – munkaügyi igény elévülése |
| **Fegyelmi határozatok** | Lezárás + 5 év (jogerőre emelkedéstől) | Pp. elévülési szabályok |
| **Teljesítményértékelési lapok** | Jogviszony megszűnése + 50 év (közszféra) / 5 év (Mt.) | Kttv. 228. §; Mt. 286. § |
| **Analitikus riportok (PDF/XLSX)** | 5 év | Art. 78. § – adóellenőrzési alap; belső döntéstámogatási célra |
| **Preview outputok** | 24 óra | Operatív szükséglet; nem archiválandó |
| **ReportJob rekordok** | A hozzá tartozó output megőrzési idejéig | Auditnyom |

---

## 10. GDPR adatkezelési kategorizáció

| Kategória | Érintett property-k | Jogalap | Megjegyzés |
|---|---|---|---|
| **Generált dokumentumok tartalma** | `ReportOutput.contentRef` (a tényleges fájl) | Jogi kötelezettség (6(1)c) – nyomtatványok esetén; Szerződés (6(1)b); Jogos érdek (6(1)f) – analitikus riportok | A tartalom osztályozása a `ReportDataContract.accessedClassifications` alapján |
| **Render model pillanatkép** | `ReportOutput.renderModelSnapshot` | Azonos a tartaloméval | PII/SPECIAL adatot tartalmaz; same retention mint az output |
| **Generálási audit** | `ReportJob.*`, `ReportOutput.generatedAt`, `downloadCount` | Elszámoltathatóság (GDPR 5(2)) | Ki generált mit, mikor, hányszor töltötte le |
| **Sablon metaadatok** | `ReportTemplate.*`, `ReportTemplateVersion.*` | Jogos érdek (6(1)f) | Nem tartalmaznak személyes adatot |
| **Aláírási adatok** | `ReportOutput.signedById`, `signedAt` | Jogi kötelezettség (6(1)c) | Okirat-értékű dokumentumoknál |

**GDPR 15. cikk és a generált dokumentumok:**
Ha egy munkavállaló hozzáférési kérést nyújt be, joga van megtudni, hogy milyen dokumentumokat generáltak róla, mikor és ki kérte azokat. A `ReportJob.requestedById` + `ReportOutput.generatedAt` + `subjectEntityId` alapján ez visszakereshető. Az igazolások és a munkaszerződések vonatkozásában maga a generált dokumentum is az érintett személyes adatait tartalmazza – ezek az érintett kérésére kiadandók (GDPR 15(3). cikk).

---

## 11. Hozzáférési szintek (javasolt)

| Szerep | ReportTemplate (olvasás) | ReportJob indítása | Output letöltése | Sablon szerkesztése |
|---|---|---|---|---|
| **Munkavállaló** | Csak saját típusok (igazolások) | Saját igazolások önkiszolgáló generálása | Saját outputjai | Nem |
| **Közvetlen vezető** | Beosztottakra vonatkozó típusok | Beosztott-igazolások, csapatriportok | Saját és beosztottakra vonatkozó | Nem |
| **HR ügyintéző** | Összes, hatáskörön belül | Hatáskörén belüli alanyokra | Hatáskörön belüli outputok | Nem |
| **Bérszámfejtő** | `FIN` osztályozású sablonok | Bérszámfejtési outputok | Bérszámfejtési outputok | Nem |
| **HR vezető** | Összes | Összes | Összes | Draft szerkesztés |
| **Sablon-adminisztrátor** | Összes | Tesztelési célból | Összes | Teljes (Draft → Published) |
| **Rendszeradminisztrátor** | Technikai mezők | Nem | Nem (tartalom) | Csak technikai |

---

## 12. Validációs szabályok

| Szabály | Validáció | Megjegyzés |
|---|---|---|
| `ReportJob` csak `PUBLISHED` státuszú `ReportTemplate`-re indítható | Kötelező | `DRAFT` sablonból csak preview generálható |
| `ReportJob.parameters` teljesíti a `DataContract.requiredParameters` összes feltételét | Kötelező | Kötelező paraméterhiány esetén `FAILED` státusz, nem csendben üres output |
| `ReportJob.permissionCheckStatus = PASSED` nélkül az output nem kerül kiadásra | Kötelező | |
| `ReportOutput.contentHash` újraszámítandó és összevetendő minden letöltésnél | Kötelező | Tampering detection |
| `ReportTemplateVersion` nem törölhető, ha `ReportOutput` rekordok hivatkoznak rá | Kötelező | Referenciális integritás |
| `ReportDataContract` nem módosítható `PUBLISHED` verzió esetén – csak új verzió hozható létre | Kötelező | A stabil API garantálása |
| `ReportOutput.isPreview = false` esetén kötelező `DataAccessLog` bejegyzés az Analytics BC-ben | Kötelező | Auditálhatóság |
| `BrandingProfile`: legfeljebb egy `isDefault = true` szervezetenként | Kötelező | |
| `ReportTemplateVersion.usedComponentIds` csak `PUBLISHED` státuszú `ReportComponent`-re hivatkozhat | Kötelező | |

---

## 13. Integrációs pontok

### 13.1. Bejövő – adatforrások

| Forrás | Mit ad | Hogyan |
|---|---|---|
| **Analytics / Query BC** | Render model adatok (a `ReportDataContract` által definiált JSON struktúra) | A Report Publishing BC query-t intéz a Report DB-re a DataContract paraméterei szerint |
| **Workflow BC** | `OFFICIAL_DECISION` kategóriájú outputok triggelése (határozatok, döntések) | `WorkflowInstance.status → APPROVED/REJECTED` → automatikus `ReportJob` indítás |
| **Payroll BC** | Bérszámfejtés zárásakor `PAYROLL_SUMMARY_REPORT` típusú job automatikus indítása | `PayrollRun.status → LOCKED` → event → `ReportJob` |
| **Leave BC** | Szabadságegyenleg-igazolás önkiszolgáló generáláshoz | API hívás `LEAVE_BALANCE_STATEMENT` sablonra |

### 13.2. Kimenő – outputok célállomásai

| Cél | Esemény | Tartalom |
|---|---|---|
| **Document Management BC** | `ReportOutput.isPreview = false AND status = SUCCEEDED` → archiválás | A kész fájl + metaadatok átadása; a Document BC linkelési lehetőséget kap az `Employment`, `Person`, `WorkflowInstance` entitásokhoz |
| **Analytics / Query BC** | `DataAccessLog` bejegyzés | Minden nem-preview output generálása rögzítve |
| **Workflow BC** | Aláírás-kérés indítása | Ha `ReportTemplate.requiresSignature = true` és `signingStatus = PENDING` |
| **Értesítési rendszer** | Output elkészültéről értesítés | `ReportJob.status → SUCCEEDED` → push / email a kérelmezőnek |
| **Email / SFTP** | `ScheduledReport` által automatikusan generált és kézbesített outputok | Havi bérköltség-PDF pénzügyi vezetőknek stb. |

### 13.3. Fontosabb automatikus generálási triggerek

| Trigger esemény | Generálandó sablon | Megjegyzés |
|---|---|---|
| `Employment` létrehozva + `WorkflowInstance` jóváhagyta | `EMPLOYMENT_CONTRACT` vagy `APPOINTMENT_DECREE` | Jogviszony-típustól függően |
| `Employment.status → TERMINATED` | `TERMINATION_NOTICE`, `LEAVE_BALANCE_STATEMENT` | Mt. 80. §; Mt. 122. § – kötelező kiadni |
| `PerformanceReview.status → COMPLETED` | `PERFORMANCE_REVIEW_FORM` | Kttv. 131. §; Kjt. 66. § – a minősítési lap kötelező |
| `PayrollRun.status → LOCKED` | `PAYROLL_SUMMARY_REPORT` | Bérszámfejtési zároláskor összesítő |
| `DisciplinaryAction` lezárult | `DISCIPLINARY_DECISION` | Kttv. 166. §; Kjt. 47. § – kötelező írásbeli határozat |

---

## 14. Kapcsolódó entitások (hivatkozás)

```
ReportTemplate 1 ──── N ReportTemplateVersion  (sablon → verziók)
ReportTemplate 1 ──── 1 ReportDataContract     (sablon → adatkontraktus)
ReportTemplate 1 ──── N ReportJob              (sablon → generálási kérések)

ReportTemplateVersion N ──── N ReportComponent (verzió → felhasznált komponensek)
ReportTemplateVersion N ──── 1 BrandingProfile (verzió → arculat)

ReportDataContract 1 ──── N ReportJob          (kontraktus → futtatások)

ReportJob 1 ──── 1 ReportOutput                (kérés → output)

ReportOutput N ──── 1 ReportTemplateVersion    (output → pontosan ez a verzió készítette)
ReportOutput N ──── 1 Document (Document BC)   (output → archiválva itt)

Organization 1 ──── N BrandingProfile          (szervezet → arculati profilok)

Person 1 ──── N ReportJob (requestedBy)        (személy → kért generálások)
Person 1 ──── N ReportOutput (signedBy)        (személy → aláírt dokumentumok)
```

---

## 15. Nyitott kérdések

1. **Render engine infrastruktúra döntése:** Az `HTML_PDF` layout engine mögött headless Chrome (Playwright/Puppeteer) vagy dedikált PDF renderer (pl. WeasyPrint, Prince) álljon? A döntés befolyásolja a CSS page-break viselkedést, a fejléc/lábléc kezelését, a betűtípus-renderelés minőségét és a szerver-oldali erőforrásigényt. HR nyomtatványoknál a pixel-perfect oldaltörés és a táblák oldalhatáron való viselkedése a leggyakoribb probléma – ezt érdemes korán tesztelni néhány valós sablonon.

2. **DOCX template és a jogi elvárás szétválasztása:** Több jogszabály (Kttv., Kjt.) implicit elvárja, hogy a kinevezési okirat és a munkaszerződés „nyomtatványa" szerkeszthető DOCX-ban is rendelkezésre álljon (HR ügyintézők, ügyvédek igénye). Ugyanakkor a DOCX → PDF konverzió minőségi variancéjú. Megoldás: `HTML_PDF` az archiválandó végleges output; `DOCX_PDF` csak munkamásolatként. Ez a `layoutEngine` szintű megkülönböztetést igényli az outputon is, nem csak a sablonon.

3. **Önkiszolgáló igazolás-generálás és az adathozzáférés határa:** Ha a munkavállaló önkiszolgáló portálon generál magának munkáltatói igazolást, a render modelbe csak a saját adatai kerülhetnek – de az igazoláshoz szükséges cégadatok (adószám, cégjegyzékszám, aláíró neve és beosztása) a `BrandingProfile`-ból jönnek. Ez elvileg nem sérti az adatbiztonsági modellt, de az automatikus aláírás-blokk (amelyen a HR vezető neve szerepel) hozzájárulási kérdést vet fel: a HR vezető neve egy önkiszolgáló folyamatban automatikusan megjelenhet-e? Szervezeti policy szükséges.

4. **`renderModelSnapshot` megőrzése és mérete:** A render model pillanatkép komoly méretű JSON lehet (különösen listás riportoknál). Minden `ReportOutput`-nál tárolni – az 50 éves megőrzési idő mellé – potenciálisan nagy adatmennyiséget jelent. Kompromisszum: a `PRINT_DOCUMENT` kategóriánál kötelező (a jogi bizonyítóerőhöz szükséges); az `ANALYTICS_REPORT` kategóriánál elegendő a `ReportJob.parameters` megőrzése és a Report DB snapshot-ok visszajátszhatósága – ha azok rendelkezésre állnak. Ez viszont felveti a snapshot-megőrzési horizontok összehangolását az Analytics BC-vel.

5. **E-aláírás integráció:** A `ReportOutput.signingStatus` és `signatureRef` mezők előkészítik az e-aláírást, de a konkrét integrációs mód (minősített e-aláírás ADES szabvány szerint, egyszerű e-aláírás, belső jóváhagyói aláírás) az HRMS-en kívüli döntés. Közszféra esetén (Kttv., Kit.) az okiratok minősített e-aláírása jogszabályi elvárás lehet – ezt a Wave 3 scope-ba érdemes venni, de az entitásmodell már most felkészíthető rá a `signatureRef` strukturálásával (ETSI AdES formátum).

6. **Komponenskönyvtár governance:** Ki írhat új `ReportComponent`-et, és ki publikálhatja? Ha bárki publikálhat, a `PERSON_BLOCK` masking-szabályai véletlenül felülírhatók egy rosszul konfigurált komponenssel – ami GDPR incidenst okoz. Javasolt: a komponenskönyvtár kizárólag `TEMPLATE_ADMIN` szerepkörrel szerkeszthető, és minden publikálás `requiresApprovalToPublish = true`-val kezelendő – azonos folyamaton, mint a sablonok.

7. **Batch generálás teljesítménye:** Egyes triggerek (pl. jogviszony-megszüntetés hullám szervezeti átalakítás esetén, havi bérszámfejtés zárásakor 200 dolgozó összesítője) egyszerre sok `ReportJob`-ot indítanak. A render engine kapacitás-tervezése (párhuzamos worker-ek száma, queue prioritizálás, `isPreview` vs. éles job szétválasztása) infrastruktúra-döntés – de az entitásmodellben a `ReportJob.status = QUEUED` + `queuedAt` mezők ezt a várakozási sort kezelik. A queue-mélység monitorozása SLA szempontból kritikus: egy havi bérszámfejtés után az összes bérpapírnak legkésőbb a kifizetés napjáig elérhetőnek kell lennie (Mt. 155. §).
