# 🎬 NIVEAU 1 : Chapitre 04 - Plugins & Slash Commands - Aperçu Interactif

> **🎯 Objectif :** Étendre Claude Code CLI avec des plugins personnalisés et slash commands
> **🧠 Science :** Active Learning + Problem-Based Learning + Transfer Learning
> **📊 Progression :** [■■■■■■□□□□] 60% du parcours Niveau 1
> **⏱️ Durée :** 120 minutes

---

## 🎮 ACTIVATION : Avant de Commencer

### 🤔 Question Réflexive (Metacognition)

> Tu as maintenant un CLI avec Claude AI intégré. Mais chaque utilisateur a des besoins différents...
>
> **Réfléchis 60 secondes :**
> - Comment permettrais-tu aux utilisateurs d'ajouter leurs propres fonctionnalités ?
> - Comment créer des "raccourcis" pour des tâches répétitives ?
> - Comment partager ces fonctionnalités avec d'autres développeurs ?
> - Où stockerais-tu ces extensions ? Comment les charger automatiquement ?

**💭 Réfléchis avant de scroller...**

---

## 📚 Section 1 : Architecture des Plugins

### 💡 CONCEPT

**En une phrase :** Un plugin est un dossier avec un fichier JSON de configuration qui ajoute des fonctionnalités à Claude Code.

**🎨 Analogie :**
> C'est comme les extensions Chrome : chaque extension ajoute des boutons, des menus, des fonctionnalités... mais Chrome les charge tous automatiquement !

### 🔍 EXPLORATION

**Structure d'un Plugin :**

```
plugins/
└── my-plugin/
    ├── .claude-plugin/
    │   └── plugin.json          # Configuration du plugin
    ├── commands/                # Slash commands (ex: /commit)
    │   └── my-command.md
    ├── agents/                  # Agents spécialisés
    │   └── my-agent.md
    ├── hooks/                   # Hooks pre/post execution
    │   └── hooks.json
    └── README.md                # Documentation
```

**plugin.json - Configuration :**

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Description de mon plugin",
  "commands": ["commands"],
  "agents": ["agents"],
  "hooks": ["hooks/hooks.json"]
}
```

**🎯 Ce qui se passe :**
1. Au démarrage, Claude Code scanne le dossier `plugins/`
2. Pour chaque plugin, il lit `.claude-plugin/plugin.json`
3. Il charge les commands, agents, hooks définis
4. Les slash commands deviennent disponibles (ex: `/my-command`)

### 🎮 CODE : Plugin Discovery

**Comment Claude Code découvre les plugins :**

```javascript
// plugin-loader.js
const fs = require('fs');
const path = require('path');

class PluginLoader {
  constructor(pluginsDir = './plugins') {
    this.pluginsDir = pluginsDir;
    this.plugins = [];
  }

  // Découvrir tous les plugins
  discoverPlugins() {
    const dirs = fs.readdirSync(this.pluginsDir);

    for (const dir of dirs) {
      const pluginPath = path.join(this.pluginsDir, dir);
      const configPath = path.join(pluginPath, '.claude-plugin/plugin.json');

      // Vérifier si c'est un plugin valide
      if (fs.existsSync(configPath)) {
        const config = JSON.parse(fs.readFileSync(configPath, 'utf-8'));
        this.plugins.push({
          name: config.name,
          path: pluginPath,
          config: config
        });
        console.log(`✅ Plugin découvert : ${config.name}`);
      }
    }

    return this.plugins;
  }

  // Charger les slash commands d'un plugin
  loadCommands(plugin) {
    const commands = [];
    const commandsDirs = plugin.config.commands || [];

    for (const cmdDir of commandsDirs) {
      const cmdPath = path.join(plugin.path, cmdDir);
      if (!fs.existsSync(cmdPath)) continue;

      const files = fs.readdirSync(cmdPath);
      for (const file of files) {
        if (file.endsWith('.md')) {
          const cmdName = path.basename(file, '.md');
          const cmdContent = fs.readFileSync(
            path.join(cmdPath, file),
            'utf-8'
          );
          commands.push({ name: cmdName, content: cmdContent });
          console.log(`  → Command: /${cmdName}`);
        }
      }
    }

    return commands;
  }

  // Charger tous les plugins
  loadAll() {
    this.discoverPlugins();
    const allCommands = [];

    for (const plugin of this.plugins) {
      console.log(`\n📦 Chargement : ${plugin.name}`);
      const commands = this.loadCommands(plugin);
      allCommands.push(...commands);
    }

    return { plugins: this.plugins, commands: allCommands };
  }
}

// Utilisation
const loader = new PluginLoader('./plugins');
const { plugins, commands } = loader.loadAll();

console.log(`\n✅ ${plugins.length} plugins chargés`);
console.log(`✅ ${commands.length} slash commands disponibles`);
```

**🎯 Sortie :**
```
✅ Plugin découvert : commit-commands
✅ Plugin découvert : feature-dev

📦 Chargement : commit-commands
  → Command: /commit
  → Command: /commit-push-pr

📦 Chargement : feature-dev
  → Command: /feature-dev

✅ 2 plugins chargés
✅ 3 slash commands disponibles
```

### 🎮 PRATIQUE : Créer Votre Premier Plugin

**🎯 Défi :** Créez un plugin `hello-plugin` avec un fichier `plugin.json`

```bash
mkdir -p plugins/hello-plugin/.claude-plugin
mkdir -p plugins/hello-plugin/commands
```

**📝 Créez `plugin.json` :**
```json
{
  "name": "hello-plugin",
  "version": "1.0.0",
  "description": "Mon premier plugin Claude Code",
  "commands": ["commands"]
}
```

**✅ Testez le chargement :**
```javascript
const loader = new PluginLoader('./plugins');
const { plugins } = loader.discoverPlugins();
console.log(plugins.map(p => p.name));
// ["commit-commands", "feature-dev", "hello-plugin"]
```

---

## 📚 Section 2 : Slash Commands (Commandes Markdown)

### 💡 CONCEPT

**En une phrase :** Un slash command est un fichier Markdown qui contient un prompt pour Claude, avec métadonnées YAML optionnelles.

**🎨 Analogie :**
> C'est comme un raccourci clavier, mais pour Claude. Tu tapes `/commit` et Claude reçoit tout un prompt pré-écrit pour créer un commit.

### 🔍 EXPLORATION

**Anatomie d'un Slash Command :**

```markdown
---
description: Create a git commit with AI-generated message
---

# /commit - AI Commit Generator

You are a git expert. Follow these steps:

1. Run `git status` to see changed files
2. Run `git diff` to see the changes
3. Analyze the changes and write a clear commit message
4. Use conventional commit format: `type(scope): message`
5. Commit the changes with `git add . && git commit -m "message"`

**Types:** feat, fix, docs, style, refactor, test, chore
```

**🎯 Structure :**
- **Frontmatter YAML** (optionnel) : Métadonnées entre `---`
- **Titre** : Nom et description
- **Prompt** : Instructions pour Claude

### 🎮 CODE : Parser un Slash Command

```javascript
// command-parser.js
class CommandParser {
  // Extraire le frontmatter YAML et le contenu
  static parse(markdown) {
    const frontmatterRegex = /^---\n([\s\S]*?)\n---\n([\s\S]*)$/;
    const match = markdown.match(frontmatterRegex);

    if (match) {
      // Avec frontmatter
      const yamlContent = match[1];
      const markdownContent = match[2];

      // Parser le YAML (simplifié)
      const metadata = {};
      yamlContent.split('\n').forEach(line => {
        const [key, ...valueParts] = line.split(':');
        if (key && valueParts.length) {
          metadata[key.trim()] = valueParts.join(':').trim();
        }
      });

      return {
        metadata,
        content: markdownContent.trim()
      };
    } else {
      // Sans frontmatter
      return {
        metadata: {},
        content: markdown.trim()
      };
    }
  }

  // Injecter le contexte (!bash, !file, etc.)
  static async injectContext(content) {
    // Rechercher les patterns !bash{command}
    const bashRegex = /!bash\{([^}]+)\}/g;
    let result = content;

    const matches = content.matchAll(bashRegex);
    for (const match of matches) {
      const command = match[1];
      const output = execSync(command, { encoding: 'utf-8' });
      result = result.replace(match[0], `\`\`\`\n${output}\n\`\`\``);
    }

    return result;
  }
}

// Exemple d'utilisation
const commandMd = `---
description: Create a commit
---

# /commit

Analyze these changes:

!bash{git diff}

And create a commit.
`;

const parsed = CommandParser.parse(commandMd);
console.log('Metadata:', parsed.metadata);
console.log('Content:', parsed.content);

const withContext = await CommandParser.injectContext(parsed.content);
console.log('\nWith context injected:');
console.log(withContext);
```

**🎯 Sortie :**
```
Metadata: { description: 'Create a commit' }
Content: # /commit

Analyze these changes:

!bash{git diff}

And create a commit.

With context injected:
# /commit

Analyze these changes:

```
diff --git a/file.js b/file.js
index 123..456
+ console.log('new code');
```

And create a commit.
```

### 🎮 PRATIQUE : Créer un Slash Command

**🎯 Défi 1 :** Créez `/hello` qui salue l'utilisateur

```bash
# Créez plugins/hello-plugin/commands/hello.md
```

**📝 Contenu :**
```markdown
---
description: Say hello to the user
---

# /hello - Friendly Greeting

Say a friendly hello to the user. Ask their name and what they're working on today.
Be encouraging and offer to help with their Claude Code CLI project!
```

**✅ Testez :**
```javascript
const fs = require('fs');
const cmdContent = fs.readFileSync(
  'plugins/hello-plugin/commands/hello.md',
  'utf-8'
);
const parsed = CommandParser.parse(cmdContent);
console.log(parsed);

// Résultat :
// {
//   metadata: { description: 'Say hello to the user' },
//   content: '# /hello - Friendly Greeting\n\nSay a friendly...'
// }
```

**🎯 Défi 2 :** Créez `/analyze` qui analyse le projet

```markdown
---
description: Analyze current project structure
---

# /analyze - Project Analyzer

You are a project analysis expert.

## Project Structure:
!bash{find . -type f -name "*.js" -o -name "*.json" | head -20}

## Package.json:
!bash{cat package.json}

## Git Status:
!bash{git status --short}

Analyze:
1. Project type and tech stack
2. Code organization
3. Potential improvements
4. Missing files (README, tests, etc.)

Provide actionable recommendations.
```

---

## 📚 Section 3 : Injection de Contexte (!bash)

### 💡 CONCEPT

**En une phrase :** L'injection de contexte permet d'insérer dynamiquement des données (résultats de commandes, fichiers, etc.) dans les prompts.

**🎨 Analogie :**
> C'est comme un template avec variables : `!bash{date}` devient automatiquement `2025-01-15` avant d'envoyer à Claude.

### 🔍 PATTERNS D'INJECTION

**Patterns Supportés :**

```markdown
<!-- Exécuter une commande bash -->
!bash{git log --oneline -5}

<!-- Lire un fichier -->
!file{src/main.js}

<!-- Rechercher dans le code -->
!grep{TODO}

<!-- Lister des fichiers -->
!glob{**/*.test.js}
```

### 🎮 CODE : Système d'Injection Complet

```javascript
// context-injector.js
const { execSync } = require('child_process');
const fs = require('fs');
const glob = require('glob');

class ContextInjector {
  constructor() {
    this.injectors = {
      bash: this.injectBash.bind(this),
      file: this.injectFile.bind(this),
      grep: this.injectGrep.bind(this),
      glob: this.injectGlob.bind(this)
    };
  }

  // !bash{command}
  injectBash(command) {
    try {
      const output = execSync(command, {
        encoding: 'utf-8',
        maxBuffer: 1024 * 1024 // 1MB
      });
      return `\`\`\`bash\n$ ${command}\n${output}\n\`\`\``;
    } catch (error) {
      return `\`\`\`\nError: ${error.message}\n\`\`\``;
    }
  }

  // !file{path}
  injectFile(filePath) {
    try {
      const content = fs.readFileSync(filePath, 'utf-8');
      const ext = filePath.split('.').pop();
      return `\`\`\`${ext}\n// ${filePath}\n${content}\n\`\`\``;
    } catch (error) {
      return `File not found: ${filePath}`;
    }
  }

  // !grep{pattern}
  injectGrep(pattern) {
    try {
      const output = execSync(
        `grep -r "${pattern}" . --include="*.js" --exclude-dir=node_modules`,
        { encoding: 'utf-8' }
      );
      return `\`\`\`\nMatches for "${pattern}":\n${output}\n\`\`\``;
    } catch (error) {
      return `No matches found for: ${pattern}`;
    }
  }

  // !glob{pattern}
  injectGlob(pattern) {
    try {
      const files = glob.sync(pattern);
      return `\`\`\`\nFiles matching "${pattern}":\n${files.join('\n')}\n\`\`\``;
    } catch (error) {
      return `Error: ${error.message}`;
    }
  }

  // Injecter tous les patterns
  async inject(content) {
    let result = content;

    // Pour chaque type d'injection (!bash, !file, etc.)
    for (const [type, injector] of Object.entries(this.injectors)) {
      const regex = new RegExp(`!${type}\\{([^}]+)\\}`, 'g');

      const matches = [...content.matchAll(regex)];
      for (const match of matches) {
        const param = match[1];
        const injected = injector(param);
        result = result.replace(match[0], injected);
      }
    }

    return result;
  }
}

// Utilisation
const injector = new ContextInjector();

const prompt = `
# Analyze Project

## Git Status:
!bash{git status --short}

## Main File:
!file{src/index.js}

## TODO Items:
!grep{TODO}

## Test Files:
!glob{**/*.test.js}
`;

const injected = await injector.inject(prompt);
console.log(injected);
```

### 🎮 PRATIQUE : Commande avec Injection

**🎯 Défi :** Créez `/summary` qui résume le projet

**📝 `plugins/hello-plugin/commands/summary.md` :**

```markdown
---
description: Generate project summary
---

# /summary - Project Summary Generator

Create a comprehensive project summary.

## Basic Info:
!bash{cat package.json | grep -E "name|version|description"}

## Recent Activity:
!bash{git log --oneline -10}

## Current Branch:
!bash{git branch --show-current}

## Changed Files:
!bash{git status --short}

## Code Statistics:
!bash{find . -name "*.js" | xargs wc -l | tail -1}

Generate:
1. 📊 Project overview (name, version, description)
2. 🔨 Recent work (from git log)
3. 📁 Current status (branch, changed files)
4. 📈 Code statistics
5. 🎯 What's next (based on TODOs and issues)

Format as nice markdown with emojis.
```

**✅ Testez l'injection :**

```javascript
const injector = new ContextInjector();
const cmdContent = fs.readFileSync(
  'plugins/hello-plugin/commands/summary.md',
  'utf-8'
);

const parsed = CommandParser.parse(cmdContent);
const injected = await injector.inject(parsed.content);

console.log(injected);
// Tous les !bash{...} sont remplacés par les vraies sorties !
```

---

## 🧪 MINI-PROJET : Plugin Complet "Dev Tools"

### 🎯 Mission

Créez un plugin `dev-tools` avec 3 slash commands :
1. `/analyze` - Analyse le projet
2. `/cleanup` - Suggère nettoyages
3. `/docs` - Génère documentation

### 📋 Spécifications

**Structure :**
```
plugins/dev-tools/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── analyze.md
│   ├── cleanup.md
│   └── docs.md
└── README.md
```

### 🎮 À TOI DE CODER !

**Étape 1 : Configuration**

```bash
mkdir -p plugins/dev-tools/.claude-plugin
mkdir -p plugins/dev-tools/commands
```

**`plugin.json` :**
```json
{
  "name": "dev-tools",
  "version": "1.0.0",
  "description": "Essential development tools",
  "author": "Your Name",
  "commands": ["commands"]
}
```

**Étape 2 : Commande `/analyze`**

```markdown
---
description: Analyze project structure and quality
---

# /analyze - Code Analyzer

## Project Structure:
!bash{tree -L 2 -I node_modules}

## Package Dependencies:
!bash{cat package.json | grep -A 20 dependencies}

## Code Quality:
!bash{find . -name "*.js" -not -path "./node_modules/*" | head -10 | xargs wc -l}

## Git Status:
!bash{git status --short}

## Recent Commits:
!bash{git log --oneline -5}

Provide:
1. 📊 Project summary
2. 🔍 Code quality assessment
3. ⚠️ Potential issues
4. 💡 Recommendations
```

**Étape 3 : Commande `/cleanup`**

```markdown
---
description: Suggest code cleanup actions
---

# /cleanup - Code Cleanup Advisor

## Unused Dependencies:
!bash{npm ls --all 2>&1 | grep "extraneous" || echo "All dependencies are used"}

## Large Files:
!bash{find . -type f -size +1M -not -path "./node_modules/*"}

## TODO/FIXME Comments:
!grep{TODO|FIXME}

## Git Untracked Files:
!bash{git status --short | grep "??"}

Suggest:
1. 🗑️ Files to delete
2. 📦 Dependencies to remove
3. ✏️ TODOs to address
4. 🧹 Other cleanup actions
```

**Étape 4 : Commande `/docs`**

```markdown
---
description: Generate project documentation
---

# /docs - Documentation Generator

## Project Info:
!bash{cat package.json}

## Main Entry Point:
!file{src/index.js}

## API/Exports:
!grep{module.exports|export}

## README Exists:
!bash{[ -f README.md ] && echo "✅ README found" || echo "❌ No README"}

Generate:
1. 📝 README.md template (if missing)
2. 📚 API documentation
3. 🚀 Quick Start guide
4. 💡 Usage examples
```

**Étape 5 : README du Plugin**

```markdown
# Dev Tools Plugin

Essential development tools for Claude Code CLI.

## Commands

### /analyze
Analyze project structure, dependencies, and code quality.

### /cleanup
Suggest cleanup actions (unused deps, large files, TODOs).

### /docs
Generate project documentation (README, API docs).

## Installation

```bash
# Plugin is auto-loaded from plugins/dev-tools/
```

## Usage

```bash
$ claude /analyze
$ claude /cleanup
$ claude /docs
```
```

**Étape 6 : Tester le Plugin**

```javascript
// test-plugin.js
const PluginLoader = require('./plugin-loader');
const CommandParser = require('./command-parser');
const ContextInjector = require('./context-injector');

async function testDevTools() {
  // 1. Charger le plugin
  const loader = new PluginLoader('./plugins');
  const { commands } = loader.loadAll();

  // 2. Trouver /analyze
  const analyzeCmd = commands.find(c => c.name === 'analyze');
  console.log('✅ Command found:', analyzeCmd.name);

  // 3. Parser et injecter
  const parsed = CommandParser.parse(analyzeCmd.content);
  const injector = new ContextInjector();
  const ready = await injector.inject(parsed.content);

  console.log('\n📤 Ready to send to Claude:\n');
  console.log(ready);

  // 4. Envoyer à Claude (vous avez déjà le code du Chapitre 03 !)
  // const response = await claude.sendMessage(ready);
}

testDevTools();
```

---

## 🎯 QUIZ INTERLEAVING

### Question 1 : Architecture

Quel fichier Claude Code cherche-t-il pour détecter un plugin ?

<details>
<summary>💡 Voir la réponse</summary>

**Réponse :** `.claude-plugin/plugin.json`

C'est le fichier de configuration obligatoire qui définit le nom, version, et chemins vers commands/agents/hooks.

</details>

### Question 2 : Slash Commands

Complète ce slash command :

```markdown
---
description: ???
---

# /commit

Analyze the git diff and create a commit message.

!bash{???}
```

<details>
<summary>💡 Voir la réponse</summary>

```markdown
---
description: Create AI-generated git commit
---

# /commit

Analyze the git diff and create a commit message.

!bash{git diff}
```

</details>

### Question 3 : Injection

Quel pattern utiliserais-tu pour insérer le contenu du fichier `config.json` ?

<details>
<summary>💡 Voir la réponse</summary>

**Réponse :** `!file{config.json}`

Les patterns d'injection sont :
- `!bash{command}` - Exécuter commande
- `!file{path}` - Lire fichier
- `!grep{pattern}` - Rechercher
- `!glob{pattern}` - Lister fichiers

</details>

### Question 4 : Debugging

Ce code ne charge pas les commands. Pourquoi ?

```json
{
  "name": "my-plugin",
  "command": ["commands"]
}
```

<details>
<summary>💡 Voir la réponse</summary>

**Erreur :** C'est `"commands"` (pluriel), pas `"command"`.

**Correct :**
```json
{
  "name": "my-plugin",
  "commands": ["commands"]
}
```

</details>

### Question 5 : Application

Tu veux créer `/test` qui exécute les tests et analyse les résultats. Écris le fichier.

<details>
<summary>💡 Voir la réponse</summary>

```markdown
---
description: Run tests and analyze results
---

# /test - Test Runner & Analyzer

## Run Tests:
!bash{npm test}

## Test Files:
!glob{**/*.test.js}

## Coverage (if available):
!bash{npm run coverage 2>/dev/null || echo "No coverage configured"}

Analyze:
1. 📊 Tests passed/failed
2. 🔍 Failed tests details
3. 📈 Coverage percentage
4. 💡 Suggestions to improve tests
```

</details>

---

## 📅 RÉVISION ESPACÉE

### J+1 (Demain)
- ✅ Recrée le plugin `dev-tools` sans regarder le code
- ✅ Ajoute une 4ème commande `/refactor`

### J+3 (Dans 3 jours)
- ✅ Quiz : 10 questions sur plugins et slash commands
- ✅ Crée un plugin pour TON workflow personnel

### J+7 (Dans 1 semaine)
- ✅ Crée un plugin marketplace (partageable)
- ✅ Compare avec les vrais plugins de Claude Code

### J+14 (Dans 2 semaines)
- ✅ Quiz combinant Ch01-04
- ✅ Crée une extension pour ton projet perso

### J+30 (Dans 1 mois)
- ✅ Challenge : Plugin complet avec commands + agents + hooks
- ✅ Contribue à un plugin open source

---

## 📊 AUTO-ÉVALUATION

### Compétences Acquises

Coche honnêtement ce que tu maîtrises :

**Architecture Plugins :**
- [ ] Je comprends la structure d'un plugin
- [ ] Je sais créer un `plugin.json` valide
- [ ] Je peux organiser commands/agents/hooks
- [ ] Je comprends le plugin discovery

**Slash Commands :**
- [ ] Je sais créer un fichier .md de commande
- [ ] Je maîtrise le frontmatter YAML
- [ ] Je peux parser le Markdown
- [ ] Je sais structurer un bon prompt

**Injection de Contexte :**
- [ ] Je comprends les patterns (!bash, !file, etc.)
- [ ] Je peux implémenter un système d'injection
- [ ] Je sais quand utiliser chaque pattern
- [ ] Je gère les erreurs d'injection

**Projet Pratique :**
- [ ] J'ai créé le plugin `dev-tools` complet
- [ ] J'ai testé toutes les commandes
- [ ] Je peux créer mes propres plugins
- [ ] Je comprends le cycle complet

**Score :**
- 12-16 coches : 🎉 Excellent ! Continue au Ch05
- 8-11 coches : 👍 Bien ! Pratique encore un peu
- 4-7 coches : 📚 Relis les sections difficiles
- 0-3 coches : 🔄 Reprends depuis le début du chapitre

---

## 🚀 PROCHAINES ÉTAPES

### Option 1 : Approfondir (Niveau 2)
➡️ **Chapitre 04 Détaillé - Niveau 2**

Contenu :
- Architecture plugin avancée
- Agents spécialisés
- Hot reloading de plugins
- Plugin marketplace
- Tests de plugins
- Publication npm

Durée : 6-8 heures

### Option 2 : Continuer (Chapitre Suivant)
➡️ **Chapitre 05 : Hooks & Multi-Agents**

Tu vas apprendre :
- Système de hooks (pre/post)
- Orchestration d'agents
- Parallélisation (Promise.all)
- Communication inter-agents
- Session state management

Durée : 120 minutes

### Option 3 : Projet Personnel
**Idées de Plugins à Créer :**

1. **git-tools** : /commit, /pr, /review, /cherry-pick
2. **code-quality** : /lint, /format, /security, /deps
3. **productivity** : /pomodoro, /notes, /bookmark
4. **deploy** : /build, /deploy, /rollback, /logs
5. **ai-assistant** : /explain, /optimize, /test, /document

Choisis-en un et construis-le complètement !

---

## 💡 POINTS CLÉS À RETENIR

### ✅ Plugins
- Un plugin = dossier avec `.claude-plugin/plugin.json`
- Contient : commands, agents, hooks
- Auto-découverts au démarrage
- Extensibles et partageables

### ✅ Slash Commands
- Fichiers Markdown dans `commands/`
- Frontmatter YAML optionnel
- Contiennent des prompts pour Claude
- Chargés automatiquement

### ✅ Injection de Contexte
- `!bash{cmd}` : Exécuter commande
- `!file{path}` : Lire fichier
- `!grep{pattern}` : Rechercher
- `!glob{pattern}` : Lister fichiers
- Injecté AVANT l'envoi à Claude

### ✅ Best Practices
- Documenter chaque commande (description)
- Gérer les erreurs d'injection
- Limiter les outputs (maxBuffer)
- Tester avec vrais projets
- Partager dans le marketplace

---

## 🏠 Navigation

⬅️ [Chapitre 03 : Claude API & Conversation](./03-Chapitre-03-Apercu-Interactif.md)

➡️ [Chapitre 05 : Hooks & Multi-Agents](./05-Chapitre-05-Apercu-Interactif.md)

🏠 [Retour au Niveau 1](./00-Survol-Interactif-Complet.md)

---

**🎓 Félicitations !** Tu maîtrises maintenant le système de plugins de Claude Code CLI. Tu peux créer tes propres extensions et slash commands pour automatiser ton workflow !

**Prochaine étape :** Les Hooks et Multi-Agents pour orchestrer plusieurs assistants AI en parallèle ! 🚀
