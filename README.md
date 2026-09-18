# Screener RH — Lire un CV face à une offre

> 🇫🇷 Un assistant qui lit un CV face à une offre, relève les écarts et prépare l'entretien. Application publique, démonstrateur.
> 🇬🇧 An assistant that reads a CV against a job description, surfaces gaps and prepares the interview. Public app, demonstrator.

**[Lancer la démo](https://smart-screener-ofk-zpgy6gg8dn56wvapifnxxc.streamlit.app/)**

---

## Le problème

Trier des candidatures prend du temps, et deux personnes lisant le même CV n'en retiennent pas la même chose. Dans une structure sans fonction RH dédiée, cette lecture échoit à quelqu'un dont ce n'est pas le métier.

---

## Ce que fait le système

Extraction du texte du CV, comparaison avec l'offre, restitution en trois blocs : points d'adéquation, points de vigilance, et questions à poser en entretien.

**Ce n'est pas un outil de sélection.** La distinction n'est pas rhétorique — voir la section conformité.

### Pourquoi ce n'est pas du RAG

Une architecture RAG découpe un corpus en fragments, les indexe sous forme de vecteurs, puis récupère les plus proches d'une question avant de les injecter dans le prompt. On le fait quand le corpus dépasse la fenêtre du modèle, quand il change souvent, ou quand il faut citer la source d'une réponse.

Aucune de ces trois conditions n'est remplie par un CV isolé : il tient entièrement dans la fenêtre. Le document est donc **injecté directement en contexte**. Ajouter un index aurait été de la complexité sans gain.

*Note : une version antérieure de ce dépôt décrivait l'architecture comme « RAG simplifié ». C'était une erreur de dénomination, corrigée ici.*

---

## Stack

* **Langage** — Python
* **Modèle** — GPT-4o-mini via l'API OpenAI
* **Orchestration** — LangChain
* **Extraction PDF** — PyPDF2
* **Interface** — Streamlit

---

## Décisions & arbitrages

*Section rétrospective.*

### Injection directe plutôt que RAG

**Contexte.** Corpus d'un document, largement inférieur à la fenêtre du modèle.
**Décision.** Injection directe.
**Ce que ça coûte.** Le système ne monte pas en charge sur un corpus de plusieurs centaines de CV comparés simultanément. Ce n'était pas le besoin.
**Ce que ça évite.** Un index à maintenir, une étape de récupération à évaluer, et une dépendance supplémentaire — pour zéro gain.

### Un modèle économique plutôt qu'un modèle performant

**Contexte.** Application publique, à coût d'usage non maîtrisé.
**Décision.** GPT-4o-mini.
**Ce que ça coûte.** Une qualité d'analyse inférieure à un modèle plus capable, sur les CV atypiques notamment.
**Ce qui manque pour arbitrer sérieusement.** Le coût par exécution n'a jamais été mesuré. La décision a été prise sur une intuition de coût, pas sur un chiffre.

### Une note sur cent

**Décision prise, et que je considère aujourd'hui comme discutable.** Produire une note numérique donne une illusion de précision : le modèle produit un nombre parce qu'on lui en demande un, sans que ce nombre soit stable ni comparable entre candidats.
**Ce que je ferais autrement.** Restituer les écarts et les points de vigilance sans agréger en score. L'extraction structurée fait gagner du temps sans porter de jugement — et sort du périmètre réglementaire le plus contraint.

---

## Conformité — à lire avant tout usage réel

Le tri de candidatures est un usage **explicitement identifié comme à haut risque** par le règlement européen sur l'intelligence artificielle. Recrutement, sélection et évaluation de personnes y sont nommés.

Ce dépôt est un démonstrateur et n'est pas destiné à un usage en production. Une mise en production supposerait au minimum :

* une information du candidat sur l'intervention d'un système automatisé ;
* une supervision humaine formalisée, avec une décision qui reste humaine ;
* une documentation du système, de ses données et de ses limites ;
* des tests de biais sur les sorties, par sous-population ;
* une traçabilité permettant de rejouer une évaluation en cas de contestation.

Aucun de ces éléments n'est présent ici.

---

## Limites connues

* **Le système n'est pas déterministe.** Le même CV soumis deux fois peut recevoir deux analyses différentes. Sur un outil qui compare des candidats, c'est un défaut structurel.
* **Les biais passent par des signaux indirects.** Prénom, établissement, formulation, langue. Un modèle de langage reproduit des régularités apprises sur des corpus où ces signaux corrèlent avec des jugements.
* **L'extraction PDF est un point de rupture silencieux.** Un CV en colonnes ou scanné produit un texte incohérent que le modèle interprète quand même, sans signaler le problème.
* **La note agrégée n'est pas fondée.** Voir la section décisions.

---

## Ce qui n'a pas été mesuré

* La stabilité entre deux exécutions sur un même CV
* L'accord entre les sorties du système et le jugement d'un recruteur humain
* La sensibilité aux variables indirectes — le test serait pourtant simple : un même CV décliné en variantes ne changeant que le prénom ou l'établissement, et on observe si l'analyse bouge
* Le coût et la latence par exécution

---

## Difficultés rencontrées

* **L'extraction PDF fiable** a représenté une part de travail disproportionnée par rapport à la partie modèle — un schéma récurrent sur ce type de projet.

---

*Oumar Fodé KEBE — [oufoke.github.io](https://oufoke.github.io) · [LinkedIn](https://www.linkedin.com/in/oumarfodek/)*
