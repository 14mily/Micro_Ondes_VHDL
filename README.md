# Projet Conception Circuit Numérique (CoCiNum) - Système de Contrôle d'un Micro-Onde

## Auteurs
**Émilie GNASSINGBE - Kenneth NONVIGNON**  
**E3e - B3 (2024 - 2025)**

## Encadrement
**Référent Pédagogique & Encadrant : Valentin ARBOUX**

---

## Table des Matières
1. [Introduction](#introduction)
2. [Cahier des Charges](#cahier-des-charges)
3. [Programmation en VHDL](#programmation-en-vhdl)
4. [Tests & Résultats](#tests--résultats)
5. [Problèmes Rencontrés et Solutions Apportées](#problèmes-rencontrés-et-solutions-apportées)
6. [Conclusion](#conclusion)

---

## Introduction

### Contexte
Ce projet s'inscrit dans le cadre des travaux pratiques de l'ECUE **Conception Circuit Numérique (CoCiNum)**. L'objectif était de mettre en application les connaissances acquises lors du semestre 5 à travers la conception, la simulation et la réalisation de circuits numériques.

### Objectif du Projet
Nous avons dû concevoir un **système de contrôle de micro-onde en VHDL** sur une carte **BASYS3**. Ce système devait simuler les fonctions principales d'un micro-onde, notamment :
- La mise en marche
- La cuisson
- La fin de cuisson

---

## Cahier des Charges

Nous avons suivi un cahier des charges précis pour implémenter les fonctionnalités suivantes :

- **Simulation du buzzer** : Utilisation d'une LED pour marquer la fin du temps de cuisson.
- **Simulation du magnétron** : Utilisation d'une LED clignotante pour simuler l'émission des ondes.
- **Simulation de la porte fermée** : La cuisson ne peut commencer que si la porte est fermée et s'arrête si elle s'ouvre.
- **Sélection du temps de fonctionnement** : Configuration du temps de cuisson par dizaines de minutes, minutes, dizaines de secondes et secondes via des interrupteurs.
- **Simulation du bouton central** : Activation de la cuisson par un bouton "start".
- **Simulation du décompte (Afficheur 7 segments)** : Affichage du temps restant sur un afficheur 7 segments.


## Programmation en VHDL

Nous avons modifié un **StopWatch** fourni en travaux pratiques afin de :
- Transformer un chronomètre en **minuteur**.
- Implémenter une **machine à états** pour gérer les transitions (ATTENTE, M_ON, PAUSE, M_OFF).
- Implémenter un système de **pause et reprise** de la cuisson.
- Charger dynamiquement le temps de cuisson avant démarrage.

---

## Tests & Résultats

Nous avons effectué plusieurs tests pour vérifier le bon fonctionnement du système :

- **Passage ATTENTE → M_ON** : Lorsque la porte est fermée et le bouton start enfoncé, le décompte commence.
- **Passage M_ON → PAUSE** : Lorsque la porte s'ouvre, le décompte s'arrête.
- **Passage PAUSE → M_ON** : Lorsque la porte est refermée, le décompte reprend.
- **Passage M_ON → M_OFF** : Lorsque le temps atteint **0**, la cuisson s'arrête.

---

## Problèmes Rencontrés et Solutions Apportées

1. **Transformation d'un chronomètre en minuteur** :
   - Modification des lignes de code du **CounterModN** pour **décrémenter** au lieu d'incrémenter.

2. **Mise en pause et reprise du décompte** :
   - Utilisation d'un **signal std_logic** contrôlé manuellement au lieu d'un simple interrupteur.

3. **Arrêt du minuteur à 0** :
   - Implémentation d'une **machine à états** pour gérer le signal "marche".

4. **Fixation du temps de cuisson** :
   - Utilisation d'un port **load_v** permettant de charger dynamiquement le temps choisi avant le lancement de la cuisson.

---

## Conclusion

Ce projet en VHDL fut une expérience enrichissante qui nous a permis d'allier **théorie et pratique**.
Nous avons pu :
- Appliquer nos connaissances en **conception numérique**.
- Développer un **système fonctionnel** en respectant un cahier des charges.
- Renforcer nos compétences en **résolution de problèmes** et en **programmation VHDL**.

---
