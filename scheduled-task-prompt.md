---
name: Evelien concert watch
description: Wekelijkse check op nieuwe concert-, festival- en comedy-aankondigingen in Nederland en in Düsseldorf, Parijs, Brussel, Londen en Oslo, voor Evelien's vaste artiestenlijst
---

Je controleert of er nieuwe concert-, festival- of comedyoptredens zijn aangekondigd in Nederland en in vijf buitenlandse steden (zie STAP 2) voor een vaste lijst favoriete artiesten van Evelien (een vriendin van de gebruiker). De gebruiker beheert deze routine; Evelien bekijkt de resultaten op de website evelien.concerts.smartism.art.

Deze routine kan op meerdere machines draaien tegen dezelfde GitHub-repo (https://github.com/s-m-a-r-t-ism/Evelien-NL-Concert-Watch). Alle paden hieronder zijn expres generiek/relatief geformuleerd zodat deze instructies ONGEWIJZIGD op elke machine werken. Gebruik nooit een hardcoded absoluut pad (zoals een specifieke gebruikersnaam of schijfletter) in deze routine of in updates erop.

Let op: dit is NIET de eigen NL Concert Watch van de gebruiker (repo NL-Concert-Watch-MZ, scheduled task nl-concert-watch). Die twee routines staan volledig los van elkaar. Raak in deze routine nooit bestanden of de repo van NL-Concert-Watch-MZ aan.

STAP 0 — Snelle voorcheck (ALTIJD als allereerste actie, in één enkele tool-aanroep, zonder eerst rond te kijken):
Voer dit ene PowerShell-commando uit vanuit de working directory van deze taak (het werkt zowel als die map de repo-clone zelf is als wanneer de clone in een submap Evelien-NL-Concert-Watch staat):
`$d = if (Test-Path .\precheck.ps1) { '.' } else { '.\Evelien-NL-Concert-Watch' }; powershell -NoProfile -ExecutionPolicy Bypass -File "$d\precheck.ps1"`
Het script precheck.ps1 (in de repo) doet STAP 1b volledig voor je: git pull, de SKILL.md-zelf-sync, het lezen van machine-name.txt en de dubbel-werk-check. Het geeft regels terug als REPO:, PULL:, SKILL:, MACHINE: en als laatste RESULT:.
- "RESULT: SKIP ...": stop direct. Doe niets anders (geen bestanden lezen, geen WebSearches, geen commit/push). Stuur alleen een kort berichtje als "Evelien Concert Watch: al deze week gecheckt op <datum> door <machine>, sla over om dubbel werk te voorkomen." plus, als SKILL: UPDATED meldt, de zin "SKILL.md was verouderd t.o.v. de repo en is bijgewerkt voor de volgende run.", en de regel "(Run door: <MACHINE>)". Dit is de volledige output van deze run.
- "RESULT: GO ...": ga door met STAP 1 (STAP 1b is dan al gedaan, sla die over). Gebruik het REPO:-pad als de map voor alle verdere bestands- en git-acties.
- MACHINE: MISSING: handel af zoals beschreven in STAP 1.
- Alleen als het script zelf niet gevonden wordt of crasht: val terug op de handmatige STAP 1 en 1b hieronder.

STAP 1 — Lees de trackingstate en herken deze machine:
Lees het bestand concert-watch-state.json in de working directory van deze scheduled task (dit IS je lokale clone van bovenstaande repo; twijfel je, controleer dan met `git remote -v` dat dit s-m-a-r-t-ism/Evelien-NL-Concert-Watch is). Dit bevat:
- "artists": de lijst met artiesten om te volgen, ALFABETISCH gesorteerd. Houd deze volgorde aan als je de lijst afwerkt en rapporteert, zodat de gebruiker de voortgang kan volgen.
- "known_shows": een lijst van eerder gevonden/gerapporteerde optredens (elk met minstens artist, date, venue_or_city, source_url, en optioneel ticketswap_url)
- "last_run": tijdstip van de vorige run (kan null zijn bij de eerste run)
- "last_run_by": naam van de machine die de vorige run deed (kan null zijn)

Is de "artists"-lijst leeg: stop direct, wijzig niets en commit/push niets. Stuur alleen het bericht "Evelien Concert Watch: de artiestenlijst is nog leeg, voeg eerst artiesten toe aan concert-watch-state.json." plus de "Run door: ..."-regel.

Lees ook %USERPROFILE%\.claude\scheduled-tasks\evelien-concert-watch\machine-name.txt. Dit bevat de vaste, door de gebruiker gekozen naam van DEZE machine. Dit bestand is lokaal per machine (staat niet in de git repo). Gebruik deze naam in STAP 6 en STAP 7. Als het bestand niet bestaat, kijk dan of %USERPROFILE%\.claude\scheduled-tasks\nl-concert-watch\machine-name.txt bestaat en neem die naam over (kopieer het bestand). Bestaat dat ook niet, vraag de gebruiker eenmalig hoe deze machine genoemd moet worden, maak het bestand aan met dat antwoord, en ga daarna verder.

STAP 1b — Dubbel-werk-check en zelf-sync (wordt normaal al door precheck.ps1 in STAP 0 gedaan; onderstaande is de handmatige fallback):
Voordat je de zoekronde start, haal eerst de laatste stand van de repo op (git pull --rebase origin main, vanuit de working directory van deze taak). Een andere machine kan intussen al een nieuwere state gepusht hebben, of de routine-instructies zelf hebben bijgewerkt.

Zelf-sync: vergelijk direct na de git pull de routine-sectie van het zojuist opgehaalde scheduled-task-prompt.md (alles vanaf de frontmatter "---" tot aan de "---" vóór de installatiesectie) met je eigen lokale SKILL.md-bestand (%USERPROFILE%\.claude\scheduled-tasks\evelien-concert-watch\SKILL.md, het bestand dat je nu als instructies gebruikt).
- Zijn ze inhoudelijk gelijk: niets doen, ga verder.
- Verschillen ze: overschrijf de lokale SKILL.md met de routine-sectie uit scheduled-task-prompt.md (letterlijk, met de generieke paden; pas ze NIET aan naar een lokaal pad), zodat de VOLGENDE run met de actuele instructies start. Deze huidige run loopt gewoon af met de instructies waarmee hij begonnen is. Vermeld dit kort in het chatbericht bij STAP 6, bijv. "SKILL.md was verouderd t.o.v. de repo en is bijgewerkt voor de volgende run."

Dubbel-werk-check: lees daarna "last_run" opnieuw uit het (mogelijk bijgewerkte) concert-watch-state.json.
- Als "last_run" niet null is EN minder dan 6 dagen geleden ligt (t.o.v. nu): stop hier. Doe GEEN WebSearches, wijzig de state niet, en commit/push niets (de eventuele SKILL.md-zelf-sync mag wel al zijn gebeurd). Stuur alleen een kort berichtje zoals "Evelien Concert Watch: al deze week gecheckt op <datum/machine indien bekend>, sla over om dubbel werk te voorkomen." Dit is dan de volledige output van deze run.
- Als "last_run" null is, of 6 dagen of langer geleden ligt: ga gewoon door met STAP 2.

STAP 2 — Zoek nieuwe optredens (HEEL Nederland + vijf buitenlandse steden, alle soorten locaties):
Het zoekgebied is:
- heel Nederland, en
- Düsseldorf (Duitsland), Parijs (Frankrijk), Brussel (België), Londen (Verenigd Koninkrijk) en Oslo (Noorwegen), inclusief venues en festivals in de directe omgeving/regio van die steden (bijv. Brussel: ook Vorst/Forest National; Düsseldorf: ook arenas in de buurt zoals in Keulen/Oberhausen alleen als ze binnen ca. 50 km liggen; Londen: ook Wembley, The O2, Hyde Park-concerten; Parijs: ook La Défense Arena, Saint-Denis).
Optredens in andere buitenlandse steden horen er NIET bij, ook niet als ze dichtbij Nederland liggen (bijv. Antwerpen, Gent, Hamburg, Berlijn of Manchester tellen niet mee).
Zet bij buitenlandse optredens altijd stad en land in "venue_or_city", bijv. "The O2, Londen (VK)" of "Forest National, Brussel (BE)". Bij Nederlandse optredens is alleen "venue, stad" genoeg.

Gebruik WebSearch om voor elke artiest in de lijst (in alfabetische volgorde) te controleren op aangekondigde optredens in dit zoekgebied: concerten, clubshows, festivals, comedy-/stand-upshows, maar ook kleine, alternatieve, tijdelijke of ongebruikelijke locaties (bijv. kerken, industriële locaties, buitenpodia, kleine cafépodia, pop-up locaties, boerderijen, campings). Zoek dus NIET alleen naar de grote/bekende venues. Gebruik brede zoektermen zoals "<artiest> Nederland 2026 tickets", "<artiest> concert Netherlands", "<artiest> Nederland optreden", site:songkick.com, site:eventim.nl, site:ticketmaster.nl, site:podiuminfo.nl, en algemene zoekopdrachten zonder venue-namen vooraf in te vullen, zodat ook onbekende/kleine podia naar boven komen. Het handigst is vaak eerst de officiële tourpagina of Songkick/Bandsintown-pagina van de artiest: daar zie je in één keer of er data in Nederland of in een van de vijf steden zijn. Voor de buitenlandse steden zijn ook nuttig: "<artiest> <stad> 2026" (bijv. "<artiest> London 2026", "<artiest> Paris concert", "<artiest> Düsseldorf Konzert", "<artiest> Bruxelles concert", "<artiest> Oslo konsert"), site:ticketmaster.co.uk, site:ticketmaster.fr, site:ticketmaster.be, site:eventim.de, site:ticketmaster.no.

Artiestspecifieke tips:
- Michael McIntyre en Nikki Glaser zijn comedians: zoek naar stand-up-/comedyshows en -tours (bijv. "<naam> tour 2026", "<naam> stand-up London"), niet naar concerten.
- Antoon is een Nederlandse zanger; de naam is ook een gewone voornaam, dus zoek met varianten als "Antoon zanger tour", "Antoon concert" en check dat het om de zanger gaat.
- Ma$e (Amerikaanse rapper) staat ook bekend als "Mase"; zoek met beide spellingen en check dat het om de rapper gaat.
- Q-Tip (van A Tribe Called Quest): tel ook optredens mee als A Tribe Called Quest waar Q-Tip deel van uitmaakt; noem dat dan in venue_or_city of in het chatbericht.
- FS Green is een Nederlandse dj/producer: let ook op clubnachten en dance-festivals.
- Ikkimel is een Duitse rapper: Düsseldorf/omgeving en Duitse ticketsites (eventim.de) zijn hier extra relevant.

Zoek zonder tijdslimiet, ook naar shows die pas over een jaar of later plaatsvinden.

Werk hierbij bewust LICHT (token-efficiënt), dit is een routinematige wekelijkse check, geen diepgravend onderzoek:
- Spawn GEEN parallelle subagents om de artiestenlijst te verdelen. Doorloop de lijst gewoon zelf, artiest voor artiest.
- Doe per artiest in principe 1 brede zoekopdracht. Alleen als die een treffer oplevert, doe je een tweede/derde zoekopdracht om de datum/venue/officiële link te bevestigen (zie STAP 3). Bij "niets gevonden" ga je gewoon door naar de volgende artiest.
- Ga niet op zoek naar en verifieer geen geruchten, "farewell tour"-praatjes, tribute-acts of nepnieuws-achtige claims. Als een bron niet betrouwbaar oogt (geen venue/festival/ticketing-site), sla 'm over in plaats van 'm actief te ontkrachten.
- Voor artiesten die niet meer bestaan of al jaren niet meer touren volstaat een enkele snelle controlezoekopdracht.

STAP 3 — Zoek de venue-link (BELANGRIJK):
Songkick, Ticketmaster, Bandsintown, setlist.fm e.d. zijn prima om een optreden te ONTDEKKEN, maar het uiteindelijke "source_url"-veld moet zoveel mogelijk naar de EIGEN pagina van de venue/het festival zelf linken (bijv. paradiso.nl, afaslive.nl, tivolivredenburg.nl, pinkpop.nl, ziggodome.nl, downtherabbithole.nl, patronaat.nl, en in het buitenland bijv. theo2.co.uk, accorhotelsarena.com, forest-national.be, pspdaeiarena.de, oslospektrum.no), niet naar een reseller of aggregator. Doe dus na het vinden van een show altijd een extra zoekopdracht als "<venue naam>.nl <artiest> <datum>" om de officiële eventpagina te vinden. Als er geen specifieke eventpagina op de venue-site bestaat, gebruik dan de algemene agenda-pagina van de venue zelf. Alleen als er echt geen enkele officiële venue/festival-pagina te vinden is, mag je terugvallen op de beste beschikbare aggregator-link.

STAP 4 — Bepaal wat nieuw is:
Vergelijk elke gevonden show met "known_shows" (op basis van artiest + datum + venue). Alles wat niet al in known_shows staat, is "nieuw". Bij de allereerste run (last_run is null) is de hele gevonden lijst per definitie "nieuw"; rapporteer die volledig als startpunt.

STAP 5 — Zoek de TicketSwap-link voor elk nieuw gevonden optreden:
Zoek voor elk NIEUW optreden (uit STAP 4) ook de bijbehorende TicketSwap-eventpagina op (bijv. via WebSearch met "<artiest> <venue> <datum> ticketswap" of site:ticketswap.nl / site:ticketswap.com; TicketSwap werkt ook in het VK, Frankrijk, België, Duitsland en Noorwegen), en noteer die URL als "ticketswap_url". Als je geen exacte match vindt voor de juiste artiest/venue/datum, laat "ticketswap_url" dan weg in plaats van te gokken of een verkeerde/algemene link te gebruiken.

STAP 6 — Rapporteer aan de gebruiker:
Stuur ALTIJD een kort chatbericht naar de gebruiker (dit is de enige afgesproken manier van rapporteren: GEEN e-mail versturen, geen Gmail-concepten aanmaken). Begin het bericht met "Evelien Concert Watch", zodat het niet verward wordt met de eigen NL Concert Watch van de gebruiker.
- Sluit af met een regel welke machine de run deed, bijv. "(Run door: WP-RDP)".
- Als er nieuwe optredens zijn: noem per nieuw optreden de artiest, datum, venue/stad, de bron-URL, en (indien gevonden) de TicketSwap-link. Groepeer alfabetisch op artiest.
- Als er niets nieuws is: stuur alleen een kort berichtje zoals "Evelien Concert Watch: geen nieuwe optredens deze week." plus de "Run door: ..."-regel.
- Controleer voordat je iets als "nieuw" rapporteert of de datum niet al in het verleden ligt t.o.v. vandaag.

STAP 7 — Werk de state bij:
Schrijf concert-watch-state.json (in dezelfde working directory als STAP 1) opnieuw weg met:
- dezelfde "artists"-lijst, alfabetisch gesorteerd (tenzij de gebruiker later een update geeft)
- "known_shows" aangevuld met alle nieuw gevonden shows, elk met artist, date, venue_or_city, source_url, en ticketswap_url indien gevonden in STAP 5 (verwijder geen oude shows, ook niet als het optreden al is geweest; dat voorkomt dubbele meldingen)
- "last_run" op de huidige datum/tijd
- "last_run_by" op de machinenaam uit machine-name.txt (zie STAP 1)

STAP 8 — Publiceer de bijgewerkte state naar GitHub:
Commit en push het bijgewerkte concert-watch-state.json naar de GitHub-repo (https://github.com/s-m-a-r-t-ism/Evelien-NL-Concert-Watch.git), zodat de website evelien.concerts.smartism.art automatisch up-to-date blijft. Haal eerst de laatste stand op om conflicten te voorkomen. Voer uit vanuit de working directory van deze taak (je lokale repo-clone):
- git add concert-watch-state.json
- git commit -m "Update concert-watch-state.json - <datum van vandaag>" (sla deze stap over als er niets gewijzigd is)
- git pull --rebase origin main
- git push origin main

De artiestenlijst staat in concert-watch-state.json onder "artists" (dat is de enige bron, er staat bewust geen kopie in deze instructies).

Belangrijk: verstuur GEEN e-mails en maak geen Gmail-concepten. De gebruiker heeft expliciet gekozen voor alleen een chatbericht in Claude; Evelien kijkt zelf op de website.

---

INSTALLATIE / UPDATE OP EEN ANDERE MACHINE:
Scheduled tasks in Claude Code staan lokaal per machine geregistreerd (niet gesynchroniseerd via git). Om deze routine op een pc te (her)installeren:
1. Zorg dat op die pc git is ingesteld met lees- en schrijftoegang tot https://github.com/s-m-a-r-t-ism/Evelien-NL-Concert-Watch.git.
2. Kloon/pull deze repo lokaal, op een zelfgekozen pad. Dat pad hoeft nergens anders te worden vastgelegd, want deze instructies verwijzen er nooit hardcoded naar.
3. Kopieer de inhoud van dit bestand (alles boven deze installatiesectie, dus vanaf de "---" frontmatter t/m de laatste alinea, LETTERLIJK inclusief de generieke paden) naar %USERPROFILE%\.claude\scheduled-tasks\evelien-concert-watch\SKILL.md op die machine. Deze installatiesectie zelf hoort niet in de SKILL.md.
4. Registreer de scheduled task op die machine (bijv. via de `schedule`-skill of het `mcp__scheduled-tasks__create_scheduled_task`-tool), met als working directory de repo-clone uit stap 2, en als planning: wekelijks op zondag om 21:30 (cron `30 21 * * 0`), een half uur na de eigen NL Concert Watch.
5. machine-name.txt wordt bij de eerste run overgenomen van de nl-concert-watch-taak als die er is, of anders gevraagd. Dit bestand hoort NIET in git.
6. Bij toekomstige inhoudelijke updates aan deze routine synchroniseert elke machine zichzelf automatisch bij zijn eerstvolgende geplande run (STAP 1b). Wil je een update meteen laten doorwerken, laat Claude dan op die machine uitvoeren: "Doe git pull in de lokale clone van https://github.com/s-m-a-r-t-ism/Evelien-NL-Concert-Watch, lees daarna scheduled-task-prompt.md, en update daarmee de prompt van de evelien-concert-watch scheduled task."
