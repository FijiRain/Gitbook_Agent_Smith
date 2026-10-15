# Tool Use & Interfaces Agent-Machine (ACI)

## Les 3 catégories d'outils

Les outils (*tools*) permettent d’étendre les capacités d'un agent en lui donnant accès à des APIs externes ou au système lui-même. Chaque outil doit posséder une **description standardisée** (nom, docstring, typage des paramètres) qui facilite les échanges `Agent → Outil`. On distingue généralement trois catégories d’outils :

1. **Les outils de type « Data » (Perception / Lecture)**
   Ils permettent à l'agent de récupérer du contexte ou une information nécessaire pour la suite de son raisonnement.
   * *Exemples dans Agent Smith :* `read_file`, `list_files`, `search_code`, `search_function_or_class_definition_in_code`, `find_references`, `get_patch`.
2. **Les outils de type « Action » (Modification / Exécution)**
   Ils permettent à l'agent d’interagir avec le système et de produire des effets de bord (*side effects*).
   * *Exemples dans Agent Smith :* `edit_file`, `run_tests`, `run_command`.
3. **Les outils de type « Orchestration »**
   Il s'agit en réalité d’autres sous-agents appelés comme des outils par l'agent principal pour déléguer une sous-tâche spécialisée.

{% hint style="danger" %}
**Règle d'or :** La qualité d’un agent dépend énormément de sa capacité à utiliser pertinemment ses outils, et cette capacité repose directement sur une **description d'outil de haute qualité** et un **format d'entrée/sortie adapté au LLM**.
{% endhint %}

---

## Concevoir des outils pour un LLM : l'annexe d'Anthropic (ACI)

{% hint style="info" %}
**Pourquoi cette section est cruciale pour Agent Smith :**
Voici une traduction et adaptation de l'annexe technique d’Anthropic issue de [*Building Effective Agents*](https://www.anthropic.com/engineering/building-effective-agents) (complétée par [*Writing effective tools for AI agents*](https://www.anthropic.com/engineering/writing-tools-for-agents) et l'article [*SWE-agent*](https://arxiv.org/abs/2405.15793)). Elle répond directement aux problèmes que vous rencontrerez sur le benchmark **SWE-bench** (notamment l'édition de fichiers et la génération de diffs).
{% endhint %}

Quel que soit le système agentique que vous construisez, les outils constituent une partie critique de votre agent. Les définitions et spécifications de vos outils doivent faire l'objet **d'autant d'attention en matière de *prompt engineering* que votre prompt système général**.

### Pourquoi certains formats font échouer les LLMs ?

Il existe souvent plusieurs façons de spécifier une même action :
* Pour modifier un fichier, on peut demander au modèle d'écrire un **diff unifié**, ou de remplacer un bloc via **`edit_file(filepath, old_str, new_str)`**, ou encore de réécrire l'intégralité du fichier.
* Pour une sortie structurée, on peut demander du code dans un bloc **Markdown** (```` ```python ````) ou à l'intérieur d'un objet **JSON**.

En ingénierie logicielle classique, ces différences sont purement cosmétiques et peuvent être converties de l'une à l'autre sans perte d'information. **Cependant, certains formats sont beaucoup plus difficiles à produire sans erreur pour un LLM que d'autres :**
* **Écrire un *diff* brut** oblige le modèle à calculer et annoncer dans l'en-tête du *hunk* (`@@ -l,s +l,s @@`) le nombre exact de lignes modifiées **avant même** d'avoir généré le nouveau code !
* **Écrire du code à l'intérieur de JSON** (par rapport à un bloc de code Markdown) oblige le modèle à échapper manuellement chaque saut de ligne (`\n`), chaque tabulation et chaque guillemet (`\"`), ce qui multiplie les erreurs de syntaxe.

### Les 3 recommandations pour le format des outils

1. **Donnez au modèle suffisamment de tokens pour « réfléchir »** (*Thought*) avant de générer son appel d'outil et de se retrouver coincé dans un format rigide.
2. **Gardez le format proche de ce que le modèle a naturellement rencontré à l'entraînement** sur Internet (les blocs de code Markdown Python sont beaucoup plus naturels pour un LLM que du code sérialisé en JSON).
3. **Éliminez toute « surcharge » de formatage** (*formatting overhead*), comme devoir compter précisément des numéros de lignes ou échapper manuellement de longues chaînes de caractères.

---

## De l'IHM à l'ACI (*Agent-Computer Interface*)

Une excellente règle générale consiste à réfléchir à l'effort que nous consacrons aux **interfaces homme-machine (IHM / HCI)** et à investir autant de soin dans la création de bonnes **interfaces agent-machine (*ACI, Agent-Computer Interfaces*)** :

* **Mettez-vous à la place du modèle :** Est-il évident de savoir comment utiliser cet outil à partir de sa description et de ses paramètres, ou faudrait-il réfléchir longuement pour le comprendre ? Si c'est flou pour un humain, ce le sera aussi pour le LLM. Une bonne définition d'outil contient des exemples d'utilisation, les cas limites (*edge cases*), le format exact des entrées/sorties et des frontières claires entre les différents outils.
* **Soignez le nommage et les docstrings :** Comment modifier les noms ou les descriptions des paramètres pour rendre leur rôle limpide ? Rédigez la spécification de chaque outil comme une excellente *docstring* destinée à un développeur junior de votre équipe.
* **Testez et observez les erreurs réelles du modèle :** Inspectez les traces d'exécution (`steps` dans `solution.json`) sur vos premières tâches pour observer où le modèle se trompe d'arguments, puis itérez sur vos outils.
* **Appliquez le principe du *poka-yoke* (détrompeur) à vos outils :** Concevez les arguments et les vérifications de vos outils de manière à rendre les erreurs impossibles ou immédiatement explicites.

{% hint style="success" %}
**Qu'est-ce que le principe du *poka-yoke* ?**
C’est le principe du port HDMI ou USB-C : le design physique de la prise permet à lui seul de savoir instantanément comment elle se branche et empêche de la brancher à l'envers.

**Exemple concret d'Anthropic sur SWE-bench :**
*« Lors de la construction de notre agent pour SWE-bench, nous avons en réalité consacré davantage de temps à optimiser nos outils qu'à optimiser le prompt général. Par exemple, nous avons constaté que le modèle commettait des erreurs avec les outils utilisant des chemins de fichiers relatifs une fois que l'agent avait changé de répertoire courant. Pour résoudre ce problème, nous avons modifié l'outil afin qu'il exige systématiquement des **chemins de fichiers absolus** (ex: `/testbed/src/module.py`) — et nous avons constaté que le modèle utilisait alors cette méthode sans faire d'erreur. »*
{% endhint %}

---

## Visualisation d'une boucle ReAct avec Tool Use

Voici une visualisation animée (issue de la documentation Hugging Face `smolagents`) illustrant ce qui se passe étape par étape dans une boucle agentique de type ReAct utilisant du code Python pour appeler des outils :

![Fonctionnement d'un agent CodeAct/ReAct sur une question simple](https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/transformers/Agent_ManimCE.gif)
*Source : [Animation Agent_ManimCE.gif — Hugging Face](https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/transformers/Agent_ManimCE.gif)*

### Squelette minimal d'une boucle ReAct en Python

Pour démystifier la boucle `Thought → Action → Observation` avant d'y greffer la Sandbox et le serveur MCP, voici à quoi ressemble le cœur d'un agent ReAct minimaliste en Python pur :

```python
import os
from dotenv import load_dotenv
from groq import Groq

load_dotenv()
client = Groq(api_key=os.getenv("GROQ_API_KEY"))

class Agent:
    def __init__(self, client: Groq, system_prompt: str) -> None:
        self.client = client
        self.messages = [{"role": "system", "content": system_prompt}]

    def __call__(self, user_message: str) -> str:
        if user_message:
            self.messages.append({"role": "user", "content": user_message})
        result = self.execute()
        self.messages.append({"role": "assistant", "content": result})
        return result

    def execute(self) -> str:
        completion = self.client.chat.completions.create(
            messages=self.messages,
            model="openai/gpt-oss-120b",
            stop=["PAUSE", "<end_code>"],  # Empêche le LLM d'halluciner l'Observation !
        )
        return completion.choices[0].message.content
```

{% hint style="warning" %}
**Point de vigilance crucial pour Agent Smith — Les `stop_sequences` :**
Comme le souligne le sujet (Section V.6), que se passe-t-il si le LLM continue de générer du texte après avoir écrit son bloc de code ou son appel d'outil ? **Il va inventer lui-même une fausse `Observation:` !**
Pour l'en empêcher, utilisez le paramètre `stop` (ou `stop_sequences`) de l'API LLM (par exemple `stop=["<end_code>", "</tool_call>"]`) afin de couper la génération dès la fin du bloc de code et laisser la vraie Sandbox exécuter l'action.
{% endhint %}
