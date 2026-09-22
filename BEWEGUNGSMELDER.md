# Zusätzlicher Bewegungsmelder

Basis: mb-stern/Reolink, Branch `beta`, Commit `4c5df91d79930c873a2019161ac3eff80ffc7993`.

Die Erweiterung ist direkt in der Reolink-Instanz konfigurierbar. Sie ist zunächst deaktiviert. Die vorhandenen Erkennungsvariablen, 5-Sekunden-Rücksetztimer, Schnappschüsse und Archive behalten ihre bisherige Logik.

## Einbau

Diese Erweiterung ist für das bestehende Reolink-Modul bestimmt. Für den parallelen Live-Test zum Store-Modul den Branch `test/reolink-v2` dieses Forks verwenden. Dieser Feature-Branch behält die Original-IDs und darf nicht parallel zur Store-Bibliothek installiert werden.

## Einstellungen

| Feld | Bedeutung |
| --- | --- |
| Zusätzlichen Bewegungsmelder aktivieren | Standardmäßig aus; aktiviert nur die neue Lichtsteuerung. |
| Mensch, Tier, Fahrzeug | Ein-/Aus-Schalter; die jeweiligen Variablen dieser Kamera werden automatisch gefunden. Jede Kombination ist möglich. |
| Schaltvariable | Boolean-Variable mit Standardaktion oder eigenem Aktionsskript. `true` schaltet ein, `false` aus. |
| Helligkeitsvariable | Integer- oder Float-Variable des Helligkeitssensors. |
| Einschalten unter Schwellwert | Nur Werte **kleiner** als dieser Wert erlauben Einschalten. Einheit entspricht dem Sensor, zum Beispiel Lux. Standard: 30. |
| Nachlaufzeit | Sekunden seit der letzten positiven Erkennung, 1–86400; Standard: 120. |

Die vorhandenen Variablen der eigenen Kamera können direkt ausgewählt werden. Dafür müssen die Bewegungsvariablen in der bestehenden Modulkonfiguration aktiviert bleiben. Alle gewählten Quellen wirken gemeinsam auf dieselbe Schaltvariable.

## Verhalten

- Eine positive Erkennung schaltet bei ausreichender Dunkelheit ein.
- Jede weitere positive Erkennung verlängert die Nachlaufzeit, auch wenn das eingeschaltete Licht den Helligkeitssensor aufhellt.
- Bei der eigenen Kamera werden Webhook-Erkennungen und alle positiven Polling-Antworten berücksichtigt, auch ohne Änderung des Boolean-Wertes.
- Die Erkennungsarten Mensch, Tier und Fahrzeug lassen sich einzeln ein- und ausschalten; ihre Kamera-Variablen werden automatisch gefunden.
- Negative Erkennungen und die bestehenden 5-Sekunden-Rücksetzungen schalten das Ziel nicht aus und verlängern die Nachlaufzeit nicht.
- Der eigene Timer prüft die Frist jede Sekunde. Das Ausschalten erfolgt bei der nächsten Ausführung nach Fristablauf; Symcon-Auslastung kann dies verzögern.
- Helligkeitsänderungen allein schalten nicht ein; es ist eine neue Erkennung erforderlich.
- Bereits vor der Erkennung eingeschaltete Ziele werden nicht übernommen und deshalb auch nicht durch diese Automatik ausgeschaltet.
- Wird ein automatisch eingeschaltetes Ziel während der Nachlaufzeit manuell bedient, hat diese Bedienung keine separate Vorranglogik: Die geplante Ausschaltung bleibt bestehen. Pro Ziel sollte nur eine solche Automatik zuständig sein.
- Deaktivieren der Automatik oder der Instanz, ungültige Konfiguration und Wechsel der Zielvariable schalten das bisher von der Automatik eingeschaltete Ziel aus.
- Eine geänderte Nachlaufzeit wird ab der letzten gespeicherten Erkennung berechnet. Ein Symcon-Neustart erhält die ausstehende Ausschaltung.
- Bei Fehlern beim Ausschalten wird nach fünf Sekunden erneut versucht. Fehlermeldungen erscheinen im Debug der Instanz.

## Änderungen am bestehenden Modul

Die neue Datei `MotionLighting.php` kapselt Einstellungen, Formular, Variablenmeldungen und Timer. `module.php` bindet sie ein und ergänzt Aufrufe bei Erstellung, Konfigurationsübernahme, Formularaufbau sowie Webhook und Polling. `SetMoveTimer()` und `ResetMoveTimer()` sind unverändert.

Schalten erfolgt über Symcons `RequestAction`, damit die hinterlegte Geräteaktion ausgeführt wird: [Symcon-Dokumentation](https://www.symcon.de/de/service/dokumentation/befehlsreferenz/variablenzugriff/requestaction/).

## Prüfung

PHP 8.3.31: Syntaxprüfung beider Moduldateien bestanden. 49 automatisierte Prüfungen mit simulierten Symcon-Funktionen bestanden, einschließlich Webhook und Polling im tatsächlichen Modul, Nachtriggern, Schwellenwert, unverändertem 5-Sekunden-Reset, Erkennungskombinationen und Zielwechsel, Neustart und Aktionsfehlern.

Ausführen mit PHP 8.2 oder neuer:

```sh
php -l REOCAM/module.php
php -l REOCAM/MotionLighting.php
php tests/motion-lighting.php
```

Ein Live-Test mit Symcon und Kamera steht aus. Dafür zunächst eine Test-Schaltvariable mit Aktion verwenden: Nachlaufzeit kurz einstellen, Erkennung mehrfach auslösen, Ausschaltung ab letzter Erkennung prüfen und anschließend Helligkeit oberhalb des Schwellwerts testen.
