# Samenvatting van de Rest API van Informat

### Bron bestand:
#### [Leerlingen WEB API - Implementation Guide V1.11.pdf](https://helpdesk.informat.be/hc/nl/article_attachments/11827141571346)

- De Leerlingen Web API is een RESTful API die toegang biedt tot leerlinggegevens.
- Authenticatie gebeurt via OAuth 2.0 client credentials flow. Er moet een access token opgehaald worden met client id en client secret.
- Elke aanroep moet headers bevatten voor timestamp en instellingsnummer.
- Resources die beschikbaar zijn:
- Voorinschrijvingen - CRUD operaties
- Inschrijvingen - Ophalen alle inschrijvingen
- Leerlingen - Ophalen alle leerlingen en één specifieke leerling
- Foutafhandeling gebeurt via HTTP status codes en custom error codes.
- JSON wordt gebruikt voor requests en responses.

Hier is een code voorbeeld om in Node.js/Express leerlingen op te halen:

```
const express = require('express');
const fetch = require('node-fetch');

const app = express();

app.get('/students', async (req, res) => {

  // Haal access token op
  const tokenResponse = await fetch('https://identityserver.be/token', {
    method: 'POST', 
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded'
    },
    body: new URLSearchParams({
      'client_id': 'clientId',
      'client_secret': 'clientSecret',
      'grant_type': 'client_credentials' 
    })
  });

  const tokenData = await tokenResponse.json();
  const accessToken = tokenData.access_token;

  // Roep Leerlingen API aan
  const apiResponse = await fetch('https://leerlingenapi.be/students', {
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'InstituteNo': 'instellingsnummer',
      'Timestamp': new Date().toISOString() 
    } 
  });

  const students = await apiResponse.json();

  // Stuur leerlingen terug naar client
  res.json(students);

});

app.listen(3000);

```

Hier is een voorbeeld hoe je de GET /students/{studentId} endpoint kunt implementeren in Express:

```
const express = require('express');
const fetch = require('node-fetch');

const app = express();

app.get('/students/:studentId', async (req, res) => {

  const studentId = req.params.studentId;

  // Haal access token op
  // ...
  
  // Roep Leerlingen API aan
  const apiResponse = await fetch(`https://leerlingenapi.be/students/${studentId}`, { 
    headers: {
      // authorization, timestamp, etc
    }
  });

  const studentData = await apiResponse.json();

  res.json(studentData);

});

app.listen(3000);

```

### clientId en clientSecret
De clientId en clientSecret worden gebruikt in de OAuth 2.0 client credentials flow om een access token op te halen. Dit access token wordt dan meegestuurd in de Authorisation header bij elke API call naar de Leerlingen API.

In het code voorbeeld worden de clientId en clientSecret dus gebruikt in deze fetch call:
```
const tokenResponse = await fetch('https://identityserver.be/token', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded' 
  },
  body: new URLSearchParams({
    'client_id': 'clientId', 
    'client_secret': 'clientSecret',
    'grant_type': 'client_credentials'
  }) 
});
```

### client_credentials
client_credentials is hier de naam van de OAuth 2.0 grant type die gebruikt wordt. Dit geeft aan Identity Server door dat er client credentials (de clientId en clientSecret) meegestuurd worden om een access token aan te vragen.

De volledige regel in het code voorbeeld is:

```
body: new URLSearchParams({
  'client_id': 'clientId',
  'client_secret': 'clientSecret', 
  'grant_type': 'client_credentials' 
})
```

### instellingsnummer
Het instellingsnummer identificeert de school of organisatie. Het is een uniek nummer van 6 cijfers dat aan je toegekend wordt.

In het code voorbeeld wordt het instellingsnummer meegegeven in de headers van de API call:
```
const apiResponse = await fetch('https://leerlingenapi.be/students', {

  headers: {
    'Authorization': `Bearer ${accessToken}`,
    'InstituteNo': 'instellingsnummer', 
    'Timestamp': new Date().toISOString()
  }

});
```
### persoonId
Elke leerling heeft een unieke persoonId die gebruikt kan worden om die specifieke leerling op te vragen.

In het voorbeeld dat ik eerder gaf worden alle leerlingen opgehaald met deze call:
```
const apiResponse = await fetch('https://leerlingenapi.be/students', {
  // ...headers
});
```
Om één leerling op te vragen kan je het volgende endpoint gebruiken:
```
GET /students/{studentId}?schoolYear={schoolYear}&refdate={referenceDate}
```

Hierbij geef je de persoonId van de leerling mee als {studentId}.

Bijvoorbeeld:

```
const studentId = 'c5806777-4bce-4d50-afc5-9cb47f1a5f14';

const apiResponse = await fetch(`https://leerlingenapi.be/students/${studentId}`, {
  // ...headers 
});

const studentData = await apiResponse.json();
```
studentData zal nu alleen de gegevens van de opgegeven leerling bevatten.

De persoonId wordt teruggegeven in de response wanneer je alle leerlingen ophaalt, dus zo kan je de id achterhalen van een specifieke leerling.

Hier is een voorbeeld hoe je de GET /students/{studentId} endpoint kunt implementeren in Express:
```
const express = require('express');
const fetch = require('node-fetch');

const app = express();

app.get('/students/:studentId', async (req, res) => {

  const studentId = req.params.studentId;

  // Haal access token op
  // ...
  
  // Roep Leerlingen API aan
  const apiResponse = await fetch(`https://leerlingenapi.be/students/${studentId}`, { 
    headers: {
      // authorization, timestamp, etc
    }
  });

  const studentData = await apiResponse.json();

  res.json(studentData);

});

app.listen(3000);
```

Het belangrijkste is dat je de studentId opvangt uit de req.params in Express. Deze bevat de variabelen die je definiëert in de route.

Dus bij een call naar /students/123 zal req.params.studentId de waarde 123 bevatten.

De schooljaar en referentiedatum kunnen optioneel als query parameters meegegeven worden:
```
/students/123?schoolYear=2022-23&refDate=2023-01-01
```

Deze zijn beschikbaar via req.query in Express.

---
Hier is een voorbeeld om leerling data op te halen op basis van een voornaam of achternaam met de Leerlingen API in Express:

```
const express = require('express');
const fetch = require('node-fetch');

const app = express();

app.get('/students', async (req, res) => {

  const { lastName, firstName } = req.query;

  // Haal access token op
  // ...

  // Roep Leerlingen API aan met filters
  const apiResponse = await fetch('https://leerlingenapi.be/students?lastName=${lastName}&firstName=${firstName}', {
    headers: {
      // authorization, timestamp, etc 
    }
  });

  const students = await apiResponse.json();
  
  res.json(students);

});

app.listen(3000);
```

Hier wordt lastName en firstName uit de query parameters gehaald. Deze kunnen dan meegegeven worden als filters in de API call naar /students.

Je kan dit testen met requests zoals:

```
GET /students?lastName=Jansens
GET /students?firstName=Piet
GET /students?lastName=Jansens&firstName=Piet

```

De Leerlingen API zal dan de data filteren op basis van deze parameters.

Als je enkel firstName meegeeft in de query parameters, en geen lastName, dan wordt er gefilterd op basis van alleen de voornaam. lastName is in dat geval null.

De Leerlingen API negeert query parameters die niet ingevuld zijn. Dus in het voorbeeld:

```
GET /students?firstName=Piet
```

Wordt er gefilterd op voornaam Piet, en wordt lastName genegeerd omdat deze niet is meegegeven.

Je kunt dus combinaties maken van filters op zowel voornaam als achternaam, maar het is ook mogelijk om slechts één van beide mee te geven. De API zal dan filteren op basis van de ingevulde parameters en de andere negeren.

Dit geldt voor alle optionele filters in de Leerlingen API. Parameters die niet expliciet meegegeven worden, worden genegeerd en hebben geen invloed op het filteren van de resultaten.

### veldnamen
Hier is een lijst van de data velden die beschikbaar zijn in de Leerlingen API met een indicatie of ze gebruikt kunnen worden als filter:

|Veldnaam | Datatype | Doel|
|-------- | -------- | --- |
|pPerson | integer | Unique identifier of student|
|personId | GUID | Unique identifier of student|
|lastName | string | Last name of student|
|firstName | string | First name of student|
|dateOfBirth | date | Date of birth of student|
|nickname | string | Nickname of student|
|additionalNames | string | Additional first names of student|
|initials | string | Initials of student|
|countryOfBirth | string | Country of birth of student|
|placeOfBirth | string | Place of birth of student|
|nationalityCode | string | Nationality of student|
|nationalRegistrationNo | string | National registration number (Belgian students)|
|foreignRegistrationNo | string | Foreign registration number (non-Belgian students)|
|sex | string | Gender of student ('M' or 'V')|
|nameOfDoctor | string | Name of doctor of student|
|phoneOfDoctor | string | Phone number of doctor of student|
|positionInFamily | integer | Position in family of student|
|registrationId | GUID | Identifier of active registration|
|instituteNo | string | Official institute number|
|mainStructure | string | Main structure of institute|
|school | string | Name of school|
|studentNo | string | Student number of student|
|locationCode | string | Code of location|
|location | string | Name of location|
|startDate | date | Start date of registration|
|endDate | date | End date of registration|
|departmentCode | string | Code of department|
|educationNo | string | Administrative group|
|departmentYear | string | Department year|
|status | integer | Status of registration|
|degree | string | Degree|
|grade | integer | Grade|
|languageOptions | string | Chosen language options|
|financeCode | string | Financing code|
|religionCode | string | Religion|
|isOkan | boolean | OKAN indicator|
|classRegistrations | array | List of class registrations|

De velden naam en voornaam kunnen dus gebruikt worden als filterparameters. Andere velden worden genegeerd als ze meegegeven worden als filter.

Buiten de leerlingen data is er nog eventueel gekoppelde data waarin bijvoorbeeld de contacten van elke leerling opgevraagd kan worden zoals naam en voornaam van ouders met bijhorende contactgegevens. Kan ook gelinkte data per specifieke leerlingen opgevraagd worden?

Ja, in de Leerlingen API is het mogelijk om gekoppelde relatie- en contactgegevens op te vragen voor een specifieke leerling.

Wanneer je de details van een leerling ophaalt (via GET /students/{studentId}), dan bevat de response onder andere de velden:

- adressen - voor de adresgegevens van de leerling
- relaties - voor de relatiegegevens zoals ouders
- comnrs - voor bijkomende contactnummers
- emails - voor e-mailadressen

Elke relatie bevat ook weer de velden adressen, comnrs en emails met de contactgegevens voor die specifieke relatie.

Dus bijvoorbeeld voor de ouders kan je via de relaties de naam, voornaam, adres, telefoonnummers en e-mailadressen ophalen.

Je kan op deze data filters toepassen aan de client-kant, maar er zijn geen specifieke filter-opties op deze velden in de API zelf.

Bij het ophalen van de leerlingdetails krijg je dus de volledige gekoppelde gegevens voor die leerling. Filteren gebeurt dan in de verdere verwerking.

Hier is een voorbeeld om in Express de namen en e-mailadressen van de ouders op te vragen voor een specifieke leerling op basis van de studentId:

```

const express = require('express');
const fetch = require('node-fetch');

const app = express();

app.get('/students/:studentId', async (req, res) => {

  const studentId = req.params.studentId;

  // Ophalen leerling
  const apiResponse = await fetch(`https://leerlingenapi.be/students/${studentId}`);
  const student = await apiResponse.json();

  // Filteren relaties
  const parents = student.relaties.filter(r => r.type === 'Moeder' || r.type === 'Vader');
  
  // Mapperen naar naam + email
  const results = parents.map(p => {
    return {
      name: `${p.voornaam} ${p.naam}`,
      email: p.emails.find(e => e.email)?.email // optioneel email
    }; 
  });

  res.json(results);

});

app.listen(3000);
```

Het filtert eerst de relaties waar type 'Moeder' of 'Vader' is.

Vervolgens wordt dit gemapt naar een nieuw object met naam en optioneel email adres.

Dit returned bijvoorbeeld:

```
[
  { "name": "Jan Janssens", "email": "jan.janssens@email.com" },
  { "name": "Marie Peeters" }
]

```


De : achter /students/ geeft aan dat {studentId} een variable parameter is in de route.

Dus in Express kan je routes definiëren met vaste delen en variabele delen, zoals:

```
app.get('/users/:userId', (req, res) => {
  // userId is variabel
});

app.get('/posts/:postId/comments/:commentId', (req, res) => {
  // postId en commentId zijn variabel
});

```
Alles wat achter een : staat wordt een variabele parameter in Express.

De waarde van deze variabele wordt dan beschikbaar in req.params, bijvoorbeeld:

```
app.get('/users/:userId', (req, res) => {
  const userId = req.params.userId; // waarde van :userId
});
```

In het voorbeeld met /students/:studentId wordt {studentId} dus een variabele parameter die de waarde van de studentId bevat voor die specifieke route.

Dit zorgt ervoor dat je één route kan definiëren die dynamisch werkt voor elke studentId, in plaats van een aparte route te moeten maken voor elke student.


De req vertegenwoordigd dus een object met de key me naam params waarvan de waarde een object is dat dynamisch van key/value paren voorzien wordt afhankelijk van de combinatie / en : in de eerste string die de app.get functie meekrijgt. 

De req (request) in Express is een object dat alle informatie over de binnenkomende HTTP request bevat.

Een van de properties hierin is req.params. Dit is een object dat de waarden bevat van de variabele parameters in de route.

Bijvoorbeeld met de route:

```
app.get('/users/:userId/posts/:postId', (req, res) => {

  // req.params is hier: 
  {
    userId: '1234', 
    postId: '5678' 
  }

});
```

Voor elke parameter die je definieert met : in de route, wordt een key/value paar toegevoegd aan het req.params object.

De keys komen overeen met de naam (userId, postId). De values worden ingevuld op basis van de werkelijke waarden in de request URL.

Dus het is inderdaad een dynamisch object dat gevuld wordt op basis van de route definitie en de binnenkomende request.

Door req.params te gebruiken kan je op een flexibele manier data uit de URL halen in je request handlers.

### CRUD (create - read - update -delete)
Er zijn enkele minimale vereiste velden die je moet meegeven bij het creëren (C in CRUD) van een nieuwe voorinschrijving via de Leerlingen API.

Volgens de documentatie zijn de verplichte velden bij een POST /preregistrations/save request:

- lastName - Achternaam leerling
- firstName - Voornaam leerling
- dateOfBirth - Geboortedatum
- countryOfBirthCode - Geboortelandcode
- nationalityCode - Nationaliteitcode
- sex - Geslacht
- isHomeless - Indicator dakloze
- migrating - Indicator trekkende bevolking
- isIndicatorPupil - Indicatorleerling
- preRegistrationId - Unieke ID voorinschrijving
- schoolyear - Schooljaar
- institute - Instellingsnummer
- structure - Hoofdstructuur
- locationId - Vestigingsnummer
- admgrpId - Administratieve groep
- startDate - Startdatum
- registrationStatus - Status inschrijving

Daarnaast moet er minimaal één Relationships object meegegeven worden met ook verplichte velden voor naam en voornaam.

Andere velden zoals adres, contactgegevens etc. zijn optioneel.

Maar met de verplichte velden hierboven kan je al een minimale voorinschrijving aanmaken in de API.

Hier is een code voorbeeld om een bestaande voorinschrijving te updaten (U in CRUD) met de Leerlingen API:
```
const updateRegistration = async (preRegistrationId, updatedData) => {

  const response = await fetch(`https://leerlingenapi.be/preregistrations/${preRegistrationId}/update`, {
    method: 'POST',
    headers: {
      // authentication, timestamps, etc
    },
    body: JSON.stringify(updatedData) 
  });

  return response.json();

}

const updatedData = {
  lastName: 'New Last Name'  
};

updateRegistration('123e4567-e89b-12d3-a456-426614174000', updatedData);
```

Het mechanisme is hetzelfde als bij het aanmaken:

Een POST request naar /preregistrations/:id/update
De gewijzigde data meesturen in de body
De API updated dan de bestaande voorinschrijving
Let op: sommige velden zoals schooljaar, administratieve groep etc. kunnen na acceptatie van de voorinschrijving niet meer gewijzigd worden.

In dit voorbeeld wordt enkel het lastName veld geüpdatet voor de voorinschrijving met het opgegeven preRegistrationId.

De andere velden blijven ongewijzigd, alleen lastName krijgt een nieuwe waarde.

Dus stel dat de oorspronkelijke data van de voorinschrijving was:


```
{
  "lastName": "Jansen",
  "firstName": "Piet",
  // ...andere velden  
}

```
Dan zal na het uitvoeren van het voorbeeld de data geüpdatet zijn naar:

```

{
  "lastName": "New Last Name", 
  "firstName": "Piet",
  // ...andere velden blijven ongewijzigd
}
```

Je hebt groot gelijk dat het preRegistrationId van de bestaande voorinschrijving eerst opgehaald zou moeten worden, bijvoorbeeld via:

```
// Haal bestaand preRegistrationId op 
const registration = await fetchRegistrationDetails('123abc');


// Voer update uit
await updateRegistration(registration.preRegistrationId, {
  lastName: 'New Last Name'
});

```

Maar voor de eenvoud van het voorbeeld ga ik er hier van uit dat het preRegistrationId al bekend is.


De datatypes uit de lijst met velden zijn inderdaad grotendeels direct te gebruiken voor het aanmaken van overeenkomstige kolommen in een MySQL CREATE TABLE statement.

Er zijn een paar punten om rekening mee te houden:

Voor de GUID kolommen (personId, registrationId) kan je het best datatype VARCHAR(36) gebruiken in MySQL. Dit ondersteunt strings tot 36 karakters zoals GUID's.
Voor datums kun je DATETIME gebruiken in MySQL.
Voor booleans zoals isOkan gebruik je TINYINT(1) in MySQL.
Voor arrays zoals classRegistrations moet je een afzonderlijke koppel-tabel aanmaken.
Dus bijvoorbeeld:

```

CREATE TABLE students (
  pPerson INT,
  personId VARCHAR(36),
  lastName VARCHAR(50),
  firstName VARCHAR(50),
  dateOfBirth DATETIME,
  nickname VARCHAR(100),
  //...
  isOkan TINYINT(1),
  //...
)
```

Uiteraard kun je de lengtes en datatypes nog finetunen naar wens. Maar wat de types betreft kun je de lijst grotendeels zo overnemen in MySQL.

De inschatting van het potentieel aantal karakters op basis van het doel van het veld ziet er goed uit. Kun je alle velden inschatten en als een geheel verwerken de het create table students statement?

Hier is een voorbeeld CREATE TABLE statement voor de students tabel met inschattingen van de kolomtypes en lengtes op basis van het doel van de velden:

```

CREATE TABLE students (
  pPerson INT,
  personId VARCHAR(36),
  lastName VARCHAR(50), 
  firstName VARCHAR(50),
  dateOfBirth DATETIME,
  nickname VARCHAR(100),
  additionalNames VARCHAR(100),
  initials VARCHAR(20),
  countryOfBirth VARCHAR(100),
  placeOfBirth VARCHAR(100),
  nationalityCode VARCHAR(10),
  nationalRegistrationNo VARCHAR(15),
  foreignRegistrationNo VARCHAR(15), 
  sex VARCHAR(1),
  nameOfDoctor VARCHAR(100),
  phoneOfDoctor VARCHAR(20),
  positionInFamily INT,
  registrationId VARCHAR(36),
  instituteNo VARCHAR(10),
  mainStructure VARCHAR(10),
  school VARCHAR(100),
  studentNo VARCHAR(20),
  locationCode VARCHAR(10),
  location VARCHAR(100),
  startDate DATETIME,
  endDate DATETIME,
  departmentCode VARCHAR(20),
  educationNo VARCHAR(20),
  departmentYear VARCHAR(50),
  status TINYINT,
  degree VARCHAR(20),
  grade TINYINT,
  languageOptions VARCHAR(100),
  financeCode VARCHAR(10),
  religionCode VARCHAR(10),
  isOkan TINYINT(1)
);

```
Voor de arrays zoals classRegistrations zou je nog aparte koppel-tabellen moeten aanmaken.

De lengtes zijn een inschatting, dus kan je nog aanpassen waar nodig. Maar hopelijk geeft dit een goed startpunt voor het opzetten van de database tabel!

Is status een toegelaten veldnaam in mySql?

Goede vraag! status is inderdaad een gereserveerd woord in MySQL. Dit betekent dat je het niet zoals dit kan gebruiken als kolomnaam in een tabel.

Als je status probeert te gebruiken als kolomnaam zal MySQL een foutmelding geven zoals:

```
ERROR 1064 (42000): You have an error in your SQL syntax; 
check the manual that corresponds to your MySQL server 
version for the right syntax to use near 'status TINYINT' at line 1
```

Er zijn een aantal opties om dit op te lossen:

Gebruik een andere kolomnaam zoals registrationStatus in plaats van status
Zet het gereserveerde woord tussen backticks:
sql

```
`status` TINYINT
```
Voorzie de kolomnaam van een prefix of suffix:

```
registration_status TINYINT
```
```
CREATE TABLE students (
  ...,
  registration_status TINYINT,
  ...
);
```

Met Meta Data
```
CREATE TABLE students (
  registration_status TINYINT COMMENT 'Status of registration',
  
  first_name VARCHAR(50) COMMENT 'First name of student',
  
  date_of_birth DATE COMMENT 'Date of birth of student',
  
  -- ...
);
```

Op deze manier wordt de comment mee opgeslagen in de tabel definitie. Je kunt comments ook later nog toevoegen met:

```

ALTER TABLE students MODIFY COLUMN registration_status TINYINT COMMENT 'Status of registration';

```
De comment is dan zichtbaar wanneer je de tabel beschrijving opvraagt, bijvoorbeeld via:

```

SHOW FULL COLUMNS FROM students;
```

Of via een tool zoals phpMyAdmin.

Op deze manier kun je extra documentatie/metadata toevoegen aan de kolommen.



Hier is het CREATE TABLE statement voor de students tabel inclusief comments met het doel per veld:

```

CREATE TABLE students (
  pPerson INT COMMENT 'Unique identifier of student',
  
  personId VARCHAR(36) COMMENT 'Unique GUID identifier of student',
  
  lastName VARCHAR(50) COMMENT 'Last name of student',
  
  firstName VARCHAR(50) COMMENT 'First name of student',
  
  dateOfBirth DATETIME COMMENT 'Date of birth of student',
  
  nickname VARCHAR(100) COMMENT 'Nickname of student',
  
  additionalNames VARCHAR(100) COMMENT 'Additional first names of student',
  
  initials VARCHAR(20) COMMENT 'Initials of student',
  
  countryOfBirth VARCHAR(100) COMMENT 'Country of birth of student',
  
  placeOfBirth VARCHAR(100) COMMENT 'Place of birth of student',
   
  nationalityCode VARCHAR(10) COMMENT 'Nationality of student',
  
  nationalRegistrationNo VARCHAR(15) COMMENT 'National registration number (Belgian students)',

  foreignRegistrationNo VARCHAR(15) COMMENT 'Foreign registration number (non-Belgian students)',

  sex VARCHAR(1) COMMENT 'Gender of student (M or V)',

  nameOfDoctor VARCHAR(100) COMMENT 'Name of doctor of student',

  phoneOfDoctor VARCHAR(20) COMMENT 'Phone number of doctor of student',

  positionInFamily INT COMMENT 'Position in family of student',

  registrationId VARCHAR(36) COMMENT 'Identifier of active registration',

  instituteNo VARCHAR(10) COMMENT 'Official institute number',

  mainStructure VARCHAR(10) COMMENT 'Main structure of institute',
  
  school VARCHAR(100) COMMENT 'Name of school',

  studentNo VARCHAR(20) COMMENT 'Student number of student',

  locationCode VARCHAR(10) COMMENT 'Code of location',

  location VARCHAR(100) COMMENT 'Name of location',

  startDate DATETIME COMMENT 'Start date of registration',

  endDate DATETIME COMMENT 'End date of registration',

  departmentCode VARCHAR(20) COMMENT 'Code of department',

  educationNo VARCHAR(20) COMMENT 'Administrative group',

  departmentYear VARCHAR(50) COMMENT 'Department year',

  registrationStatus TINYINT COMMENT 'Status of registration',

  degree VARCHAR(20) COMMENT 'Degree',

  grade TINYINT COMMENT 'Grade',

  languageOptions VARCHAR(100) COMMENT 'Chosen language options',

  financeCode VARCHAR(10) COMMENT 'Financing code',

  religionCode VARCHAR(10) COMMENT 'Religion',

  isOkan TINYINT(1) COMMENT 'OKAN indicator'

);

```

JSON heeft geen ingebouwde ondersteuning voor metadata zoals comments zoals SQL databases.

JSON is een simpel data formaat dat enkel datawaardes beschrijft, zoals:

```

{
  "lastName": "Jansen",
  "firstName": "Piet" 
}
```

Er zijn wel een aantal opties om toch extra metadata mee te geven in een JSON document:

1. Veldnamen met prefix/suffix

Bijvoorbeeld:

```
{
  "_firstName": "Piet",
  "lastName_desc": "Achternaam"
}
```

2. Apart metadata blok

Bijvoorbeeld:

```
{
  "metadata": {
    "lastName": {
      "description": "Achternaam" 
    }
  },
  "data": {
    "lastName": "Jansen"
  }
}
```
3. Custom velden

Bijvoorbeeld:

```

{
  "lastName": "Jansen",
  "lastNameDescription": "Achternaam" 
}
```

Dus metadata kan worden meegegeven, maar is niet ingebouwd in het JSON formaat. Je moet een eigen conventie definiëren.

SQL heeft met comments dus ingebouwde ondersteuning voor metadata, wat handig is.

De type specificatie vervalt ook in JSON neem ik aan. Daarbuiten heeft het geen nut om een eigen conventie op te bouwen voor het type van elk veld.

JSON heeft een beperkt aantal datatypes:

String
Number
Boolean
Null
Object
Array

Maar je defineert niet expliciet het datatype per veld. Het wordt impliciet bepaald door de waarde.

Bijvoorbeeld:


```
{
  "lastName": "Jansen", // String
  "age": 35, // Number
  "admin": false // Boolean
}

```

Dus samenvattend:

Geen expliciete datatype definitie in JSON
Impliciet bepaald door value
Eigen conventie voor datatypes heeft weinig meerwaarde

Hier is een voorbeeld van hoe je de metadata voor de students tabel velden kunt opnemen in een JSON object met camelCase keys:

```

{
  "pPersonDescription": "Unique identifier of student",
  "personIdDescription": "Unique GUID identifier of student",
  "lastNameDescription": "Last name of student",
  "firstNameDescription": "First name of student",
  "dateOfBirthDescription": "Date of birth of student",
  "nicknameDescription": "Nickname of student",
  "additionalNamesDescription": "Additional first names of student",
  "initialsDescription": "Initials of student",
  "countryOfBirthDescription": "Country of birth of student",
  "placeOfBirthDescription": "Place of birth of student",
  "nationalityCodeDescription": "Nationality of student",
  "nationalRegistrationNoDescription": "National registration number (Belgian students)",
  "foreignRegistrationNoDescription": "Foreign registration number (non-Belgian students)",
  "sexDescription": "Gender of student (M or V)",
  "nameOfDoctorDescription": "Name of doctor of student",
  "phoneOfDoctorDescription": "Phone number of doctor of student",
  "positionInFamilyDescription": "Position in family of student",
  "registrationIdDescription": "Identifier of active registration",
  "instituteNoDescription": "Official institute number",
  "mainStructureDescription": "Main structure of institute",
  "schoolDescription": "Name of school",
  "studentNoDescription": "Student number of student",
  "locationCodeDescription": "Code of location",
  "locationDescription": "Name of location",
  "startDateDescription": "Start date of registration",
  "endDateDescription": "End date of registration",
  "departmentCodeDescription": "Code of department",
  "educationNoDescription": "Administrative group",
  "departmentYearDescription": "Department year",
  "registrationStatusDescription": "Status of registration",
  "degreeDescription": "Degree",
  "gradeDescription": "Grade",
  "languageOptionsDescription": "Chosen language options",
  "financeCodeDescription": "Financing code",
  "religionCodeDescription": "Religion",
  "isOkanDescription": "OKAN indicator"
}

```
Zo kun je de metadata op een gestructureerde manier opnemen, zonder dat er al datawaardes voor de velden aanwezig hoeven te zijn.

De metadata met de omschrijvingen geeft al voldoende context over het doel van de velden.

Het is aan te raden om te wachten tot de data beschikbaar is, en dan het object op te bouwen met zowel keys als values, bijvoorbeeld:

```

{
  "pPerson": 123,
  "pPersonDescription": "Unique identifier of student",
  
  "lastName": "Jansen",
  "lastNameDescription": "Last name of student",
  
  "dateOfBirth": "1990-03-25",
  "dateOfBirthDescription": "Date of birth of student"
}

```

Op die manier hou je het object clean en bevat het enkel relevante data zodra dit beschikbaar is.

De keys zonder waarden kunnen dus beter weggelaten worden. De metadata in de omschrijvingen is voldoende context.

Zodra de data er is, kun je het dan aanvullen zoals hierboven.

De response van de Leerlingen API is opgebouwd volgens een standaard structuur, die vervolgens getailored kan worden naar behoefte.

De standaard response ziet er ongeveer als volgt uit:

```

{
  "metadata": {
    "timestamp": "...",
    "apiVersion": "1",
    // etc
  },

  "student": {
    "id": "...",
    "firstName": "...",
    // student fields
  },

  "linkedData": {
    "adresses": [
      // array of adress objects
    ],
    "contacts": [
      // array of contact objects 
    ]
  }

}

```
Dus je hebt:

Metadata over de API call
De basisdata van de student
Geneste datastructuren voor gekoppelde data zoals adressen en contacten
Deze structuur is standaard voor elke student. Maar je kunt het antwoord tailoren door bijvoorbeeld:

Specifieke velden van de student op te vragen via query parameters
Nested data weg te laten indien niet nodig
Dus je kunt bijvoorbeeld een call doen voor enkel:

```

{
  "student": {
    "firstName": "Piet"
  }
}

```

Door de standaard structuur kun je het antwoord heel flexibel aanpassen aan je behoefte.
