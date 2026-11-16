# Evelien's NL Concert Watch

Wekelijkse routine die controleert of Evelien's favoriete artiesten (muzikanten en comedians) optreden in Nederland, of in Düsseldorf, Parijs, Brussel, Londen of Oslo. De resultaten staan op [evelien.concerts.smartism.art](https://evelien.concerts.smartism.art).

Gebaseerd op [NL-Concert-Watch-MZ](https://github.com/s-m-a-r-t-ism/NL-Concert-Watch-MZ), maar volledig los daarvan: eigen repo, eigen artiestenlijst, eigen scheduled task (`evelien-concert-watch`).

## Bestanden

- `concert-watch-state.json`: de gevolgde artiestenlijst, alle al gemelde optredens (`known_shows`) en het tijdstip van de laatste run. Wordt elke week bijgewerkt door de scheduled task.
- `index.html`: de website, leest `concert-watch-state.json` in.
- `scheduled-task-prompt.md`: de instructies voor de scheduled task, plus installatie-uitleg.
- `precheck.ps1`: snelle voorcheck (git pull, zelf-sync van de instructies, dubbel-werk-check).
- `CNAME`: custom domain voor GitHub Pages.

## Scheduled task instellen

De scheduled task staat lokaal in de Claude-app (`~/.claude/scheduled-tasks/evelien-concert-watch/`) en moet apart aangemaakt worden op elke machine waar je 'm wil laten draaien.

- **Schema:** elke zondagavond 21:30, cron `30 21 * * 0` (een half uur na de eigen NL Concert Watch, zodat ze elkaar niet in de weg zitten)
- **Prompt:** zie `scheduled-task-prompt.md`. Kopieer de routine-sectie letterlijk als prompt.

## Rapportage

De routine stuurt alleen een chatbericht in Claude (geen e-mail/Gmail-concepten) met nieuwe optredens sinds de vorige run, of een kort berichtje als er niets nieuws is. Evelien ziet alles op de website.
