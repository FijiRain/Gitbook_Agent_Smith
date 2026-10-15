# Organisation d'Équipe & Feuille de Route

Dans cette partie, nous voyons comment décomposer concrètement le projet **Agent Smith** pour travailler efficacement en groupe de 3 :
1. **Deux approches de répartition du travail** (Division Architecturale vs Division par Sprints Agiles) ;
2. **Une feuille de route (*Roadmap*) séquentielle** pour savoir exactement dans quel ordre attaquer les briques techniques sans se disperser.

---

## 1. Plans de répartition du travail à 3

Voici deux manières complémentaires d'organiser une équipe de trois développeurs, affinées à partir des exigences exactes du sujet et de cette documentation.

{% tabs %}
{% tab title="Approche 1 — Division Architecturale (Par Couche)" %}
### Approche 1 : Division Structurale Architecturale (Horizontal)

Chaque membre devient le référent d'un des trois grands sous-systèmes du sujet :

| Responsabilité Architecturale | Périmètre Technique Détaillé | Livrables Logiciels Exigés | Points de Focalisation (Focus) |
| :--- | :--- | :--- | :--- |
| **Développeur 1 : L'Orchestrateur** *(Le Cerveau)* | Interfaçage avec les API LLM (OpenRouter, Groq, etc.), gestion du parsing multi-format, rotation des clés API, et orchestration de la boucle `Thought → Code → Observation`. | Logique de retries, extracteur de code (Python, XML, Hermes, ReAct), suivi des limites globales (tokens/itérations/temps), modèles `StepMetrics` & `SolutionOutput`, ingénierie du prompt système et `stop_sequences`. | Fiabilité des requêtes réseau, robustesse du parsing face aux hallucinations du LLM, et optimisation de la fenêtre de contexte. |
| **Développeur 2 : La Sandbox** *(Le Gardien)* | Création de l'environnement Python restreint (CLI REPL + moteur d'exécution). Implémentation du contrôle des ressources, analyse statique de l'AST, et isolation des chemins d'accès. | Validateur d'AST, restriction de `__builtins__`, blocage réseau, gestionnaire de timeout d'exécution (`30s` par bloc) et limite RAM (`512 Mo`), client MCP interne, génération dynamique du manuel, et injection de `final_answer`. | Sécurité absolue de l'exécution (passage de `exam_sandbox.sh`), interception chirurgicale des erreurs (*Tracebacks* explicites), et gestion propre des processus fils. |
| **Développeur 3 : Écosystème MCP & Outils** *(Les Bras)* | Développement des serveurs MCP (transports `stdio` et `HTTP`) et implémentation des outils de manipulation de fichiers, recherche de code et exécution de tests. | Fichiers `mcp_tools_mbpp.py` et `mcp_tools_swebench.py` à la racine du dépôt. Orchestration et nettoyage des conteneurs Docker pour l'épreuve SWE-bench. | Maîtrise du protocole MCP / JSON-RPC, implémentation des 9 outils obligatoires au format exact (`cat -n`, `grep`), et génération de patchs Git conformes. |

#### Détail des rôles dans cette configuration

* **Le Développeur 1 (L'Orchestrateur)** se concentre sur l'intelligence et la coordination. Il gère les requêtes vers les modèles distants, met en place la rotation intelligente des clés API pour contourner les *rate limits* des fournisseurs gratuits, et construit l'extracteur capable de convertir des sorties hétérogènes (blocs Markdown Python, balises XML `<invoke>`, formats JSON/Hermes `<tool_call>`, format ReAct) en appels Python pour la sandbox. Il veille également au respect des limites globales de la tâche (`120s` / `6k` tokens sur MBPP ; `900s` / `300k` tokens sur SWE-bench) et produit le `SolutionOutput`.
* **Le Développeur 2 (La Sandbox)** opère comme l'ingénieur sécurité du projet. Son objectif est de construire un environnement d'exécution Python standard (sans `RestrictedPython`) hermétique à toute évasion logicielle. Il implémente les limites d'exécution par bloc (`max_execution_time_seconds = 30s`, `max_memory_mb = 512`) via des processus isolés et signaux système (`SIGTERM`, `SIGKILL`), valide l'AST avant exécution, et nettoie méticuleusement l'espace de noms global (`__builtins__`).
* **Le Développeur 3 (MCP & Outils)** agit comme l'expert du monde extérieur et des protocoles. Il maîtrise le cycle de vie MCP (`stdio` et `Streamable HTTP`) et développe les serveurs autonomes capables d'interagir avec les fichiers ou les conteneurs Docker exigés par SWE-bench (`${TESTBED_PATH}`). Il traduit les spécifications des outils en fonctions fiables dotées d'excellentes *docstrings* (ACI).
{% endtab %}

{% tab title="Approche 2 — Division par Sprints Agiles (Vertical)" %}
### Approche 2 : Division par Itérations Agiles (Fonctionnalités de bout en bout)

Plutôt que de séparer le travail uniquement par couche technique isolée (au risque que chacun code dans son coin pendant 3 semaines sans pouvoir rien tester ensemble), l'équipe avance par **paliers fonctionnels verticaux** de complexité croissante :

| Itération (Sprint) | Objectif Stratégique Global | Tâches Assignées (D1, D2, D3) | Livrables du Sprint |
| :--- | :--- | :--- | :--- |
| **Sprint 1 : Le MVP MBPP** *(Semaines 1–2)* | Validation du concept central et premier succès de bout en bout sur une tâche algorithmique simple. | **D1 :** Boucle itérative basique et prompt initial.<br>**D2 :** Mécanisme `exec()` isolé basique, `final_answer()` et capture `stdout`/`stderr`.<br>**D3 :** Premier outil `run_tests` pour MBPP. | Un agent capable de lire un fichier JSON MBPP, de générer du code, de lancer les tests et de produire une première solution valide. |
| **Sprint 2 : Sécurisation & Standardisation MCP** *(Semaines 3–4)* | Respect strict des normes du sujet, implémentation de la sécurité Sandbox et du protocole MCP complet. | **D1 :** Implémentation complète de `StepMetrics`, `SolutionOutput`, rotation de clés et parseur multi-format.<br>**D2 :** Analyse AST, blocage des builtins/imports/chemins/réseau, timeouts, REPL `uv run sandbox`.<br>**D3 :** Serveur `mcp_tools_mbpp.py` (`stdio` & `HTTP`) et découverte dynamique. | Une architecture résiliente et sécurisée qui valide `exam_sandbox.sh` et `exam_mbpp.sh` (≥ 4/5). |
| **Sprint 3 : L'Épreuve SWE-bench & Benchmark** *(Semaines 5–7)* | Confrontation aux dépôts réels dans Docker, navigation dans le code et campagne de benchmark multi-modèles. | **D1 :** Optimisation du prompt SWE-bench, gestion des contextes longs (`300k` tokens), campagne de tests des 5 modèles et rédaction de `BENCHMARK_REPORT.md`.<br>**D2 :** Intégration de la Sandbox avec le workflow Docker et feedbacks d'erreur avancés.<br>**D3 :** Serveur `mcp_tools_swebench.py` avec les 9 outils obligatoires et gestion propre du cycle de vie des conteneurs Docker. | Résolution autonome d'Issues GitHub réelles (≥ 2/3 sur `exam_swebench.sh`), génération de patchs Git propres et rapport d'évaluation complet. |

#### Pourquoi cette approche fonctionne très bien ?
Elle garantit que l'équipe dispose **en permanence d'une version fonctionnelle et testable de bout en bout**, ce qui élimine l'effet tunnel. Dès la fin du Sprint 1, l'agent tourne sur MBPP ; le Sprint 2 verrouille la sécurité et le standard MCP ; le Sprint 3 concentre toute l'énergie sur le défi majeur du SWE-bench et l'étude comparative des modèles.
{% endtab %}
{% endtabs %}

---

## 2. Feuille de route séquentielle (*Roadmap*)

Voici l'ordre logique recommandé pour construire le projet brique par brique sans se disperser :

### Étape 0 — Fondations et Sanctuarisation de l'Infrastructure
Avant d'écrire la moindre ligne de code impliquant un appel réseau vers un LLM, l'infrastructure logicielle doit être posée proprement :
1. Initialiser le dépôt avec **`uv`** et **Python 3.10** (exigences strictes du chapitre IV.1).
2. Configurer le fichier **`.env`** pour le chargement sécurisé des clés API et verrouiller immédiatement le **`.gitignore`** (pour exclure `.env`, caches, outputs générés et images).
3. Modéliser les structures de données en implémentant tous les **modèles Pydantic** exigés par le sujet : `SandboxConfig`, `MBPPTaskInput`, `SWEBenchTaskInput`, `StepMetrics` et `SolutionOutput`.

### Étape 1 — Prototype de la Sandbox Locale
Développer le cœur de la sandbox :
1. Une fonction isolée capable de recevoir une chaîne de code Python arbitraire, de vérifier son arbre syntaxique (`ast`), et de l'exécuter dans un namespace restreint (`__builtins__` filtrés, imports contrôlés par `authorized_imports`, accès fichiers restreint à `allowed_directories`, réseau coupé).
2. Capturer proprement les flux `stdout` et `stderr`, gérer le timeout (`30s`) et la limite RAM (`512 Mo`) via un processus isolé, et restituer des messages d'erreur explicites sans jamais faire crasher le processus parent.
3. Injecter la primitive native `final_answer()` et proposer le mode interactif REPL (`uv run sandbox`).

### Étape 2 — L'Orchestrateur et la Boucle Agentique
Déplacer l'effort vers le cerveau de l'agent :
1. Construire l'**extracteur de code multi-format** capable de traverser la réponse brute du LLM (`llm_output`) pour en extraire le code Python ou convertir les formats alternatifs (XML `<invoke>`, JSON/Hermes `<tool_call>`, ReAct `Action:`) en appels de fonctions Python.
2. Implémenter le **client LLM multi-provider** avec rotation fluide des clés API, *exponential backoff* sur les erreurs `429`/`5xx`, paramétrage des `stop_sequences`, et comptage précis des tokens et de la latence (`StepMetrics`).
3. Coupler cette boucle d'appels API avec la Sandbox pour valider les premières séquences autonomes `Thought → Code → Observation`.

### Étape 3 — Le Point de Bascule : Intégration du Protocole MCP
Connecter la Sandbox au monde extérieur :
1. Intégrer dans la Sandbox un **client MCP** supportant à la fois les transports **`stdio`** et **`Streamable HTTP`**.
2. Créer le premier serveur MCP (`mcp_tools_mbpp.py`) exposant `run_tests` pour valider le **handshake MCP** complet.
3. Exploiter la découverte dynamique (`tools/list`, `resources/list`, `prompts/list`) pour générer automatiquement le **Manuel de la Sandbox** injecté dans le prompt système du modèle, et exposer dynamiquement chaque outil MCP comme une fonction Python appelable dans le namespace de la sandbox.

### Étape 4 — L'Épreuve Terminale : SWE-bench, Docker & Benchmark Report
Attaquer la partie la plus ambitieuse du sujet :
1. Développer `mcp_tools_swebench.py` avec les **9 outils obligatoires** (`read_file`, `edit_file`, `list_files`, `search_code`, `search_function_or_class_definition_in_code`, `find_references`, `run_tests`, `get_patch`, `run_command`) en soignant les formats de retour (`cat -n`, `grep`) et les messages d'erreur pour guider le LLM.
2. Intégrer la gestion des conteneurs Docker SWE-bench (montage ou accès à `${TESTBED_PATH}`, génération du diff via `git -c core.fileMode=false diff`, et **nettoyage systématique du conteneur** après exécution).
3. Tester et itérer d'abord sur les tâches recommandées par le sujet (`sympy__sympy-14711`, `sympy__sympy-13480`, `pydata__xarray-4629`) en inspectant les 3 à 5 premières itérations.
4. Lancer la campagne d'évaluation croisée d'au moins **5 modèles sur au moins 3 tâches SWE-bench**, conserver les fichiers `solution.json` correspondants et rédiger le **`BENCHMARK_REPORT.md`** ainsi que le **`README.md`** final en anglais.
