# Plan de correction de la documentation PCD V0.1 (11-2026)

| Version | Date | Auteur | Commentaire |
| --- | --- | --- | --- |
| 0.1 | 06-11-2026 | Gaël Brebel | Initialisation suite à l'audit réalisé lors de la rédaction des kits d'onboarding |

**Document maintenu par :** PM
**Prochaine révision planifiée :** follow-up du 13/11/2026

L'audit de la documentation fait pendant la préparation des kits d'onboarding a relevé 20 incohérences ou manques. Les corrections visibles au follow-up sont à faire pour le 13/11/2026, les autres pour le jalon 4 du 04/12/2026.

## Décisions à prendre d'abord

Trois arbitrages conditionnent plusieurs corrections. Ils sont à tracer dans le journal de décisions.

| N° | Décision | Corrections concernées | Responsable | Échéance |
| --- | --- | --- | --- | --- |
| D1 | Fixer une cible unique de couverture de tests (65 % ou 70 %) et confirmer DEC-018 | C3, C4, C5 | Gaël, Timoty | 13/11/2026 |
| D2 | Définir le format de nom de branche depuis la migration vers Linear (`VIBE-<n>` conservé ou non) | C1 | Timoty, Gaël | 13/11/2026 |
| D3 | Réattribuer ou supprimer le rôle de Tech Lead (évaluation des bugs, maintenance du MDW) | C14, C16 | Gaël, Sayuri | 13/11/2026 |

## Corrections

Chaque correction suit la norme ND : nouvelle version, journal de version à jour, copie PDF sur le Drive et dans le dossier documentation du dépôt.

| N° | Document | Incohérence relevée | Correction attendue | Responsable | Échéance | Statut |
| --- | --- | --- | --- | --- | --- | --- |
| C1 | GN V1.1 - Git normage | La branche est créée depuis le « ticket Jira » | Remplacer par Linear et appliquer le format décidé en D2 | Gaël, Timoty | 13/11/2026 | À faire |
| C2 | KNA V0.1 - Kit du nouvel arrivant | Accès et lien Jira ; aucun renvoi vers les kits par poste | Remplacer Jira par Linear ; ajouter les liens vers OBB, OBW, OBM et OBQ | Gaël | 13/11/2026 | À faire |
| C3 | DOD V1 - Definition of Done | Une seule review exigée (deux selon DEC-008) ; statut « dans Jira » (§ 5) ; couverture à 65 % | Passer à deux reviews, remplacer Jira par Linear, aligner la couverture sur D1 | Timoty | 13/11/2026 | À faire |
| C4 | SQC - Stratégie de qualité de code | Couverture bloquante à 70 % (§ 3) mais vérifiée à 65 % (§ 6) | Aligner les deux paragraphes sur D1 | Timoty | 13/11/2026 | À faire |
| C5 | FP V1 - Fiches de postes | Couverture à 65 % (fiche DevOps/QA) | Aligner sur D1 | Gaël | 13/11/2026 | À faire |
| C6 | SQC - Stratégie de qualité de code | Titre en V1.1 alors que le journal contient une version 1.2 du 11-09-2026 | Renommer le document et son PDF en V1.2 | Gaël | 13/11/2026 | À faire |
| C7 | FP V1 - Fiches de postes | Franck chef d'équipe Mobile et Web et Tech Lead ; Louënig, Salma et Mickaël absents ; fiche Full stack « à venir » | Mathieu chef Mobile, Gaël chef Web, Franck en business consulting (DEC-007, DEC-025, DEC-026) ; ajouter les nouveaux membres | Gaël | 13/11/2026 | À faire |
| C8 | OBS V1.3 | Version 1.3 datée du 10-09-2026, avant la 1.2 du 11-09-2026 | Corriger la date en 10-10-2026 | Gaël | 13/11/2026 | À faire |
| C9 | PBL V1.2 - Product backlog (DEPRECATED) | Toujours classé dans 01_Cadrage_Projet alors que le backlog est dans Linear | Déplacer dans un dossier d'archive | Gaël | 13/11/2026 | À faire |
| C10 | LU V1 - Liens utiles | Lien Jira obsolète ; Linear, GitHub, Figma, Jenkins et Grafana absents | Mettre à jour les liens | Gaël | 13/11/2026 | À faire |
| C11 | Documentation d'accès back-end | Annoncée au point du 06/11, absente du Drive | Déposer dans 06_Technique, sans identifiants en clair | Louënig | 13/11/2026 | À faire |
| C12 | Kits OBB, OBW, OBM, OBQ V0.1 | Liens Swagger, Figma et GitHub manquants ; sujet T9 absent du Drive | Compléter après C10, mettre en forme selon la ND, déposer dans 10_On_Boarding ; déposer le sujet T9 dans 00_Referentiel | Gaël | 13/11/2026 | À faire |
| C13 | QP V0.4 - Plan qualité | Jira pour les branches, les bugs et les tickets ; CI/CD décrite sous GitHub Actions, « à définir » | Remplacer par Linear et Jenkins ; décrire le pipeline réel | Gaël, Timoty | 04/12/2026 | À faire |
| C14 | QP V0.4 - Plan qualité | Sections Rôles et OBS vides ; Franck Tech Lead ; nouveaux membres absents ; Tech Lead cité dans l'évaluation des bugs | Renvoyer vers les FP et l'OBS à jour ; appliquer D3 | Gaël | 04/12/2026 | À faire |
| C15 | SQC - Stratégie de qualité de code | Vérifications décrites sous GitHub Actions (§ 6) ; hook de pré-commit et Dependabot classés en « améliorations futures » (§ 8) | Décrire Jenkins ; déplacer le hook (DEC-033) et Dependabot (DEC-012) dans l'existant | Timoty | 04/12/2026 | À faire |
| C16 | MDW V0.1 - Middleware | « Maintenu par TechLead », rôle sans titulaire ; contenu limité à un paragraphe | Appliquer D3 ; compléter avec l'implémentation réelle | Sayuri | 04/12/2026 | À faire |
| C17 | Rapport d'étonnement de Louënig (brouillon) | Date de mise à jour en 2024 ; même paragraphe répété dans les six dernières sections | Corriger la date et rédiger les sections | Louënig | 04/12/2026 | À faire |
| C18 | MM V0.3 et POC CE V0.3 (carte mobile) | Possibles références à MapBox ou Google Maps, remplacés par MapLibre (DEC-020) | Vérifier et mettre à jour | Félix | 04/12/2026 | À faire |
| C19 | Documentation technique Web | Aucun document Web dans 06_Technique | Créer un document : architecture front, intégration Swagger, tests Vitest | Gaël, Salma, Mickaël | 04/12/2026 | À faire |
| C20 | Documentation d'infrastructure | Aucun document sur Jenkins, K3s, Grafana et les sous-domaines VibeUp | Créer un document : accès, pipelines, métriques, procédure de reprise | Timoty | 04/12/2026 | À faire |
