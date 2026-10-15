# Prompts & Schémas d'Outils Utiles

Voici différents prompts et schémas d'outils réels qui pourront vous servir d’inspiration directe pour concevoir le prompt système et les docstrings MCP d'**Agent Smith**.

---

## 1. Prompt officiel utilisé par Anthropic (Claude 3.5 Sonnet) sur SWE-bench

Ce prompt (publié par Anthropic dans [*Raising the bar on SWE-bench Verified*](https://www.anthropic.com/news/swe-bench-sonnet) — *il faisait « que » 49 % à l'époque en 2024, le gros nul*) est un modèle de clarté : il donne l'emplacement du dépôt, interdit de modifier les tests existants, et dicte une **méthodologie de débogage en 5 étapes** (explorer → reproduire le bug → éditer → relancer le test de reproduction → vérifier les cas limites) :

```xml
<uploaded_files>
{location}
</uploaded_files>
I've uploaded a python code repository in the directory {location} (not in /tmp/inputs).
Consider the following PR description:

<pr_description>
{pr_description}
</pr_description>

Can you help me implement the necessary changes to the repository so that the
requirements specified in the <pr_description> are met?
I've already taken care of all changes to any of the test files described in the
<pr_description>. This means you DON'T have to modify the testing logic or any of the tests
in any way!

Your task is to make the minimal changes to non-tests files in the {location} directory to
ensure the <pr_description> is satisfied.

Follow these steps to resolve the issue:
1. As a first step, it might be a good idea to explore the repo to familiarize yourself with its
structure.
2. Create a script to reproduce the error and execute it with `python <filename.py>` using the
BashTool, to confirm the error
3. Edit the sourcecode of the repo to resolve the issue
4. Rerun your reproduce script and confirm that the error is fixed!
5. Think about edgecases and make sure your fix handles them as well

Your thinking should be thorough and so it's fine if it's very long.
```

{% hint style="success" %}
**Comment l'adapter pour Agent Smith ?**
Dans *Agent Smith* (Section V.1), votre prompt système doit reprendre cette même méthodologie en 5 étapes, mais en l'adaptant au paradigme **Code Agent (`Thought → Code → Observation`)** :
1. Inclure le **Manuel de la Sandbox** généré dynamiquement à partir des schémas du serveur MCP connecté (`read_file`, `edit_file`, `search_code`, `run_tests`, `get_patch`, etc.).
2. Fournir un exemple concret de cycle `Thought:` suivi d'un bloc ```` ```python ... ``` <end_code> ```` et de l'`Observation:` résultante.
3. Rappeler explicitement au modèle comment terminer la tâche :
   * Sur **MBPP** : appeler `final_answer(solution_code_string)` après avoir vérifié que `run_tests(...)` passe.
   * Sur **SWE-bench** : appeler `final_answer(get_patch())` uniquement après avoir vérifié que les tests passent.
{% endhint %}

---

## 2. Exemple de description d'outil d'édition (`str_replace_editor`)

Pour concevoir la description de vos outils MCP (et tout particulièrement `read_file`, `list_files` et `edit_file(filepath, old_str, new_str)` imposés en Section V.5.1), observez la précision des consignes données au LLM dans l'outil `str_replace_editor` d'Anthropic (principe d'**ACI** et de **poka-yoke**) :

```json
{
  "name": "str_replace_editor",
  "description": "Custom editing tool for viewing, creating and editing files\n* State is persistent across command calls and discussions with the user\n* If `path` is a file, `view` displays the result of applying `cat -n`. If `path` is a directory, `view` lists non-hidden files and directories up to 2 levels deep\n* The `create` command cannot be used if the specified `path` already exists as a file\n* If a `command` generates a long output, it will be truncated and marked with `<response clipped>`\n* The `undo_edit` command will revert the last edit made to the file at `path`\n\nNotes for using the `str_replace` command:\n* The `old_str` parameter should match EXACTLY one or more consecutive lines from the original file. Be mindful of whitespaces!\n* If the `old_str` parameter is not unique in the file, the replacement will not be performed. Make sure to include enough context in `old_str` to make it unique\n* The `new_str` parameter should contain the edited lines that should replace the `old_str`"
}
```

### Ce qu'il faut retenir de ce schéma pour votre `edit_file(filepath, old_str, new_str)` :
* Avertir explicitement le LLM que `old_str` doit correspondre **exactement** (espaces et indentation compris) au contenu lu via `read_file`.
* Refuser le remplacement (et renvoyer un message d'erreur clair au LLM !) si `old_str` n'apparaît pas dans le fichier **ou** s'il apparaît plusieurs fois (en invitant alors le modèle à inclure 2 ou 3 lignes de contexte supplémentaires autour de la ligne à modifier pour rendre `old_str` unique).
* Vérifier la syntaxe Python du fichier modifié (`ast.parse`) juste après l'édition afin d'avertir immédiatement le LLM s'il a introduit une `SyntaxError` ou une `IndentationError` !

---

## 3. Code complet du projet Agent Smith

{% hint style="danger" %}
**mdr t’as cru**

*mettre code du project ici svp (Shawn)*
{% endhint %}
