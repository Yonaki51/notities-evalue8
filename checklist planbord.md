 
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
## portal
- [x] phase 0 - order information select boxes slaan op bij on blur.
- [x] phase 4 email sturen met incorrecte data zorgt voor error
- [x] phase 1 hardware website lijkt kapot te zijn. (==alleen bij al afgeronde orders?==)
- [ ] freshdesk update contact zonder iets in te vullen geeft error (validation en if-statement in de controller zetten)
- [ ] freshdesk "delete selected"  kan niet gebruikt worden. (ik ben blind. maak dit duidelijker.)
- [ ] Een maand kiezen in een agenda werkt niet
## core
- [x] na lange tijd inactiviteit wordt pagina een blanco wit scherm en moet je terug naar https://portal.evalue8.local/ en opnieuw inloggen
- [x] de log-in knop bij users laat niet gewoon "inloggen" zien.  (missing translation.)
- [x] user management add user: resend verification mail is kale html op de pagina
- [x] je kan save meerdere keren achter elkaar klikken
- [ ] Connection could not be established with host "mailhog:1025": stream_socket_client(): php_network_getaddresses: getaddrinfo for mailhog failed: Name or service not known
- [ ] ~~controller selecteren bij het bewerken  van labeladmins laat php documentatie zien.~~
- [ ] ~~naar de groups sectie gaan in user management opent een nieuw tablad~

support@evalue8.nl
wachtwoord123

# nieuwe functionaliteiten na testen.
- [x] config maken voor het aanpassen van het "opslaan" van de input boxes op de phase 0 page.
- [ ] Op de pagina met setups hoort een knop te staan zodat er een e-mail (vanuit sjabloon) verstuurd kan worden. (Send support installation mail)
- [ ] fases op planbord pagina gelijktrekken met de bestandnamen van de code.
