# RAG viz

**Interroga i tuoi documenti e guarda dove si trova la risposta.**

RAG viz è un'app a singolo file HTML che costruisce un indice di retrieval (la parte "R" di un sistema RAG) direttamente nel browser e ne mostra lo **spazio semantico in 2D**. Mentre scrivi nella barra di ricerca, la mappa si aggiorna in tempo reale: vedi quali frammenti, parole e documenti sono più vicini alla tua domanda.

> Nessun server, nessuna API key, nessun upload: i file restano sul tuo dispositivo.

---

## Funzionalità

- **Più formati insieme:** `.pdf`, `.json`, `.txt`, `.md`, `.csv`, anche in blocco, con trascinamento o selezione file.
- **Mappa spaziale in tempo reale**
  - i **punti** sono i frammenti di testo, colorati per documento;
  - gli **anelli** con il nome sono i documenti, posizionati sul centroide dei loro frammenti;
  - le **etichette** sono le parole, collocate dove compaiono di più; quelle più vicine alla query si evidenziano;
  - un marcatore pulsante rappresenta la query, collegato con linee ai frammenti più simili (lo spessore dipende dalla similarità).
- **Pannello risultati:** frammenti ordinati per similarità, con barra di punteggio e termini evidenziati. Passando il mouse su una card si illumina il punto corrispondente sulla mappa e viceversa. Un clic su un punto lo fissa in cima alla lista.
- **Sezioni attivabili:** raggruppa i documenti in sezioni, ciascuna con il proprio **interruttore on/off**. Una sezione spenta esce dalla mappa e dalla ricerca.
- **Persistenza in `.json`:** salva e riapri l'intero stato (sezioni, switch, testi, ultima query). In più c'è un salvataggio automatico nel browser.
- **Copia contesto:** compone un prompt "contesto + domanda" con i frammenti recuperati, pronto da incollare in un LLM.
- **Navigazione:** zoom con la rotella, spostamento trascinando, tooltip al passaggio sui punti. Tema chiaro e scuro automatico.

---

## Avvio rapido

1. Scarica `rag-viz.html`.
2. Aprilo in un browser moderno (Chrome, Edge, Firefox, Safari 16.4+).
3. Trascina i tuoi file, oppure premi **Esempio** per caricare tre documenti dimostrativi divisi in due sezioni.
4. Scrivi una domanda nella barra di ricerca.

Per pubblicarlo come pagina web, ad esempio con **GitHub Pages**, basta rinominarlo `index.html` e abilitare Pages sul repository.

> La pagina carica `pdf.js` da cdnjs e il font IBM Plex Sans da Google Fonts. Serve quindi una connessione internet per leggere i PDF; il font ha un fallback di sistema.

---

## Come funziona

```
file → testo → frammenti (~35 parole) → vettori TF‑IDF → PCA 2D → mappa
                                                  ↑
                                  query → stesso spazio → coseno → top‑8
```

1. **Parsing**
   - `.txt` e `.md` sono letti come testo.
   - `.pdf` è estratto pagina per pagina con pdf.js.
   - `.json` viene appiattito in righe `chiave: valore`.
   - `.csv` viene trasformato in righe `colonna: valore` (separatore `,` o `;` rilevato automaticamente).
2. **Chunking:** il testo è diviso in frasi e raggruppato in frammenti di circa 35 parole.
3. **Vettorizzazione:** ogni frammento diventa un vettore da 1024 dimensioni con ponderazione TF‑IDF su parole e **trigrammi di caratteri** (hashing). Questo rende la ricerca tollerante a refusi e forme flesse. Le stopword italiane e inglesi sono rimosse.
4. **Proiezione 2D:** una PCA con power iteration porta i vettori su due assi.
5. **Retrieval:** la query è vettorizzata nello stesso spazio e confrontata con tutti i frammenti tramite **similarità del coseno**. Vengono mostrati i primi 8 con punteggio sopra soglia.
6. **Posizione della query:** il marcatore si colloca sul baricentro dei frammenti migliori, pesato per similarità al quadrato.

---

## Formato del file di salvataggio

```json
{
  "app": "rag-viz",
  "v": 1,
  "saved": "2026-11-23T10:00:00.000Z",
  "tsel": "s1",
  "secs": [
    { "id": "s1", "name": "Cucina", "on": true },
    { "id": "s2", "name": "Tecnologia", "on": false }
  ],
  "docs": [
    { "name": "cucina-siciliana.txt", "sec": "s1", "parts": ["frammento 1…", "frammento 2…"] }
  ],
  "query": "melanzane"
}
```

Non contiene vettori né coordinate: l'indice e la mappa vengono ricostruiti all'apertura. Trascinare un file di questo tipo nella pagina lo carica come salvataggio e non come documento.

---

## Limiti noti

- **Gli embedding sono lessicali, non neurali.** La vicinanza si basa su parole e sottostringhe condivise, non sul significato profondo: "automobile" e "macchina" non risultano vicine se non condividono termini.
- **Solo retrieval:** la generazione della risposta è a carico dell'utente tramite "Copia contesto".
- **PDF scansionati:** senza OCR non viene estratto alcun testo.
- **Scala:** la PCA gira nel browser; con migliaia di frammenti il ricalcolo (anche al cambio di una sezione) rallenta.
- **Mappa 2D:** è una proiezione, quindi due punti vicini nello spazio reale possono apparire distanti e viceversa. Spegnere una sezione ricalcola gli assi e sposta i punti.
- **Salvataggio automatico:** usa `localStorage` (circa 5 MB); con corpus grandi può non bastare, quindi conviene esportare il `.json`.

---

## Roadmap

- [ ] Embedding neurali locali (Transformers.js / ONNX) come alternativa a TF‑IDF
- [ ] Layout alternativi (UMAP, t‑SNE)
- [ ] Colore per sezione e filtri per documento
- [ ] Chiamata diretta a un LLM per generare la risposta
- [ ] OCR per PDF scansionati
- [ ] Esportazione della mappa in PNG/SVG

---

## Stack

HTML, CSS e JavaScript vanilla in un solo file, canvas 2D per il rendering e [pdf.js](https://mozilla.github.io/pdf.js/) per i PDF.

## Licenza

Scegli la licenza che preferisci, ad esempio MIT, e aggiungi il file `LICENSE` al repository.
