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

#### Hier is een code voorbeeld om in Node.js/Express leerlingen op te halen:

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

#### Hier is een voorbeeld hoe je de GET /students/{studentId} endpoint kunt implementeren in Express:

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