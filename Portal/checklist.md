
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




## feedback
- [x] fragmenten veranderen naar snippets
- [ ] ~~switch case voor allemaal if-statements?~~
- [ ] "selecteer optie" weghalen en als default op nee zetten.

