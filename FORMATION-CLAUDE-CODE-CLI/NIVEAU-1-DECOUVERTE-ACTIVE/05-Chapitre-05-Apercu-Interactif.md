# 🎬 NIVEAU 1 : Chapitre 05 - Hooks & Multi-Agents - Aperçu Interactif

> **🎯 Objectif :** Orchestrer plusieurs assistants AI et automatiser avec des hooks
> **🧠 Science :** Active Learning + Problem-Based Learning + Systems Thinking
> **📊 Progression :** [■■■■■■■□□□] 75% du parcours Niveau 1
> **⏱️ Durée :** 120 minutes

---

## 🎮 ACTIVATION : Avant de Commencer

### 🤔 Question Réflexive (Metacognition)

> Imagine que tu as créé ton Claude Code CLI. Tu veux maintenant ajouter des "réflexes" automatiques...
>
> **Réfléchis 60 secondes :**
> - Comment pourrais-tu valider automatiquement les commandes dangereuses ?
> - Comment orchestrer 3 experts différents pour reviewer ton code ?
> - Où intercepterais-tu les actions AVANT qu'elles s'exécutent ?
> - Comment gérerais-tu plusieurs conversations AI en parallèle ?

**💭 Réfléchis avant de scroller...**

---

## 📚 Section 1 : Système de Hooks

### 💡 CONCEPT

**En une phrase :** Les hooks sont des scripts qui s'exécutent automatiquement avant (pre) ou après (post) certaines actions.

**🎨 Analogie :**
> C'est comme les hooks Git (`pre-commit`, `post-push`) : du code qui s'exécute automatiquement à des moments clés, sans que tu doives y penser !

### 🔍 EXPLORATION

**Types de Hooks :**

```
hooks/
├── session-start       → Au démarrage de Claude Code
├── session-end         → À la fin de la session
├── pre-bash           → AVANT l'exécution d'une commande bash
├── post-bash          → APRÈS l'exécution bash
├── pre-tool           → AVANT l'utilisation d'un outil (Read, Write, etc.)
├── post-tool          → APRÈS l'utilisation d'un outil
└── user-message       → Quand l'utilisateur envoie un message
```

**Configuration des Hooks :**

```json
// plugins/my-plugin/hooks/hooks.json
{
  "pre-bash": {
    "description": "Validate bash commands before execution",
    "handler": "./validate-bash.py"
  },
  "post-bash": {
    "description": "Log bash command execution",
    "handler": "./log-bash.js"
  },
  "session-start": {
    "description": "Initialize session",
    "handler": "./session-start.sh"
  }
}
```

### 🎮 CODE : Hook Pre-Bash (Sécurité)

**Exemple : Bloquer les commandes dangereuses**

```python
#!/usr/bin/env python3
# validate-bash.py

import sys
import json
import re

# Lire l'input du hook
hook_input = json.loads(sys.stdin.read())
command = hook_input.get("command", "")

# Commandes dangereuses
DANGEROUS_PATTERNS = [
    r"rm\s+-rf\s+/",           # rm -rf /
    r"dd\s+if=.*of=/dev/sd",   # dd vers un disque
    r":\(\)\{\s*:\|:&\s*\};:", # Fork bomb
    r"mkfs\.",                 # Formater un disque
    r"chmod\s+-R\s+777\s+/",   # chmod 777 recursif sur /
]

def is_dangerous(cmd):
    """Vérifier si la commande est dangereuse"""
    for pattern in DANGEROUS_PATTERNS:
        if re.search(pattern, cmd):
            return True
    return False

# Validation
if is_dangerous(command):
    # Bloquer la commande
    result = {
        "status": "blocked",
        "message": f"❌ COMMANDE BLOQUÉE : Cette commande est dangereuse\n\n{command}\n\nRaison : Risque de destruction de données",
        "command": None  # Ne pas exécuter
    }
else:
    # Autoriser la commande
    result = {
        "status": "approved",
        "message": "✅ Commande validée",
        "command": command  # Exécuter normalement
    }

# Retourner le résultat
print(json.dumps(result))
sys.exit(0 if result["status"] == "approved" else 1)
```

**🎯 Fonctionnement :**
1. Claude veut exécuter `rm -rf /important`
2. Le hook `pre-bash` intercepte
3. Le script Python valide la commande
4. Si dangereuse → Bloque + Alerte
5. Si safe → Laisse passer

### 🎮 CODE : Hook Loader

```javascript
// hook-manager.js
const { spawn } = require('child_process');
const fs = require('fs');
const path = require('path');

class HookManager {
  constructor() {
    this.hooks = {};
  }

  // Charger tous les hooks depuis les plugins
  loadHooks(plugins) {
    for (const plugin of plugins) {
      const hooksConfig = plugin.config.hooks || [];

      for (const hookConfigPath of hooksConfig) {
        const fullPath = path.join(plugin.path, hookConfigPath);
        if (!fs.existsSync(fullPath)) continue;

        const config = JSON.parse(fs.readFileSync(fullPath, 'utf-8'));

        // Pour chaque type de hook (pre-bash, post-bash, etc.)
        for (const [hookType, hookDef] of Object.entries(config)) {
          if (!this.hooks[hookType]) {
            this.hooks[hookType] = [];
          }

          this.hooks[hookType].push({
            plugin: plugin.name,
            handler: path.join(plugin.path, 'hooks', hookDef.handler),
            description: hookDef.description
          });

          console.log(`🪝 Hook loaded: ${hookType} (${plugin.name})`);
        }
      }
    }
  }

  // Exécuter les hooks d'un type donné
  async executeHooks(hookType, data) {
    const hooks = this.hooks[hookType] || [];
    const results = [];

    for (const hook of hooks) {
      console.log(`🔄 Executing hook: ${hookType} from ${hook.plugin}`);

      try {
        const result = await this.runHookHandler(hook.handler, data);
        results.push({ hook: hook.plugin, result });

        // Si un hook bloque (status: blocked)
        if (result.status === 'blocked') {
          console.log(`🚫 Hook blocked: ${result.message}`);
          return { blocked: true, message: result.message };
        }
      } catch (error) {
        console.error(`❌ Hook error: ${error.message}`);
      }
    }

    return { blocked: false, results };
  }

  // Exécuter un handler de hook
  runHookHandler(handlerPath, data) {
    return new Promise((resolve, reject) => {
      const ext = path.extname(handlerPath);
      let command;

      // Déterminer comment exécuter selon l'extension
      if (ext === '.py') {
        command = 'python3';
      } else if (ext === '.js') {
        command = 'node';
      } else if (ext === '.sh') {
        command = 'bash';
      } else {
        return reject(new Error(`Unsupported hook type: ${ext}`));
      }

      const process = spawn(command, [handlerPath]);

      // Envoyer les données au hook via stdin
      process.stdin.write(JSON.stringify(data));
      process.stdin.end();

      let output = '';
      process.stdout.on('data', chunk => {
        output += chunk.toString();
      });

      process.on('close', code => {
        try {
          const result = JSON.parse(output);
          resolve(result);
        } catch (error) {
          reject(new Error(`Hook returned invalid JSON: ${output}`));
        }
      });

      // Timeout de 5 secondes
      setTimeout(() => {
        process.kill();
        reject(new Error('Hook timeout'));
      }, 5000);
    });
  }
}

// Utilisation
const hookManager = new HookManager();
hookManager.loadHooks(plugins);

// Avant d'exécuter une commande bash
async function executeBash(command) {
  // Exécuter hooks pre-bash
  const hookResult = await hookManager.executeHooks('pre-bash', { command });

  if (hookResult.blocked) {
    console.log(hookResult.message);
    return; // Ne pas exécuter
  }

  // Exécuter la commande
  const output = execSync(command, { encoding: 'utf-8' });

  // Exécuter hooks post-bash
  await hookManager.executeHooks('post-bash', { command, output });

  return output;
}
```

### 🎮 PRATIQUE : Créer un Hook de Logging

**🎯 Défi :** Créez un hook qui log toutes les commandes bash exécutées

**📝 `plugins/dev-tools/hooks/hooks.json` :**
```json
{
  "post-bash": {
    "description": "Log all bash commands",
    "handler": "./log-commands.js"
  }
}
```

**`log-commands.js` :**
```javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');

// Lire l'input
const input = JSON.parse(require('fs').readFileSync(0, 'utf-8'));
const { command, output } = input;

// Log file
const logFile = path.join(process.env.HOME, '.claude-code/bash-history.log');

// Créer l'entrée de log
const timestamp = new Date().toISOString();
const logEntry = `
[${timestamp}]
Command: ${command}
Output: ${output.substring(0, 200)}${output.length > 200 ? '...' : ''}
---
`;

// Append au fichier
fs.appendFileSync(logFile, logEntry);

// Retourner succès
console.log(JSON.stringify({
  status: 'logged',
  message: 'Command logged successfully'
}));
```

---

## 📚 Section 2 : Multi-Agents (Orchestration)

### 💡 CONCEPT

**En une phrase :** Au lieu d'un seul Claude, orchestrez plusieurs "agents" spécialisés qui travaillent en parallèle sur différentes tâches.

**🎨 Analogie :**
> C'est comme une équipe de développement : le dev fait le code, le testeur vérifie, le security expert audite, et le tech writer documente - tous en parallèle !

### 🔍 EXPLORATION

**Agents Spécialisés :**

```
agents/
├── code-reviewer.md     → Expert en code review
├── security-auditor.md  → Expert sécurité
├── test-writer.md       → Expert tests
├── doc-writer.md        → Expert documentation
└── optimizer.md         → Expert performance
```

**Exemple d'Agent :**

```markdown
# Code Reviewer Agent

You are an expert code reviewer with 15 years of experience.

## Your Mission:
Review the provided code for:
1. 🐛 Bugs and logic errors
2. 🎨 Code style and readability
3. 🔧 Best practices violations
4. ⚡ Performance issues

## Context:
!file{src/main.js}

## Output Format:
- **Issues Found:** Count
- **Severity:** Critical / High / Medium / Low
- **Details:** List each issue with line number
- **Recommendations:** Specific fixes

Be thorough but constructive.
```

### 🎮 CODE : Multi-Agent Orchestrator

```javascript
// multi-agent.js
class MultiAgentOrchestrator {
  constructor(claudeClient) {
    this.claude = claudeClient;
    this.agents = new Map();
  }

  // Charger les agents depuis les plugins
  loadAgents(plugins) {
    for (const plugin of plugins) {
      const agentsDirs = plugin.config.agents || [];

      for (const agentDir of agentsDirs) {
        const agentPath = path.join(plugin.path, agentDir);
        if (!fs.existsSync(agentPath)) continue;

        const files = fs.readdirSync(agentPath);
        for (const file of files) {
          if (file.endsWith('.md')) {
            const agentName = path.basename(file, '.md');
            const agentPrompt = fs.readFileSync(
              path.join(agentPath, file),
              'utf-8'
            );

            this.agents.set(agentName, {
              name: agentName,
              prompt: agentPrompt,
              plugin: plugin.name
            });

            console.log(`🤖 Agent loaded: ${agentName}`);
          }
        }
      }
    }
  }

  // Exécuter un agent unique
  async runAgent(agentName, context = {}) {
    const agent = this.agents.get(agentName);
    if (!agent) {
      throw new Error(`Agent not found: ${agentName}`);
    }

    console.log(`🚀 Running agent: ${agentName}`);

    // Injecter le contexte dans le prompt
    const injector = new ContextInjector();
    const prompt = await injector.inject(agent.prompt);

    // Appeler Claude avec le prompt de l'agent
    const response = await this.claude.sendMessage([
      { role: 'user', content: prompt }
    ]);

    return {
      agent: agentName,
      response: response.content[0].text
    };
  }

  // Exécuter plusieurs agents EN PARALLÈLE
  async runAgentsParallel(agentNames, context = {}) {
    console.log(`🎬 Orchestrating ${agentNames.length} agents in parallel...`);

    const startTime = Date.now();

    // Créer les promises pour chaque agent
    const promises = agentNames.map(name => this.runAgent(name, context));

    // Exécuter tous en parallèle
    const results = await Promise.all(promises);

    const duration = Date.now() - startTime;
    console.log(`✅ All agents completed in ${duration}ms`);

    return results;
  }

  // Exécuter plusieurs agents EN SÉQUENCE (avec dépendances)
  async runAgentsSequential(agentNames, context = {}) {
    console.log(`🎬 Orchestrating ${agentNames.length} agents sequentially...`);

    const results = [];

    for (const agentName of agentNames) {
      const result = await this.runAgent(agentName, context);
      results.push(result);

      // Le résultat d'un agent peut être utilisé par le suivant
      context[`${agentName}_result`] = result.response;
    }

    return results;
  }

  // Agréger les résultats de plusieurs agents
  aggregateResults(results) {
    const summary = {
      agents: results.length,
      timestamp: new Date().toISOString(),
      results: {}
    };

    for (const result of results) {
      summary.results[result.agent] = {
        length: result.response.length,
        preview: result.response.substring(0, 100) + '...'
      };
    }

    return summary;
  }
}

// Utilisation
const orchestrator = new MultiAgentOrchestrator(claude);
orchestrator.loadAgents(plugins);

// Scénario 1 : Code review complet (parallèle)
const reviewResults = await orchestrator.runAgentsParallel([
  'code-reviewer',
  'security-auditor',
  'test-writer',
  'doc-writer'
]);

console.log('Code Review Results:');
for (const result of reviewResults) {
  console.log(`\n### ${result.agent}:\n${result.response}`);
}

// Scénario 2 : Refactoring pipeline (séquentiel)
const refactorResults = await orchestrator.runAgentsSequential([
  'code-analyzer',    // Analyse le code
  'optimizer',        // Suggère optimisations basées sur l'analyse
  'test-generator'    // Génère tests pour le code optimisé
]);
```

### 🎮 PRATIQUE : Créer 3 Agents Collaboratifs

**🎯 Défi :** Créez un système de review avec 3 agents spécialisés

**1. Code Quality Agent :**

```markdown
# Code Quality Agent

You are a code quality expert focused on maintainability and readability.

## File to Review:
!file{src/index.js}

## Your Tasks:
1. 📏 Check code formatting and style
2. 📝 Verify naming conventions
3. 🧩 Assess modularity and structure
4. 📊 Rate code complexity (1-10)

## Output:
- **Quality Score:** X/10
- **Issues:** List with line numbers
- **Recommendations:** Specific improvements
```

**2. Security Agent :**

```markdown
# Security Agent

You are a security expert specializing in Node.js applications.

## File to Audit:
!file{src/index.js}

## Security Checks:
1. 🔒 Input validation
2. 🛡️ SQL/Command injection risks
3. 🔑 Secrets in code
4. 📦 Vulnerable dependencies

## Output:
- **Security Level:** Safe / Warning / Critical
- **Vulnerabilities:** List with severity
- **Fixes:** Code examples
```

**3. Performance Agent :**

```markdown
# Performance Agent

You are a performance optimization expert.

## File to Optimize:
!file{src/index.js}

## Performance Analysis:
1. ⚡ Algorithmic complexity (Big O)
2. 💾 Memory usage patterns
3. 🔄 Unnecessary computations
4. 🚀 Optimization opportunities

## Output:
- **Performance Grade:** A/B/C/D/F
- **Bottlenecks:** List with impact
- **Optimizations:** Before/after code
```

**Orchestration :**

```javascript
// review-orchestrator.js
async function fullCodeReview(filePath) {
  console.log(`📊 Starting full code review of ${filePath}\n`);

  const orchestrator = new MultiAgentOrchestrator(claude);
  orchestrator.loadAgents(plugins);

  // Exécuter les 3 agents en parallèle
  const results = await orchestrator.runAgentsParallel([
    'code-quality-agent',
    'security-agent',
    'performance-agent'
  ]);

  // Générer le rapport final
  console.log('\n' + '='.repeat(60));
  console.log('FULL CODE REVIEW REPORT');
  console.log('='.repeat(60));

  for (const result of results) {
    console.log(`\n### 🤖 ${result.agent.toUpperCase()}`);
    console.log(result.response);
    console.log('\n' + '-'.repeat(60));
  }

  // Agréger les scores
  const scores = {
    quality: extractScore(results[0].response, 'Quality Score'),
    security: extractSecurityLevel(results[1].response),
    performance: extractGrade(results[2].response)
  };

  console.log('\n### 📊 OVERALL SCORES');
  console.log(JSON.stringify(scores, null, 2));

  return { results, scores };
}

// Tester
fullCodeReview('src/index.js');
```

---

## 📚 Section 3 : Session State Management

### 💡 CONCEPT

**En une phrase :** Gérer l'état partagé entre tous les agents et hooks pendant une session.

**🎨 Analogie :**
> C'est comme un tableau blanc partagé pendant une réunion : tout le monde peut y lire et écrire des infos pour collaborer.

### 🎮 CODE : Session State Manager

```javascript
// session-state.js
class SessionState {
  constructor() {
    this.state = {
      sessionId: this.generateSessionId(),
      startTime: Date.now(),
      user: {},
      context: {},
      history: [],
      agents: {}
    };
  }

  generateSessionId() {
    return `session_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  // Sauvegarder une valeur
  set(key, value) {
    this.state.context[key] = value;
    this.log('set', { key, value });
  }

  // Récupérer une valeur
  get(key) {
    return this.state.context[key];
  }

  // Sauvegarder le résultat d'un agent
  setAgentResult(agentName, result) {
    this.state.agents[agentName] = {
      timestamp: Date.now(),
      result: result
    };
  }

  // Récupérer le résultat d'un agent
  getAgentResult(agentName) {
    return this.state.agents[agentName]?.result;
  }

  // Ajouter à l'historique
  addToHistory(entry) {
    this.state.history.push({
      timestamp: Date.now(),
      ...entry
    });
  }

  // Logger les événements
  log(event, data) {
    console.log(`[${this.state.sessionId}] ${event}:`, data);
  }

  // Exporter l'état pour persistance
  export() {
    return JSON.stringify(this.state, null, 2);
  }

  // Importer un état sauvegardé
  import(stateJson) {
    this.state = JSON.parse(stateJson);
  }

  // Statistiques de session
  getStats() {
    return {
      sessionId: this.state.sessionId,
      duration: Date.now() - this.state.startTime,
      historyEntries: this.state.history.length,
      agentsUsed: Object.keys(this.state.agents).length,
      contextKeys: Object.keys(this.state.context).length
    };
  }
}

// Utilisation avec agents
const session = new SessionState();

// Un agent stocke ses résultats
const result1 = await orchestrator.runAgent('code-reviewer');
session.setAgentResult('code-reviewer', result1.response);

// Un autre agent utilise ces résultats
const codeReview = session.getAgentResult('code-reviewer');
session.set('reviewCompleted', true);

// À la fin
console.log('Session Stats:', session.getStats());
fs.writeFileSync('session.json', session.export());
```

---

## 🧪 MINI-PROJET : Système de CI/CD avec Multi-Agents

### 🎯 Mission

Créez un système de CI/CD complet avec 5 agents orchestrés.

### 📋 Spécifications

**Agents à créer :**
1. **linter-agent** : Vérifie le style de code
2. **tester-agent** : Exécute les tests
3. **security-agent** : Scan de sécurité
4. **builder-agent** : Build le projet
5. **deploy-agent** : Suggère déploiement

**Workflow :**
```
[Linter] ──┐
           │
[Tester] ──┼──> [Builder] ──> [Deploy]
           │
[Security]─┘
```

### 🎮 À TOI DE CODER !

**Structure :**
```
plugins/ci-cd-system/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   ├── linter-agent.md
│   ├── tester-agent.md
│   ├── security-agent.md
│   ├── builder-agent.md
│   └── deploy-agent.md
└── orchestrator.js
```

**`orchestrator.js` complet :**

```javascript
#!/usr/bin/env node

const MultiAgentOrchestrator = require('./multi-agent');
const SessionState = require('./session-state');
const claudeClient = require('./claude-client');

async function runCICD() {
  console.log('🚀 Starting CI/CD Pipeline...\n');

  const orchestrator = new MultiAgentOrchestrator(claudeClient);
  const session = new SessionState();

  orchestrator.loadAgents(plugins);

  // Phase 1: Quality Checks (parallèle)
  console.log('📊 Phase 1: Quality Checks');
  const qualityResults = await orchestrator.runAgentsParallel([
    'linter-agent',
    'tester-agent',
    'security-agent'
  ]);

  // Sauvegarder dans la session
  for (const result of qualityResults) {
    session.setAgentResult(result.agent, result.response);
  }

  // Vérifier si tous les checks passent
  const allPassed = qualityResults.every(r =>
    r.response.includes('✅') || r.response.includes('PASSED')
  );

  if (!allPassed) {
    console.log('❌ Quality checks failed. Stopping pipeline.');
    return { status: 'failed', results: qualityResults };
  }

  session.set('qualityChecksPassed', true);

  // Phase 2: Build (séquentiel)
  console.log('\n🔨 Phase 2: Build');
  const buildResult = await orchestrator.runAgent('builder-agent', {
    qualityResults: session.get('qualityChecksPassed')
  });

  session.setAgentResult('builder-agent', buildResult.response);

  // Vérifier le build
  if (!buildResult.response.includes('BUILD SUCCESS')) {
    console.log('❌ Build failed. Stopping pipeline.');
    return { status: 'build-failed', results: [buildResult] };
  }

  session.set('buildSuccess', true);

  // Phase 3: Deploy Suggestion
  console.log('\n🚀 Phase 3: Deploy Suggestion');
  const deployResult = await orchestrator.runAgent('deploy-agent', {
    buildArtifacts: session.get('buildSuccess')
  });

  session.setAgentResult('deploy-agent', deployResult.response);

  // Rapport final
  console.log('\n' + '='.repeat(60));
  console.log('CI/CD PIPELINE COMPLETE');
  console.log('='.repeat(60));
  console.log(session.getStats());

  // Sauvegarder la session
  fs.writeFileSync('ci-cd-session.json', session.export());

  return { status: 'success', session };
}

// Exécuter
runCICD().then(result => {
  console.log('\n✅ Pipeline finished:', result.status);
}).catch(error => {
  console.error('\n❌ Pipeline error:', error);
  process.exit(1);
});
```

---

## 🎯 QUIZ INTERLEAVING

### Question 1 : Hooks

À quel moment un hook `pre-bash` s'exécute-t-il ?

<details>
<summary>💡 Voir la réponse</summary>

**Réponse :** AVANT l'exécution d'une commande bash.

Les hooks `pre-*` interceptent l'action avant qu'elle se produise, permettant de valider, bloquer, ou modifier l'action.

</details>

### Question 2 : Multi-Agents

Quelle méthode JavaScript permet d'exécuter plusieurs agents en parallèle ?

<details>
<summary>💡 Voir la réponse</summary>

**Réponse :** `Promise.all([agent1(), agent2(), agent3()])`

`Promise.all` exécute toutes les promises en parallèle et attend que toutes se terminent.

</details>

### Question 3 : Session State

À quoi sert `session.setAgentResult(agentName, result)` ?

<details>
<summary>💡 Voir la réponse</summary>

**Réponse :** Sauvegarder le résultat d'un agent pour que d'autres agents puissent l'utiliser plus tard.

C'est essentiel pour la collaboration entre agents : un agent analyse, un autre utilise cette analyse.

</details>

### Question 4 : Code

Ce hook bloque-t-il correctement les commandes dangereuses ?

```python
if "rm -rf" in command:
    return {"status": "approved"}
```

<details>
<summary>💡 Voir la réponse</summary>

**Non !** Il devrait bloquer (`"blocked"`) au lieu d'approuver.

**Correct :**
```python
if "rm -rf" in command:
    return {"status": "blocked", "message": "Dangerous command"}
```

</details>

### Question 5 : Architecture

Tu veux : (1) analyser le code, (2) optimiser selon l'analyse, (3) tester le code optimisé.

Utilises-tu `runAgentsParallel` ou `runAgentsSequential` ?

<details>
<summary>💡 Voir la réponse</summary>

**Réponse :** `runAgentsSequential`

Car chaque étape dépend de la précédente :
- L'optimisation nécessite l'analyse
- Les tests nécessitent le code optimisé

C'est une dépendance séquentielle, pas du parallélisme.

</details>

---

## 📅 RÉVISION ESPACÉE

### J+1 (Demain)
- ✅ Recrée le hook `validate-bash.py` sans aide
- ✅ Ajoute 3 nouveaux patterns dangereux
- ✅ Crée un hook `post-tool` qui log tous les fichiers modifiés

### J+3 (Dans 3 jours)
- ✅ Crée 3 agents pour ton projet personnel
- ✅ Orchestrer ces agents en parallèle
- ✅ Quiz : 10 questions sur hooks et agents

### J+7 (Dans 1 semaine)
- ✅ Construis un système CI/CD complet
- ✅ Compare avec les vrais outils (GitHub Actions, CircleCI)

### J+14 (Dans 2 semaines)
- ✅ Quiz combinant Ch01-05
- ✅ Projet : Extension de ton propre CLI avec hooks + agents

### J+30 (Dans 1 mois)
- ✅ Challenge : 5 agents collaboratifs pour une vraie tâche
- ✅ Contribue à un projet open source avec agents

---

## 📊 AUTO-ÉVALUATION

Coche ce que tu maîtrises :

**Hooks :**
- [ ] Je comprends pre vs post hooks
- [ ] Je peux créer un hook Python/JS
- [ ] Je sais bloquer des actions dangereuses
- [ ] Je comprends la config `hooks.json`

**Multi-Agents :**
- [ ] Je comprends l'orchestration parallèle vs séquentielle
- [ ] Je peux créer un agent spécialisé
- [ ] Je sais utiliser `Promise.all`
- [ ] Je peux agréger les résultats d'agents

**Session State :**
- [ ] Je comprends le state management
- [ ] Je peux partager des données entre agents
- [ ] Je sais persister une session
- [ ] Je comprends l'utilité des stats

**Projet :**
- [ ] J'ai créé le système CI/CD complet
- [ ] J'ai orchestré 5 agents
- [ ] Tout fonctionne correctement
- [ ] Je peux adapter à d'autres use cases

**Score :**
- 12-16 coches : 🎉 Excellent ! Ch06 final !
- 8-11 coches : 👍 Bien ! Pratique encore
- 4-7 coches : 📚 Relis les sections difficiles
- 0-3 coches : 🔄 Reprends depuis le début

---

## 🚀 PROCHAINES ÉTAPES

### Option 1 : Projet Final (Chapitre 06)
➡️ **Chapitre 06 : Projet Final Intégré**

Tu vas :
- Assembler TOUS les concepts (Ch01-05)
- Construire un Claude Code CLI complet
- Plugins + Hooks + Multi-Agents
- Tests et déploiement

Durée : 60 minutes

### Option 2 : Approfondir (Niveau 2)
➡️ **Chapitre 05 Détaillé - Niveau 2**

Contenu :
- Hooks avancés (timeout, retry, fallback)
- Agent communication protocols
- State persistence (Redis, SQLite)
- Load balancing entre agents
- Monitoring et observability

Durée : 6-8 heures

---

## 💡 POINTS CLÉS À RETENIR

### ✅ Hooks
- Interceptent les actions (pre/post)
- Peuvent bloquer ou modifier
- Scripts Python/JS/Bash
- Configuration dans `hooks.json`

### ✅ Multi-Agents
- Agents spécialisés (experts)
- Parallèle (`Promise.all`) vs Séquentiel
- Chaque agent = prompt Markdown
- Orchestration = coordination intelligente

### ✅ Session State
- État partagé entre agents/hooks
- Persistance possible
- Collaboration et coordination
- Stats et monitoring

### ✅ Best Practices
- Valider toujours les entrées (hooks)
- Timeout pour éviter blocages
- Logger tous les événements
- Gérer les erreurs gracieusement

---

## 🏠 Navigation

⬅️ [Chapitre 04 : Plugins & Slash Commands](./04-Chapitre-04-Apercu-Interactif.md)

➡️ [Chapitre 06 : Projet Final Intégré](./06-Chapitre-06-Apercu-Interactif.md)

🏠 [Retour au Niveau 1](./00-Survol-Interactif-Complet.md)

---

**🎓 Félicitations !** Tu maîtrises maintenant les hooks et l'orchestration multi-agents ! Tu peux créer des systèmes AI sophistiqués avec plusieurs experts collaboratifs.

**Prochaine étape :** Le Chapitre Final où tu assembleras TOUT pour créer ton propre Claude Code CLI ! 🚀
