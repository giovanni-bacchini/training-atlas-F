# Guida OpenRocket: massa, baricentro e inerzie del razzo senza motore

In RocketPy il motore è modellato a parte (`simulation_inputs/propulsion_data/`), quindi `geometry.csv` descrive il razzo **senza motore**. Lo script `ork_positions.py` ricava dal `.ork` posizioni e forme, ma non questi quattro numeri, che si prendono da OpenRocket:

| `geometry.csv` | Cosa è | Da dove |
|---|---|---|
| `rocket_dry_mass` | massa del razzo senza motore [kg] | disegno, configurazione senza motore |
| `center_of_mass_without_motor` | baricentro del razzo senza motore, dalla punta dell'ogiva [m] | export della simulazione col motorino, subito dopo il burnout |
| `rocket_dry_inertia_11` | momento d'inerzia attorno a un asse perpendicolare al razzo [kg·m²] | export della simulazione col motorino, subito dopo il burnout |
| `rocket_dry_inertia_33` | momento d'inerzia attorno all'asse del razzo [kg·m²] | export della simulazione col motorino, subito dopo il burnout |

I nomi dei menu sono quelli della versione inglese di OpenRocket (24.12). In italiano i nomi sono tradotti, ma i pulsanti sono negli stessi posti.

## 1. Massa: configurazione senza motore
1. Apri `simulation_inputs/geometry_data/v1.8/rocket.ork` (File → Open).
2. Nella scheda **Rocket design**, in alto c'è il menu a tendina **Flight configuration**. Scegli la configurazione **senza motore** ("[No motors]").
3. In alto a sinistra nel disegno OpenRocket scrive la massa ("Mass with no motors"). Convertila in kg.

La massa va presa senza motore: nella configurazione con il motore c'è dentro anche lui.

## 2. Baricentro e inerzie: simulazione col motorino
OpenRocket calcola le inerzie solo durante una simulazione, e per simulare serve un motore. Nel `.ork` c'è un motore piccolo messo vicino al baricentro (`ork_positions.py` scrive quale: riga "Motore ignorato"). Cambia pochissimo baricentro e inerzie, e serve solo a far partire la simulazione. Subito dopo il burnout il motorino ha finito il propellente: è la configurazione più vicina al razzo senza motore.

Il baricentro **non** va letto dal disegno: lì OpenRocket lo arrotonda al centimetro, l'export invece ha più cifre.

1. Nella scheda **Motors & Configuration** controlla che ci sia una configurazione con il motorino.
2. Nella scheda **Simulations** seleziona la simulazione che usa quella configurazione (oppure creane una con **New simulation**), poi premi **Run simulations**.
3. Seleziona la simulazione e premi **Plot / export**, poi apri la scheda **Export data**.
4. Nella lista delle variabili togli tutto e spunta solo:
   - **Time**;
   - **Altitude**;
   - **Mass**;
   - **Longitudinal moment of inertia**;
   - **Rotational moment of inertia**;
   - **CG location**.
5. Nelle opzioni:
   - separatore dei campi: virgola (`,`);
   - lascia attivi i commenti e gli **eventi di volo** (flight events): nel file compaiono come righe che iniziano con `#`.
6. Premi **Export** e salva il file come `simulation_inputs/geometry_data/v1.8/openrocket_simulationdata.csv`, accanto al `.ork`.

Nel file cerca la riga `# Event BURNOUT occurred at t=...`: i valori vanno letti nella **prima riga subito dopo**.

| Colonna dell'export | Va in | Conversione |
|---|---|---|
| CG location (cm) | `center_of_mass_without_motor` | cm → m |
| Longitudinal moment of inertia (kg·m²) | `rocket_dry_inertia_11` | — |
| Rotational moment of inertia (kg·m²) | `rocket_dry_inertia_33` | — |

## 3. Completare `geometry.csv`
1. Lavora sulla copia in `simulation_inputs/geometry_data/v1.8/geometry.csv`, non sul file nella cartella dello strumento.
2. Metti i quattro valori nella colonna `value` delle righe giuste, al posto di `---`.
3. Controlla le unità: OpenRocket dà la massa in grammi e il CG in centimetri, `geometry.csv` vuole kg e metri.
4. Controlli di buon senso:
   - la massa nell'export subito dopo il burnout deve essere la massa senza motore più la cassa vuota del motorino (circa un etto);
   - i valori devono essere vicini a quelli della v1.6: è lo stesso razzo con qualche modifica. Una massa di 19 000 o un baricentro di 156 sono errori di unità.
