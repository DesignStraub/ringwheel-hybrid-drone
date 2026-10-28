# Elektronik Hybrid-Drohne

## Aufbau

```
Akku 4S ──XT60──┬── 4-in-1-ESC (Stack) ── 4× Motor 2306
                │        ▲ 8-pol. SH (M1–M4, VBAT, CUR, TLM)
                │        │
                │   ┌────┴──────────── Hybrid-Controller HC-1 ───────────────┐
                │   │ J4 → ESC   U6 SN74CBTLV3257: S=0 FC (Flug), S=1 ESP32 (Fahrt) │
                │   │ J3 ← FC    J5 FC-UART (MSP: RC-Kanäle + Arm-Status)          │
                └───┤ J1 VBAT    U1 Buck 6 V → 2× DRV8833 (Klappmotoren)            │
                    │            U2 3,3 V → ESP32-S3, Sensoren                     │
                    │            Q1–Q4 Hubmagnet-Treiber (PWM)                      │
                    └── J6–J9 (GH-8) ── je Ecke: Eckplatine EP-1 ──┐               │
                                         AS5600 Gelenkwinkel        │ GH-5 durchs Gelenk
                                         N20-Klappmotor             └─ Hall AH1806 (Spulenbohrung)
                                                                       + Hubmagnet-Spule
Flugsteuerung (F722) ◄── ELRS-Empfänger
```

## FPV (DJI O4 Air Unit Lite)

- **Kamera:** in der Spitze der Keilnase (ab v12), 15° nach oben geneigt. Sie sitzt bündig in einem Fenster von 14 × 13 mm, das senkrecht zur Kameraachse steht, und wird von zwei Wangen (1,6 mm, lichter Abstand 14,2 mm) gehalten. Befestigt wird sie im mitgelieferten 14-mm-Kameraadapter (14 × 20 mm, je Seite zwei M2-Löcher im Abstand von 16 mm, laut Testbericht nicht vorgeschnitten, selbstschneidende Schrauben liegen bei). Jede Wange hat dafür zwei Langlöcher 2,2 × 5 mm entlang der Kameraachse, 4–9 mm hinter der Nasenspitze. Damit lässt sich die genaue Lochlage am echten Teil ausgleichen, denn die Quellen nennen unterschiedliche Kameramaße. Die Schrauben erreicht man durch vier Zugangsbohrungen Ø 6 in den Nasenseiten. Bei Regen am besten mit Klebeband abdecken. Die 15° sind ein Kompromiss aus Flug, wo die Drohne nach vorne kippt, und Fahrt, wo der Rumpf waagerecht liegt. Nase, Rumpf und Heck bilden einen gemeinsamen Innenraum, das Kabel läuft direkt zum Sender.
- **Videosender:** außen auf dem Deckel, damit er Luft bekommt. Das Kabel führt durch die Bohrung im Deckel, die Antenne steht senkrecht.
- **Strom:** vom 9-V- bzw. 10-V-Ausgang der Flugsteuerung. **Nicht direkt an den 4S-Akku**, die O4 verträgt maximal 13,2 V.
- **Daten:** ein freier UART der Flugsteuerung für die Bildschirmanzeige (MSP DisplayPort). Der Hybrid-Controller nutzt einen anderen UART.
- Das Videobild läuft in beiden Modi. Beim Fahren ist die Flugsteuerung entschärft, liefert aber weiter Bildschirmanzeige und RC-Kanäle.
- **Später möglich:** Der freie Pin IO26 des ESP32 (Testpunkt TP3) kann einen Mini-Servo ansteuern. Damit ließe sich die Kamera beim Umschalten automatisch neigen, z. B. 25° im Flug und 5° beim Fahren.

## Fahrgetriebe und Kupplungssensor (ab v13)

- **Getriebe:** In jedem Radmodul sitzt eine Planetenstufe 1:4 (Modul 0,7, Sonne 24, drei Planeten 24, Hohlrad 72 Zähne). Die Sonne läuft auf dem Achsrohr. Das Hohlrad ist mit seinem Boden fest auf das Achsrohr geklebt, der Planetenträger ist die Radnabe.
- **Axiale Sicherung (ab v15):** Der Planetenträger hat eine Schürze mit Innenlippe, die beim Aufstecken unter einen Bund am Hohlrad schnappt (0,5 mm Überdeckung, 0,2 mm Spiel). Damit kann das Rad nicht abziehen, auch nicht bei Seitenkräften beim Fahren. Die Sonne liegt oben am Träger an und ist so mitgesichert.
- **Kupplung:** Die Schaltmuffe treibt die Sonne über drei Mitnehmerfinger an und gleitet darauf 4,5 mm. Oben greift sie wie bisher in die Klauen der Motorglocke (Fahrt). Unten rastet sie in die Klauen des Planetenträgers (Flug). Dann steht das Getriebe als Block, und das Rad ist arretiert.
- **Hall-Sensor:** Der Muffensensor MS-1 passt nicht mehr, außen um die Muffe drehen jetzt Träger und Speichen. Ein einzelner AH1806 sitzt unter der Motorplatte in der Bohrung der Hubmagnet-Spule. Er sieht einen Ringmagneten (NdFeB, ca. 16 × 12,5 × 1 mm) oben auf der Muffennabe. Bei gekuppelter Muffe ist der Abstand 0,5 mm, im Flug 5 mm. Drei Litzen (VCC, GND, OUT) und 100 nF direkt am Sensor laufen durch ein Loch im Achsrohr zur Eckplatine. Stecker und Belegung an der Eckplatine bleiben gleich, die zwei Spulenadern laufen im selben Kabel mit. Auf MS-1 saßen nur Stecker, AH1806 und 100 nF, es fällt also keine Schutzbeschaltung weg.
- **Achtung, am Prüfstand testen:** Wenn die Spule bestromt ist (Fahrt), kann ihr eigenes Feld den Sensor auslösen. Eindeutig ist die Rückmeldung deshalb nur bei abgeschalteter Spule. Für die Sicherheit reicht das, denn vor dem Scharfschalten im Flug ist die Spule immer aus. Ob die Muffe beim Fahren wirklich eingerastet ist, prüft die Firmware zusätzlich über die Motordrehzahl (bidirektionales DShot): kurzer Gasstoß, die Drehzahl muss zum Rad passen.

## Projekte

| Ordner | Platine | Lagen | Größe | Stück |
|---|---|---|---|---|
| `Hybrid-Controller` | Steuerung Flug/Fahrt | 4 | 62 × 42 mm | 1 |
| `Eckplatine` | Gelenkwinkel + Kabelverteilung | 2 | 24 × 24 mm | 4 |

Fertigungsdaten (Gerber-ZIP, Stückliste, Bestückungsdatei für JLCPCB) liegen in `Fertigung/`, 3D-Modelle in `STEP/`. Der frühere Muffensensor MS-1 entfällt ab v13 (siehe Fahrgetriebe) und ist deshalb nicht mehr in diesem Repo.

Zum Öffnen braucht man **KiCad 10**.

**Stand Routing:** Beide Platinen sind vollständig geroutet. DRC-Ergebnis: 0 Fehler, 0 offene Verbindungen und 0 Abweichungen zum Schaltplan. Übrig sind nur Hinweise zum Bestückungsdruck, die JLCPCB beim Fertigen automatisch freischneidet.

Der Hybrid-Controller wurde mit Freerouting 2.5 geroutet, danach automatisch nachbearbeitet:
- Restverbindungen nachgezogen
- Masse-Vias zur GND-Ebene gesetzt (Stitching)
- doppelte Vias entfernt

Lagen: oben, In2 und unten für Signale, In1 als durchgehende GND-Ebene, auf allen Lagen zusätzliche GND-Flächen.

Regeln (JLCPCB-Standard für 4 Lagen):
- Signale 0,15 mm, Mindestabstand 0,127 mm, an Pads verjüngt auf 0,09 mm
- 6 V / Akku 0,4 mm, Motor und Spulen 0,3 mm
- Vias 0,5/0,25 mm, Masse-Vias teils 0,4/0,2 mm

Das Routing ist automatisch entstanden. Vor der Bestellung lohnt sich ein Blick in KiCad, vor allem auf USB und die Leistungspfade.

## Sicherheitslogik (für die Firmware)

1. Nach dem Einschalten: S = 0, die FC steuert die Motoren. Die DRV8833-Treiber schlafen, die Hubmagnete sind aus. Die Hardware erzwingt das über Pull-downs.
2. Fahrmodus nur, wenn die FC per MSP **entschärft** meldet und alle vier Hall-Sensoren **gekuppelt** melden. Ab v13 zusätzlich die Drehzahlprobe über DShot (siehe Fahrgetriebe), weil die bestromte Spule den Hall-Sensor beeinflussen kann.
3. Klappen nur im Stillstand und entschärft. Ab v14 begrenzen feste Anschläge an jeder Gabel den Weg auf 0° bis 90°. Der N20 fährt gegen den Anschlag, bis der AS5600 die Endlage meldet (0° bzw. 90° plus Toleranz) oder der Strom am DRV8833 ansteigt. Dann wird er abgeschaltet, die selbsthemmende Schnecke hält die Lage. Stöße beim Landen gehen so in den Anschlag statt in die Zähne.
4. Fahrt: Erst DShot 0 vom ESP32 senden, dann S = 1 schalten. Danach Drehrichtung per DShot-Befehl 20/21 im Stand setzen und schließlich Gas geben.
5. Zurück zum Flug: Motoren stoppen, S = 0, Kupplung lösen (Hall prüfen), hochklappen. Erst danach darf die FC scharf schalten.

Der Umschalter U6 wird über einen eigenen 3,3-V-Regler aus den 5 V der FC versorgt. Fällt der ESP32 aus, bleibt der Flugpfad deshalb erhalten.

## Belegung ESP32-S3-MINI-1

| GPIO | Funktion | GPIO | Funktion |
|---|---|---|---|
| 1, 2, 4, 5 | Winkel VL, VR, HL, HR (ADC) | 35, 36, 37, 42 | Hall VL, VR, HL, HR |
| 6 | Akkuspannung (100k/10k) | 7, 48, 33, 34 | Hubmagnet VL, VR, HL, HR |
| 8–11 | DShot M1–M4 (RMT) | 12 | Umschalter S |
| 13–16 | DRV8833 links (VL/HL) | 38–41 | DRV8833 rechts (VR/HR) |
| 17 / 18 | UART zur FC (TX / RX) | 21 / 47 | DRV nSLEEP / nFAULT |
| 3 | Status-LED | 19 / 20 | USB |

## Hinweise

- **Motorpins an den Ecksteckern:** Bei VL, VR und HR sind Pin 7 und 8 (N20-Motor) absichtlich vertauscht. So laufen die Leiterbahnen kreuzungsfrei. Die Drehrichtung dieser drei Klappmotoren invertiert die Firmware.
- **USB-D+:** Die beiden D+-Pads der USB-C-Buchse (A6, B6) werden über den Schutz-IC USBLC6-2 gebrückt, dessen Pins 1 und 6 intern verbunden sind (Durchschleif-Layout). D− ist direkt gebrückt.
- **USB-C sitzt an der linken Platinenkante.** Ab v15 hat die linke Seitenwand dafür einen Durchbruch von 13 × 7,5 mm mit R2-Ecken, mittig vor der Buchse (x −64,5 mm, z 54,9 mm) und unterhalb des hinteren Arms. Er ist groß genug für einen normalen USB-C-Stecker mit Knickschutz.
- **Vias in Pads:** Unter dem ESP32-Modul (GND-Pads), am GND-Pad von C24 und am GND-Pin von U3 sitzt je ein Via im Pad. Bei JLCPCB dafür am besten „Via-in-Pad (Epoxy gefüllt, abgedeckt)“ wählen, sonst kann Lot in die Bohrung abfließen.

- Die Belegung von J3 und J4 (ESC-Kabel) hängt vom Hersteller ab. Vor der Bestellung mit dem gekauften Stack abgleichen und notfalls die Reihenfolge im Schaltplan tauschen.
- Die Klappmotoren sind per DRV8833 auf ca. 1 A begrenzt (0,2-Ohm-Shunts).
- Die Eckplatine ist mit einem eigenen Gitter-Router entflochten, der Hybrid-Controller mit Freerouting plus Nachbearbeitung. Vor der Bestellung in KiCad öffnen, den DRC laufen lassen und die Leiterbahnen auf Optik und kurze Wege prüfen.
- Die Drehung der Bauteile in der Bestückungsdatei in der JLCPCB-Vorschau prüfen, besonders bei ICs und Steckern.
