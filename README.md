<h1 align="center">Stabilisation d'un pendule inversé<br>par volant d'inertie</h1>

<p align="center"><b>TIPE 2026 &nbsp; &nbsp; CPGE PSI &nbsp;|&nbsp; Thème « Cycles et boucles »</b></p>

<p align="center">
  <img src="https://img.shields.io/badge/MATLAB-System_Identification-E16737?style=flat-square">
  <img src="https://img.shields.io/badge/Python-NumPy_%7C_pandas_%7C_Matplotlib-3776AB?style=flat-square&logo=python&logoColor=white">
  <img src="https://img.shields.io/badge/Arduino-Uno_R4_WiFi-00878F?style=flat-square&logo=arduino&logoColor=white">
  <img src="https://img.shields.io/badge/Automatique-Routh_%7C_Bode_%7C_Avance_de_phase-6A1B9A?style=flat-square">
</p>

> **Problématique :** dans quelles conditions un système instable modélisé par un pendule inversé peut-il être stabilisé par une boucle de rétroaction inertielle à volant d'inertie, et quels critères physiques permettent de décider du recours à un déplacement du support ?

**En une phrase :** j'ai conçu une maquette de pendule inversé stabilisé par volant d'inertie (*reaction wheel*), identifié son modèle dynamique expérimentalement, synthétisé une commande en cascade implémentée en temps réel sur microcontrôleur, puis délimité — par Monte-Carlo — la frontière entre les perturbations rattrapables sans bouger le support et celles qui exigent un pas de récupération (*push recovery*).

> [!IMPORTANT]
> **Résultat principal.** Le pendule revient à l'équilibre et y est stabilisé en temps réel pendant environ 3,5 s ; la divergence coïncide avec la saturation du moteur à 9 V (emballement du volant). Le modèle identifié — fit de 84,1 % sur la vitesse angulaire — est recoupé à 13 % près par la physique, et la sortie de la région de capture calculée par Monte-Carlo coïncide avec cette saturation puis la divergence.

**Portfolio et vidéos expérimentales :** https://seyli132.github.io/TIPE_2026_Portfolio/

---

## Pourquoi un pendule inversé ?

Les robots humanoïdes comme Atlas sont structurellement instables : leur équilibre se réduit à un pendule inversé qu'il faut compenser en permanence. Deux stratégies existent : une **réaction inertielle interne** (volant d'inertie) ou un **déplacement du support** (faire un pas). Ce projet détermine quand l'une suffit et quand l'autre s'impose.

<p>
  <img src="Boston_Dynamics_atlas.jpg" height="260" alt="Robot humanoïde Atlas, exemple de système structurellement instable en équilibre dynamique">
  <img src="photo_equilibre.jpg" height="260" alt="Pierres empilées en équilibre précaire au bord de la mer, analogie du pendule inversé">
</p>

*Gauche : robot Atlas (© Boston Dynamics). Droite : un équilibre précaire — chaque pierre est un pendule inversé.*

## Banc d'essai : composants et rôles

![Maquette du pendule inversé à volant d'inertie](images/diapo0.001.png)
 | Composant | Rôle dans la chaîne |
 |---|---|
 | MPU-6500 (gyroscope + accéléromètre) | Mesure de l'angle θ et de la vitesse angulaire θ̇ |
 | MCC + volant d'inertie | Actionneur : crée un couple interne par réaction inertielle |
 | Arduino Uno R4 WiFi + motor shield | Calcule les correcteurs en temps réel (Te = 20 ms) et pilote le moteur en PWM |
 | Batterie 9 V | Alimentation |
 | Structure Lego | Châssis du pendule |

<p>
  <img src="materiel/capteur%20MPU%206500.png" width="23%" alt="Capteur gyroscopique MPU-6500">
  <img src="materiel/arduino_uno.jpeg" width="23%" alt="Carte Arduino Uno R4 WiFi">
  <img src="materiel/arduino_motorshield.jpeg" width="23%" alt="Arduino motor shield">
  <img src="materiel/lego_motor_M.jpeg" width="23%" alt="Moteur Lego et volant d'inertie">
</p>

## De la mesure à la commande

**Chaîne d'énergie et d'information — de la mesure à la commande**

![](images/chaine_energie_chaine_info.jpg)

**Graphe des liaisons et schéma cinématique**

<p>
  <img src="images/graphe_des_liaisons.jpg" width="49%" alt="Graphe des liaisons du pendule inversé à volant d'inertie">
  <img src="images/schema_cinematique.jpg" width="49%" alt="Schéma cinématique du pendule inversé à volant d'inertie">
</p>

---

## Méthode : du modèle au critère de décision

```mermaid
flowchart LR
    A["Modèle analytique<br>TMD + linéarisation"] --> B["Identification expérimentale<br>PRBS + MATLAB"]
    B --> C["Synthèse des correcteurs<br>Routh + Bode + avance de phase"]
    C --> D["Implémentation embarquée<br>Tustin + filtres IMU"]
    D --> E["Analyse des limites<br>saturation, portrait de phase"]
    E --> F["Critère de capturabilité<br>Capture Point + Monte-Carlo"]
```

## Déroulé détaillé

> [!NOTE]
> **Cliquez sur un bouton pour dérouler l'étape correspondante, ses diapositives et leurs explications.** Le premier bloc est déjà ouvert pour montrer le principe.

<details open>
<summary><kbd><b>1 · Problématique</b></kbd></summary>

<br>

![Problématique du TIPE](images/diapo05.png)

Stabiliser un pendule inversé, tout le monde sait le faire avec un chariot. Ici, le support est **fixe** : la seule action disponible est le couple de réaction du volant d'inertie, exactement comme un humanoïde qui compenserait sans bouger les pieds. La question devient alors : *quand* cette action suffit-elle, et *quand* faut-il obligatoirement déplacer le support (faire un pas) ?

</details>

<details>
<summary><kbd><b>2 · Modélisation</b></kbd></summary>

<br>

![Pendule inversé à volant d'inertie : graphe des liaisons et schéma cinématique](images/diapo09.png)

Système étudié : un pendule rigide articulé en O, surmonté d'un volant d'inertie entraîné par un moteur CC. Hypothèses : problème plan, approximation des petits angles. Le théorème du moment dynamique appliqué au volant (en A) puis à l'ensemble {pendule + volant} (en O), suivi d'une linéarisation, donne :

$$\ddot{\theta} \approx \frac{mgh}{I_O}\,\theta - \frac{I_{2A}}{I_O}\,\ddot{\varphi}$$

![Schéma-bloc du pendule inversé avec volant d'inertie](images/diapo11.png)

Le schéma-bloc (moteur CC, capteur gyroscopique, mécanique couplée) aboutit à la fonction de transfert en boucle ouverte. Le modèle complet (4 pôles, 2 zéros) est réduit à **3 pôles / 2 zéros** en négligeant l'inductance du moteur — choix confirmé ensuite par l'identification.

</details>

<details>
<summary><kbd><b>3 · Identification expérimentale</b></kbd></summary>

<br>

**L'astuce centrale du projet : on ne peut pas identifier directement un système instable.** Impossible d'exciter un pendule inversé autour de son équilibre de façon contrôlée : il tombe. Mais un **pendule simple** motorisé, lui, est stable — et possède exactement la même mécanique, le même moteur et le même capteur. Seule la contribution du poids change de signe. J'ai donc :

1. identifié le modèle du pendule **stable** ;
2. transposé au pendule inversé par **inversion du terme gravitaire** (les équations montrent que seul ce terme change de signe).

![Deux maquettes : pendule simple et pendule inversé à volant d'inertie motorisé](images/diapo13.png)

L'excitation utilise un signal **PRBS** (Pseudo Random Binary Sequence) : la commande moteur alterne aléatoirement entre deux valeurs, ce qui balaye toutes les fréquences d'intérêt en un seul essai. Le gyroscope enregistre la vitesse angulaire du pendule en réponse.

![Excitation PRBS du moteur et vitesse angulaire mesurée](images/diapo14.png)

Les fréquences sont choisies physiquement : fréquence propre f₀ ≈ 1,4 Hz ; f_max = 5 Hz (critère de Ljung, ≥ 2,5·f₀) ; échantillonnage f_e = 50 Hz, soit **Te = 20 ms** (Shannon appliqué avec une marge ×10).

![Identification de la FTBO via la System Identification Toolbox de MATLAB](images/diapo16.png)

Deux modèles candidats sont identifiés : **avec inductance** (4 pôles, 2 zéros) et **sans inductance** (3 pôles, 2 zéros). C'est la solution sans inductance qui est retenue : plus simple et suffisamment fidèle, elle confirme *a posteriori* la réduction analytique du modèle.

![Modèles identifiés : avec inductance (4 pôles, 2 zéros) vs sans inductance (3 pôles, 2 zéros) — solution retenue](images/diapo17.png)

La confrontation modèle théorique ↔ identification est éloquente : les formes coïncident, et l'inversion du signe du terme gravitaire entre pendule simple et pendule inversé se retrouve **exactement** dans les fonctions de transfert identifiées :

$$\text{FTBO identifiée (pendule inversé, inductance négligée)}=\frac{-1{,}431\times 10^{-3}\,p}{1+0{,}2681\,p-0{,}01518\,p^{2}-1{,}192\times 10^{-4}\,p^{3}}$$

![Modèle théorique vs identification MATLAB, pendule simple et inversé](images/diapo18.png)

Dernier contrôle — le recoupement physique : les paramètres (masses, distances, résistance interne) sont mesurés avec leurs incertitudes. L'inertie équivalente du volant déduite de l'identification (6,08×10⁻⁴ kg·m²) est comparée à sa valeur théorique (6,99×10⁻⁴ kg·m²) → **écart de 13 %**, cohérent avec les incertitudes. Le modèle n'est pas seulement ajusté numériquement : il est validé par la physique. Fit global : **84,1 %** sur la vitesse angulaire mesurée.

![Paramètres mesurés avec incertitudes et recoupement de l'inertie du volant](images/diapo19.png)

</details>

<details>
<summary><kbd><b>4 · Synthèse des correcteurs</b></kbd></summary>

<br>

Premier constat, brutal : la FTBO identifiée du pendule inversé **échoue au critère de Routh-Hurwitz**. Aucun gain ne peut stabiliser la boucle fermée telle quelle — la stabilité ne peut pas venir de la plante.

![FTBO instable : échec du critère de Routh](images/diapo20.png)

La solution : une commande **en cascade**. Le correcteur interne du second ordre $C_{int}(p)=K_1\left(1+\frac{2\xi_0}{\omega_0}p+\frac{p^2}{\omega_0^2}\right)$ (K₁ = 40, ξ₀ = 1, ω₀ = 20 rad/s) restructure la FTBO, qui **valide alors le critère de Routh**.

![FTBO corrigée : validation du critère de Routh](images/diapo21.png)

La boucle externe se dimensionne ensuite sur le **diagramme de Bode** de la FTBO :

![Diagramme de Bode de la FTBO](images/diapo23.png)

**Cahier des charges :** rapidité — temps d'établissement < 0,1 s → ω_0dB = 35 rad/s ; stabilité — marge de phase **M_φ = 85°**. Pour l'atteindre, un correcteur à avance de phase $C(p)=K\,\frac{1+aTp}{1+Tp}$, avec les paramètres retenus K = 87,8 ; a = 1,514 ; T = 0,0232.

![Cahier des charges et correcteur à avance de phase](images/diapo24.png)

Le dimensionnement aboutit aux marges visées sur la FTBO corrigée :

![Dimensionnement de l'avance de phase sur le diagramme de Bode](images/diapo25.png)

</details>

<details>
<summary><kbd><b>5 · Implémentation embarquée et essais</b></kbd></summary>

<br>

Sur l'Arduino Uno R4 WiFi (Te = 20 ms) : **fusion capteurs** par filtre complémentaire (α = 0,98) gyro/accéléromètre pour l'assiette θ, filtre passe-bas du 1er ordre pour θ̇ ; **discrétisation rigoureuse** des deux correcteurs par transformation bilinéaire (Tustin) → équations aux différences exécutées en temps réel.

L'essai de stabilisation montre un pendule ramené vers l'équilibre et **stabilisé environ 3,5 s autour de 0°**… mais la stabilisation en régime permanent n'est pas maintenue : **emballement du volant** (rotation à vitesse constante saturée).

![Essai de stabilisation : évolution temporelle (Te = 20 ms), emballement du volant](images/diapo28.png)

Diagnostic des 4 causes :

![Limites physiques : saturation, petits angles, frottement sec, bruit capteur](images/diapo30.png)

1. saturation du moteur (limité à 9 V) ;
2. approximation des petits angles ;
3. frottement sec non négligeable (zone morte) ;
4. bruit du gyroscope lors de l'acquisition.

La simulation de la réponse à un couple perturbateur de 0,2 N·m confirme la saturation comme cause dominante :

![Simulation : réponse à un couple perturbateur de 0,2 N·m](images/diapo31.png)

Et le **portrait de phase** expérimental — chaque point coloré par la tension d'actionneur — rend la saturation directement visible : dès que la tension atteint 9 V (points saturés), la trajectoire dévie de la séparatrice.

![Portrait de phase expérimental coloré par la tension d'actionneur](images/diapo32.png)

Vidéos : [premier essai de stabilisation](videos/premier_essai_stabilisation%20.mp4) · [identification sur mesures réelles](videos/system_identification_mesures.mp4)

</details>

<details>
<summary><kbd><b>6 · Capturabilité</b></kbd></summary>

<br>

Pour un humanoïde, le **polygone de support** (zone entre les pieds) délimite les positions rattrapables. Pour un pendule inversé, il se réduit à un **point** — c'est là que tout se joue.

![Point de capture et polygone de support](images/diapo34.png)

Le volant d'inertie rend ce point **dynamique** : la réaction inertielle déplace le point de capture, ce qui ouvre deux stratégies : stabiliser par action inertielle seule, ou rapprocher le point de capture et déclencher un pas (*push recovery*).

![Critère et nécessité du push recovery](images/diapo36.png)

La région de capture est calculée par **Monte-Carlo** (10 000 tirages) en propageant les incertitudes de mesure (masse, hauteur, ω₀) → séparatrice et frontières de capture avec bande de confiance à 95 %.

![Région de capture théorique par Monte-Carlo (10 000 tirages)](images/diapo37.png)

Confrontation au réel : la trajectoire stabilisée reste à l'intérieur de la région de capture prédite ; **les points qui en sortent coïncident avec la saturation à 9 V puis la divergence**. La frontière calculée sépare donc bien les essais rattrapables des essais perdus.

![Région de capture (modèle linéaire) et points expérimentaux](images/diapo38.png)

Perspectives d'optimisation identifiées :

![Perspectives : moteur et tension accrus, modèle non linéaire, frottement sec, structure PLA](images/diapo39.png)

</details>

---

## Compétences démontrées
 | Domaine | Éléments concrets |
 |---|---|
 | Automatique | Analyse harmonique (Bode), critère de Routh-Hurwitz, correcteurs en cascade, avance de phase, marges de stabilité |
 | Identification expérimentale | Excitation PRBS, critères de Shannon-Nyquist et de Ljung, System Identification Toolbox (MATLAB) |
 | Mécanique | Théorème du moment dynamique, linéarisation, inerties équivalentes, propagation d'incertitudes |
 | Temps réel embarqué | Échantillonnage 50 Hz, discrétisation de Tustin, filtre complémentaire IMU, PWM moteur |
 | Programmation | MATLAB, Python (NumPy, pandas, Matplotlib), C++ embarqué (Arduino) |
 | Data / statistique | Méthode de Monte-Carlo, fit quantitatif de modèles sur mesures |

## Calendrier
 | Période | Étape |
 |---|---|
 | Août 2025 | Choix du thème (robotique et stabilisation) |
 | Sept. 2025 | Réduction au modèle du pendule inversé |
 | Oct. 2025 | Étude bibliographique |
 | Déc. 2025 | Problématique et modélisation MATLAB |
 | Janv.–Févr. 2026 | Conception de la maquette, premiers essais |
 | Mai 2026 | Finalisation des expérimentations |
 | Juin 2026 | Analyse finale et validation du modèle |

## Bibliographie

[1] J. Pratt et al., *Capture Point: A Step toward Humanoid Push Recovery*, IEEE-RAS International Conference on Humanoid Robots (ICHR), 2006 — https://doi.org/10.1109/ICHR.2006.321385
[2] J. R. C. Vasconcelos, E. M. Astorga González, P. M. G. del Foyo, *Design and Control of a Flywheel Inverted Pendulum System*, Congresso Brasileiro de Automática (CBA), 2019 — https://www.sba.org.br/open_journal_systems/index.php/cba/article/download/390/354/
[3] M. Olivares, P. Albertos, *Linear control of the flywheel inverted pendulum*, Systems & Control Letters, 2013 — https://doi.org/10.1016/j.sysconle.2013.05.010
[4] A. Chemori et al., *Le pendule inversé stabilisé par volant d'inertie* — https://www.academia.edu/23380289
[5] R. Olfati-Saber, *Global Stabilization of a Flat Underactuated System: the Inertia Wheel Pendulum*, 40th IEEE Conference on Decision and Control (CDC), 2001 — https://authors.library.caltech.edu/records/be5gg-jbe32
[6] O. Boubaker, R. Iriarte (dir.), *The Inverted Pendulum in Control Theory and Robotics: From Theory to New Innovations*, IET, 2017 — https://doi.org/10.1049/pbce111e

---

## Annexes techniques

Démonstrations détaillées : chaîne fonctionnelle, circuit, et justifications mathématiques (diapos 43 à 49).

<details>
<summary><kbd><b>Annexes</b></kbd></summary>

<br>

**Chaîne fonctionnelle** — de la mesure gyroscope à la commande moteur

![Chaîne fonctionnelle](images/diapo43.png)

**Circuit électrique** — MCC, motor shield, alimentation 9 V

![Circuit électrique](images/diapo44.png)

**Équations du schéma-bloc** — théorèmes du moment dynamique et linéarisation

![Équations du schéma-bloc](images/diapo45.png)

**Justification du signal PRBS** — critères de Ljung et Shannon-Nyquist, stratégie PWM

![Justification du signal PRBS](images/diapo46.png)

**Critère de Routh-Hurwitz** — condition de stabilité d'ordre 3, ajustabilité des paramètres

![Critère de Routh-Hurwitz](images/diapo47.png)

**Justification de l'inertie rotorique** — loi des mailles au MCC, moment cinétique de l'arbre moteur

![Justification de l'inertie rotorique](images/diapo48.png)

**Région de capture : démonstration et fit** — changement de variable, séparatrice, formule du fit

![Région de capture : démonstration et fit](images/diapo49.png)

</details>

---

## Auteur

**DAOUDI Ilyes Azouz** — CPGE PSI — TIPE 2026

Portfolio : https://seyli132.github.io/TIPE_2026_Portfolio/
