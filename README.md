# TIPE 2026 – Stabilisation dynamique d’un pendule inversé

---

## Accès au portfolio expérimental

https://seyli132.github.io/TIPE_2026_Portfolio

Ce site regroupe les vidéos expérimentales et les démonstrations du système physique étudié.

---

# 1. Résumé du projet

Ce TIPE étudie la stabilisation dynamique de robots anthropomorphes à travers le modèle du pendule inversé couplé à un volant d’inertie.

L’objectif est d’analyser les conditions de stabilisation d’un système instable et de distinguer les situations où une régulation inertielle est suffisante de celles nécessitant une stratégie de récupération de l’équilibre.

Le travail repose sur une approche combinant :
- modélisation mécanique (Lagrange)
- identification expérimentale
- commande en boucle fermée
- analyse des limites physiques

---

# 2. Problématique

Dans quelles conditions un système instable modélisé par un pendule inversé peut-il être stabilisé par une boucle de rétroaction inertielle, et quels critères physiques permettent de décider du recours à un déplacement du support ?

---

# 3. Contributions

- Modélisation d’un pendule inversé motorisé à volant d’inertie
- Mise en œuvre d’une stabilisation par régulation inertielle
- Identification expérimentale du système réel
- Étude des limites physiques (saturation, énergie, vitesse)
- Introduction d’un critère de capturabilité basé sur le Capture Point

---

# 4. Démarche scientifique

## 4.1 Modélisation
Formulation du système par la mécanique lagrangienne et analyse de la dynamique sous-actionnée.

## 4.2 Commande
Stabilisation par boucle fermée en position angulaire utilisant un volant d’inertie comme actionneur de couple interne.

## 4.3 Limites physiques
Analyse des contraintes :
- saturation des actionneurs
- énergie disponible
- vitesse maximale du volant

---

# 5. Architecture du système

## Chaîne d’énergie et d’information

![Chaîne énergie et information](images/chaine_energie_chaine_info.jpg)

---

## Graphe des liaisons

![Graphe des liaisons](images/graphe_des_liaisons.jpg)

---

## Schéma cinématique

![Schéma cinématique](images/schema_cinematique.jpg)

---

# 6. Portfolio expérimental

https://seyli132.github.io/TIPE_2026_Portfolio

Le portfolio constitue la partie expérimentale centrale du projet.

Il contient :
- tests de stabilisation
- réponses à excitation PRBS
- validation du modèle identifié

---

# 7. Critère de capturabilité

L’étude introduit un critère inspiré du Capture Point permettant de distinguer :
- les états stabilisables par action inertielle
- les états nécessitant une stratégie de déplacement du support (push recovery)

Ce critère permet de relier la commande inertielle à des concepts de robotique humanoïde.

---

# 8. Bibliographie

[1] J. Pratt et al., *Capture Point: A Step toward Humanoid Push Recovery*, ICHR, 2006.  
https://doi.org/10.1109/ICHR.2006.321385  

[2] J. R. C. Vasconcelos et al., *Design and Control of a Flywheel Inverted Pendulum System*.  

[3] M. Olivares, P. Albertos, *Linear control of the flywheel inverted pendulum*, Systems & Control Letters, 2013.  
https://doi.org/10.1016/j.sysconle.2013.05.010  

[4] A. Chemori et al., *Le pendule inversé stabilisé par volant d’inertie*.  
https://www.academia.edu/23380289  

[5] R. Olfati-Saber, *Global Stabilization of a Flat Underactuated System*.  

[6] O. Boubaker, R. Iriarte, *The Inverted Pendulum in Control Theory and Robotics*.

---

# 9. Chronologie du projet (DOT)

- Août 2025 : choix du thème (robotique et stabilisation)
- Septembre 2025 : réduction au modèle du pendule inversé
- Octobre 2025 : étude bibliographique
- Décembre 2025 : définition de la problématique et modélisation Matlab
- Janvier 2026 : conception de la maquette et premiers tests
- Février 2026 : échanges avec des chercheurs en robotique
- Mai 2026 : finalisation des expérimentations
- Juin 2026 : analyse finale et validation du modèle

---

# Auteur

DAOUDI Ilyes Azouz  
CPGE PSI – TIPE 2026  

GitHub: https://github.com/Seyli132
