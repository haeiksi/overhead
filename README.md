[README(2).md](https://github.com/user-attachments/files/35517991/README.2.md)
# Overhead – Live-Flugradar im LED-Anzeigetafel-Stil

Eine einzelne `index.html` (HTML, CSS, Vanilla JS – keine Frameworks, keine Build-Schritte). Zeigt das nächstgelegene Flugzeug über deinem Standort, bis zu 3 weitere Flüge und im Leerlauf eine große LED-Uhr.

## 1. Standort einstellen

**Variante A – direkt auf dem Tablet:** Zahnrad oben rechts antippen (erscheint bei Berührung des Displays, blendet sich nach 5 s aus) → Ortsname, Koordinaten, Radius, Intervall und Nachtmodus eintragen → *Speichern*. Die Werte liegen im `localStorage` des Browsers.

**Variante B – im Code:** In `index.html` das Objekt `DEFAULTS` am Anfang des Skripts anpassen (`LAT`, `LON`, `NAME`, `RADIUS_KM`, `POLL_SECONDS`, `NIGHT_START`, `NIGHT_END`).

Am einfachsten: im Einstellungs-Menü auf **„Aktuellen Standort ermitteln“** tippen und den Standortzugriff erlauben. Breiten-/Längengrad und Ortsname werden eingetragen (Ortsname über OpenStreetMap Nominatim, Ersatz BigDataCloud); mit *Speichern* übernimmst du die Werte. Das braucht HTTPS (GitHub Pages passt) und ist auf Tablets ohne GPS nur so genau wie WLAN/Mobilfunk, also eventuell einige 100 m bis km daneben. Manuell findest du Koordinaten z. B. in Google Maps (Rechtsklick auf den Standort → Koordinaten kopieren). Voreingestellt ist Siegertsbrunn (48,0195 / 11,7191). Gespeicherte Einstellungen im Browser haben Vorrang; mit `?reset=1` an der Adresse werden sie gelöscht.

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

- **Datenquellen:** `adsb.fi` zuerst, dann `adsb.lol`, `adsb.one` und `airplanes.live` als Fallback. Jede Quelle wird erst direkt aus dem Browser und dann über den Proxy abgefragt. Fehlgeschlagene Quellen werden pausiert (403: 30 Min, 429: 10 Min, Netz-/CORS-Fehler: 60 Min direkt bzw. 5 Min über Proxy, sonst 1 Min), damit sie nicht mit Anfragen überhäuft werden. Nach Speichern der Einstellungen oder `?reset=1` werden die Pausen aufgehoben. Ohne neue Daten für 2 Minuten wechselt die Anzeige auf „KEINE VERBINDUNG“, statt veraltete Flugzeuge zu zeigen.
- **Radius:** Eingabe in km, intern Umrechnung in Seemeilen (÷ 1,852).
- **Smart-Polling:** Standard alle 15 s (Minimum 10 s). Im **Nachtmodus** (Standard 23:30–06:00) werden keine Abfragen gesendet, das Display wird gedimmt und nur die Uhr angezeigt. Zeitfenster über Mitternacht werden korrekt behandelt.
- **Flugzeug-Foto:** Vorschaubild über `api.planespotters.net` (mit Scanlines und Retro-Filter); ohne Foto wird der Bereich ausgeblendet. Die Fotografen-Nennung wird eingeblendet.
- **Route:** Abflug- und Zielflughafen (IATA-Code und Name) kommen anhand des Rufzeichens von `api.adsbdb.com`. Das klappt nur bei Linienflug-Rufzeichen (z. B. DLH123), die Daten sind community-gepflegt und können veraltet oder falsch sein. Ohne Eintrag steht „ROUTE UNBEKANNT“. Da adsbdb pro Rufzeichen nur eine Route kennt, Rufzeichen aber für verschiedene Flüge wiederverwendet werden, prüft die App die Route gegen die aktuelle Flugzeugposition: liegt das Flugzeug weit abseits der Strecke Abflug–Ziel, steht „ROUTE UNSICHER“ statt einer falschen Route.
- **Flugzeuge am Boden** werden ausgeblendet; Entfernungen werden aus den Positionsdaten selbst berechnet.

## Hinweise

- Die Flugdaten-APIs sind kostenlos, aber für den privaten, nicht-kommerziellen Gebrauch gedacht – das Intervall bitte nicht unter 10 s setzen. adsb.fi verlangt die Nennung der Quelle (steht in der Fußzeile) und einen Link auf <https://adsb.fi>. Nach Berichten (Stand Nov. 2026, nicht offiziell bestätigt) sperrt airplanes.live die öffentliche API für Nicht-Feeder (403), adsb.fi und airplanes.live lehnen Anfragen von Cloudflare-Workern ab, und adsb.lol erlaubt ca. 500 Anfragen pro Tag und IP (danach 429).
- Schriften (Share Tech Mono, VT323) kommen von Google Fonts; offline wird automatisch eine Monospace-Systemschrift verwendet.
- Läuft ohne Offline-Cache; ohne Internet zeigt die Seite „KEINE VERBINDUNG“ und versucht es weiter.

## Wenn „Failed to fetch“ erscheint: CORS-Proxy einrichten

Manche ADS-B-APIs (z. B. airplanes.live, adsb.lol) erlauben Browser-Abfragen von fremden Seiten nicht oder verlangen inzwischen Zugang. Dann blockiert der Browser die Antwort. Lösung: ein kleiner Proxy, der die Anfrage weiterleitet und die nötigen CORS-Header ergänzt. Dafür liegt `worker.js` bei (Cloudflare Workers, kostenloser Account reicht).

1. Auf <https://dash.cloudflare.com> → *Workers & Pages* → *Create* → *Hello World* Worker anlegen.
2. *Edit code* → den Inhalt von `worker.js` einfügen → *Deploy*.
3. Die Worker-Adresse (z. B. `https://overhead.dein-name.workers.dev/`) kopieren.
4. In Overhead: Zahnrad → Feld **CORS-Proxy** → Adresse eintragen → *Speichern*. Alternativ im Code bei `DEFAULTS` unter `PROXY_URL` eintragen.

Der Worker erlaubt nur die in `ALLOWED` genannten Hosts und cached Antworten 10 Sekunden. Die Fußzeile zeigt pro Quelle den Fehler (z. B. `HTTP 403`), damit du siehst, welche Quellen antworten.

## Einstellungen öffnen, falls das Zahnrad nicht erreichbar ist

- Taste **S** (oder `,`) drücken, **Esc** schließt wieder.
- **3x schnell** auf den Ortsnamen in der Kopfzeile tippen.
- Per URL konfigurieren (wird gespeichert, die Adresszeile wird danach bereinigt):
  `https://<name>.github.io/overhead/?lat=50.11&lon=8.68&name=Frankfurt&radius=20&proxy=https://overhead.dein-name.workers.dev/`
  Weitere Parameter: `poll` (Sekunden). Mit `?reset=1` werden die gespeicherten Einstellungen gelöscht.

**Hinweis zum Worker:** `worker.js` erlaubt jetzt zusätzlich `api.adsbdb.com` (für die Route). Nach dem Update den Code im Cloudflare-Worker ersetzen und neu deployen. Ohne das fragt die Seite die Route direkt ab, was je nach Browser an CORS scheitern kann.
