
## planboard nieuwe functionaliteiten (snippets)
- [ ] Het toevoegen van snippets op de pagina van fase 0 - orderinformatie.
	- Deze snippets gaan lijken op de clickable variables van de mailtemplates pagina
	- [x] dropdown "installatie evalue8" toevoegen. Bij "ja" worden de andere dropdowns weergegeven.
	- [x] clickable variables maken voor de snippets
	- [ ] ~~clickable variables toevoegen aan app/Livewire/Planboard/Component/ModalEmailTemplateComponent.php. Zelfde principe als setupArticles.~~ 
		- [x] render maken van een html, waar de if's komen voor de snippets

## snippets part 2 electric boogaloo (helemaal klote D:)
- [x] migration maken met de volgende dingen
	- snippet naam
	- inhoud
	- koppeling 
- [x] nieuwe pagina toevoegen in planboard configurator on snippets te beheren
	- [x] tabel maken met de elementen die in de migration zitten
		- [x] translations maken voor de tabel
	- [x] @can maken voor de pagina
	- [x] translations maken voor de knop

## Portal Episode 3: Revenge of the snippets
- [x] Zorg dat de snippets ingeladen worden vanuit de database naar de email
- **keuze tussen twee (of meer) opties**
	- laad alle snippets in de blade pagina en kies dan welke weergegeven moeten worden.
	- laad de snippets die gekozen zijn van tevoren in op de livewire pagina en stuur deze mee naar de blade pagina.





## project log bugfixes
- [x] toevoegen/verwijderen van een artikel aan een setup zorgt niet voor een save notification
- toevoegen van seeders met fake data gaat niet helemaal lekker.
- in de project logs moet kijken of we de nieuwste records eerst tonen
	- dus eerst kijken hoe de json file wordt uitgelezen en hoe we dit kunnen filteren
- sowieso moet er nog de nieuwe table component geset worden voor de project log
- kijken welke inputs er niet gelogd worden (effe in codex gooien om te kijken waar hij mee komt)
	- It does not appear to include: <- dit had ik effe snel in codex gegooit, maar moet dus nog even dubbel gechecked worden
		- articles added to or removed from setups;
		- attachments;
		- Freshdesk ticket records;
		- sent emails;
		- other related records without an observer.
		
	- ### Concurrent writes can lose entries
		Every observer currently performs:

		1. Read the entire JSON file.
		2. Append entries in PHP.
		3. Write the entire JSON file back.

		There is no file lock. Two simultaneous autosave requests can both read the same old file, append their own change, and overwrite each other. The last write wins.
		Because the interface now autosaves many inputs, concurrent requests are realistic.
		
		Dit is de reden dat we eigenlijk de logs in de database willen opslaan omdat we hier dan geen last meer van hebben. Aangezien elke wijziging dus gelijk saved, kan wat hierboven staan dus wel voorkomen. (Als je meer wil weten vraag het even aan codex waarom dit gebeurt en wat we eraan kunnen doen)		
		
		- uitzoeken hoe we de logs kunnen opslaan in de database inplaats van een json file