# Projet : Flight Controller FPV custom (KiCad)

## Vue d'ensemble
Contrôleur de vol FPV custom conçu sous KiCad, basé sur STM32H743VIT6 (LQFP100).
Architecture en feuilles hiérarchiques sous une feuille racine `FPV_drone.kicad_sch` :
- `Power_functions.kicad_sch` — étage d'alimentation
- `FPV_brain.kicad_sch` — MCU + périphériques numériques
- `Fpv_telemtry.kicad_sch` — IMU, baromètre, flash SPI, capteur de courant
- `Analog_video.kicad_sch` — chaîne vidéo analogique (OSD)
- `Mosfet_driver.kicad_sch` — sous-feuille référencée, contenu non encore audité

## Composants clés
- MCU : STM32H743VIT6 (LQFP100) — envisage un passage en TFBGA100 (STM32H743VIH6)
- Double IMU : 2× ICM-42688-P (redondance)
- Baromètre : SPA06-003
- Flash SPI : W25Q128JVPIQ
- Capteur de courant : INA186A1IDCKR + shunt 0,5 mΩ/3W
- OSD : AT7456E
- Buck converters : 2× DCP3603BCMR (buck 1 → 5V, buck 2 → 9V)
- LDO : 2× AP2112K-3.3
- Power mux : TPS2121RUXT
- Protection USB : USBLC6-2SC6 ; protection surtension entrée : TVS SMBJ33A
- Connecteur batterie : XT30/XT60 (protection anti-inversion assurée mécaniquement, pas de P-MOSFET dédié)

## Vérifications déjà faites (calculs confirmés)
- Buck 1 (IC5) : R14=73,2kΩ / R15=15kΩ → VOUT calculé = 5,00V (config exactement conforme à la reco ST)
- Buck 2 (IC8) : R22=143kΩ / R23=15kΩ → VOUT calculé ≈ 8,95V (~9V, cohérent avec le label)
- Bobines des bucks : Würth 74438367100, 10µH, 3,3A assigné — cohérent avec le 3A max du DCP3603, marge de ripple correcte
- Ferrite FB1 (BLM18BD421SZ1D, ~420Ω@100MHz, ~200-250mA assigné) : adaptée à un rail bas courant type VDDA, pas à une ligne haute consommation
- Cristal Y1 (ABM3B-8.000MHZ-10-D1G-T, CL nominal = 10pF) : C1=C2=12pF donnent CL≈9-10pF avec la capa parasite typique de PCB — bon calcul, cohérent avec la spec du cristal
- Décimalage STM32H743 : 9×100nF + 2×2,2µF (VCAP1/VCAP2) + 1×1µF — cohérent avec les besoins du boîtier LQFP100

## Points à corriger / vérifier
1. **LDO — priorité haute** : IC4 (VOUT) et IC10 (VOUT) portent tous les deux le label `+3.3V`. Intention confirmée : IC4 alimente MCU+capteurs, IC10 doit alimenter *seulement* le 2e IMU sur un rail propre séparé. Il faut renommer un des deux nets (ex. `+3.3V_IMU2`) pour que la séparation soit électriquement réelle — actuellement les deux LDO sont en parallèle sur le même nœud.
2. **Fpv_telemtry.kicad_sch — découplage manquant** (l'utilisateur sait que ce fichier est en cours) : U3/U4 (ICM-42688-P), U1 (SPA06-003), IC9 (W25Q128JVPIQ) n'ont aucun condensateur de découplage local. Seul C36 (100nF) existe, à côté de l'INA186 (IC6). À faire : 100nF sur VDD + 100nF sur VDDIO pour chaque IMU + condensateur dédié sur la broche REGOUT (vérifier valeur exacte dans la datasheet ICM-42688-P, table application schematic), 100nF standard pour le baro et le flash.
3. **Analog_video.kicad_sch** : à ce stade contient uniquement le symbole AT7456E posé sans aucun câblage (pas d'alim, pas de découplage, pas de connexion SPI/vidéo). Reste entièrement à concevoir.
4. **Power symbols mal nommés** : `#PWR051` utilise le symbole `power:+5V` mais affiche `+9V` en valeur — fonctionnellement correct mais trompeur à la lecture. À nettoyer avec des symboles dédiés par rail.
5. **TFBGA100 (si migration confirmée)** : sur ce boîtier, VREF+ n'est PAS sorti séparément — bondé en interne sur VDDA, buffer de référence interne à garder désactivé. Vérifier que le schéma actuel ne dépend pas d'un VREF+ séparé avant de relayouter. Référence composant à changer : STM32H743VIT6 → STM32H743VIH6.
6. Pas de fichier `.kicad_pro` racine fourni à ce jour — feuilles analysées en tant que fichiers `.kicad_sch` individuels + `FPV_drone.kicad_sch`.

## Environnement utilisateur
- Utilise Claude Code en local sur un terminal Fedora Linux.
- KiCad pour la conception (schématique + PCB), fabrication DIY (photolithographie UV, gravure au perchlorure de fer) et services pro (JLCPCB).
