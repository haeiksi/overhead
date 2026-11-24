[README(1).md](https://github.com/user-attachments/files/35352786/README.1.md)
# overhead
Displaying Planes in your area
# Overhead – Live-Flugradar im LED-Anzeigetafel-Stil

Eine einzelne `index.html` (HTML, CSS, Vanilla JS – keine Frameworks, keine Build-Schritte). Zeigt das nächstgelegene Flugzeug über deinem Standort, bis zu 3 weitere Flüge und im Leerlauf eine große LED-Uhr.

## 1. Standort einstellen

**Variante A – direkt auf dem Tablet:** Zahnrad oben rechts antippen (erscheint bei Berührung des Displays, blendet sich nach 5 s aus) → Ortsname, Koordinaten, Radius, Intervall und Nachtmodus eintragen → *Speichern*. Die Werte liegen im `localStorage` des Browsers.

**Variante B – im Code:** In `index.html` das Objekt `DEFAULTS` am Anfang des Skripts anpassen (`LAT`, `LON`, `NAME`, `RADIUS_KM`, `POLL_SECONDS`, `NIGHT_START`, `NIGHT_END`).

Koordinaten findest du z. B. in Google Maps (Rechtsklick auf den Standort → Koordinaten kopieren). Standard ist Frankfurt – bitte ändern.

## 2. Lokal starten

Auf dem PC im Projektordner:

```bash
python3 -m http.server 8000
```

- Am PC: <http://localhost:8000>
- Am Tablet (gleiches WLAN): `http://<IP-des-PCs>:8000`

Die Datei lässt sich auch direkt per Doppelklick öffnen (`file://`). Meist funktionieren die API-Abfragen dann trotzdem; Wake Lock und Vollbild sind dort aber oft eingeschränkt.

## 3. Empfohlen: GitHub Pages (HTTPS)

1. Neues GitHub-Repository anlegen, `index.html` (und diese README) hochladen.
2. *Settings → Pages → Branch: main / root* aktivieren.
3. Auf dem Tablet die Adresse `https://<dein-name>.github.io/<repo>/` öffnen.

HTTPS ist die Voraussetzung dafür, dass der **Screen Wake Lock** im Browser funktioniert.

## 4. Android-Tablet einrichten

1. Seite in **Chrome** öffnen.
2. Menü (⋮) → **Zum Startbildschirm hinzufügen** → so startet die Seite ohne Adressleiste.
3. Alternativ das Vollbild-Symbol unten rechts antippen (`requestFullscreen`).
4. Falls die Meldung „WAKE LOCK NICHT VERFÜGBAR“ in der Fußzeile erscheint: *Android-Einstellungen → Display → Bildschirm-Timeout → Nie* (bzw. Entwickleroptionen → „Aktiv lassen“ beim Laden) und das Tablet ans Netzteil hängen.
5. Optional: Kiosk-/Bildschirm-Fixierung nutzen, damit niemand versehentlich die Seite verlässt.

## Funktionen im Überblick

- **Datenquellen:** `api.airplanes.live` mit automatischem Fallback auf `api.adsb.lol` (bei Fehler, Timeout oder Rate-Limit).
- **Radius:** Eingabe in km, intern Umrechnung in Seemeilen (÷ 1,852).
- **Smart-Polling:** Standard alle 15 s (Minimum 10 s). Im **Nachtmodus** (Standard 23:30–06:00) werden keine Abfragen gesendet, das Display wird gedimmt und nur die Uhr angezeigt. Zeitfenster über Mitternacht werden korrekt behandelt.
- **Flugzeug-Foto:** Vorschaubild über `api.planespotters.net` (mit Scanlines und Retro-Filter); ohne Foto wird der Bereich ausgeblendet. Die Fotografen-Nennung wird eingeblendet.
- **Flugzeuge am Boden** werden ausgeblendet; Entfernungen werden aus den Positionsdaten selbst berechnet.

## Hinweise

- Beide Flugdaten-APIs sind kostenlos, aber für den privaten Gebrauch gedacht – das Intervall bitte nicht unter 10 s setzen.
- Schriften (Share Tech Mono, VT323) kommen von Google Fonts; offline wird automatisch eine Monospace-Systemschrift verwendet.
- Läuft ohne Offline-Cache; ohne Internet zeigt die Seite „KEINE VERBINDUNG“ und versucht es weiter.
