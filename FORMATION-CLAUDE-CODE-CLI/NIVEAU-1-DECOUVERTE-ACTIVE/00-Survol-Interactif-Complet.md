# 🎬 NIVEAU 1 : SURVOL INTERACTIF COMPLET

> **🎯 Objectif :** Comprendre ET pratiquer l'ENSEMBLE de l'architecture Claude Code CLI en 90 minutes
> **🧠 Science :** Active Learning + Immediate Feedback + Low Cognitive Load
> **📊 Progression :** [■□□□□□□□□□] 10% du parcours total
> **⏱️ Durée :** 90 minutes

---

## 🎮 ACTIVATION : Bienvenue !

### 🤔 Question Réflexive (Metacognition)

> Tu vas apprendre à construire un assistant AI qui code pour toi, comme Claude Code CLI.
>
> **Réfléchis 2 minutes :**
> - Si tu pouvais automatiser UNE tâche de développement, laquelle ?
> - Comment voudrais-tu communiquer avec cet assistant ? (chat, commandes, fichiers...)
> - Quels "pouvoirs" devrait-il avoir ? (lire code, modifier fichiers, exécuter commandes...)

**💭 Note tes réflexions - on y reviendra à la fin !**

---

## 📚 VUE D'ENSEMBLE : Les 6 Chapitres

Cette formation est organisée en 6 chapitres progressifs qui s'assemblent comme des Legos :

```
Chapitre 1: CLI & Architecture
    ↓ (Base : Commander.js + Configuration)
Chapitre 2: Outils Built-in
    ↓ (Read, Write, Edit, Grep, Glob, Bash)
Chapitre 3: Claude API & Conversation
    ↓ (Communication avec l'IA)
Chapitre 4: Plugins & Slash Commands
    ↓ (Extensions personnalisées)
Chapitre 5: Hooks & Multi-Agents
    ↓ (Automatisation + Orchestration)
Chapitre 6: Projet Final
    ↓ (Tout assembler !)

    = Claude Code CLI Complet 🎉
```

**Dans ce survol, tu vas voir ET pratiquer chaque chapitre en version condensée !**

---

## 🏗️ CHAPITRE 1 : CLI & Architecture (Aperçu 15 min)

### 💡 Le Concept

Un CLI (Command Line Interface) est un programme qui s'exécute dans le terminal avec des commandes.

**Exemple :** `git commit -m "message"` est un CLI !

### 🎮 CODE RAPIDE : Ton Premier CLI

**`hello-cli.js` :**

```javascript
#!/usr/bin/env node

// Simple CLI qui salue
const args = process.argv.slice(2);
const name = args[0] || 'World';

console.log(`👋 Hello, ${name}!`);
```

**Test :**
```bash
node hello-cli.js Alice
# 👋 Hello, Alice!
```

### 🚀 Avec Commander.js (Framework Professionnel)

```javascript
#!/usr/bin/env node
const { Command } = require('commander');

const program = new Command();

program
  .name('my-cli')
  .description('Mon premier CLI professionnel')
  .version('1.0.0');

program
  .command('greet <name>')
  .description('Saluer quelqu\'un')
  .option('-l, --loud', 'En majuscules')
  .action((name, options) => {
    const greeting = `👋 Hello, ${name}!`;
    console.log(options.loud ? greeting.toUpperCase() : greeting);
  });

program.parse();
```

**Test :**
```bash
node my-cli.js greet Alice
# 👋 Hello, Alice!

node my-cli.js greet Alice --loud
# 👋 HELLO, ALICE!
```

### ✅ Ce que tu retiens

- ✅ Un CLI = programme exécutable dans le terminal
- ✅ `process.argv` pour lire les arguments
- ✅ Commander.js pour des CLIs professionnels
- ✅ Commandes, options, flags

**➡️ Chapitre complet : 90 minutes avec architecture modulaire et configuration**

---

## 🛠️ CHAPITRE 2 : Outils Built-in (Aperçu 15 min)

### 💡 Le Concept

Pour que Claude puisse t'aider, il a besoin "d'outils" : lire fichiers, écrire code, chercher patterns...

### 🎮 CODE RAPIDE : Les 6 Outils Essentiels

**1. Read - Lire un fichier :**

```javascript
const fs = require('fs');

function read(filePath) {
  const content = fs.readFileSync(filePath, 'utf-8');
  return content;
}

console.log(read('package.json'));
```

**2. Write - Écrire un fichier :**

```javascript
function write(filePath, content) {
  fs.writeFileSync(filePath, content, 'utf-8');
  console.log(`✅ Written to ${filePath}`);
}

write('hello.txt', 'Hello, World!');
```

**3. Grep - Chercher du texte :**

```javascript
const { execSync } = require('child_process');

function grep(pattern, path = '.') {
  const cmd = `grep -r "${pattern}" ${path} --include="*.js"`;
  const output = execSync(cmd, { encoding: 'utf-8' });
  return output;
}

console.log(grep('TODO'));
// Trouve tous les TODOs dans le code
```

**4. Glob - Trouver des fichiers :**

```javascript
const glob = require('glob');

function findFiles(pattern) {
  return glob.sync(pattern);
}

const jsFiles = findFiles('**/*.js');
console.log(`Found ${jsFiles.length} JS files`);
```

**5. Bash - Exécuter des commandes :**

```javascript
function bash(command) {
  const output = execSync(command, { encoding: 'utf-8' });
  return output;
}

console.log(bash('git status --short'));
```

**6. Edit - Modifier un fichier :**

```javascript
function edit(filePath, oldText, newText) {
  let content = fs.readFileSync(filePath, 'utf-8');
  content = content.replace(oldText, newText);
  fs.writeFileSync(filePath, content);
  console.log(`✅ Edited ${filePath}`);
}

edit('config.js', 'port: 3000', 'port: 8080');
```

### ✅ Ce que tu retiens

- ✅ **Read/Write** : Lire et créer des fichiers
- ✅ **Edit** : Modifier du code
- ✅ **Grep/Glob** : Chercher dans le code
- ✅ **Bash** : Exécuter des commandes

**➡️ Chapitre complet : 90 minutes avec implémentations sécurisées**

---

## 🤖 CHAPITRE 3 : Claude API & Conversation (Aperçu 20 min)

### 💡 Le Concept

Connecter ton CLI à l'IA Claude pour qu'il puisse comprendre tes demandes et utiliser les outils.

### 🎮 CODE RAPIDE : Premier Appel à Claude

```javascript
const Anthropic = require('@anthropic-ai/sdk');

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function askClaude(question) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 1024,
    messages: [
      { role: 'user', content: question }
    ]
  });

  return message.content[0].text;
}

// Test
askClaude('Explique-moi ce qu\'est un CLI en une phrase').then(response => {
  console.log('Claude:', response);
});
```

### 🚀 Avec Tool Use (Claude Utilise Tes Outils)

```javascript
async function claudeWithTools(userMessage) {
  const tools = [
    {
      name: 'read_file',
      description: 'Read the contents of a file',
      input_schema: {
        type: 'object',
        properties: {
          file_path: { type: 'string', description: 'Path to the file' }
        },
        required: ['file_path']
      }
    }
  ];

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 1024,
    tools: tools,
    messages: [
      { role: 'user', content: userMessage }
    ]
  });

  // Si Claude veut utiliser un outil
  if (message.stop_reason === 'tool_use') {
    const toolUse = message.content.find(block => block.type === 'tool_use');

    if (toolUse.name === 'read_file') {
      const filePath = toolUse.input.file_path;
      const content = fs.readFileSync(filePath, 'utf-8');

      console.log(`🔧 Claude used tool: read_file(${filePath})`);
      console.log('Content:', content.substring(0, 100) + '...');
    }
  }

  return message.content[0].text;
}

// Test
claudeWithTools('Read package.json and tell me the version');
```

### ✅ Ce que tu retiens

- ✅ Anthropic SDK pour communiquer avec Claude
- ✅ Messages API (user/assistant)
- ✅ **Tool use** : Claude peut utiliser tes outils !
- ✅ Conversation multi-tour possible

**➡️ Chapitre complet : 120 minutes avec conversation loop**

---

## 🧩 CHAPITRE 4 : Plugins & Slash Commands (Aperçu 15 min)

### 💡 Le Concept

Créer des extensions (plugins) avec des "slash commands" comme `/commit`, `/review`...

### 🎮 CODE RAPIDE : Structure d'un Plugin

```
plugins/
└── my-plugin/
    ├── .claude-plugin/
    │   └── plugin.json      # Configuration
    └── commands/
        └── hello.md         # Slash command
```

**`plugin.json` :**

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Mon premier plugin",
  "commands": ["commands"]
}
```

**`commands/hello.md` :**

```markdown
---
description: Say hello
---

# /hello

Say a friendly hello to the user. Ask their name and what they're working on.
```

### 🚀 Avec Injection de Contexte

**`commands/analyze.md` :**

```markdown
---
description: Analyze current directory
---

# /analyze

Analyze the project structure:

## Files:
!bash{find . -type f -name "*.js" | head -10}

## Git Status:
!bash{git status --short}

## Package Info:
!file{package.json}

Provide insights about the project structure and status.
```

**Ce qui se passe :**
1. Tu tapes `my-cli /analyze`
2. Les `!bash{...}` et `!file{...}` sont remplacés par les vraies valeurs
3. Le prompt complet est envoyé à Claude
4. Claude analyse et répond !

### ✅ Ce que tu retiens

- ✅ Plugins = dossiers avec `plugin.json`
- ✅ Slash commands = fichiers Markdown
- ✅ Injection de contexte : `!bash`, `!file`, `!grep`, `!glob`
- ✅ Extensible à l'infini !

**➡️ Chapitre complet : 120 minutes avec plugin marketplace**

---

## 🪝 CHAPITRE 5 : Hooks & Multi-Agents (Aperçu 15 min)

### 💡 Le Concept

**Hooks :** Scripts qui s'exécutent automatiquement (comme dans Git)
**Multi-Agents :** Orchestrer plusieurs experts IA en parallèle

### 🎮 CODE RAPIDE : Hook de Sécurité

**`hooks/validate-bash.py` :**

```python
#!/usr/bin/env python3
import sys, json

input_data = json.loads(sys.stdin.read())
command = input_data['command']

# Bloquer les commandes dangereuses
if 'rm -rf /' in command:
    print(json.dumps({
        "status": "blocked",
        "message": "❌ DANGEROUS COMMAND BLOCKED"
    }))
    sys.exit(1)

print(json.dumps({"status": "approved"}))
```

### 🚀 Multi-Agents en Parallèle

```javascript
async function codeReview(filePath) {
  // 3 experts en parallèle !
  const results = await Promise.all([
    askAgent('code-quality-agent', filePath),
    askAgent('security-agent', filePath),
    askAgent('performance-agent', filePath)
  ]);

  console.log('Code Quality:', results[0]);
  console.log('Security:', results[1]);
  console.log('Performance:', results[2]);
}

async function askAgent(agentName, context) {
  // Chaque agent a son propre prompt spécialisé
  const agentPrompt = loadAgent(agentName);
  return await askClaude(agentPrompt + '\n\nFile: ' + context);
}
```

### ✅ Ce que tu retiens

- ✅ **Hooks** : Automatisation pre/post actions
- ✅ **Multi-agents** : Plusieurs experts collaboratifs
- ✅ **Promise.all** : Parallélisation
- ✅ Orchestration intelligente

**➡️ Chapitre complet : 120 minutes avec CI/CD multi-agents**

---

## 🎯 CHAPITRE 6 : Projet Final (Aperçu 10 min)

### 💡 Le Concept

Assembler TOUS les composants en un CLI complet et fonctionnel !

### 🎮 ARCHITECTURE FINALE

```javascript
#!/usr/bin/env node
const { Command } = require('commander');
const Anthropic = require('@anthropic-ai/sdk');

class ClaudeCodeCLI {
  constructor() {
    this.claude = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });
    this.tools = this.registerTools();
    this.plugins = this.loadPlugins();
    this.hooks = this.loadHooks();
  }

  registerTools() {
    return {
      read: (path) => fs.readFileSync(path, 'utf-8'),
      write: (path, content) => fs.writeFileSync(path, content),
      bash: (cmd) => execSync(cmd, { encoding: 'utf-8' })
    };
  }

  loadPlugins() {
    // Charger tous les plugins depuis ./plugins/
    // ...
  }

  async executeCommand(commandName) {
    // 1. Trouver le slash command
    const command = this.findCommand(commandName);

    // 2. Injecter le contexte (!bash, !file, etc.)
    const prompt = await this.injectContext(command.content);

    // 3. Envoyer à Claude avec tools
    const response = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      tools: this.getToolDefinitions(),
      messages: [{ role: 'user', content: prompt }]
    });

    // 4. Gérer les tool uses
    if (response.stop_reason === 'tool_use') {
      await this.handleToolUse(response);
    }

    return response;
  }
}

// CLI avec Commander.js
const program = new Command();
const cli = new ClaudeCodeCLI();

program
  .command('chat <message>')
  .action(async (message) => {
    const response = await cli.chat(message);
    console.log('Claude:', response);
  });

program
  .command('run <command>')
  .action(async (cmd) => {
    await cli.executeCommand(cmd);
  });

program.parse();
```

### ✅ Ce que tu auras construit

Un CLI complet avec :
- ✅ Interface Commander.js
- ✅ 6 outils built-in
- ✅ Intégration Claude avec tools
- ✅ Système de plugins
- ✅ Slash commands personnalisés
- ✅ Hooks de sécurité
- ✅ Multi-agents si besoin

**➡️ Chapitre complet : 90 minutes avec guide step-by-step**

---

## 🧪 MINI-PROJET DU SURVOL : CLI Simple Mais Complet

### 🎯 Mission

Construis un CLI minimal qui combine les 6 chapitres en 30 minutes !

### 📝 Code Complet

**`simple-claude-cli.js` :**

```javascript
#!/usr/bin/env node
const { Command } = require('commander');
const Anthropic = require('@anthropic-ai/sdk');
const fs = require('fs');
const { execSync } = require('child_process');

// Initialiser Claude
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY || 'your-key-here'
});

// Définir les outils
const tools = [
  {
    name: 'read_file',
    description: 'Read a file',
    input_schema: {
      type: 'object',
      properties: {
        path: { type: 'string' }
      },
      required: ['path']
    }
  },
  {
    name: 'bash',
    description: 'Execute a bash command',
    input_schema: {
      type: 'object',
      properties: {
        command: { type: 'string' }
      },
      required: ['command']
    }
  }
];

// Fonction principale
async function chat(message) {
  console.log('\n💬 You:', message);

  const response = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 2048,
    tools: tools,
    messages: [{ role: 'user', content: message }]
  });

  // Gérer les tool uses
  for (const block of response.content) {
    if (block.type === 'tool_use') {
      console.log(`\n🔧 Tool: ${block.name}`);

      let result;
      if (block.name === 'read_file') {
        result = fs.readFileSync(block.input.path, 'utf-8');
      } else if (block.name === 'bash') {
        result = execSync(block.input.command, { encoding: 'utf-8' });
      }

      console.log('Result:', result.substring(0, 200));
    } else if (block.type === 'text') {
      console.log('\n🤖 Claude:', block.text);
    }
  }
}

// CLI
const program = new Command();

program
  .name('simple-claude-cli')
  .description('Simple Claude Code CLI')
  .version('1.0.0');

program
  .command('chat <message>')
  .description('Chat with Claude')
  .action(chat);

program.parse();
```

### 🎮 Test

```bash
# Configurer l'API key
export ANTHROPIC_API_KEY=your-key-here

# Tester
node simple-claude-cli.js chat "Read package.json and tell me the version"

# Résultat :
# 💬 You: Read package.json and tell me the version
# 🔧 Tool: read_file
# Result: { "name": "my-cli", "version": "1.0.0", ... }
# 🤖 Claude: The version in your package.json is 1.0.0
```

**🎉 Félicitations ! Tu as un CLI AI-powered fonctionnel en 30 lignes !**

---

## 🎯 QUIZ INTERLEAVING (Tous Chapitres)

### Question 1 : Architecture Globale

Dans quel ordre ces composants s'exécutent-ils ?

a) Claude → Tools → Plugins → Hooks
b) Hooks → Plugins → Claude → Tools
c) Plugins → Claude → Hooks → Tools

<details>
<summary>💡 Voir la réponse</summary>

**Réponse : b) Hooks → Plugins → Claude → Tools**

**Flux complet :**
1. **Hooks pre-** (validation)
2. **Plugins** (charger le slash command)
3. **Claude** (analyser et décider)
4. **Tools** (exécuter les actions)
5. **Hooks post-** (logging, cleanup)

</details>

### Question 2 : Concepts Clés

Match les concepts :

| Concept | Description |
|---------|-------------|
| 1. Slash command | A. Script Python/JS qui valide |
| 2. Tool use | B. Fichier Markdown avec prompt |
| 3. Hook | C. Claude utilise tes fonctions |
| 4. Agent | D. Expert IA spécialisé |

<details>
<summary>💡 Voir la réponse</summary>

**Réponses :**
- 1-B : Slash command = Markdown avec prompt
- 2-C : Tool use = Claude utilise tes fonctions
- 3-A : Hook = Script Python/JS qui valide
- 4-D : Agent = Expert IA spécialisé

</details>

### Question 3 : Code Challenge

Ce code fait quoi ?

```javascript
const results = await Promise.all([
  askAgent('linter'),
  askAgent('security'),
  askAgent('tester')
]);
```

<details>
<summary>💡 Voir la réponse</summary>

**Réponse :** Il exécute 3 agents **en parallèle** (linter, security, tester) et attend que tous terminent.

**Avantage :** 3x plus rapide que séquentiel !

Si séquentiel (3 x 10s = 30s)
Si parallèle (max(10s, 10s, 10s) = 10s)

</details>

### Question 4 : Débogage

Un utilisateur tape `/commit` mais rien ne se passe. Où chercher ?

<details>
<summary>💡 Voir la réponse</summary>

**Checklist de débogage :**

1. **Plugin chargé ?**
   ```javascript
   console.log(this.plugins); // Voir si my-plugin est présent
   ```

2. **Command trouvée ?**
   ```javascript
   const cmd = this.findCommand('commit');
   console.log(cmd); // Doit retourner le contenu de commit.md
   ```

3. **Injection OK ?**
   ```javascript
   const prompt = await this.injectContext(cmd.content);
   console.log(prompt); // Les !bash{} sont remplacés ?
   ```

4. **API key configurée ?**
   ```bash
   echo $ANTHROPIC_API_KEY
   ```

</details>

### Question 5 : Best Practice

Quel code est meilleur ?

**A :**
```javascript
const content = fs.readFileSync(userInput);
```

**B :**
```javascript
if (userInput.includes('..')) {
  throw new Error('Invalid path');
}
const content = fs.readFileSync(userInput);
```

<details>
<summary>💡 Voir la réponse</summary>

**Réponse : B est meilleur !**

**Pourquoi :**
- A est vulnérable : `userInput = "../../../../etc/passwd"` 😱
- B valide l'entrée (path traversal attack bloqué)

**Encore mieux :**
```javascript
const path = require('path');
const safePath = path.resolve(userInput);
if (!safePath.startsWith(process.cwd())) {
  throw new Error('Access denied');
}
const content = fs.readFileSync(safePath);
```

Toujours valider les inputs utilisateur !

</details>

---

## 📊 AUTO-ÉVALUATION : Es-tu Prêt ?

Coche honnêtement ce que tu as compris :

**Chapitre 1 - CLI & Architecture :**
- [ ] Je comprends ce qu'est un CLI
- [ ] Je sais utiliser Commander.js
- [ ] Je peux créer des commandes avec options

**Chapitre 2 - Outils Built-in :**
- [ ] Je peux lire/écrire des fichiers (Read/Write)
- [ ] Je comprends Grep/Glob pour chercher
- [ ] Je sais exécuter Bash de façon sécurisée

**Chapitre 3 - Claude API :**
- [ ] Je peux appeler l'API Anthropic
- [ ] Je comprends le concept de "tool use"
- [ ] Je sais créer une conversation

**Chapitre 4 - Plugins :**
- [ ] Je comprends la structure d'un plugin
- [ ] Je sais créer un slash command
- [ ] Je peux injecter du contexte (!bash, !file)

**Chapitre 5 - Hooks & Agents :**
- [ ] Je comprends les hooks pre/post
- [ ] Je sais orchestrer des agents
- [ ] Je peux utiliser Promise.all

**Chapitre 6 - Intégration :**
- [ ] Je vois comment tout s'assemble
- [ ] Je peux construire un CLI complet

**Score (/18) :**
- **15-18 coches** : 🎉 Excellent ! Passe aux chapitres détaillés
- **10-14 coches** : 👍 Bien ! Continue avec les chapitres
- **5-9 coches** : 📚 Relis le survol et prends des notes
- **0-4 coches** : 🔄 Recommence doucement, c'est normal !

---

## 📅 CALENDRIER DE RÉVISION

### J+1 (Demain)
- ✅ Relis le survol (30 min)
- ✅ Refais le mini-projet `simple-claude-cli.js`
- ✅ Modifie-le pour ajouter un 3ème outil

### J+3 (Dans 3 jours)
- ✅ Quiz : Refais les 5 questions sans regarder
- ✅ Crée ton propre CLI minimal (1h)

### J+7 (Dans 1 semaine)
- ✅ Lis le Chapitre 1 complet
- ✅ Commence les exercices pratiques

### J+14 (Dans 2 semaines)
- ✅ Tu devrais être au Chapitre 3
- ✅ Quiz combinant Ch1-3

### J+30 (Dans 1 mois)
- ✅ Projet final terminé
- ✅ Ton propre Claude Code CLI fonctionnel !

---

## 🚀 PROCHAINES ÉTAPES

### Option 1 : Plonger dans les Chapitres

➡️ **[Chapitre 01 : CLI & Architecture](./01-Chapitre-01-Apercu-Interactif.md)**

Tu apprendras :
- Architecture modulaire professionnelle
- Commander.js en profondeur
- Configuration persistante avec `conf`
- CLI Notes complet (CRUD)
- 90 minutes de pratique

### Option 2 : Créer Immédiatement

Utilise le code du survol pour créer ton premier CLI :

```bash
mkdir my-first-cli
cd my-first-cli
npm init -y
npm install commander @anthropic-ai/sdk

# Copie le code de simple-claude-cli.js
# Teste !
```

### Option 3 : Explorer le Vrai Claude Code

```bash
git clone https://github.com/anthropics/claude-code
cd claude-code
# Explore la structure avec ce que tu as appris !
```

---

## 💡 POINTS CLÉS À RETENIR

### Les 6 Piliers de Claude Code CLI

1. **CLI Framework** (Commander.js) - Interface utilisateur
2. **Tools Built-in** (Read, Write, etc.) - Actions concrètes
3. **Claude API** (Anthropic SDK) - Intelligence IA
4. **Plugins** (Extensions) - Personnalisation infinie
5. **Hooks** (Automatisation) - Validation et logging
6. **Multi-Agents** (Orchestration) - Expertise collaborative

### L'Équation Magique

```
CLI Framework
  + Tools (Read, Write, Bash, etc.)
  + Claude API (avec tool use)
  + Plugins (slash commands)
  + Hooks (pre/post)
  + Multi-Agents (parallèle)
  = Claude Code CLI 🎉
```

### Mindset pour Réussir

- 🧪 **Expérimente** : Code tous les exemples !
- 🔄 **Itère** : Fais des erreurs, c'est normal
- 🎯 **Pratique** : 1 concept = 1 mini-projet
- 💬 **Partage** : Explique à quelqu'un ce que tu apprends
- 🚀 **Construis** : Crée des projets personnels

---

## 🏠 Navigation

➡️ [Chapitre 01 : CLI & Architecture](./01-Chapitre-01-Apercu-Interactif.md)

➡️ [Voir Tous les Chapitres](../README.md)

🏠 [Retour Accueil Formation](../../README.md)

---

**🎓 Félicitations d'avoir terminé le survol !**

Tu as maintenant une vue d'ensemble complète de Claude Code CLI. Les 6 prochains chapitres vont approfondir chaque concept avec des dizaines d'exercices pratiques.

**Temps total de formation estimé :** 40-60 heures
**Résultat :** Tu peux construire n'importe quel CLI AI-powered !

**Prêt ? C'est parti ! 🚀**

---

*Formation basée sur l'Approche Hybride Optimale - 100 ans de recherche en sciences cognitives*
