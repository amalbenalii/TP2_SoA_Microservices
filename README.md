# API RESTful avec Express JS et SQLite3

## Description
Ce projet est une API RESTful simple permettant de gérer un registre de personnes. L'API permet de créer, lire, mettre à jour et supprimer des enregistrements dans une base de données SQLite3. Cette API utilise **Express.js**, un framework minimaliste pour Node.js, qui simplifie la gestion des routes et des requêtes HTTP.

## Technologies utilisées
- **Node.js** : Environnement d'exécution JavaScript côté serveur, permettant de créer des applications rapides et évolutives.
- **Express.js** : Framework pour Node.js, facilitant la création d'API en simplifiant le routage, la gestion des requêtes et des réponses.
- **SQLite3** : Base de données relationnelle légère, idéale pour les applications de petite à moyenne envergure.

## Prérequis
- **Node.js** installé sur votre machine pour exécuter le serveur.
- **Postman** (ou un autre outil de test d'API) pour envoyer des requêtes HTTP et tester l'API.

## Installation
1. Clonez le dépôt ou créez un nouveau dossier pour le projet.
2. Ouvrez un terminal et naviguez vers le dossier du projet.
3. Initialisez un nouveau projet Node.js avec la commande suivante :
   ```bash
   npm init -y
   ```
4. Installez les dépendances nécessaires pour le projet :
   ```bash
   npm install express sqlite3
   ```

## Structure des fichiers
```
/mon-projet-api
|-- database.js  # Configuration de la base de données SQLite3
|-- index.js     # Fichier principal de l'application Express.js
|-- package.json # Fichier de configuration du projet Node.js
|-- README.md    # Documentation du projet
```

## Configuration de la base de données
Dans `database.js`, nous configurons SQLite3 pour gérer notre base de données. Si la base n'existe pas, elle sera créée automatiquement avec une table `personnes`.

```javascript
const sqlite3 = require('sqlite3').verbose();

const db = new sqlite3.Database('./maBaseDeDonnees.sqlite', sqlite3.OPEN_READWRITE | sqlite3.OPEN_CREATE, (err) => {
  if (err) {
    console.error(err.message);
  } else {
    console.log('Connecté à la base de données SQLite.');
    db.run(`CREATE TABLE IF NOT EXISTS personnes (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      nom TEXT NOT NULL,
      adresse TEXT
    )`, (err) => {
      if (err) {
        console.error(err.message);
      } else {
        const personnes = ['Bob', 'Alice', 'Charlie'];
        personnes.forEach((nom) => {
          db.run(`INSERT INTO personnes (nom) VALUES (?)`, [nom]);
        });
      }
    });
  }
});

module.exports = db;
```

## Mise en place de l'API
Dans `index.js`, nous créons les différentes routes pour interagir avec la base de données. Express.js facilite la définition des routes et la gestion des requêtes HTTP.

```javascript
const express = require('express');
const db = require('./database');
const app = express();

app.use(express.json()); // Middleware pour analyser les corps de requêtes JSON

const PORT = 3000;

// Route d'accueil
app.get('/', (req, res) => {
  res.json("Registre de personnes! Choisissez le bon routage!");
});

// Récupérer toutes les personnes
app.get('/personnes', (req, res) => {
  db.all("SELECT * FROM personnes", [], (err, rows) => {
    if (err) {
      res.status(400).json({ "error": err.message });
      return;
    }
    res.json({ "message": "success", "data": rows });
  });
});

// Récupérer une personne par ID
app.get('/personnes/:id', (req, res) => {
  const id = req.params.id;
  db.get("SELECT * FROM personnes WHERE id = ?", [id], (err, row) => {
    if (err) {
      res.status(400).json({ "error": err.message });
      return;
    }
    res.json({ "message": "success", "data": row });
  });
});

// Créer une nouvelle personne
app.post('/personnes', (req, res) => {
  const { nom, adresse } = req.body;
  if (!nom) {
    res.status(400).json({ "error": "Le champ 'nom' est obligatoire." });
    return;
  }
  db.run("INSERT INTO personnes (nom, adresse) VALUES (?, ?)", [nom, adresse], function(err) {
    if (err) {
      res.status(400).json({ "error": err.message });
      return;
    }
    res.json({ "message": "success", "data": { id: this.lastID } });
  });
});

// Mettre à jour une personne
app.put('/personnes/:id', (req, res) => {
  const id = req.params.id;
  const { nom, adresse } = req.body;
  if (!nom) {
    res.status(400).json({ "error": "Le champ 'nom' est obligatoire." });
    return;
  }
  db.run("UPDATE personnes SET nom = ?, adresse = ? WHERE id = ?", [nom, adresse, id], function(err) {
    if (err) {
      res.status(400).json({ "error": err.message });
      return;
    }
    res.json({ "message": "success" });
  });
});

// Supprimer une personne
app.delete('/personnes/:id', (req, res) => {
  const id = req.params.id;
  db.run("DELETE FROM personnes WHERE id = ?", [id], function(err) {
    if (err) {
      res.status(400).json({ "error": err.message });
      return;
    }
    res.json({ "message": "success" });
  });
});

// Démarrage du serveur
app.listen(PORT, () => {
  console.log(`Serveur en cours d'exécution sur le port ${PORT}`);
});
```
## Test avec Postman
1. Ouvrez Postman et créez une nouvelle collection pour votre API.
2. Configurez les requêtes pour tester les routes (GET, POST, PUT, DELETE).
3. Exécutez les requêtes et vérifiez les réponses pour vous assurer que tout fonctionne correctement.
## Exemple de Réponses API

### GET toutes les personnes
**Requête :**
```
GET /personnes
```

**Réponse :**
```json
{
  "message": "success",
  "data": [
    { "id": 1, "nom": "Bob", "adresse": null },
    { "id": 2, "nom": "Charlie", "adresse": null },
    { "id": 3, "nom": "Alice", "adresse": null },
    { "id": 4, "nom": "Bob", "adresse": null },
    { "id": 6, "nom": "Charlie", "adresse": null },
    { "id": 7, "nom": "Bob", "adresse": null },
    { "id": 8, "nom": "Alice", "adresse": null },
    { "id": 9, "nom": "Charlie", "adresse": null }
  ]
}
```

### GET une personne par ID
**Requête :**
```
GET /personnes/3
```

**Réponse :**
```json
{
  "message": "success",
  "data": { "id": 3, "nom": "Alice", "adresse": null }
}
```

### POST (Créer une nouvelle personne)
**Requête :**
```
POST /personnes
Content-Type: application/json

{
  "nom": "David",
  "adresse": "123 Rue des Fleurs"
}
```

**Réponse :**
```json
{
  "message": "success",
  "data": { "id": 10 }
}
```

### PUT (Mettre à jour une personne)
**Requête :**
```
PUT /personnes/2
Content-Type: application/json

{
  "nom": "Charles",
  "adresse": "456 Avenue des Champs"
}
```

**Réponse :**
```json
{
  "message": "success"
}
```

### DELETE (Supprimer une personne)
**Requête :**
```
DELETE /personnes/4
```

**Réponse :**
```json
{
  "message": "success"
}
```
## Contact
**Nom de l'étudiant :** Amal Ben Ali

**Matière :** SoA et Microservices

**Enseignant :** Dr. Salah Gontara

**Année Universitaire :** 2024/2025



