 
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
 
# Bugs
- [ ] na lange tijd inactiviteit wordt pagina een blanco wit scherm en moet je terug naar https://portal.evalue8.local/ en opnieuw inloggen
- [x] phase 0 - order information select boxes slaan op bij on blur.
- [x] phase 4 email sturen met incorrecte data zorgt voor error
- [ ] Een maand kiezen in een agenda werkt niet
- [ ] geen knop om een project op te slaan?
- [ ] controller selecteren bij het bewerken  van labeladmins laat php documentatie zien.
- [ ] user aanmaken lukt wel bij user management, maar laat wel een error zien. Ook bij Deleten
- [ ] de log-in knop bij users laat niet gewoon "inloggen" zien.  lijkt code te displayen.
- [ ] naar de groups sectie gaan in user management opent een nieuw tablad
- [ ] freshdesk update contact zonder iets in te vullen geeft error
- [ ] freshdesk "delete selected"  kan niet gebruikt worden.
- [ ] phase 1 hardware website lijkt kapot te zijn. (==alleen bij al afgeronde orders?==)


# nieuwe functionaliteiten na testen.
- [x] config maken voor het aanpassen van het "opslaan" van de input boxes op de phase 0 page.
- [ ] Op die pagina met setups hoort een knop te staan zodat er een e-mail (vanuit sjabloon) verstuurd kan worden. (Send support installation mail)

