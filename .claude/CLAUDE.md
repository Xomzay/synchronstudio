# Synchronstudio — eigener Fork

Stand: 26.09.2026. Sprache mit dem Besitzer: Deutsch, kurz und direkt. Er ist Hobby-Synchronsprecher, kein Programmierer. Anleitungen immer Schritt für Schritt mit den genauen Knopfnamen (GitHub Desktop, github.com).

## Was das ist

Fork von `synchron-studio/synchronstudio` (Web-App zum gemeinsamen Synchronisieren von Szenen). Live unter `https://xomzay.github.io/synchronstudio/`. Veröffentlicht wird per GitHub Actions, Workflow „Deploy Pages (slim)“ (`.github/workflows/deploy-pages.yml`): führt `npm test` aus und lässt beim Veröffentlichen alle `scenes/**/*.mp4` weg. Die Videos lädt die Seite direkt aus dem Repo über jsDelivr (nur Dateien bis 20 MB). Größere Videos landen über `rawUrlFor()` beim Notfallweg GitHub Raw.

Updates vom Original kommen über „Sync fork“ auf github.com. Bei einem Konflikt NIEMALS „Discard commits“ — das würde alle eigenen Szenen und Patches löschen.

## Eigene Änderungen an `client.js` (müssen Syncs überleben)

Jede Abweichung vom Original ist ein möglicher Merge-Konflikt. Deshalb nur kleine, gezielte Patches. Neue Funktionen lieber als Wunsch an den Entwickler oder in PackWandler bauen, nicht hier.

1. **Adress-Erkennung** (`_GH_SEITE`, `_GH_BESITZER`, `_GH_PROJEKT`): `CDN_BASE` und `GH_RAW_BASE` zeigen auf den Fork statt aufs Original. Ohne das lädt die Seite die eigenen Szenen nicht.
2. **Gemeinsame Zeilen** (`roleOfLine`): Liefert die eigene Rolle aus `l.chars` statt immer `l.chars[0]`. Vorher schickte `finishBooth()` die Aufnahme einer gemeinsamen Zeile unter fremder Rolle ab → Premiere startete zu früh, dem anderen Spieler fehlten Zeilen. Betrifft auch die Originalszenen `chickenjockey` und `dexter_cargo`.
3. **Aufnahmestart** (Klick-Handler von `#btn-line-rec`): Spult schon während des Countdowns zu `l.t` und überspringt den Seek, wenn das Video schon dort steht. Vorher kam die Aufnahme erst Sekunden nach der „1“ oder brach ab.
4. **Originale vorladen** (`vorladenOriginale()`, Aufruf in `renderLine()`, `stallMs: 6000` in `getLineOrigBuffer()`): Holt die Originale der aktuellen und nächsten zwei Lines im Hintergrund. Vorher lange Sanduhr bei „Original anhören“ nach „Passt, weiter“, im schlimmsten Fall 30 s.

Alle vier sind dem Entwickler gemeldet. Übernimmt er einen davon, unseren Patch ersatzlos zugunsten seiner Version aufgeben.

## Eigene Szenen

Einträge mit `"packwandler": true` in `scenes-index.json` stammen vom Besitzer, erzeugt mit PackWandler. Szenen des Entwicklers nie löschen oder ändern.

Szenen bestehen aus drei Teilen, die exakt zusammenpassen müssen:
- `scenes.json` (volle Einträge)
- `scenes-index.json` (Kurzliste — Feldreihenfolge muss exakt dem entsprechen, was `tools/sync-scene-index.cjs` erzeugt)
- `scenedata/<id>.json` (Zeilen: `t`, `end`, `chars[]`, `who`, `text`, `orig`)

Zeilen dürfen mehrere Rollen in `chars` haben: Das sind gemeinsame Zeilen, beide sprechen gleichzeitig. Jede Zeile hat ihre Originalstimme als eigene kleine MP3 in `orig`.

`scenes-index.json` nie von Hand bearbeiten.

## Prüfen vor jedem Push

```
npm ci --ignore-scripts --no-audit --no-fund
npm test                                   # 59 Tests, müssen alle bestehen
node tools/sync-scene-index.cjs --check    # Index muss exakt passen, sonst veröffentlicht die Automatik nicht
node --check client.js
```

Bewährt für Verhaltensfehler: `client.js` in jsdom mit der echten `index.html` laden, Netzwerk und Medien stubben, und den Fehler erst mit der alten Version nachstellen, dann mit der neuen gegenprüfen.

## PackWandler

Eigenes Python-Programm des Besitzers (tkinter + ffmpeg), liegt NICHT in diesem Repo. Aktuell Version 6.4. Wandelt Choicer-Voicer-Packs, Editor-Exporte und Website-Exporte in eingebaute Szenen für diesen Fork um, auch mehrere auf einmal in ein gemeinsames Paket. Holt die aktuelle Szenenliste über die GitHub-API und hält jede Datei unter 95 MB.

## Ablauf beim Besitzer

1. GitHub Desktop: **Fetch origin**, dann **Pull origin**
2. PackWandler-Ordner: Inhalt kopieren, im Projektordner einfügen und ersetzen
3. **Commit to main** → **Push origin**
4. github.com → **Actions** → grüner Haken
5. Seite mit **Strg+Shift+R** neu laden

## Nie ins Repo

Zugangs-Token, `packwandler_einstellungen.json` (enthält den Token im Klartext) und andere Zugangsdaten. Das Repo ist öffentlich.
