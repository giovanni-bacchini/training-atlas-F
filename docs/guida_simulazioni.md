# Guida alle simulazioni del compito

La documentazione completa degli script è in [atlas_model.md](atlas_model.md) e [montecarlo.md](montecarlo.md). Qui c'è quello che serve per il compito.

## 1. Come funziona
- `atlas_model.py` definisce il razzo. In testa si scelgono le **versioni** degli input; il modello legge i CSV di quelle versioni in `simulation_inputs/`.
- `montecarlo.py` fa molti voli (qui 200). In ognuno estrae ogni parametro da una distribuzione normale: media = colonna `value` del CSV, deviazione standard = colonna `std`. Dai 200 voli escono media e deviazione standard di ogni risultato e la zona di atterraggio.

Per ogni caso del compito si fanno sempre le stesse tre cose:
1. creare gli input nuovi (se il caso lo chiede);
2. scegliere le versioni in testa ad `atlas_model.py`;
3. cambiare `output_dir_name` in `montecarlo.py` e lanciare.

## 2. Preparare `montecarlo.py`

### Le opzioni
In testa a `montecarlo.py`, nel blocco `PARAMETERS`:

```python
output_dir_name = '0_baseline'   # cambia a ogni caso
number_of_simulations = 200
weather_data = 'c'               # 'f' solo nel caso 5
seed = None
multiple_core = True

ballistic = False
show_dispersion_plot = False
save_dispersion_plot = True
show_compare_plot = False
save_compare_plot = True
sensitivity_analysis = False
```

| Opzione | Cosa fa |
|---|---|
| `output_dir_name` | nome della cartella in `simulation_output/montecarlo_output/`. Se esiste già chiede se sovrascriverla |
| `number_of_simulations` | numero di voli |
| `weather_data` | meteo: `c` media della settimana di EuRoC (solo Santa Margarida), `f` previsione |
| `seed` | `None` = voli diversi a ogni lancio. Lascia `None` |
| `show_dispersion_plot` | apre a schermo istogrammi e mappa: il programma si ferma a ogni finestra finché non la chiudi. Lascia `False` |
| `save_dispersion_plot` | salva gli istogrammi in `dispersion/`. Serve `True` |
| `show_compare_plot` | apre a schermo i grafici di tutti i voli sovrapposti. Lascia `False` |
| `save_compare_plot` | salva i grafici di tutti i voli in `comparison/`. Serve `True` |

Con `save_compare_plot = True` i voli girano uno alla volta invece che in parallelo, perché i grafici hanno bisogno di tutti i voli nello stesso processo. 200 voli richiedono qualche minuto.

### Commentare i grafici di confronto che non servono
`montecarlo.py` salva dieci grafici di confronto. Ognuno contiene 200 voli e salvarli tutti richiede molti minuti per caso. Al compito ne servono due: **accelerazioni** e **forze aerodinamiche**. Gli altri vanno commentati.

Cerca in `montecarlo.py` il blocco `if save_compare_plot:` (verso la fine del file, dentro `main()`). Metti un `#` davanti alle righe dei grafici che non servono:

```python
        if save_compare_plot:
            save_compare_plots(
                comparison_object=comparison,
                plots=[
                    #("velocities", {"legend": False}),
                    ("accelerations", {"legend": False}),
                    #("attitude_angles", {"legend": False}),
                    #("euler_angles", {"legend": False}),
                    ("aerodynamic_forces", {"legend": False}),
                    #("aerodynamic_moments", {"legend": False}),
                    #("angular_velocities", {"legend": False}),
                    #("trajectories_3d", {}),
                    #("rail_buttons_forces", {"legend": False}),
                    #("stability_margin", {"legend": False}),
                ],
                output_dir=output_comparison,
                formats=["svg","pickle"],
            )
```

Questa modifica resta per tutti i casi.

## 3. Lanciare un caso
1. Controlla le versioni in testa ad `atlas_model.py` con la tabella di [COMPITO.md](../COMPITO.md).
2. Cambia `output_dir_name` in `montecarlo.py`.
3. Lancia dalla cartella della repo:
   ```bash
   python montecarlo.py
   ```
4. **Leggi le prime righe che stampa**: sito, geometria, aerodinamica, motore, recupero. Devono essere quelle del caso. Se una è sbagliata, ferma con `Ctrl+C`, correggi e rilancia.
5. Se la cartella esiste già, il programma chiede se sovrascriverla: `y` per sì.

Errori frequenti:

| Messaggio | Causa |
|---|---|
| `fields still to fill (---)` | in un CSV è rimasto un `---`: il messaggio dice quali campi |
| `Expected 5 fields` | c'è una virgola in una nota o manca una colonna |
| `No such file or directory` su un file di `simulation_inputs` | versione scritta male in `atlas_model.py`, oppure file con un altro nome o in un'altra cartella |
| `Input directory for LAUNCH_SITE ... not found` | nome del sito diverso da quello della cartella |

### Cosa esce
In `simulation_output/montecarlo_output/<output_dir_name>/`:

| File | Contenuto |
|---|---|
| `results.txt` | numero di voli, sito, versioni usate, media e deviazione standard di ogni grandezza |
| `Atlas.disp_inputs.json` | i parametri estratti per ogni volo, una riga per volo |
| `Atlas.disp_outputs.json` | i risultati di ogni volo, sulla stessa riga |
| `Atlas.disp_errors.txt` | i voli falliti (di solito vuoto) |
| `dispersion/svg/`, `dispersion/pickle/` | un istogramma per grandezza |
| `launch_site/<Sito>_launch_site.svg` | mappa con apogei, atterraggi ed ellissi a 1σ, 2σ, 3σ |
| `comparison/svg/`, `comparison/pickle/` | i grafici di tutti i voli: `accelerations` e `aerodynamic_forces` |

I `.pickle` sono le figure salvate per riaprirle: non vanno su git (ci pensa `.gitignore`), il resto della cartella sì.

## 4. Il caso 5: Rovigo con la previsione
A Rovigo non esiste il file del meteo medio, quindi `weather_data = 'c'` non funziona: si usa la previsione `f`, scaricata gratis da [Open-Meteo](https://open-meteo.com).

In `atlas_model.py`:
```python
LAUNCH_SITE = "rovigo"
LAUNCH_DATE = (2026, 11, 20, 13)      # esempio: (anno, mese, giorno, ora UTC)
FORECAST_MODEL = "icon_d2"
```
In `montecarlo.py`: `weather_data = 'f'`.

**Scegliere una data valida**
- ICON-D2 è il modello meteo tedesco a 2 km di risoluzione, copre l'Europa centrale e dà le previsioni fino a **48 ore** avanti.
- Se la data è troppo lontana o passata da troppo, il modello si ferma con un errore (`no wind in the forecast ... too far ahead` oppure `Open-Meteo refused the forecast`).
- L'ora è richiesta in **UTC**.

**Il file della previsione**
- Al primo lancio lo script scarica la previsione del giorno in `simulation_inputs/environment_data/rovigo/rovigo_forecast_icon_d2_<AAAAMMGG>.json` e poi riusa quel file.
- Se il file di quel giorno esiste già (per esempio perché avevi fatto un tentativo ieri), lo script **non** riscarica le previsioni ma usa quelle vecchie. Per averne una più recente cancella il file prima di lanciare.
- Dentro il file, `downloaded` dice quando è stato scaricato. A terminale lo script scrive la stessa cosa e il vento a 10 m.
- Serve internet solo al momento del download.
- Deve restare **un solo** `.json` di previsione: quello usato per il caso 5. Se hai fatto tentativi con altri giorni, cancella gli altri.

## 5. Preparare la cartella di consegna
Per ogni caso, in `consegne/<nome del caso>/` vanno 4 file. La cartella in `simulation_output/` non si cancella.

1. Copia la cartella `simulation_output/montecarlo_output/<nome>/` dentro `consegne/` (stesso nome).
2. Nella copia, porta questi tre grafici nella radice della cartella:
   - `launch_site/<Sito>_launch_site.svg`;
   - `comparison/svg/accelerations.svg`;
   - `comparison/svg/aerodynamic_forces.svg`.
3. Cancella dalla copia tutto il resto: i `.json`, `Atlas.disp_errors.txt`, le cartelle `dispersion/`, `launch_site/`, `comparison/`, `sensitivity/`.
4. Apri `results.txt` della copia con l'editor e cancella i blocchi delle grandezze non richieste. Restano:
   - l'**intestazione** (le prime righe con numero di simulazioni, cartella, sito e versioni), intatta;
   - solo queste grandezze, ognuna con media e deviazione standard:

| Grandezza | Unità |
|---|---|
| Out of Rail Time | s |
| Out of Rail Velocity | m/s |
| Apogee Time | s |
| Apogee Altitude | m |
| Impact Velocity | m/s |
| Initial Static Margin | c |
| Out of Rail Static Margin | c |
| Final Static Margin | c |
| Maximum Velocity | m/s |
| Maximum Acceleration | m/s² |
| Drogue Parachute Trigger Time | s |
| Drogue Parachute Fully Inflated Time | s |
| Drogue Parachute Fully Inflated Velocity | m/s |

I numeri non si toccano. Si cancellano solo i blocchi in più.

Alla fine `consegne/<nome>/` contiene esattamente:
```
consegne/1_geometria_v1.8/
├── results.txt
├── Santa_Margarida_launch_site.svg
├── accelerations.svg
└── aerodynamic_forces.svg
```
(nel caso 5 la mappa si chiama `Rovigo_launch_site.svg`).

## 6. Controlli prima di passare al caso successivo
- L'intestazione di `results.txt` riporta le versioni del caso.
- Rispetto al caso precedente è cambiato quello che ti aspetti? Per esempio, un main più piccolo deve dare una velocità d'impatto più alta e lasciare uguale l'apogeo. Se cambia qualcosa che non dovrebbe, o non cambia niente, c'è un errore negli input o nelle versioni.
- Puoi fare un controllo con `git status`: elenca i file nuovi, modificati o eliminati rispetto all'ultimo commit. Ogni file dell'elenco deve essere voluto e al suo posto.
