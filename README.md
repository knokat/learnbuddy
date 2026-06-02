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
```

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
