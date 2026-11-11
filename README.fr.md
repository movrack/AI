# AI

[English](README.md) | Français

Des agents et des compétences pour les assistants de programmation, écrits une seule fois et réutilisables dans n'importe quel projet.

Chaque agent est un simple fichier Markdown : un petit en-tête (nom, description, outils) et des instructions. Un agent
ne nomme jamais un projet : il lit les règles du projet dans lequel il travaille (`AGENTS.md`, `CLAUDE.md`, `README`,
glossaire) et les applique. Les instructions sont en anglais ; les rapports sont rédigés en anglais, sauf si votre configuration indique autre chose.

## Agents

| Agent | Rôle | Écrit | Appel parlé |
|---|---|---|---|
| [`john-117`](agents/john-117.md) | Orchestrateur (le Master Chief de Halo) : analyse, découpe le travail, choisit l'agent et le modèle, délègue, contrôle les preuves, fait son rapport | rien | « lance John-117 » |
| [`nemesis`](agents/nemesis.md) | Avocat du diable des spécifications et des plans : contradictions, cas limites, omissions, risques, faisabilité ; liste classée des failles | rien | « lance Némésis » |
| [`mamy`](agents/mamy.md) | Juge si chaque écran peut être compris sans aide par une personne âgée, non technicienne | rien (la session enregistre son rapport) | « lance mamy » |
| [`ally`](agents/ally.md) | Audit d'accessibilité, WCAG 2.2 AA : clavier, lecteur d'écran, contraste mesuré, cibles tactiles, zoom, animations | rien (la session enregistre son rapport) | « lance Ally » |
| [`stryker`](agents/stryker.md) | Test de mutation (mutmut pour Python, StrykerJS pour JS/TS) ; sur demande, ajoute des tests pour tuer les mutants survivants | son rapport ; des tests uniquement en mode « kill » | « lance Stryker » |
| [`heimdall`](agents/heimdall.md) | Gardien de la livraison : niveau de qualité, tests, artefacts de build, versions, traductions, glossaire, fuites vers un dépôt public, hygiène, documentation | son rapport uniquement | « lance le gardien » |
| [`iris`](agents/iris.md) | CSS et cohérence visuelle : motifs dupliqués stylés différemment, jetons de style, collisions de noms de classes, thèmes clair et sombre, contraste ; sur demande, harmonise les styles dans une feuille de style partagée | son rapport ; des styles uniquement en mode « harmonise » | « lance Iris » |
| [`c3po`](agents/c3po.md) | Vérificateur de traductions : chaque texte existe dans chaque langue (clés, variables, pluriels, fichiers) et signifie la même chose partout (sens, nombres, négations, termes du glossaire) ; sur demande, corrige uniquement les fichiers de traduction | son rapport ; des fichiers de traduction uniquement en mode correction | « lance C-3PO » |
| [`parker`](agents/parker.md) | Photographe : démarre l'instance de développement avec des données de test stables et photographie chaque écran (petit et grand, clair et sombre) | des captures d'écran et un index | « lance le photographe » |

Tous travaillent **uniquement sur demande**, ne touchent jamais à un environnement réel, ne font jamais de commit ni de
push, et ne modifient jamais le code de production (`stryker` n'ajoute des tests que lorsqu'on lui demande de tuer des mutants).

## Rapports

Chaque agent qui produit un résultat l'écrit dans `reports/ai/<agent>/YYYY-MM-DD-<subject>.md` dans le projet
examiné (`reports/` est ignoré par Git). `parker` écrit ses captures d'écran et un `INDEX.md` dans
`reports/ai/parker/YYYY-MM-DD/`. `john-117` n'écrit aucun fichier par défaut : il consolide les rapports des agents
qu'il lance. Les rapports ont la même forme (périmètre et contexte, constats classés, top 5, ce qui est clair et ne
doit pas être touché) et la même échelle de gravité : **Bloquant / Majeur / Mineur**. `mamy` et `ally` fournissent une liste de contrôle
que la session peut transformer en ticket de suivi ; `heimdall` rend un verdict (PEUT PASSER / PEUT PASSER AVEC RÉSERVES /
NE PASSE PAS) et une liste de contrôle.

## Règles communes

Tous les agents suivent le même bloc de « règles de base » : uniquement sur demande, lire d'abord les règles du projet, ne jamais toucher à un
environnement réel (ni commit, ni push, ni déploiement), des preuves plutôt que des promesses (« non vérifié » quand ce n'est pas prouvé), aucun secret
ni donnée personnelle, ne jamais mentionner un assistant ou une IA dans ce qui est copié dans un dépôt, et la même échelle
de gravité. Les rapports sont en anglais, sauf si la configuration ou la demande de l'utilisateur indique autre chose. Chaque fichier commence par un commentaire `# Name:` qui explique le nom.

## Format d'un fichier d'agent

```markdown
---
name: my-agent                  # required, same as the file name, lowercase and dashes
description: When to use it.    # required, one line
tools: read, search, execute, edit, mcp:chrome-devtools/take_screenshot   # or * for all tools
model: inherit                  # optional
---
Instructions of the agent (Markdown).
```

Outils génériques : `read`, `search`, `edit`, `execute`, `web`, `agent`, `mcp:<server>/<tool>`. Faites-les correspondre à votre
assistant lorsque vous installez un agent (par exemple `read` → `Read`, `search` → `Grep`/`Glob`, `execute` →
`Bash`, `edit` → `Edit`/`Write`, `mcp:chrome-devtools/take_screenshot` → `mcp__chrome-devtools__take_screenshot`).

## Installation

**Un simple lien symbolique ne suffit pas.** La ligne `tools` de ces fichiers utilise des noms neutres (`read`, `search`,
`execute`, `edit`, `mcp:<server>/<tool>`), alors que chaque assistant attend les siens (par exemple `Read`, `Grep`/`Glob`,
`Bash`, `Edit`/`Write`, `mcp__<server>__<tool>` pour Claude Code), et que certains utilisent un tout autre format de fichier (TOML
pour Codex). Un agent lié tel quel se retrouverait avec des noms d'outils que l'assistant ne connaît pas.

Un script testé est fourni dans la compétence `install-agents`. Trois façons d'installer :

1. **Lancer la compétence ou le script (recommandé).** Clonez ce dépôt (par exemple dans `~/Dev/AI`), puis lancez
   `python3 skills/install-agents/scripts/sync_agents.py --dry-run` pour voir le plan, et la même commande sans
   `--dry-run` pour l'appliquer (ou demandez à votre assistant de lancer la compétence `install-agents`). Redémarrez l'assistant.
   Relancez-le après chaque `git pull` ; les fichiers générés ne sont jamais modifiés à la main.
2. **Écrire votre propre script** pour un assistant que le script ne couvre pas (section suivante).
3. **À la main, un seul agent.** Copiez le fichier dans le dossier d'agents de votre assistant (pour Claude Code :
   `~/.claude/agents/<name>.md` pour tous vos projets, ou `.claude/agents/<name>.md` dans un seul projet) et
   traduisez vous-même la ligne `tools` avec le tableau ci-dessous.

Les agents pilotés par navigateur (`mamy`, `ally`, `parker`, et `iris` pour les vérifications visuelles) ont besoin du serveur MCP Chrome DevTools.

## Script de synchronisation

### Utiliser le script fourni

```
python3 skills/install-agents/scripts/sync_agents.py --dry-run      # show the plan, write nothing
python3 skills/install-agents/scripts/sync_agents.py                # install into the detected assistants
```

Options : `--assistants claude,codex,gemini,copilot` (par défaut : ceux dont le dossier de configuration existe), `--scope user|project`
avec `--project PATH` pour les fichiers au niveau du projet, `--only NAME[,NAME]`, `--source PATH`. Il est idempotent, ne supprime
jamais rien et sauvegarde tout fichier étranger dans `~/.config/ai-agents/_backup/`. Tests :
`python3 -m unittest discover skills/install-agents/tests -v`. Les formats de fichier de Codex, Gemini CLI et Copilot CLI
suivent le tableau ci-dessous ; vérifiez-les dans la documentation actuelle de chaque assistant.

### Écrire votre propre script de synchronisation (pour un assistant non couvert)

Ce que le script doit faire :
- **Lire** chaque `agents/*.md` : un en-tête entre deux lignes `---` (`name`, `description`, `tools`, `model` facultatif ;
  ignorer les lignes de commentaire `# ...`) et le corps (les instructions). Vérifier que `name` est égal au nom du fichier.
- **Traduire la ligne `tools`** avec ce tableau (nom neutre à gauche) :

| Neutre | Claude Code | Gemini CLI | Copilot CLI |
|---|---|---|---|
| `read` | `Read` | `read_file`, `read_many_files` | `read` |
| `search` | `Grep`, `Glob` | `grep_search`, `glob`, `list_directory` | `search` |
| `edit` | `Edit`, `Write` | `replace`, `write_file` | `edit` |
| `execute` | `Bash` | `run_shell_command` | `execute` |
| `web` | `WebFetch`, `WebSearch` | `web_fetch`, `google_web_search` | `web` |
| `agent` | `Agent` | aucun | `agent` |
| `mcp:<server>/<tool>` | `mcp__<server>__<tool>` | `mcp_<server>_<tool>` | `<server>/<tool>` |
| `*` (ou pas de ligne) | pas de champ `tools` = tous les outils | idem | idem |

- **Écrire un fichier par assistant**, dans le format et à l'endroit qu'il attend (vérifiez la documentation de chaque assistant ;
  c'est ce qu'écrit le script de l'auteur) :

| Assistant | Fichier | Champs d'en-tête | Corps |
|---|---|---|---|
| Claude Code | `~/.claude/agents/<name>.md` (ou `<project>/.claude/agents/`) | `name`, `description`, `tools` (séparés par des virgules), `model` | les instructions |
| Codex | `~/.codex/agents/<name_with_underscores>.toml` | `name`, `description` | `developer_instructions = """..."""` |
| Gemini CLI | `~/.gemini/agents/<name>.md` | `name`, `description`, `kind: local`, `tools` (liste) | les instructions |
| Copilot CLI | `~/.copilot/agents/<name>.agent.md` (ou `<project>/.github/agents/`) | `name`, `description`, `tools` (tableau), `user-invocable: true` | les instructions |

- **Être prudent** : idempotent (ne pas réécrire un fichier identique), une option `--dry-run`, une ligne « fichier généré, ne pas modifier »
  en tête de chaque sortie, une sauvegarde de tout fichier existant qu'il n'a pas généré avant de le remplacer, et ne jamais
  rien supprimer. Analyser l'en-tête avec un vrai langage (bibliothèque standard de Python), pas avec `sed`.

#### Un prompt à donner à votre assistant IA

> Écris un script Python 3 (bibliothèque standard uniquement) nommé `sync_agents.py` qui génère les fichiers d'agents de mes
> assistants IA à partir des sources Markdown de `~/Dev/AI/agents/*.md`.
>
> Format source : un en-tête entre deux lignes `---` avec `name` (doit être égal au nom du fichier sans `.md`),
> `description`, `tools` (noms neutres séparés par des virgules : `read`, `search`, `edit`, `execute`, `web`, `agent`,
> `mcp:<server>/<tool>`, ou `*` pour tous les outils ; absent signifie tous) et `model` facultatif ; les lignes commençant par `#` dans
> l'en-tête sont des commentaires à ignorer ; puis le corps avec les instructions.
>
> Génère, pour chaque agent, les fichiers des assistants que j'utilise : [LISTE : Claude Code, Codex, Gemini CLI, Copilot CLI].
> Traduis la ligne `tools` avec cette correspondance : [COLLE LES DEUX TABLEAUX DE LA SECTION « Ce que le script doit faire » DU
> README]. Suis la documentation actuelle de chaque assistant pour le format et l'emplacement des fichiers. Chaque fichier généré
> commence par un commentaire indiquant qu'il est généré par ce script et ne doit pas être modifié.
>
> Exigences : `--dry-run` montre ce qui serait créé ou mis à jour sans rien écrire ; idempotent (un fichier identique est
> laissé tel quel, signalé comme inchangé) ; si un fichier cible existe, diffère et n'a pas été généré par ce script, copie-le dans
> `~/.config/ai-agents/_backup/<timestamp>/` avant de le remplacer ; ne supprime jamais rien ; échoue avec un message clair
> sur un en-tête absent ou invalide ; affiche une ligne par fichier écrit. Accepte aussi `--project <path>` pour écrire les
> copies au niveau du projet (`.claude/agents/`, `.github/agents/`) dans un dépôt.
>
> Puis montre-moi comment le lancer et comment le tester sur un agent.

Lancez d'abord le script avec `--dry-run`, lisez ce qu'il écrirait, puis lancez-le pour de bon et redémarrez l'assistant.

## Compétences

| Compétence | Usage |
|---|---|
| [`install-agents`](skills/install-agents/SKILL.md) | Installer ou mettre à jour les agents de ce dépôt dans Claude Code, Codex, Gemini CLI ou Copilot CLI, avec une simulation, des sauvegardes et des tests |
| [`argus`](agents/argus.md) | Triage des tickets : complétude d’un bug ou d’une demande, recherche de doublon et de fonction existante, étiquettes de triage et un commentaire | étiquettes et commentaires sur les tickets | « fais le triage des tickets » |
| [`chrono`](agents/chrono.md) | Complexité algorithmique et performance : chemins chauds, analyse en O(), mesures avant et après, la plus petite conception plus rapide | aucune par défaut ; code seulement sur demande expresse | « cherche ce qui est lent et pourquoi » |

## Contribuer

Le propriétaire est le seul à pousser directement sur `main`. Les demandes passent par des issues et les modifications par des pull requests : voir [CONTRIBUTING.md](CONTRIBUTING.md) et [SECURITY.md](SECURITY.md).

## Licence

[MIT](LICENSE)
