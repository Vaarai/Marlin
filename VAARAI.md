# Anet A8 Vaarai — configuration et maintenance

## État actuel

- La configuration source définit `NOZZLE_TO_PROBE_OFFSET { 0, -29, -2.06 }`.
- Le dernier firmware installé connu est le build Marlin 2.1.2.9 du 9 novembre
  2026 à 21:43:22, avec EEPROM activée et un défaut compilé Z−2,00. La
  modification source à Z−2,06 n'a pas été téléversée.
- L'utilisateur a testé et sauvegardé `M851 Z-2.06` en EEPROM ; sa relecture
  après redémarrage n'a pas été communiquée. Les valeurs EEPROM peuvent différer
  des défauts compilés : ne pas réinitialiser sans préserver la calibration.
- Nivellement bilinéaire 2×2 et G34 sont activés. Le maillage sauvegardé a été
  retrouvé après redémarrage ; G28 désactive sa compensation, à réactiver avec
  `M420 S1` après vérification d'un maillage valide.

Les paragraphes datés ci-dessous décrivent des étapes historiques.

**Historique :** nivellement bilinéaire **2×2**, marge40 mm, téléversé
et vérifié (Marlin2.1.2.9 `Nov 9 2026 18:56:24`, Flash89984/RAM3443). Points de
sonde X40/180 × Y40/180 ; buse Y69/209 compte tenu de l'offset−29. Aucun G29 ni
déplacement n'a été exécuté par l'assistant. M503 indique M420 S0 Z10, M851
X0/Y−29/Z−2. EEPROM désactivée, G28 désactive le maillage et le démarrage Orca
ne lance pas G29. Premier essai et dégagement pince arrière à valider par utilisateur.

**Historique :** Z−2,00 validé par l'utilisateur et enregistré dans
`Configuration.h`. Build80500 octets / RAM3332 téléversé et vérifié sur COM5 ;
M115 `Nov 9 2026 18:04:38`, M503 confirme M851 X0/Y−29/Z−2,00. EEPROM désactivée,
la valeur est persistante dans le firmware. Les paragraphes datés ci-dessous sont historiques.

**Historique :** firmware téléversé `Nov 8 2026 21:24:26`, origine XY prévue
au coin avant gauche du verre, X_MIN−11/X_MAX242, Y_MIN−11/Y_MAX219 ; sonde au
centre110/110, buse110/139. M851 X0/Y−29/Z−2,15 relu. Profil Orca offset0x0,
zone X0..220/Y8..212 pour les pinces. Repère et Z au nouveau point à vérifier
par l'utilisateur. Les chiffres datés ci-dessous sont historiques.

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

**Mise à jour du8 novembre2026 :** `M851 X0/Y−29/Z−2,15` est installé et relu
après flash (Marlin2.1.2.9 `Nov 8 2026 19:22:05`, 80500 octets vérifiés).
Z−2,15 est validé au papier par l’utilisateur au point précédent ; le recontrôle
au nouveau point sûr reste à faire. L’origine Orca corrigée est **−11x−17** et
le point sûr courant est sonde121/117, buse121/146. Les paragraphes de build et
coordonnées datés ci-dessous décrivent les étapes précédentes.

| Élément | Configuration |
|---|---|
| Machine | Anet A8 modifiée, **MKS Gen L V2.1**, `BOARD_MKS_GEN_L_V21`, ATmega2560 intégré |
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
| EEPROM | Activée ; réglages persistants avec M500 |
| Sonde et secours Z | Homing Z sur 3D Touch D11/D19 et Z_SAFE_HOMING installés ; Z−2,06 sauvegardé selon l'utilisateur ; chaîne NC à implémenter |

Le switch Z actuel a été signalé comme non fiable ; sa remise en état reste à
faire. Le `KILL_PIN` standard n’est pas une butée anticollision rapide validée.

Le matériel a été précisé par l’utilisateur après le premier commit : l’ancienne
déclaration `BOARD_RAMPS_13_EFB` est remplacée par la cible MKS dédiée. Les
broches des fonctions actives sont identiques, mais pas celles des servos :
SERVO0 est **D11** sur cette carte, SERVO1 **D21**. D7 pilote la sortie HE1,
il ne doit pas être utilisé par une sonde configurée comme sur RAMPS 1.3.

Câblage de sonde historique : commande/alimentation sur **J24 D11/5V/GND**,
détection sur **Z+ J8 S(D19)/GND**, en préservant Z− D18 pour le fin de course
ou la future chaîne NC. BLTOUCH et le homing sur sonde sont activés dans les
sources ; la surveillance NC reste à faire. Offsets de sonde **`{0,-29,0}`** :
X0 et Y−29 sont confirmés, **Z reste à calibrer**. Cette configuration est
téléversée et vérifiée le7 novembre2026 ; M503 confirme **M851 X0/Y−29/Z0**.

La correction d’origine Orca remesurée **X11/Y−17** est distincte de ces offsets
de sonde et reste dans le profil slicer ; ne pas les confondre ni les cumuler
avec une correction M206 sans réexaminer le repère machine.

Essais utilisateur : `M280 P0 S10` sort la pointe, `M280 P0 S90` la rentre,
`M119` lit `z_probe`. Le mode switch `M280 P0 S60`, si pris en charge, facilite
la lecture du déclenchement ; attendre une seconde entre commandes. `M401/M402`
ne conviennent pas à un essai strictement sans mouvement d’axes : le chemin
de déploiement Marlin peut dégager Z. Les essais exécutés doivent rester limités
à la sonde seule, selon l’autorisation utilisateur décrite plus bas.

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

Après correction de la cible MKS Gen L V2.1, compilation réussie avec les mêmes
tailles Flash/RAM. Comparaison des broches actives par préprocesseur AVR : moteurs,
fins de course mini, thermistances, hotend, plateau et ventilateur inchangés.
La correction de carte est une modification locale postérieure au commit initial.

Build des essais du5 novembre2026, BLTOUCH, M280 et diagnostic natif `PINS_DEBUGGING` :
**Flash79800/253952, RAM3320/8192**.
Contrôle du préprocesseur : servo0=D11, sonde=D19, Z−=D18, pull-up de sonde actif,
homing sur sonde désactivé.

**Installé et vérifié le 5 novembre 2026 sur COM5** : Marlin2.1.2.9,
compilation `Nov 5 2026 22:49:53`, 79800 octets écrits et vérifiés par avrdude,
signature ATmega2560 `0x1e9801`. M115 annonce Z_PROBE=1 et M503 conserve les
réglages principaux. Maintenance USB seule, sans12 V selon le contexte utilisateur.

M43 S a mesuré une impulsion de contact **8 ms ±4 ms**, compatible BLTouch
pré-V3.1. M119 réactive le mode switch dans cette version : pointe sortie `open`,
pointe rentrée `TRIGGERED` à la lecture. La lecture M119 seule n’avait pas capté
le contact bref. La pointe a été commandée rentrée en fin d’essai.
Après les irrégularités initiales, une série de cinq M43 S donne **5/5 contacts**,
largeurs10/8/10/12/12 ms (±4 ms), et l’utilisateur confirme **5 rappels nets**
au premier contact, sans aide ni aller-retour parasite. La répétabilité en hauteur
et le homing/palpage restent à valider. Aucun déplacement XYZ/E ni chauffe effectué.

**Configuration uniquement :** aucun changement sous `Marlin/src/`. Les fonctions
M280/M119/M43 S sont celles de Marlin ; seul PINS_DEBUGGING a été décommenté
pour rendre M43 S disponible. Ne pas utiliser M43 T dans ces essais de sonde.

Configuration installée le7 novembre2026 : offsets **X0/Y−29/Z0 provisoire**,
homing Z sur sonde et **Z_SAFE_HOMING** activés. Centre de sonde en coordonnées
firmware **99/117**, soit buse **99/146**. Calcul : centre de zone utile Orca220×200
=(110,100), moins correction Orca(11,−17), puis moins offset sonde(0,−29) pour la buse.
Vérification hors ligne des coordonnées et des courses réussie. Approches Z2 puis1 mm/s.
Build **Flash80452/253952, RAM3320/8192**, 80452 octets écrits puis vérifiés
sur COM5, signature `0x1e9801`. M115 : Marlin2.1.2.9, `Nov 7 2026 15:32:26`.
M503 confirme M8510/−29/0 et M2060/0/0 ; réglages principaux conservés.
Aucun homing ni mouvement d’axes exécuté par l’assistant. Pointe rentrée, port fermé.
Ne pas imprimer avec le nouveau homing avant calibration de Z ; M500 ne sauvegarde
pas cette calibration, qui doit être reportée dans les sources après validation.
Le homing est ensuite confirmé correct par l’utilisateur ; le Z-offset reste à mesurer.

## Intégrer une prochaine stable

1. Partir de `anet-a8-vaarai`, vérifier `git status` et synchroniser avec le fork
   par `git pull --ff-only`. L’arbre doit être propre avant intégration.
2. Vérifier la dernière **release stable officielle**, ses notes et son tag ;
   ne pas prendre automatiquement une branche de développement ou un tag mobile.
3. Récupérer le tag exact : `git fetch upstream tag <version>`.
4. Examiner `git diff HEAD <version> -- Marlin/Configuration.h
   Marlin/Configuration_adv.h`, les changements de format, les notes de migration,
   les pins MKS Gen L V2.1 et leur base RAMPS, le HAL AVR et les fonctions actives listées ci-dessus.
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

Connexion habituelle : **Raspberry Pi / OctoPrint**. Le contrôleur n’est branché
au **PC que pour la maintenance, sans alimentation 12 V**. C’est le contexte à
retenir à l’annonce d’une connexion PC, sauf contradiction. Le 5 V USB peut
encore alimenter la logique et la sonde ; la pointe n’est donc pas présumée inactive.

Les mouvements XYZ, homing, palpage avec déplacement d’axes, extrusion/rétraction
physique, chauffes, autotunes et lancements/reprises d’impression sont exécutés
exclusivement par l’utilisateur. L’assistant ne déclenche pas ces actions et
ne transmet pas de G-code d’impression à la machine.

Clarification utilisateur : **la 3D Touch seule peut être actionnée par l’assistant**,
sans mouvement XYZ/E ni chauffe. Utiliser les commandes directes de servo/état ;
M401/M402 ne sont pas équivalents, car leur routine peut dégager Z.

Après installation par l’utilisateur, les lectures `M115`, `M503`, `M119` et
`M900` servent à comparer la version, les réglages, les états des entrées et la
disponibilité du Linear Advance avant les essais physiques utilisateur.
