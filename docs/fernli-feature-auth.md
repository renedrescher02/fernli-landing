# 🔐 Feature-Konzept: User Authentication & Onboarding

## 1. Login-Methoden (Social & Email)

Um eine maximale Conversion-Rate beim ersten App-Start zu erreichen, wird ein hybrider Login-Ansatz verfolgt. Das Ziel ist es, den Zugang zur App so barrierefrei wie möglich zu gestalten.

* **🍎 Sign in with Apple:**
    * **Vorteil:** Nahtlose Integration für iOS-Nutzer.
    * **Privacy:** Unterstützung von "E-Mail-Adresse verbergen" für maximalen Datenschutz.
    * **Requirement:** Zwingend erforderlich für den App Store, sobald andere Social-Logins angeboten werden.

* **🔷 Google Sign-In:**
    * **Vorteil:** Schneller Zugang für Android-Nutzer und User mit Google-Konto auf iOS.
    * **Sync:** Erleichtert die Synchronisation von Daten über verschiedene Geräte hinweg.

* **📧 Klassischer E-Mail-Login:**
    * **Option:** Für Nutzer, die keine Social-Logins verwenden möchten.
    * **Prozess:** Passwort-Management und "Passwort vergessen"-Workflow.

---

## 2. Der "Fast-Entry" Workflow

Der Login-Prozess wird direkt mit dem ersten Pflanzen-Setup verknüpft, um den "Time-to-Value" zu verkürzen.

1. **Welcome Screen:** Kurze visuelle Einführung in die Plant-Buddy-Welt.
2. **Auth-Wall:** Auswahl der Login-Methode (Apple / Google / Email).
3. **Handoff:** Nach dem Login startet sofort die Wahl zwischen **Quick Mode** oder **Deep Scan**, damit der Nutzer direkt seine erste Pflanze anlegen kann.

---

## 3. Technische & UX-Anforderungen

* **Cross-Platform Sync:** Login stellt sicher, dass die digitalen Buddies auf allen Geräten (iPhone, iPad) synchron sind.
* **Security:** Verschlüsselte Speicherung der Nutzerdaten gemäß DSGVO-Standards.
* **One-Click Registration:** Bei Social-Logins wird das Nutzerprofil (Name/Bild) automatisch generiert — kein Formular ausfüllen.

---

## 4. Business Logic: Warum Social Login?

* **Conversion Rate:** Reduziert die Absprungrate im Onboarding um bis zu 50 %, da kein neues Passwort erstellt werden muss.
* **Trust:** Bekannte Login-Anbieter erhöhen das Vertrauen in eine neue App.
* **Retention:** Einfacheres Wiederanmelden nach Neuinstallation oder Gerätewechsel.

---

*Erstellt: März 2026 — Fernli v1 Feature Planning*
