# Introduction générale

J’espère que vous allez lire cette documentation, la comprendre et l’apprécier. Le but est d’organiser les connaissances autour du sujet **Agent Smith** et de les assimiler afin de dégager une organisation et des directives claires pour parachever ce projet particulièrement exigeant. Il ne s’agit pas d’un guide « clé en main » pour réussir le projet, mais d'une sélection structurée de ressources et de concepts pour avancer de manière autonome.

J’aime beaucoup faire de la recherche bibliographique et structurer une réflexion ; aussi, dans un style d’écriture qui m’est propre, je tenterai d’expliquer point par point les différents concepts en annotant le texte comme si mon objectif était de me l’expliquer à moi-même.

Je ne peux pas assurer que le contenu sera 100 % pertinent uniquement pour le projet *Agent Smith* : j’ai surtout essayé de reconstituer un **état de l’art** et une **vulgarisation globale** des thèmes abordés pour éviter que ce projet ne soit une marche à l’aveugle vers l’inconnu. Certaines notions seront peut-être un peu plus détaillées que ce qu'exige strictement l’application pratique.

---

## Pourquoi ce projet est-il particulier ?

Avec toutes les lectures que j’ai pu faire, un constat revient systématiquement : **la discipline des Agents IA est extrêmement récente** (beaucoup de sources de référence datent de l’année dernière au maximum, voire de quelques mois seulement !). Par conséquent, il n’existe pas encore de « manuel classique » pour concevoir des agents IA au sens où le demande *Agent Smith*.

C’est sans doute ce qui constitue une grande partie de la difficulté du projet, et la raison pour laquelle 42 est aussi permissif avec l’usage de l’IA sur cet exercice. Mais bon, *vibe-coder* un outil qui sert lui-même à *vibe-coder* n’a aucun intérêt pédagogique à mon sens : être confronté à la difficulté et apprendre à la surmonter est beaucoup plus gratifiant que de réduire ses efforts juste pour valider le projet.

{% hint style="info" %}
**Rappel important sur la philosophie de ce document**

Compte tenu de la densité du projet, ne soyez pas étonnés si cette documentation est conséquente. J'ai fait mon possible pour rester concis et lisible, mais certains passages restent denses et techniques.

**Toutes les parties ne sont pas des « guides pour réussir Agent Smith étape par étape », mais un condensé d’informations théoriques centralisées ayant pour fil conducteur le sujet Agent Smith.** Le but est de démystifier des concepts abstraits pour vous donner les clés de compréhension, pas de mâcher le travail d'implémentation. Certaines approches décrites ici sont vraies dans la littérature scientifique sans être forcément requises telles quelles dans le projet.
{% endhint %}

J’invite le lecteur à creuser davantage les sujets qui l'intéressent : toute la bibliographie (y compris des ressources complémentaires que je n’ai pas directement citées dans le corps du texte) se trouve dans la section [Bibliographie Commentée](04-annexes/bibliographie.md).

---

## 🗺️ Comment naviguer dans ce GitBook ?

| Partie | Contenu | Pages clés |
| :--- | :--- | :--- |
| **I. Fondations Théoriques** | Comprendre ce qui distingue un LLM classique d'un système agentique (mémoire, planification, action, outils). | [Qu'est-ce qu'un Agent IA ?](01-fondations/qu-est-ce-qu-un-agent-ia.md)<br>[Architecture : Mémoire, Planification & Action](01-fondations/architecture-memoire-planning.md)<br>[Acquisition de Capacités](01-fondations/capability-acquisition.md)<br>[Tool Use & ACI](01-fondations/tool-use-et-aci.md) |
| **II. Piliers Techniques** | Les briques concrètes imposées par le sujet : le protocole MCP, la sécurité, les API LLM et l'isolation de la Sandbox Python. | [MCP : Le Model Context Protocol](02-ecosysteme-technique/model-context-protocol.md)<br>[Sécurité Agentique & Risques MCP](02-ecosysteme-technique/securite-mcp.md)<br>[Fournisseurs d'API LLM & Clés](02-ecosysteme-technique/fournisseurs-api-et-cles.md)<br>[La Sandbox](02-ecosysteme-technique/sandbox-et-isolation.md) |
| **III. Benchmarks & Projet** | Les épreuves de validation (MBPP, SWE-bench), la répartition du travail à 3 et la check-list de rendu. | [Les Benchmarks : MBPP & SWE-bench](03-benchmarks-et-organisation/benchmarks-mbpp-et-swebench.md)<br>[Organisation & Feuille de Route](03-benchmarks-et-organisation/organisation-et-feuille-de-route.md)<br>[Check-list Exhaustive du Sujet](03-benchmarks-et-organisation/checklist-du-sujet.md) |
| **IV. Annexes** | Définitions rapides, articles de recherche, vidéos et exemples de prompts système officiels. | [Glossaire](04-annexes/glossaire.md)<br>[Bibliographie Commentée](04-annexes/bibliographie.md)<br>[Prompts & Schémas d'Outils Utiles](04-annexes/prompts-utiles.md) |

---

*Rédigé par **@tmoga***
