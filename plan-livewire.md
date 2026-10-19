# Geneste Livewire-projecttabs vervangen door Blade

## Doel en afbakening

We vervangen de geneste Livewire-opbouw van de planboard-projectpagina’s door Blade-views, eenvoudige Blade-components en normale Laravel-requests. Het gaat om `resources/views/livewire/planboard/edit.blade.php`, de views onder `resources/views/livewire/planboard/edit/` en de bijbehorende Livewire-classes. Controllers laden de formuliergegevens. Form Requests valideren invoer en controleren rechten. Bestaande services en repositories verwerken wijzigingen.

`app/Livewire/Tables/BaseTable.php` en de daarop gebaseerde tabellen mogen blijven. We hoeven Livewire dus niet volledig uit de projectpagina’s te verwijderen. We verwijderen de Livewire-componentboom voor tabnavigatie en projectformulieren. Zelfstandige tabelcomponents en één zelfstandig e-mailcomponent zijn toegestaan. Een behouden Livewire-component mag geen andere Livewire-components renderen; Blade-components binnen zo’n component mogen wel.

Dit document beschrijft de migratie; het bevat nog geen implementatie. De inventarisatie is gebaseerd op de huidige routes, views, Livewire-classes en tests.

Onder de scope vallen:

- Project bewerken en bekijken via een gewone Blade-paginaschil.
- De zeven projectonderdelen, inclusief contactpersonen, setups, uploads en e-mailvensters, zonder geneste Livewire-formuliercomponents.
- Bestaande `BaseTable`-tabellen behouden en rechtstreeks vanuit de Blade-weergaven gebruiken, zonder bovenliggende Livewire-tabcomponents. `BaseTable` zelf hoeft hiervoor niet aangepast of vervangen te worden.

Het projectenoverzicht en project aanmaken hoeven voor deze wijziging niet te veranderen. De hele configurator blijft buiten beschouwing. De e-mailvensters binnen projecten horen wel bij de migratie.

Livewire blijft buiten deze migratiescope beschikbaar. Verwijder de Composer-dependency, globale assets of gedeelde Livewire-components dus niet als onderdeel van deze wijziging.

## 1. Huidige situatie

De routes staan in `routes/portal/planboard.php`. `PlanboardController` geeft voor bewerken en bekijken alleen een project-ID door; de Livewire-components laden vervolgens de gegevens en verwerken acties.

### Pagina’s en vervangingen

| Onderdeel | Huidige implementatie | Voorgestelde vervanging |
| --- | --- | --- |
| Projectenoverzicht | `resources/views/planboard/index.blade.php`, `app/Livewire/Planboard/ProjectTable.php` | Behouden; geen vervanging nodig |
| Project bewerken en bekijken | `resources/views/planboard/{edit,show}.blade.php`, `app/Livewire/Planboard/Edit.php` | Gedeelde Blade-weergave met server-side bepaalde bewerkrechten |
| Tabnavigatie en samenvatting | `resources/views/livewire/planboard/edit.blade.php` | Blade-tabpanelen met lokale JavaScript-tabwissels; invoer blijft in de pagina aanwezig |
| Orderinformatie | `app/Livewire/Planboard/Edit/OrderInformation.php` | Formulier voor order-, bedrijfs- en installatiegegevens; aparte acties voor Exact en Freshdesk |
| Contactpersonen | `Edit/SubComponents/ContactPersonDetails.php` | Blade-rijen en expliciete create/update/delete-acties |
| Hardware en setups | `Edit/Hardware.php`, `Edit/SubComponents/SetupDetails.php` | Formulieren voor hardware en setups; aparte acties voor artikelen, providers en bijlagen |
| Registratie | `Edit/Registration.php`, `RegistrationTable.php`, `RegistrationStatusToggle.php` | Blade-weergave per product met behoud van de `BaseTable`-tabel; statuswijzigingen via PATCH-acties |
| Verzending | `Edit/Shipping.php` | Formulier met opslaan-knop |
| Installatie | `Edit/Installation.php` | Formulier met opslaan-knop en e-mailacties |
| Facturatie | `Edit/Invoicing.php` | Formulier met opslaan-knop |
| Projectlog | `Edit/ProjectLog.php` | Server-side laden, sorteren en pagineren; lokale uitklapknoppen |
| E-mailvensters | `Component/ModalEmailTemplateComponent.php` | Behouden als zelfstandig Livewire-component, rechtstreeks aangeroepen vanuit Blade, zonder geneste Livewire-components |

De paden met `Edit/` en `Component/` in deze tabel vallen onder `app/Livewire/Planboard/`.

### Afhankelijkheden buiten de planboardmap

Bij het vervangen van de tabstructuur moeten we rekening houden met gedeelde onderdelen:

- De project-, registratie- en Freshdesk-tabellen erven van `app/Livewire/Tables/BaseTable.php`. Behoud deze basis en de bestaande tabelmogelijkheden. Render registratie- en Freshdesk-tabellen rechtstreeks vanuit de nieuwe Blade-view met een project-ID en eventuele productcontext, zonder Livewire-tabparent.
- `config/table-component.php` bevat kolommen, routes, rechten en render-components. De registratieconfiguratie verwijst ook naar de Livewire-statusschakelaar.
- `app/Livewire/Forms/EmailTemplateForm.php` bevat validatie voor de project-e-mailcomponent. Deze Livewire Form mag binnen het zelfstandige e-mailcomponent blijven; dit is geen geneste gerenderde Livewire-component.
- `app/Traits/HasAttachments.php` bevat gedeeld uploadgedrag. Behoud de regels en opslagafspraken, maar gebruik geen Livewire-uploadobjecten in nieuwe controllers.
- De TinyMCE-component staat in `app/View/Components/Core/Tinymce/TinyEditor.php` en `resources/views/components/core/tinymce/tiny-editor.blade.php`. Controleer ook de bijbehorende JavaScript-initialisatie.
- Controleer `Edit/SubComponents/AccountRow.php` en `ColorSelector.php` op daadwerkelijke aanroepplaatsen voordat we ze migreren of verwijderen.

Laat gedeelde onderdelen intact. Hergebruik `BaseTable` voor tabellen en bouw geen tweede tabelframework. Houd de nieuwe paginaschil en projectformulieren onafhankelijk van Livewire. De behouden tabellen en het e-mailcomponent beheren ieder hun eigen interactie, zonder gedeelde parentstatus.

## 2. Gewenste opzet

### Rendering en acties

- Behoud bestaande GET-route-namen voor bekijken en bewerken waar mogelijk.
- Vervang `wire:click="switchTab(...)"` en `<livewire:dynamic-component>` door Blade-tabpanelen en lokale JavaScript-tabnavigatie. Wissel tabs zonder paginaherlading, opslaan of verwijderen van eerder getoonde panelen.
- Houd de bewerkbare Blade-tabpanelen in dezelfde pagina aanwezig en verberg alleen de inactieve panelen. Daardoor blijven invoer, editorinhoud en geselecteerde bestanden bij tabwissels behouden. De panelen mogen zelfstandige tabellen en het e-mailcomponent bevatten, maar geen Livewire-tabparent.
- Laad de lokale formuliergegevens voor de projecttabs via de controller. Haal kostbare Exact- of Freshdesk-gegevens alleen op waar nodig. Als we een paneel later laden, behouden we het na de eerste keer in de pagina; vervang nooit een gewijzigd paneel met verse serverdata.
- Gebruik anonieme Blade-components onder `resources/views/components/planboard/` voor presentatie, bijvoorbeeld samenvatting, tabs, contactrij en setupformulier.
- Houd queries, API-aanroepen en opslag buiten Blade-components.
- Gebruik gewone POST-, PATCH- en DELETE-formulieren met CSRF-bescherming en waar nodig `@method(...)`.
- Stuur na een succesvolle wijziging een redirect naar dezelfde tab met een flashmelding. Zo voorkomt verversen een herhaalde formulieractie.
- Gebruik de bestaande services en repositories. Voeg alleen een service toe voor samenhangende bedrijfslogica die nu in een Livewire-component zit, zoals e-mailvoorbereiding.

Een mogelijke routeverdeling voor de implementatie:

| Actie | Methode en pad, ter voorstel |
| --- | --- |
| Project bekijken/bewerken | Bestaande `/planboard/view/{project}` en `/planboard/edit/{project}`; `?tab=shipping` selecteert het aanvankelijke paneel |
| Projectformulier opslaan | Eén PATCH-actie voor de bewerkbare gegevens over de tabs heen, met expliciete veldgroepen en validatie |
| Contactpersoon beheren | `POST /planboard/{project}/contacts`, `PATCH`/`DELETE /planboard/{project}/contacts/{contact}` |
| Setup beheren | `POST /planboard/{project}/setups`, `PATCH`/`DELETE /planboard/{project}/setups/{setup}` |
| Artikelen en bestanden beheren | Onder de betreffende project/setup-resource |
| Registratiestatus aanpassen | `PATCH /planboard/{project}/contacts/{contact}/registrations/{product}` |
| E-mail voorbereiden/versturen | Acties in het zelfstandige Livewire-component met server-side autorisatie en validatie; geen eigen AJAX-endpoint nodig |

Leg de definitieve route-namen tijdens de eerste implementatiestap vast. Gebruik afgebakende acties; maak geen endpoint dat willekeurige modelvelden accepteert.

### Opslaan en lokale interactie

**Afgesproken gedrag: geen autosave na een veldwijziging.** De gebruiker slaat projectformulierwijzigingen bewust op met een **Opslaan**-knop. Tabwissels slaan niets op en verliezen geen invoer.

- Gebruik één projectformulier met veldgroepen over de bewerkbare tabs heen en één vaste **Opslaan**-knop voor de hele projectpagina. Houd deze knop tijdens scrollen en op iedere tab zichtbaar, bijvoorbeeld in een sticky actiebalk. Plaats de knop niet in een afzonderlijk tabpaneel. Dit geldt voor de bewerkpagina; de alleen-lezen weergave krijgt geen opslaan-knop.
- Een klik op **Opslaan** verstuurt en valideert de projectformuliergegevens van alle tabs, inclusief inactieve tabs, en slaat de wijzigingen op. Laat samenhangende lokale wijzigingen in één transactie verwerken. De knop verstuurt geen e-mail en herhaalt geen afzonderlijke tabelacties.
- Laat de knop zichtbaar tijdens opslag, toon een laadstatus en voorkom dubbel indienen. Maak de actiebalk ook op kleine schermen bereikbaar zonder velden of foutmeldingen te bedekken.
- Wisselen tussen projecttabs toont geen waarschuwing: de niet-opgeslagen invoer blijft lokaal in de pagina aanwezig. Gebruik JavaScript voor zichtbaarheid en eventueel `history.replaceState` voor de tab-URL, zonder navigatie naar een nieuwe pagina.
- Gebruik de bevestigingsdialoog bij het verlaten van de projectpagina als invoer verloren zou gaan. E-mailvoorbereiding vraagt apart om opgeslagen projectgegevens.
- Toon een melding als opslaan slaagt of mislukt. Gebruik `old()` en veldfouten voor alle tabs. Open bij validatiefouten de eerste tab met fouten en markeer andere tabs met fouten; behoud ook de invoer uit de overige tabs.
- Voorkom dat browservalidatie op verborgen velden vastloopt: activeer het betreffende paneel vóór focus op een ongeldig veld. Servervalidatie blijft leidend.
- Geef aparte formulieren op dezelfde pagina eigen foutgroepen of herkenbare veldnamen, zodat fouten en oude invoer bij het juiste formulier verschijnen.
- Gebruik beperkte JavaScript voor conditionele formuliervelden, bevestigingen, modals en TinyMCE. Gebruik daarvoor geen Livewire-events of `$wire`. De behouden tabellen en het e-mailcomponent mogen hun eigen Livewire-interactie gebruiken.
- Laat kritieke acties met normale formulierverzending werken. Voeg alleen `fetch` toe als er een concrete gebruikersbehoefte is.
- Vermijd geneste HTML-formulieren. Plaats bijvoorbeeld een verwijderformulier buiten een setup-bewerkformulier.

### Navigeren met niet-opgeslagen wijzigingen

Als navigatie gewijzigde formuliergegevens zou verliezen, toon dan deze dialoog:

> Je hebt nog niet opgeslagen veranderingen, wil je deze alsnog opslaan?

Toon twee knoppen:

- **Opslaan**: sla de wijzigingen op. Ga pas na succesvolle opslag naar de oorspronkelijk gekozen bestemming.
- **Wijzigingen verwerpen**: sla niets op en ga naar de gekozen bestemming. Dit is de Nederlandstalige variant van “discard data”.

Gedrag en uitvoering:

- Vergelijk de actuele formulierwaarden met de beginwaarden. Een veld wijzigen en terugzetten naar de beginwaarde telt niet als een niet-opgeslagen wijziging. Neem checkboxes, selecties, TinyMCE-inhoud en geselecteerde uploads mee.
- Beperk wijzigingsdetectie tot bewerkbare formulieren. Zoeken, sorteren en pagineren in een behouden `BaseTable`-tabel tellen niet als niet-opgeslagen formulierdata. Als zo’n tabelactie een formulier vervangt of de pagina verlaat, geldt de waarschuwing wel.
- Onderschep navigatie die de projectpagina verlaat en invoer zou verliezen, waaronder menu-items en de terugknop waar technisch mogelijk. Gewone tabwissels binnen het project behouden de invoer en tonen geen dialoog. Toon ook geen dialoog als er niets is gewijzigd.
- Bewaar de gekozen bestemming zolang de dialoog openstaat. Gebruik bij opslaan een gecontroleerde interne return-URL; voorkom open redirects.
- Bij **Opslaan** gebruikt de pagina de normale formulieractie en servervalidatie. Bij validatie- of opslagfouten blijft de gebruiker op het formulier, met de ingevoerde waarden en foutmeldingen. Navigeer niet door en markeer het formulier niet als opgeslagen.
- Als de pagina meerdere gewijzigde formulieren bevat, moet **Opslaan** alle wijzigingen verwerken die door de navigatie verloren zouden gaan. Leg per pagina vast welke formulieren daarbij horen; voorkom dat slechts één formulier opslaat en de rest zonder waarschuwing verdwijnt. Gebruik waar nodig één samenhangende opslagactie.
- Schakel de dialoogknoppen tijdens opslaan uit om dubbel indienen te voorkomen. Een gewone klik op de opslaan-knop van het formulier mag niet opnieuw deze navigatiedialoog openen.
- Sluiten van de dialoog met Escape of het sluitkruis annuleert de navigatie en behoudt de invoer. Zorg voor focusbeheer en toetsenbordbediening.
- Gebruik bij sluiten van het browsertabblad, herladen of andere niet-onderschepbare navigatie `beforeunload` zolang er wijzigingen zijn. Browsers tonen daar hun eigen waarschuwing; de aangepaste tekst en de twee knoppen zijn daar niet beschikbaar. Automatisch opslaan tijdens het sluiten van de browser is geen betrouwbare optie.
- Reset de beginwaarden na succesvolle opslag als de pagina openblijft. Verwijder de waarschuwing pas als er geen niet-opgeslagen wijzigingen meer zijn.

## 3. Gedrag dat we moeten behouden of bewust wijzigen

### Orderinformatie en contactpersonen

`OrderInformation.php` bevat zowel veldopslag als een expliciete `save()`, Exact-import, Freshdesk-koppeling en contactbeheer. Splits deze acties zonder de bedrijfsregels te verliezen.

- Breng validatie, verplichte velden en de voorwaarden voor een voltooid project in kaart. Een nieuw conceptproject kan nog onvolledig zijn.
- Behoud de mapping tussen camelCase-invoer en databasevelden via de bestaande modelmapping, of kies expliciete veldmapping. Geef nooit ongefilterde requestdata door.
- Onderscheid `null`, `false` en `true` bij ja/nee-velden. Verwerk niet-aangevinkte checkboxes expliciet.
- Behoud productselectie, labelkeuze en de gevolgen voor vragenlijsten, hardware en registraties.
- Behoud de regels voor hoofdcontactpersoon, loginrechten en unieke e-mailadressen. Test dat we geen twee hoofdcontactpersonen krijgen.
- Laat de server voor Exact-import bepalen welke order en welk project de gebruiker mag wijzigen. Leg vast hoe import omgaat met bestaande en nog niet opgeslagen invoer.
- Behoud bruikbare foutmeldingen bij ontbrekende API-configuratie, verlopen tokens en ongeldige orders.

### Hardware, setups en registraties

- Behoud de berekening van beschikbare artikelen en aantallen per setup. Vertrouw niet op omschrijvingen of beschikbare aantallen uit een hidden input.
- Controleer aan de serverkant dat een setup, artikel, contactpersoon of bestand bij het project hoort.
- Behoud netwerkvalidatie, inclusief de voorwaardelijke velden voor een statisch IP-adres.
- Controleer wat wijzigen van de internetprovider doet voor het project en de setups. Neem dit gedrag mee in één samenhangende opslagactie.
- Behoud de voorwaarden voor integratiepartnergegevens, QMS-verwerking en Freshdesk-links.
- Verwerk productregistraties bij product- of loginrechtenwijzigingen. `Registration::mount()` synchroniseert nu registraties bij het openen van de bewerktab. Verplaats deze mutatie naar de betreffende schrijfoperatie; een GET-request hoort geen registraties aan te passen.
- Behoud de bevestiging bij het terugzetten van een afgeronde registratie en de bijbehorende datumstatus.
- Gebruik transacties voor wijzigingen die meerdere lokale records samen moeten bijwerken.

### E-mailvensters binnen projecten en Freshdesk

Behoud voor e-mailvoorbereiding en verzending één zelfstandig Livewire-component. Livewire verzorgt de asynchrone requests, uploadstatus, validatiefouten en laadstatus. Eigen AJAX- of `fetch`-code is hiervoor niet nodig. Gebruik de bestaande Freshdesk-service voor de verzending.

Roep het component rechtstreeks vanuit de Blade-projectweergave aan en geef het project-ID en e-mailtype mee. Controleer die context en de rechten opnieuw op de server bij uploaden en verzenden; vertrouw niet op browserwaarden. Het component mag geen Livewire-tabparent nodig hebben en geen andere Livewire-components renderen. Editor, bijlagenlijst en knoppen horen bij hetzelfde component en mogen gewone Blade-components gebruiken.

Voorbeeld van de gewenste opbouw:

```text
Projectpagina (Blade)
├── Projectformulier (Blade)
├── Freshdesk-tabel (zelfstandig Livewire/BaseTable)
└── E-mailvenster (zelfstandig Livewire)
    ├── Editor (Blade/JavaScript)
    ├── Bijlagenlijst (Blade)
    └── Verzendknop (Blade met Livewire-actie)
```

Controleer vóór het openen van het e-mailvenster of het projectformulier niet-opgeslagen wijzigingen heeft. Laat de gebruiker die eerst opslaan of verwerpen met de afgesproken dialoog. Laad daarna de actuele opgeslagen projectgegevens voor de e-mail, zodat de gebruiker geen e-mail op basis van verouderde gegevens verstuurt. Het verzenden van e-mail telt niet als het opslaan van projectformulierwijzigingen.

De e-mailcomponent blijft verantwoordelijk voor templatekeuze, variabelen invullen, ontvangers omzetten, bijlagen selecteren, agentcontrole en Freshdesk-verzending.

- Behoud de keuze van bestaande templates op basis van e-mailtype, product, provider en integratieprovider. Neem de validatie over die de project-e-mailcomponent nodig heeft.
- Behoud de gerenderde templatevariabelen voor project-, contact- en setupgegevens en artikeloverzichten.
- Synchroniseer TinyMCE-inhoud met het zelfstandige e-mailcomponent vóór verzending. Een lokale JavaScript-/Livewire-koppeling hiervoor is toegestaan; gebruik geen eventketen via andere Livewire-components.
- Behoud compatibiliteit voor andere gebruikers van de gedeelde editor. De editor mag binnen het e-mailcomponent `wire:ignore` gebruiken om ongewenste herinitialisatie te voorkomen.
- Behoud templatebijlagen, uitgesloten bijlagen en extra uploads. Onderscheid uitsluiten voor één verzending van definitief verwijderen.
- Valideer ontvangers en controleer de Freshdesk-agentrechten op de server. Vertrouw niet op de zichtbaarheid van een verzendknop.
- Behoud custom fields, tickethistorie en terugkoppeling na verzending. Controleer in de Freshdesk-service welke projectstatussen of verzenddatums mee veranderen.
- Voorkom dubbel verzenden. Een uitgeschakelde knop helpt, maar de server moet herhaalde inzendingen ook afvangen. Bepaal het herstelpad voor een timeout waarbij Freshdesk de mail mogelijk al heeft verwerkt.
- Houd externe API-aanroepen buiten langlopende databasetransacties. Een lokale rollback kan een verstuurde e-mail niet terugdraaien.
- Geef bij een validatiefout aan dat de gebruiker bestanden opnieuw moet kiezen. Een browser vult een file-input niet opnieuw; sla uploads alleen tijdelijk op als we daarvoor expliciet kiezen.

### Uploads, downloads en verwijderen

- Gebruik voor Blade-projectformulieren multipart-formulieren en Laravel `UploadedFile`. Het zelfstandige e-mailcomponent mag Livewire temporary uploads blijven gebruiken.
- Neem de bestaande toegestane bestandstypen, limieten, mediacollecties en opslagdisks over. Controleer ook PHP- en webserverlimieten.
- Bescherm upload-, download- en verwijderacties met de relevante import-, export-, edit- en destroy-rechten.
- Autoriseer mediabestanden via hun eigenaar. Een bestaand media-ID is geen bewijs dat een gebruiker het bestand mag gebruiken.
- Controleer ook de bestaande huisstijlroute `/planboard/corporate-identity/upload`: die accepteert nu een `projectID` uit de request. Koppel de actie bij voorkeur aan route model binding en controleer toegang tot dat project.
- Handel mislukte uploads en verweesde bestanden af zonder andere bijlagen te verwijderen.

### Tabellen en projectlog

Behoud `BaseTable` en de bestaande tabelconfiguratie. Alleen de aanroepplaats verandert: de Blade-weergave toont de bestaande Livewire-tabel rechtstreeks, zonder bovenliggende Livewire-tabcomponent. Controleer dat deze functies blijven werken:

- Kolommen, datumweergave, badges, links en lege resultaten.
- Zoekvelden, standaardfilters, meervoudige sortering, reset en paginering.
- Bewerk-, bekijk-, verwijder- en bulkacties met hun rechten en bevestigingen.
- Speciale sortering, zoals registratiestatus per product.

Behoud de bestaande Livewire-afhandeling voor zoeken, filters, sorteren, paginering en tabelacties. Controleer conflicten tussen de `tab`-parameter en de URL-status van tabellen. Gebruik aparte paginaparameters als één pagina meerdere lijsten toont.

Vervang de registratie-statusschakelaar uit de oude `Edit/`-structuur door een Blade-formulier met PATCH-actie. Pas de betreffende tabelrenderer aan zonder `BaseTable` te herschrijven. Laat de nieuwe statusactie na opslaan terugkeren naar de registratieweergave.

`ProjectLog.php` leest wijzigingslogs en gebruikt caching, sortering en paginering. Behoud de logweergave en controleer cacheverversing na opslaan. Houd updates via de bestaande opslagpaden en model-events, zodat `app/Observers/planboard/ProjectObserver.php` blijft werken. Gebruik geen bulkupdates die deze observer omzeilen.

## 4. Beveiliging en routegedrag

- Behoud de huidige buitenste `can:list planboard-projects`-controle en de aanvullende rechten per actie, tenzij we bewust het rechtenmodel aanpassen.
- Controleer rechten bij iedere schrijf-request. Een Blade `@can`, `disabled`-veld of `canEdit`-prop beveiligt geen endpoint.
- Bepaal bekijken versus bewerken op de server. De show-pagina mag geen mutatie uitvoeren, ook niet bij een handmatig verstuurde request door iemand met alleen kijkrechten.
- Controleer projecttoegang en waar van toepassing groeps-/tenantafbakening bij queries en mutaties. Gebruik scoped bindings of expliciete relatiequeries voor onderliggende resources.
- Valideer de `tab`-parameter tegen de zeven bekende tabs. Gebruik nooit gebruikersinvoer als vrije view- of componentnaam.
- Gebruik veilige interne redirectroutes in plaats van onbeperkt vertrouwen op een meegestuurde return-URL of `Referer`.
- Escape gewone tekst en laat HTML alleen toe waar de bestaande editor-/templatefunctie dat vereist. Beoordeel opgeslagen HTML en templatevariabelen op XSS.
- Beslis hoe we gelijktijdig bewerken afhandelen. Grotere formulieren kunnen oude waarden terugschrijven. Sla alleen velden van het betreffende formulier op en overweeg een versie-/`updated_at`-controle voor conflicten.

## 5. Aanpak in stappen

### Stap 1: leg het huidige gedrag vast

1. Maak per pagina en tab een lijst van velden, validatie, acties, rechten en neveneffecten uit de betreffende component.
2. Controleer de huidige schermen met een bewerker en een gebruiker met alleen kijkrechten.
3. Spreek expliciet opslaan, conceptvalidatie, e-mailvoorbereiding en het gedrag bij niet-opgeslagen wijzigingen af.
4. Leg routes, veldmapping en foutafhandeling vast. Voeg regressietests voor kritieke bedrijfsregels toe vóór het verwijderen van Livewire-code.

### Stap 2: migreer een kleine, complete gebruikershandeling

Begin met verzending: Blade-rendering, validatie, expliciete opslag en tests. Migreer tegelijk de tabschil naar lokale JavaScript-tabwissels die panelen behouden. Breid het gezamenlijke projectformulier daarna per onderdeel uit. Tijdens de tussenfase mogen nog niet gemigreerde tabs hun bestaande Livewire-component renderen; voorkom dat die het nieuwe projectformulier nesten en beschouw de migratie pas als afgerond wanneer autosave uit de projectformulieren verdwenen is.

Controleer na deze stap dat bekijken alleen leest, tabwissels invoer behouden zonder opslag, validatiefouten de invoer in alle tabs behouden en de observer wijzigingen blijft loggen. Gebruik die werkwijze daarna voor installatie en facturatie.

### Stap 3: gebruik de bestaande tabellen vanuit Blade

Behoud de registratie- en Freshdesk-tabellen als zelfstandige Livewire-components op basis van `BaseTable`. Roep ze rechtstreeks aan vanuit de Blade-weergave van het betreffende projectonderdeel, in plaats van vanuit een Livewire-tabcomponent. Geef bijvoorbeeld het project-ID en eventuele productcontext rechtstreeks mee.

De tabel behoudt zoeken, filters, sortering, paginering en bestaande tabelacties. We passen `BaseTable` hiervoor niet aan en vervangen de tabellen niet door nieuwe Blade-tabellen. Controleer alleen of een tabel nu afhankelijk is van events of reactieve properties van de oude Livewire-tabparent; vervang die afhankelijkheid waar nodig door rechtstreeks meegegeven context. Controleer autorisatie, filtering op project/product en URL-status. Het projectenoverzicht blijft ongewijzigd.

### Stap 4: migreer orderinformatie en contactbeheer

Splits opslaan, Exact-import, Freshdesk-koppeling en contactbeheer in gerichte acties. Test conceptprojecten, productwissels en contactvalidatie. Verplaats registratiesynchronisatie naar de relevante schrijfoperaties.

### Stap 5: migreer hardware, setups en registratie

Vervang setupbeheer, artikelverdeling, providerwijzigingen, netwerkvelden en registratiestatus. Voeg ownership- en transactietests toe. Migreer uploads en bestandsacties met behoud van mediaopslag.

### Stap 6: migreer e-mailstromen en projectlog

Behoud het e-mailvenster als zelfstandig Livewire-component met de bestaande Freshdesk-service. Roep het rechtstreeks vanuit Blade aan en verwijder afhankelijkheden van de oude tabparent. Test templatevariabelen, bijlagen, autorisatie, foutgevallen en laadstatus. Controleer de waarschuwing voor niet-opgeslagen project- en e-mailinvoer en voorkom dubbele verzending. Vervang daarna de projectlogweergave en controleer cachegedrag.

### Stap 7: ruim op na functionele controle

- Verwijder de vervangen Livewire-tabparent, formuliercomponents, views en listeners. Verplaats behouden tabelclasses onder `Edit/` zo nodig naar een passende tabelnamespace en werk aanroepplaatsen bij.
- Behoud `BaseTable`, gebruikte tabelcomponents, het zelfstandige e-mailcomponent en hun tests. Deel behouden components in op verantwoordelijkheid, bijvoorbeeld `Planboard/Tables/` en `Planboard/Email/`, in plaats van onder de oude tabparent. Verwijder alleen tests die uitsluitend de vervangen Livewire-projectformulier- of tabaanroepen testen.
- Behoud en verplaats tests voor bedrijfsregels naar HTTP-, service- of componenttests.
- Controleer gebruikers buiten de migratiescope voordat we gedeelde forms, traits, tabelconfiguratie of JavaScript wijzigen. Behoud de gedeelde onderdelen die zij nodig hebben.
- Verwijder planboard-specifieke debounceconfiguratie alleen als geen gebruiker overblijft.
- Werk eventuele gebruikersdocumentatie bij voor de nieuwe opslagknoppen.

Houd iedere stap werkend en afzonderlijk terug te draaien. Verwijder oude componentcode pas als de bijbehorende route en gebruikershandeling geen beroep meer doen op die code.

## 6. Verificatie

Er bestaan al relevante tests:

- `tests/Feature/Livewire/Planboard/EditTest.php`
- `tests/Feature/Livewire/OrderInformationTest.php`
- `tests/Feature/Livewire/Planboard/Component/ModalEmailTemplateComponentTest.php`
- `tests/Feature/Livewire/Traits/HasAttachmentsTest.php`
- `tests/Feature/Auth/LivewireTableAuthorizationTest.php`
- `tests/Unit/Observers/ProjectObserverTest.php`
- `tests/Unit/View/Components/TinyEditorTest.php` en `TinyEditorCompatibilityTest.php`
- De bestaande Exact-, Freshdesk- en attachment-servicetests.

Lees deze tests als gedragsbeschrijving. Pas niet alleen de aanroepen aan: laat de nieuwe tests dezelfde uitkomsten en foutgevallen bewijzen. Verwijder gedeelde Livewire-tests niet zolang andere modules ze nodig hebben.

### Vereiste controles per migratiestap

- HTTP-tests voor toegestane en geweigerde acties, ongeldige invoer en redirects.
- Tests met een onderliggend record van een ander project en, waar van toepassing, een andere tenant/groep.
- Tests voor weergave zonder mutaties en voor validatiefouten met behoud van ingevoerde waarden.
- Querytests voor filters, sortering, paginering en veilige bulkacties.
- `Storage::fake()` en uploadtests voor bestandstypen, limieten en eigenaarcontroles.
- Fakes/mocks voor Exact en Freshdesk; stuur tijdens tests geen echte e-mail en wijzig geen echte orders.
- Browsercontroles voor tabs, TinyMCE, conditionele velden, bevestigingen, uploadfouten en dubbel indienen.
- Controleer op iedere tab en bij scrollen dat de centrale **Opslaan**-knop zichtbaar blijft, ook op kleine schermen. Test dat de knop wijzigingen uit meerdere tabs opslaat en tijdens opslag een laadstatus toont zonder dubbel indienen toe te staan.
- Browsertests voor tabbehoud: wijzig meerdere tabs, wissel heen en terug en controleer dat tekst, checkboxes, TinyMCE-inhoud en geselecteerde bestanden behouden blijven, zonder opslagrequest of dialoog. Controleer dat **Opslaan** ook wijzigingen in inactieve tabs opslaat en validatiefouten geen andere tabinvoer wissen.
- Browsertests voor de navigatiedialoog bij verlaten van de projectpagina: ongewijzigde invoer, wijzigen en terugzetten, menunavigatie, opslaan en doorgaan, verwerpen en doorgaan, dialoog sluiten en validatie-/opslagfouten. Test ook meerdere gewijzigde formulieren, TinyMCE en uploads.
- Controleer de standaard browserwaarschuwing bij herladen en sluiten met gewijzigde invoer, en controleer dat deze na succesvolle opslag niet meer verschijnt.

Voer eerst de tests van het gewijzigde onderdeel uit, bijvoorbeeld `php artisan test --filter=Planboard`. Geef nieuwe tests een passende naam of kies hun expliciete pad. Draai vóór afronding de relevante bestaande regressietests en de projectbrede checks volgens de repository-/CI-configuratie.

Controleer bij de eindcontrole ook `php artisan route:list --path=planboard`, Blade-compilatie en referenties met `rg` op `<livewire:`, `@livewire`, `wire:`, `$wire`, `Livewire`, `@this` en `entangle`. Zoek in de gewijzigde projectviews, Blade-components, tabel-renderers en JavaScript. Verifieer dat de oude tabparent, dynamische Livewire-tabselectie en geneste formuliercomponents geen gebruikers meer hebben. Referenties voor behouden `BaseTable`-tabellen en het zelfstandige e-mailcomponent zijn toegestaan. Controleer dat deze components zelf geen andere Livewire-components renderen.

## 7. Klaar wanneer

- [ ] De projectpagina’s gebruiken een Blade-paginaschil en Blade-formulieren, zonder de oude geneste Livewire-tabstructuur.
- [ ] `resources/views/livewire/planboard/edit.blade.php` en de vervangen formulierviews onder `resources/views/livewire/planboard/edit/` zijn niet meer nodig.
- [ ] `BaseTable`-tabellen blijven werken als zelfstandige components, zonder afhankelijkheid van een Livewire-tabparent.
- [ ] Formulieracties gebruiken normale Laravel-routes en server-side autorisatie en validatie; behouden tabelacties mogen via `BaseTable` blijven lopen.
- [ ] GET-requests binnen de migratiescope maken of wijzigen geen domeingegevens.
- [ ] Veldwijzigingen in projectformulieren veroorzaken geen autosave. De gebruiker slaat bewust op met **Opslaan**.
- [ ] Gebruikers kunnen tussen projecttabs wisselen zonder opslag, waarschuwing of verlies van invoer, editorinhoud en geselecteerde bestanden.
- [ ] Op de bewerkpagina blijft één centrale **Opslaan**-knop zichtbaar op iedere tab en tijdens scrollen, ook op kleine schermen.
- [ ] Een klik op **Opslaan** verwerkt de projectformuliergegevens van alle tabs, inclusief wijzigingen in inactieve tabs. Tijdens opslag blijft de knop zichtbaar met een laadstatus en kan de gebruiker niet dubbel indienen.
- [ ] Bij validatiefouten blijven de waarden van alle tabs beschikbaar en opent een tab met fouten.
- [ ] Bij onderschepbare navigatie die gewijzigde invoer zou verliezen verschijnt de afgesproken dialoog met **Opslaan** en **Wijzigingen verwerpen**.
- [ ] **Opslaan** bewaart alle betrokken wijzigingen en navigeert alleen bij succes naar de gekozen bestemming; bij fouten blijft de invoer beschikbaar. **Wijzigingen verwerpen** navigeert zonder opslaan.
- [ ] Herladen en sluiten met gewijzigde invoer gebruiken de standaard browserwaarschuwing waar de browser dat ondersteunt.
- [ ] Exact-import, Freshdesk-verzending, registraties, artikelverdeling en bestandsbeheer behouden het afgesproken gedrag.
- [ ] Show-pagina’s blijven alleen-lezen; ownership- en rechtencontroles werken ook bij directe HTTP-requests.
- [ ] Projectlogs blijven wijzigingen registreren en tonen.
- [ ] Tabnavigatie en projectformulieropslag vereisen geen Livewire-runtime. Behouden tabellen en het zelfstandige e-mailcomponent mogen Livewire gebruiken, zonder geneste Livewire-components.
- [ ] Het e-mailcomponent verzorgt uploads, validatie en verzending asynchroon via Livewire en de bestaande Freshdesk-service, zonder eigen AJAX-laag.
- [ ] Bestaande gebruikers van gedeelde Livewire-code buiten de migratiescope blijven werken.
- [ ] Relevante tests, routecontroles en browsercontroles slagen.

## Nog af te spreken vóór implementatie

1. De definitieve veldgroepen en validatie voor het gezamenlijke projectformulier; expliciet opslaan en behoud van invoer bij tabwissels staan vast.
2. De presentatie van het zelfstandige Livewire-e-mailcomponent, bijvoorbeeld als Bootstrap-modal, en het gedrag bij sluiten met niet-opgeslagen e-mailinvoer.
3. De regels voor onvolledige conceptprojecten en voor import over bestaande gegevens.
4. Het gewenste gedrag bij gelijktijdig bewerken en mogelijke dubbele e-mailverzending.
5. Hoe we de behouden registratie- en Freshdesk-tabellen zonder afhankelijkheid van de oude tabparent renderen. Bestaande tabelacties blijven behouden.
