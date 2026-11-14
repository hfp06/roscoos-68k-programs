<div align="center">

# roscoOS/68k

</div>

##Un système d'exploitation complet pour la carte rosco_m68k**
##A complete operating system for the rosco_m68k board**



Programmes natifs prêts à l'emploi pour **roscoOS/68k**, l'OS maison pour la carte
**ROSCO_M68K Classic v2.42** (MC68010 @ 10 ou 12 MHz).

Ce dépôt ne contient que les **binaires** (`.bin`) des programmes. Le noyau
roscoOS et les sources ne sont pas publiés ici.

---

## 🚀 Quick Start (5 minutes)

1. **Flasher les 2 ROMs** modifiées (`rosco_m68k_even.rom`, `rosco_m68k_odd.rom`)
   — voir [Mise à jour des ROMs](#mise-à-jour-des-roms-firmware)
2. **Copier les `.bin`** sur la carte SD (dossier racine)
3. **Démarrer la ROSCO_M68K** et accéder au shell `roscoos / >`
4. **Tester** : `run hello.bin`
5. **Explorer** :
   - `run mbasic_rosco5m.bin` — BASIC V4.0 (nécessite 4 Mo)
   - `run mini-edit5m.bin` — Éditeur de texte
   - `run guess.bin` — Jeu de devinette

> ⚠️ **Sans les ROMs modifiées**, les programmes ne peuvent pas accéder à la SD.
> ⚠️ **Les versions `5m`** nécessitent l'extension mémoire 4 Mo.

---

## Philosophie du projet

roscoOS/68k est conçu pour se rapprocher le plus possible d'**OS-9**
(l'OS temps réel de Microware, référence sur 6809 puis 68000), tout en
restant adapté au matériel de la ROSCO_M68K Classic v2.42.

L'objectif n'est pas de cloner OS-9 ligne par ligne, mais d'en retrouver
l'esprit :

- **Multitâche coopératif** avec ordonnancement simple et prévisible
- **Appels système stables** via `TRAP`, à la manière des I/O d'OS-9
- **Shell natif** avec commande `run`, gestion de fichiers, redirections
- **Programmes binaires** position-indépendants ou liés à adresse fixe,
  chargeables en foreground ou background
- **Accès unifié aux périphériques** (SD, série, console) depuis les
  programmes utilisateurs, pas seulement depuis le shell

Cette filiation OS-9 guide les choix d'architecture (TRAP #13, structure
SD, gestion des tâches) et la feuille de route (TTY virtuel, multi-shell,
préemption IRQ).

---

## Contexte matériel

- **Carte** : ROSCO_M68K Classic v2.42
- **CPU** : Motorola MC68010 @ 10 ou 12 MHz
- **OS** : roscoOS/68k (kernel v0.9.4 stable, multitâche coopératif)
- **Shell** : shell natif roscoOS, commande `run` pour lancer les programmes
- **Inspiration** : OS-9 (Microware) — multitâche, TRAP, shell natif

---

## Prérequis

- Une carte ROSCO_M68K Classic v2.42 fonctionnelle
- roscoOS/68k installé et démarré (shell accessible)
- Un terminal série (ex. Tera Term sous Windows)
- **Les binaires fournis ici ne fonctionnent que sous roscoOS/68k.**
  Ils ne sont pas compatibles avec un autre OS ni avec un émulateur générique.
- **Mise à jour des 2 ROMs requise** (ROMs modifiées pour permettre
  l'écriture sur la carte SD) — voir section suivante.

---

## 🎬 Démonstration vidéo (7 min)

[![Présentation roscoOS/68k](roscoOS%201.png)](https://github.com/hfp06/roscoOS-68k/releases/download/v0.9.7-video/RoscoOS-Basic.mp4)

*Cliquez sur l'image pour télécharger et lancer la vidéo (MP4, 15 Mo).*

⬇️ **[Télécharger la vidéo (MP4, 15 Mo)](https://github.com/hfp06/roscoOS-68k/releases/download/v0.9.7-video/RoscoOS-Basic.mp4)**

---

## 🛠️ Mise à jour des ROMs (firmware)

### Pourquoi mettre à jour les ROMs ?

À l'origine, le firmware de la ROSCO_M68K (version 2.42) ne permettait pas
à un programme chargé en RAM (comme `mbasic.bin`) d'accéder directement à
la carte SD. Les routines de lecture/écriture existaient, mais elles
n'étaient pas exposées de manière fiable aux programmes utilisateurs
(les appels `TRAP #13` échouaient ou n'étaient pas routés correctement).

**La solution** : modifier le code source du firmware pour exposer
officiellement la lecture et l'écriture de secteurs SD via des appels
système (`TRAP`) stables et documentés.

### Ce qui a été modifié dans le firmware

Deux nouvelles fonctions ont été ajoutées au gestionnaire `TRAP #13`
(dans le fichier `syscalls_asm.asm`) :

| Numéro de fonction (D0) | Nom | Description |
|---|---|---|
| 20 | `SD_READ_SECTOR` | Lit un secteur (512 octets) depuis la SD |
| 21 | `SD_WRITE_SECTOR` | Écrit un secteur (512 octets) sur la SD |

### Comment ça marche ?

Le programme (en RAM) place :

- le numéro de secteur dans `D1`,
- l'adresse du buffer dans `A2`,
- le numéro de fonction (20 ou 21) dans `D0`.

Il exécute `TRAP #13`. Le firmware effectue l'opération et retourne
`0` (succès) ou `1` (échec) dans `D0`.

> **Important** : pour que ces appels fonctionnent, le programme doit
> fournir une structure SD valide (un pointeur vers une zone mémoire
> dédiée) dans le registre `A1`. C'est le seul point d'attention.

### Ce que cela permet pour le BASIC (et pour la suite)

Grâce à ces deux TRAP, nous avons pu :

- Intégrer la bibliothèque FAT (`libsdfat.a`) directement dans `mbasic_projet.bin`.
- Créer 12 commandes SD dans le BASIC : `FILES`, `SAVE`, `LOAD`, `CD`,
  `MKDIR`, `RMDIR`, `KILL`, `REN`, `COPY`, `TYPE`, `CHAIN`, et `DIR`.
- Tester et valider le tout avec un programme de non-régression
  (21 tests OK, 0 KO).

Pour la suite, cette base est universelle :

- N'importe quel programme chargé en RAM (pas seulement le BASIC) peut
  désormais lire et écrire sur la SD en utilisant ces TRAP.
- C'est une fondation solide pour d'autres projets : un OS (comme OS-9
  ou CP/M), un éditeur de texte, un gestionnaire de fichiers, etc.

### Points de vigilance pour les futures évolutions

- **Compatibilité** : le firmware modifié reste compatible avec les
  programmes existants (les `TRAP #13` FC 0 à 19 sont inchangés).
- **Sécurité** : l'écriture directe sur la SD est une opération sensible.
  Toujours sauvegarder les données importantes avant de tester de
  nouvelles routines.
- **Compilateur** : la compilation du firmware doit se faire avec
  `m68k-linux-gnu-gcc-11` (les versions 13 et 15 produisent du code
  incompatible avec le 68000 de la ROSCO).
- **Fichiers sources** : les modifications sont documentées dans
  `syscalls_asm.asm` et dans le `README.md` du projet.

### En résumé

La mise à jour des ROMs est la clé de voûte du projet. Elle transforme la
ROSCO_M68K d'une machine où la SD n'était accessible que par le shell en
une plateforme ouverte où tout programme peut lire et écrire sur la carte
SD. C'est ce qui a rendu possible le BASIC avec gestion de fichiers, et
c'est ce qui ouvrira la porte à de futurs projets (OS, applications, etc.).

Ces appels système s'inspirent directement des I/O d'OS-9 : un numéro de
fonction dans un registre, des paramètres dans d'autres registres, un
`TRAP` pour basculer en mode superviseur, un code de retour dans `D0`.
C'est le même modèle mental que `I$Read`, `I$Write`, etc. sous OS-9.

---

## Extension mémoire 4 Mo : indispensable pour les programmes « 5m »

Les programmes suffixés `5m` (ex. `mbasic_rosco5m.bin`, `mini-edit5m.bin`,
`busy5m.bin`) **exigent l'extension mémoire 4 Mo** (carte d'extension RAM
officielle pour ROSCO_M68K).

Sans cette extension :

- Les programmes `5m` **ne peuvent pas être chargés** : ils sont linkés à
  `0x200000`, adresse qui n'existe pas sur une carte sans extension RAM.
- Il faut utiliser les versions **1 Mo** (sans suffixe), linkées à
  `0x080000`, qui fonctionnent sur toute ROSCO_M68K Classic v2.42.

### Recommandation

L'extension mémoire 4 Mo est **vivement conseillée** pour bénéficier
pleinement des programmes :

| Programme | Version 1 Mo (sans extension) | Version 5 Mo (avec extension 4 Mo) |
|---|---|---|
| **mbasic** | Mêmes fonctionnalités, mais **limité** en nombre de lignes, taille des tableaux et nombre de variables | Mêmes fonctionnalités, **sans ces limitations** (LFN 255, SAVE > 8 Ko, tableaux et variables étendus) |
| **mini-edit** | Fonctionnel | Fonctionnel, avec plus de mémoire de travail |
| **busy** | Fonctionnel | Fonctionnel |

> En résumé : les versions `5m` ne sont pas « une autre version » du
> programme, ce sont **les mêmes programmes** avec accès à la mémoire
> étendue. Le BASIC 1 Mo et le BASIC 5 Mo partagent le même code ; seule
> la capacité mémoire allouée change.

---

## 📸 Aperçu

### Shell roscoOS

![roscoOS shell](roscoOS%201.png)
![roscoOS dir](roscoOS%202.png)
![roscoOS ps](roscoOS%203.png)

### BASIC V4.0 (mbasic_rosco5m.bin)

![BASIC banner](Basic%201.png)
![BASIC programme](Basic%202.png)
![BASIC resultat](Basic%203.png)

---



## Programmes disponibles

| Programme | Taille | Extension 4 Mo | Description |
|---|---|---|---|
| `mbasic/mbasic_rosco.bin` | 91 860 o | Non | BASIC V4.0 (version 1 Mo, foreground) |
| `mbasic/mbasic_rosco5m.bin` | 91 860 o | **Oui** | BASIC V4.0 (version 5 Mo, background) |
| `demos/hello.bin` | 1 286 o | Non | Exemple « Hello World » |
| `demos/guess.bin` | 1 520 o | Non | Jeu de devinette |
| `tools/mini-edit.bin` | 7 924 o | Non | Éditeur de texte (version 1 Mo) |
| `tools/mini-edit5m.bin` | 7 924 o | **Oui** | Éditeur de texte (version 5 Mo) |
| `demos/busy.bin` | 758 o | Non | Test de préemption CPU (version 1 Mo) |
| `demos/busy5m.bin` | 758 o | **Oui** | Test de préemption CPU (version 5 Mo) |

### Versions « 1 Mo » vs « 5 Mo »

Les programmes existent en deux versions :

- **Version 1 Mo** (sans suffixe) : linkée à `0x080000`, chargée en
  **foreground** via `run nom.bin`. Fonctionne sur **toute** ROSCO_M68K
  Classic v2.42, sans extension mémoire.
- **Version 5 Mo** (suffixe `5m`) : linkée à `0x200000`, chargée en
  **background** via `run nom5m.bin &`. **Exige l'extension mémoire 4 Mo**
  (carte d'extension RAM officielle). Sans elle, le chargement échoue.

> **Limitation connue** : le mode background (`&`) n'est supporté que pour
> les versions `5m`. Les versions 1 Mo ne peuvent pas être lancées avec `&`.

---

## Retours et questions

- **Issues** : pour signaler un bug ou poser une question précise
- **Discussions** : pour échanger sur roscoOS/68k, OS-9, ou proposer des idées

---

## Utilisation depuis le shell roscoOS

> ⚠️ **NE PAS OUBLIER DE METTRE À JOUR LES 2 ROMs** avant toute utilisation
> (voir la section « Mise à jour des ROMs » ci-dessus).

1. Copier les binaires sur la carte (carte SD ou transfert série, selon
   votre configuration).
2. Depuis le shell roscoOS, lancer un programme en foreground :


---

## 🙏 Crédits

- **ROSCO_M68K** — Xark et la communauté (https://github.com/rosco-m68k/rosco_m68k)
- **FatFs** — ChaN (http://elm-chan.org/fsw/ff/)
- **E-Kermit** — Columbia University
- **m68k-linux-gnu-gcc-11** — équipe GCC
- **Tera Term** — T. Teranishi

Sans ces projets, roscoOS/68k n'existerait pas.

## 📢 Politique de publication

| Composant | Statut | Licence |
|-----------|--------|---------|
| **Programmes** (`.bin`) | ✅ Publiés | MIT |
| **ROMs modifiées** | ✅ Publiées | Voir licence ROSCO_M68K |
| **Screenshots** | ✅ Publiés | MIT |
| **Sources du kernel** (`roscoos.bin`, `sched.c`, etc.) | ⏳ Non publiées | — |

---

# FAQ — roscoOS/68k

### Le projet est-il open source ?

**Non, pas encore.** roscoOS/68k est un projet indépendant. Les **binaires** et la **documentation** sont mis à disposition de la communauté pour permettre à chacun de tester, utiliser et donner son avis sur le système. Le **code source du noyau n'est pas publié**.

### Pourquoi ne pas publier les sources ?

C'est un **choix de développement**, pas une faiblesse technique. Le noyau est en pleine maturation (passage du multitâche coopératif au multitâche préemptif). L'auteur souhaite en garder la maîtrise pendant cette phase pour garantir la cohérence et la qualité du projet.

Une **publication éventuelle des sources** sera envisagée lorsque le projet aura atteint un niveau de maturité suffisant. Aucune date n'est fixée à ce jour.

### Puis-je réutiliser les binaires ?

**Oui.** Les binaires et la documentation sont distribués sous **licence MIT** (voir `LICENSE` et `LICENSE-CLARIFICATION.md`). Vous pouvez les utiliser, les modifier et les redistribuer, à condition de conserver la mention de licence.

En revanche, la licence MIT **ne vous donne aucun droit sur le code source absent**.

### Le dépôt s'appelle "roscoos-68k-programs", est-ce seulement une collection de programmes ?

**Non.** Le nom du dépôt est historique, mais le contenu est bien celui d'un **système d'exploitation complet** : noyau, ordonnanceur, shell, BASIC, éditeur, gestion des périphériques. Les binaires publiés sont les **composants** de cet OS, pas des utilitaires isolés.

### Sur quel matériel roscoOS/68k fonctionne-t-il ?

Sur la carte **rosco_m68k** (processeur Motorola 68000), avec 4 Mo de RAM et l'extension de bus. Le projet est développé et testé sur cette configuration.

### Quelle est la différence entre la Phase 1 et la Phase 2 ?

- **Phase 1 — Multitâche coopératif** (opérationnel) : les tâches rendent volontairement la main au noyau pour permettre aux autres de s'exécuter.
- **Phase 2 — Multitâche préemptif** (en développement) : le noyau reprend périodiquement le contrôle du processeur via une interruption et son ordonnanceur, sans coopération des tâches.

### Comment signaler un bug ou proposer une amélioration ?

Ouvrez une [issue](https://github.com/hfp06/roscoOS-68k/issues) sur ce dépôt. Les retours d'expérience sont précieux pour faire avancer le projet.

### Puis-je contribuer au code ?

Le code source n'étant pas publié, les contributions directes au noyau ne sont pas possibles pour l'instant. En revanche, les **retours d'utilisation**, les **rapports de bugs** et les **suggestions** sont les bienvenus et seront étudiés avec attention.

---

*English version of this FAQ will be added soon.*

---

# Clarification de la licence / License clarification

## 🇫🇷 Français

La licence **MIT** appliquée à ce dépôt (voir fichier `LICENSE`) couvre **exclusivement** :

- les fichiers **binaires** publiés (par exemple `.bin`, `.rom`, images disque, etc.) ;
- la **documentation** textuelle (README, manuels, notes).

Elle **ne couvre pas** et **ne peut en aucun cas être interprétée comme couvrant** :

- le **code source du noyau** de roscoOS/68k, qui n'est pas publié ;
- les sources des outils internes, non publiées ;
- toute éventuelle version future des sources, sauf mention contraire explicite.

**La présence de la licence MIT ne constitue pas une offre de publication des sources.** Le noyau de roscoOS/68k reste la propriété intellectuelle de son auteur, qui conserve l'ensemble des droits sur celui-ci.

Toute personne qui télécharge un binaire est libre de l'utiliser, de le modifier et de le redistribuer conformément à la licence MIT, **sans pour autant obtenir un quelconque droit sur le code source absent**.

## 🇬🇧 English

The **MIT** license applied to this repository (see `LICENSE` file) covers **exclusively**:

- the published **binary** files (e.g. `.bin`, `.rom`, disk images, etc.);
- the **documentation** (README, manuals, notes).

It **does not cover**, and **shall in no way be construed as covering**:

- the **roscoOS/68k kernel source code**, which is not published;
- the sources of internal tools, which are not published;
- any future source release, unless explicitly stated otherwise.

**The presence of the MIT license does not constitute an offer to publish the source code.** The roscoOS/68k kernel remains the intellectual property of its author, who retains all rights to it.

Anyone downloading a binary is free to use, modify and redistribute it in accordance with the MIT license, **without thereby acquiring any right over the absent source code**.

---


## 🎯 Feuille de route

- [x] **Phase 1** — Multitâche coopératif (`sched_create`, `yield`, `sleep`, `kill`)
- [ ] **Phase 2** — Préemption IRQ (recherche Motorola `rte` 68010 en cours)
- [ ] **RTC DS3231** — I2C, pour TIME$/DATE$ persistantes
- [ ] **Carte graphique** — En cours de conception
- [ ] **Publication des sources du kernel** — v1.0

## 📝 Changelog

Voir [CHANGELOG.md](CHANGELOG.md) pour l'historique des versions.
