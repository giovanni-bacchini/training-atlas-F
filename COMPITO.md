# Compito: aggiornare le simulazioni di Atlas

Ci sono cinque richieste, arrivano una dopo l'altra. Una geometria nuova, un'aerodinamica nuova, un paracadute diverso, un motore diverso, un altro sito di lancio. Per ogni richiesta devi preparare gli input, aggiornare `atlas_model.py`, lanciare una Monte Carlo e consegnare i risultati.

Gli obiettivi sono quattro:
1. aggiornare le simulazioni passando da `atlas_model.py`, nel modo corretto;
2. saper creare i file di `simulation_inputs/` di geometria e aerodinamica con gli script, e quelli di recovery, motore e sito di lancio modificando i CSV a mano;
3. simulare il volo con una Monte Carlo generando gli output richiesti;
4. consegnare i risultati, **rispettando categoricamente l'ordine della repo**.

Il quarto è fondamentale, un file lasciato in giro o un'intestazione non aggiornata sono considerati degli errori. Le regole sono in [docs/regole_repo.md](docs/regole_repo.md), leggile prima di cominciare.

È concesso chiedere aiuto se hai domande :D

## I sei casi
I casi sono **cumulativi**: ognuno parte dal precedente e cambia una sola cosa.

| Caso | Nome (cartella di output e di consegna) | Cosa cambia | Input da creare |
|---|---|---|---|
| 0 | `0_baseline` | niente, usa la repo così com'è | — |
| 1 | `1_geometria_v1.8` | geometria | `simulation_inputs/geometry_data/v1.8/` |
| 2 | `2_aero_v1.8` | aerodinamica | `simulation_inputs/aerodynamic_data/rocket_body/v1.8_nonozzle/` |
| 3 | `3_recovery_v1.1` | paracadute main | `simulation_inputs/recovery_data/v1.1/` |
| 4 | `4_motore_v1.1` | curva di spinta | `simulation_inputs/propulsion_data/SRAD/v1.1/` |
| 5 | `5_rovigo` | sito di lancio e meteo | `simulation_inputs/environment_data/rovigo/launch_site.csv` |

La versione simulata si aggiorna in testa ad `atlas_model.py`, ovviamente va cambiata caso per caso. Il tipo di meteo (`weather_data`) si sceglie invece in testa a `montecarlo.py`.

| Caso | `GEOMETRY` | `AERODYNAMICS` | `MOTOR` | `RECOVERY` | `LAUNCH_SITE` | Meteo |
|---|---|---|---|---|---|---|
| 0 | v1.6 | v1.4/CD_Mach_Test_Atlas_v_1.2_SDRAD_cd_mach.csv | SRAD/v1.0 | v1.0 | santa_margarida | c |
| 1 | **`v1.8`** | come sopra | SRAD/v1.0 | v1.0 | santa_margarida | c |
| 2 | v1.8 | **`v1.8_nonozzle/CD_no_nozzle_1.8_cd_mach.csv`** | SRAD/v1.0 | v1.0 | santa_margarida | c |
| 3 | v1.8 | come sopra | SRAD/v1.0 | **`v1.1`** | santa_margarida | c |
| 4 | v1.8 | come sopra | **`SRAD/v1.1`** | v1.1 | santa_margarida | c |
| 5 | v1.8 | come sopra | SRAD/v1.1 | v1.1 | **`rovigo`** | **`f`** |

`AIRBRAKES` resta sempre `None`. Per questo compito non sono richiesti voli con gli airbrakes

Ogni Monte Carlo ha **200 simulazioni**, con `save_dispersion_plot = True` e `save_compare_plot = True`. Prima di lanciarle vanno commentati i grafici di confronto che non servono, altrimenti il salvataggio dura una decina di minuti: come si fa è in [docs/guida_simulazioni.md](docs/guida_simulazioni.md), sezione 2.

## I file di partenza: `temp/`
I file nuovi arrivano tutti in `temp/`, come se te li avessero mandati in chat:

| File | Cosa è | Caso |
|---|---|---|
| `temp/rocket.ork` | modello OpenRocket di Atlas v1.8 | 1 |
| `temp/CD_no_nozzle_1.8.CSV` | export aerodinamico di RASAero di Atlas v1.8 | 2 |
| `temp/thrust_curve.csv` | nuova curva di spinta del motore SRAD | 4 |

Ogni file va spostato nel posto che gli spetta, in `temp/` non deve restarne una copia. Alla fine del compito `temp/` deve essere vuota e va **eliminata**.

---

## Caso 0: `0_baseline`
Serve a controllare che tutto funzioni e a prendere confidenza con gli script.

1. Apri `atlas_model.py` e controlla che le versioni in testa siano quelle della riga 0 della tabella.
2. Prepara `montecarlo.py` come spiegato in [docs/guida_simulazioni.md](docs/guida_simulazioni.md) (opzioni e grafici di confronto da commentare). Questa preparazione si fa una volta e vale per tutti i casi.
3. Metti `output_dir_name = '0_baseline'` e fai partire la simulazione [comando: `python montecarlo.py`].
4. Prepara la cartella `consegne/0_baseline/` (guida, sezione 5).

## Caso 1: `1_geometria_v1.8`
C'è un nuovo modello OpenRocket.

1. Crea la cartella `simulation_inputs/geometry_data/v1.8/` e sposta lì `temp/rocket.ork`.
2. Lancia lo script `tools/geometry/ork_positions/ork_positions.py`, passandogli il `geometry.csv` della versione precedente:
   ```bash
   python tools/geometry/ork_positions/ork_positions.py simulation_inputs/geometry_data/v1.8/rocket.ork --geometry simulation_inputs/geometry_data/v1.6/geometry.csv
   ```
   Lo script (spiegato nel suo [README](tools/geometry/ork_positions/README.md)):
   - stampa le posizioni di tutti i componenti e il confronto con la v1.6, con `<!>` dove i valori sono cambiati: guardalo, dice cosa è cambiato nel razzo;
   - scrive `tools/geometry/ork_positions/v1.8/geometry.csv`, con le std prese dalla v1.6 e `---` nei campi che il `.ork` non contiene.
3. Ricava da OpenRocket i 4 valori mancanti (`rocket_dry_mass`, `rocket_dry_inertia_11`, `rocket_dry_inertia_33`, `center_of_mass_without_motor`) seguendo [docs/guida_openrocket.md](docs/guida_openrocket.md). L'export della simulazione va salvato come `simulation_inputs/geometry_data/v1.8/openrocket_simulationdata.csv`.
4. **Copia** `tools/geometry/ork_positions/v1.8/geometry.csv` in `simulation_inputs/geometry_data/v1.8/geometry.csv` e completa la copia: i 4 valori di OpenRocket e `button_angular_position` (0, come nella v1.6). Non deve restare nessun `---`. Il file scritto dallo script resta com'è nella cartella dello strumento.
5. Controlla l'intestazione (le righe con `#`): deve dire `Atlas v1.8`. Lo script la scrive già giusta.
6. In `atlas_model.py` metti `GEOMETRY = "v1.8"`.
7. `output_dir_name = '1_geometria_v1.8'`, lancia la Monte Carlo, prepara `consegne/1_geometria_v1.8/`.

Alla fine in `simulation_inputs/geometry_data/v1.8/` ci sono tre file: `rocket.ork`, `openrocket_simulationdata.csv`, `geometry.csv`.

## Caso 2: `2_aero_v1.8`
Aerodinamica ha mandato l'export di RASAero della v1.8.

L'export di RASAero va in **due** posti: nella cartella dello strumento che lo converte e, insieme al file convertito, negli input. È la stessa logica del `.ork`, che sta accanto al `geometry.csv` che ne deriva.

1. Crea `tools/aerodynamic/cd_mach_rasaero_convert/v1.8_nonozzle/` e sposta lì `temp/CD_no_nozzle_1.8.CSV`.
2. Crea `simulation_inputs/aerodynamic_data/rocket_body/v1.8_nonozzle/`.
3. Converti l'export (lo script è spiegato nel suo [README](tools/aerodynamic/cd_mach_rasaero_convert/README.md)):
   ```bash
   python tools/aerodynamic/cd_mach_rasaero_convert/rasaero_cdmach_generator.py tools/aerodynamic/cd_mach_rasaero_convert/v1.8_nonozzle/CD_no_nozzle_1.8.CSV -o simulation_inputs/aerodynamic_data/rocket_body/v1.8_nonozzle/CD_no_nozzle_1.8_cd_mach.csv
   ```
4. **Copia** l'export anche in `simulation_inputs/aerodynamic_data/rocket_body/v1.8_nonozzle/CD_no_nozzle_1.8.CSV`.
5. Controlla il file convertito: righe `mach,cd` senza intestazione, da Mach 0.01 a 25.
6. In `atlas_model.py` metti `AERODYNAMICS = "v1.8_nonozzle/CD_no_nozzle_1.8_cd_mach.csv"`.
7. `output_dir_name = '2_aero_v1.8'`, lancia, prepara la consegna.

## Caso 3: `3_recovery_v1.1`
Recovery cambia il paracadute main con uno più piccolo.

| Parametro | Valore | std | Nota |
|---|---|---|---|
| `cd_s_main` | 10.0 m² | 0.2 | paracadute più piccolo |

1. Copia la cartella `simulation_inputs/recovery_data/v1.0/` e chiama la copia `v1.1`. La v1.0 resta lì perché una versione già usata non si modifica.
2. In `v1.1/recovery.csv` cambia la riga `cd_s_main` (valore, std e nota). Gli altri parametri restano uguali.
3. **Aggiorna l'intestazione**: la prima riga deve dire `Recupero v1.1` e descrivere cosa c'è dentro (drogue come prima, main con CdS 10 m²).
4. In `atlas_model.py` metti `RECOVERY = "v1.1"`.
5. `output_dir_name = '3_recovery_v1.1'`, lancia, prepara la consegna.

## Caso 4: `4_motore_v1.1`
Propulsione ha mandato una nuova curva di spinta: spinta più bassa del 10 % e combustione più lunga del 10 % rispetto alla v1.0, con la stessa forma.

| Parametro | Valore |
|---|---|
| `impulse` | 10178.22 N·s |
| `burn_time` | 2.9766 s |

1. Copia la cartella `simulation_inputs/propulsion_data/SRAD/v1.0/` e chiama la copia `v1.1`.
2. **Sposta** `temp/thrust_curve.csv` in `SRAD/v1.1/`, al posto della curva copiata dalla v1.0.
3. In `v1.1/motor.csv` aggiorna `impulse` e `burn_time` con i valori della tabella.
4. **Aggiorna l'intestazione** di `motor.csv`: deve dire che è la v1.1 e da dove viene la curva (curva della v1.0 con spinta -10 % e durata +10 %). Le righe che restano vere (hardware, posizioni) si tengono.
5. In `atlas_model.py` metti `MOTOR = "SRAD/v1.1"`.
6. `output_dir_name = '4_motore_v1.1'`, lancia, prepara la consegna.

## Caso 5: `5_rovigo`
Si lancia da Rovigo invece che da Santa Margarida. La cartella `simulation_inputs/environment_data/rovigo/` ha già le immagini della mappa, ma manca `launch_site.csv`.

| Parametro | Valore | std | Nota |
|---|---|---|---|
| `latitude` | 45.078861° | 0 | |
| `longitude` | 12.172131° | 0 | |
| `elevation` | 1.0 m | 0 | il terreno sta tra 0 e 2 m sul livello del mare |
| `rail_length` | 12 m | 0.005 | |
| `inclination` | 84° | 0.5 | rispetto all'orizzontale |
| `heading` | 315° | 1 | rispetto al nord |

1. Copia `simulation_inputs/environment_data/santa_margarida/launch_site.csv` in `simulation_inputs/environment_data/rovigo/launch_site.csv`.
2. Metti i valori della tabella e **aggiorna l'intestazione**: `# Rovigo (ACME) - sito e rampa`.
3. In `atlas_model.py`:
   - `LAUNCH_SITE = "rovigo"`;
   - `FORECAST_MODEL = "icon_d2"`;
   - `LAUNCH_DATE` = un giorno e un'ora **validi per la previsione**: entro 48 ore da quando simuli, ora in UTC (guida, sezione 4).
4. In `montecarlo.py` metti `weather_data = 'f'` perché a Rovigo non c'è il meteo medio `c`.
5. `output_dir_name = '5_rovigo'`, lancia, prepara la consegna.

La previsione scaricata (`simulation_inputs/environment_data/rovigo/rovigo_forecast_icon_d2_<giorno>.json`) è un input della simulazione quindi è parte della consegna.

---

## Cosa consegnare
Si consegna su GitHub, un caso alla settimana, lavorando sulla tua branch: come si fa è in [docs/guida_git.md](docs/guida_git.md). `main` non si tocca.

| Caso | Consegna |
|---|---|
| 0, 1 | commit e push sulla tua branch |
| 2 | commit, push e **pull request** verso `main` (casi 0-2) |
| 3, 4 | commit e push sulla tua branch |
| 5 | commit, push e **pull request** verso `main` (casi 3-5) |

Il caso della settimana è consegnato quando è pushato.

**1. Gli input, nelle loro cartelle**
- [ ] `simulation_inputs/geometry_data/v1.8/`: `rocket.ork`, `openrocket_simulationdata.csv`, `geometry.csv`
- [ ] `tools/geometry/ork_positions/v1.8/geometry.csv` (quello scritto dallo script, con i suoi `---`)
- [ ] `tools/aerodynamic/cd_mach_rasaero_convert/v1.8_nonozzle/CD_no_nozzle_1.8.CSV`
- [ ] `simulation_inputs/aerodynamic_data/rocket_body/v1.8_nonozzle/`: `CD_no_nozzle_1.8.CSV`, `CD_no_nozzle_1.8_cd_mach.csv`
- [ ] `simulation_inputs/recovery_data/v1.1/recovery.csv`
- [ ] `simulation_inputs/propulsion_data/SRAD/v1.1/`: `motor.csv`, `thrust_curve.csv`
- [ ] `simulation_inputs/environment_data/rovigo/`: `launch_site.csv`, `rovigo_forecast_icon_d2_<giorno>.json`

**2. Gli output, in `simulation_output/montecarlo_output/`**
- [ ] le 6 cartelle complete, con i nomi della tabella: `0_baseline`, `1_geometria_v1.8`, `2_aero_v1.8`, `3_recovery_v1.1`, `4_motore_v1.1`, `5_rovigo`

**3. Le consegne, in `consegne/`**
- [ ] 6 sottocartelle con gli stessi nomi, ognuna con solo questi 4 file:
  - `results.txt`, con l'intestazione e solo le grandezze richieste (guida, sezione 5);
  - la mappa degli atterraggi (`Santa_Margarida_launch_site.svg` oppure `Rovigo_launch_site.svg`);
  - `accelerations.svg`;
  - `aerodynamic_forces.svg`.

**4. L'ordine**
- [ ] `temp/` eliminata
- [ ] nessun file fuori posto, nessuna cartella di prova, nessuna copia con nomi tipo `geometry (1).csv` o `motor_old.csv`
- [ ] le versioni di partenza (`v1.6`, `v1.0`, `santa_margarida`...) identiche a come le hai trovate
- [ ] `atlas_model.py` e `montecarlo.py` lasciati nello stato del caso 5
