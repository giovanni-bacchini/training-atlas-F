# Regole di ordine della repo

La repo è usata da più persone e le simulazioni restano come riferimento per mesi. Chi apre una cartella deve capire subito cosa contiene, da dove vengono i numeri e con quali input è stato ottenuto un risultato. Per questo l'ordine non è un dettaglio ma è parte del lavoro.

## 1. Ogni file ha un posto
| Cosa | Dove |
|---|---|
| modello e simulazioni (`atlas_model.py`, `montecarlo.py`, `reanalysis.py`) | radice |
| dati di ingresso | `simulation_inputs/<tipo>/<versione>/` |
| risultati delle Monte Carlo | `simulation_output/montecarlo_output/<output_dir_name>/` |
| strumenti, con i file che leggono e scrivono | `tools/<argomento>/<strumento>/<versione>/` |
| documentazione | `docs/` e i README accanto agli strumenti |

Dentro `simulation_inputs/`:

| Tipo | Cartella | File |
|---|---|---|
| geometria | `geometry_data/<versione>/` | `geometry.csv`, `rocket.ork`, `openrocket_simulationdata.csv` |
| aerodinamica | `aerodynamic_data/rocket_body/<versione>/` | il file `mach,cd` e l'export di RASAero da cui viene |
| motore | `propulsion_data/SRAD/<versione>/` o `COTS/<nome>/` | `motor.csv`, `thrust_curve.csv` |
| recupero | `recovery_data/<versione>/` | `recovery.csv` |
| sito | `environment_data/<sito>/` | `launch_site.csv`, file meteo, immagine della mappa |

**File sorgente.** Il file da cui un input è stato ricavato sta accanto all'input: il `.ork` accanto a `geometry.csv`, l'export di RASAero accanto al file `mach,cd`.

**File degli strumenti.** Quello che uno strumento legge o scrive sta nella sua cartella, per versione: il `geometry.csv` scritto da `ork_positions.py` in `tools/geometry/ork_positions/<versione>/`, l'export di RASAero da convertire in `tools/aerodynamic/cd_mach_rasaero_convert/<versione>/`.

## 2. Una versione non si modifica: se ne crea una nuova
Una versione già usata in una simulazione resta com'è, altrimenti i risultati vecchi non corrispondono più ai loro input. Per cambiare un valore:
1. si copia la cartella della versione e si rinomina la copia (`v1.0` → `v1.1`);
2. si modifica la copia;
3. si aggiorna l'intestazione della copia;
4. si sceglie la nuova versione in testa ad `atlas_model.py`.

## 3. L'intestazione dice cosa c'è nel file
Le prime righe di ogni CSV di parametri iniziano con `#` e dicono versione e fonte dei dati. Quando si crea una versione nuova l'intestazione va riscritta.

Esempio, `recovery.csv`:
```
# Recupero v1.0 - drogue Rocketman 3 ft, main Rocketman 14 ft (Cd 0,97, senza spill hole)
```

Lo stesso vale per la colonna `note`, se cambi un valore, la nota deve dire da dove viene il nuovo.

## 4. Formato dei CSV di parametri
```
name,value,std,unit,note
fin_span,0.195,0.0005,m,
```
- `std` è la deviazione standard usata dalla Monte Carlo; 0 = valore fisso.
- I nomi nella colonna `name` sono quelli che legge `atlas_model.py`: non si cambiano.
- **Niente virgole nelle note** e nelle intestazioni dopo la riga `name,...`: la virgola separa le colonne e il file non si legge più (errore `Expected 5 fields`). Si usano punto e virgola o barre.
- I decimali si scrivono con il punto.
- `---` vuol dire "da completare": il modello si ferma e dice quali campi mancano.

## 5. Le versioni si scelgono solo in `atlas_model.py`
Per cambiare input non si toccano i percorsi nel codice e non si sovrascrivono i file ma si cambia la versione in testa ad `atlas_model.py` (`GEOMETRY`, `AERODYNAMICS`, `MOTOR`, `RECOVERY`, `LAUNCH_SITE`). `montecarlo.py` stampa le versioni usate e le scrive in `results.txt`, così si possono ricontrollare.

## 6. Gli output
- Ogni Monte Carlo da conservare ha un `output_dir_name` suo, che dice cosa contiene.
- `prova` è la cartella per i tentativi: non va su git e si sovrascrive.
- Una cartella di output non si modifica a mano, se serve una versione ridotta dei risultati, si lavora su una copia.
- Se una simulazione è sbagliata si rilancia con lo stesso nome e si sovrascrive.

## 7. Git
- Non si lavora su `main`: si lavora sulla propria branch e su `main` si arriva con una pull request ([guida_git.md](guida_git.md)).
- Nessun file generato da strumenti che si possono rilanciare (`.pickle`, `__pycache__/`): ci pensa `.gitignore`.

## 8. Niente file in giro
- Nessun file nella radice oltre a quelli che ci sono già.
- Nessuna copia di sicurezza (`geometry_old.csv`, `motor (1).csv`, `copia di ...`) perché la storia dei file la tiene git.
- Le cartelle temporanee si eliminano quando sono vuote.
- Prima di ogni commit lancia `git status`: elenca i file nuovi, modificati o eliminati rispetto all'ultimo commit. Ogni file dell'elenco deve essere voluto e al suo posto.
