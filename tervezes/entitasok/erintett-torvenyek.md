# Érintett törvények – HRMS entitástervezés

> **Kontextus:** Greenfield integrált HRMS rendszer tervezése, magyarországi jogszabályi környezetben.
> **Cél:** Rögzíteni azokat a törvényeket és jogszabályokat, amelyek meghatározzák, milyen entitásokat, property-ket és adatkezelési szabályokat kell figyelembe venni a rendszer tervezésekor.
> **Utolsó frissítés:** 2025. február

---

## I. Foglalkoztatási jogviszonyokat szabályozó törvények

A magyar foglalkoztatási jog **fragmentált**: a különböző szektorok dolgozóira eltérő jogállási törvények vonatkoznak. Egy integrált HRMS-nek képesnek kell lennie ezeket a jogviszony-típusokat párhuzamosan kezelni, mert egyetlen munkáltató (pl. egy önkormányzat) alatt is előfordulhat, hogy egyes dolgozók az Mt., mások a Kttv. vagy a Kjt. hatálya alatt állnak.

### 1. Mt. – Munka Törvénykönyve

- **Jogszabály:** 2012. évi I. törvény
- **Hatály:** Versenyszféra munkaviszonyai (általános háttérjogszabály a legtöbb jogállási törvényhez)
- **Jogviszony típusa:** Munkaviszony (munkaszerződés alapján)

**HRMS szempontból releváns területek:**

- Munkaszerződés tartalmi elemei: munkahely, munkakör, alapbér
- Munkaidő-nyilvántartás (Mt. 134. §) – a munkáltató köteles nyilvántartani a rendes és rendkívüli munkaidőt, készenlétet, ügyeletet
- Szabadság-nyilvántartás: alapszabadság (20 nap) + életkor szerinti pótszabadság + egyéb pótszabadságok (gyermek után, fogyatékosság, apasági)
- Felmondási idő és végkielégítés számítása (a munkaviszonyban töltött idő alapján)
- **Munkavállalói adatkezelés (Mt. 10–11/A. §):** 2019-es GDPR salátatörvénnyel (2019. évi XXXIV. tv.) módosítva – a munkáltató csak a munkaviszony létesítése, fenntartása, megszüntetése szempontjából szükséges adatokat kezelheti; okirat bemutatása kérhető, másolat általában nem
- Köztulajdonban álló munkáltatókra vonatkozó speciális szabályok (Mt. 204–207. §)
- **2025-ös módosítások:** apasági szabadság igénybevételi határideje a gyermek születésétől számított 4. hónap végéig kiterjesztve; munkaszüneti napi rendkívüli munkavégzés pótléka egyértelműsítve (100%)

---

### 2. Kjt. – Közalkalmazottak jogállásáról szóló törvény

- **Jogszabály:** 1992. évi XXXIII. törvény
- **Hatály:** Folyamatosan szűkül (2021: egészségügy kikerült → Eszjtv.; 2024: köznevelés nagy része kikerült → Púétv.); **továbbra is hatályos:** szociális szféra, gyermekvédelem, kulturális intézmények, felsőoktatás egyes területei
- **Jogviszony típusa:** Közalkalmazotti jogviszony (kinevezés alapján)

**HRMS szempontból releváns területek:**

- **Fizetési osztály (A–J) és fokozat (1–14) rendszer** – a besorolás az iskolai végzettség és a közalkalmazotti jogviszonyban töltött idő alapján történik
- Illetménytábla: garantált illetmény a besorolás alapján + illetménykiegészítések
- **Jubileumi jutalom:** 25, 30, 35, 40 éves közalkalmazotti jogviszony után
- Minősítési rendszer
- Felmentési idő (akár 8 hónap is lehet, a jogviszony hosszától függően)
- Besoroláshoz szükséges adatok: iskolai végzettség, szakképzettség, szakképesítés, jogviszonyban töltött idő

---

### 3. Púétv. – Pedagógusok új életpályájáról szóló törvény (Státusztörvény)

- **Jogszabály:** 2023. évi LII. törvény
- **Hatály:** 2024. január 1-től – az állami, tankerületi és önkormányzati fenntartású köznevelési intézmények pedagógusai és nevelő-oktató munkát közvetlenül segítő dolgozói; az egyházi/magán fenntartók pedagógusaira is kiterjed a pedagógus-előmenetel
- **Jogviszony típusa:** Köznevelési foglalkoztatotti jogviszony (a korábbi közalkalmazotti jogviszony helyett)

**HRMS szempontból releváns területek:**

- **Pedagógus-előmeneteli rendszer fokozatai:** Gyakornok → Pedagógus I. → Pedagógus II. → Mesterpedagógus → Kutatótanár
- Illetménytábla a fokozat és a fizetési kategória (szakmai gyakorlati idő) alapján
- **Illetményeltérítés:** 2025. szeptember 1-jétől a munkáltató a teljesítményértékelés alapján az illetményt ±20%-kal eltérítheti
- Minősítési rendszer és teljesítményértékelés adatai
- Köznevelési foglalkoztatotti jutalom (a korábbi jubileumi jutalom helyett)
- **50 nap alapszabadság** (korábban 46)
- Többlettanítási óradíj (tartós helyettesítés, összevont csoportok)
- **Pedagóguskamarai nyilvántartás adatkörei:** OM-azonosító, munkahely neve és címe, jogviszony kezdete/vége, előmeneteli fokozat, végzettségek, etikai eljárások
- Továbbképzési kötelezettség nyilvántartása (hétévenkénti ciklus)
- **2025-ös béremelés** (453/2024. Korm. rendelet): Ped.I: +21,4%, Ped.II: +19%, Mesterpedagógus: +13,8%, Kutatótanár: +12%
- Esélyteremtési illetményrész, mesterfokozat és tantárgyak után járó illetménynövekedés

---

### 4. Eszjtv. – Egészségügyi szolgálati jogviszonyról szóló törvény

- **Jogszabály:** 2020. évi C. törvény
- **Hatály:** Állami és önkormányzati fenntartású egészségügyi szolgáltatók, azok fenntartói, és az ott egészségügyi szolgálati jogviszonyban álló személyek; egyházi fenntartók döntés alapján alkalmazhatják
- **Jogviszony típusa:** Egészségügyi szolgálati jogviszony (egészségügyi szolgálati munkaszerződés alapján)
- **Háttérjogszabály:** Mt. (az Eszjtv.-ben meghatározott eltérésekkel)
- **Végrehajtási rendelet:** 528/2020. (XI. 28.) Korm. rendelet

**HRMS szempontból releváns területek:**

- **Fizetési fokozat rendszer** az egészségügyi szolgálati jogviszonyban töltött idő alapján (beszámítható korábbi jogviszonyok részletes szabályozása)
- Illetménytábla + illetményen felüli díjak: ügyeleti díj, készenléti díj, önként vállalt többletmunka díja, képesítési pótlék, vezetői juttatás, műszakpótlék
- **Kötelező 3 hónapos próbaidő** (legfeljebb 4 hónap)
- **Alapnyilvántartás** a törvény 2. melléklete szerint – szigorúan meghatározott adatkörök, amelyeken túl adatszerzés és nyilvántartás nem végezhető
- Összeférhetetlenségi szabályok: további jogviszony létesítéséhez előzetes engedély szükséges az engedélyező szervtől
- Büntetlen előélet igazolása (részletesen meghatározott kizáró bűncselekmények)
- Kamarai tagság követelménye
- Személyi hatály: egészségügyi dolgozók (Eütev. 4. § a) és c) pont) és egészségügyben dolgozók (Eütev. 4. § b) pont)
- Magánintézmények NEM tartoznak az Eszjtv. hatálya alá

---

### 5. Kit. – Kormányzati igazgatásról szóló törvény

- **Jogszabály:** 2018. évi CXXV. törvény
- **Hatály:** Kormányzati igazgatási szervek (minisztériumok, kormányhivatalok, központi hivatalok) és foglalkoztatottjaik
- **Jogviszony típusa:** Kormányzati szolgálati jogviszony

**HRMS szempontból releváns területek:**

- **Álláshely-alapú személyügyi igazgatás** – a szervezet álláshelyekből áll, az álláshelyhez rendelt feladatok, követelmények és besorolási kategória tartozik
- Besorolási kategóriák és sávos illetményrendszer
- Teljesítményértékelés rendszere (éves ciklus)
- Hivatásetikai nyilvántartás
- Továbbképzési kötelezettség és annak nyilvántartása
- Vagyonnyilatkozat-tételi kötelezettség (bizonyos vezetői szintektől)
- Közigazgatási szakvizsga követelmény
- Próbaidő és pályázati eljárás szabályai
- Cafeteria/illetményen kívüli juttatások

---

### 6. Kttv. – Közszolgálati tisztviselőkről szóló törvény

- **Jogszabály:** 2011. évi CXCIX. törvény
- **Hatály:** Önkormányzati igazgatási szervek (jegyzői hivatal, polgármesteri hivatal, közös önkormányzati hivatal) köztisztviselői és ügykezelői; a Kit. megjelenésével a korábban egységes közszolgálati szabályozás kettévált
- **Jogviszony típusa:** Közszolgálati jogviszony (kinevezés alapján)

**HRMS szempontból releváns területek:**

- Besorolási és előmeneteli rendszer (a Kit.-hez hasonló, de önálló szabályozás)
- Illetménytábla, illetménykiegészítés, nyelvpótlék
- Közigazgatási szakvizsga nyilvántartása
- Vagyonnyilatkozat-tételi kötelezettség kezelése
- Teljesítményértékelési rendszer
- Jubileumi jutalom
- Apasági szabadságra vonatkozó szabályok 2025-ös módosítása

---

### 7. Küt. – Különleges jogállású szervekről szóló törvény

- **Jogszabály:** 2019. évi CVII. törvény
- **Hatály:** 2020. január 1-től – különleges jogállású szervek (pl. Alkotmánybíróság Hivatala, NAIH, Alapvető Jogok Biztosának Hivatala, GVH, NMHH, MTA Titkársága, Magyar Művészeti Akadémia Titkársága, Közbeszerzési Hatóság, Nemzeti Választási Iroda stb.) és az általuk foglalkoztatottak
- **Jogviszony típusa:** Közszolgálati jogviszony (sajátos szabályokkal) + lehetőség munkaviszonyra is (XVI. fejezet)

**HRMS szempontból releváns területek:**

- A Kit./Kttv.-hez hasonló, de **önálló besorolási kategóriák** – a szervek két csoportba sorolódnak (I. csoport: Köztársasági Elnöki Hivatal, AB Hivatala; II. csoport: többi szerv)
- Az álláshelyek besorolását a szervek maguk határozhatják meg a költségvetési keret figyelembevételével
- **Alaplétszám** meghatározása a szerv vezetőjének hatásköre
- Büntetlen előélet igazolása, foglalkozástól eltiltás vizsgálata
- Továbbképzési kötelezettség (499/2021. Korm. rendelet alapján)
- Az egyes szerveket létrehozó törvények eltérő rendelkezéseket állapíthatnak meg

---

## II. Adatvédelmi és adatkezelési jogszabályok

### 8. GDPR – Általános Adatvédelmi Rendelet

- **Jogszabály:** Az Európai Parlament és a Tanács (EU) 2016/679 rendelete
- **Hatály:** 2018. május 25-től közvetlenül alkalmazandó minden EU tagállamban
- **Jelentőség HRMS szempontból:** A munkavállalói személyes adatok kezelésének elsődleges jogszabályi kerete

**Főbb jogalapok munkavállalói adatkezeléshez:**

- **Szerződés teljesítése** (6. cikk (1) b)) – munkaszerződés/kinevezés alapján szükséges adatkezelés
- **Jogi kötelezettség** (6. cikk (1) c)) – adó-, TB-, munkaidő-nyilvántartási kötelezettségek teljesítése
- **Jogos érdek** (6. cikk (1) f)) – munkáltatói ellenőrzés, teljesítményértékelés (érdekmérlegelés szükséges)
- **Hozzájárulás** (6. cikk (1) a)) – munkaviszonyban csak **kivételesen** alkalmazható a felek közti alá-fölérendeltség miatt; csak ha a munkavállalót nem érheti hátrány a megtagadásból

**Különleges adatok (9. cikk):**

- Egészségügyi adatok, szakszervezeti tagság, biometrikus adatok – csak szűk kivételekkel kezelhetők
- Munkaköri alkalmassági vizsgálat adatai → jogi kötelezettség alapján kezelhetők, de az orvosi diagnózis nem

**Érintetti jogok (HRMS-ben implementálandók):**

- Hozzáférési jog (15. cikk) – a munkavállaló betekintést kérhet saját adataiba
- Helyesbítés joga (16. cikk)
- Törléshez való jog (17. cikk) – munkaviszonyban korlátozott (megőrzési kötelezettségek)
- Adathordozhatóság joga (20. cikk) – géppel olvasható formátumban
- Tiltakozás joga (21. cikk) – jogos érdek alapú adatkezelés ellen

**HRMS tervezési követelmények a GDPR-ból:**

- Beépített adatvédelem (privacy by design, 25. cikk)
- Adatvédelmi hatásvizsgálat (DPIA, 35. cikk) – magas kockázatú adatkezelés esetén
- Adatkezelési nyilvántartás (30. cikk)
- Adatvédelmi incidensek kezelése és nyilvántartása (33–34. cikk)
- Megőrzési idő korlátok és automatikus törlés/anonimizálás támogatása

---

### 9. Info tv. – Az információs önrendelkezési jogról és az információszabadságról szóló törvény

- **Jogszabály:** 2011. évi CXII. törvény
- **Hatály:** A GDPR magyar kiegészítő (háttér)normája; bűnüldözési, nemzetbiztonsági és honvédelmi célú adatkezelésre az Info tv. alkalmazandó (nem a GDPR)
- **Felügyeleti hatóság:** NAIH (Nemzeti Adatvédelmi és Információszabadság Hatóság)

**HRMS szempontból releváns területek:**

- Közérdekű adatok kezelése és közzététele – közszférás munkáltatóknál a vezetők neve, beosztása, illetménye közérdekű adat
- **2025. január 1-i módosítás:** az Áht. szerinti valamennyi törzskönyvi jogi személy köteles közzétenni meghatározott közérdekű adatokat a Központi Információs Közadat Nyilvántartás felületén (kif.gov.hu)
- Ha nem bűnüldözési/nemzetbiztonsági/honvédelmi célú az adatkezelés → GDPR alkalmazandó
- Az Info tv. 2. § (3) bekezdése a GDPR-t tette generálisan követendő normává

---

## III. Társadalombiztosítási és adójogi jogszabályok

### 10. Tbj. – Társadalombiztosítás ellátásairól szóló törvény

- **Jogszabály:** 2019. évi CXXII. törvény
- **HRMS szempontból:** Biztosítási jogviszony létrejötte, járulékfizetési kötelezettség, T1041-es bejelentés, biztosítotti bejelentés NAV felé, szolgálati idő nyilvántartása

### 11. Szja tv. – Személyi jövedelemadóról szóló törvény

- **Jogszabály:** 1995. évi CXVII. törvény
- **HRMS szempontból:** Adóelőleg-számítás, adókedvezmények (CSOK, 25 év alattiak kedvezménye, 30 év alatti anyák kedvezménye, személyi kedvezmény, első házasok kedvezménye), cafeteria/béren kívüli juttatások adózása, éves adóbevallás (M30-as igazolás)

### 12. Szocho tv. – Szociális hozzájárulási adóról szóló törvény

- **Jogszabály:** 2018. évi LII. törvény
- **HRMS szempontból:** Szociális hozzájárulási adó számítása, kedvezmények (pl. munkaerőpiacra lépők, tartósan álláskereső, megváltozott munkaképességű, kutatói kedvezmény)

---

## IV. Munkavédelmi és esélyegyenlőségi jogszabályok

### 13. Mvt. – Munkavédelmi törvény

- **Jogszabály:** 1993. évi XCIII. törvény
- **HRMS szempontból:** Munkabaleset nyilvántartás, munkavédelmi oktatás nyilvántartása, foglalkozás-egészségügyi alkalmassági vizsgálatok nyilvántartása
- **2025-ös módosítások:** munkabaleset és munkáltató fogalmának módosítása, dokumentációkezelési szabályok változása

### 14. Ebktv. – Egyenlő bánásmódról szóló törvény

- **Jogszabály:** 2003. évi CXXV. törvény
- **HRMS szempontból:** Diszkrimináció tilalma a foglalkoztatás minden fázisában (toborzás, kiválasztás, bérezés, előmenetel, megszüntetés); az HRMS-nek nem szabad olyan adatszerkezetet kialakítani, amely elősegíti a diszkriminatív döntéshozatalt

---

## V. Egyéb foglalkoztatási jogállási törvények

Egy teljes körű HRMS-nek – különösen ha közszférás ügyfeleket is kiszolgál – ismernie kell ezeket a jogviszony-típusokat is:

| Rövidítés | Törvény | Jogviszony típusa | Fő alkalmazási terület |
|-----------|---------|-------------------|----------------------|
| Hszt. | 2015. évi XLII. tv. | Hivatásos szolgálati jogviszony | Rendvédelmi szervek (rendőrség, katasztrófavédelem, büntetés-végrehajtás) |
| Honvédelmi tv. | 2021. évi CXL. tv. | Katonai szolgálati jogviszony | Honvédség |
| Bjt. | 2011. évi CLXII. tv. | Bírói szolgálati jogviszony | Bíróságok |
| Üsztv. | 2011. évi CLXIV. tv. | Ügyészségi szolgálati jogviszony | Ügyészségek |
| Iasz. | 1997. évi LXVIII. tv. | Igazságügyi alkalmazotti szolgálati jogviszony | Bírósági dolgozók (nem bírák) |
| Szkt. | 2019. évi LXXX. tv. | Munkaviszony (speciális szabályokkal) | Szakképző intézmények oktatói |
| NAVtv. | 2010. évi CXXII. tv. | Kormányzati szolgálati jogviszony (speciális) | NAV hivatásos állomány |

---

## VI. Jogviszony-típusok és háttérjogszabályok összefoglaló mátrixa

| Jogviszony típus | Elsődleges törvény | Háttérjogszabály | Jellemző munkáltató |
|---|---|---|---|
| Munkaviszony | **Mt.** | – | Versenyszféra, egyházi/magán intézmények |
| Közalkalmazotti jogviszony | **Kjt.** | Mt. | Szociális, kulturális, gyermekvédelmi intézmények |
| Köznevelési foglalkoztatotti jogviszony | **Púétv.** | Mt. (részben) | Állami/önkormányzati köznevelési intézmények |
| Egészségügyi szolgálati jogviszony | **Eszjtv.** | Mt. | Állami/önkormányzati egészségügyi szolgáltatók |
| Kormányzati szolgálati jogviszony | **Kit.** | Mt. (korlátozott) | Minisztériumok, kormányhivatalok |
| Közszolgálati jogviszony (önkormányzat) | **Kttv.** | Mt. (korlátozott) | Önkormányzati hivatalok |
| Közszolgálati jogviszony (különleges) | **Küt.** | Kit./Kttv. mintára | Különleges jogállású szervek |

---

## VII. Következő lépések

Az érintett törvények ismeretében a következő entitások tervezését kell elvégezni:

1. **Person (Személy)** – természetes személyhez kötődő, jogviszony-független alapadatok
2. **Employment (Jogviszony)** – a jogviszony típusától függő property-kkkel
3. **Organization / OrgUnit (Szervezet / Szervezeti egység)** – szektorhoz kötött struktúra
4. **Position / Job (Munkakör / Álláshely)** – jogállási törvénytől függő besorolási rendszerrel
5. **Compensation (Javadalmazás)** – illetmény, pótlékok, juttatások a jogviszony-típus szabályai szerint
6. **TimeTracking (Munkaidő-nyilvántartás)** – az Mt. és ágazati törvények előírásai alapján
7. **Leave (Szabadság/Távollét)** – jogviszony-specifikus szabadság-jogosultságok
8. **Qualification (Végzettség/Képesítés)** – besoroláshoz, előmenetelhez szükséges adatok

Minden entitásnál külön vizsgálandó:
- Mely property-k **kötelezőek jogszabály alapján** (jogi kötelezettség → GDPR 6(1)c)
- Mely property-k **szükségesek a szerződés teljesítéséhez** (GDPR 6(1)b)
- Mely property-k **különleges adatnak** minősülnek (GDPR 9. cikk)
- Milyen **megőrzési idők** vonatkoznak rájuk
- Milyen **hozzáférési szintek** szükségesek
