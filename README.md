# docker-wordpress
Ambiente di sviluppo Docker per Wordpress con PHP-apache, MariaDB e MailHog.
Docker environment stack for Wordpress with PHP-apache, MariaDB and MailHog for developments.


## Panoramica

Richiesto **docker-compose** 


## Composizione dello Stack

* Wordpress - PHP-apache _(WEB)_
* MariaDB _(DB)_
* MailHog _(SMTP-TEST)_


## Immagini Docker

* wordpress:${WP_CORE_VERSION}-php${PHP_VERSION}-apache
* mariadb:10.3
* mailhog/mailhog


## Riferimenti

* https://hub.docker.com/_/wordpress
* https://hub.docker.com/_/mariadb
* https://hub.docker.com/r/mailhog/mailhog/
* https://hub.docker.com/_/php


## Alberatura di Progetto

```
docker-wordpress
│   .env
│   .env.example
│   .gitignore
│   docker-compose.yml
│   LICENSE
│   README.md
│       
├───db.backup
│   └───.gitignore
│
├───db.init
│   └───zzz-migrate.sql.example
│
└───wp-content
    ├───mu-plugins
    ├───plugins
    └───themes
```


## Variabili d'Ambiente

### File delle variabili: `.env`

_Il file `.env` è fondamentale! Utilizzare `.env.example` come template._

**Crea una copia del file `.env.example` rinominandolo `.env`**

Edita le variabili per il tuo ambiente, di seguito una descrizione di queste.

#### Variable description

* `PROJECT_NAME`: Nome assegnato allo stack docker.
* `VERSION_ID`: ID del progetto. (Da usare se si prevedono più versioni dello stack o del progetto).
* `WEB_PORT_EXPOSED`: Porta HTTP esposta per il sito web _(default 80)_.
* `PHP_VERSION`: Versione PHP che si desidera utilizzare _(default 7.2)_.
* `DB_PORT_EXPOSED`: Usata se si desidera connettersi al database anche con un cliente esterno _(HeidiSQL_, _MySQL Workbench, ecc.)_.
* `DB_ROOTPASS`: Password dell'utente root di MySQL. Ehy, in produzione è inaccettabile ma qui siamo su un ambiente di sviluppo ;)
* `DB_USER`: Username dell'utente database.
* `DB_PASS`: Password dell'utente database.
* `DB_NAME`: Nome del database.
* `WP_CORE_VERSION`: Versione di Wordpress che si desidera utilizzare.
* `WP_TABLE_PREFIX`: Prefisso delle tabelle sul database utilizzato da wordpress.


---


# Suggerimenti
**Cose di cui dovresti essere informato**


## Ambiente

Verifica in prima istanza il file `.env` se non esiste prendi spunto dal file `.env.example`

Data la gestione poco dinamica degli URL imposta da Wordpress sui contenuti caricati tramite la _dashboard wp-admin_, è buona norma aggiungere sul file `hosts` del proprio terminale un puntamento locale al DNS utilizzato dall'ambiente finale.
es. `127.0.0.1        www.ilmiositonelmondo.net`
Questa è una procedura opzionale ma fortemente consigliata se è previsto, terminati gli sviluppi, di migrare il sito in produzione.

Il repository così configurato (vedi bene il `.gitignore`) potrebbe essere eseguito, per il deploy, direttamente sul webserver (testing, pre-produzione, debug, produzione, ecc.) tramite **git**  senza dover utilizzare FTP/SCP/SSH.

Se devi lavorare su una versione già esistente del sito WP ed hai a disposizione il dump del [database](#Database) e i sorgenti [wordpress](#Wordpress) del sito, aggiungi il dump (`.sql` o `.gz`) dentro la cartella `db.init` e l'intera cartella `wp-content` nella cartella principale di progetto ed esegui il comando di build di [Docker](#Docker), ma prima cancella, se già presenti, i volumi già esistenti del tuo stack altrimenti non verranno apportate modifiche al tuo ambiente.


## Wordpress

Dopo la build, Wordpress e il server web gireranno all'interno di un container ad eccezione della cartella `wp-content` che sarà disponibile nella cartella principale di progetto.

### Informazioni per utenti esperti

Il file `wp-config.php` viene editato con i parametri indicati nel file delle variabili d'ambiente `.env`. Qualora fosse necessario apportare modifiche a tale file questo può avvenire accedendo direttamente all'interno del container Docker tramite il comando `docker-compose exec wp bash` e una volta dentro il container con bash ci troviamo in un ambiente linux ma con molti ma molti meno applicativi e comandi a disposizione, ovvero non sono presenti editor testuali (`nano`, `vi`, `pico`, ecc.) ma è presente il comando `sed` _(buona fortuna e in culo alla balena!)_
Se avere a disposizione i sorgenti del core WP è fondamentale, nel `docker-compose.yml`, dobbiamo indicare come `volume` di Wordpress un _percorso localmente raggiungibile_ anziché un _volume docker_ (righe 18,19 e 47 del file). Usando un _percorso localmente raggiungibile_ dopo la build sarà presente la cartella `wp` nella cartella principale di progetto. Qui potrà essere editato il file `wp-config.php` e potranno essere eseguiti operazioni tramite il proprio file manager su `wp-admin` e `wp-includes`. NB la `wp-content` usata dallo stack sarà sempre quella presente nella cartella principale di progetto e non quella dentro `wp`. 

Una lista di variabili d'ambiente da utilizzare è disponibile nel [Docker Hub Wordpress](https://hub.docker.com/_/wordpress). Queste variabili vanno aggiunte al file `docker-compose.yml` prima di una build. 

Comunque se non hai capito quello che è scritto o cosa siano i _volumi di docker_ forse è il caso che lasci il `docker-compose.yml` così com'è e ti fidi di Docker e dei manutentori di Wordpress del Docker Hub.

E' consigliato modificare il file `wp-config.php` solo per aggiungere gli switchs utili al _debug_, per esempio:

```
ini_set('log_errors', 'On');
ini_set('error_log', '/CUSTOM_LOGS_PATH/php-errors.log');
define('WP_ALLOW_REPAIR', true);
define('WP_DEBUG', false);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', true);
```


## Database

Per il database viene utilizzato un container MariaBD, nulla vieta di usare un container MySQL. I sorgenti dei databases risiedono in un _volume docker_. Salvare questi sorgenti su un percorso localmente raggiungibile potrebbe compromettere la build sopratutto se vengono utilizzate alcune versioni di MariaDB o MySQL.


### Importazione in caso di migrazione

Qualora avessimo a disposizione il dump di un database, di formato `.sql` o `.gz` può essere aggiunto dentro la cartella `db.init`. I file `.sql` o `.gz` presenti all'interno di questa cartella vengono eseguiti alla prima inizializzazione del server, nel nostro caso una _build_. Qui possono essere inseriti dump o file con all'interno query SQL. I file vengono eseguiti in ordine alfa-numerico crescente.
Es. abbiamo un dump di un sito in produzione che vogliamo importare sull'ambiente di sviluppo docker e dobbiamo modificare gli URL del sito e l'email di admin. Quindi caricherò il file `mio-dump.sql` dentro la cartella e modifico il file `zzz-migrate.sql` dove al suo interno sono già presenti alcune query da usare come esempio per svolgere le operazioni sopra descritte.


### Backup e Restore

Per effettuare i backup e restore del database è stata mappata una cartella `db.backup` presente nella cartella principale di progetto.

I backup si effettuano accedendo nella _bash_ del container _db_.

`docker-compose exec db bash`

Al suo interno ci troviamo in un ambiente linux dove è possibile eseguire comandi mysql.


#### Backup

##### Dump in SQL
`mysqldump --force --opt -uroot -ppassword --databases wordpress > /backup/wordpress-$(date +%U).sql`

##### Dump in GZ
`mysqldump --force --opt -uroot -ppassword --databases wordpress | gzip > /backup/wordpress-$(date +%U).sql.gz`


#### Restore

##### Aggiorna DB da SQL
`mysql -uroot -ppassword wordpress < /backup/wordpress-#.sql`

##### Nuovo DB da SQL
`mysql -uroot -ppassword < /backup/wordpress-#.sql`

##### Aggiorna DB da GZ
`gunzip < /backup/wordpress-#.sql.gz | mysql -uroot -ppassword wordpress`

##### Nuovo DB da GZ
`gunzip < /backup/wordpress-#.sql.gz | mysql -uroot -ppassword`

#### Per ripristino

Fai un _down_ dello stack e cancella il volume attuale del DB (vedi comandi [docker](#Docker)), inserisci uno dei backup (`.sql` o `.gz`) all'interno di `db-init` ed esegui nuovamente `docker-compose build -d`


### Connessione con client esterno

Per connettersi tramite client MySQL esterno (HeidiSQL_, _MySQL Workbench_, ecc.) i parametri da utilizzare sono i seguenti.

* _host_: `127.0.0.1` o `db`
* _port_: la variabile d'ambiente `DB_PORT_EXPOSED`
* _username_: `root` (in alternativa la variabile d'ambiente `DB_USER`)
* _password_: 
  * la variabile d'ambiente `DB_ROOTPASS` per `root` (accesso a tutti databases)
  * la variabile d'ambiente `DB_PASS` per `DB_USER` (accesso al solo database `DB_NAME`)


## MailHog

In questo stack è stato aggiunto MailHog che è un sistema di sviluppo semplice e poco invasivo per testare l'invio email.

Su Wordpress si consiglia di installare un plugin per invio email tramite SMTP (consiglio _[Easy WP SMTP](https://it.wordpress.org/plugins/easy-wp-smtp/)_).
I parametri di configurazione SMTP sono:
* _host_: `mailhog`
* _port_: `1025`
* nessuna autenticazione

Per controllare le email inviate (compresi MIME, Formati, ecc.) dal browser vai su http://localhost:8025


## Docker

Comandi docker in pillole.

* `docker-compose up -d --build` - costruisce e inizializza lo stack
* `docker-compose down` - rimuove lo stack mantenendo database e sorgenti
* `docker-compose start` - avvia lo stack
* `docker-compose stop`  - arresta lo stack 
* `docker-compose down --volumes` - come _down_ ma cancella anche i volumi (cancella database e sorgenti se non mappati localmente)
* `docker-compose config` - verifica la configurazione dello stack

* `docker-compose exec SERVICE_NAME bash` - accesso tramite terminale all'interno di un  container
  * SERVICE_NAME = `wp` - per accedere sul server web
  * SERVICE_NAME = `db` - per accedere sul server database
  * SERVICE_NAME = `mailhog` - per accedere sul server smtp

* `docker volume ls` lista dei volumi docker
* `docker volume rm NOME-VOLUME` cancella il volume docker indicato dal nome

