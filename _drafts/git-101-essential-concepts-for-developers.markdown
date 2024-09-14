---
alayout: post
title:  "GIT 101: Essential Concepts for Developers"
categories: git guide vcs
mermaid: true
---
Git e' il piu diffuso distributed Version Control System (VCS) in uso al mondo. Il suo scopo e' di aiutare a mantenere e gestire un insieme di file che cambia nel tempo registrando tutte le modifiche effettuati ad esso e aiutando a gestire la progressione concorrente di molteplici persone sullo stesso insieme di file.

Nonostante sia principalmente utilizzato in programmazione per la gestione di source code, puo venir utilizzato per la gestione di qualunque file di testo che cambia nel tempo (purtroppo e' inutile per file come Word o Excel che non sono in formato "raw text").

## Teoria - Caratteristiche di una repository

--- immagine globale con repository, commit, branch e remote

La **repository** e' il nome che viene dato ad un progetto che viene gestito tramite git.

L'idea di fondo di GIT e' che ogni cambiamento di stato del codice sorgente e' rappresentabile da una lista di "differenze", in particolare linee aggiunte e linee rimosse. Se ad esempio modifichi una linea, essa contera come una rimozione e un aggiunta. Questi "pacchetti di modifiche" vengono chiamati **commit** e sono il mattone fondamentale che git utilizza per funzionare. Qualunque funzionalita di git si puo ricondurre a come creare/gestire/modificare/fondere/ispezionare commit.

In maniera che quasi ricorda la piu moderna blockchain, ogni commit contiene una lista di link ai commit precedenti, se presenti, e viene rappresentato da un hash univoco.

--- esempio di un commit

Questo permette di poter ricostruire l'albero dei commit tramite questi link, e abbiamo una sorta di certezza che i commit precedenti non possono essere modificati senza ricreare tutti i commit, permettendo anche in caso di disastri/modifiche "indietro nel tempo", di non rischiare di perdere a caso i dati.

Ogni repository e' quindi rappresentabile come un albero di commit tra loro interconnessi

--- esempio di grafico di commit e branch

Per aiutare a tenere traccia di dove si e' rimasti con i lavori, esistono le **branch**, tra cui una speciale chiamata master/main.

Una branch altro non e' che un "cursore" at uno specifico commit, che puo venir spostato ad un altro commit come ad esempio un commit appena creato sopra quello attuale

--- grafico di movimento branch quando crei un nuovo commit

Grazie a questa proprieta le branch vengono di solito utilizzate per rappresentare i "rami di lavoro", ovvero quei commit su cui si sta lavorando per fare qualcosa. Un esempio classico di configurazione di una repository e' avere una branch master con la versione "production"/finale del software, e varie branch su cui vengono lavorate le varie feature che, una volta completate, verranno immesse dentro master.

Andiamo ora a nominare l'ultima caratteristica essenziale della teoria, ovvero il **merge**.

Quando hai piu branch con uno stato differente, puoi ritrovarti a dover immettere le modifiche di una delle branch all'interno dell'altra. Questo processo si chiama merge e crea un commit speciale figlio delle teste di entrambe le branch, in cui le differenze tra le due branch vengono unificate, eventualmente con aiuto umano se necessario.

--- grafico di un merge

Dopo questa infarinatura sui principi fondamentali di GIT in locale, e' ora tempo di guardare la parte "distributed" di GIT, ovvero la possibilita di sincronizzare il proprio codice con una repository remota su piattaforme come GitHub e GitLab.

## Teoria - Distributed Repository

Una repository GIT ha gia le sue utilita in offline per gestire e tenere traccia delle proprie modifiche al proprio codice, ma la sua forza diventa ancora piu evidente quando si lavora ad un progetto assieme multiple persone o multiple macchine.

Il principio fondamentale e' che ogni repository puo avvere dei **remote** ovvero dei link speciali, come ad esempio `git@github.com:utente/repoprogetto.git`, che git puo utilizzare per leggere lo stato remoto della repository. La stragrande maggioranza delle repo ha un singolo remote chiamato **origin**.

L'idea e' poter scaricare dal remote la lista delle branch remote e i commit online e poterci poi lavorare offline, che appariranno come branch speciali con nome `origin/nomebranch`.

--- grafico di esempio di un albero commit con repo locali e remote.

Oltre al poter fetchare informazioni dai remote, si puo anche pusharle, caricando i propri commit e la posizione delle proprie branch online. Diventera (spero) tutto piu chiaro alla fine della spiegazione sui comandi.

## Comandi - How to do!

### Creazione Repository

I primi comandi importanti sono quelli per creare una repo.

I comandi tipici sono:

```bash
git init  # inizializza una repository vuota nella cartella attuale
git clone CLONE_URL  # crea una nuova repository copiando lo stato di una repository remota
```

Il modo piu classico di creare una repo e' generalmente creare una repo su GitHub e clonarla, in modo da avere gia tutta la logica online preconfigurata.

Se si procede a clonare una repository di qualcun'altro per cui si hanno permessi di lettura ma NON scrittura, la repository puo essere modificata in locale ma non inviare le modifiche online. Per poter creare una propria versione della repo di qualcun'altro, esiste il **fork**, che spieghero piu avanti.

Importante ricordare the git ha un manuale di prima categoria estremamente completo, a volte fin troppo, che si puo accedere con `git --help`. Il manuale e' disponibile per ogni sottocomando, come ad esempio `git clone --help` o `git checkout --help`.

### Status e Informazioni

Il secondo set di comandi sono quelli di informazioni

##### `git status`

Questo comando permette di osservare lo stato attuale del commit in cui ci troviamo, chiamato nel sistema `HEAD`.

```
fpasqua@EisterBox:~/git/eisterman.github.io$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   test

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   _posts/2024-07-07-welcome-to-jekyll.markdown

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        _posts/GIT.md
```

Grazie a questo comando possiamo vedere a colpo d'occhio le informazioni piu importanti sullo stato attuale della repo, in particolare:

- `On branch`, ovvero su che branch stiamo lavorando e se sono presenti differenze tra la branch locale e quella remota

- `Changes to be committed`, ovvero i file le cui modifiche sono state selezionate per l'aggiunta nel prossimo commit creato

- `Changes not staged for commit`, ovvero file che sono stati modificati ma che non verranno aggiunti al prossimo commit creato

- `Untracked files`, ovvero file che non sono presenti in GIT e che quindi non vengono tracciati. Praticamente i file ignorati da GIT.

Comondamente git status mostra anche qualche comando utile per aggiungere/togliere dai commit, cosi da non doverseli cercare ogni volta.

##### `git log` e `git tree`

Comando estremamente potente permette di vedere lista e dettagli dei commit, la posizione delle varie branch e, con le giuste opzioni il grafo dei commit.

La versione piu semplice e' `git log` senza opzioni che mostra solo una generica lista di commit in ordine di tempo, senza mostrare i legami relativi tra i vari alberi

```
commit 39b3fea912a4ffe687b6d22199b2ada4d8b65778 (HEAD -> hotfix_release, origin/hotfix_release)
Author: Federico Pasqua (eisterman) <federico.pasqua.96@gmail.com>
Date:   Mon Jul 8 10:47:37 2024 +0200

    re.parkadmin.charge: Add sum endpoint to obtain the precalculated sum of the charge requested by filter.

commit bf35aae8e50e45850b3c1a1f42f070f460f1d9c4
Author: Production RE 1 <robot@rossinienergy.com>
Date:   Fri Jul 5 14:47:08 2024 +0000

    re.parkadmin.chargemap: Use GET instead of POST

commit 6bc42a94599cd5be1f3e12c9865297a10c94ebd0
Author: RE Staging1 <robot@rossinienergy.com>
Date:   Fri Jul 5 13:33:42 2024 +0000

    MIGRATION 036 - Chargemap Business Admin
```

Un informazione importante e' capire in che stato attualmente e' la repository. Questo viene indicato da `HEAD` che rappresenta lo stato attuale. Ogni volta che si vede in giro `HEAD` generalmente ci si riferisce al commit attualmente selezionato il cui stato e' impostato nella repo.

Il Git Log grezzo non e' il modo piu semplice per capire cosa sta succedendo, in particolare per la mancanza delle connessioni tra i commit, per quello esiste `git log --graph` che mostra la stessa lista di prima, ma con i collegamenti tra i commit

```
* commit 6bc42a94599cd5be1f3e12c9865297a10c94ebd0
| Author: RE Staging1 <robot@rossinienergy.com>
| Date:   Fri Jul 5 13:33:42 2024 +0000
|
|     MIGRATION 036 - Chargemap Business Admin
|
*   commit f2367b5c1c78ddc0ebc0a8d2efa54232a51c9f1c
|\  Merge: fbd610a f99a3da
| | Author: Federico Pasqua (eisterman) <federico.pasqua.96@gmail.com>
| | Date:   Fri Jul 5 15:27:18 2024 +0200
| |
| |     Merge branch 'refs/heads/chargemap_integr' into hotfix_release
| |
| * commit f99a3dad16fc7ae81a0d814c6566b9e4687dc84a (origin/chargemap_integr)
| | Author: Federico Pasqua (eisterman) <federico.pasqua.96@gmail.com>
| | Date:   Mon Jul 1 14:04:00 2024 +0200
| |
| |     re.parkadmin.cmb: Add export endpoint proxy
| |
```

Rimane comunque molto clutterato da informazioni non necessarie nel 90% dei casi, quindi ho costruito un comando che mostra un albero semplice e immediato:

```bash
git log --graph --full-history --all --color --pretty=format:"%x1b[31m%h%x09%x1b[32m%d%x1b[0m%x20%s"
```

Questa variante di log permette di avere un albero compatto con solo branch, Commit Hash e Commit message:

```
* 39b3fea        (HEAD -> hotfix_release, origin/hotfix_release) re.parkadmin.charge: Add sum endpoint to obtain the precalculated sum of the charge requested by filter.
* bf35aae        re.parkadmin.chargemap: Use GET instead of POST
* 6bc42a9        MIGRATION 036 - Chargemap Business Admin
*   f2367b5      Merge branch 'refs/heads/chargemap_integr' into hotfix_release
|\
| * f99a3da      (origin/chargemap_integr) re.parkadmin.cmb: Add export endpoint proxy
| * 997867e      re.parkadmin: Add ChargeMap Business Admin integration model fields
* | fbd610a      re.charge: Add Grace Period during the reboot time of the Pilotage, to not close charges for RP that take too much time to be back online.
|/
| * 81d8f6e      (origin/fix_csauth_v1public) re.public.auth: Add qrcode_allowed to FeaturedUser to replace the old legacy retrieve_user_charge_permissions
|/
* 776f16c        re.reports.yearly: Send Yearly Report only for park with at least one CS with QRCode
```

Per evitare di doverlo scrivere ogni volta una buona idea e' crearsi un alias bash oppure impostarlo come git alias globale. Io preferisco la seconda opzione.

Runnando il comando

```
git config --global alias.tree "log --graph --full-history --all --color --pretty=format:\"%x1b[31m%h%x09%x1b[32m%d%x1b[0m%x20%s\""
```

Viene inserito nel proprio `~/.gitconfig` file la configurazione che consente di richiamare questo lungo comando semplicemente con `git tree`. Uno strumento veramente potente per il lavoro quotidiano.

##### `git show`

Questo comando permette di vedere il contenuto di un commit, ovvero tutti i suoi metadata e infine il `diff` ovvero la lista di aggiunte e rimozioni che il commit contiene. Il nome viene dall'utility utilizzata per generare questi file ed il relativo formato, ovvero `diff`!

```
roberto@production-api-charge-re:~/backend_rossinienergy$ git show  f99a3da
commit f99a3dad16fc7ae81a0d814c6566b9e4687dc84a (origin/chargemap_integr)
Author: Federico Pasqua (eisterman) <federico.pasqua.96@gmail.com>
Date:   Mon Jul 1 14:04:00 2024 +0200

    re.parkadmin.cmb: Add export endpoint proxy

diff --git a/re_restapi/views/parkadmin/current/chargemap.py b/re_restapi/views/parkadmin/current/chargemap.py
new file mode 100644
index 0000000..672bc61
--- /dev/null
+++ b/re_restapi/views/parkadmin/current/chargemap.py
@@ -0,0 +1,49 @@
+import logging
+import requests
+
+from rest_framework import serializers, status
+from rest_framework.decorators import api_view, permission_classes
+from rest_framework.response import Response
+from drf_spectacular.utils import extend_schema, inline_serializer, OpenApiTypes
+from re_restapi.libs.permissionviewset import *
```

Tra le informazioni nel metadata abbiamo autore, data e commit message. Da `diff` in poi, c'e la rappresentazione in diff format delle modifiche contenute nel commit.

### Creazione Commit

La creazione di commit e' il cardine del lavoro quotidiano su git.

Il processo logico dietro e' dividere il lavoro in step:

1. Ti assicuri di essere sulla branch corretta su cui devi lavorare, ad esempio `master`.

2. Fai le tue modifiche, test, etc...

3. You stage the modification for the future commit, ovvero selezioni quelle di cui hai bisogno di inserire nel nuovo commit

4. Crei il commit aggiungendo un commit message che descriva BENE le modifiche che hai appena fatto. Ricorda sempre che mentre puo venir voglia di fare commit message come "bugfix", nel momento in cui hai un disastro e devi tornare indietro sui tuoi passi, messaggi poco chiari come questo ti obbligano a leggere _commit per commit_ le modifiche facendoti perdere millenni di tempo!

Facciamo ora un esempio di creazione commit.

Immaginiamo di avere una repo in cui siamo su master e dobbiamo modificare due file, `A.py` e `B.py`. Se volete provare potete andare in una cartella vuota e scrivere questo sul terminale per inizializzare la branch al mio stesso initial state:

```bash
git init
touch A.py B.py
git add A.py B.py
git commit -m "Initial Commit"
```

Continuando col tutorial capirete cosa viene fatto in questo snippet.

Dopo esserci assicurati con `git status` che siamo nel posto giusto, facciamo le nostre modifiche.

`git status` a questo punto ci mostrera una cosa del genere:

```
fpasqua@EisterBox:~/git/test$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   A.py
        modified:   B.py

no changes added to commit (use "git add" and/or "git commit -a")
```

A questo punto cominciamo a poter fare due cose: `git add` e `git restore`.

##### `git restore`

Questo comando permette di cancellare le modifiche effettuate e non committate su un file trackato da git. Se ad esempio facessi `git restore B.py` tutte le modifiche che ho fatto su `B.py` che lo rendono diverso dallo stato attuale della repository, verrebbero cancellate.

Puo risultare comodo in certe situazioni.

##### `git add`

Questo comando essenziale permette di flaggare un file modificato per il commit, ovvero renderlo **staged**.

Se facciamo dunque `git add A.py` il successivo `git state` ci dara:

```
fpasqua@EisterBox:~/git/test$ git add A.py
fpasqua@EisterBox:~/git/test$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   A.py

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   B.py
```

Qui vediamo come `A.py` sia incluso nel prossimo commit mentre B.py no.

Notiamo anche che `git status` ci ha suggerito un nuovo comando per fare l'unstage.

##### `git restore --staged`

Questa variante di `git restore` permette di rimuovere un file da un commit senza pero cancellarne le modifiche. Praticamente e' l'inversa esatta di `git add`. Questo comando non provoca alcuna perdita di dati quindi puo essere usato con tranquillita, prestando attenzione che ci sia l'opzione `--staged` quando si usa, altrimenti si perderanno le modifiche nel file!

Continuando con il nostro esempio, possiamo ora creare un commit.

##### `git commit`

La creazione di un commit viene eseguita facendo direttamente `git commit`, che aprira una finestra in un editor di testoin cui inserire il commit message.

Una volta salvato il file temporale, git creera il commit.

Una variante a questa procedura molto comoda se il commit message non e' cosi lungo da richiedere un editor di testo, e' `git commit -m "testo del commit message"` che permette di dare il commit message direttamente da linea di comando.

```
fpasqua@EisterBox:~/git/test$ git commit -m "Modifico A.py aggiugendo feature X"
[master 31ebe42] Modifico A.py aggiugendo feature X
 1 file changed, 1 insertion(+)
fpasqua@EisterBox:~/git/test$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   B.py

no changes added to commit (use "git add" and/or "git commit -a")
```

E facendo `git tree` possiamo notare il nuovo commit sulla branch master:

```
fpasqua@EisterBox:~/git/test$ git tree
* 31ebe42        (HEAD -> master) Modifico A.py aggiugendo feature X
* bd883a1        Initial Commit
```

Queste sono le info basilari per creare un commit su una branch selezionata.

Pero se potessimo solo creare commit, diventerebbe difficile gestire il lavoro di piu persone, e GIT e' nato esattamente per gestire quello.

Andiamo ora piu a fondo sulle branch ed i loro superpoteri.

### Branch

Come abbiam detto in precedenza una branch si puo immaginare come un "cursore" che punta ad un commit e va ad indicare dove ci troviamo a lavorare.

Esistono innumerevoli approcchi alla gestione delle branch ma il piu semplice da capire, che non e' per forza il migliore specialmente per progetti con piu persone, e' chiamato [Feature Branch Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow), dove si ha la master branch tenere traccia della versione finale del software, e esistono branch separate per ogni feature, che poi vengono fuse dentro master quando la feature e' pronta:

--- grafico di feature branch con master + 1 branch gia mergiata e 1 da mergiare

Le operazioni fondamentali che si possono fare sulle branch sono:

1. Checkout: selezionare una branch come quella in uso. Il contenuto della repository verra messo nello stato attuale della branch.

2. Creazione: creare una nuova branch che punti ad un commit gia esistente.

3. Merge: immettere i commit della branch B all'interno di A, fondendo i due percorsi e creando un "merge commit" speciale in A.

4. Reset: resettare lo stato di una branch ad un commit precedente.

5. Rebase: L'Arte Segreta di riscrivere la Storia. Operazione estremamente potente che permette di riscrivere l'intera struttura di una branch.

##### `git checkout`

Probabilmente l'operazione piu importante di tutto GIT. Questo comando permette di selezionare lo stato attuale della repository. 

La sintassi e' `git checkout <commit|branch>` ovvero si puo addirittura selezionare un singolo commit per riportare la repo a quello stato. Questo stato speciale in cui si checkout un commit e non una branch viene detto *Detached Head*, perche HEAD lo stato attuale della repo non e' direttamente collegato ad una branch, ed ogni nuovo commit rischia di perdersi nei meandri del grafo di Git. Questo perche Git considera "utili" solo i commit collegati alla storia di una branch.

Checkout su branch e' un operazione comune, permette di mettere la repo nello stato associato a quella branch ed eventualmente di fare e committare modifiche su di essa.

Ricordate in `git status` dove dice `On branch master` ? Quella e' la branch attualmente in checkout, e dove verranno eseguiti i nuovi commit.

--- immagine checkout branch con HEAD che si sposta da master a branch feature1

##### `git checkout -b`

Checkout e' lo stesso comando che si usa per CREARE una nuova branch.

```
fpasqua@EisterBox:~/git/test$ git tree
* 31ebe42        (HEAD -> master) Modifico A.py aggiugendo feature X
* bd883a1        Initial Commit
fpasqua@EisterBox:~/git/test$ git checkout -b feature1
Switched to a new branch 'feature1'
fpasqua@EisterBox:~/git/test$ git tree
* 31ebe42        (HEAD -> feature1, master) Modifico A.py aggiugendo feature X
* bd883a1        Initial Commit
fpasqua@EisterBox:~/git/test$
```

Come possiamo notare da `git tree`, adesso HEAD punta alla nuova branch feature1 e non a master! Vuol dire che se adesso noi creassimo un nuovo commit modificando un file `B.py` e committandolo, il nuovo commit sarebbe associato a feature1!

```
fpasqua@EisterBox:~/git/test$ git add B.py
fpasqua@EisterBox:~/git/test$ git commit -m "Modifico B.py per feature 1"
[feature1 86b5f83] Modifico B.py per feature 1
 1 file changed, 1 insertion(+), 1 deletion(-)
fpasqua@EisterBox:~/git/test$ git tree
* 86b5f83        (HEAD -> feature1) Modifico B.py per feature 1
* 31ebe42        (master) Modifico A.py aggiugendo feature X
* bd883a1        Initial Commit
```

In questo modo si puo lavorare su piu percorsi senza mischiare i cambiamenti delle varie feature, in modo da poter capire facilmente se si rompe qualcosa, COSA e PERCHE si e' rotto.

Ipotizziamo ora che in master ci siano altri commit:

```
fpasqua@EisterBox:~/git/test$ git checkout master
Switched to branch 'master'
fpasqua@EisterBox:~/git/test$ touch nuovofile
fpasqua@EisterBox:~/git/test$ git add nuovofile
fpasqua@EisterBox:~/git/test$ git commit -m "Aggiungi Nuovo File!"
[master 4c0a83f] Aggiungi Nuovo File!
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 nuovofile
fpasqua@EisterBox:~/git/test$ git tree
* 4c0a83f        (HEAD -> master) Aggiungi Nuovo File!
| * 86b5f83      (feature1) Modifico B.py per feature 1
|/
* 31ebe42        Modifico A.py aggiugendo feature X
* bd883a1        Initial Commit
```

Ora immaginiamo che feature1 sia completa e debba venir reimmessa in master. Questa operazione si chiama merge.

##### `git merge`

Il merge funziona secondo una logica preimpostata, che puo anche venir cambiata da argomento, ma che generalmente funziona cosi:

1. `git checkout branch-che-riceve` per andare nella branch in cui devo immettere `feature-branch`

2. `git merge feature-branch` per avviare il merge. Si aprira un editor di testo per modificare il messaggio di commit di default (che generalmente va bene)

3. Se il merge puo venir fatto tramite *fast-forwarding*, ovvero senza creare un merge commit, viene fatto in questo modo

4. Se il FF non e' possibile, viene creato un merge commit all'interno di `branch-che-riceve` con tutte le modficihe di `feature-branch`

5. Se ci sono dei conflitti, come ad esempio porzioni di file modificati da entrambe le branch, si attiva la modalita **merge conflict** che richiede all'operatore di risolverli a mano.

--- immagine di fast-forwarding

Ecco un esempio nel caso in cui non ci siano conflitti:

```
fpasqua@EisterBox:~/git/test$ git tree
* 4c0a83f        (HEAD -> master) Aggiungi Nuovo File!
| * 86b5f83      (feature1) Modifico B.py per feature 1
|/
* 31ebe42        Modifico A.py aggiugendo feature X
* bd883a1        Initial Commit
fpasqua@EisterBox:~/git/test$ git merge feature1
Merge made by the 'ort' strategy.
 B.py | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
fpasqua@EisterBox:~/git/test$ git tree
*   13b099b      (HEAD -> master) Merge branch 'feature1'
|\
| * 86b5f83      (feature1) Modifico B.py per feature 1
* | 4c0a83f      Aggiungi Nuovo File!
|/
* 31ebe42        Modifico A.py aggiugendo feature X
* bd883a1        Initial Commit
```

Come si puo notare, la branch `feature1` non viene minimamente toccata.
