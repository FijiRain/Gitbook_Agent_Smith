# Bibliographie Commentée

Cette bibliographie rassemble toutes les sources utilisées pour rédiger cette documentation (articles de recherche, blogs d'ingénierie, documentations officielles, vidéos et dépôts open-source), enrichies de notes de lecture pour savoir quoi prioriser.

**Légende :** 📄 Article de recherche · 📝 Article d'ingénierie · 🎥 Vidéo · 📚 Documentation officielle · 💻 Code / Repo · 📖 Livre

---

## 1. Agents IA & LLMs (Fondations)

* 📝 [**Building Effective AI Agents — Anthropic Engineering (déc. 2024)**](https://www.anthropic.com/engineering/building-effective-agents)
  * **Priorité : ⭐⭐⭐⭐⭐** — Le texte de référence sur ce qui distingue un *workflow* (chemin d'exécution prédéfini) d'un véritable *agent* (le LLM dirige dynamiquement ses actions), l'*Augmented LLM*, et l'annexe sur la conception d'outils (*Agent-Computer Interfaces*).
* 📄 [**A Survey on Large Language Model based Autonomous Agents — Wang et al. (2024, *Frontiers of Computer Science*)**](https://link.springer.com/article/10.1007/s11704-024-40231-1)
  * **Priorité : ⭐⭐⭐⭐⭐** — Le grand panorama académique qui structure notre Chapitre I : construction de l'agent, mémoire (*short-term / long-term*, formule de scoring récence/pertinence/importance), planification (*Single-Path* vs *Multi-Path*, avec/sans feedback), module d'action et *Capability Acquisition*.
* 📄 [**ReAct: Synergizing Reasoning and Acting in Language Models — Yao et al. (ICLR 2023)**](https://arxiv.org/pdf/2210.03629)
  * **Priorité : ⭐⭐⭐⭐⭐** — L'article fondateur de la boucle `Thought → Action → Observation` qui ancre le raisonnement du modèle dans la vérité terrain de l'environnement.
* 🎥 [**Python: Create a ReAct Agent from Scratch — YouTube**](https://www.youtube.com/watch?v=hKVhRA9kfeM)
  * Implémentation minimale du pattern ReAct en Python pur, sans LangChain ni aucun framework — idéal pour visualiser le squelette de la boucle avant d'y brancher la Sandbox et MCP.
* 📝 [**A Practical Guide to Building Agents — OpenAI**](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
  * Guide pratique sur l'architecture de la boucle d'exécution (*run loop*), les appels d'outils, les conditions d'arrêt et les *guardrails*.
* 🎥 [**Agent Harness Explained in 8min — YouTube**](https://www.youtube.com/watch?v=1a1VXDdIyrk)
  * Courte vidéo très claire pour comprendre la notion de *harness* (le harnais logiciel qui encadre, orchestre et évalue un agent).
* 📄 [**Generative Agents: Interactive Simulacra of Human Behavior — Park et al. (UIST 2023)**](https://arxiv.org/pdf/2304.03442)
  * L'article de Stanford d'où est tiré le mécanisme de mémoire réflective et l'arbre de réflexion de *Klaus Mueller* présenté dans la section Mémoire.
* 📄 [**Reflexion: Language Agents with Verbal Reinforcement Learning — Shinn et al. (NeurIPS 2023)**](https://arxiv.org/abs/2303.11366)
  * Complète ReAct en montrant comment faire verbaliser à l'agent une auto-critique après un échec de test pour améliorer l'itération suivante sans fine-tuning.

---

## 2. Code Agents (Agir par du code Python plutôt que par du JSON)

* 📄 [**Executable Code Actions Elicit Better LLM Agents (CodeAct) — Wang et al. (ICML 2024)**](https://arxiv.org/abs/2402.01030)
  * **Priorité : ⭐⭐⭐⭐⭐** — L'article scientifique qui justifie le choix central d'*Agent Smith* : remplacer les appels d'outils JSON par du code Python exécutable (`Thought → Code → Observation`), permettant d'utiliser des variables persistantes, des boucles et des conditions.
* 📝 [**Introducing smolagents: simple agents that write actions in code — Hugging Face (déc. 2024)**](https://huggingface.co/blog/smolagents)
  * La meilleure vulgarisation de l'approche *CodeAct* : explique pourquoi les LLMs sont bien meilleurs pour écrire du code Python que pour formater du JSON.
* 💻 [**agents.py — huggingface/smolagents (GitHub)**](https://github.com/huggingface/smolagents/blob/main/src/smolagents/agents.py)
  * **Attention : interdiction formelle d'importer `smolagents` dans votre projet**, mais lire son code source est ultra-instructif pour voir à quoi ressemble une boucle `CodeAgent` propre avec extraction de code et sandbox en moins de 1 000 lignes.

---

## 3. Model Context Protocol (MCP)

* 🎥 [**MCP expliqué à fond : le "port USB-C" qui branche l'IA sur le monde réel — YouTube**](https://www.youtube.com/watch?v=v2FigSIz_pI)
  * Excellente vidéo en français qui illustre le problème $M \times N$, les rôles Host / Client / Serveur, les 3 primitives (`tools`, `resources`, `prompts`), JSON-RPC et le handshake.
* 📚 [**What is the Model Context Protocol (MCP)? — Documentation Officielle**](https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro)
  * Le point d'entrée officiel du protocole (architecture, cycle de vie, transports `stdio` et `Streamable HTTP`).
* 📚 [**Build an MCP Server — Documentation Officielle**](https://modelcontextprotocol.io/docs/2026-07-28/develop/build-server)
  * Guide officiel pour construire un serveur MCP en Python (`mcp_tools_mbpp.py` et `mcp_tools_swebench.py`).
* 🎥 [**How Model Context Protocol (MCP) Actually Works — YouTube**](https://www.youtube.com/watch?v=cGuyrANVi4A)
  * Analyse technique sous le capot des échanges de messages entre client et serveur MCP.
* 📚 [**FastMCP — Documentation**](https://gofastmcp.com/getting-started/welcome)
  * Framework haut-niveau permettant de comprendre comment décorer simplement des fonctions Python pour les exposer en MCP.
* 📝 [**Code Execution with MCP: Building More Efficient Agents — Anthropic Engineering (nov. 2025)**](https://www.anthropic.com/engineering/code-execution-with-mcp)
  * Fait exactement le pont entre les sections 2 et 3 : comment un *Code Agent* exécute du code dans une sandbox qui appelle des outils exposés via MCP.

---

## 4. Tool Use, Prompts & Context Engineering

* 📝 [**Writing Effective Tools for AI Agents — Using AI Agents — Anthropic Engineering (sept. 2025)**](https://www.anthropic.com/engineering/writing-tools-for-agents)
  * Guide concret à lire absolument avant de coder les 9 outils obligatoires du SWE-bench : nommage sans ambiguïté, gestion de la troncature, retours riches en signal.
* 📄 [**SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering — Yang et al. (NeurIPS 2024)**](https://arxiv.org/abs/2405.15793)
  * **Priorité : ⭐⭐⭐⭐⭐** — L'article qui a inspiré la liste d'outils obligatoires d'*Agent Smith* (`read_file` avec numéros de lignes, `search_code`, `edit_file`).
* 📄 [**Toolformer: Language Models Can Teach Themselves to Use Tools — Schick et al. (NeurIPS 2023)**](https://arxiv.org/pdf/2302.04761)
  * Un des textes historiques sur l'apprentissage de l'usage d'outils par les LLMs.
* 🎥 [**JSON-RPC Client — YouTube**](https://www.youtube.com/watch?v=FmeyLUKHI4Q&t=22s)
  * *(Note : pire vidéo du monde, mais permet de visualiser rapidement la structure d'une requête JSON-RPC).*
* 📝 [**Effective Context Engineering for AI Agents — Anthropic Engineering (sept. 2025)**](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
  * Très utile pour gérer le budget serré de tokens (`6 000` input tokens sur MBPP, `300 000` sur SWE-bench).
* 💻 [**Hermes-Function-Calling — NousResearch (GitHub)**](https://github.com/NousResearch/Hermes-Function-Calling)
  * Référence pour le format `<tool_call>{"name": ..., "arguments": ...}</tool_call>` cité dans la section V.1 du sujet.

---

## 5. Benchmarks : MBPP & SWE-bench

* 📄 [**SWE-bench: Can Language Models Resolve Real-World GitHub Issues? — Jimenez et al. (ICLR 2024)**](https://arxiv.org/abs/2310.06770) ([PDF OpenReview](https://openreview.net/pdf?id=VTF8yNQM66))
  * **Priorité : ⭐⭐⭐⭐⭐** — Le papier fondateur du benchmark SWE-bench : construction des instances, conteneurs Docker, tests `FAIL_TO_PASS` et `PASS_TO_PASS`.
* 📝 [**Raising the Bar on SWE-bench Verified with Claude 3.5 Sonnet — Anthropic (oct. 2024)**](https://www.anthropic.com/news/swe-bench-sonnet)
  * **Priorité : ⭐⭐⭐⭐⭐** — Étude de cas incontournable dont nous avons extrait les prompts dans la page [Prompts & Schémas d'Outils Utiles](prompts-utiles.md). Montre qu'une architecture simple avec un bon prompt et des outils bien pensés suffit à atteindre l'état de l'art.
* 📚 [**SWE-bench Official Leaderboards**](https://www.swebench.com/)
  * Utile pour choisir vos 5 modèles à comparer dans `BENCHMARK_REPORT.md` et observer comment les meilleurs systèmes structurent leurs agents.
* 📄 [**Program Synthesis with Large Language Models (MBPP) — Austin et al. (2021)**](https://arxiv.org/abs/2108.07732)
  * Le papier d'origine du benchmark MBPP.

---

## 6. Sandbox, Sécurité Python & Isolation

* 📚 [**Secure Code Execution — Hugging Face (`smolagents` documentation)**](https://huggingface.co/docs/smolagents/tutorials/secure_code_execution)
  * Explique les risques d'exécution de code généré par un LLM, l'analyse par AST et les limites de l'isolation Python (source du schéma sur le spectre de sécurité en page Sandbox).
* 💻 [**openedx/codejail: Secure Code Execution — GitHub**](https://github.com/openedx/codejail)
  * Exemple réel d'une sandbox Python open-source en production : isolation par sous-processus et limites de ressources (`resource`).
* 📝 [**Sandboxing Untrusted Python Code: Secure Execution Strategies — UBOS (janv. 2026)**](https://ubos.tech/news/sandboxing-untrusted-python-code-secure-execution-strategies-and-ubos-solutions/)
  * Panorama comparatif des approches d'isolation (processus, conteneurs, micro-VM, restrictions langage) — idéal pour alimenter la section *Sandbox design* de votre `README.md`.
* 📝 [**Introducing LangSmith Sandboxes: Secure Code Execution for Agents — LangChain**](https://www.langchain.com/blog/introducing-langsmith-sandboxes-secure-code-execution-for-agents#why-sandboxes)
  * Vue industrielle du problème d'exécution de code non fiable (source du second schéma d'isolation de la section Sandbox).
* 📝 [**Notes on Sandboxing Untrusted Code — Why Python Can't Be Easily Sandboxed (GitHub Gist)**](https://gist.github.com/mavdol/2c68acb408686f1e038bf89e5705b28c)
  * Résume très bien les vecteurs d'évasion classiques en Python (`__subclasses__`, introspection de frames, builtins) à bloquer dans votre validateur AST et votre namespace.
* 📚 [**OWASP Top 10 for Large Language Model Applications**](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
  * Référence sécurité sur la *Prompt Injection* (LLM01), l'*Insecure Output Handling* (LLM02) et l'*Excessive Agency* (LLM08).
* 📚 [**Documentation Standard Python : `ast`, `resource`, `subprocess`, `multiprocessing`**](https://docs.python.org/3.10/library/ast.html)
  * Les briques de la bibliothèque standard Python 3.10 autorisées pour construire votre sandbox.

---

## 7. Outils de Développement & Recul Général

* 📚 [**Documentation officielle de `uv` (Astral)**](https://docs.astral.sh/uv/) & [**Documentation officielle de `Pydantic` v2**](https://docs.pydantic.dev/latest/)
* 📖 [**AI Engineering: Building Applications with Foundation Models — Chip Huyen (O'Reilly, 2025)**](https://github.com/chiphuyen/aie-book)
  * L'ouvrage de référence le plus complet sur l'ingénierie des systèmes LLM, l'évaluation et les benchmarks.
