# Plan - Réorganisation VibeUp

**Date :** 24 juin 2026  
**Rédigé par :** Gaël (PM)  
**Statut :** En cours de validation — point équipe prévu le 25/06/2026

---

## 1\. Contexte

Deux événements simultanés imposent une réorganisation du projet :

- **Absence de Franck** pour une durée indéterminée. Franck occupait les rôles de Tech Lead, lead mobile, lead web, et référent architectural. C'est la perte de compétence la plus impactante possible au regard de la matrice technique.  
- **Jalon 2 non réalisé.** Le MVP prévu pour le 29/06/2026 (feed, map, création d'événement) n'a pas été livré, l'équipe s'étant concentrée sur d'autres projets de formation sur cette période.

Ces deux problèmes se cumulent à 5 jours de l'échéance initiale du jalon 2, en entrée de période de vacances.

---

## 2\. Redistribution des rôles de Franck

La redistribution proposée s'appuie sur la matrice de compétences existante.

| Rôle | Titulaire précédent | Repris par | Remarques |
| :---- | :---- | :---- | :---- |
| Lead mobile | Franck | **Mathieu** | Profil le plus polyvalent de l'équipe mobile. À confirmer avec lui. |
| Lead web | Franck | **Gaël** | En cumul avec le rôle PM. Charge à surveiller. |
| Décisions d'architecture | Franck | **Sayuri \+ Timoty** | En binôme : Sayuri côté applicatif, Timoty côté infra/qualité. |
| Design / Commercial | Franck | **Mis en veille** | Non prioritaire pour les jalons à venir. |

**Point de vigilance :** Gaël cumule désormais PM \+ lead web. Ce cumul est viable sur le court terme mais devra être réévalué à l'arrivée des renforts. Si la charge devient critique, le lead web peut être transféré à un renfort compétent en React/TS.

---

## 3\. Réflexion en cours sur le recrutement

### Contexte

L'équipe passe de 7 à 6 membres effectifs. Le CDC prévoyait déjà \+1 à \+2 renforts à partir du jalon 3\. Avec l'absence de Franck, ces renforts deviennent structurels et non optionnels.

### Profils recherchés

**Priorité 1 — Développeur mobile React Native**

Le profil le plus urgent. Mathieu reprend le lead mobile mais perd son référent technique expert.

Compétences indispensables :

- React Native — niveau autonome minimum (2/3)  
- Expo (managed ou bare workflow)  
- Navigation mobile  
- Tests mobile (Jest / Detox)

Compétences appréciées :

- Supabase (sous-maîtrisé dans toute l'équipe actuelle)  
- Publication App Store / Google Play (aucun membre actuel maîtrise ce point)

**Priorité 2 — Développeur fullstack Node.js / React**

Soulage le backend (Sayuri \+ Haifa seules) et le web (Gaël seul). Un bon fullstack JS/TS apporte plus de valeur qu'un pur spécialiste dans ce contexte.

Compétences indispensables :

- Node.js / Express — niveau autonome (2/3)  
- React / TypeScript — niveau autonome (2/3)  
- PostgreSQL

Compétences appréciées :

- JWT / Auth (l'US-32 authentification arrive dès le jalon 3\)  
- Supabase

### Ce qu'on ne cherche pas

Profil DevOps pur (Timoty couvre ce territoire), designer (mis en veille), PM/PO supplémentaire.

### Timing d'intégration cible

Les renforts doivent être intégrés **avant septembre 2026** pour être onboardés pendant le jalon 2 (le plus accessible techniquement) plutôt qu'en plein jalon 3 où l'US-32 est bloquante pour tout le reste.

---

## 4\. Recalibrage du calendrier

### Hypothèses retenues

| Paramètre | Valeur |
| :---- | :---- |
| Équipe effective | 6 membres (sans Franck) |
| Renforts | \+1 à \+2 dès jalon 3 |
| Proportion temps école | 1/3 inchangée |
| Période de vacances (non comptabilisée) | 06/07/2026 → 11/09/2026 |
| Jours travaillés en juillet | 4 jours pleins ESP |
| Réserve de contingence | Supprimée (à risque — voir point 4.3) |
| Échéance finale | Avril 2027 — non repoussable |

### Nouveau calendrier proposé

| Jalon | Contenu | Nouvelle échéance | Équipe | Capacité estimée |
| :---- | :---- | :---- | :---- | :---- |
| **J2 — MVP** | Feed carousel, map, création d'événement, contrats API | **25 sept. 2026** | 6 (+renfort éventuel) | \~20 j/h (dont 4j juillet) |
| **J3 — Auth & interactions** | Authentification, compte promoteur, abonnements, tags vibe | **04 déc. 2026** | 7-8 | \~50 j/h |
| **J4 — Algo & filtres** | Recommandation, filtres avancés, admin, stats | **05 mars 2027** | 8-9 | \~55 j/h |
| **J5 — Enrichissement** | Notifications, métriques, modération, stories, participation | **25 avril 2027** | 8-9 | \~40 j/h |
| **J7+ — Hors scope** | Monétisation, web client complet, gamification | Post-lancement | — | — |

Les jalons 5 et 6 du CDC original sont **fusionnés** en un seul jalon final (J5), les US correspondantes étant les plus légères individuellement.

### Évolution de la charge par rapport au planning initial

Le CDC initial prévoyait 161 j/h disponibles pour 7 membres sur toute la période, avec 20% de réserve soit 128,8 j/h effectifs de dev.

Avec la réorganisation actuelle :

- Équipe réduite à 6 → capacité brute ramenée à **\~138 j/h**  
- Suppression de la réserve de 20% → **138 j/h disponibles** pour le dev  
- Charge totale estimée des jalons 2 à 5 : **\~165 j/h**  
- **Déficit résiduel : \~27 j/h**, comblé par les renforts (+1 renfort ≈ \+20-25 j/h, \+2 renforts ≈ équilibre atteint)

### Risque lié à la suppression de la réserve

La réserve de 20% a été supprimée pour absorber la perte de capacité liée à l'absence de Franck. Cela signifie qu'il n'y a plus de marge pour :

- Les bugs et imprévus techniques  
- Le temps d'onboarding des renforts  
- Les rechiffrages à la hausse

**Recommandation :** maintenir une réserve minimale de 10% si les renforts sont confirmés à \+2 personnes. Un point de recalibrage est prévu fin septembre (après le jalon 2\) pour ajuster selon la vélocité réelle mesurée.

---

## 5\. Priorités pour les 4 jours de juillet

Sur les 4 jours ESP disponibles en juillet, environ 2 à 4 j/h seront absorbés par le recrutement (entretiens, onboarding éventuel). Les 20 à 22 j/h restants sont répartis comme suit :

**Backend / architecture (Sayuri \+ Haifa \+ Timoty)**

- Finalisation des contrats API (US-76) — débloquant pour tout le dev mobile  
- Modélisation et setup Supabase  
- Documentation des décisions d'architecture

**Mobile (Mathieu \+ Félix)**

- Map interactive avec données mockées (React Native Maps \+ géolocalisation)  
- Structure de navigation mobile et design system de base  
- En parallèle des contrats API — pas de dev applicatif métier avant que les endpoints soient définis

**PM (Gaël)**

- Rédaction et diffusion des profils de recrutement  
- Organisation des entretiens

---

## 6\. Points ouverts à valider lors du point du 25/06

- Confirmation de Mathieu sur le lead mobile  
- Confirmation de Sayuri \+ Timoty sur le co-pilotage architecture  
- Disponibilités réelles sur les 4 jours de juillet (présence / remote)  
- État des POC et travaux partiels non remontés dans Jira  
- Contacts déjà identifiés pour le recrutement  
- Validation collective du nouveau calendrier

---

*Document de travail — à mettre à jour après le point du 25/06/2026*  
