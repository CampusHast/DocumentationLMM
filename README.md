# Introductie

- Het doel is een management systeem te realiseren voor alle uitgeleende laptops, iPads, e-readers, etc. op de diverse middelbare scholen van de scholengemeenschap `Sint-Quintinus`.

- Per toestel wil men bijhouden aan welke leerling het is uitgeleend, of er een ondertekend uitleencontract aanwezig is, welk digitaal leermiddel is uitgeleend, wanneer het is uitgeleend, in welke staat het zich bevindt, welke bijbehorende accessoires zijn uitgeleend, contractdetails, etc.

# Mogelijke oplossingen

- Webgebaseerde tool met database voor de inventory. 

- Node.js en Express als backend, React voor de frontend.

- Gebruik van MongoDB als NoSQL database voor flexibiliteit.

- Authenticatie via Active Directory van de school. 

- Genereren van PDF contracten en uitleenbevestigingen.

- Push notificaties naar leerlingen/ouders via SMS of mail.

- Rapportagefuncties en dashboard. 

- Mobiele app voor uitleenproces met scanning/RFID. 

- Laravel als overkoepelend framework.

- PowerShell scripts via Intune voor beheer van Windows laptops.

# Documentatie

De documentatie is met MkDocs opgezet als statische site generator. Hier gaat de incrementale documentatie komen voor de stapsgewijze opbouw van dit project.

Om de website op te starten voer je onderstaande commando in terminal:

```
mkdocs serve
```
