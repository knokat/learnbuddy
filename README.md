# Lernfuchs 🦊

Kindgerechte Lern-App im Duolingo-Stil. Läuft als statische Web-App (GitHub Pages),
optimiert für das iPad (Safari). Kein Build-Schritt, keine Server-Komponente.

## So ist die App aufgebaut

```
index.html              ← der App-Motor (UI + Übungslogik). Bleibt immer gleich groß.
manifest.json           ← winzige Liste aller eingebauten Themen.
projects/
  villach.json          ← Inhalte des Themas „Stadtteile von Villach"
  villach-map.jpg       ← Karte (normale Bilddatei, kein Base64)
  _vorlage.json         ← Kopiervorlage für neue Themen
```

**Wichtig fürs Tempo:** Themen-Inhalte stecken NICHT in `index.html`, sondern in
eigenen Dateien unter `projects/`. Der Startbildschirm lädt nur das winzige
`manifest.json`. Die vollen Übungen eines Themas werden erst geladen, wenn das Kind
das Thema antippt – und danach im Speicher behalten. Dadurch bleibt die App schnell,
egal wie viele Themen dazukommen.

## Neues Thema hinzufügen (3 Schritte)

1. `projects/_vorlage.json` kopieren, z. B. nach `projects/mein-thema.json`,
   und mit Inhalten füllen (siehe Fragetypen unten).
2. Falls das Thema eine Karte braucht: Bild nach `projects/` legen und im JSON
   unter `"map"` den Pfad eintragen (z. B. `"projects/mein-thema-karte.jpg"`).
3. In `manifest.json` eine Zeile zur `projects`-Liste ergänzen:
   ```json
   { "id": "mein-thema", "name": "Mein Thema", "icon": "🌍",
     "subtitle": "Klasse X", "file": "projects/mein-thema.json", "lessonCount": 2 }
   ```
   `lessonCount` = Anzahl der freigeschalteten (nicht gesperrten) Lektionen –
   nur für die Fortschrittsanzeige am Startbildschirm.

Alternativ: ein Thema direkt in der App über **„Neues Projekt anlegen" → Datei**
importieren. Solche Projekte werden lokal am Gerät gespeichert (localStorage) und
müssen nicht ins Repo.

## Fragetypen

```jsonc
// Karte antippen (braucht "map" + "districts" im Projekt)
{ "type": "map", "prompt": "Tippe auf <b>Lind</b>!", "target": 1 }

// Multiple Choice (answer = Index der richtigen Option, 0-basiert)
{ "type": "choice", "prompt": "Wo liegt X?", "options": ["A","B","C","D"], "answer": 0 }

// Eintippen (accept = erlaubte Antworten, Groß/Klein egal, Rechtschreibung zählt)
{ "type": "type", "prompt": "Schreibe ...", "accept": ["lind"], "placeholder": "…", "highlight": 1 }

// Paare verbinden
{ "type": "match", "prompt": "Verbinde!", "pairs": [ { "left": "🐶", "right": "Hund" } ] }

// Ergänzen: Bekanntes steht da (mit Symbol), Kind tippt das Fehlende ein
{ "type": "collect", "prompt": "Was fehlt noch?",
  "given":  [ { "symbol": "🏟️", "name": "Sport-Stadion" } ],
  "blanks": [ { "symbol": "🏫", "name": "Volksschule", "accept": ["volksschule","vs"] } ] }
// Lückentext: Satz mit Lücken, Kind tippt die fehlenden Wörter
{ "type": "cloze", "prompt": "Ergänze den Satz!",
  "segments": [
    { "text": "In Seebach gibt es ein " },
    { "blank": { "name": "Naherholungsgebiet", "accept": ["naherholungsgebiet"] } },
    { "text": "." }
  ] }
```

**Feedback bei falscher Antwort:** Bei jeder Aufgabe (außer Paare verbinden, das sich selbst löst)
kann das Kind nach einer falschen Antwort wählen: **„↻ Nochmal"** (nochmal versuchen – bei
Tipp-/Ergänz-/Lückentext-Aufgaben bleibt das Getippte erhalten, damit man nur korrigieren muss)
oder **„Lösung zeigen"** (zeigt die richtige Antwort an). Ein Herz wird pro Frage höchstens
einmal abgezogen, egal wie oft man es erneut versucht.

**Inhaltsregel für `collect` (wichtig):**
1. Unter `given` dürfen NUR Dinge stehen, die in derselben Lektion bereits in einer früheren
   Frage vorkamen — dem Kind wird nichts als „bekannt" präsentiert, was es nicht geübt hat.
2. Unter `given` müssen ALLE bereits in der Lektion vorgekommenen Einträge dieser Kategorie
   stehen (z. B. alle bisher genannten Einrichtungen des Stadtteils). Sonst könnte das Kind
   etwas, das es schon gelernt hat, als „fehlend" eintippen, obwohl es gar nicht als Lösung
   hinterlegt ist.
3. `blanks` enthält nur **neue** Einträge, die vorher noch nicht gezeigt wurden.
4. Gleiche Dinge bekommen dasselbe Symbol wie zuvor in der Lektion, und innerhalb einer
   `collect`-Frage ist jedes Symbol eindeutig (z. B. 🏫 = HAK/HAS, 🎒 = Volksschule).

**Inhaltsregel für Frage-Texte (`prompt`):** Der `prompt` (die Aufgaben-Überschrift) darf die
gesuchte Antwort nicht verraten — vor allem bei `type`, `cloze` und den `collect`-`blanks`, wo
das Kind die Antwort selbst eintippt. Das Lösungswort (oder ein wesentlicher Wortbestandteil)
darf nicht in der Überschrift stehen. Beispiel: Ein Lückentext, bei dem „Naherholungsgebiet"
gesucht ist, darf dieses Wort nicht in der Überschrift nennen. (Bei `choice` unkritisch, da die
Antwort ohnehin unter den Optionen steht.)

Für Karten-Themen zusätzlich im Projekt:
```jsonc
"map": "projects/karte.jpg",
"districts": [ { "n": 1, "name": "Lind", "x": 63.5, "y": 14 } ]  // x/y = Prozent-Position auf dem Bild
```

## Hosting

Alle Dateien ins Repo, GitHub Pages aktivieren, App über die Pages-URL aufrufen.
**Hinweis:** Lokales Öffnen per Doppelklick (`file://`) funktioniert nicht, weil der
Browser dann das Nachladen der JSON-Dateien blockiert. Immer über die Web-Adresse
(oder einen lokalen Server) testen.

## Fortschritt / Daten

Pro Gerät im localStorage: erledigte Lektionen (`lernfuchs_done`), XP (`lernfuchs_xp`),
Streak (`lernfuchs_streak`), eigene importierte Projekte (`lernfuchs_userprojects`).
