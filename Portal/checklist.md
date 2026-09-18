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

## Nieuwe functionaliteiten na bugfixes
- [x] config maken voor het aanpassen van het "opslaan" van de input boxes op de phase 0 page.
- [x] Op de pagina met setups hoort een knop te staan zodat er een e-mail (vanuit sjabloon) verstuurd kan worden. (Send support installation mail)
- [x] fases op planbord pagina gelijktrekken met de bestandnamen van de code.

## aanpassen emailtemplate
- [ ] Nieuwe clickable variable aanmaken genaamd "setuparticles". gebruik hard coded data
	- [ ] eerst kijken hoe de setup  clickable variable werkt.
- [ ] Kijken hoe de mapped variables werken. 
	- [ ] daarna inserten in email template (ook evt hardcoded data gebruiken.)
- [ ] Kijken hoe de database queries worden opgebouwd.
- [ ] database data als vardump in de mailmodal krijgen.
- [ ] opmaak voor de vardump. omzetten naar html.