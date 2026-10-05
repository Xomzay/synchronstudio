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
5. **Mehr Outtakes** (05.10.2026): `OUTTAKE_MAX` 8 → 30 pro Spieler, `OUTTAKE_POOL_MAX` 24 → 100, neu `OUTTAKE_POOL_MAX_BYTES` (6 MB) und `begrenzeOuttakePool()` statt `.slice(0, OUTTAKE_POOL_MAX)` in `ingestOuttakesFromPlayer()` und `publishOuttakesPool()`. Vorher fielen alle Fehlversuche über 8 pro Spieler bzw. 24 insgesamt weg. Die Größenbremse verhindert, dass das Outtakes-Paket an alle die Leitung verstopft.
6. **Noise Gate passt sich der Aufnahme an** (05.10.2026, `applyGateToBuffer()`): Die Schwelle war fest (`gateAmount * 0.14`). Bei leisen Mikros (dynamisches Fifine K688 des Besitzers) lag normale Sprache darunter und wurde um ca. 23 dB abgesenkt — man hörte sich nur mit dem Mund direkt am Mikro. Jetzt: `threshold = min(feste Schwelle, max(95.-Perzentil × 0,7 × gate, 10.-Perzentil × (2 + 2 × gate)))`. Die Schwelle ist nie höher als vorher, das Gate kann also nie mehr wegschneiden als im Original. Der Sprach-Schutz (`speechProtect`) bleibt bewusst an der festen Schwelle (`schutzUntergrenze`), sonst hält er bei leisen Mikros Rauschen für Sprache. Geprüft: laute Mikros und Gate aus bit-identisch zum Original, reines Rauschen wird weiter genauso gedämpft. Das Gate-Lämpchen in der Booth (`startGateLoop`, rein visuell) ist unverändert.
7. **Mikro-Hinweis** (05.10.2026, `mikroHinweis()`, Aufruf in `initMicScreen()` und im `onchange` von `#mic-device`): Erkennt am Gerätenamen Bluetooth-Freisprech-Mikros („Hands-Free“, „Freisprech“) und Laptop-/Webcam-Mikros („Array“, „Webcam“, „Integriert“ …) und zeigt im Mikro-Setup einen Hinweis. Ändert nichts am Ton. Hintergrund: Ein Mitspieler klang in Discord klar, auf der Seite dumpf und weit weg.
8. **Mehr Fun Facts** (05.10.2026): `FORK_FUN_FACTS()` mit 50 zusätzlichen Fakten (Stimme, Synchron, Anime, Games), `naechsterFunFact()` mischt alle (Original + Fork) und zeigt jeden einmal, bevor neu gemischt wird. `FUN_FACTS()` des Originals bleibt unangetastet, geändert ist nur der Inhalt von `rotateFunFact()`.
9. **Mikro-Check** (05.10.2026, `mikroCheck(buf)` im Klick-Handler von `#btn-mic-record`): Wertet die 3-Sekunden-Testaufnahme aus (95.-/10.-Perzentil des Pegels, Spitzenwert) und meldet „fast nichts angekommen“, „übersteuert“, „zu leise“, „viel Rauschen“ oder „Pegel passt“. Reine Anzeige, ändert keine Einstellungen.
10. **Premiere-Steuerung in der Kino-Leiste** (05.10.2026): Im Kino-Modus versteckt das CSS alle Knöpfe der Premiere-Karte, auch das vorhandene „⏸ Pause für alle“ — es war während der Premiere unerreichbar. Neu: `forkKinoKnopf()`, `updateForkPremButtons()` (Aufruf als erste Zeile in `updatePremPauseBtn()`), `premStopAll()`. Zwei runde Host-Knöpfe in `#cinema-vol`: ⏸/▶ ruft den bestehenden Pause-Handler, ⏹ beendet die Premiere für alle (zwei Klicks, 3 s Bestätigungsfenster). Beenden: Mitschnitt verwerfen (`invalidatePremCache()`), Stimmen stoppen, Video ans Ende, dann Bewertung wie nach dem normalen Ende. Netz: neue Nachricht `premStop` (in `GUEST_IN`, Gast-Handler) und `hostCmd` `premStop` für den logischen Host. Spulen bewusst nicht eingebaut (Stimmen laufen zeitgesteuert über WebAudio, Spulen müsste bei allen synchron neu planen).
11. **Take-Speicher** (05.10.2026): Jeder fertige Take und jedes SKIP landet sofort in IndexedDB (`ss-takes`, Store `takes`, Schlüssel `<scene.id>|<sortierte Rollen>|<idx 5-stellig>`, 14 Tage Haltbarkeit, `alteTakesAufraeumen()` 8 s nach Start). Neu: `takeDb/takeTx/takeRaum/takeKey/takeSichern/takeVergessen/takeRaumLeeren/takeStand/updateTakeSpeicherHinweis/takeFortsetzenAnbieten`. `startBooth()` bietet einen gespeicherten Stand zum Fortsetzen an (Kasten wird per JS erzeugt — index.html bleibt unverändert, der Besitzer lädt nur client.js hoch); „Fortsetzen“ füllt `takes` und springt auf die erste Line ohne Take, „Neu anfangen“ löscht den Raum. `finishBooth()` löscht den Raum 1,2 s nach dem Abgeben. Alles in try/catch, `takeDb()` hat 4-s-Timeout: ohne IndexedDB (auch in jsdom) verhält sich alles exakt wie vorher. Geprüft mit fake-indexeddb: speichern, wiederherstellen, Rollenwechsel = eigener Stand, „Neu anfangen“ leert.
12. **Nachbessern nach der Premiere** (05.10.2026): `renderRedoPanel()` und `redoLine()` erlauben das Korrektur-Panel jetzt auch auf `scr-playback`, sobald das Premieren-Video `ended` ist (vorher sperrte `premiereLocked` es dauerhaft). Aufruf von `renderRedoPanel("redo-panel-prem")` im `ended`-Handler von `playMixInternal()`. Der Weg danach ist der bestehende: `finishRedo()` → `applyTrackUpdate()` → `publishMix()`.

13. **Teil-Abgabe bei „Trotzdem starten“** (05.10.2026): Drückte der Host den Notausgang, waren die schon aufgenommenen Lines der noch Sprechenden komplett weg — sie hatten `finishBooth()` nie erreicht, ihre Rolle fehlte in `collected`. Neu: `teilAbgabeJetzt()` (Gast, einmalig pro Runde über `teilAbgabeGemacht`, nur wenn `#scr-booth` aktiv) und `forceMixMitTeilAbgabe()` (Host: `broadcast({t:"teilAbgabe"})`, selbst abgeben, 2,5 s warten, dann `maybeFinishTracks(true)`). Neue Nachricht `teilAbgabe` in `GUEST_IN`; aufgerufen aus dem Klick-Handler von `#btn-force-mix` und aus `hostCmd` `forceMix`. Datenformat unverändert — es ist derselbe `tracks`-Weg wie bei normalem Fertigwerden.
14. **Lückenfüller nie stumm** (05.10.2026): Der Schalter heißt „Original-Stimmen (unbesetzte Rollen)“, schaltete aber ALLE Original-Einsprünge stumm — auch die Lücken einer Rolle, die jemand spricht. Ergebnis war eine komplett stumme Figur. Neu: `loadMix()` markiert Original-Items mit `lueckenFueller: true`, wenn mindestens eine Rolle der Line schon echte Aufnahmen im Mix hat (`besetzteRollen`); `isOrigItemAudible()` gibt für diese immer `true` zurück. Unbesetzte Rollen verhalten sich unverändert. Geprüft: Lückenfüller hörbar bei Schalter aus und bei stummgeschalteter Rolle, unbesetzte Rolle weiter stumm.

15. **Versprecher-Rangliste** (05.10.2026, `renderBlooperRangliste()`, Aufruf oben in `updateOuttakesBtn()`): Zählt die Outtakes pro `name` und zeigt die Top 5 als eine Zeile unter der Outtakes-Leiste. Rein lokal und rein Anzeige; wird beim Leeren von `outtakes` wieder entfernt (deshalb der Aufruf VOR dem frühen `return`).

**Beim Abschluss-Durchgang gefundene und behobene Eigenfehler:** Wiederherstellen überschrieb frisch aufgenommene Takes (jetzt nur echte Lücken); nach einer Korrektur im Anschluss an die Premiere startete nur bei den Gästen automatisch neu, der Host blieb stehen (neu: `premiereLiefSchon`, im `ended`-Handler gesetzt, in `premStart()` zurückgesetzt, am Ende von `loadMix()` ausgewertet); die Rangliste blieb beim Leeren der Outtakes stehen.

16. **Warnung beim Schließen** (05.10.2026): `beforeunload`-Listener — ist `#scr-booth` aktiv und sind noch Lines ohne Take offen, zeigt der Browser seinen Standard-Dialog.
17. **Tasten in der Booth** (05.10.2026): `keydown`-Listener — Pfeil rechts = `#btn-line-next`, Pfeil links = `#btn-line-prev`, Enter = `#btn-line-play`. Nur bei aktiver Booth, ohne Modifier, nicht in Eingabefeldern, nur wenn der Knopf sichtbar und nicht deaktiviert ist. Die Leertaste (Aufnehmen) stammt aus dem Original und bleibt unberührt.
18. **Dauer-Pegelanzeige** (05.10.2026, `zeigePegel()`): Hängt in der `draw`-Schleife von `startVizOn()` (max. 4×/s, gedrosselt über `startVizOn._pegelT`) und schreibt „🟢 Pegel ok / 🔉 zu leise / 🔴 zu laut“ neben `#booth-status`.

**Basis:** Dieser Stand ist der Merge von Fork-Patch 1–4 mit dem Original v9.25.4 (Sync vom 05.10.2026). Konflikte gab es nur in `scenes.json` und `scenes-index.json`; beide wurden als Vereinigung beider Szenenlisten aufgelöst (158 Szenen). `client.js` hat sauber automatisch gemerged.

19. **Speicher-Gesundheitsprüfung** (05.10.2026, `takeSpeicherPruefen()`, aufgerufen am Anfang von `takeFortsetzenAnbieten()`): schreibt und liest einmal pro Sitzung einen Probeeintrag. Scheitert das (Privatfenster, Richtlinie, volle Platte), wird `takeSpeicherAn` abgeschaltet und die Booth-Zeile warnt ehrlich, statt „gespeichert“ zu behaupten.
20. **„Rest im Original“** (05.10.2026, `restImOriginalLassen()` + `updateRestKnopf()`): Knopf neben „Überspringen“, setzt alle offenen Lines auf `SKIP` und gibt ab. Mit Rückfrage und Warnung, wie viele davon keine Originalspur haben. Sichtbar nur bei aktiver Booth, `redoMode === null` und mehr als einer offenen Line.
21. **Hinweis auf wirklich stumme Lines** (05.10.2026, in `loadMix()`): zählt Lines ohne Take UND ohne Originalspur und sagt es in `play-status`.

**Beim Review-Durchgang gefundene und behobene Eigenfehler (05.10.2026):** `zeigePegel()` hängte ein Kind in `#booth-status`, das `status()` per `textContent` 4×/s wieder wegwarf — jetzt eigenes Geschwister-Element. Enter löste bei fokussiertem Knopf zwei Aktionen aus (Browser-Klick + „Anhören“) — jetzt nur noch, wenn kein Button/Link den Fokus hat. `finishBooth()` konnte doppelt laufen (Rest-Knopf + `teilAbgabe` + normales Fertigwerden) — neuer Riegel `boothAbgegeben`, zurückgesetzt in `startBooth()`. Der Rest-Knopf erschien auch, wenn keine der offenen Lines eine Originalspur hat (hätte nur Stille erzeugt) — jetzt ausgeblendet.

**Hinweis zur Teil-Aufnahme:** Lines ohne Take bekommen schon immer automatisch die Originalstimme — `loadMix()` füllt alles, was `coveredIdx` nicht enthält und `lineHasOrig()` erfüllt. Stumm bleibt eine Line nur, wenn die Szene für sie gar keine Original-MP3 hat.

Patch 1–4 sind dem Entwickler gemeldet, 5–21 noch nicht. Übernimmt er einen davon, unseren Patch ersatzlos zugunsten seiner Version aufgeben.

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
