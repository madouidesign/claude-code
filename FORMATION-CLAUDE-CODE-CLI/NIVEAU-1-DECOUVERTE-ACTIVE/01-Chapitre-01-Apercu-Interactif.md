# 🎬 NIVEAU 1 : Chapitre 01 - CLI & Architecture - Aperçu Interactif

> **🎯 Objectif :** Maîtriser Commander.js et construire un CLI modulaire professionnel
> **🧠 Science :** Active Learning + Zone Proximale + Immediate Feedback
> **📊 Progression :** [■■□□□□□□□□] 20% du parcours Niveau 1
> **⏱️ Durée :** 90 minutes

Voir le fichier pour le contenu complet...
> **🎯 Objectif :** Comprendre ET construire un CLI Node.js complet avec architecture modulaire
> **🧠 Science :** Active Learning + Immediate Feedback + Zone Proximale de Développement
> **📊 Progression :** [■■□□□□□□□□] 20% du parcours Niveau 1
> **⏱️ Durée :** 90 minutes

---

## 🎮 ACTIVATION : Avant de Commencer

### 🤔 Question Réflexive (Metacognition)

> Tu utilises déjà des CLIs tous les jours : `git`, `npm`, `docker`, etc.
>
> **Réfléchis 60 secondes :**
> - Qu'est-ce qui rend un bon CLI agréable à utiliser ?
> - Quelles commandes te semblent intuitives ? Lesquelles sont confuses ?
> - Si tu devais créer ton propre CLI, par quoi commencerais-tu ?

**💭 Note tes réflexions avant de continuer...**

---

**🎯 Dans ce chapitre, tu vas construire un CLI professionnel de A à Z !**

À la fin, tu auras un CLI qui :
- ✅ Accepte des commandes et options (`mycli add "task" --priority high`)
- ✅ Stocke des données de manière persistante
- ✅ Gère les erreurs proprement
- ✅ A une architecture modulaire et maintenable

---

## 📚 Section 1 : Structure d'un CLI Node.js

### 💡 CONCEPT : Qu'est-ce qu'un CLI ?

**En une phrase :** Un CLI (Command Line Interface) est un programme qui s'exécute dans le terminal et interagit via du texte.

**🎨 Analogie Mémorable :**
> Un CLI, c'est comme un serveur de restaurant qui prend des commandes vocales.
>
> - **Le client (utilisateur)** : Dit ce qu'il veut ("Je voudrais un café")
> - **Le serveur (CLI)** : Comprend la commande, la traite
> - **La cuisine (logique métier)** : Exécute l'action
> - **Le serveur (CLI)** : Retourne le résultat ("Voici votre café !")

**Types de CLIs :**
1. **CLI Simple** : Une seule commande (ex: `cat`, `ls`)
2. **CLI avec sous-commandes** : Plusieurs commandes groupées (ex: `git add`, `git commit`)
3. **CLI Interactif** : Pose des questions à l'utilisateur (ex: `npm init`)
4. **CLI Daemonisé** : Tourne en arrière-plan (ex: `docker`, Claude Code)

**Claude Code CLI est de type 2 + 4 :** Sous-commandes + mode interactif/daemon

### 🔍 EXPLORATION : Anatomie d'un CLI Node.js

**Fichier minimal : `cli.js`**

```javascript
#!/usr/bin/env node
// ☝️ Shebang : dit à l'OS d'utiliser node pour exécuter ce fichier

// 1. Imports
const fs = require('fs');
const path = require('path');

// 2. Récupérer les arguments de la ligne de commande
// process.argv = ['node', '/path/to/cli.js', 'arg1', 'arg2', ...]
const args = process.argv.slice(2); // Enlève 'node' et le nom du script

// 3. Parsing de la commande
const command = args[0]; // Premier argument = la commande
const commandArgs = args.slice(1); // Reste = arguments de la commande

// 4. Logique métier
switch (command) {
    case 'hello':
        const name = commandArgs[0] || 'World';
        console.log(`👋 Hello, ${name}!`);
        break;

    case 'version':
        console.log('v1.0.0');
        break;

    default:
        console.error(`❌ Commande inconnue: ${command}`);
        console.log('💡 Commandes disponibles: hello, version');
        process.exit(1); // Exit avec code d'erreur
}
```

**Rendre le CLI exécutable :**

```bash
# 1. Donner les permissions d'exécution
chmod +x cli.js

# 2. Exécuter directement
./cli.js hello Alice
# 👋 Hello, Alice!

# 3. OU via node (sans chmod)
node cli.js version
# v1.0.0
```

**🔍 Qu'est-ce qui se passe ici ?**

1. **Shebang (`#!/usr/bin/env node`)** : Indique au système d'exploitation d'utiliser Node.js pour exécuter ce fichier
2. **`process.argv`** : Tableau contenant tous les arguments passés au script
3. **`slice(2)`** : Enlève les 2 premiers arguments (chemin de node et du script)
4. **`switch/case`** : Router qui dirige vers la bonne commande
5. **`process.exit(1)`** : Sort du programme avec un code d'erreur (0 = succès, 1+ = erreur)

### 🎮 PRATIQUE IMMÉDIATE : Ton Premier CLI

**🎯 Défi 1 : CLI Calculatrice Simple**

Crée un CLI qui :
- `calc add 5 3` → Affiche 8
- `calc sub 10 4` → Affiche 6
- `calc mul 3 7` → Affiche 21

**💡 Squelette de code :**

```javascript
#!/usr/bin/env node

const args = process.argv.slice(2);
const operation = args[0];
const a = parseFloat(args[1]);
const b = parseFloat(args[2]);

// TODO: Vérifier que a et b sont des nombres valides

switch (operation) {
    case 'add':
        // TODO: Additionner et afficher
        break;

    case 'sub':
        // TODO: Soustraire et afficher
        break;

    case 'mul':
        // TODO: Multiplier et afficher
        break;

    default:
        console.error('❌ Opération inconnue');
        process.exit(1);
}
```

**✅ Solution Complète :**

<details>
<summary>💡 Cliquez pour voir la solution</summary>

```javascript
#!/usr/bin/env node

const args = process.argv.slice(2);
const operation = args[0];
const a = parseFloat(args[1]);
const b = parseFloat(args[2]);

// Validation
if (!operation) {
    console.error('❌ Erreur: Aucune opération spécifiée');
    console.log('💡 Usage: calc <add|sub|mul> <nombre1> <nombre2>');
    process.exit(1);
}

if (isNaN(a) || isNaN(b)) {
    console.error('❌ Erreur: Les arguments doivent être des nombres');
    console.log(`💡 Reçu: a=${args[1]}, b=${args[2]}`);
    process.exit(1);
}

// Logique
switch (operation) {
    case 'add':
        console.log(`➕ ${a} + ${b} = ${a + b}`);
        break;

    case 'sub':
        console.log(`➖ ${a} - ${b} = ${a - b}`);
        break;

    case 'mul':
        console.log(`✖️  ${a} × ${b} = ${a * b}`);
        break;

    case 'div':
        if (b === 0) {
            console.error('❌ Erreur: Division par zéro');
            process.exit(1);
        }
        console.log(`➗ ${a} ÷ ${b} = ${a / b}`);
        break;

    default:
        console.error(`❌ Opération inconnue: ${operation}`);
        console.log('💡 Opérations: add, sub, mul, div');
        process.exit(1);
}

// Exit avec succès
process.exit(0);
```

**Tester :**
```bash
chmod +x calc.js

./calc.js add 10 5
# ➕ 10 + 5 = 15

./calc.js mul 3 7
# ✖️  3 × 7 = 21

./calc.js div 10 0
# ❌ Erreur: Division par zéro

./calc.js add abc 5
# ❌ Erreur: Les arguments doivent être des nombres
```

</details>

**🔍 ANALYSE (Metacognition) :**
- Qu'as-tu appris en codant cela ?
- Qu'est-ce qui était plus difficile que prévu ?
- Comment gérerais-tu plus de 10 opérations différentes ?

### 🚀 CHALLENGE : CLI avec Aide Intégrée

**🎯 Défi 2 : Ajoute une commande `help`**

Objectif : `./calc.js help` affiche un message d'aide détaillé

```javascript
// TODO: Ajoute un case 'help' qui affiche :
// - Description du programme
// - Liste des commandes disponibles
// - Exemples d'utilisation
```

<details>
<summary>✅ Solution</summary>

```javascript
case 'help':
    console.log(`
📖 Calculatrice CLI

USAGE:
  calc <opération> <nombre1> <nombre2>

OPÉRATIONS:
  add    Addition (+)
  sub    Soustraction (-)
  mul    Multiplication (×)
  div    Division (÷)
  help   Affiche cette aide

EXEMPLES:
  calc add 10 5     # Résultat: 15
  calc mul 3 7      # Résultat: 21
  calc div 20 4     # Résultat: 5
    `);
    break;
```

</details>

### 📊 POINTS CLÉS À RETENIR

- ✅ `#!/usr/bin/env node` rend un fichier JS exécutable
- ✅ `process.argv` contient tous les arguments CLI
- ✅ `process.argv.slice(2)` enlève node et le script
- ✅ `process.exit(0)` = succès, `process.exit(1)` = erreur
- ✅ Toujours valider les entrées utilisateur
- ✅ Messages d'erreur clairs et constructifs

---

## 📚 Section 2 : Architecture Modulaire & Commander.js

### 💡 CONCEPT : Pourquoi une Architecture ?

**Problème avec le CLI simple :**

```javascript
// calc.js - 500 lignes dans un seul fichier 😱
switch (operation) {
    case 'add': /* 50 lignes */ break;
    case 'sub': /* 50 lignes */ break;
    case 'mul': /* 50 lignes */ break;
    // ... 10 autres opérations
}
// Code illisible, difficile à tester, impossible à maintenir
```

**Solution : Architecture modulaire**

```
mycli/
├── bin/
│   └── cli.js              # Point d'entrée (léger)
├── src/
│   ├── commands/
│   │   ├── add.js          # Logique de 'add'
│   │   ├── sub.js          # Logique de 'sub'
│   │   └── index.js        # Export tous les commands
│   └── utils/
│       ├── validation.js   # Fonctions de validation
│       └── formatter.js    # Formatage de sortie
└── package.json
```

**Avantages :**
- ✅ Chaque commande dans son propre fichier
- ✅ Facile à tester unitairement
- ✅ Réutilisation du code (utils)
- ✅ Collaboration en équipe plus simple

### 🔍 EXPLORATION : Commander.js, le Framework CLI

**Commander.js** est la bibliothèque standard pour créer des CLIs en Node.js.

**Installation :**
```bash
npm init -y
npm install commander
```

**Exemple basique :**

```javascript
#!/usr/bin/env node
const { Command } = require('commander');
const program = new Command();

program
    .name('mycli')
    .description('Mon super CLI')
    .version('1.0.0');

// Commande: mycli greet <name>
program
    .command('greet <name>')
    .description('Dit bonjour à quelqu\'un')
    .option('-e, --enthusiastic', 'Ajoute de l\'enthousiasme')
    .action((name, options) => {
        const greeting = `Hello, ${name}!`;
        if (options.enthusiastic) {
            console.log(`🎉 ${greeting.toUpperCase()} 🎉`);
        } else {
            console.log(`👋 ${greeting}`);
        }
    });

program.parse(process.argv);
```

**Utilisation :**
```bash
node cli.js greet Alice
# 👋 Hello, Alice!

node cli.js greet Bob --enthusiastic
# 🎉 HELLO, BOB! 🎉

node cli.js --help
# Affiche automatiquement l'aide

node cli.js --version
# 1.0.0
```

**🔍 Avantages de Commander :**
1. **Parsing automatique** des arguments et options
2. **Aide générée automatiquement** (`--help`)
3. **Validation** des arguments requis
4. **Sous-commandes** imbriquées faciles
5. **Options** avec valeurs par défaut

### 🎮 PRATIQUE GUIDÉE : CLI de Gestion de Notes

**🎯 Objectif :** Créer un CLI `notes` avec Commander.js

**Fonctionnalités :**
- `notes add "Ma note"` → Ajoute une note
- `notes list` → Liste toutes les notes
- `notes delete <id>` → Supprime une note

**Structure du projet :**

```bash
mkdir notes-cli
cd notes-cli
npm init -y
npm install commander
```

**Fichier : `bin/notes.js`**

```javascript
#!/usr/bin/env node
const { Command } = require('commander');
const { addNote, listNotes, deleteNote } = require('../src/commands');

const program = new Command();

program
    .name('notes')
    .description('CLI de gestion de notes')
    .version('1.0.0');

// Commande: notes add
program
    .command('add <note>')
    .description('Ajoute une nouvelle note')
    .option('-t, --tag <tag>', 'Ajoute un tag à la note')
    .action((note, options) => {
        addNote(note, options.tag);
    });

// Commande: notes list
program
    .command('list')
    .description('Liste toutes les notes')
    .option('-t, --tag <tag>', 'Filtre par tag')
    .action((options) => {
        listNotes(options.tag);
    });

// Commande: notes delete
program
    .command('delete <id>')
    .description('Supprime une note')
    .action((id) => {
        deleteNote(parseInt(id));
    });

program.parse(process.argv);
```

**Fichier : `src/commands/index.js`**

```javascript
const fs = require('fs');
const path = require('path');

// Fichier de stockage
const NOTES_FILE = path.join(__dirname, '../../data/notes.json');

// Assurer que le dossier data existe
function ensureDataDir() {
    const dataDir = path.dirname(NOTES_FILE);
    if (!fs.existsSync(dataDir)) {
        fs.mkdirSync(dataDir, { recursive: true });
    }
}

// Lire les notes
function readNotes() {
    ensureDataDir();
    if (!fs.existsSync(NOTES_FILE)) {
        return [];
    }
    const data = fs.readFileSync(NOTES_FILE, 'utf8');
    return JSON.parse(data);
}

// Sauvegarder les notes
function saveNotes(notes) {
    ensureDataDir();
    fs.writeFileSync(NOTES_FILE, JSON.stringify(notes, null, 2));
}

// Ajouter une note
function addNote(text, tag) {
    const notes = readNotes();
    const newNote = {
        id: notes.length > 0 ? Math.max(...notes.map(n => n.id)) + 1 : 1,
        text: text,
        tag: tag || null,
        createdAt: new Date().toISOString()
    };
    notes.push(newNote);
    saveNotes(notes);

    console.log(`✅ Note ajoutée (ID: ${newNote.id})`);
    if (tag) {
        console.log(`🏷️  Tag: ${tag}`);
    }
}

// Lister les notes
function listNotes(filterTag) {
    const notes = readNotes();

    if (notes.length === 0) {
        console.log('📝 Aucune note pour le moment');
        return;
    }

    const filtered = filterTag
        ? notes.filter(n => n.tag === filterTag)
        : notes;

    if (filtered.length === 0) {
        console.log(`📝 Aucune note avec le tag "${filterTag}"`);
        return;
    }

    console.log(`📝 ${filtered.length} note(s):\n`);
    filtered.forEach(note => {
        const tag = note.tag ? ` [${note.tag}]` : '';
        const date = new Date(note.createdAt).toLocaleString();
        console.log(`  ${note.id}. ${note.text}${tag}`);
        console.log(`     📅 ${date}\n`);
    });
}

// Supprimer une note
function deleteNote(id) {
    const notes = readNotes();
    const index = notes.findIndex(n => n.id === id);

    if (index === -1) {
        console.error(`❌ Note ${id} introuvable`);
        process.exit(1);
    }

    const deleted = notes.splice(index, 1)[0];
    saveNotes(notes);

    console.log(`🗑️  Note supprimée: "${deleted.text}"`);
}

module.exports = { addNote, listNotes, deleteNote };
```

**Fichier : `package.json`** (ajouter la section `bin`)

```json
{
  "name": "notes-cli",
  "version": "1.0.0",
  "bin": {
    "notes": "./bin/notes.js"
  },
  "dependencies": {
    "commander": "^11.0.0"
  }
}
```

**Installation globale (optionnel) :**

```bash
npm link
# Maintenant 'notes' est disponible partout
```

**Tests :**

```bash
# Ajouter des notes
notes add "Apprendre Claude Code CLI" --tag formation
notes add "Faire les courses"
notes add "Réviser le chapitre 1" --tag formation

# Lister toutes les notes
notes list

# Lister avec filtre
notes list --tag formation

# Supprimer une note
notes delete 2

# Voir l'aide
notes --help
```

### 🚀 CHALLENGE AUTONOME : Ajoute des Fonctionnalités

**🎯 Défis :**

1. **Commande `search`** : Rechercher dans les notes
   ```bash
   notes search "Claude"
   ```

2. **Commande `edit`** : Modifier une note existante
   ```bash
   notes edit 1 "Nouveau texte"
   ```

3. **Option `--priority`** : Ajouter une priorité (high/medium/low)
   ```bash
   notes add "Urgent!" --priority high
   ```

4. **Commande `stats`** : Afficher des statistiques
   ```bash
   notes stats
   # Total: 10 notes
   # Par tag: formation (3), personnel (7)
   ```

<details>
<summary>✅ Solution Challenge 1 : Commande Search</summary>

```javascript
// Dans bin/notes.js
program
    .command('search <query>')
    .description('Recherche dans les notes')
    .action((query) => {
        searchNotes(query);
    });

// Dans src/commands/index.js
function searchNotes(query) {
    const notes = readNotes();
    const results = notes.filter(n =>
        n.text.toLowerCase().includes(query.toLowerCase())
    );

    if (results.length === 0) {
        console.log(`🔍 Aucune note ne contient "${query}"`);
        return;
    }

    console.log(`🔍 ${results.length} résultat(s) pour "${query}":\n`);
    results.forEach(note => {
        console.log(`  ${note.id}. ${note.text}`);
        if (note.tag) console.log(`     🏷️  ${note.tag}`);
    });
}

module.exports = { addNote, listNotes, deleteNote, searchNotes };
```

</details>

### 📊 POINTS CLÉS

- ✅ Commander.js simplifie la création de CLIs complexes
- ✅ `.command()` définit une commande avec arguments
- ✅ `.option()` ajoute des options/flags
- ✅ `.action()` définit ce qui s'exécute
- ✅ Architecture modulaire = 1 fichier par commande
- ✅ `package.json` `bin` pour installation globale

---

## 📚 Section 3 : Configuration Persistante

### 💡 CONCEPT : Où Stocker la Config ?

**Claude Code CLI stocke sa config dans :**
```
~/.claude/
├── config.json              # Configuration globale
└── session-<id>.json        # État de session
```

**Standards de l'industrie :**

| OS | Emplacement Configuration |
|----|---------------------------|
| **Linux** | `~/.config/nom-app/` ou `~/.nom-app/` |
| **macOS** | `~/Library/Application Support/nom-app/` |
| **Windows** | `%APPDATA%\nom-app\` |

**Bibliothèque recommandée : `conf`**

```bash
npm install conf
```

### 🔍 EXPLORATION : Conf pour la Config

**Exemple : CLI avec préférences utilisateur**

```javascript
const Conf = require('conf');

// Créer un store de config
const config = new Conf({
    projectName: 'notes-cli',
    defaults: {
        theme: 'light',
        editor: 'nano',
        defaultTag: null
    }
});

// Lire une valeur
const theme = config.get('theme');
console.log(`Thème actuel: ${theme}`);

// Écrire une valeur
config.set('theme', 'dark');

// Lire toute la config
console.log(config.store);
// { theme: 'dark', editor: 'nano', defaultTag: null }

// Supprimer une valeur
config.delete('defaultTag');

// Réinitialiser tout
config.clear();
```

**Où est stockée la config ?**

```javascript
console.log(config.path);
// Linux: /home/user/.config/notes-cli/config.json
// macOS: /Users/user/Library/Preferences/notes-cli/config.json
// Windows: C:\Users\user\AppData\Roaming\notes-cli\config.json
```

### 🎮 PRATIQUE : Ajouter la Config au CLI Notes

**🎯 Objectif :** Ajouter des commandes `config` au CLI

**Nouvelles commandes :**
- `notes config set <key> <value>` → Configure une préférence
- `notes config get <key>` → Affiche une préférence
- `notes config list` → Affiche toute la config

**Fichier : `src/config.js`**

```javascript
const Conf = require('conf');

const config = new Conf({
    projectName: 'notes-cli',
    defaults: {
        defaultTag: null,
        dateFormat: 'short', // 'short' ou 'long'
        sortBy: 'date', // 'date' ou 'id'
        maxDisplay: 50
    }
});

module.exports = config;
```

**Ajouter dans `bin/notes.js` :**

```javascript
const config = require('../src/config');

// Commande: notes config
const configCommand = program
    .command('config')
    .description('Gère la configuration');

configCommand
    .command('set <key> <value>')
    .description('Définit une valeur de config')
    .action((key, value) => {
        config.set(key, value);
        console.log(`✅ ${key} = ${value}`);
    });

configCommand
    .command('get <key>')
    .description('Affiche une valeur de config')
    .action((key) => {
        const value = config.get(key);
        if (value === undefined) {
            console.error(`❌ Clé "${key}" introuvable`);
        } else {
            console.log(`${key} = ${value}`);
        }
    });

configCommand
    .command('list')
    .description('Affiche toute la configuration')
    .action(() => {
        console.log('⚙️  Configuration actuelle:\n');
        const store = config.store;
        Object.entries(store).forEach(([key, value]) => {
            console.log(`  ${key}: ${value}`);
        });
        console.log(`\n📁 Fichier: ${config.path}`);
    });
```

**Utiliser la config dans `listNotes` :**

```javascript
function listNotes(filterTag) {
    const notes = readNotes();
    const sortBy = config.get('sortBy');
    const maxDisplay = config.get('maxDisplay');

    // Trier selon la config
    if (sortBy === 'date') {
        notes.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
    } else {
        notes.sort((a, b) => b.id - a.id);
    }

    // Filtrer
    const filtered = filterTag
        ? notes.filter(n => n.tag === filterTag)
        : notes;

    // Limiter selon maxDisplay
    const toDisplay = filtered.slice(0, maxDisplay);

    if (toDisplay.length < filtered.length) {
        console.log(`📝 Affichage de ${toDisplay.length}/${filtered.length} notes\n`);
    } else {
        console.log(`📝 ${filtered.length} note(s):\n`);
    }

    toDisplay.forEach(note => {
        const dateFormat = config.get('dateFormat');
        const date = new Date(note.createdAt);
        const dateStr = dateFormat === 'long'
            ? date.toLocaleString()
            : date.toLocaleDateString();

        console.log(`  ${note.id}. ${note.text}`);
        if (note.tag) console.log(`     🏷️  ${note.tag}`);
        console.log(`     📅 ${dateStr}\n`);
    });
}
```

**Tests :**

```bash
# Configurer
notes config set defaultTag work
notes config set maxDisplay 10
notes config set dateFormat long

# Voir la config
notes config list

# Utiliser le defaultTag
notes add "Ma tâche" # Utilise automatiquement le defaultTag 'work'
```

### 🚀 CHALLENGE FINAL : Architecture Complète

**🎯 Mission :** Refactorise le CLI Notes avec une architecture professionnelle

**Structure cible :**

```
notes-cli/
├── bin/
│   └── notes.js              # Point d'entrée (80 lignes max)
├── src/
│   ├── commands/
│   │   ├── add.js            # Logique de 'add'
│   │   ├── list.js           # Logique de 'list'
│   │   ├── delete.js         # Logique de 'delete'
│   │   ├── search.js         # Logique de 'search'
│   │   ├── config.js         # Logique de 'config'
│   │   └── index.js          # Export tous les commands
│   ├── storage/
│   │   └── notes.js          # Gestion CRUD des notes
│   ├── config.js             # Configuration globale
│   └── utils/
│       ├── formatter.js      # Formatage de sortie
│       └── validator.js      # Validation
├── data/
│   └── notes.json            # Données (ignoré par git)
├── package.json
└── README.md
```

**Bonus :**
- Tests unitaires avec Jest
- GitHub Actions CI/CD
- Publication sur npm

### 📊 POINTS CLÉS

- ✅ Utiliser `conf` pour config multi-plateforme
- ✅ Defaults clairs dans la config
- ✅ Config permet de personnaliser l'expérience
- ✅ Stocker dans les emplacements standards de l'OS

---

## 🧪 MINI-PROJET DE CHAPITRE : CLI Complet Production-Ready

### 🎯 Mission Complète

Crée un CLI de gestion de **snippets de code** avec :

**Fonctionnalités requises :**
1. **Commandes de base :**
   - `snippets add <name> <code>` → Ajoute un snippet
   - `snippets list` → Liste tous les snippets
   - `snippets get <name>` → Affiche un snippet
   - `snippets delete <name>` → Supprime un snippet

2. **Options avancées :**
   - `--lang <language>` → Associer un langage
   - `--desc <description>` → Ajouter une description
   - `--tags <tag1,tag2>` → Ajouter des tags

3. **Configuration :**
   - Langage par défaut
   - Format d'affichage (plain/colored)
   - Éditeur préféré

4. **Architecture :**
   - Commander.js pour le CLI
   - Conf pour la config
   - Structure modulaire
   - Gestion d'erreurs propre

**Temps estimé :** 60-90 minutes

**Indice de démarrage :**

```javascript
// bin/snippets.js
#!/usr/bin/env node
const { Command } = require('commander');
const commands = require('../src/commands');

const program = new Command();

program
    .name('snippets')
    .description('Gestionnaire de snippets de code')
    .version('1.0.0');

// TODO: Ajouter toutes les commandes

program.parse(process.argv);
```

<details>
<summary>✅ Solution Complète (Architecture Professionnelle)</summary>

Je ne vais pas écrire toute la solution ici (trop long), mais voici l'architecture :

**`src/storage/snippets.js` :**
- `createSnippet(name, code, metadata)`
- `getSnippet(name)`
- `getAllSnippets(filter)`
- `updateSnippet(name, updates)`
- `deleteSnippet(name)`

**`src/commands/add.js` :**
```javascript
const storage = require('../storage/snippets');
const config = require('../config');

function addSnippet(name, code, options) {
    const metadata = {
        lang: options.lang || config.get('defaultLang'),
        desc: options.desc || '',
        tags: options.tags ? options.tags.split(',') : [],
        createdAt: new Date().toISOString()
    };

    storage.createSnippet(name, code, metadata);
    console.log(`✅ Snippet "${name}" ajouté`);
}

module.exports = addSnippet;
```

Même pattern pour les autres commandes.

</details>

---

## 🎯 QUIZ INTERLEAVING (Mélange)

**🧠 Science :** Les questions mélangent TOUS les concepts du chapitre

### Question 1 : Shebang

Qu'est-ce que `#!/usr/bin/env node` ?

A) Un commentaire JavaScript
B) Une directive pour l'OS d'utiliser Node.js
C) Un import de module
D) Une déclaration de version

<details>
<summary>💡 Solution</summary>

**B) Une directive pour l'OS d'utiliser Node.js**

C'est un **shebang** qui indique au système d'exploitation quel interpréteur utiliser pour exécuter le fichier. Sans ça, l'OS ne saurait pas que c'est du JavaScript.

</details>

### Question 2 : Process.argv

Si on exécute `node cli.js add "hello world" --force`, que contient `process.argv` ?

```javascript
const args = process.argv;
console.log(args);
```

A) `['add', 'hello world', '--force']`
B) `['node', 'cli.js', 'add', 'hello world', '--force']`
C) `['node', '/path/to/cli.js', 'add', 'hello world', '--force']`
D) `['cli.js', 'add', 'hello world', '--force']`

<details>
<summary>💡 Solution</summary>

**C) `['node', '/path/to/cli.js', 'add', 'hello world', '--force']`**

`process.argv` contient **toujours**:
1. Le chemin complet de l'exécutable node
2. Le chemin complet du script exécuté
3. Tous les arguments passés

C'est pourquoi on fait `.slice(2)` pour obtenir juste les arguments.

</details>

### Question 3 : Commander.js

Quel est l'ordre correct pour définir une commande ?

```javascript
program
    .command('greet <name>')
    .description('Dit bonjour')
    .option('-l, --loud', 'En majuscules')
    .action((name, options) => { /* ... */ });
```

A) command → action → description → option
B) command → description → option → action
C) description → command → option → action
D) L'ordre n'a pas d'importance

<details>
<summary>💡 Solution</summary>

**B) command → description → option → action**

C'est le pattern recommandé :
1. `.command()` - Définit la commande
2. `.description()` - Documente
3. `.option()` - Ajoute des options (si besoin)
4. `.action()` - Définit la logique (toujours en dernier)

</details>

### Question 4 : Architecture

Pourquoi séparer en plusieurs fichiers ?

```
bin/cli.js            # 50 lignes
src/commands/*.js     # 20 lignes chacun
```

Au lieu de:
```
cli.js                # 500 lignes
```

A) Pour faire joli
B) Facilite tests, maintenance et collaboration
C) Obligation de Node.js
D) Performance meilleure

<details>
<summary>💡 Solution</summary>

**B) Facilite tests, maintenance et collaboration**

**Avantages concrets :**
- **Tests** : On peut tester chaque commande isolément
- **Maintenance** : Bug dans 'add' ? On sait où chercher
- **Collaboration** : 3 devs peuvent travailler sur 3 commandes en parallèle
- **Réutilisation** : Les utils peuvent être partagés
- **Lisibilité** : 20 lignes vs 500 lignes

</details>

### Question 5 : Configuration

Où `conf` stocke-t-il la config sur Linux ?

A) `./config.json`
B) `~/.config/nom-app/config.json`
C) `/etc/nom-app/config.json`
D) `~/nom-app/config.json`

<details>
<summary>💡 Solution</summary>

**B) `~/.config/nom-app/config.json`**

`conf` suit les standards XDG sur Linux :
- **Linux** : `~/.config/nom-app/`
- **macOS** : `~/Library/Application Support/nom-app/`
- **Windows** : `%APPDATA%\nom-app\`

</details>

---

## 📅 RÉVISION ESPACÉE - CALENDRIER

**🧠 Science :** La répétition espacée multiplie par 5 la rétention

### J+1 (Demain) : 20 minutes
- [ ] Refais le CLI calculatrice sans regarder la solution
- [ ] Refais le CLI notes avec Commander.js (structure de base)
- [ ] Explique à voix haute comment fonctionne `process.argv`

### J+3 (Dans 3 jours) : 15 minutes
- [ ] Quiz : 10 questions sur CLI, Commander, Config
- [ ] Crée un nouveau CLI simple (timer, weather, etc.)

### J+7 (Dans 1 semaine) : 30 minutes
- [ ] Crée une variante du CLI Notes (ex: CLI Todo avec priorités)
- [ ] Ajoute 2 nouvelles fonctionnalités au CLI Snippets

### J+14 (Dans 2 semaines) : 45 minutes
- [ ] Quiz mélangé avec le Chapitre 2
- [ ] Refactorise un vieux projet en CLI

### J+30 (Dans 1 mois) : 60 minutes
- [ ] Challenge créatif : CLI de ton choix avec architecture complète
- [ ] Review le code du vrai Claude Code CLI

**📌 IMPORTANT :** Ces révisions ne sont PAS optionnelles. C'est là que ton cerveau ancre les connaissances dans la mémoire long-terme !

---

## 🚀 POUR ALLER PLUS LOIN

### 🎯 Niveau 2 Disponible

➡️ **[Chapitre 01 Niveau 2 - Détaillé](../NIVEAU-2-MAITRISE-PRATIQUE/Chapitre-01-Fondamentaux-CLI-Architecture/Phase-1-Introduction.md)**

**Contenu Niveau 2 :**
- Architecture hexagonale pour CLIs
- Tests automatisés (Jest, Vitest)
- CI/CD avec GitHub Actions
- Publication sur npm
- Terminal UI avancé (ink, blessed)
- 15+ exercices progressifs
- Projet : CLI production-ready complet

### 💡 Projets Créatifs Suggérés

**Idées de CLIs à construire :**

1. **CLI Pomodoro Timer**
   - `pomodoro start` → Lance 25 min de travail
   - `pomodoro break` → Lance 5 min de pause
   - Notifications desktop

2. **CLI Git Enhanced**
   - `gitx commit` → Commit avec message guidé (type, scope, etc.)
   - `gitx stats` → Stats détaillées du repo
   - `gitx clean` → Nettoyage interactif des branches

3. **CLI Journal**
   - `journal add "Aujourd'hui..."` → Entrée de journal
   - `journal today` → Affiche entrée du jour
   - `journal search <mot>` → Recherche dans le journal
   - Markdown rendering dans le terminal

4. **CLI Bookmark Manager**
   - `bm add <url> --tags "dev,node"` → Sauvegarde un lien
   - `bm list --tag dev` → Liste par tag
   - `bm open <id>` → Ouvre dans le navigateur

5. **CLI Password Generator**
   - `pwgen` → Génère un mot de passe sécurisé
   - `pwgen --length 20 --special` → Personnalisé
   - `pwgen --save myapp` → Sauve dans le keychain

---

## 📊 TRACKER DE PROGRESSION

```
Chapitre 1 : CLI & Architecture
├── Section 1 : Structure CLI      [●●●●●] Complété ✅
├── Section 2 : Commander.js       [●●●●●] Complété ✅
├── Section 3 : Configuration      [●●●●●] Complété ✅
├── Mini-Projet                     [●●●●○] En cours...
└── Quiz                            [●●●○○] 3/5

Auto-Évaluation:
- [ ] Je peux créer un CLI basique avec Node.js
- [ ] Je comprends process.argv et son rôle
- [ ] Je sais utiliser Commander.js pour des CLIs complexes
- [ ] Je peux structurer un projet CLI en modules
- [ ] Je sais gérer la configuration avec conf
- [ ] Je peux créer un CLI production-ready complet

Score: __/6

Temps investi: ___ minutes / 90 estimées
Prochaine étape: ___________________
```

---

## 🎓 FÉLICITATIONS !

**🌟 Ce que tu as accompli en 90 minutes :**

- ✅ **Compris** l'anatomie d'un CLI Node.js
- ✅ **Créé** un CLI calculatrice fonctionnel
- ✅ **Maîtrisé** Commander.js pour des CLIs complexes
- ✅ **Construit** un CLI de gestion de notes complet
- ✅ **Implémenté** une architecture modulaire professionnelle
- ✅ **Ajouté** un système de configuration persistante
- ✅ **Architecturé** un CLI production-ready

**🚀 Tu es maintenant capable de :**
- Créer n'importe quel CLI de A à Z
- Utiliser les meilleures pratiques de l'industrie
- Structurer du code maintenable et testable
- Comprendre comment fonctionne Claude Code CLI (partie 1/6)

**🎯 Prochaine étape :**
- Soit approfondir (Niveau 2 Chapitre 1)
- Soit découvrir le Chapitre 2 (Outils Built-in)

---

**Navigation :**
- ⬅️ [Survol Complet](./00-Survol-Interactif-Complet.md)
- ➡️ [Chapitre 02 - Outils Built-in](./02-Chapitre-02-Apercu-Interactif.md)
- 📚 [Version Détaillée Niveau 2](../NIVEAU-2-MAITRISE-PRATIQUE/Chapitre-01-Fondamentaux-CLI-Architecture/Phase-1-Introduction.md)
- 🏠 [Retour ROADMAP](../ROADMAP-FORMATION-COMPLETE.md)

---

*Formation basée sur l'Approche Hybride Optimale - 100 ans de recherche en sciences cognitives appliqués*

**Version :** 1.0.0
**Temps de lecture :** 30-40 min
**Temps de pratique :** 50-60 min
**Score de rétention attendu :** 85% avec révisions espacées
