# Remerciements & Transparence IA

## Remerciements

J'aimerais remercier mes collègues **@chehir**, **@anfarenc**, **@shapetit** et **@alsimoni** pour la relecture de la première version de ce document et leurs retours constructifs.

---

## Déclaration sur l'usage de l'IA dans ce document

Voici le détail transparent de l'usage qui a été fait de l'IA pour l'élaboration de cette documentation :

### Travail personnel
* **Recherche documentaire et assimilation théorique** : lecture complète, analyse et synthèse des articles de recherche fondateurs (ReAct, CodeAct, SWE-agent, SWE-bench, Generative Agents...) ainsi que des documentations officielles (Anthropic, Model Context Protocol, OpenAI...) et études de toutes les ressources citées dans la bibliographie.
* **Rédaction et style de vulgarisation** : rédaction des explications, des schémas d'exemples, des réflexions pédagogiques, des notes de mise en garde et des analogies avec le projet Agent Smith.
* **Conception architecturale** : identification des points critiques du sujet (cloisonnement de la Sandbox, double gestion des timeouts, interface ACI, rotation multi-providers). Création du découpage par parties et analyse structurelle de ce qu'elles devaient contenir.

### Ce pour quoi l'IA a été utilisée comme outil d'assistance
L'IA (modèles Claude et Gemini) a été sollicitée comme accélérateur méthodologique et technique pour :
1. **Compléter la recherche bibliographique** : assistance pour retrouver les liens officiels, les identifiants arXiv et vérifier l'exactitude des sources citées.
2. **Structurer et affiner le plan d'équipe** : formalisation sous forme de tableaux comparatifs des deux approches d'organisation (découpage structurel horizontal et "sprints" verticaux) à partir d'idées et d'objectifs déjà réfléchis.
3. **Élaborer la feuille de route (*Roadmap*)** : mise en forme séquentielle des jalons du plan choisi pour guider l'avancement pas à pas.
4. **Mise en page et conversion GitBook** : conversion du manuscrit PDF d'origine en Markdown enrichi (blocs d'alertes `{% hint %}`, onglets comparatifs `{% tabs %}`, formatage des formules mathématiques KaTeX pour un centrage propre, et structuration du sommaire `SUMMARY.md`).
5. **Déploiement et intégration Git Sync** : assistance technique pour configurer le schéma `gitbook-docs.yaml` et résoudre les erreurs de synchronisation.

{% hint style="info" %}
**Garantie pédagogique :** Aucun code solution du projet n'a été produit ou dissimulé. L'ensemble de la réflexion, des arbitrages techniques et des explications reste sous l'entière maîtrise intellectuelle de l'auteur.
{% endhint %}
