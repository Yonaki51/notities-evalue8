 
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
			- [ ] willekeurig email verwisselen naar de echte emails.
		- [ ] Stroom en netwerk aansluitingen bekend: Ja/Nee -> Ja zorgt voor snippets  
		- [ ] Netwerk gegevens bekend: Ja/Nee -> Nee zorgt voor snippets
		- [x] visuele bevestiging dat de pagina is opgeslagen.
 
# Bugs
- [ ] na lange tijd inactiviteit wordt pagina een blanco wit scherm en moet je terug naar https://portal.evalue8.local/ en opnieuw inloggen
- [ ] phase 0 - order information select boxes slaan op bij on blur.
- [ ] phase 4 email sturen met incorrecte data zorgt voor error
- [ ] Een maand kiezen in een agenda werkt niet
- [ ] geen knop om een project op te slaan?

