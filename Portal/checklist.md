## de tekstuele aanpassingen: Fase 0 - Orderinformatie optimaliseren 
- [x] Tab Orderinformatie veranderen naar Fase 0 - Orderinformatie
	- [x] Bij de overige fases koppelstreepje tussen fase en cijfer weghalen.  
	- [x] Fase 1 Voorbereiding veranderen naar Fase 1 - Hardware  
	- [x] Invoicing naar Fase 5 - Facturering  
	- [x] validation lijkt kapot bij email templates  
- [x] toevoegen van een installateur  
	- [x] nieuwe pagina maken waar we een installateur kunnen toevoegen  
		- [x] naam
			- [x] veranderen naar installation companies
		- [x] email
- [x] integratiepartners voor DS en QM  
	- [x] nieuwe pagina maken waar we een integratiepartner kunnen toevoegen  
		- [x] naam  
			- [x] veranderen naar integration partners
		- [x] email 
		- [x] sectie (DS of QM)
- [x] integrationpartners en installateurs toevoegen aan de seeders.
- [x] de klikbare variabelen bij het aanpassen van een template fixen; die lijken nu kapot te zijn.
- [x] internet en integration providers folder verplaatsen naar plaboard/configurator.
	- [x] in de controller de views aanpassen
  
  
- [ ] Fase 0 - Orderinformatie 
	- [ ] Installatie door eValue8: Ja/Nee/N.v.t. -> Ja zorgt voor snippets  
		- [ ] Aansluitpunten stroom bekend: Ja/Nee -> Nee zorgt voor snippet: Aansluitpunt stroom onbekend.  
		- [ ] Aansluitpunten netwerk bekend: Ja/Nee -> Nee zorgt voor snippets: Aansluitpunt netwerk onbekend.  
		- [x] Installateur: Dropdown met installateurs, voor nu MDB Networks en HQ Healthcare  
			- [x] deze hoeven niet hard-coded in een migratie.
		- [ ] Stroom en netwerk aansluitingen bekend: Ja/Nee -> Ja zorgt voor snippets  
		- [ ] Netwerk gegevens bekend: Ja/Nee -> Nee zorgt voor snippets
		- [x] visuele bevestiging dat de pagina is opgeslagen.

## aanpassen emailtemplate
- [x] Nieuwe clickable variable aanmaken genaamd "setuparticles". gebruik hard coded data
	- [x] eerst kijken hoe de setup  clickable variable werkt.
- [x] Kijken hoe de mapped variables werken. 
	- [x] daarna inserten in email template (ook evt hardcoded data gebruiken.)
- [x] Kijken hoe de database queries worden opgebouwd.
- [x] oefenen met tabel layouts.
- [x] database data als vardump in de mailmodal krijgen.
- [x] opmaak voor de vardump. omzetten naar html.
- [x] gebruikte bestanden opschonen
- [x] styling toevoegen? (en op welke manier)


## planboard configurator
- [x] @can toevoegen bij knoppen die het nog niet hebben (die niet disabled zijn)
- [x] ervoor zorgen dat de "send support installation mail" knop altijd zichtbaar bljjft. Niet alleen als queue management is aangevinkt.

## planboard nieuwe functionaliteiten (snippets)
- [ ] Het toevoegen van snippets op de pagina van fase 0 - orderinformatie.
	- Deze snippets gaan lijken op de clickable variables van de mailtemplates pagina
	- [ ] dropdown "installatie evalue8" toevoegen. Bij "ja" worden de andere dropdowns weergegeven.
	- [ ] clickable variables maken voor de snippets
	- [ ] clickable variables toevoegen aan app/Livewire/Planboard/Component/ModalEmailTemplateComponent.php. Zelfde principe als setupArticles. 
		- render maken van een html, waar de if's komen voor de snippets




refactor website
	haal de debounce eruit
	maak het minder bloated

