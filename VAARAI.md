# Anet A8 Vaarai — configuration et maintenance

## Branche de référence

- Branche machine : **`anet-a8-vaarai`**, publiée sur `origin` (fork
  `https://github.com/Vaarai/Marlin.git`).
- Dépôt officiel : `upstream`, `https://github.com/MarlinFirmware/Marlin.git`.
- Base stable intégrée : **Marlin 2.1.2.9**, tag officiel, commit `7d300b42ac`.
- Paramètres machine : `Marlin/Configuration.h` et `Marlin/Configuration_adv.h`.
  Ces fichiers versionnés sont la référence ; ne pas les remplacer par les
  exemples ou les configurations par défaut lors d’une mise à jour.

La branche machine porte les personnalisations au-dessus des versions officielles.
Chaque prochaine modification doit être un commit explicite décrivant son objet
et ses validations. Les sorties `.pio/` ne sont pas versionnées.

## Matériel et fonctions actuels

| Élément | Configuration |
|---|---|
| Machine | Anet A8 modifiée, RAMPS `BOARD_RAMPS_13_EFB`, Mega2560 |
| Pilotes | DRV8825 X/Y/Z/Z2/E0 |
| Double Z | Deux pilotes indépendants ; Z2 sur E1, STEP36/DIR34/ENABLE30 |
| Thermistances | Type 5 pour hotend et plateau |
| Géométrie firmware | Plateau 220×220 ; minima −33/−10/0 ; maxima 220/220/230 |
| Pas/mm X/Y/Z/E | 100 / 100 / 400 / 111,112 |
| Vitesses maximales | 400 / 400 / 8 / 50 mm/s |
| Accélérations maximales | 500 / 500 / 200 / 1000 mm/s² |
| Accélérations P/R/T | 500 / 1000 / 500 mm/s² |
| Mouvement | S-curve, Junction Deviation 0,02 ; arcs G2/G3 |
| Linear Advance | Activé ; K firmware 0 ; K0,06 fourni par le profil PLA OrcaSlicer |
| Régulation | PID hotend 22,20 / 1,08 / 114 ; Kc100 ; plateau bang-bang |
| Protections | Thermiques, watchdog, emergency parser ; extrusion minimale 160 °C |
| Communication | Série 0, 250000 bauds ; host actions/prompts/notifications |
| EEPROM | Désactivée : M500 ne sauvegarde pas les réglages |
| Sonde et secours Z | BLTouch et chaîne NC commune prévus, pas encore implémentés |

Le switch Z actuel a été signalé comme non fiable ; sa remise en état reste à
faire. Le `KILL_PIN` standard n’est pas une butée anticollision rapide validée.

## Compilation hors ligne

Depuis la racine du dépôt, avec PlatformIO Core installé :

```sh
pio run -e mega2560
```

Pour une validation de migration, commencer par un nettoyage :

```sh
pio run -e mega2560 -t clean
pio run -e mega2560
```

Produits : `.pio/build/mega2560/firmware.hex` et `firmware.elf`.

Validation du 26 octobre 2026 : compilation propre réussie sur Atmel AVR 4.0.1,
Arduino AVR 5.1.0, AVR GCC 7.3.0 ; Flash **66250/253952**, RAM **3198/8192**.
Seul avertissement : attribution automatique de Z2 au port E1 libre, cohérente
avec le câblage confirmé. Configuration inchangée par rapport à la version locale
2.1.2.7 ; aucune adaptation de fonctionnalité requise pour 2.1.2.9.

Il s’agit d’une validation hors ligne. Le dernier firmware installé connu reste
2.1.2.7, compilation du 25 octobre 2026 à 17:24:43.

## Intégrer une prochaine stable

1. Partir de `anet-a8-vaarai`, vérifier `git status` et synchroniser avec le fork
   par `git pull --ff-only`. L’arbre doit être propre avant intégration.
2. Vérifier la dernière **release stable officielle**, ses notes et son tag ;
   ne pas prendre automatiquement une branche de développement ou un tag mobile.
3. Récupérer le tag exact : `git fetch upstream tag <version>`.
4. Examiner `git diff HEAD <version> -- Marlin/Configuration.h
   Marlin/Configuration_adv.h`, les changements de format, les notes de migration,
   les pins RAMPS, le HAL AVR et les fonctions actives listées ci-dessus.
5. Intégrer le tag dans la branche machine avec `git merge <version>`.
   Résoudre les conflits en conservant les intentions de la configuration machine
   et en adaptant explicitement les paramètres renommés ou modifiés. Ne pas
   résoudre globalement par « ours » ou « theirs ».
6. Vérifier le diff des configurations et les macros effectives, notamment
   polarités/sens moteurs, broches Z2, thermistances, géométrie, limites, Linear
   Advance, S-curve, JD et protections ; compiler proprement pour `mega2560`.
7. Mettre à jour cette fiche avec la base intégrée, les adaptations et le résultat
   du build ; créer un commit pour les adaptations supplémentaires si nécessaire.
8. Pousser par `git push origin anet-a8-vaarai`. Garder l’historique publié :
   les prochaines stables s’intègrent par merge, sans rebase ni push forcé.

## Exécution sur l’imprimante

Tous les mouvements, homing, probing, extrusion/rétraction physique, chauffes,
autotunes et lancements/reprises d’impression sont exécutés exclusivement par
l’utilisateur. L’assistant prépare et vérifie hors ligne ; il ne déclenche pas
ces actions et ne transmet pas de G-code d’impression à la machine.

Après installation par l’utilisateur, les lectures `M115`, `M503`, `M119` et
`M900` servent à comparer la version, les réglages, les états des entrées et la
disponibilité du Linear Advance avant les essais physiques utilisateur.
