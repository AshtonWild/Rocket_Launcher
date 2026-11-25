# Manufacturing

Both boards are sized and ruled for JLCPCB's standard 2-layer service. Any fab with similar limits will work.

## Files

| Board | Gerbers | Parts list |
|---|---|---|
| Rocket V1 | `Rocket_V1/JLCPCB/Rocket_V1_JLCPCB_gerbers.zip` | `Rocket_V1/JLCPCB/Rocket_V1_BOM.csv` |
| Ground Station V1 | `Ground_Station_V1/JLCPCB/Ground_Station_V1_JLCPCB_gerbers.zip` | `Ground_Station_V1/JLCPCB/Ground_Station_V1_BOM.csv` |

Upload the zip as-is. It holds the copper, solder mask, silkscreen and outline layers plus the drill files.

## Order settings

| Setting | Value |
|---|---|
| Layers | 2 |
| Thickness | 1.6 mm |
| Copper weight | 1 oz |
| Surface finish | Lead-free HASL |
| Everything else | Defaults |

Keep the copper at 1 oz or heavier. The battery, servo and igniter tracks (1.2–1.5 mm wide) were sized for it.

## What the boards need from the fab

| | Rocket V1 | Ground Station V1 | JLCPCB minimum |
|---|---|---|---|
| Board size | 70 × 50 mm | 100 × 70 mm | |
| Narrowest track | 0.15 mm | 0.2 mm | 0.127 mm |
| Via (hole / pad) | 0.3 / 0.6 mm | 0.3 / 0.6 mm | 0.3 / 0.6 mm standard |
| Smallest copper ring around a hole | 0.35 mm | 0.15 mm | 0.13 mm |
| Copper to board edge | 0.5 mm | 0.5 mm | 0.3 mm |
| Silkscreen text | 1.0 mm tall, 0.15 mm line | 1.0 mm tall, 0.15 mm line | 1.0 mm, 0.15 mm |

## Assembly

Every part is through-hole, so the boards are meant to be hand-soldered. No pick-and-place files are provided.

## Before you order

- Read the "Known issues" section of [rocket-v1.md](rocket-v1.md) and [ground-station-v1.md](ground-station-v1.md).
- Check the XT30 polarity on Rocket V1 against your battery lead.
- Check your GPS, microSD and IMU module pin orders against the headers.

## Regenerating the files

After changing a board, re-export from the project folder with KiCad's command-line tool:

```sh
kicad-cli pcb export gerbers --layers F.Cu,B.Cu,F.Paste,B.Paste,F.Silkscreen,B.Silkscreen,F.Mask,B.Mask,Edge.Cuts \
  --subtract-soldermask --no-x2 --use-drill-file-origin -o JLCPCB/gerbers/ <board>.kicad_pcb
kicad-cli pcb export drill --format excellon --excellon-units mm --excellon-separate-th -o JLCPCB/gerbers/ <board>.kicad_pcb
```

Then zip the contents of `JLCPCB/gerbers/`. Run `kicad-cli pcb drc` first and fix any errors.
