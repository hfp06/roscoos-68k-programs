# roscoOS/68k — Programmes natifs (binaires)

Programmes natifs prêts à l'emploi pour **roscoOS/68k**, l'OS maison pour la carte
**ROSCO_M68K Classic v2.42** (MC68010 @ 10 ou 12 MHz).

Ce dépôt ne contient **que les binaires** (`.bin`) des programmes. Le noyau
roscoOS et les sources ne sont pas publiés ici.

---

## Philosophie du projet

roscoOS/68k est conçu pour se rapprocher le plus possible d'**OS-9**
(l'OS temps réel de Microware, référence sur 6809 puis 68000), tout en
restant adapté au matériel de la ROSCO_M68K Classic v2.42.

L'objectif n'est pas de cloner OS-9 ligne par ligne, mais d'en retrouver
l'esprit :

- **Multitâche coopératif** avec ordonnancement simple et prévisible
- **Appels système stables** via `TRAP`, à la manière des I/O d'OS-9
- **Shell natif** avec commandes `run`, gestion de fichiers, redirections
- **Programmes binaires position-indépendants ou liés à adresse fixe**,
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
- **OS** : roscoOS/68k (kernel v0.9.3 stable, multitâche coopératif)
- **Shell** : shell natif roscoOS, commande `run` pour lancer les programmes
- **Inspiration** : OS-9 (Microware) — multitâche, TRAP, shell natif
  
---

## Prérequis

- Une carte ROSCO_M68K Classic v2.42 fonctionnelle
- roscoOS/68k installé et démarré (shell accessible)
- Un terminal série (ex. Tera Term sous Windows)
- **Les binaires fournis ici ne fonctionnent que sous roscoOS/68k.**
  Ils ne sont pas compatibles avec un autre OS ni avec un émulateur générique.
- Ils demandent une mise à jour des 2 ROMs, (ROMs modifiées pour permettre l'écriture sur la carte SD)
---

🛠️ Mise à jour des ROMs (Firmware) - Ce qu'il faut retenir
1. Pourquoi mettre à jour les ROMs ?
À l'origine, le firmware de la ROSCO_M68K (version 2.42) ne permettait pas à un programme chargé en RAM (comme mbasic.bin) d'accéder directement à la carte SD.
Les routines de lecture/écriture existaient, mais elles n'étaient pas exposées de manière fiable aux programmes utilisateurs (les appels TRAP #13 échouaient ou n'étaient pas routés correctement).

La solution : Modifier le code source du firmware pour exposer officiellement la lecture et l'écriture de secteurs SD via des appels système (TRAP) stables et documentés.

2. Ce qui a été modifié dans le firmware
Ont été ajouté deux nouvelles fonctions au gestionnaire TRAP #13 (dans le fichier syscalls_asm.asm) :

Numéro de fonction (D0)	Nom	Description
20	SD_READ_SECTOR	Lit un secteur (512 octets) depuis la SD
21	SD_WRITE_SECTOR	Écrit un secteur (512 octets) sur la SD
Comment ça marche ?
Le programme (en RAM) place le numéro de secteur dans D1, l'adresse du buffer dans A2, et le numéro de fonction (20 ou 21) dans D0.

Il exécute TRAP #13.

Le firmware effectue l'opération et retourne 0 (succès) ou 1 (échec) dans D0.

Important : Pour que ces appels fonctionnent, le programme doit fournir une structure SD valide (un pointeur vers une zone mémoire dédiée) dans le registre A1. C'est le seul point d'attention.

3. Ce que cela permet pour le BASIC (et pour la suite)
Grâce à ces deux TRAP, nous avons pu :

Intégrer la bibliothèque FAT (libsdfat.a) directement dans mbasic_projet.bin.

Créer 12 commandes SD dans le BASIC : FILES, SAVE, LOAD, CD, MKDIR, RMDIR, KILL, REN, COPY, TYPE, CHAIN, et DIR.

Tester et valider le tout avec un programme de non-régression (21 tests OK, 0 KO).

Pour la suite, cette base est universelle :

N'importe quel programme chargé en RAM (pas seulement le BASIC) peut désormais lire et écrire sur la SD en utilisant ces TRAP.

C'est une fondation solide pour d'autres projets : un OS (comme OS-9 ou CP/M), un éditeur de texte, un gestionnaire de fichiers, etc.

4. Points de vigilance pour les futures évolutions
Compatibilité : Le firmware modifié reste compatible avec les programmes existants (les TRAP #13 FC 0 à 19 sont inchangés).

Sécurité : L'écriture directe sur la SD est une opération sensible. Il faut toujours sauvegarder les données importantes avant de tester de nouvelles routines.

Compilateur : La compilation du firmware doit se faire avec m68k-linux-gnu-gcc-11 (les versions 13 et 15 produisent du code incompatible avec le 68000 de la ROSCO).

Fichiers sources : Les modifications sont documentées dans syscalls_asm.asm et dans le README.md du projet.

5. En résumé
La mise à jour des ROMs est la clé de voûte du projet. Elle transforme la ROSCO_M68K d'une machine où la SD n'était accessible que par le shell en une plateforme ouverte où tout programme peut lire et écrire sur la carte SD. C'est ce qui a rendu possible le BASIC avec gestion de fichiers, et c'est ce qui ouvrira la porte à de futurs projets (OS, applications, etc.).

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
- Il faut utiliser les versions **1 Mo** (sans suffixe), linkées à `0x080000`,
  qui fonctionnent sur toute ROSCO_M68K Classic v2.42.

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

## Programmes disponibles

| Programme | Taille | Extension 4 Mo | Description |
|---|---|---|---|
| `mbasic/mbasic_rosco.bin` | 91.860 ko | Non | BASIC V4.0 (version 1 Mo, foreground) |
| `mbasic/mbasic_rosco5m.bin` | 91.860 Ko | **Oui** | BASIC V4.0 (version 5 Mo, background) |
| `demos/hello.bin` | 1.286 ko | Non | Exemple « Hello World » |
| `demos/guess.bin` | 1.520 ko | Non | Jeu de devinette |
| `tools/mini-edit.bin` | 0.7924 ko | Non | Éditeur de texte (version 1 Mo) |
| `tools/mini-edit5m.bin` | 0.7924 ko | **Oui** | Éditeur de texte (version 5 Mo) |
| `demos/busy.bin` | 0.758 ko | Non | Test de préemption CPU (version 1 Mo) |
| `demos/busy5m.bin` | 0.758 ko | **Oui** | Test de préemption CPU (version 5 Mo) |

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

## Utilisation depuis le shell roscoOS
 
NE PAS OUBLIER DE METTRE A JOUR LES 2 ROMs
 
1. Copier les binaires sur la carte (carte SD ou transfert série, selon votre
   configuration).
2. Depuis le shell roscoOS, lancer un programme en foreground :
run hello.bin

text

3. Lancer un programme en background (version `5m` uniquement) :
run busy5m.bin &

text

4. Pour BASIC :
run mbasic_rosco.bin

text

ou en background :
run mbasic_rosco5m.bin &

text

---

## Détail des programmes

### mbasic — BASIC V4.0

- Multi-statement (`:` pour séparer les instructions)
- Noms de fichiers longs (LFN 255 caractères)
- `SAVE` de programmes > 8 Ko
- Validé : MEGATEST 36/36, REGTEST 34/34
- **Deux versions** : `mbasic_rosco.bin` (1 Mo, sans extension RAM) et
  `mbasic_rosco5m.bin` (5 Mo, avec extension 4 Mo). Le code est identique ;
  la version 1 Mo est limitée en nombre de lignes, taille des tableaux et
  nombre de variables.

### mini-edit — éditeur de texte

Éditeur de texte natif, disponible en version 1 Mo (foreground) et 5 Mo
(background).

### busy — test de préemption CPU

Programme de test pour vérifier le comportement du multitâche coopératif de
roscoOS. Disponible en version 1 Mo et 5 Mo.

### hello / guess — exemples

Petits programmes d'exemple (`hello.bin` : Hello World ; `guess.bin` : jeu de
devinette) pour découvrir le shell roscoOS.

---

## Limitations connues

- Background (`&`) : uniquement pour les versions `5m`.
- Les binaires sont liés à des adresses fixes (`0x080000` ou `0x200000`) :
ne pas les charger à une autre adresse.
- Compatibilité limitée à roscoOS/68k sur ROSCO_M68K Classic v2.42.
- **Par rapport à OS-9** : roscoOS/68k n'implémente pas encore les modules
  OS-9 (fichiers `.mod`), le format de système de fichiers RBF, ni la
  préemption IRQ complète. Le multitâche est coopératif, pas préemptif.
  Voir la feuille de route pour les chantiers ouverts.
  - **Extension 4 Mo requise** pour tous les binaires `5m` (versions 1 Mo
  disponibles pour les cartes sans extension).
- **BASIC 1 Mo** : mêmes fonctionnalités que le 5 Mo, mais limité en nombre
  de lignes, taille des tableaux et nombre de variables.
  
---

## Feuille de route (chantiers ouverts)

- Préemption IRQ (passage d'un multitâche coopératif à préemptif)
- Gestionnaire de bus-error (robustesse mémoire)
- TTY virtuel (plusieurs sessions)
- Multi-shell
- Polish du shell
- Correction d'un bug mbasic mono-ligne complexe

Ces chantiers rapprochent progressivement roscoOS/68k du modèle OS-9.

---

## Licence

Ce dépôt est publié sous licence **MIT**. Voir le fichier [LICENCE](LICENCE).

---

## Auteur

- **hfp06** — développement de roscoOS/68k et des programmes natifs
- Projet démarré en juillet 2026

---

## Mises à jour

Les nouvelles versions stables seront publiées via des **releases tagguées**.
Les binaires sont mis à jour par commits sur la branche `main`.
