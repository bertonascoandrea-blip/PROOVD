# PROOVD Admin — Setup Supabase

Istruzioni per configurare le tabelle Supabase necessarie al funzionamento del pannello admin.

---

## 1. Esegui le query SQL su Supabase

Vai su **Supabase → SQL Editor** ed esegui i seguenti script nell'ordine indicato.

### Tabella `settings`

Usata per salvare la API key Groq e altre configurazioni globali.

```sql
-- Tabella settings
CREATE TABLE IF NOT EXISTS settings (
  key TEXT PRIMARY KEY,
  value TEXT NOT NULL,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE settings ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Lettura pubblica settings"
  ON settings FOR SELECT
  USING (true);

CREATE POLICY "Scrittura admin settings"
  ON settings FOR ALL
  USING (true);
```

### Tabella `platform_config`

Usata per salvare i benchmark CPM/CTR/CVR di ogni piattaforma pubblicitaria.

```sql
-- Tabella platform_config
CREATE TABLE IF NOT EXISTS platform_config (
  id SERIAL PRIMARY KEY,
  platform_name TEXT NOT NULL UNIQUE,
  cpm DECIMAL(10,2) DEFAULT 0,
  ctr DECIMAL(5,3) DEFAULT 0,
  cvr DECIMAL(5,3) DEFAULT 0,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE platform_config ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Lettura pubblica platform_config"
  ON platform_config FOR SELECT
  USING (true);

CREATE POLICY "Scrittura admin platform_config"
  ON platform_config FOR ALL
  USING (true);
```

### Policy aggiuntiva per la tabella `profiles`

Permette all'admin panel di leggere tutti i profili utente.

```sql
-- Consente la lettura pubblica dei profili (necessario per dashboard admin)
CREATE POLICY "Admin può leggere tutti i profili"
  ON profiles FOR SELECT
  USING (true);
```

> **Nota:** Se esiste già una policy di SELECT su `profiles`, potrebbe generare un conflitto. In quel caso rinomina la policy o adatta la condizione `USING`.

---

## 2. Variabili Supabase

| Variabile       | Valore                                                             |
|----------------|--------------------------------------------------------------------|
| `SUPABASE_URL` | `https://mzbguicvbeitlukhxbqk.supabase.co`                       |
| `SUPABASE_KEY` | `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIs...` |

---

## 3. Cosa aggiungere a `index.html`

Aggiungi le seguenti funzioni **prima** della funzione di inizializzazione principale dell'app (`initApp()` o equivalente). Questo garantisce che la configurazione venga caricata da Supabase prima che l'app si avvii.

### Pseudocodice / implementazione di riferimento

```javascript
// === CONFIGURAZIONE REMOTA DA SUPABASE ===
const SUPABASE_URL = 'https://mzbguicvbeitlukhxbqk.supabase.co';
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'; // chiave completa

/**
 * loadGroqKey()
 * Recupera la API key Groq dalla tabella settings di Supabase.
 * Se trovata, la salva in window.GROQ_API_KEY per l'uso nel coach AI.
 * In caso di errore, l'app continua senza key (degraded mode).
 */
async function loadGroqKey() {
  try {
    const res = await fetch(
      SUPABASE_URL + '/rest/v1/settings?select=value&key=eq.groq_api_key',
      {
        headers: {
          'apikey': SUPABASE_KEY,
          'Authorization': `Bearer ${SUPABASE_KEY}`
        }
      }
    );
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const data = await res.json();
    if (data[0]?.value) {
      window.GROQ_API_KEY = data[0].value;
      console.log('[PROOVD] Groq API key caricata da Supabase');
    }
  } catch (e) {
    console.warn('[PROOVD] Impossibile caricare Groq key:', e.message);
    // Fallback: usa chiave locale se disponibile
    // window.GROQ_API_KEY = window.GROQ_API_KEY_FALLBACK;
  }
}

/**
 * loadPlatformConfig()
 * Recupera i benchmark CPM/CTR/CVR per ogni piattaforma da Supabase.
 * Se trovati, sovrascrive i valori di default in window.PLATFORM_CONFIG.
 * In caso di errore, l'app usa i valori hardcoded di default.
 */
async function loadPlatformConfig() {
  try {
    const res = await fetch(
      SUPABASE_URL + '/rest/v1/platform_config?select=*',
      {
        headers: {
          'apikey': SUPABASE_KEY,
          'Authorization': `Bearer ${SUPABASE_KEY}`
        }
      }
    );
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const platforms = await res.json();
    if (Array.isArray(platforms) && platforms.length > 0) {
      // Converti array in mappa per lookup rapido: { 'Meta Ads': { cpm, ctr, cvr }, ... }
      window.PLATFORM_CONFIG = {};
      platforms.forEach(p => {
        window.PLATFORM_CONFIG[p.platform_name] = {
          cpm: parseFloat(p.cpm),
          ctr: parseFloat(p.ctr),
          cvr: parseFloat(p.cvr)
        };
      });
      console.log('[PROOVD] Config piattaforme caricata da Supabase:', Object.keys(window.PLATFORM_CONFIG));
    }
  } catch (e) {
    console.warn('[PROOVD] Impossibile caricare config piattaforme:', e.message);
    // L'app userà i valori DEFAULT_PLATFORMS definiti localmente
  }
}

/**
 * Punto di ingresso dell'app — carica config remota prima di init
 * Sostituisci initApp() con il nome della tua funzione di avvio.
 */
Promise.all([loadGroqKey(), loadPlatformConfig()])
  .then(() => {
    console.log('[PROOVD] Config remota caricata, avvio app...');
    initApp(); // <- chiama qui la tua funzione di inizializzazione
  })
  .catch(e => {
    console.error('[PROOVD] Errore init:', e);
    initApp(); // avvia comunque l'app con valori default
  });
```

### Utilizzo di `window.GROQ_API_KEY` nel coach AI

Ovunque nel codice usi la chiave API per chiamare Groq o Anthropic, usa la variabile globale:

```javascript
// Prima (chiave hardcoded — NON fare così)
const API_KEY = 'gsk_...';

// Dopo (chiave caricata da Supabase, con fallback)
const API_KEY = window.GROQ_API_KEY || 'chiave-fallback-locale';

// Esempio di chiamata al coach AI
async function askCoach(prompt) {
  const key = window.GROQ_API_KEY;
  if (!key) {
    showToast('AI Coach non disponibile: API key mancante', 'error');
    return null;
  }
  const res = await fetch('https://api.groq.com/openai/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${key}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      model: 'llama3-70b-8192',
      messages: [{ role: 'user', content: prompt }],
      max_tokens: 500
    })
  });
  const data = await res.json();
  return data.choices?.[0]?.message?.content || null;
}
```

### Utilizzo di `window.PLATFORM_CONFIG` nel simulatore

```javascript
// Nel simulatore campagne, usa i valori remoti se disponibili
function getPlatformBenchmarks(platformName) {
  const remote = window.PLATFORM_CONFIG?.[platformName];
  if (remote) return remote;

  // Fallback ai valori hardcoded
  const defaults = {
    'Meta Ads':       { cpm: 8.50,  ctr: 1.2, cvr: 2.1 },
    'Google Search':  { cpm: 15.00, ctr: 3.5, cvr: 4.2 },
    'YouTube Ads':    { cpm: 6.00,  ctr: 0.8, cvr: 1.5 },
    // ... etc
  };
  return defaults[platformName] || { cpm: 5.00, ctr: 1.0, cvr: 2.0 };
}
```

---

## 4. Struttura tabelle — Riepilogo

| Tabella          | Colonne principali                          | Scopo                              |
|-----------------|---------------------------------------------|-------------------------------------|
| `settings`      | `key (PK)`, `value`, `updated_at`           | Configurazioni globali (API keys)   |
| `platform_config` | `platform_name (UNIQUE)`, `cpm`, `ctr`, `cvr` | Benchmark piattaforme pubblicitarie |
| `profiles`      | `id`, `email`, `game_state (JSONB)`, `created_at` | Dati utente e stato di gioco     |

---

## 5. Accesso al pannello admin

1. Apri `admin.html` direttamente nel browser (nessun server richiesto)
2. Password: `proovd-admin-2024`
3. La sessione persiste finché il tab del browser rimane aperto (sessionStorage)
4. Per sicurezza, cambia la password nel file `admin.html` prima del deploy

---

*Generato per PROOVD Admin v1.0.0*
