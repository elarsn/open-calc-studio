# Utvecklingsplan: Open Calc Studio med klimatkalkyl

Fork av [OpenAEC-Foundation/open-calc-studio](https://github.com/OpenAEC-Foundation/open-calc-studio) för Stadsverkstad AB.
Upprättad 2026-11-26. Utgångsläge: `main` vid commit `ee1edf6` (efter release 2026.48.0).

## Mål

Ett kalkylprogram där kostnad och klimatpåverkan räknas parallellt i samma kalkyl, med spårbar data, och där en IFC-modell kan mängdas och få ett första kalkylförslag automatiskt. Verktyget är tänkt för tidiga skeden, där stomval och systemval fortfarande går att påverka.

## Avgränsningar

- Programmet gör en klimatkalkyl, inte en certifierad LCA och inte en klimatdeklaration enligt Boverket. Det ska framgå i gränssnitt och rapport.
- Bara fritt tillgänglig data byggs in. Licensierad data (till exempel Wikells, Bidcon) byggs inte in.
- Kostnadstotaler får aldrig ändras av klimatmodulen.

## Arbetssätt och grenar

| Gren | Innehåll |
|---|---|
| `main` | Speglar uppströms `main`. Inga egna ändringar. |
| `stadsverkstad` | Integrationsgren. Här ligger planen och allt eget arbete som är klart. |
| `feat/<namn>` | En gren per delsteg, slås ihop till `stadsverkstad` via pull request. |

- Synka mot uppströms inför varje nytt steg: hämta uppströms `main`, slå ihop till `stadsverkstad`, kör tester.
- Håll ändringarna additiva: nya filer och valfria fält, så få ingrepp i befintliga filer som möjligt. Det gör synken billig, eftersom uppströms släpper ungefär en version i veckan.
- Varje delsteg avslutas med `npx tsc --noEmit` och `npm run test:local` utan fel.
- Steg 1 följer uppströms egen skiss (`docs/research/calculation-packages-2026-09-28/issues/07-embodied-carbon.md`) så att det går att erbjuda tillbaka som pull request.

## Steg 1: Klimat i grundprogrammet

Datamodell, beräkning och gränssnitt. Ingen automatisk datahämtning än, faktorer matas in för hand eller följer med från ett bibliotek.

**1.1 Datamodell**
- Nytt valfritt fält på resursrader (`rowType: 'regel'`): klimattilldelning med faktor (kg CO2e per deklarerad enhet), deklarerad enhet, omräkning från radens enhet, livscykelskede, källa, version, datum och datatyp (generisk, specifik, egen).
- Samma fält valfritt på biblioteksartiklar (`ResourceLibraryItem`), så att ett bibliotek kan bära både pris och klimatfaktor.
- Mönstret följer `PriceOrigin`: ursprunget sparas på raden och är självbärande.
- Filformatet `.ifcCalc` förblir bakåtkompatibelt. Filer utan klimatdata öppnas och sparas oförändrade.

**1.2 Beräkningsmodul**
- Ren modul i `src/services/carbon/`, utan beroende till React eller store.
- Summerar bara bladresurser, uppåt genom poster och kapitel. Påslag (staart), moms och avrundning ingår inte.
- Respekterar aktiv variant, utan dubbelräkning.
- Saknad faktor ger ofullständig täckning, aldrig noll. Täckningsgrad redovisas per post, kapitel och totalt.
- Enheter: omräkning bara mellan kända enheter eller med uttrycklig omräkningsfaktor (till exempel kg per m³). Okänd kombination ger fel, ingen gissning.
- Start med A1 till A3. Modellen ska tåla A4 och A5 senare utan omskrivning.

**1.3 Gränssnitt**
- Valfria kolumner i rutnätet: klimatfaktor och kg CO2e, med tydlig markering när täckningen är ofullständig.
- Redigering av faktor och källa i egenskapspanelen.
- Summering bredvid kostnadstotalen: total kg CO2e, täckningsgrad, och kg CO2e per m² när en yta finns i projektegenskaperna.
- Jämförelse mellan varianter: kostnad och klimat sida vid sida.
- Klimatdelen går att slå av per kalkyl. Avslagen ska programmet se ut som i dag.
- Texter via i18n, minst engelska, svenska och nederländska.

**1.4 Rapport och dokumentation**
- Rapportavsnitt med antaganden, täckning och källor per faktor.
- Uppdatera `docs/ifccalc-formaat.md` och användarhandledningen.

**Klart när**
- 10 m³ med 100 kg CO2e/m³ ger 1 000 kg CO2e.
- En resurs utan faktor ger uttryckligt ofullständigt resultat.
- Kostnadstotaler är identiska med och utan klimatmodulen, i hela den befintliga testsviten.
- Spara och öppna bevarar källuppgifterna.
- Äldre filer öppnas utan ändring.

## Steg 2: Klimat- och kostnadsdata med källhantering

**2.1 Källregister**
- Varje bibliotek har ett stabilt id, version, hämtningsdatum, källadress, licens och ansvarig utgivare.
- Uppdatering visas alltid som förhandsgranskning (före och efter), på samma sätt som dagens prisuppdatering. Inget skrivs över tyst. Låsta rader rörs inte.
- En kalkyl ska kunna visa exakt vilken version av varje källa den bygger på.

**2.2 Klimatdata**

| Källa | Åtkomst | Roll |
|---|---|---|
| Boverkets klimatdatabas | Öppet API (JSON, XML), fritt med källangivelse | Grundbibliotek, generiska data. Byggs in som ögonblicksbild och kan uppdateras från programmet. |
| Egen klimatfusklapp (Excel) | Lokal fil | Eget bibliotek, importeras som JSON eller Excel. |
| Environdec EPD-bibliotek | API med konto, formatet ILCD+EPD | Senare: produktspecifika EPD:er. |
| ECO Portal | API, avgift för kommersiella verktyg | Utreds, villkoren behöver klargöras först. |

Att kontrollera mot Boverkets filspecifikation innan implementation: vilka skeden som finns (A1 till A3, A4, A5), typiska respektive konservativa värden, och hur densitet och spill anges.

**2.3 Kostnadsdata**
- Byggmästarens kalkyldatablad som eget prisbibliotek, med källa, utgåva och prisdatum på varje artikel.
- Rättigheterna till kalkyldatabladen ska klargöras innan något läggs i det publika repot. Tills dess hålls filen lokalt och läses in via "Importera bibliotek".
- Indexuppräkning med SCB:s byggkostnadsindex (öppet API): priser räknas om från prisdatum till valt datum, med index och datum redovisade. Originalpriset ligger kvar.
- Egna erfarenhetspriser från tidigare kalkyler som separat bibliotek.

**Klart när**
- Boverkets databas kan hämtas och uppdateras från programmet, och en versionsändring visas som förhandsgranskning.
- Varje pris och varje klimatfaktor i en kalkyl kan spåras till källa, version och datum.
- Programmet fungerar utan nätverk med senast hämtade data.

## Steg 3: IFC-import

**3.1 Del 1: Mängdning från IFC**
- Bygger på befintlig mängdavtagning (`src/services/ifc/ifcTakeoff.ts`), som i dag läser exporterade basmängder och kräver manuell koppling.
- Lägg till mängder beräknade ur geometri när modellen saknar basmängder, med tydlig markering av ursprung.
- Gruppering per elementtyp, material, våningsplan och klassifikation (BSAB, CoClass när det finns).
- Stöd för materialskikt, så att en vägg kan delas upp i sina lager.
- Mängdlista som går att granska innan något hamnar i kalkylen.
- Ny modellrevision ska visa vilka mängder som ändrats.

**3.2 Del 2: Automatiskt kalkylförslag**
- Regelbaserad matchning: elementtyp, material och klassifikation matchas mot byggdelsrecept. Ett recept är en post med resurser och produktionsmoment (material, arbete, maskin), var och en med pris och klimatfaktor.
- Förslag visas med motivering och säkerhetsgrad. Användaren godkänner, ändrar eller avvisar. Inget läggs in i kalkylen utan godkännande.
- Omatchade element redovisas som en egen lista, så att täckningen syns.
- Receptbiblioteket byggs upp stegvis, med start i stomme (bjälklag, väggar, pelare, balkar, grund).
- Valfritt senare: förslag via den befintliga AI-assistenten för element som reglerna inte fångar.

**Klart när**
- En stommodell kan läsas in och ge mängdlista, kalkylförslag, kostnad och klimat, med täckningsgrad för alla tre.
- Resultatet har kontrollerats mot en handräknad kalkyl på minst ett av Stadsverkstads egna projekt.

## Bygge och distribution

- Windows-installationen byggs via GitHub Actions i forken, med ett eget förenklat arbetsflöde utan kodsignering. Uppströms arbetsflöde kräver signeringsnycklar som forken inte har.
- Installationen blir osignerad, Windows varnar vid installation.
- Den inbyggda uppdateringen stängs av eller pekas om, annars skriver en officiell uppdatering över ändringarna.
- Programmet får ett eget visningsnamn och app-id så att det kan ligga installerat bredvid originalet.

## Licens

- Koden är LGPL-3.0-only. Ändringar får göras och användas fritt. Sprids programmet ska källkoden och licenstexterna följa med.
- Forken är publik. Lägg inga licensierade prislistor, kunduppgifter eller projektdata i repot.
- Boverkets data kräver källangivelse.

## Öppna frågor

1. Ska steg 1 erbjudas uppströms som pull request när det är klart?
2. Rättigheter till Byggmästarens kalkyldatablad: får innehållet spridas, eller bara användas lokalt?
3. Vilka skeden utöver A1 till A3 behövs i praktiken (A4, A5)?
4. Vilken klassifikation använder de IFC-modeller som ska läsas in (BSAB 96, CoClass, ingen)?
