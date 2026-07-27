# tillalbert.de

Persönliche Website von Prof. Dr. Till Albert. Statisches HTML, kein Build-Schritt,
keine Abhängigkeiten außer den mitgelieferten Dateien.

## Wo liegt was

```
TillAlbert.dc.html   Die gesamte Seite: Inhalte (DE/EN), Styles, Impressum,
                     Datenschutz. Quellformat von Claude Design (.dc.html).
support.js           Laufzeit von Claude Design. Interpretiert <x-dc>,
                     <helmet> und <sc-if> und rendert die Seite.
fonts/               Selbst gehostete Webfonts (woff2).
vendor/              React, ReactDOM und Babel — lokal statt vom CDN.
Till-400x600.jpg     Porträtfoto.
netlify.toml         Deployment-Konfiguration (siehe unten).
```

Die Seite ist zweisprachig. Beide Sprachfassungen stehen im HTML nebeneinander
(`<span data-lang="de">` / `<span data-lang="en">`); umgeschaltet wird per CSS
über `data-active-lang` am Wurzelelement. Wer Text ändert, muss **beide**
Fassungen anfassen.

Impressum und Datenschutz sind keine eigenen Dateien, sondern Unterseiten im
selben Dokument (`<sc-if value="{{ showImpressum }}">` bzw. `showDatenschutz`),
umgeschaltet über den Zustand in der Komponentenklasse am Dateiende.

## Warum fonts/ und vendor/ lokal liegen

Beides ist eine Datenschutzentscheidung, kein Performance-Tuning.

Ursprünglich lud die Seite die Schriften von Google Fonts und React/Babel von
`unpkg.com`. In beiden Fällen wird beim bloßen Seitenaufruf die IP-Adresse der
Besucher an einen Dritten in den USA übertragen — ohne Einwilligung und ohne
dass es sich vermeiden ließe. Genau das war Gegenstand der Google-Fonts-
Rechtsprechung.

Deshalb:

- **fonts/** — Roboto Slab (Apache 2.0), IBM Plex Sans und IBM Plex Mono
  (SIL OFL 1.1). Google Fonts liefert diese Familien als Variable Fonts aus,
  eine Datei deckt daher alle Schnitte ab. Die `@font-face`-Regeln stehen im
  `<helmet>`-Block des HTML.

- **vendor/** — die drei Bibliotheken, die `support.js` sonst von `unpkg.com`
  nachlädt. `support.js` bringt dafür einen Override mit: `window.__resources`
  bildet die CDN-URL auf einen lokalen Pfad ab. Der Block steht im `<head>`
  **vor** dem Einbinden von `support.js` — die Reihenfolge ist zwingend.

Die Dateien in `vendor/` sind unveränderte Originale und wurden gegen die
SRI-Hashes geprüft, die in `support.js` hinterlegt sind. Die Dateinamen tragen
die Versionsnummer, damit dauerhaftes Caching gefahrlos ist.

Wenn `support.js` jemals ausgetauscht wird, muss geprüft werden, ob sich die
erwarteten CDN-URLs geändert haben. Passt der Schlüssel in `window.__resources`
nicht mehr zeichengenau, greift der Override still nicht mehr und die Seite lädt
wieder von `unpkg.com` — ohne dass etwas sichtbar kaputtgeht.

## Deployment

Netlify, Domain bei IONOS registriert. `netlify.toml` setzt:

- `publish = "."` — kein Build, Auslieferung direkt aus dem Wurzelverzeichnis.
- Rewrite von `/` auf `/TillAlbert.dc.html`. Die Seite heißt nicht `index.html`,
  ohne diese Regel liefert `/` einen 404.
- Cache-Header und ein Satz Sicherheits-Header.

## Lokal ansehen

Ein einfaches Öffnen per Doppelklick reicht nicht — die Seite braucht einen
HTTP-Server, sonst blockiert der Browser das Laden der Schriften.

```bash
python3 -m http.server 8000
# http://localhost:8000/TillAlbert.dc.html
```

## Prüfung vor dem Livegang

Ein Blick ins Netzwerk-Panel der Entwicklerwerkzeuge sollte **keine** Anfragen
an andere Hosts als die eigene Domain zeigen. Taucht dort `fonts.googleapis.com`,
`fonts.gstatic.com` oder `unpkg.com` auf, ist die Aussage in der
Datenschutzerklärung nicht mehr zutreffend.
