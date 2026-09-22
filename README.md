# Reolink V2 – Testmodul für Symcon

Separate Testversion auf Basis von [mb-stern/Reolink](https://github.com/mb-stern/Reolink), Branch `beta`, Commit `4c5df91d79930c873a2019161ac3eff80ffc7993`. Original von Stefan Künzli, MIT-Lizenz unverändert enthalten. Dies ist keine offizielle neue Version des Store-Moduls.

Zusätzlich zur Kameraerkennung gibt es eine konfigurierbare Lichtsteuerung: Ein-/Aus-Schalter für Mensch, Tier und Fahrzeug (eigene Kamera-Variablen automatisch), eine Boolean-Schaltvariable mit Aktion, eine numerische Helligkeitsvariable, Schwellwert und Nachlaufzeit ab letzter Erkennung. Die bisherigen 5-Sekunden-Reset-Timer bleiben unverändert. Jede positive Erkennung unterhalb des Lux-Schwellwerts verlängert den zusätzlichen Timer; auch bei laufendem Licht wird die Helligkeit vor jeder Verlängerung geprüft.

## Parallel zum Store-Modul installieren

1. In Symcon **Kerninstanzen → Modules / Module Control** öffnen und folgende Repository-URL hinzufügen:

   ```text
   https://github.com/Sol-IoTiv/Reolink.git
   ```

2. Prüfen, dass der Branch **`test/reolink-v2`** gewählt ist (Standardbranch dieses Test-Forks). Nur dieser enthält die parallel installierbare V2; nicht auf `main`, `beta` oder `feature/bewegungsmelder` wechseln, solange das Store-Modul installiert ist.
3. Eine neue Instanz **Reolink V2 (Test)** anlegen. Das bisherige Store-Modul und dessen Instanzen bleiben installiert.
3. In der neuen Instanz Kamera-Zugangsdaten eintragen. Für einen ersten parallelen Test **Polling** aktivieren. Den vorhandenen Kamera-Webhook zunächst unverändert lassen; er versorgt weiterhin die Originalinstanz. Bei Nutzung des V2-Webhooks muss der in der neuen Instanz angezeigte Pfad verwendet werden; falls die Kamera nur ein Ziel unterstützt, erhält die Originalinstanz danach keine Webhook-Meldungen mehr.
4. In **Bewegungsmelder / Lichtsteuerung** zunächst den Schalter **Mensch als Auslöser verwenden** einschalten, außerdem Helligkeitsvariable und ein ausgeschaltetes Boolean-Schaltziel mit Standardaktion/Aktionsskript. Die Bewegungsvariablen müssen in der Instanz aktiviert bleiben.
5. Nachlaufzeit auf 10 Sekunden einstellen, Schwellwert zum Test über den aktuellen Sensorwert setzen, zusätzliche Automatik aktivieren und Änderungen übernehmen.
6. Erkennung auslösen, dann wiederholen: Ausschaltung erst nach Ablauf der Nachlaufzeit seit der letzten Erkennung. Anschließend bei ausgeschaltetem Ziel den Schwellwert unter den Sensorwert setzen: Eine neue Erkennung darf nicht einschalten.

Es müssen keine Dateien der Store-Installation ausgetauscht werden. Bestehende Kamera-Einstellungen/Objekt-IDs werden nicht automatisch in die neue Instanz übernommen. Zwei Instanzen mit Polling/API-Abfragen erzeugen zusätzliche Kamera-Anfragen; für den Test unnötige API-Abfragen der V2-Instanz deaktivieren. Nicht zwei Lichtautomatiken dasselbe Ziel steuern lassen.

## Verhalten und Grenzen

- Schwellwert gilt strikt: Helligkeit muss kleiner sein. Einheit entspricht dem Sensor.
- Mindestens eine Erkennungsart einschalten; jede Kombination ist möglich. Alle drei Schalter sind nach dem Update zunächst aus. Mensch, Tier und Fahrzeug wirken gemeinsam auf ein Ziel.
- Eigene Webhook- und positive Polling-Erkennungen verlängern auch ohne Boolean-Zustandswechsel. Es werden automatisch die Variablen der eigenen Kamera verwendet.
- Bereits eingeschaltete Ziele werden nicht übernommen. Manuelle Bedienung während einer automatisch begonnenen Nachlaufzeit hebt die geplante Ausschaltung nicht auf.
- Der zusätzliche Timer prüft jede Sekunde. Last und Aktionslaufzeit können die tatsächliche Ausschaltung verzögern.
- Deaktivieren, ungültige Konfiguration oder Zielwechsel beenden eine eigene laufende Lichtsteuerung. Ausstehende Ausschaltungen bleiben bei Neustart erhalten; fehlgeschlagene Ausschaltaktionen werden nach fünf Sekunden wiederholt.
- Der Live-Test in Symcon steht noch aus. Bei Fehlern Instanz-Debug prüfen; keine Zugangsdaten in öffentliche Issues posten.

## Technische Trennung und Tests

Für parallele Installation wurden Bibliotheks-ID, Modul-ID, Klassenname, Trait-Name, Funktionspräfix (`REOCAMV2`), Variablenprofile und Webhook-Präfix (`reolink_v2_`) getrennt. Die eigentliche Erweiterung liegt in `REOCAMV2/MotionLighting.php`.

```sh
php -l REOCAMV2/module.php
php -l REOCAMV2/MotionLighting.php
php tests/motion-lighting.php
```

Die Tests simulieren Symcon-Funktionen. Sie ersetzen keinen Test mit Symcon und Kamera.

Nach erfolgreichem Test soll ausschließlich die Bewegungsmelder-Erweiterung gegen das Originalprojekt vorgeschlagen werden. Die V2-Umbenennungen gehören nicht in diesen späteren Pull Request. Es wurde noch kein Pull Request an mb-stern erstellt.


## Statusanzeige

Die Boolean-Variable **Bewegungsmelder aktiv** ist eine reine Anzeige ohne Schaltaktion. Sie ist An, wenn der Hauptschalter und mindestens eine Erkennungsart eingeschaltet sind. Sie zeigt die gespeicherte Auswahl nach Änderungen übernehmen; keine Bewegung, keinen Lichtzustand und keine Prüfung der Betriebsbereitschaft. 85 automatisierte Prüfungen mit simulierten Symcon-Funktionen bestanden, einschließlich aller Schalterkombinationen.


## Hauptschalter der Lichtsteuerung

**„Zusätzlichen Bewegungsmelder aktivieren“ ist der Hauptschalter.** Nur wenn dieser eingeschaltet ist, darf die neue Automatik das Licht schalten. Die Schalter Mensch, Tier und Fahrzeug legen zusätzlich fest, welche Erkennungsarten auslösen dürfen. Nach dem Einstellen **Änderungen übernehmen**.

Ist der Hauptschalter aus, bleiben die normale Kamera-Erkennung und die bisherigen Erkennungsvariablen aktiv; die zusätzliche Lichtsteuerung ist ausgeschaltet. Die Statusvariable „Bewegungsmelder aktiv“ zeigt An, wenn der Hauptschalter und mindestens eine Erkennungsart eingeschaltet sind. Sie zeigt die Aktivierung, nicht den aktuellen Lichtzustand.

## Beispiel: smarter Bewegungsmelder für die Hauseinfahrt

Das Einfahrtslicht soll bei Menschen oder Fahrzeugen einschalten, aber nicht bei einer als Tier erkannten Katze oder einem Hund.

| Einstellung | Beispiel |
| --- | --- |
| Zusätzlichen Bewegungsmelder aktivieren | An |
| Mensch als Auslöser verwenden | An |
| Tier als Auslöser verwenden | Aus |
| Fahrzeug als Auslöser verwenden | An |
| Schaltvariable | Einfahrtslicht: Boolean-Variable mit Geräteaktion |
| Helligkeitsvariable | Außen-Helligkeitssensor in lux |
| Einschalten unter Schwellwert | 30 lux, als anpassbarer Startwert |
| Nachlaufzeit | 120 Sekunden |

Unter 30 lux schaltet eine erkannte Person oder ein erkanntes Fahrzeug das zuvor ausgeschaltete Einfahrtslicht ein. Jede weitere passende Erkennung unterhalb des Lux-Schwellwerts startet die Nachlaufzeit erneut. Nach 120 Sekunden ohne weitere passende Erkennung sendet die Automatik false an die Schaltaktion.

Eine ausschließlich als Tier gemeldete Erkennung schaltet nicht ein und verlängert die Nachlaufzeit nicht. Die Tier-Erkennungsvariable der Kamera funktioniert trotzdem weiter. Die Unterscheidung hängt von der Klassifizierung der Kamera ab: Fehlklassifizierungen sind möglich; werden gleichzeitig eine Person oder ein Fahrzeug erkannt, darf das Licht einschalten. Es ist daher keine Garantie, dass Katzen oder Hunde unter allen Umständen ausgeschlossen werden.


## Helligkeit während der Nachlaufzeit (ab Testversion 0.6)

Der Lux-Schwellwert gilt sowohl zum Einschalten als auch zum Verlängern. Beispiel: Bei 20 lux wird eingeschaltet. Steigt die Helligkeit auf mindestens 30 lux, verlängern neue Erkennungen die laufende Nachlaufzeit nicht mehr. Das Licht geht nach Ablauf der Zeit seit der letzten passenden Erkennung unterhalb von 30 lux aus – auch bei fortlaufender Personen- oder Fahrzeugerkennung. Die Lux-Überschreitung startet keine neue Frist und schaltet nicht sofort aus. Erst unterhalb von 30 lux dürfen passende Erkennungen wieder einschalten oder verlängern. Dies gilt auch, wenn das eingeschaltete Licht selbst den Sensor aufhellt.
