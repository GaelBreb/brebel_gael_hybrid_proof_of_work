Une bonne partie de la matinée a été consacrée à la résolution d'un problème de connexion aux VMs via SSH pour mon adresse ip publique

[rapport de debug](./rapport_de_debug.md)

Puis mise à jour de quelques éléments pour la durée de vie des certificats et automatiser le régénération au besoin via ansible : 

[commit modification durée de vie CA et régénération certificats](https://github.com/EpitechMscProPromo2027/T-NSA-810-TLS_1/commit/3ee5f4e926177a182d9f8b3729679febed120ba7)


Cette après midi j'ai travailler sur le T-ESP :

J'avais un sérieux doute sur le chiffrage qu'on avait appliqué en terme de jours/hommes, notamment parce que la charge pour chaque développeurs n'allait pas être la même, donc j'ai décidé d'en discuter avec Claude.

En est ressorti que pour cette IA, effectivement il y avait du sous-chiffrage, mais elle a aussi confirmé une surcharge évidente pour le back-end.

Elle m'a généré 2 fichiers :
- [analyse_charge_vibeup.docx](./claude_conv/analyse_charge_vibeup.docx)
- [plan_action_vibeup.docx](./claude_conv/plan_action_vibeup.docx)

J'ai pris connaissance de ces documents et même si je ne suis pas entièrement d'accord avec toutes les suggestions faites, notamment parce que je connais mieux les capacités de chacun et nous avons déjà discuté de comment se répartir le travail, j'ai revu le chiffrage de certaines US selon ses recommandations.

Je garde aussi les suggestions de préparer un kit d'onboarding (plus complet que ce qu'on a aujourd'hui) ainsi que de mutualiser les US de filtrage.

Il y a eu aussi une petite mise à jour de la matrice de compétence ainsi que la matrice technique. En plus d'une demande aux membres de l'équipe de faire de même avec leurs compétences réelles plutôt que celles qu'on avait estimées.