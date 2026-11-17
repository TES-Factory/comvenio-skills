---
type: tech-teilsystem
kennung: skills-katalog
titel: Skills-Katalog — Skills, Action-Katalog, Prüf- und Sync-Skripte
repository: comvenio-skills
stand: teilweise
entitaeten: []
routen: []
cli: []
dateien:
- skills/**
- catalog/**
- scripts/**
- '*.mjs'
use_cases:
- UC-1
- UC-2
- UC-3
- UC-4
entscheidungen: []
faehigkeiten:
- fach-skills
- workflow-skills
- evals
- befehlsflaeche-pruefen
- action-ids-pruefen
- input-pruefen
- ausnahmemarke
- katalog-sync
- fixture-tests
- qualitaetsprofil
quellen: []
letzte_beschreibung:
  klammer: null
  commit: null
---

# Skills-Katalog — Skills, Action-Katalog, Prüf- und Sync-Skripte

## Zweck

comvenio-skills ist das öffentliche Paket der Agent Skills für Vereinsverantwortliche, die Comvenio mit Claude, Codex oder einem anderen kompatiblen KI-Assistenten über das `comvenio` CLI bedienen: 18 Skills (14 Fachbereiche, 4 Workflows über mehrere Bereiche) mit Evals, eine Kopie des Action-Katalogs der Connector-Release und Skripte, die jeden Skill gegen die Befehlsfläche des CLI und den Katalog prüfen und den Katalog aus der Kundendoku von comvenio-cli erneuern. Wer einen Skill installiert, bekommt keine Anleitung, die mit einem Fehler endet. <!-- q: geraetetoken-abbau-01 §3; geraetetoken-abbau-01 §4.2; commit:6b0bd23; commit:50544b9; code:scripts/validate-skills.mjs#checkCommands; code:scripts/sync-action-catalog.mjs#listArticles -->

## Use Cases

**UC-1 Skills installieren.** Akteur: Vereinsverantwortliche mit ihrem KI-Assistenten. Auslöser: `npx skills add TES-Factory/comvenio-skills --all` oder `--skill <name>`. Ablauf: Der Assistent übernimmt die Skills; Voraussetzung sind das installierte `comvenio` CLI und eine bestätigte `comvenio login`-Anmeldung. Ergebnis: Assistent kennt die Arbeitsabläufe je Bereich. <!-- q: commit:6b0bd23; commit:887cc88; geraetetoken-abbau-01 §3 -->

**UC-2 Vereinsarbeit über einen Skill.** Akteur: KI-Assistent im Auftrag eines Vereinsverantwortlichen. Auslöser: Anliegen in Vereinssprache, etwa eine Trainingsserie oder eine Speisekarte. Ablauf: Der Skill beginnt mit `comvenio whoami --json` und `comvenio action list`, liest den Stand, ruft Actions mit `comvenio action call <action-id> --input '<json>' --json` und bestätigt kritische Actions mit `comvenio action confirm`; fehlt eine Action, beschreibt er den Weg in der Web-App. Ergebnis: Änderung im angemeldeten Verein nach den Rechten des Menschen. <!-- q: geraetetoken-abbau-01 §4.3; geraetetoken-abbau-01; commit:d29eb95; commit:d2ab764 -->

**UC-3 Katalog erneuern.** Akteur: Comvenio-Team. Auslöser: neue Connector-Actions in comvenio-cli. Ablauf: `node scripts/sync-action-catalog.mjs <pfad-zu-comvenio-cli> [--ref origin/main]` liest aus jedem Themenartikel den erzeugten Abschnitt „Befehle und Actions“ und schreibt `catalog/actions.json` mit Kennung, Bereich, Teilaktionen, Risiko, Scopes und Artikel je Action. Ergebnis: Katalog im Stand der Kundendoku. <!-- q: geraetetoken-abbau-01 §4.2; code:scripts/sync-action-catalog.mjs#listArticles; code:scripts/sync-action-catalog.mjs#readArticle; code:scripts/sync-action-catalog.mjs#parseRisks -->

**UC-4 Skills vor der Auslieferung prüfen.** Akteur: Comvenio-Team, Testgate. Auslöser: `npm test` (Prüfung und Fixture-Tests). Ablauf: `validate-skills.mjs` sammelt jeden `comvenio`-Aufruf aus Skill-Dateien und Evals und meldet je Fund Skill, Datei und Zeile; die Fixture-Tests spielen Fälle wie klassischer Befehl, unbekannte Action, Ausnahmemarke, Fortsetzung und verbotene `--input`-Felder durch. Ergebnis: Exit 0 oder eine Fundliste. <!-- q: geraetetoken-abbau-01 §4.1; geraetetoken-abbau-01 §4.4; code:scripts/validate-skills.mjs#scanFile; code:scripts/test-validate-skills.mjs#copyCase; commit:02d9125 -->

## Funktionsweise

**Befehlsfläche.** Erlaubt sind nur `login`, `logout`, `whoami`, `action list|call|confirm`, `agent chat`, `finance` und `help`; klassische Fachbefehle, die nur mit Geräte-Token liefen, sind ein Fund. In Markdown zählen Codeblöcke und Inline-Code, in JSON die Strings der Evals, wobei Prosa nur die kleingeschriebene Programmangabe oder einen entfernten klassischen Befehl trifft. <!-- q: geraetetoken-abbau-01 §4.1; code:scripts/validate-skills.mjs#checkCommands; code:scripts/validate-skills.mjs#scanFile; commit:d2ab764 -->

**Umgehungen.** Fortsetzungszeilen werden wie in der Shell zusammengefügt (eine Action-ID oder ein Schalter über zwei Zeilen wird erkannt); `--input` wird in allen Formen gelesen (`'…'`, `"…"`, `=…`, ungequotet) und darf nie `club_id` oder eine Bestätigung tragen; ein Geräte-Token-Schalter ist ein Fund. <!-- q: code:scripts/validate-skills.mjs#joinContinuations; code:scripts/validate-skills.mjs#inputPayloads; commit:02d9125 -->

**Ausnahmemarke.** `<!-- klassisch-beispiel -->` auf eigener Zeile vor einem Codeblock nimmt dessen Befehle aus — nur, wenn der Block als „nicht mehr verfügbar“ gekennzeichnet ist; eine Marke ohne Block oder vor einem ungekennzeichneten Block ist selbst ein Fund. <!-- q: geraetetoken-abbau-01 §4.3; code:scripts/validate-skills.mjs#scanFile; commit:02d9125 -->

**Katalog.** Die Prüfung nimmt Action-IDs nur aus `catalog/actions.json`; ein fehlender oder unlesbarer Katalog ist ein Werkzeugfehler (Exit 2), nie ein grüner Lauf. Der Sync sammelt erst alle Probleme und schreibt bei einem einzigen nichts: Themenartikel ohne erzeugten Abschnitt, unlesbare Action-Zeile oder Abschnitt ohne Action und ohne „Noch keine Action“. <!-- q: geraetetoken-abbau-01 §4.2; geraetetoken-abbau-01; code:scripts/sync-action-catalog.mjs#fail; code:scripts/sync-action-catalog.mjs#category -->

<!-- techdoc:erzeugt zustaende -->
_Keine Einträge._
<!-- /techdoc:erzeugt -->


## Datenmodell

<!-- techdoc:erzeugt datenmodell -->
_Keine Einträge._
<!-- /techdoc:erzeugt -->

<!-- techdoc:erzeugt uml-klassen -->
_Keine Einträge._
<!-- /techdoc:erzeugt -->


## Schnittstellen

<!-- techdoc:erzeugt schnittstellen -->
**Routen**

_Keine Einträge._

**CLI**

_Keine Einträge._

**Hooks**

_Keine Einträge._

**Ereignisse**

_Keine Einträge._
<!-- /techdoc:erzeugt -->


## Abhängigkeiten

Quelle des Katalogs ist die Kundendoku von comvenio-cli (Abschnitt „Befehle und Actions“ der Themenartikel), gelesen aus dem Arbeitsbaum oder einer Git-Revision; die Skills selbst rufen nur das `comvenio` CLI, das mit der OAuth-Anmeldung des Menschen gegen den Comvenio AI Connector arbeitet. Verteilt werden sie über `npx skills`. <!-- q: geraetetoken-abbau-01 §4.2; geraetetoken-abbau-01; code:scripts/sync-action-catalog.mjs#git; commit:d29eb95 -->

<!-- techdoc:erzeugt abhaengigkeiten -->
Nutzt: —

Genutzt von: —
<!-- /techdoc:erzeugt -->


## Regeln

- Skills nennen nur Befehle der Befehlsfläche und Action-IDs aus dem Katalog; Aufrufe mit `--json`. <!-- q: geraetetoken-abbau-01; code:scripts/validate-skills.mjs#checkCommands -->
- Jeder Skill beginnt mit `comvenio whoami --json` und `comvenio action list`; kritische Actions laufen über `comvenio action confirm`, nie mit einem getippten Ja. <!-- q: geraetetoken-abbau-01; commit:d2ab764 -->
- `--input` trägt nie `club_id` oder eine Bestätigung — die Anmeldung bindet den Verein, `action confirm` trägt die Bestätigung. <!-- q: code:scripts/validate-skills.mjs#inputPayloads; commit:02d9125 -->
- Fehlt eine Action, beschreibt der Skill den Weg in der Web-App mit Menüpfad, ohne Befehl und ohne versteckte Schnittstelle. <!-- q: geraetetoken-abbau-01 §4.3 -->
- Kein neues Recht: Was erlaubt ist, entscheiden Scopes und Rolle im Verein. <!-- q: geraetetoken-abbau-01 -->
- Der Katalog wird nie teilweise geschrieben, ein fehlender Katalog ist nie grün. <!-- q: code:scripts/sync-action-catalog.mjs#fail; geraetetoken-abbau-01 -->
- Ab der Umstellung gilt die Prüfung im Testgate (`npm test`). <!-- q: geraetetoken-abbau-01 §4.4 -->

## Fehlerbilder

- **Prüfung lässt Umgehungen durch.** Symptom: Klassische Befehle, Geräte-Token-Schalter oder verbotene `--input`-Felder blieben unentdeckt, wenn sie über Fortsetzungszeilen geteilt, anders geschrieben oder in anderer `--input`-Form standen; eine Ausnahmemarke wirkte ohne Kennzeichnung; der Sync schrieb trotz fehlender Abschnitte. Ursache: Die Prüfung las physische Zeilen und nur eine Schreibweise. Codestelle: `joinContinuations`, `inputPayloads`, `scanFile` und `fail` im Sync, Fixture-Tests in `scripts/test-validate-skills.mjs`. <!-- q: commit:02d9125; code:scripts/validate-skills.mjs#joinContinuations; code:scripts/validate-skills.mjs#inputPayloads -->
- **Qualitätsprofil ungültig.** Symptom: Der Harness-Beleg wies das Profil mit `profil-ungueltig` ab. Ursache: Der Beleg erlaubt bei npm nur `run <skript>`, `npm test` allein nicht. Codestelle: `.harness-quality.json` mit `npm run test`. <!-- q: commit:f5cac8a -->

<!-- techdoc:luecke weitere Fehlerbilder: keine BUG-Kennung und keine Gotcha des Projekts zu comvenio-skills gefunden -->

## Code-Anker

<!-- techdoc:erzeugt code-anker -->
| Symbol | Datei |
|---|---|
| category | scripts/sync-action-catalog.mjs |
| fail | scripts/sync-action-catalog.mjs |
| git | scripts/sync-action-catalog.mjs |
| listArticles | scripts/sync-action-catalog.mjs |
| parseRisks | scripts/sync-action-catalog.mjs |
| readArticle | scripts/sync-action-catalog.mjs |
| copyCase | scripts/test-validate-skills.mjs |
| checkCommands | scripts/validate-skills.mjs |
| inputPayloads | scripts/validate-skills.mjs |
| joinContinuations | scripts/validate-skills.mjs |
| listFiles | scripts/validate-skills.mjs |
| scanFile | scripts/validate-skills.mjs |
<!-- /techdoc:erzeugt -->


## Gebaut und geplant

| Fähigkeit | stand | Quelle |
|---|---|---|
| Prüfung der Befehlsfläche und der Action-IDs gegen den Katalog | gebaut | <!-- q: code:scripts/validate-skills.mjs#checkCommands; geraetetoken-abbau-01 §4.1 --> |
| Prüfung aller `--input`-Formen und Fortsetzungszeilen | gebaut | <!-- q: code:scripts/validate-skills.mjs#inputPayloads; code:scripts/validate-skills.mjs#joinContinuations --> |
| Ausnahmemarke für gekennzeichnete Gegenbeispiele | gebaut | <!-- q: code:scripts/validate-skills.mjs#scanFile; geraetetoken-abbau-01 §4.3 --> |
| Katalog-Sync aus der Kundendoku von comvenio-cli | gebaut | <!-- q: code:scripts/sync-action-catalog.mjs#parseRisks; code:scripts/sync-action-catalog.mjs#listArticles; geraetetoken-abbau-01 §4.2 --> |
| Fixture-Tests der Prüfung | gebaut | <!-- q: code:scripts/test-validate-skills.mjs#copyCase --> |

<!-- techdoc:luecke Stand der 18 Skills und ihrer Evals nicht als Tabellenzeile belegbar: Skill-Texte sind Markdown und JSON ohne Code-Anker im Inventar; belegt über commit:d2ab764 (Umstellung auf Anmeldung und Actions) und die grüne Prüfung -->
