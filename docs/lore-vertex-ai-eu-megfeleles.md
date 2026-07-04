# Lore — Google Vertex AI: EU-adatrezidencia és zero-retention/no-training

*Kiegészítő kutatási/döntési dokumentum a Projektkoncepció → AI: futási mód és LLM-elhelyezés szekcióhoz. Azt a kérdést járja körül, hogy a Google Vertex AI konkrétan mennyire és milyen feltételekkel elégíti ki a korábban rögzített elv-alapú LLM-elhelyezési szabályt (EU-adatrezidencia + zero-retention + no-training + DPA/sub-processor-transzparencia). Állapot 2026 közepén; szerződéses feltételek és termékkínálat gyorsan változhatnak — beszerzés előtt frissen ellenőrizendő.*

---

## Vezetői összefoglaló

A Google Vertex AI **igen**, kínál megoldást mindkét kérdéskörre — EU-adatrezidenciára és zero-retention/no-training megfelelésre —, de egyik sem feltétel nélküli, automatikus alapállapot. Három szűkítés van, amit a beszerzésnél és az implementációnál explicit kezelni kell:

1. Az adatrezidencia-garancia **modellfüggő** — csak a Google saját (első-féli) modelljeire garantált, a platformon elérhető harmadik-fél modellekre nem automatikusan.
2. A zero-retention **nem alapértelmezett** — külön szerződéses lépés, amit be kell szerezni.
3. Egy konfigurációs csapda (a **globális végpont**) csendben megszüntetheti a rezidencia-garanciát, ha nem figyelünk rá.

Ezen túl a Vertex AI a hyperscaler-kategórián belül a **legerősebb** szerződéses csomagot nyújtja, de nem old meg mindent — a CLOUD Act-eredetű reziduális szuverenitási kockázat (lásd: *Projektkoncepció → AI: futási mód és LLM-elhelyezés* és *Technikai terv → EU-szuverenitás mint jövőbeli út*) egy amerikai anyacégű szolgáltatónál változatlanul fennáll.

---

## 1. EU-adatrezidencia

### Hogyan működik

A Vertex AI **régió-rögzítést** (region pinning) kínál projekt-létrehozáskor, EU-régiókkal, mint Belgium (`europe-west1`), Hollandia (`europe-west4`), Finnország (`europe-north1`) és Varsó (`europe-central2`). Google kínálja a Geminit a Vertex AI-n keresztül régió-rögzítéssel, beleértve az olyan EU-régiókat, mint Belgium, Hollandia, Finnország és Lengyelország. A garancia lényege, hogy a feldolgozás fizikailag a kiválasztott régióban marad: a Google saját Gemini-modelljei (2.5 Pro, 2.5 Flash, Imagen) a megadott régióban maradnak, a promptok és válaszok a régióban dolgozódnak fel és tárolódnak, és sosem érnek amerikai szervert.

### A modellfüggő szűkítés — ez a legfontosabb technikai csapda

A garancia **nem terjed ki egyformán mindenre**, amit a Vertex AI platform kínál. A Vertex AI harmadik-fél modelleket is kínál egy „MaaS" (Model-as-a-Service) kategórián keresztül — DeepSeek, Llama, Qwen és más nyílt forráskódú modellek —, és ezek más kategóriába esnek az adatrezidencia szempontjából, mint a Google saját modelljei. 2026 elejére a DeepSeek, Llama és Qwen elérhetőséget mutat europe-west1-ben (Belgium) és europe-west4-ben (Hollandia), de ezeket a modelleket másképp kezelik, mint a Google sajátjait — mindig ellenőrizni kell az aktuális adatrezidencia-dokumentációt, mielőtt GDPR-érzékeny munkára támaszkodnánk rájuk.

**Következmény a Lore-ra nézve:** a provider-absztrakciós rétegnek nem elég azt tudnia, hogy „Vertex AI, EU-régió" — a konkrét **modellcsaládot** is validálnia kell. MVP-re ez azt jelenti, hogy a Google első-féli modelljeire (Gemini-család) kell támaszkodni, nem a MaaS-en át elérhető nyílt modellekre, amíg ezek rezidencia-státusza nincs eset-specifikusan megerősítve.

### A globális végpont csapdája

Egyes modelleknél Google globális végpontot is kínál a jobb rendelkezésre állásért és az alacsonyabb hibaarányért, de a globális végpontnak külön kvótája van, és nem támogatja az adatrezidencia-követelményeket. Vagyis ha egy hívás — konfigurációs hiba, alapértelmezett SDK-beállítás vagy egy jövőbeli refaktor miatt — a globális végpontra megy a régió-pinnelt helyett, a rezidencia-garancia **csendben** megszűnik, hibaüzenet nélkül.

**Következmény:** ez egy explicit ellenőrzési pont, amit érdemes a teszteset-katalógusba felvenni: minden LLM-hívás igazoltan a region-pinnelt végpontra megy, sosem a globálisra. Nem elég a kódreview-ra bízni — automatizált teszt kell rá.

### Kevésbé nyilvánvaló régió-választási szempont

Két EU-régió (Frankfurt és Amszterdam/Hollandia) egyaránt GDPR-megfelelő, de eltérő modell-elérhetőséggel — Frankfurtban korlátozott a modell-elérhetőség, míg Hollandia teljes modell-támogatást nyújt. Ez tisztán gyakorlati szempont: a régió kiválasztásakor nem csak a jogi megfelelést, hanem a tényleges modell-hozzáférhetőséget is ellenőrizni kell az adott régióban.

### Mire *nem* terjed ki a rezidencia-garancia

Azoknál a régiós végpontoknál, amelyek nincsenek explicit listázva a támogatott táblázatokban — például egyes közel-keleti régióknál —, nincs garancia arra, hogy az ML-feldolgozás egy meghatározott helyen történik; ezek a régebbi modelleket kiszolgáló végpontok nem kínálnak ML-feldolgozási garanciát. Ez arra figyelmeztet, hogy a rezidencia-garancia **explicit listázott** régiókra és modellekre vonatkozik — nem általános platform-tulajdonság, hanem eset-specifikusan ellenőrizendő minden egyes modell-régió párra.

---

## 2. Zero-retention és no-training

### Az alapszabály tier-függő

A fizetős Gemini API-n és a Vertex AI-n Google feltételei kimondják, hogy nem használják fel az ügyfél promptjait vagy válaszait a termékeik fejlesztésére, ami magában foglalja a modell-tréninget is. Az ingyenes Google AI Studio tier-en és a nem fizetős Gemini API-kvótán viszont Google felhasználja a beküldött tartalmat a termékek biztosítására, fejlesztésére és javítására, és emberi felülvizsgálók is láthatják azt. Egy kivétel van: ha valaki az Európai Gazdasági Térségben, Svájcban vagy az Egyesült Királyságban van, a fizetős szolgáltatásokra vonatkozó adatfeltételek minden szolgáltatásra érvényesek, beleértve az ingyenes tier-eket is.

**Következmény:** a Lore-nak strukturálisan **csak a fizetős/enterprise Vertex AI-rétegre** szabad támaszkodnia — ez amúgy is a tervezett út —, és ez a tier-választás önmagában dokumentálandó compliance-döntés, nem csak költségdöntés.

### A zero-retention nem automatikus — külön beszerzendő

A Vertex AI zero-data-retention-ekvivalens feltételeket támogat jogosult enterprise ügyfeleknek, szerződéses módosítások útján a Data Processing Addendumhoz; ennek engedélyezéséhez a Google Cloud fiókcsapatot kell megkeresni. Ez azt jelenti, hogy a zero-retention **nem egy checkbox a konzolban**, hanem egy szerződéses tárgyalási lépés — be kell tervezni a beszerzési folyamatba, nem feltételezni alapállapotként.

### A memória-cache nüansza

Alapból a Google publikált Gemini-modelljei memóriában gyorsítótárazzák az ügyféladatot (bemenetek, kimenetek és származtatott adatok) a válaszidő csökkentésére; ez az adat csak memóriában (nem lemezen) tárolódik, projekt-szinten izolált, és 24 órás TTL-lel rendelkezik. Google álláspontja szerint ez a gyorsítótárazott adat kizárólag a szolgáltatás teljesítményének javítására szolgál, megfelel a kiválasztott helyre vonatkozó összes adatrezidencia-követelménynek, és nem sérti a zero data retentiont. Ez a funkció projekt-szinten kikapcsolható.

**Következmény:** ha a szerződési feltétel vagy egy adott ügyfél szigorúbb értelmezést kíván a „zero retention" alól (ami kizárja a 24 órás in-memory cache-t is), ezt a funkciót explicit ki kell kapcsoltatni projekt-szinten — nem elég a szerződéses zero-retention-kikötésre hagyatkozni, ha a technikai alapbeállítás ettől eltér.

### Abúzus-monitorozás

Google promptokat naplózhat biztonsági és visszaélés-észlelési céllal. Ez egy további árnyalat a „teljes" zero-retention narratívához képest — érdemes a DPA-tárgyaláson tisztázni, hogy ez a naplózás milyen megőrzési idővel és hozzáférési körrel történik.

### A szerződéses csomag összességében

A Vertex AI Gemini — az API-alapú termék fejlesztőknek és AI-alkalmazásokat építő vállalatoknak — nyújtja a legteljesebb GDPR-pozíciót: teljes enterprise DPA GDPR-specifikus mellékletekkel, zero-data-retention opciók, kizárólag EU-s következtetés a jogosult Vertex AI munkaterheléseken, és granuláris al-adatfeldolgozói kontrollok. Ezzel szemben a konzumer Gemini (gemini.google.com) nem rendelkezik adatfeldolgozási melléklettel, megőrizheti és emberi felülvizsgálatnak vetheti alá a promptokat, és nem alkalmas munkáltatói nevében kezelt EU-s személyes adat feldolgozására — ez megerősíti, hogy a termékváltozat (konzumer vs. Workspace vs. Vertex AI enterprise) alapvetően eltérő GDPR-pozíciót jelent, és a Lore-nak egyértelműen az enterprise Vertex AI rétegre kell épülnie.

---

## 3. Amit ez a technikai tervhez konkrétan hozzátesz

A korábban rögzített elv-alapú döntés (provider-absztrakció + szerződéses minimum + transzparencia) helyes maradt, de a Vertex AI-specifikus kutatás **három végrehajtási részlettel** egészíti ki:

1. **Modellcsalád-korlátozás.** A provider-absztrakciós réteg konfigurációjában MVP-re a Google első-féli modelljeire (Gemini-család) kell szűkíteni a Vertex AI-n belül — a MaaS-en át elérhető nyílt modelleket (DeepSeek, Llama, Qwen) csak eset-specifikus rezidencia-megerősítés után szabad bevonni.
2. **Végpont-fegyelem.** Minden LLM-hívásnak igazoltan region-pinnelt végpontra kell mennie, sosem a globálisra — ez felveendő a teszteset-katalógusba automatizált ellenőrzésként, nem csak kódreview-elvként.
3. **Zero-retention mint beszerzési lépés.** A zero-data-retention-ekvivalens szerződéses feltétel nem alapértelmezett — a Google Cloud fiókcsapattal külön tárgyalandó és a DPA-ban rögzítendő, a projekt-szintű in-memory cache kikapcsolásával együtt, ha a szerződéses igény ezt megköveteli.

## 4. Amit ez nem old meg

A Vertex AI EU-régiós, enterprise-szintű használata **nem szünteti meg** a korábban kifejtett CLOUD Act-eredetű reziduális szuverenitási kockázatot — a Google amerikai anyacégű vállalat, és a Standard Contractual Clauses-t Google DPA-i tartalmazzák és hivatkozhatók az EU–US adattovábbításokra, ami önmagában jelzi, hogy adattovábbítási kérdés továbbra is fennáll a háttérben. Ha egy jövőbeli ügyfél a CLOUD Act-kockázat **nullázását** írja elő feltételként, az továbbra is EU-honos szolgáltatóra váltást igényelne — amit a provider-absztrakció elve tesz lehetővé kód-átírás nélkül.

---

## Összegző táblázat

| Szempont | Vertex AI (Google első-féli modellek, EU-régió, enterprise/fizetős tier) |
|---|---|
| EU-adatrezidencia | Igen, region-pinninggel — csak explicit listázott régió-modell párokra garantált |
| Harmadik-fél (MaaS) modellek rezidenciája | Eset-specifikusan ellenőrizendő, nem automatikus |
| Zero-data-retention | Elérhető, de külön szerződéses lépés (account team + DPA-módosítás) |
| No-training | Igen, a fizetős/enterprise tier alapfeltétele — az ingyenes tier-en nem |
| In-memory cache (24h TTL) | Alapból bekapcsolva; projekt-szinten kikapcsolható |
| Globális végpont | Kerülendő — nem támogatja a rezidencia-garanciát |
| DPA / SCC | Teljes enterprise DPA, GDPR-mellékletekkel, SCC-kre hivatkozva |
| CLOUD Act reziduális kockázat | Fennáll (amerikai anyacég) — szerződéssel nem zárható ki teljesen |

*(Ellenőrizendő beszerzés előtt: a mindenkori régió-modell elérhetőségi tábla, a zero-retention-megállapodás pontos szövege, és az aktuális EU–US adattovábbítási keret állapota. Ez a dokumentum mérnöki-stratégiai tájékozódást nyújt, nem jogi tanácsot.)*
