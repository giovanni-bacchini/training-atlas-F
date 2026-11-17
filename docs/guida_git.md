# Guida git: branch, push e pull request

## 1. Cos'è una branch e perché serve
Una branch è una linea di lavoro separata dentro la stessa repo. `main` è la versione di riferimento: quella che tutti usano e che deve sempre funzionare. Chi lavora non scrive direttamente su `main`, ma su una sua branch: lì può fare tentativi ed errori senza rompere niente agli altri.

Quando il lavoro è finito e controllato, si chiede di portarlo su `main` con una **pull request** (PR): qualcuno lo rivede, commenta, e se va bene lo unisce (*merge*).

Nella repo del team funziona così, e qui fai lo stesso:
- `main` contiene il kit di partenza e **non si tocca**: non ci fai commit e non ci pushi;
- lavori sulla tua branch, che si chiama come te (nome in minuscolo, per esempio `mario`);
- su `main` il lavoro arriva solo tramite pull request, e il merge lo faccio io.

## 2. Prima volta: clonare e creare la branch
```bash
git clone <indirizzo della repo>
cd <nome della repo>
git switch -c mario
git push -u origin mario
```
- `git switch -c mario` crea la branch `mario` partendo da dove sei (`main`) e ti ci sposta.
- `git push -u origin mario` la crea anche su GitHub. Con `-u` git si ricorda il collegamento: dalla volta dopo basta `git push`.

Per sapere su che branch sei:
```bash
git branch
```
La branch attuale ha l'asterisco. Prima di ogni commit controlla di essere sulla tua e non su `main`.

Per spostarti su una branch che esiste già: `git switch mario`.

## 3. Lavorare: commit e push
Un **commit** salva sul tuo computer una fotografia dei file che gli indichi, con un messaggio. Il **push** manda i commit su GitHub.

```bash
git status                      # cosa è cambiato dall'ultimo commit
git add <file o cartella>       # cosa entra nel prossimo commit
git commit -m "caso 1: geometria v1.8"
git push
```
- Con `git status` controlla che ogni file nell'elenco sia voluto e al suo posto, prima di `git add`.
- Si può fare `git add .` (tutto), ma solo dopo aver letto `git status`.
- Messaggi brevi che dicono cosa hai fatto. Più commit per caso vanno bene: per esempio uno per gli input e uno per gli output.
- Finché non fai push, il lavoro è solo sul tuo computer: io non lo vedo.

Se hai fatto un commit su `main` per sbaglio, **non pushare** e chiedi aiuto.

## 4. Pull request
Una PR chiede di unire la tua branch in `main`. Contiene tutti i commit della branch che `main` non ha ancora.

**Quando farla.** Una PR non si apre a ogni commit: si apre quando un blocco di lavoro è **finito, controllato e pronto per essere rivisto**. Una PR troppo piccola fa perdere tempo a chi la rivede; una troppo grande è difficile da rivedere. In questo compito i momenti sono fissati (vedi la tabella in [COMPITO.md](../COMPITO.md)): alla fine del caso 2 e alla fine del caso 5.

**Come aprirla.**
1. Pusha tutto (`git status` deve dire che non c'è niente da committare).
2. Su GitHub, nella pagina della repo, compare il pulsante **Compare & pull request** sulla tua branch. Altrimenti: scheda **Pull requests** → **New pull request**.
3. Controlla la direzione: **base: `main`** ← **compare: `mario`**.
4. Titolo: cosa contiene (per esempio `Casi 0-2: baseline, geometria e aerodinamica v1.8`). Descrizione: l'elenco dei casi e quello che vuoi segnalare (dubbi, cose che non tornano).
5. **Create pull request**.

**Dopo.**
- Rivedo la PR e lascio commenti sui file.
- Per correggere non apri una PR nuova: correggi sulla tua branch, fai commit e push. La PR si aggiorna da sola.
- Quando va bene faccio io il merge. Tu **non** premi "Merge".
- Dopo il merge continui a lavorare sulla stessa branch.
