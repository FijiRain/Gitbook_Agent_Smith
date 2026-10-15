# Check-list Exhaustive du Sujet

{% hint style="info" %}
**Comment utiliser cette page :**
Cette liste centralise **l'intégralité des exigences du cahier des charges** d'*Agent Smith*. Ne la lisez pas d'une traite pour vous faire peur : utilisez-la comme une **check-list de vérification** au fil du développement et juste avant la soutenance pour être certains de n'avoir rien oublié !
{% endhint %}

---

## 1. Architecture Globale & Contraintes Techniques

* [ ] Utiliser exclusivement **Python 3.10** et le gestionnaire de paquets **`uv`**.
* [ ] **Interdiction totale** d'utiliser des bibliothèques d'orchestration d'agents existantes (`llama-index`, `langgraph`, `smolagents`, `crewai`, `autogen`, etc.).
* [ ] Gérer toutes les erreurs gracieusement (aucun crash non rattrapé n'est toléré pendant l'évaluation).
* [ ] Placer les fichiers des serveurs MCP (`mcp_tools_mbpp.py` et `mcp_tools_swebench.py`) **à la racine** du dépôt Git.
* [ ] Faire en sorte que les outils MCP fonctionnent de manière **indépendante** de la boucle de l'agent (ils sont testés individuellement à l'évaluation).

---

## 2. La Boucle Agentique & L'Orchestrateur

* [ ] Implémenter la boucle de base : **`Thought → Code → Observation`**.
* [ ] Rendre `max_iterations` configurable en paramètre de la boucle agentique.
* [ ] Configurer des `stop_sequences` (ex: `<end_code>`, `</tool_call>`) lors des appels LLM pour éviter que le modèle n'hallucine ses propres observations.
* [ ] Implémenter une couche d'extraction de code capable de gérer différents formats de sortie LLM et de convertir les formats non-Python en appels de fonctions Python pour la sandbox :
  * [ ] Blocs de code Python Markdown (```` ```python ... ``` <end_code> ````)
  * [ ] Appels d'outils XML style Anthropic (`<invoke name="..."><parameter>...</parameter></invoke>`)
  * [ ] Appels d'outils JSON / Hermes (`<tool_call>{"name": "...", "arguments": {...}}</tool_call>`)
  * [ ] Format ReAct classique (`Action: tool_name` / `Action Input: {...}`)
* [ ] Fournir un **feedback explicite** au LLM dans l'observation pour chacune de ces situations (ne jamais laisser le modèle deviner après un échec silencieux) :
  * [ ] Aucun bloc de code valide n'a été trouvé dans la réponse du modèle
  * [ ] Un bloc de code était malformé mais a été interprété quand même (expliquer comment)
  * [ ] L'exécution a atteint le timeout et la sortie est partielle
  * [ ] La sortie d'un outil a été tronquée à cause des limites de taille
  * [ ] Une édition a introduit une erreur de syntaxe ou une violation de lint
* [ ] Implémenter le suivi complet d'utilisation (*usage tracking*) : tokens d'entrée/sortie, retries, latence (`request_time_ms`), nombre total de requêtes.
* [ ] Produire des fichiers JSON de sortie strictement conformes aux modèles Pydantic `StepMetrics` et `SolutionOutput`, incluant notamment `system_prompt`, `api_url`, `model_name`, `llm_output`, `sandbox_input`, `sandbox_output` et `retries`.

---

## 3. La Sandbox

* [ ] Créer l'interface CLI interactive REPL (`uv run sandbox`) qui exécute chaque entrée dans le namespace de la sandbox et s'arrête proprement avec `exit` ou `Ctrl+D` (`EOF`).
* [ ] Supporter les arguments CLI de configuration :
  * [ ] `uv run sandbox`
  * [ ] `uv run sandbox sandbox_template.json`
  * [ ] `uv run sandbox --mcp-stdio "python mcp_tools_mbpp.py" sandbox_template.json`
  * [ ] `uv run sandbox --mcp-server <URL>`
  * [ ] `uv run sandbox --mcp-stdio "python mcp_tools_swebench.py" sandbox_template.json`
* [ ] Supporter les deux transports MCP : **`stdio`** et **`Streamable HTTP`**.
* [ ] Exposer les 3 primitives MCP : **tools**, **resources** et **prompts**.
* [ ] Découvrir dynamiquement les outils de n'importe quel serveur MCP connecté (le système sera testé avec un serveur MCP inconnu) et les exposer comme des fonctions Python appelables dans la sandbox.
* [ ] Générer dynamiquement le **« Manuel de la Sandbox »** à partir des schémas d'outils du serveur MCP connecté pour l'injecter dans le prompt système du LLM.
* [ ] Implémenter la fonction native **`final_answer()`** directement dans le namespace de la sandbox (toujours présente, **ce n'est pas un outil MCP**) pour capturer la réponse et stopper la boucle agentique.
* [ ] Propager correctement les exceptions de contrôle de flux (`KeyboardInterrupt`, `SystemExit`) vers l'orchestrateur sans les étouffer.
* [ ] Rendre la sandbox configurable via le modèle Pydantic `SandboxConfig` et des fichiers JSON (`authorized_imports`, `allowed_directories`, `max_execution_time_seconds`, `max_memory_mb`).
* [ ] **Sécurité (en pur Python standard, sans `RestrictedPython`)** — réussir tous les tests de `exam_sandbox.sh` :
  * [ ] Liste blanche d'imports (`authorized_imports`)
  * [ ] Restriction d'accès au système de fichiers (`allowed_directories`)
  * [ ] Blocage total des connexions réseau entrantes et sortantes
  * [ ] Timeout d'exécution (`max_execution_time_seconds`, appliqué uniquement au code exécuté dans la sandbox)
  * [ ] Limite d'utilisation de la RAM (`max_memory_mb`)
  * [ ] Blocage / remplacement des `__builtins__` dangereux pour empêcher toute escalade de privilèges

---

## 4. Implémentations Spécifiques aux Benchmarks (MBPP & SWE-bench)

### Pour MBPP
* [ ] Créer l'interface CLI : `uv run python -m agent_mbpp --task-file ... --output ... --model-name ... --provider-url ...`
* [ ] Implémenter le modèle Pydantic `MBPPTaskInput`.
* [ ] Implémenter `mcp_tools_mbpp.py` avec l'outil obligatoire `run_tests` (et tout outil additionnel utile).
* [ ] Respecter les limites strictes : **10 itérations max**, **6 000 input tokens max**, **1 500 output tokens max**, **120s timeout**.
* [ ] Atteindre le seuil de réussite de **4 / 5** sur `exam_mbpp.sh`.

### Pour SWE-bench
* [ ] Créer l'interface CLI : `uv run python -m agent_swebench --task-file ... --output ... --model-name ... --provider-url ...`
* [ ] Implémenter le modèle Pydantic `SWEBenchTaskInput`.
* [ ] Implémenter `mcp_tools_swebench.py` avec les **9 outils obligatoires** :
  * [ ] `read_file(filepath, start_line, end_line)` (format `cat -n` : `<line_number>: <line_content>`)
  * [ ] `edit_file(filepath, old_str, new_str)`
  * [ ] `list_files(directory, pattern)`
  * [ ] `search_code(pattern, file_pattern)` (format `/absolute/path.py:<line_number> <line_content>`)
  * [ ] `search_function_or_class_definition_in_code(name)` (même format que `search_code`)
  * [ ] `find_references(name, filepath, line)` (même format que `search_code`)
  * [ ] `run_tests()`
  * [ ] `get_patch()`
  * [ ] `run_command(command, workdir)` (retourne `stdout`, `stderr`, et `exit_code`)
* [ ] Supporter la variable d'environnement `TESTBED_PATH` pour localiser le dépôt.
* [ ] Générer et soumettre un patch Git valide via `git -c core.fileMode=false diff` passé à `final_answer(get_patch())`.
* [ ] Nettoyer systématiquement le conteneur Docker après l'exécution du programme.
* [ ] Respecter les limites strictes : **30 itérations max**, **300 000 input tokens max**, **10 000 output tokens max**, **900s timeout**.
* [ ] Atteindre le seuil de réussite de **2 / 3** sur `exam_swebench.sh`.

---

## 5. Fournisseurs d'API LLM & Gestion des Clés

* [ ] Utiliser **exclusivement des tiers gratuits (*free tiers*)** d'API (aucun plan payant ni compte avec facturation activée).
* [ ] Supporter **plusieurs fournisseurs LLM** et plusieurs modèles avec une abstraction propre permettant de changer de fournisseur sans refactoring.
* [ ] Implémenter la **rotation multi-tokens** : supporter plusieurs clés API par fournisseur pour gérer les *rate limits* et l'épuisement de quotas.
* [ ] Charger toutes les clés API exclusivement via les variables d'environnement / fichier `.env` passé à l'évaluation (**zéro clé API en dur dans le code**).

---

## 6. Rapports (`BENCHMARK_REPORT.md` & `README.md`) et Rendu Git

* [ ] Rédiger `BENCHMARK_REPORT.md` à la racine du dépôt, comparant **au moins 5 modèles** sur un même ensemble d'**au moins 3 tâches SWE-bench**, avec :
  * [ ] **1. Setup** : modèles/fournisseurs comparés, tâches choisies et justification.
  * [ ] **2. Results table** : Pass/Fail, itérations, input tokens, output tokens, temps total pour chaque couple `modèle × tâche`.
  * [ ] **3. Provider reliability** : temps de réponse moyen par requête, nombre de retries, disponibilité globale.
  * [ ] **4. Intermediary metrics** (au moins 2 parmi : étape de première lecture/édition du bon fichier, étape où les échecs de tests diminuent pour la première fois, nombre d'itérations entre le premier passage des tests et `final_answer`).
  * [ ] **5. Ablation study** : au moins une comparaison *avant / après* d'une modification de l'agent (prompt, outils ou paramètres) sur les mêmes tâches avec le même modèle.
  * [ ] **6. Conclusions** : modèle(s) retenu(s) pour le pipeline final et modèles écartés, justifiés par les données.
  * [ ] Inclure dans le dépôt les fichiers `solution.json` ayant servi de base au rapport.
* [ ] Rédiger `README.md` **en anglais** à la racine du dépôt :
  * [ ] Toute première ligne en italique : `*This project has been created as part of the 42 curriculum by <login1>[, <login2>[, <login3>[...]]].*`
  * [ ] Sections obligatoires : *Description*, *Instructions*, *Resources* (incluant la déclaration transparente de l'usage de l'IA), *System architecture*, *Agent loop explanation*, *Sandbox design*, *Tool implementation details*, *Benchmark results and analysis*.
* [ ] Fournir les fichiers de configuration de la sandbox et des modèles.
* [ ] Vérifier que le dépôt Git ne contient **aucune image Docker, aucun poids de modèle lourd, aucune clé API, ni fichiers d'outputs parasites**.
