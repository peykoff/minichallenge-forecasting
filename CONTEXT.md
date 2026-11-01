# Mini Challenge 2026 – Contesto del progetto

> ⚠️ **REGOLA: questo file va aggiornato e pushato ogni volta che si lavora sul progetto.**
> Prima di iniziare: `git pull`. Alla fine: aggiorna le sezioni interessate + il **Registro modifiche** in fondo, poi commit e push **insieme** alle modifiche al codice.
> Se il contesto non è aggiornato, gli altri lavorano su informazioni vecchie.

---

## 1. La consegna

- **Corso:** M-B5203E Time Series, Analytics and Forecasting (SUPSI)
- **Scadenza: 5 novembre 2026**
- **Gruppi:** 2-3 persone
- **Cosa consegnare:** un notebook (o codice ampiamente commentato) con una pipeline di analisi e forecasting:
  1. analisi esplorativa (grafici, correlazioni, ACF, …)
  2. costruzione del modello (stazionarietà, trasformazioni, scelta e fit del modello, controllo dei residui)
  3. cross-validation e selezione del modello
  4. presentazione dei risultati
- **Requisiti del prof sul codice:** pulito e organizzato, **efficiente** (se gira per 1 ora è considerato inutilizzabile), commentato, **corto** ("le implementazioni ChatGPT tendono a essere lunghe, voglio correggere il *vostro* codice").
- Testo completo della consegna: `2026 Mini Challenge.pdf` (incompleto) o la [pagina Notion](https://isaac-sene.notion.site/2026-Mini-Challenge-3f7dc1779d9b804ebe0ae8079ddb212e).

## 2. Dataset scelto: Hotels – Los Angeles

**Task:** previsione a 1-7 giorni della colonna `Revenue` per una città, usando i dati 2013-2020.

**Perché Hotels** (confrontato con gli altri 3 dataset proposti):

| Dataset | Frequenza / orizzonte | Problemi | Verdetto |
|---|---|---|---|
| **Hotels** | giornaliero, 1-7 giorni | serie corta, nessun NaN; rottura COVID nel 2020 | ✅ scelto: veloce, struttura chiara |
| Weather | orario, 1-24 ore | ~70k punti, buco di 73 ore, valori -9999 nel vento | alternativa valida ma più lenta |
| ETT (oil) | 15 min, 1-96 passi | ~70k punti, stagionalità 96 → SARIMA impraticabile | scartato |
| AIR | orario, 1-24 ore | 12 stazioni, molti NaN, picchi fino a 999 | scartato |

**Perché Los Angeles:** la città non è scelta a priori: nella sezione 2.1 del notebook si confrontano tutte le 8 città (ACF lag 7/364, variance ratio e spectral entropy del corso, MAPE del seasonal naive 7 sul 2019) e si sceglie automaticamente quella con il MAPE più basso → Los Angeles (ACF lag 7 = 0.83, MAPE 12.1% vs 17.9% della seconda, Dallas). Il resto del notebook dipende solo dalla variabile `CITY`. Nota: su variance ratio e spectral entropy Boston è leggermente migliore di LA (LA seconda), ma ha un errore del seasonal naive molto più alto (21.5%): si sceglie in base al test a-posteriori, come suggerito dal corso.

**Note sui dati:**
- Le colonne sono descritte come "Cumulative" ma in realtà sono **valori giornalieri** (non serve differenziare).
- Dati sintetici, ma il **crollo COVID da marzo 2020 è presente** (-85% circa, recupero parziale fino a ottobre 2020).
- Atlanta parte dal 2012, le altre città dal 2013.

## 3. Struttura della cartella

Repo GitHub: <https://github.com/peykoff/minichallenge-forecasting>

```
Minichallenge-Forecasting/
├── CONTEXT.md                 ← questo file
├── TODO.md                    ← cose da fare fino alla consegna
├── CODING_GUIDELINES.md       ← regole comuni per codice e celle (anche per progetti futuri)
├── .gitignore                 ← esclude .DS_Store, .ipynb_checkpoints/, __pycache__/
├── 2026 Mini Challenge.pdf    ← consegna (stampa parziale della pagina Notion)
├── hotels_LA.ipynb            ← NOTEBOOK PRINCIPALE (pipeline completa)
├── Data/                      ← dataset del prof (pickle): hotels, weather, oil_1
└── Theory/                    ← slide e notebook del corso usati come riferimento
```
`air.pk` (42 MB) non è nel repo perché non serve; se necessario si scarica dal link "data" della sezione AIR della pagina Notion.

**Setup:**
```bash
pip install statsforecast
```
Servono anche: `pandas numpy matplotlib seaborn scipy statsmodels` (già in Anaconda).
Il notebook va eseguito dalla cartella principale del repo (legge `Data/hotels.pk`). **Runtime ≈ 35 secondi.**

## 4. Stato attuale del notebook `hotels_LA.ipynb`

| Sezione | Contenuto | Stato |
|---|---|---|
| 1. Data loading | tutte le 8 città (Revenue), 2013-2020, nessun NaN | ✅ |
| 2. EDA | 2.1 confronto città e scelta di `CITY`; poi time plot, boxplot giorno/mese, correlazioni, ACF/PACF, MSTL (7, 365) | ✅ |
| 3. Trasformazioni | Box-Cox/log, test ADF + KPSS | ✅ |
| 4. CV + baseline | `rolling_cv`, naive, mean, seasonal naive 7 e 364, metriche, residui | ✅ |
| 5. Modelli | ETS, SARIMA, ARIMA + lag364 (statsforecast) | ✅ |
| 6. Risultati | tabella, outlier (Hampel su resto MSTL, 6.3), previsioni vs reali + errori 2019-2020 (6.5), MAPE per orizzonte, diagnostica residui, previsioni con PI, conclusioni | ✅ |

### Scelte metodologiche (da saper giustificare all'esame!)
- **Log del revenue:** la deviazione standard mensile cresce col livello; dopo il log è costante (λ Box-Cox MLE = 0.58, scelto log per semplicità).
- **Stazionarietà:** `y` e `log y` non stazionari; `Δ7 log y` stazionario (ADF p≈0, KPSS p≥0.10) → D=1, d=0. AutoARIMA conferma.
- **COVID:** selezione del modello con CV su **2019**; il **2020 è uno stress test separato** (nessun modello stimato sul pre-COVID può prevedere la rottura).
- **CV rolling-origin:** a ogni giorno del 2019 si prevedono i 7 giorni successivi (~360 origini).
- **Parametri stimati una sola volta** su 2014-2018; in CV il modello viene solo aggiornato con i nuovi dati (`forward`, equivalente a `refit=False` in `statsforecast.cross_validation`). Motivo: runtime. Verificato che sul 2019 le previsioni sono quasi identiche a un refit completo.
- La stima parte dal **2014** perché il regressore `lag364` è NaN nel 2013.
- Back-transform con `exp` → si ottiene la **mediana** della previsione (non la media).

### Risultati attuali

| Modello | MAPE CV 2019 | MAPE stress 2020 |
|---|---|---|
| **ARIMA + lag364** (ARIMA(1,1,2)(2,0,0)[7] con regressore log y₍t−364₎) | **5.9 %** | 27.0 % |
| seasonal naive 364 (miglior baseline) | 6.5 % | 142.8 % |
| ETS(A,N,A) | 9.5 % | **11.8 %** |
| SARIMA(5,0,2)(2,1,0)[7] | 9.6 % | 13.5 % |
| seasonal naive 7 | 11.7 % | 13.7 % |

- **Modello selezionato: ARIMA + lag364** → MAE −11% rispetto alla baseline migliore, ~4% MAPE a 1 giorno, residui senza autocorrelazione (Ljung-Box p = 0.35 a lag 7).
- **Outlier:** filtro di Hampel come nelle slide (finestra 15 giorni, mediana ± 3 std) sul resto MSTL di log y fino al 2018 → 7 giorni (0.3%), quasi tutti Memorial Day e Labor Day, imputati con la mediana locale. Il modello stimato sui dati puliti **peggiora** sul 2019 (MAPE 6.2% vs 5.9%) e migliora poco nel 2020 (26.1% vs 27.0%): sono festività mobili, servirebbero dummy. Si tiene il modello sui dati originali.
- **Punti deboli:** festività mobili (es. Thanksgiving 2019 non allineato con il 2018) e rotture strutturali (nel 2020 ETS è il più robusto).

## 5. TODO / idee aperte

- [ ] **Rivedere e personalizzare** le 21 celle di testo (Observations, spiegazioni, conclusioni): nuova bozza più semplice già nel notebook, va riletta e riformulata con parole nostre. Elenco celle: 1, 7, 9, 13, 16, 19, 22, 27, 30, 31, 38, 41, 42, 44, 46, 50, 53, 54, 57, 62, 63 (priorità: 63 Conclusioni).
- [x] Rieseguire tutto (`Restart & Run All`) su ogni PC per verificare che funzioni
- [ ] (opzionale) Dummy per le festività (Thanksgiving, Natale, 4 luglio, …) → dovrebbe ridurre i picchi nei residui
- [ ] (opzionale) Combinazione ETS + ARIMA + lag364 per robustezza dopo rotture strutturali
- [ ] (opzionale) Provare `Demand` ritardata come regressore esogeno (correlazione con Revenue 0.96, probabilmente poco guadagno)
- [x] Rileggere il notebook intero prima della consegna: corto, commentato, niente codice inutile

## 6. Come lavorare in gruppo

- **`git pull` prima di iniziare**, sempre.
- I notebook `.ipynb` sono JSON: i **conflitti di merge sono molto difficili da risolvere**. Avvisate il gruppo quando modificate il notebook principale, così ci lavora una persona alla volta. Per gli esperimenti usate una copia (es. `hotels_LA_<nome>.ipynb`) e poi riportate nel principale solo ciò che funziona.
- Se aprite il notebook mentre qualcun altro (o uno script) lo modifica, chiudetelo **senza salvare** e riapritelo, altrimenti sovrascrivete le modifiche.
- Alla fine di ogni sessione: **aggiorna questo file** (stato, risultati, TODO, registro) → commit → push.

## 7. Registro modifiche

> Aggiungere una riga **in cima** a ogni sessione di lavoro: data, chi, cosa è stato fatto.

| Data | Chi | Modifiche |
|---|---|---|
| 2026-10-31 | [nome] | Codice reso più leggibile (niente docstring, commenti essenziali, spaziature, loop al posto di comprehension annidate e if/else in linea). Creato `CODING_GUIDELINES.md` con le regole comuni. |
| 2026-10-31 | [nome] | Notebook allineato al materiale del corso: rimosso il modello SARIMA + Fourier e il coefficiente di variazione (non trattati), lag364 giustificato come time embedding / ARIMAX, Hampel con std come nelle slide (7 outlier), testo AIC e MSTL chiarito. Runtime ≈ 35 s. |
| 2026-10-30 | [nome] | Creato `TODO.md` con le attività rimanenti, la preparazione all'esame e gli extra opzionali. |
| 2026-10-30 | [nome] | Riscritte in modo più semplice le 21 celle di testo del notebook (bozza da rivedere), metriche citate con MAE/nMAE prima del MAPE; aggiornata la pipeline nella prima cella. |
| 2026-10-30 | [nome] | Verifica rispetto alle linee guida del prof: aggiunta spiegazione Box-Cox/stazionarietà (3.2), sezione outlier con filtro di Hampel (6.3), grafico previsioni vs reali ed errori 2019-2020 (6.5). Sezioni 6.3/6.4 rinumerate in 6.4/6.6. |
| 2026-10-30 | [nome] | Aggiunti variance ratio e spectral entropy (indicatori di forecastability del corso) alla sezione 2.1 del notebook. |
| 2026-10-30 | [nome] | Notebook ristrutturato: si caricano tutte le città, la scelta della città (LA) avviene nell'EDA (sezione 2.1) in base a indicatori di forecastability, non più a priori. Variabile `la` rinominata `data`, sottosezioni EDA rinumerate. |
| 2026-10-12 | [nome] | Progetto spostato nel repo GitHub. Aggiunto `.gitignore`, rimossi dal tracking i file `.DS_Store`, aggiornati i percorsi in questo file. |
| 2026-10-12 | [nome] | Scelta del dataset (Hotels / Los Angeles) dopo esplorazione dei 4 dataset. Creato `hotels_LA.ipynb` con pipeline completa: EDA, trasformazioni e test di stazionarietà, CV rolling-origin su 2019 + stress test 2020, 4 baseline e 4 modelli statistici, selezione (ARIMA + lag364, MAPE 5.9%), diagnostica residui, previsioni con intervalli. Creato questo `CONTEXT.md`. |
