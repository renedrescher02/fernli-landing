# 🌿 Fernli – Blueprint FINAL
> *"Plant care for people who are just winging it."*
> Version: 3.2 · März 2026 · Claude Code-ready

---

## ⚠️ Scope Doctrine
> **Fernli v1 optimiert auf Vertrauen, Einfachheit und Habit-Building – nicht auf perfekte botanische Diagnose.**

Jede Entscheidung in diesem Dokument folgt diesem Satz. Nichts steht hier aus Romantik.

---

## 0. Wie dieses Dokument zu lesen ist

Dieses Dokument ist das einzige Grundgesetz von Fernli. Es ist gleichzeitig:
- **Produktreferenz** – alle Entscheidungen, alle Begründungen
- **Claude Code Bauplan** – direkt als Entwicklungsauftrag verwendbar

Jeder Begriff wird genau einmal definiert und dann konsequent überall gleich verwendet. Wenn hier `needs_attention` steht, steht es in der Datenbank, in der Engine und in den Modul-Prompts genauso.

---

## 1. Die Vision

Fernli ist ein digitaler Pflanzen-Begleiter. Jede Pflanze bekommt einen Buddy – eine animierte SVG-Figur die den wahrscheinlichsten Zustand der Pflanze visuell kommuniziert. Das emotionale Interface motiviert zur echten Pflanzenpflege.

### Der Kernsatz
> *„Der Buddy zeigt den wahrscheinlichsten Zustand deiner Pflanze – basierend auf ihrer Art, Umgebung und Pflegehistorie."*

Die App schätzt. Sie weiß nicht. Fernli arbeitet mit Heuristiken, Nutzerangaben und Wetter-Proxies – keine Sensoren, keine permanente Überwachung. Diese Ehrlichkeit schützt das Vertrauen.

### Das Herzstück
> **Der Nutzer sorgt für den Buddy → dadurch sorgt er für die Pflanze.**

### Produkt-Leitplanken

| Prinzip | Bedeutung |
|---|---|
| Utility First | Emotion dient der Funktion – nie umgekehrt |
| Ehrliche Schätzung | Fernli sagt "wahrscheinlich durstig", nicht "du musst jetzt gießen" |
| Korrekturen sind Kernfunktion | Nutzer kann immer eingreifen – das macht die App klüger |
| Konservativ statt falsch | Im Zweifel: weniger Alarm, nicht mehr |
| Habit-Loop first | Fernli ist zuerst ein Gewohnheits-Produkt, dann ein KI-Produkt |

---

## 2. Das Zustandsmodell – Vollständige Definition

Dieses Modell gilt überall: im UI, in der Datenbank, in der Engine, in den Modul-Prompts. Keine Ausnahmen, keine anderen Begriffe.

### 2.1 Die 4 Basis-Zustände

| State (Code) | Anzeigename | Visuell | Auslöser |
|---|---|---|---|
| `happy` | Thriving | Aufrecht, strahlend, Glanz-Effekte | Alle Parameter im grünen Bereich |
| `thirsty` | Thirsty | Schlaff, hängend, matter | Gieß-Intervall überschritten (artspezifisch) |
| `needs_attention` | Needs Attention | Besorgte Mimik, Ausrufezeichen | Fehlende Logs, Unsicherheit, schwache Signale |
| `critical` | Critical | Sehr schlaff, rote Wangen, dringlicher Ausdruck | Starke Überfälligkeit (definiert in 2.4) |

### 2.2 Die 2 Overlays

Overlays überlagern den Basis-Zustand visuell – sie ersetzen ihn nicht. Ein `thirsty`-Buddy kann gleichzeitig Schlafmütze tragen.

| Overlay (DB-Spalte) | Visuell | Auslöser |
|---|---|---|
| `is_sleeping` | Schlafmütze, geschlossene Augen, langsames Atmen | 22:00–07:00 lokale Zeit |
| `is_winter_overlay` | Beanie, Schal, ruhigere Haltung | Dezember–Februar (Nordhalbkugel) |

**Winter Overlay:** Rein visuell, keine eigene Zustandslogik. Fernli zeigt einmalig: "Im Winter wachsen die meisten Pflanzen langsamer – das ist normal."

**Day/Night Cycle:** Botanisch begründet (keine Photosynthese nachts), technisch trivial. Bleibt in v1.

### 2.3 Der Reason Layer (Pflicht unter jedem Zustand)

`needs_attention` ist kein Müllcontainer. Jeder State hat immer einen konkreten `primary_reason`:

```
state: needs_attention
primary_reason: "Seit 9 Tagen kein Gieß-Log"

state: needs_attention
primary_reason: "Standort wirkt eher zu dunkel für diese Pflanze"

state: needs_attention
primary_reason: "Mehrere Faktoren unklar – kurzes Update wäre super"

state: thirsty
primary_reason: "Letzte Bewässerung vor 10 Tagen, Intervall bei 7 Tagen"
```

`primary_reason` erscheint als kurzer Text unter dem Buddy. Nie nur ein Icon ohne Erklärung.

**Format von `primary_reason`**
- Kurz, konkret, maximal 90 Zeichen
- Immer verständlich für Nicht-Experten
- Kein Fachjargon, kein "Die KI hat ermittelt..."

Gültige Beispiele:
- `"Letzte Bewässerung vor 10 Tagen, Intervall bei 7 Tagen"`
- `"Seit 9 Tagen kein Gieß-Log"`
- `"Standort wirkt eher dunkel für diese Pflanze"`

Ungültige Beispiele:
- `"confidence_score unterhalb des Schwellenwerts"` ← zu technisch
- `"Die regelbasierte Engine hat folgende Faktoren berücksichtigt: ..."` ← zu lang

### 2.4 Critical – Konkrete Trigger-Regel

`critical` wird ausgelöst wenn einer dieser Fälle zutrifft:
- `days_since_watering >= adjusted_interval + 5`, oder
- `days_since_any_log >= 14` und letzter bekannter State war nicht `happy`, oder
- Mindestens 3 der definierten negativen Modifikatoren (siehe Abschnitt 5.1) gleichzeitig aktiv

**Wichtig:** Fehlende Logs allein bedeuten nicht automatisch Vernachlässigung. Bei fehlenden Daten bevorzugt Fernli `needs_attention` statt `critical`, außer die übrigen Signale sprechen klar für ein erhöhtes Risiko. `days_since_any_log >= 14` allein führt zu `needs_attention`, nicht zu `critical`.

Bei `critical` erscheint keine Skelett-Darstellung, sondern:
> *"Wir machen uns Sorgen um deine [Pflanze]. Alles okay?"*

Optionen: "Ja, habe mich gekümmert" → State neu berechnen / "Nein, sie ist eingegangen" → Nutzer entscheidet selbst.

### 2.5 "Needs Light" – kein Zustand, eine Empfehlung

- ❌ Kein `needs_light` State – existiert nirgendwo im System
- ✅ Licht-Problem → `recommended_action: adjust_location`
- ✅ Im Onboarding: Hinweis bei dunkel eingeschätztem Standort
- ✅ Als `primary_reason` unter `needs_attention` wenn Licht der Hauptfaktor ist

### 2.6 Unsicherheits-Sprache

**Bedeutung von `confidence`**

`confidence` ist eine interne Heuristik zur Einschätzung der Regel-Engine-Sicherheit – keine wissenschaftliche Wahrscheinlichkeit. Sie dient dazu, Sprache und Reminder-Verhalten konservativ zu steuern. Ein Wert von 0.8 bedeutet nicht "80% Wahrscheinlichkeit" – er bedeutet "die Engine hat genug Datenpunkte für eine klare Aussage."

| Konfidenz | State | Sprache |
|---|---|---|
| Hoch (>80%) | `thirsty` | "Deine Monstera braucht heute Wasser." |
| Mittel (50–80%) | `thirsty` | "Deine Monstera ist wahrscheinlich durstig." |
| Niedrig (<50%) | `needs_attention` | "Schau mal nach deiner Monstera – wir sind uns nicht ganz sicher." |
| Keine Daten | `needs_attention` | "Wir haben lange nichts gehört. Alles okay?" |

Keine Daten = `needs_attention` mit `primary_reason: "Wir haben lange nichts gehört"` und `recommended_action: check`. Kein eigener State.

### 2.7 State Priority (bei mehreren aktiven Signalen)

```
1. critical         → schlägt alles
2. thirsty          → schlägt needs_attention und happy
3. needs_attention  → wenn Unsicherheit/fehlende Daten dominant
4. happy            → nur wenn keine anderen Signale aktiv
```

Overlays werden unabhängig vom Basis-State berechnet und angewendet.

---

## 3. Der Core Loop – Das wichtigste Feature

Fernli ist zuerst ein Habit-Loop-Produkt, dann ein KI-Produkt. Wenn der Loop nicht sitzt, rettet keine gute KI die App.

```
Push → Quick-Log (1 Tap) → State neu berechnet → Buddy reagiert → Vertrauen wächst → Retention
```

### Minimal-Loop (App muss nicht geöffnet werden)

```
Push: "Deine Monstera hätte gerne Wasser 🌿"
→ [Gegossen ✓]  [Später erinnern]
→ Tap "Gegossen" → care_log gesetzt → State wird neu berechnet → Buddy reagiert beim nächsten Öffnen
```

**Wichtig:** Nach Quick-Log wird der State neu berechnet – er wird nicht automatisch `happy`. War vorher `critical`, springt er auf `thirsty` oder `needs_attention` je nach Datenlage.

### Extended Loop (App geöffnet)

```
Home Screen → Buddy + current_state + primary_reason
→ Großer CTA: "Heute gegossen"
→ Tap → care_log + sichtbare Buddy-Reaktionsanimation
→ Kurze positive Bestätigung: "Gut gemacht 🌿"
```

### Kleine Rückkopplung für Retention
- Buddy-Erholungs-Animation nach jedem Log (sichtbar, belohnend)
- Ruhige Bestätigung auf Detailseite: "Seit 5 Tagen gut versorgt"
- Keine Streak-Mechanik in v1

---

## 4. Die Recommendation Engine

### Action-Typen (vollständige Liste, keine anderen)

| Action (Code) | Bedeutung | Angezeigt als |
|---|---|---|
| `water` | Jetzt gießen | "Zeit zum Gießen" |
| `check` | Kurz nachsehen / bestätigen | "Kurz nachsehen" |
| `wait` | Nichts tun | (kein aktiver Hinweis) |
| `adjust_location` | Standort langfristig anpassen | "Hellerer Platz wäre sinnvoll" |

### UI-Verhalten bei `recommended_action = wait`

Wenn `recommended_action = wait`, zeigt Fernli:
- den Buddy mit `current_state` und `primary_reason`
- keinen aktiven Alarm
- keinen prominenten CTA außer optional "Pflege eintragen"

Für `wait` werden keine aktiven Reminder-Notifications ausgelöst, außer saisonale Hinweise oder ein `check_in` bei längerer Inaktivität (>7 Tage ohne Log).

### Engine-Architektur (dreischichtig)

```
Schicht 1: Regelbasierte Engine (TypeScript, deterministisch)
  Input:  Pflanzenart + Perenual-Basiswerte, care_logs, Wetter (leichter Modifier),
          Fensterrichtung, Jahreszeit, species_confidence
  Output: current_state, is_sleeping, is_winter_overlay, primary_reason,
          recommended_action, due_at, urgency, confidence, rationale

Schicht 2: Recommendation Layer
  → Mapped Output auf strukturiertes care_recommendations-Objekt
  → Action-Typ: water | check | wait | adjust_location
  → urgency: low | medium | high | critical

Schicht 3: LLM (Claude Haiku oder GPT-4o mini, via Edge Function)
  → Formuliert rationale in natürliche Sprache → display_text
  → Generiert Notification-Text mit artspezifischer Buddy-Persönlichkeit
  → Kommuniziert Unsicherheit konservativ und ehrlich
  → Kein "Diagnose"-Framing – immer "Problemhinweis / wahrscheinliche Ursachen"
```

### Wetter als leichter Modifier (nicht mehr)

Wetterdaten sind ein schwacher Proxy für Indoor-Pflanzen. Sie modifizieren das Intervall leicht, überstimmen aber nie die Kern-Regellogik:

```
Temperatur > 25°C    → Intervall -1 bis -2 Tage
Luftfeuchtigkeit < 40% → Intervall -1 Tag
Alles andere         → kein Einfluss
```

Formulierung: "Es war diese Woche warm – etwas früher gießen könnte sinnvoll sein." Nie: "Das Wetter erfordert sofortiges Gießen."

### Der Moat

Fernli schickt der KI automatisch den vollständigen Kontext. Der Nutzer tippt nur "graue Blätter":

```
Art: Monstera Deliciosa (vom Nutzer bestätigt)
In App seit: 47 Tagen
Letzte Bewässerung: vor 8 Tagen (adjusted_interval: 7 Tage)
Standort: München, Westfenster, Wohnzimmer
Wetter: 22°C, 45% Luftfeuchtigkeit
Symptom: graue Blätter
```

---

## 5. Regel-Engine Parameter (v1 Startpflanzen)

| Parameter | Monstera | Kaktus | Pothos |
|---|---|---|---|
| Basis-Intervall | 7 Tage | 14 Tage | 7 Tage |
| Temperatur > 25°C | -1 Tag | -2 Tage | -1 Tag |
| Luftfeuchtigkeit < 40% | -1 Tag | 0 | -1 Tag |
| Winter (Dez–Feb) | +3 Tage | +7 Tage | +2 Tage |
| Südfenster | -1 Tag | -1 Tag | -1 Tag |
| Nordfenster | +1 Tag | +2 Tage | +1 Tag |

**Minimum-Intervall:** Monstera 3 Tage · Kaktus 7 Tage · Pothos 3 Tage

**Confidence-Regel:** Art nicht vom Nutzer bestätigt → `confidence` automatisch -20%

### 5.1 Negative Modifikatoren für die Critical-Regel

Für die Regel "3+ negative Modifikatoren gleichzeitig aktiv" zählen in v1 nur diese Faktoren:

| Modifikator | Bedingung |
|---|---|
| Gieß-Überfälligkeit | `days_since_watering > adjusted_interval` |
| Hohe Temperatur | `temperature > 25°C` |
| Niedrige Luftfeuchtigkeit | `humidity < 40%` |
| Starke Sonneneinstrahlung | `window_direction = 'S'` |

**Zählen nicht als negativer Modifikator:**
- `species_confirmed_by_user = false` → beeinflusst nur `confidence`, nicht `critical`
- `window_direction = 'unknown'` → erhöht Unsicherheit, kein Risiko-Signal
- Fehlende Logs → führen zu `needs_attention`, nicht direkt zu `critical`

Damit ist die Critical-Berechnung deterministisch und nicht interpretierbar.

---

## 6. Onboarding

### Prinzip: Login kommt nach dem Wow-Moment

```
❌ Alt: Login → Foto → Buddy
✅ Final: Foto → Artvorschlag → Buddy erscheint → Login zum Speichern
```

### Flow

```
1. App öffnen
2. Direkt zur Kamera: "Fotografiere deine Pflanze"
3. Foto → Vision API (Edge Function) → Top 3 Vorschläge mit Konfidenz
4. Nutzer bestätigt oder wählt manuell
   → Konfidenz < 60%: manuelle Auswahl erzwingen
5. Standort-Setup: Stadt / Zimmertyp / Fensterrichtung (Kompass)
6. ✨ Buddy erscheint ← Wow-Moment
7. Buddy zeigt current_state + primary_reason + erste Empfehlung
8. "Buddy speichern?" → Apple/Google Login via Supabase
9. Fertig.
```

### Failure Paths

| Schritt | Problem | Fallback |
|---|---|---|
| Kamera | Berechtigung verweigert | Erklärung + Einstellungen-Link |
| Erkennung | Konfidenz < 60% | Manuelle Auswahl erzwingen |
| Erkennung | Kein Netz | Lokal cachen, später senden |
| Kompass | Berechtigung fehlt | Manuelle Auswahl N / O / S / W |
| Fensterrichtung | Nutzer weiß es nicht | "Nicht sicher" → konservative Schätzung |
| Login | Fehlschlag | Lokale Session, später verknüpfen |

### Licht-Kalkulator (korrekt geframet)

Kompass → Fensterrichtung → Licht-Einschätzung. Praktische Alltagsnäherung, kein Präzisionsinstrument. Fernli sagt nie "du hast exakt 800 Lux" – sondern "Südfenster bedeutet wahrscheinlich viel Licht."

### Manuelle Korrekturen (Kernfunktion)

Jederzeit änderbar: Pflanzenart, letztes Gießdatum, Fensterrichtung, Zimmertyp. Buddy-Zustand widersprechen: "Meine Pflanze sieht eigentlich okay aus" → `confirmed_ok`-Log + State neu berechnen.

### Wirkung von `confirmed_ok`

Wenn ein Nutzer "Meine Pflanze sieht eigentlich okay aus" bestätigt, wird ein `care_log` mit `action_type = confirmed_ok` gespeichert. Diese Aktion bewirkt in v1:
- `last_user_confirmation_at = now`
- `check_in`-Notifications werden für 7 Tage unterdrückt
- `needs_attention` aufgrund fehlender Daten oder Unsicherheit wird abgeschwächt
- `critical` aufgrund echter Wasserüberfälligkeit (`days_since_watering >= adjusted_interval + 5`) wird **nicht** überschrieben
- State wird danach neu berechnet

`confirmed_ok` ist ein Vertrauenssignal des Nutzers – kein harter Override gegen eindeutige Regel-Engine-Signale.

### Erwartetes Output-Schema der Vision API

Die Vision API (Edge Function) liefert in v1 immer dieses Format:

```json
{
  "top_3_species": ["Monstera Deliciosa", "Philodendron", "Pothos"],
  "confidence_scores": [0.82, 0.11, 0.07],
  "needs_manual_confirmation": false,
  "reasoning_short": "Charakteristische Blattform und Fenestration erkannt."
}
```

Regel: Wenn `confidence_scores[0] < 0.6`, muss `needs_manual_confirmation = true` sein und die manuelle Auswahl wird im Onboarding erzwungen.

---

## 7. Buddy-Design & Assets

- **v1:** SVG + einfache React-Native-Animationen (kein Lottie)
- **v1:** Statische SVGs mit einfachen React-Native-Animationen via `Animated` API (Wackeln, Pulsieren, Schlaf-Atmung)
- **v1:** Stil der Landing Page (bereits existiert, Claude Code erweitert)
- **v2:** Lottie-Animationen oder professionelle Illustrationen nach Validierung

Lottie wird in v1 nicht verwendet.

### Buddy-Persönlichkeiten (nur in Notification-Tonalität)

| Pflanze | Persönlichkeit | Notification-Beispiel |
|---|---|---|
| Monstera | Expressiv, leicht dramatisch | "Deine Monstera hätte gerne Wasser 🌿" |
| Kaktus | Stoisch, kurz | "Kaktus. Wasser. Heute." |
| Pothos | Entspannt, freundlich | "Dein Pothos wäre dankbar für etwas Wasser 💧" |

---

## 8. Notifications

### Prinzip: Utility First mit Persönlichkeit

Information steht immer vorne. Persönlichkeit durch Tonalität und Emoji – nie auf Kosten der Klarheit.

### Notification-Typen

| Type (Code) | Frequenz | Beispiel (Monstera) |
|---|---|---|
| `water_reminder` | Bedarfsbasiert | "Deine Monstera hätte gerne Wasser 🌿" |
| `needs_attention` | Wenn mehrere Signale | "Schau kurz nach deiner Monstera – irgendwas stimmt nicht." |
| `check_in` | Nach 7 Tagen ohne Log | "Alles okay bei deiner Monstera? Kurzes Update wäre super." |
| `seasonal` | 1x pro Saison | "Winter ist da ❄️ – deine Pflanzen brauchen jetzt weniger." |

### Notification-Caps (Pflicht)
- Max. 1 dringende Notification pro Pflanze pro 24 Stunden
- Max. 3 Notifications pro Woche gesamt (Free)
- `check_in` frühestens nach 7 Tagen ohne Log
- Kein Retry bei Fehler innerhalb von 6 Stunden

### Notification-Priorität

Wenn mehrere Typen gleichzeitig möglich wären, gilt diese Reihenfolge. Pro Zeitfenster wird immer nur die höchstpriorisierte gesendet:

```
1. water_reminder     (höchste Priorität)
2. needs_attention
3. check_in
4. seasonal           (niedrigste Priorität)
```

### Quick-Log via Notification

```
Push erscheint
→ [Gegossen ✓]  [Später erinnern]
→ "Gegossen" → care_log (source: 'notification_quick_log') → State neu berechnen
→ "Später"   → Reminder in 4 Stunden
```

App muss nicht geöffnet werden.

---

## 9. Geschäftsmodell

### Freemium-Grenze

| Feature | Gratis | Fernli+ |
|---|---|---|
| Pflanzen / Buddies | 1 | Unbegrenzt |
| 4 Basis-Zustände + 2 Overlays | ✅ | ✅ |
| Regelbasierte Pflegekalkulation | ✅ | ✅ |
| Smart Notifications + Quick-Log | ✅ | ✅ |
| Manuelle Korrekturen | ✅ | ✅ |
| Pflegehistorie | 30 Tage | Unbegrenzt |
| Fotoverlauf | ❌ | ✅ |
| Priorisierte Pflegeübersicht (Multi-Plant) | ❌ | ✅ |
| KI-Problemhinweise | ❌ | ✅ (v1.1) |
| Exklusive Saisonoutfits | ❌ | ✅ (v2) |

**Preis:** 2,99 €/Monat oder 19,99 €/Jahr

### Das echte Kaufargument
> *"Fernli kennt die ganze Geschichte deiner Pflanze – wenn etwas nicht stimmt, zeigt es dir wahrscheinliche Ursachen und nächste Schritte."*

**Sprachregeln für Premium:** Niemals "Diagnose" – immer "Problemhinweis" oder "wahrscheinliche Ursachen". Pflanzenprobleme sind oft mehrdeutig; ehrliche Sprache verhindert Erwartungsenttäuschung.

---

## 10. Todes-Logik

### v1: Kein Dead-State

Fehlende Logs ≠ Vernachlässigung. Maximum in v1: `critical`.

```
happy → thirsty → needs_attention → critical → Check-In Frage an Nutzer
```

### v2: Dead / Skelett

Nach echten Nutzerdaten: artspezifische Überlebenszeiten, lange Unsicherheitsphasen, Skelett als Konsequenz echter Datenlage.

---

## 11. Analytics – Von Tag 1

### Onboarding Events
- `app_opened`
- `camera_permission_granted` / `camera_permission_denied`
- `photo_taken`
- `species_recognized` (+ confidence)
- `species_manually_corrected`
- `buddy_appeared` ← Wow-Moment
- `login_completed`
- `onboarding_dropped` (+ step)

### Engagement Events
- `app_opened_daily`
- `notification_received` / `notification_opened`
- `quick_log_used`
- `care_log_created`
- `state_viewed`
- `species_corrected_post_onboarding`
- `recommendation_followed`

### Retention KPIs
- Retention Tag 1 / 7 / 30
- Pflanzen ohne Log nach Onboarding
- Durchschnittliche Logs pro Nutzer/Woche
- Notification → Quick-Log Conversion Rate

---

## 12. Sicherheit

### API Keys – niemals im Client

```
Client → Supabase Edge Function → KI-API (Key liegt serverseitig)
```

### Storage
- Private Buckets, signierte URLs
- Max. 10 MB, nur jpg/png/webp
- Upload Rate Limit: max. 5 Fotos/Minute pro User

### KI-Kostenkontrolle
- Max. 20 KI-Calls pro User/Tag
- Hard Cap pro User/Monat
- Retry-Schutz bei API-Fehler

### Auth
- Nur Google/Apple OAuth – kein Passwort-Login
- Account-Linking sauber (keine Duplikate)
- Push-Tokens: aktualisiert bei App-Start, gelöscht bei Logout

### Legal Copy
- Bei Empfehlungen: "Basierend auf deinen Angaben – keine Garantie"
- Bei Buddy-State: "Fernli schätzt den Zustand anhand bekannter Faktoren"

---

## 13. Datenbankstruktur (Supabase)

```sql
-- Nutzer
users (
  id uuid PRIMARY KEY,
  email text,
  plan text DEFAULT 'free',         -- free | premium
  timezone text,
  created_at timestamptz,
  last_active_at timestamptz
)

-- Pflanzen
plants (
  id uuid PRIMARY KEY,
  user_id uuid REFERENCES users(id),

  -- Art & Erkennung
  species_id text,
  species_name text,
  species_confidence float,          -- 0.0–1.0
  species_confirmed_by_user bool DEFAULT false,
  species_source text,               -- 'ai' | 'manual'

  -- Setup
  nickname text,
  photo_url text,
  location_city text,
  room_type text,                    -- living_room | bathroom | office | balcony
  window_direction text,             -- N | E | S | W | unknown
  light_assessment_method text,      -- compass | manual
  indoor_outdoor text DEFAULT 'indoor',

  -- Zustand: 4 Basis-Zustände
  current_state text,                -- happy | thirsty | needs_attention | critical
  state_confidence float,
  primary_reason text,               -- immer befüllt, konkreter Grund
  last_state_update_at timestamptz,
  last_user_confirmation_at timestamptz,

  -- Overlays: separat boolesch
  is_sleeping bool DEFAULT false,
  is_winter_overlay bool DEFAULT false,

  -- Pflege
  last_watered_at timestamptz,
  last_photo_check_at timestamptz,
  is_alive bool DEFAULT true,
  plant_status_unknown bool DEFAULT false,  -- true wenn kein belastbarer Zustand ableitbar

  -- Timestamps
  created_at timestamptz,
  deleted_at timestamptz             -- soft delete
)

-- Pflegeaktionen
care_logs (
  id uuid PRIMARY KEY,
  plant_id uuid REFERENCES plants(id),
  action_type text,                  -- watered | fertilized | repotted | confirmed_ok | user_correction
  amount_ml int,
  notes text,
  source text,                       -- 'app' | 'notification_quick_log' | 'manual_correction'
  created_at timestamptz
)

-- Empfehlungen
care_recommendations (
  id uuid PRIMARY KEY,
  plant_id uuid REFERENCES plants(id),
  recommended_action text,           -- water | check | wait | adjust_location
  due_at timestamptz,
  urgency text,                      -- low | medium | high | critical
  confidence float,
  rationale text,                    -- maschinenlesbar
  display_text text,                 -- formulierter Text für Nutzer (via LLM)
  engine_version text,
  was_followed bool,
  created_at timestamptz
)

-- Notification-Logs
notification_logs (
  id uuid PRIMARY KEY,
  plant_id uuid REFERENCES plants(id),
  user_id uuid REFERENCES users(id),
  type text,                         -- water_reminder | needs_attention | check_in | seasonal
  sent_at timestamptz,
  opened_at timestamptz,
  action_taken text                  -- quick_log | opened_app | dismissed | none
)

-- Push Tokens
push_tokens (
  id uuid PRIMARY KEY,
  user_id uuid REFERENCES users(id),
  token text UNIQUE,
  platform text,                     -- ios | android
  created_at timestamptz,
  updated_at timestamptz
)
```

### Bedeutung von `plant_status_unknown`

`plant_status_unknown = true`, wenn Fernli aufgrund fehlender oder widersprüchlicher Daten keinen belastbaren Zustand ableiten kann. Beispiele: sehr lange keine Logs, Pflanzenart nicht bestätigt und Standort unklar, letzte bekannte Daten stark veraltet.

`plant_status_unknown = false`, sobald ein neuer Log, eine Nutzerbestätigung (`confirmed_ok`) oder eine belastbare Engine-Auswertung vorliegt.

Wenn `plant_status_unknown = true`: State wird auf `needs_attention` gesetzt, `primary_reason: "Wir brauchen ein Update – wie geht es deiner Pflanze?"`.

---

## 14. MVP Scope – Final

### ✅ In v1.0
- Google & Apple Login (nach Wow-Moment, via Supabase OAuth)
- Foto → Top-3-Artvorschlag mit Konfidenz → Buddy erscheint
- 3 Pflanzenarten: Monstera, Kaktus, Pothos
- 4 Basis-Zustände: `happy` | `thirsty` | `needs_attention` | `critical`
- 2 Overlays: `is_sleeping` (22:00–07:00) | `is_winter_overlay` (Dez–Feb)
- Reason Layer: `primary_reason` immer befüllt
- Manuelle Korrekturen (Art, Gießdatum, Standort)
- Regelbasierte Care Engine (TypeScript, deterministisch)
- Wetter als leichter Modifier (Open-Meteo)
- Licht-Kalkulator via Kompass
- LLM für `display_text` und Notifications (Edge Function)
- Smart Push Notifications + Quick-Log-Action
- Notification-Caps (max 1/Pflanze/24h, max 3/Woche gesamt Free)
- Pflegehistorie (30 Tage gratis)
- Freemium: 1 Pflanze gratis
- Analytics (PostHog) von Tag 1
- Buddy-Assets: SVG + einfache React-Native-Animationen (Animated API)
- Failure Paths für alle kritischen Onboarding-Schritte

### ❌ Bewusst nicht in v1.0

| Feature | Warum | Kommt wann |
|---|---|---|
| Homescreen Widget | Expo Alpha-Status | v2 |
| Dead / Skelett-State | Fehlende Logs ≠ Vernachlässigung | v2 |
| KI-Problemhinweise | Braucht stabiles Fundament | v1.1 |
| Lottie-Animationen | Asset-Abhängigkeit, SVG reicht für v1 | v2 |
| Deep Scan / 360° | Zu komplex | v2+ |
| Affiliate-Links | Nach erstem Revenue | v2 |
| Urlaubs-Sitter | Phase 2 | v2 |
| Social Features | Phase 2 | v2+ |
| Mehr als 3 Pflanzenarten | Erst validieren, dann ausbauen | v1.1 |

---

## 15. Claude Code – Entwicklungsreihenfolge

### Modul 1: Datenbank & Auth

```
Erstelle die Supabase-Datenbankstruktur für Fernli exakt nach Schema in Abschnitt 13.
Row Level Security für alle Tabellen.
Google & Apple OAuth via Supabase.
Alle API-Keys ausschließlich in Supabase Edge Functions – niemals im Client.
```

### Modul 2: Onboarding Flow

```
Baue den Onboarding-Flow in React Native (Expo):
- Direkt zur Kamera, kein Login davor
- Foto → Vision API via Edge Function → Top 3 Vorschläge mit Konfidenz-Prozent
- Konfidenz < 60%: manuelle Auswahl erzwingen
- Standort-Setup: Stadt, Zimmertyp, Fensterrichtung via Kompass
- Buddy erscheint VOR dem Login
- Login via Supabase OAuth (Google + Apple) zum Speichern
- Failure Paths gemäß Abschnitt 6 für jeden Schritt implementieren
```

### Modul 3: Buddy & Assets

```
Implementiere das Buddy-System für 3 Pflanzenarten: Monstera, Kaktus, Pothos.

4 Basis-Zustände (current_state): happy | thirsty | needs_attention | critical
2 Overlays (separat): is_sleeping (22:00–07:00 lokale Zeit) | is_winter_overlay (Dez–Feb)
Overlays überlagern den Basis-State visuell, ersetzen ihn nicht.

State Priority: critical > thirsty > needs_attention > happy

Assets: SVG-Charaktere mit einfachen React-Native-Animationen (Animated API). Kein Lottie.
primary_reason erscheint als Text unter dem Buddy – immer befüllt, nie leer, maximal 90 Zeichen, nicht-technisch.
Buddy-Reaktionsanimation nach jedem care_log.
```

### Modul 4: Care Engine

```
Baue die regelbasierte Care Engine in TypeScript als Supabase Edge Function.

Input: Pflanzenart + Perenual-Basiswerte, care_logs, Wetter via Open-Meteo (leichter Modifier),
       window_direction, Monat/Jahreszeit, species_confidence

Output:
  current_state: happy | thirsty | needs_attention | critical
  is_sleeping: bool (lokale Uhrzeit 22:00–07:00)
  is_winter_overlay: bool (Dez–Feb)
  plant_status_unknown: bool
    → true wenn keine belastbaren Daten vorhanden
    → wenn true: current_state = needs_attention,
                 primary_reason = "Wir brauchen ein Update – wie geht es deiner Pflanze?",
                 recommended_action = check
  primary_reason: string (immer befüllt, max. 90 Zeichen, nicht-technisch)
  recommended_action: water | check | wait | adjust_location
  due_at: timestamptz
  urgency: low | medium | high | critical
  confidence: float (interne Heuristik, keine wissenschaftliche Wahrscheinlichkeit)
  rationale: string (maschinenlesbar)

Critical-Trigger (einer dieser Fälle):
  - days_since_watering >= adjusted_interval + 5
  - days_since_any_log >= 14 UND letzter bekannter State war nicht 'happy'
  - 3+ negative Modifikatoren gleichzeitig aktiv (gemäß Liste in Abschnitt 5.1)

Fehlende Logs allein (days_since_any_log >= 14) → needs_attention, nicht critical.

State Priority: critical > thirsty > needs_attention > happy

Dann: LLM (Claude Haiku via Edge Function) generiert display_text aus Engine-Output.
LLM-Tonalität je Pflanzenart (Monstera expressiv, Kaktus kurz, Pothos entspannt).
LLM formuliert immer konservativ: "wahrscheinlich", "könnte", nie "definitiv" oder "muss".
```

### Modul 5: Notifications

```
Implementiere Push Notifications via Expo Notifications.
Notification-Text aus care_recommendations.display_text.

Quick-Log-Action direkt aus Notification (ohne App öffnen):
- "Gegossen ✓" → care_log (source: 'notification_quick_log') → State neu berechnen
- "Später" → Reminder in 4 Stunden

Nach Quick-Log: State wird neu berechnet – NICHT automatisch auf 'happy' setzen.

confirmed_ok-Handling (wenn Nutzer "Pflanze sieht okay aus" bestätigt):
- care_log mit action_type = confirmed_ok speichern
- last_user_confirmation_at = now
- check_in-Notifications für 7 Tage unterdrücken
- needs_attention aufgrund Unsicherheit/fehlender Daten abgeschwächen
- critical aufgrund echter Wasser-Überfälligkeit (days_since_watering >= adjusted_interval + 5) NICHT überschreiben
- State danach neu berechnen

Notification-Caps:
- Max 1 dringende Notification pro Pflanze pro 24h
- Max 3 pro Woche gesamt (Free)
- check_in frühestens nach 7 Tagen ohne Log
- Kein Retry innerhalb von 6 Stunden bei Fehler

Typen: water_reminder | needs_attention | check_in | seasonal
Alle gesendeten Notifications in notification_logs speichern.
```

### Modul 6: Analytics

```
Integriere PostHog für Event-Tracking.

Onboarding: app_opened, camera_permission_granted/denied, photo_taken,
            species_recognized (+confidence), species_manually_corrected,
            buddy_appeared, login_completed, onboarding_dropped (+step)

Engagement: app_opened_daily, notification_received/opened,
            quick_log_used, care_log_created, state_viewed,
            species_corrected_post_onboarding, recommendation_followed

Retention: Tag-1/7/30-Marker
```

---

## 16. Technisches Setup

### Was vorhanden ist
Node.js ✅ · iPhone ✅ · Supabase Account ✅ · Anthropic API Key ✅ · VS Code ✅

### Noch einzurichten
- Expo Account (expo.dev)
- Claude Code Terminal-Tool
- Expo Go App (iPhone, App Store)

### Tech Stack

| Bereich | Technologie |
|---|---|
| Frontend | React Native (Expo) |
| Backend & DB | Supabase (Auth, DB, Storage, Edge Functions) |
| Vision Layer | GPT-4o Vision oder Claude Vision (testen) |
| Care Engine | TypeScript (Supabase Edge Function, regelbasiert) |
| Language Layer | Claude Haiku oder GPT-4o mini (via Edge Function) |
| Pflanzendatenbank | Perenual API (v1) → eigene Datenbank (v2) |
| Wetter | Open-Meteo API (kostenlos, kein Key nötig) |
| Buddy-Assets | SVG + React-Native-Animationen (Animated API) (v1) · Lottie (v2) |
| Push Notifications | Expo Notifications |
| Analytics | PostHog |

---

*Fernli Blueprint v3.2 – März 2026*
*Konsistent. Vollständig. Baubar.*
*Letzte Konsistenzrunde abgeschlossen. Bereit für Claude Code.*
