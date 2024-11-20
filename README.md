# Blockenheimer_update
[Blockenheimer](https://codeberg.org/xormetric/bblock), a version actuelle de l'application ne garantit pas la sécurité des données qui transitent, y compris celles utilisées pour la connexion indépendant de Bluesky et propre à l'application


Ce document présente les problèmes de sécurité identifiés dans le code fourni par l'utilisateurice [xormetric](https://codeberg.org/xormetric) et propose des solutions pour rendre le code plus sécurisé. Le code contient plusieurs vulnérabilités courantes qui peuvent compromettre la confidentialité et la sécurité des utilisateur·ices, mais aussi des problèmes de gestion mal pris en charge ainsi que des problèmes d'optimisation pouvant créer des incidents. Voici donc une version plus à jours.



## Précision : 

Il est important de souligner que cette analyse n'a pas pour but de porter atteinte à la personne ou à l'équipe ayant développé l'application. L'objectif principal est de soulever des problèmes liés à la sécurité des utilisateurs et des utilisatrices, dans le but d'améliorer la protection des données personnelles et de garantir la sécurité de l'application.
