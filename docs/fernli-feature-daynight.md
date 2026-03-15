# 🌙 Fernli Feature Spec — Day/Night Buddy Cycle

## Die Idee
Die Pflanzen-Charaktere in Fernli folgen dem echten Tagesrhythmus des Nutzers.
Tagsüber sind sie wach und aktiv. Nachts schlafen sie — mit geschlossenen Augen, 
Schlafmütze und sanften Zzz-Partikeln.

**Kerngefühl:** Deine Pflanze ist ein Mitbewohner, kein Werkzeug.
Öffnest du die App um 2 Uhr nachts, schläft dein Kaktus friedlich. 
Du willst ihn nicht wecken. Das gibt der App eine Seele.

**Referenz:** Tamagotchi-Effekt + Animal Crossing Echtzeit-Rhythmus + Duolingo Streak-Gefühl.

---

## Technische Umsetzung

### Zeitlogik (1 Zeile Code)
```javascript
const hour = new Date().getHours();
const isNight = hour >= 22 || hour < 7;
const isDawn  = hour >= 7  && hour < 9;   // sanfter Übergang
```

Keine API. Kein GPS. Nur die Systemzeit des Handys.

### Zwei Zustände pro Charakter

**Tag-Modus (7:00 – 21:59)**
- Normale SVG-Darstellung (wach, Augen offen)
- Hintergrund: dunkles Grün mit warmem Sonnenschein-Effekt von oben
- Leichte Blatt-Wackel-Animation (CSS)
- Normales sattes Grün

**Nacht-Modus (22:00 – 6:59)**
- Augen geschlossen: `ᵕ ᵕ` (zwei sanfte Bogen-Striche)
- Mini-Schlafmütze auf dem Kopf (kleines SVG-Element)
- Leichter Blau-Tint auf der Pflanze (Mondlicht-Effekt via CSS filter)
- Sanftes "Atmen": SVG skaliert langsam 1–2% auf und ab
- Zzz-Partikel die nach oben schweben (CSS keyframe animation)
- Hintergrund: leicht bläulicher Dunkel-Ton statt grün

### Interaktion: "Don't wake the Buddy!"
Wenn Nutzer nachts auf die schlafende Pflanze tippt:
- Pflanze macht kurz ein Auge auf, guckt verschlafen, schließt es wieder
- Text erscheint: *"Psst... Luna schläft gerade. Sie hat heute viel Photosynthese betrieben."*

---

## Copy / Wording

**Feature-Name:** Living with you, not just for you.

**Landing Page Teaser:**
> Your plant buddies follow your rhythm.  
> They wake up with the sun and tuck themselves in when you go to bed.  
> Open the app at night to find them dreaming. 🌙

**In-App Nachricht (Nacht):**
> "Psst — [Pflanzenname] schläft gerade. Bis morgen früh! 🌙"

---

## Aufwand-Schätzung

| Was | Aufwand |
|-----|---------|
| Zeitlogik | ~5 Min |
| Nacht-SVG-Zustand pro Charakter | ~30 Min |
| CSS Breathing-Animation | ~15 Min |
| Zzz-Partikel Animation | ~20 Min |
| "Tap to wake" Interaktion | ~30 Min |
| **Total** | **~2 Stunden** |

---

## Priorität & Nächste Schritte

1. Schlafenden Kaktus als SVG bauen (erster visueller Proof)
2. Zeitlogik ins bestehende HTML einbauen
3. Auf Landing Page als Teaser-Feature erwähnen
4. Für App-Launch als Core-Feature einplanen

---

*Erstellt: März 2026 — Fernli v1 Feature Planning*
