# Changelog

## 0.7 – 10.10.2026

- Helligkeitsvariable optional: Bei „Kein(e)“ schalten passende Erkennungen unabhängig von der Helligkeit ein und verlängern die Nachlaufzeit. Der Lux-Schwellwert wird ignoriert.
- Mit ausgewähltem Sensor bleibt die Lux-Prüfung für Einschalten und Verlängern erhalten. Ungültige ausgewählte Sensoren bleiben Konfigurationsfehler.
- Formularhinweise, Dokumentation und Regressionstests ergänzt.
- Webhook-URL als aktives Textfeld statt anklickbarem Label anzeigen, damit sie direkt mit Strg+A und Strg+C kopiert werden kann.
- Das Feld ist nicht an eine Moduleigenschaft gebunden; Änderungen im Feld verändern den automatisch erzeugten Webhook nicht. Beim erneuten Öffnen erscheint wieder die erzeugte URL.
