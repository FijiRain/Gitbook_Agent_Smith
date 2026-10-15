# Les Benchmarks : MBPP & SWE-bench

C’est bien beau de concevoir un super agent sur le papier, mais il faut toujours garder en tête la finalité du projet : **répondre aux benchmarks officiels dans les limites imparties** pour certifier que notre agent est fiable, autonome et produit des solutions valides.

Lors de la correction (`exam_mbpp.sh` et `exam_swebench.sh`), le script d'évaluation tire au sort des tâches issues de ces deux benchmarks et votre agent doit les résoudre sans intervention humaine. Mais sous quel format se présentent-ils, et en quoi consistent-ils exactement ?

---

## Comparatif des limites et seuils de validation (Chap. VI)

| Critère | Benchmark 1 : **MBPP** | Benchmark 2 : **SWE-bench** |
| :--- | :--- | :--- |
| **Nature des tâches** | Problèmes algorithmiques Python courts (1 fonction) | Résolution de vraies Issues GitHub dans de gros dépôts |
| **Tâches tirées en évaluation** | 5 tâches aléatoires | 3 tâches aléatoires |
| **Seuil de réussite exigé** | **4 / 5** minimum | **2 / 3** minimum |
| **Itérations maximales** | **10** itérations | **30** itérations |
| **Budget Input Tokens (cumulé)** | **6 000** tokens | **300 000** tokens |
| **Budget Output Tokens (cumulé)** | **1 500** tokens | **10 000** tokens |
| **Timeout global par tâche** | **120 secondes** (2 min) | **900 secondes** (15 min) |
| **Format passé à `final_answer()`** | Code Python de la fonction solution | Patch Git (`git -c core.fileMode=false diff`) |

---

## 1. MBPP (*Mostly Basic Python Problems*)

Il s’agit du benchmark le plus accessible des deux (et celui par lequel il faut absolument commencer votre développement). Chaque tâche est fournie sous la forme d'un fichier JSON décrivant un problème algorithmique simple en Python.

### Exemple d'entrée dans le dataset MBPP original

```json
{
  "text": "Write a python function to identify non-prime numbers.",
  "code": "import math\r\ndef is_not_prime(n):\r\n    result = False\r\n    for i in range(2, int(math.sqrt(n)) + 1):\r\n        if n % i == 0:\r\n            result = True\r\n    return result",
  "task_id": 3,
  "test_setup_code": "",
  "test_list": [
    "assert is_not_prime(2) == False",
    "assert is_not_prime(10) == True",
    "assert is_not_prime(35) == True"
  ],
  "challenge_test_list": []
}
```

### Structure Pydantic dans Agent Smith (`MBPPTaskInput`)

Dans le dataset MBPP académique officiel, le champ `"code"` contient la solution humaine de référence. Bien évidemment, **il ne faut jamais exposer de solution de référence à l’agent**, sinon le benchmark n’a aucun sens (et vaudra un `0` pour triche) ! D'ailleurs, le dump généré par `moulinette_eval dump mbpp` utilise le modèle `MBPPTaskInput` qui fournit uniquement :
* `task_id` : l'identifiant de la tâche ;
* `task_definition` : l'énoncé du problème ;
* `function_definition` : la signature attendue de la fonction Python ;
* `test_imports` et `test_list` : les assertions permettant de tester la fonction.

L’agent génère alors son propre code, l'éprouve dans la sandbox à l'aide de l'outil MCP `run_tests`, corrige sa logique si une assertion échoue, puis appelle `final_answer(your_solution_code)`.

---

## 2. SWE-bench (*Software Engineering Benchmark*)

Ce benchmark est aujourd'hui **la référence mondiale majeure** pour mesurer les compétences d'ingénierie logicielle des LLMs et des agents de code. Sur le [leaderboard officiel de SWE-bench](https://www.swebench.com/verified.html), tous les grands laboratoires (Anthropic, OpenAI, Google, DeepSeek, Qwen) comparent leurs systèmes selon leur pourcentage de résolution sur **SWE-bench Verified**.

C’est véritablement l'épreuve de SWE-bench qui fait qu'**Agent Smith** n’est pas un simple projet d'école de plus, mais marque un vrai palier vers un **« projet d'ingénierie de niveau industriel »**... *sauf si vous le vibe-codez de A à Z sans rien comprendre — auquel cas je n’ai aucun respect pour vous, et la lecture de ce document ne vous servira sûrement à rien puisque vous l’aurez oublié dans une semaine.*

### Pourquoi ce benchmark change-t-il tout ?

Contrairement au MBPP où l'agent écrit une fonction isolée de 15 lignes, SWE-bench place l’agent face à des **dépôts de production réels** (`sympy`, `django`, `scikit-learn`, `xarray`, `pytest`, etc.) contenant des centaines de milliers de lignes de code. Il ne suffit plus de savoir générer du Python : l'agent doit **naviguer dans une architecture inconnue, localiser l'origine d'un bug, éditer chirurgicalement un ou plusieurs fichiers et vérifier l'absence de régression**.

En résumé, on fournit à l'agent un vrai dépôt GitHub et le texte d'une vraie issue GitHub :

$$\text{Issue GitHub} \longrightarrow \text{Exploration du repo} \longrightarrow \text{Compréhension du bug} \longrightarrow \text{Édition (`edit\_file`)} \longrightarrow \text{Tests (`run\_tests`)} \longrightarrow \text{Observation \& Correction} \longrightarrow \text{`final\_answer(get\_patch())`}$$

### Anatomie d'une tâche SWE-bench

Chaque instance de SWE-bench a été construite à partir de l'historique réel d'un dépôt open-source :

$$\text{Tâche SWE-bench} = \text{Issue GitHub} + \text{État du dépôt avant la PR} + \text{Environnement Docker reproductible} + \text{Script d'évaluation (`eval\_script`)}$$

Pour chaque tâche, le benchmark fournit une **image Docker** contenant le dépôt exactement dans l'état où il se trouvait juste avant le commit de correction (*pre-PR*), avec toutes ses dépendances scientifiques déjà installées.

Lors de la validation, la moulinette applique le patch Git produit par votre agent et lance la suite de tests :
* Les tests **FAIL_TO_PASS** (qui échouaient avant le patch et reproduisaient le bug de l'issue) doivent désormais **passer**.
* Les tests **PASS_TO_PASS** (les tests existants du dépôt) doivent **continuer de passer** (aucune régression introduite).

> **Point clé :** La réussite ne repose pas sur l'avis subjectif d'un LLM jugeant que « le code a l'air propre », mais sur la preuve expérimentale reproductible (dans Docker) que le patch résout le problème sans rien casser.

{% hint style="info" %}
**Repère de contexte 2026 pour nuancer votre `BENCHMARK_REPORT.md` :**
Début 2026, plusieurs laboratoires (dont OpenAI) ont documenté des biais de contamination des données d'entraînement et des tests parfois trop rigides sur certaines tâches difficiles de *SWE-bench Verified*, ce qui a motivé l'émergence de variantes comme *SWE-bench Pro*.
Cela ne change rien à ce qui nous est demandé techniquement, mais c'est une **excellente remarque méthodologique à inclure dans votre `BENCHMARK_REPORT.md`** : expliquer avec recul que le score brut sur *SWE-bench Verified* est un indicateur comparatif précieux, mais pas une vérité absolue.
{% endhint %}
