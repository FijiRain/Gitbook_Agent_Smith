# Acquisition de Capacités (Capability Acquisition)

> *Comment l’architecture d’un agent permet-elle de mieux exploiter les capacités d’un LLM ?*

Il faut voir l’architecture (mémoire, planning, outils) comme le **« hardware »** d’un agent. Mais ce matériel ne suffit pas toujours à accomplir des tâches complexes : l’agent peut manquer d’expérience, de *skills* ou de méthodologie pour une tâche spécifique. Ces éléments méthodologiques correspondent au côté **« software »** de l’agent.

Dans la littérature, cette dimension est appelée **CA (*Capability Acquisition*)** et se décline selon deux grandes approches : **avec fine-tuning** ou **sans fine-tuning**.

---

## 1. Acquisition de capacités par Fine-Tuning

C’est l'approche directe : elle consiste à affiner (*fine-tune*) les poids du modèle en l'entraînant sur des datasets spécifiquement construits pour la tâche visée. Ces datasets peuvent provenir de trois sources :
* **Sources humaines** : par exemple en alignant un LLM avec des démonstrations d'experts ou des préférences humaines (RLHF / DPO).
* **Sources générées par un LLM** : un modèle génère une multitude de scénarios d’utilisation d’outils (ex: [*Toolformer*](https://arxiv.org/pdf/2302.04761)), filtre ceux qui aboutissent au bon résultat pour en former un dataset, et ce dataset sert à améliorer la maîtrise des outils par le modèle principal.
* **Sources tirées du monde réel** : historiques d'interactions d'utilisateurs, issues GitHub et Pull Requests associées, etc.

---

## 2. Acquisition de capacités sans Fine-Tuning (L'approche d'Agent Smith)

Dans le Machine Learning traditionnel, les capacités d'un modèle étaient principalement acquises lors de l'entraînement sur un dataset, puis figées dans les paramètres du réseau. Avec les LLMs modernes (et d'autant plus dans *Agent Smith*, où nous utilisons des modèles via API sans modifier leurs poids), on améliore les capacités de l'agent **sans fine-tuning** grâce à deux leviers majeurs :

### A. Le Prompt Engineering
Il s'agit de structurer et formuler les instructions verbales du prompt système pour « débloquer » certaines capacités latentes du LLM ou cadrer son comportement, notamment en lui fournissant :
* une méthodologie explicite de résolution de problème ;
* la documentation claire des outils disponibles ;
* des exemples concrets (*few-shot examples*) illustrant le format de raisonnement attendu (*« make no mistakes » lol*).

### B. Le Mechanism Engineering
Plus subtil que le simple prompt statique, le *mechanism engineering* consiste à concevoir la boucle d'exécution pour que l'agent s’auto-observe, apprenne de ses erreurs et ajuste sa trajectoire :

* **Trial-and-Error (*Tentative et erreur*)** : l’agent effectue d’abord une action (par exemple modifier une fonction et lancer `run_tests()`), puis reçoit une critique ou un signal d'erreur objectif (le *traceback* pytest ou un message du validateur syntaxique de la sandbox). Si l’action n'est pas satisfaisante, l’agent incorpore ce retour critique dans son raisonnement suivant (voir aussi l'article [*Reflexion*, Shinn et al., 2023](https://arxiv.org/abs/2303.11366)).
* **Crowd-Sourcing (*Consensus multi-agents*)** : différents agents proposent des réponses ou analyses séparées à un même problème, puis confrontent leurs pistes jusqu’à converger vers un consensus. La capacité globale est améliorée par l'incorporation de la critique croisée des autres agents.
* **Experience Accumulation (*Accumulation d'expérience*)** : au départ, l'agent ne sait pas résoudre la tâche directement ; il explore donc le dépôt ou teste des hypothèses. Une fois qu’il réussit une sous-étape, les découvertes et actions fructueuses sont conservées dans sa mémoire, lui permettant de capitaliser dessus pour la suite.
* **Self-Driven Evolution (*Évolution auto-guidée*)** : méthode permettant aux agents de s’auto-améliorer progressivement au travers d'objectifs intermédiaires auto-guidés et de boucles de rétroaction (*feedback loops*).
