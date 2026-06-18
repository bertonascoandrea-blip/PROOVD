# PROOVD — Contesto permanente del progetto

## Identità del progetto
PROOVD è una piattaforma web di simulazione del lavoro di un junior digital marketer.
È l'equivalente di eToro paper trading applicato al digital marketing:
l'utente parte con €500 token virtuali e simula campagne reali su tutte le piattaforme
di advertising, studia i dati, fa reportistica, e viene corretto da un AI coach ad ogni passo.

## Stack tecnico — REGOLE ASSOLUTE
- UN SOLO FILE: tutto in index.html (HTML + CSS + JS inline)
- Nessun framework, nessuna dipendenza npm
- Solo Google Fonts (Inter) via CDN
- AI via fetch a https://api.anthropic.com/v1/messages (model: claude-sonnet-4-6, max_tokens: 1000)
- Persistenza SOLO in localStorage
- Nessun backend, nessun server, apre direttamente nel browser

## Design system — NON DEROGARE MAI
- Background:    #0A0A0A
- Surface card:  #111111
- Surface inner: #1A1A1A
- Border:        #2A2A2A
- Blue:          #0055FF
- Lime:          #AAFF00
- Testo:         #F0F0F0
- Muted:         #777777
- Font:          Inter, system-ui, sans-serif
- Border-radius: 16px (card), 100px (pill/button)
- Stile:         dark mode premium, microanimazioni CSS, glassmorphism leggero

## Lingua
Tutto in italiano — UI, testi, messaggi, feedback AI, errori, label, tutto.

## Architettura moduli (sidebar sinistra)
0. Shell globale — sidebar, header, token system, AI coach bubble, router
1. Dashboard — overview portfolio campagne, KPI, chart andamento
2. Campaign Builder — flusso 6 step per costruire e lanciare campagne
3. Live Manager — gestione campagne attive con dati simulati in tempo reale
4. Analytics Lab — calcolatore KPI interattivo, scenario builder, benchmark
5. Reporting Studio — report builder, template, AI writer, client deck
6. Audience Studio — buyer persona, competitor analysis, customer journey
7. Interview Coach — simulazione colloquio AI, flashcard, studio guidato
8. PROOVD Score — profilo competenze, badge, esporta card

## Token economy
- Inizio: €500 token
- Lancia campagna: scala il budget scelto
- ROI > 3x: +20% bonus token
- ROI < 1x: budget perso
- Quiz corretto: +5 token
- Colloquio score 8+: +10 token
- Token a 0: Game Over modal con stats + bottone ricomincia

## Simulazione campagne
setInterval ogni 10 secondi = 1 giorno simulato.
Benchmark CPM/CTR/CVR per piattaforma definiti nell'algoritmo.
Modificatori +/- basati sulle scelte dell'utente.
Varianza giornaliera ±15% random.

## Piattaforme simulate
Meta Ads, Google Search, Google Display, YouTube Ads, TikTok Ads,
LinkedIn Ads, Pinterest Ads, Snapchat Ads, X Ads,
Email Marketing, SEO/SEM Organico, Programmatic/DSP

## Regole di sviluppo — RISPETTA SEMPRE
1. Non rompere mai funzionalità già costruite quando aggiungi nuove
2. Testa localStorage dopo ogni modulo prima di passare al successivo
3. Ogni sezione JS inizia con // === NOME SEZIONE ===
4. Ogni termine tecnico (CTR, ROAS, CPM...) ha tooltip on-hover
5. Ogni chiamata AI ha gestione errore + retry button
6. Skeleton screen mentre AI risponde, mai spinner nudi
7. Empty state con CTA se nessuna campagna presente
8. Toast notifications per eventi importanti (bottom-right)
9. Mobile responsive: sidebar collassa in bottom nav su mobile
10. Tutte le route gestite via router JS senza ricaricare la pagina

## Come riprendere una sessione
Leggi sempre CLAUDE.md + tutto index.html prima di scrivere codice.
Poi chiedi: "Da dove riprendo?" se non ti viene detto esplicitamente.
Non sovrascrivere mai codice funzionante senza prima mostrare il diff.

## Ordine di build
Fase 1: Shell globale (sidebar + header + token system + router + AI coach bubble)
Fase 2: Dashboard (modulo 1)
Fase 3: Campaign Builder (modulo 2) — PRIORITÀ ASSOLUTA
Fase 4: Live Manager (modulo 3)
Fase 5: Analytics Lab (modulo 4)
Fase 6: Reporting Studio (modulo 5)
Fase 7: Audience Studio (modulo 6)
Fase 8: Interview Coach (modulo 7)
Fase 9: PROOVD Score (modulo 8)
