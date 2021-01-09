Link al corso https://app.pluralsight.com/library/courses/using-gitflow/table-of-contents

Ci sono diversi modelli che possiamo addottare con git, per esempio:
	- modello centralizzato
		abbiamo un repo centrale da dove tutto fanno clone e push
		si lavora su unica branch 
		assomiglia a SVN
		ok per team piccoli
	- feature branch (branch delle funzionalita')
		abbiamo un repo centrale
		ogni modifica apportata al codice avviene su un branch di sviluppo spesso chiamato feature branch 
		alla fine di sviluppo le modifiche sulla feature branch sono riportate nel repo centrale, branch master per esempio
	- GitFlow (quello che vediamo in questo corso)
	- Altri (per esempio GitHub ha un suo modello)
GitFlow
	abbiamo N branch 
	di solito abbiamo sempre branch master e develop
		develop di solito contiene minor changes, piccole modifiche 
		per funzionalita' nuove creiamo sempre feature branch
	per funzionalita' creiamo branch dedicate
		queste branch partono da develop
		modifiche sono riportate su develop al termine di sviluppo della funzionalita'
	per hotfix anche creiamo branch dedicate 
		sono create partendo da branch master
		la fix dopo viene riportata sia su master che develop
	per release creiamo branch dedicate (questa branch puo' essere eliminata una volta fatto il rilascio, ovviamente riportando le modifiche su branch core 
		come master e develop)
		release branch parte da develop
		qui possiamo fare test e vari fix
		dopo il rilascio le modifiche sono riportate sia su master che develop
	vedi la presentazione per un modello grafico
	installazione di GitFlow nel nostro progetto
		GitFlow e' un insieme di script che estendono git
		e' possibile usare i comandi standard di git, script hanno il ruolo di facilitatore, in background eseguono cmq comandi git
		si installa seguendo le istruzioni su https://github.com/nvie/gitflow/wiki/Installation
			dobbiamo scaricare getopt.exe, libiconv2.dll, libintl3.dll che si trovano nei vari archivi
			mettiamo questi tre file sotto C:\Program Files\Git\bin
		creiamo nostro repo con git init
		inizializziamo il repo per utilizzo di GitFlow digitando git flow init (rispondiamo alle domande, mettendo i nomi di branch che ci piacciono)
Creazione e utilizzo di feature branch
	sono branch dove una funzionalita' viene sviluppata
	simuliamo una collaborazione tra vari developer
	dev1 e' chi ha creato il repo e inizializzato gitflow 
	dev2 vuole sviluppare una nuova funzionalita' 
		cloniamo il nostro repo 
		prendiamo in checkout branch develop 
		inizializziamo git flow NOTA: ogni repo clonato deve lanciare il comando 
			git flow init x fare inizializzazione
		creiamo feature branch lanciando 
			git flow feature start 1-user-should-login
			questa branch per momento esiste solo in locale
			facciamo qualche modifica...
		lanciamo i comandi per versionare le modifiche in locale
			git add . 
			git commit -m "my commit message"
		per pubblicare branch di sviluppo sul server lanciamo
			git flow feature publish [feature branch name]
	dev3 ora vuole vedere le modifiche fatte per la nuova funzionalita'
		abbiamo il comando di help utile per vedere tutte le opzioni che abbiamo
			git flow feature help
		prendiamo in checkout branch di sviluppo lanciando 
			git flow feature track [feature branch name]	// con comando track abbiamo lincato il repo locale e remoto, possiamo fare push e pull
		facciamo qualche modifica e commitiamo tutto in locale
		per portare le modifiche sul server lanciamo
			git push
		un'altro dev (es. dev2) puo' fare pull per scaricare le modifiche in locale
	quando abbiamo finito gli sviluppi possiamo chiudere branch della funzionalita'
		prima possiamo fare una review delle modifiche fatte 
		con questo ci aiuta GitHub 
			creiamo Pull Request dal repo di GitHub scegliendo branch develop come branch di destinazione
			la Pull Request creata ci da la possibilita' di vedere le differenze tra due branch (tab Files changed)
			per non facciamo Merge della Pull Request, che comporterebbe alla chiusura di feature branch 
				di solito questa operazione lo fa il dev 
				quindi chiudiamo la nostra Pull Request su GitHub evitando il Merge 
		per chiudere la feature branch lanciamo il comando 
			git flow feature finish [feature branch name]
			questo comando esegue merge di tutte modifiche su develop -> elimina feature branch da locale e remoto -> esegue checkout di develop in locale 
			i dev che hanno feature branch in locale devono provedere alla sua eliminazione
				git branch -d [branch name]
			dopo la chiusura di branch non abbiamo ancora pubblicato niente sul server (le nostre modifiche sono in locale su develop)
			dobbiamo creare release
Creazione di release branch
	parte da branch develop 
	puo' contenere vari fix, dopo i test 
	viene chiusa mergiando tutto su master, e develop, se si sono state fatte delle modifiche
	release branch viene creata nel momento quando decidiamo di fare la release (es. ogni 2 settimane, ogni mese etc)
	passiamo a dev2
		creiamo release branch lanciando
			git flow release start [release name]
		pubblichiamo release branch per renderla disponibile agli altri (es. Q&A team)
			git flow release publish [release name]
		chi ha bisogno puo' scaricare release branch in locale lanciando 
			git flow release track [release name]
	dev3 fa qualche modifica e pusha tutta su branch di release
	dev2 ora puo' fare merge
		facciamo pull di ultime modifiche da release branch
		prendiamo in checkout develop
		git pull da develop per essere sicuri che abbiamo tutte le modifiche in locale
		facciamo merge lanciando
			git merge [release branch name]
		facciamo push da develop per caricare le modifiche nel repo centrale 
	adesso siamo pronti per chiudere release branch
		passiamo a dev1
		dev1 fa il pull da branch di rilascio e lancia
			git flow release finish [release branch name]
			lanciando questo comando, e se abbiamo fatto le modifiche su branch di release, vediamo questo riepilogo
				Summary of actions:
				- Release branch 'sprint-1-release' has been merged into 'master'
				- The release was tagged 'sprint-1-release'
				- Release tag 'sprint-1-release' has been back-merged into 'develop'
				- Release branch 'sprint-1-release' has been merged into 'develop'
				- Release branch 'sprint-1-release' has been locally deleted; it has been remotely deleted from 'origin'
				- You are now on branch 'develop'
			cmq le modifiche di merge di release branch non sono state ancora pushate nel repo centrale (remoto)
			quando prendiamo in checkout branch master vediamo il msg che ci dice che siamo avanti di N commit in locale
			il comando di prima ci ha creato anche il tag sprint-1-release
			facciamo push di tutto, anche del tag, lanciando il comando
				git push --tags
		ora abbiamo tutte le modifiche sul server
		come abbiamo visto release branch e' stata eliminata lascindo il tag utile a identificare il commit specifico del nostro rilascio
	vediamo ora come fare di hotfix
Creazione di hotfix
	parte da master
	utile per fare una fix veloce
	le modifiche sono riportate successivamente sia su master che develop
	i comandi sono simili a quelli visti prima
	creiamo hotfix branch lanciando 
		git flow hotfix start [name]
		dobbiamo fare il bump della versione, ricordiamolo!
	facciamo qualche modifica e commitiamo tutto in locale
	chiudiamo hotfix branch lanciando 
		git flow hotfix finish [name]
	abbiamo le modifiche sia su develop che master 
	push di tutto lanciando
		git push --all origin
Creazione di build
	demo riporta utilizzo di TeamCity
	tool utile per eseguie build di nostri progetti
	possiamo fare pull di N branch ed eseguire N build contemporaneamente
	creiamo progetto TeamCity
		creiamo VCS root specificando git e indirizzo del nostro repo
		creiamo build config
			possiamo avere piu' step
			creiamo trigger per far lanciare la build
	quando la modifica viene pushato sul branch che la build sta monitorando, parte la build
	in TeamCity possiamo configurare sezione Branch Specification impostando tutti branch che vogliamo
		tutte le modifiche pushate su queste branch fanno eseguire la build
		