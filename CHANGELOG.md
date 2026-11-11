# Changelog — roscoOS/68k Programs

Format : [Keep a Changelog](https://keepachangelog.com/fr/1.1.0/).

## [Unreleased]

### Prévu
- Exemples BASIC (`samples/hello.bas`, `samples/fibonacci.bas`, ...)
- Support RTC DS3231 sur I2C
- Carte graphique dédiée (en conception)
- Publication des sources du kernel (v1.0)

## [1.0.0] - 2026-11-11

### Ajouté
- `ROSCODE1.bin` + `roscoos.bin` (71 940 o) — Kernel stable
- `mbasic_rosco.bin` + `mbasic_rosco5m.bin` (91 860 o) — BASIC V4.0
- `mini-edit.bin` + `mini-edit5m.bin` (7 924 o) — Éditeur
- `hello.bin` (1 286 o) — Hello World
- `guess.bin` (1 520 o) — Jeu de devinette
- `busy.bin` + `busy5m.bin` (758 o) — Test préemption CPU
- `rosco_m68k_even.rom` + `rosco_m68k_odd.rom` — ROMs modifiées (TRAP #13 SD)
- Documentation complète (README, philosophie OS-9, pièges)
- Screenshots démonstration
- `LICENSE` (MIT)

### Notes
- Kernel roscoOS/68k v0.9.4 stable (multitâche coopératif)
- Firmware ROSCO_M68K v2.42 modifié (TRAP #13 SD_READ_SECTOR / SD_WRITE_SECTOR)
- Les versions `5m` nécessitent l'extension mémoire 4 Mo
