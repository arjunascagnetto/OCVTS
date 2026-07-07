# 🙋 README

Il Documento e’ diviso in tre parti principali. 

1\) **INTRODUZIONE**.

2\) **BUILDERS**

3\) **DATAMART**

**Nella sezione INTRODUZIONE**  
Trovate la descrizione di cosa sia il Repository e in generale introduce i concetti base per capire i due passaggi successivi: i builders e i datamarts.

**Nella sezione BUILDERS**  
Ci sono le descrizioni di come dai dati raw L0 vengono costruite le tabelle di livello 1\. E di come da L1 si costruiscono le L2. Invece, L3 e L4 sono sempre coorte specifiche quindi non c’e’ un flusso generale di costruzione.

**Nella sezione DATAMART**  
Ci sono le descrizioni di come le variabili di livello 0 1 2 3 vengano usate per generare le variabili che voi effettivamente vedete nel datamart.

# Intro OCVTS

# Il Rer

Il Repository Epidemiologico Regionale (RER) è un data warehouse gestito da Insiel S.p.A. su mandato dell’ARCSS. Al suo interno non sono presenti dati che consentano l’identificazione diretta degli individui: ogni sei mesi viene generata una nuova “key\_anagrafe”, una chiave pseudonimizzata che identifica univocamente ciascun soggetto.

Nel RER confluiscono numerose [fonti dati](#bookmark=kix.bj34j5v9d95j), prevalentemente amministrative, ma anche alcune cliniche verticali, attraverso un articolato processo ETL e successivi controlli di consistenza. Un elemento distintivo del sistema del Friuli Venezia Giulia (FVG) è la presenza nel RER dei risultati degli esami di laboratorio eseguiti presso i laboratori pubblici della Regione (DNLAB).

La profondità temporale dei dati varia a seconda del flusso informativo: le SDO risalgono fino al 1985, mentre le anagrafiche contengono dati anche anteriori, relativi a chiunque abbia avuto contatti con il sistema sanitario regionale, indipendentemente dalla residenza. Altri flussi hanno profondità inferiori: i dati di laboratorio sono disponibili dal 2009, quelli relativi alla farmaceutica convenzionata dal 1995, il CUP dal 2013 e il PS dal 2000\.

Oltre ai flussi amministrativi, il RER include anche fonti cliniche, come C@rdioNet, un software gestionale verticale che rappresenta la cartella clinica cardiologica, compilata regolarmente da medici cardiologi e personale infermieristico a ogni contatto diretto con il paziente, a partire dal 2010\. Fino al 2015, C@rdioNet era accessibile solo tramite un portale regionale basato su Business Object; successivamente è stata integrata completamente nel RER, consentendo così l’accesso ai dati clinici in sinergia con gli altri flussi amministrativi.

Nel RER, la cartella C@rdioNet è suddivisa in 13 tabelle, prive di chiavi esterne di collegamento: tutte sono unite esclusivamente mediante la key\_anagrafe. Questo implica che, per caratterizzare un individuo in un determinato momento (es. situazione anamnestica o follow-up), è necessario definire regole temporali per correlare le informazioni provenienti dalle diverse tabelle.

# I dati

L’OCVTS utilizza i dati archiviati nel RER, sia di natura amministrativa sia clinica, come descritto sopra.

I dataset effettivamente impiegati per analisi cliniche, epidemiologiche o di governance sono il risultato di un complesso processo di costruzione di variabili a partire dai dati grezzi presenti nel RER. Tali dati vengono organizzati in livelli progressivi di raffinatezza:

**L0 – dati originari contenuti nel RER.**

A questo livello le tabelle sono quelle direttamente accessibili nel RER e derivate dai flussi informativi amministrativi e clinici, dopo un processo ETL in carico a Insiel S.p.A.. L’elenco non esaustivo delle tabelle che rientrano in questo livello include:

* Anagrafica (anagrafe generale, nascite, decessi, identificazione genitori, residenze, domicili)  
* Ricoveri ospedalieri (schede di dimissione ospedaliera)  
* ADI (assistenza domiciliare integrata)  
* PIC (prestazioni intermedie domiciliari)  
* RSA (residenze sanitarie assistenziali)  
* Esenzioni  
* Farmaceutica territoriale (sistema pubblico di distribuzione del farmaco)  
* Farmaceutica ospedaliera e diretta[^1]  
* PS (prestazioni di pronto soccorso)  
* anatomia patologica (referti codificati SNOMED)  
* CUP (prenotazioni del centro unico)  
* Prestazioni ambulatoriali  
* DNLAB (esami di laboratorio – solo laboratori pubblici FVG)  
* C@rdioNet (13 tabelle cliniche non collegate tra loro da chiavi esterne, ma tutte unite tramite key\_anagrafe)

Queste rappresentano la base dati originaria da cui si costruiscono, attraverso vari passaggi di elaborazione, le variabili di livello L1, L2, L3 e L4.

**L1 – Livello di prima aggregazione**:

DNLAB: aggregazione di codici che identificano lo stesso esame di laboratorio (es. emoglobina glicata: HBGL%, HBGL, HBA1C, A1C, ecc.).

FARMA: codifica in classi farmacologiche degli ATC (es. "alfa bloccanti": C02CA04, G04CA01, ecc.).

SDODIA: classificazione dei ricoveri secondo i codici ICD9 (es. specifici codici per lo scompenso cardiaco cronico secondo la definizione PNE).

CARDIA: classificazione delle diagnosi presenti in C@rdioNet basata su descrizione, sede e gravità (es. “OBESITÀ” con vari gradi: lieve, moderato, severo, ecc.).

**L2 – Livello di aggregazione intermedia:**

DIAGG: aggregazione di diagnosi tra SDODIA e CARDIA.

LABUNI: unificazione dei dati di laboratorio tra DNLAB e i referti C@rdioNet (es. profilo lipdico).

EVENTI: identificazione di eventi complessi (es. MACE 3p, 5p, MALE), combinando variabili L1 e L0 (es. data MACE 3p come minima tra decesso, infarto e stroke).

**L3 – Livello di diagnosi integrate:**

DIAINT: diagnosi integrate da fonti L0, L1 e L2 (es. diagnosi integrata di diabete definita dalla data minima tra esenzione 013, valori elevati di emoglobina glicata, prescrizioni farmacologiche e apertura diagnosi in DIAGG).

**L4 – Classificazione avanzata:**

CLRCV: classi di rischio cardiovascolare.

Alcune variabili sono precostituite e disponibili indipendentemente dalla coorte d’indagine; altre, come le diagnosi integrate (DIAINT), sono costruite solo per la coorte specifica in esame. Questa scelta rappresenta un compromesso tra esigenze di spazio nel RER e tempi di calcolo (es. l’etichettatura dei ricoveri richiede circa 6 ore in versione non parallelizzata).

# I protocolli

Esistono 3 diversi prototipi di studi, chiamati protocolli : 

* **CLINICO** –  attinente agli studi clinici con una coorte ben definita \[ l’unita’ statistica e’ o la persona o un evento, un esame\]  
* **PDTA** – attinente ai 3 pdta SCC, BPCO, DIABETE \[l'unità statistica sono le persone affette da tale patologia ripetute su tutti gli anni di indagine\].   
* **EPI4M** – una via di mezzo tra un clinico e un pdta su tutte e 3 le patologie

**Coorte PDTA**

**Coorte EPI4M**

**Coorte CLINICO**

La definizione della coorte nel protocollo Clinico e’ fase fondamentale di ogni studio. Essa non può essere standardizzata e richiede una stretta collaborazione tra il programmatore SAS e il referente clinico (medico o infermieristico), al fine di tradurre i criteri clinici di inclusione/esclusione nel linguaggio tecnico del RER.

Una volta definita la coorte, si procede con l’aggiunta delle variabili di follow-up per la valutazione degli outcome, qualora non siano già presenti. Le variabili L1, L2, L3 e L4 possono essere aggiunte alla coorte in modo semi-automatico: il ricercatore seleziona, tramite un foglio Excel strutturato, quali variabili includere, come rinominarle e in quale ordine disporle nel datamart finale.

# Le Variabili \[standard\]

Segue elenco delle classi di variabili gia’ pronte. Le variabili per i protocolli PDTA e EPI4M sono a selezione fissa. Per il protocollo Clinico il referente puo’ scegliere cosa trattenere.

* **diagnosi**  
  - diagnosi aggregate   
  - anemia integrata  
  - BCPO integrata  
  - ipercol familiare integrata  
  - COVID integrata  
  - diabete integrata  
  - dislipidemia integrata  
  - FA integrata  
  - ipertensione integrata  
  - IRC integrata  
  - microalbuminuria integrata  
  - obesità integrata  
  - RCVMA integrata  
  - scompenso integrata  
  - dialisi integrata  
* **esami strumentali**  
  - elettrocardiografie ECG  
  - ecografie ECO  
  - spirometrie  
  - parametri funzionali  
  - fenotipo  
  - laboratorio  
* **eventi**  
  - MALE  
  - MACE 3P 5P  
  - eventi ospedalizzazione  
  - emorragie maggiori  
* **prestazioni**  
  - prestazioni CUP \- altro  
  - prestazioni CUP \- ECG  
  - prestazioni CUP \- ECO  
    * prestazioni CUP \- ecovasco  
    * prestazioni CUP \- ecocardio	  
  - prestazioni CUP \- tutte  
  - prestazioni CUP \- spiro  
  - prestazioni CUP \- visite  
  - prestazioni Cardionet  
  - pronto soccorso  
* **score**  
  - Charlson score  
  - ESC score  
  - SCORE2  
  - classi di rischio cardiovascolare  
* **terapia**  
  - farmaceutica  
  - terapia Cardionet

# Obiettivi principali dell’OCVTS

Definizione di coorti epidemiologiche nella popolazione del FVG (es. soggetti a rischio o con patologia specifica in un certo periodo e area) per analizzare caratteristiche, prognosi, aderenza terapeutica, effetti di trattamenti, fattori prognostici/predittivi e interazioni tra caratteristiche e trattamenti.

Valutazione dei percorsi assistenziali di specifiche categorie di pazienti (es. soggetti con infarto miocardico acuto in un dato intervallo temporale e area), al fine di verificare il rispetto degli standard di cura in termini di farmaci, dispositivi, prestazioni specialistiche e setting assistenziali.

Integrazione sistematica di fonti eterogenee, incluse quelle non strutturate (es. segnali ECG, immagini ecocardiografiche), per arricchire l’insieme dei possibili fattori prognostici e migliorare la stratificazione del rischio e la diagnosi.

# Tabelle

|  | LIVELLO 0 | LIVELLO 1 | LIVELLO 2 | LIVELLO 3 |
| :---- | :---- | :---- | :---- | :---- |
| DNLAB | 2009 | MAX |  |  |
| PNLAB[^2] | 2010 \- 2016 | MAX |  |  |
| SDO | 1986 | MAX |  |  |
| FARMA TERR. | 1995 | MAX |  |  |
| AMBULATORIALE[^3] | 1998 |  |  | 2000 |

I MISSING significano che se c’e’ un flusso a quel livello usa la profondita’ del livello prima.

Le ESENZIONI non sono aggregate in tabelle per anno , quindi a me non e’ possible rispondere a questa domanda. La data minima di apertura di una esenzione e’ il 1/1/1900. 

CardioNet e’ un gestionale importato nel RER e decomposto in 13 tabelle. Noi usiamo la tabella Diagnosi per costruire le diagnosi aggregate \[LV2\]. La tabella DIAGNOSI e’ una monotabella \[data di creazione 1/1/1960 cioe’ insignificante\] e come per le esenzioni la data di prima apertura di una diagnosi e’ il 1/1/1900. In generale nei documenti dicevamo sempre che la data e’ il 1/11/2009 giorno di entrata in servizio di Andrea come direttore.

 

# BUILDERS

# FARMACEUTICA

# BUILDER 

Sono stati presi in considerazione 1705 codici ATC. Questi sono stati inseriti in 62 classi, non mutualmente esclusive \[ cioe’ lo stesso atc puo’ appartenere a piu’ classi \]. Poi per motivi di dimensione della tabella finale, le 62 classi sono state divise in 13 tabelle, dette macro classi. 

	

| MACRO CLASSE | DESCRIZIONE MACRO CLASSE |
| ----- | ----- |
| ACP | Agenti\_Cardiovascolari\_Primari |
| ACSGS | Agenti\_Cardiovascolari\_Secondari\_e\_Gestionali\_del\_Sangue |
| AMA | Agenti\_Metabolici/Antidiabetici |
| AIA | Agenti\_Immunomodulatori\_e\_Antinfiammatori |
| AAA | Agenti\_Anti-infezione\_e\_Antiallergici |
| AS | Altri\_Sistemi\_(pelle,\_occhi,\_sistema\_nervoso,\_ecc.) |
| AAD | Altri\_Agenti\_Diversi |
| BPCO | Broncopneumopatia\_Cronica\_Ostruttiva |
| DIABETE | Diabete\_Mellito |
| DM1 | Diabete\_Mellito\_tipo1 |
| IPERTENSIONE | Ipertensione\_Arteriosa |
| IPOLIPEMIZZANTI | Agenti\_Ipolipemizzanti |
| SCC | Scompenso\_cardiaco |

Vedi DIZIONARIO FARMACEUTICA: [farma\_aggregata](https://docs.google.com/spreadsheets/u/0/d/1GJGH_qW0zq_sms_itQ0cbGHTWIISMKmpa67tO6bDUss/edit)

la profondita’ di queste 13 tabelle e’ massima \[1995\], non ci sono filtri.   
Le tabelle sono in EGTASK con il nome standard DSFARMA\_\[ACRONIMO\]   
In output le tabelle L1 hanno queste variabili

1. **KEY\_ANAGRAFE**: Identificativo univoco (anagrafico) del paziente o dell’assistito.  
    *Tipo: num*  
2. **data\_prestazione**: Data in cui viene erogata la prestazione (farmaceutica).  
    *Tipo: date (a seconda della gestione)*  
3. **CLASSE**: Classe farmacologica (ad es. “BETABLOCCANTI”).  
    *Tipo: char*  
4. **farmaco**: Nome del farmaco (commerciale o principio attivo).  
    *Tipo: char*  
5. **FARMACO\_ATC\_COD**: Codice ATC del farmaco (classificazione anatomico-terapeutico-chimica).  
    *Tipo: char*  
6. **FARMAPRESCR\_PEZZI**: Il numero di pezzi comprati.  
    *Tipo: num*  
7. **costo**: Costo del farmaco, espresso in Euro.  
    *Tipo: num*  
8. **FARMACI\_DDD\_GIORNI**: Defined Daily Dose.  
    *Tipo: char*  
9. **FARMACI\_DESC**: Descrizione estesa del farmaco (denominazione completa).  
    *Tipo: char*  
10. **FARMACI\_SOSTANZA:** Altro codice o classificazione di supporto, solitamente interna.  
     *Tipo: char*  
11. **copertura**: Giorni di copertura della prescrizione. Calcolata \= Numero pezzi x DDD.  
     *Tipo: num*  
12. **anno\_prestazione**: Anno di erogazione del farmaco.  
     *Tipo: num*

La singola tabella di livello L1 e’ generata eseguendo da console il vbs seguito dal progetto , che e’ sempre lo stesso, e poi dalla variabile che descrive quale macroclasse vogliamo costruire.

# DATAMART

# SCORES

# score2op

# Charlson

# ESC

# DIAGNOSI

# Aggregate

# IRC

# DIABETE

# SCOMPENSO

# BPCO

**Merge Iniziale (temp\_farma01):**  
 I dati provenienti da *egtask.dsfarma\_bpco* e dalla coorte (\&coorte.) vengono uniti per creare `temp_farma01`.

**Elaborazione per Anno:**  
 Da `temp_farma01` si genera `temp_anno00` che viene trasposto in due dataset (date e quantità) e successivamente riunito in `temp_anno01`. Dopo il calcolo dei contatori e la definizione di `limit_anno`, si calcola `farma_anno`.

**Elaborazione per Classe:**  
 Parallelamente, `temp_classe00` viene processato con trasposizioni, merge e calcoli dei contatori (macro `limit_classe`) per ottenere `farma_classe`.

**Altri Rami (Diagnosi, Esenzioni, SDO):**

* `dia_bpcodadiag` raccoglie le informazioni diagnostiche da *EGTASK.DIAGNOSI\_AGGREGATE*.

* Le esenzioni vengono gestite in due flussi: uno che produce `dia_esenti` e l’altro `dia_esclusi`.

* I dati SDO vengono elaborati per ottenere `dia_sdo493`.

**Merge Finale:**  
 Tutte le informazioni (coorte, farma\_anno, farma\_classe, diagnostica, esenzioni, SDO) vengono unite nel dataset finale (integrata\_bpco) che contiene la data minima tra le fonti e la variabile che indica da quale fonte proviene la data.

Di seguito trovi una panoramica dello schema dei dataset che vengono creati nel codice e il modo in cui sono collegati. L'idea è che il codice parta da alcune fonti dati (come la tabella delle prescrizioni farmaceutiche, la coorte, le diagnosi, le esenzioni e i codici SDO) e, mediante trasformazioni e transposizioni, generi in parallelo due rami di elaborazione (per “anno” e per “classe”) che poi vengono combinati con le altre fonti per produrre il dataset finale integrato.

Di seguito un riassunto dei principali passaggi e dataset:

1. **Merge iniziale (temp\_farma01):**

   * **Input:**

     * *egtask.dsfarma\_bpco* (con filtri su `classe` ≠ 'OSSIGENO')

     * *\&coorte.* (contenente almeno `key_anagrafe` e `data_indice`)

   * **Output:** `temp_farma01`

2. **Branch elaborazione “anno”:**

   * **Sorting:**

     * `temp_farma01` → `temp_anno00` (ordinato per key\_anagrafe, data\_indice, data\_prescrizione)

   * **Transposizioni:**

     * `temp_anno00` → `temp_anno01d` (trasposizione di *data\_prescrizione* con prefisso `dacq`)

     * `temp_anno00` → `temp_anno01c` (trasposizione di *conf* con prefisso `acq`)

   * **Merge e contatori:**

     * Unione in `temp_anno01` e calcolo di conteggi/statistiche in `temp_counter_anno01` e `temp_counter_anno02` (definisce la macro variabile `limit_anno`)

   * **Elaborazione finale:**

     * Con i dati transposti e la macro `limit_anno`, viene calcolata la data d'inizio dell’acquisto in `farma_anno` (se la somma di acquisti raggiunge almeno 5 in un intervallo)

3. **Branch elaborazione “classe”:**

   * **Sorting:**

     * `temp_farma01` → `temp_classe00` (ordinato per key\_anagrafe, data\_indice, classe, data\_prescrizione)

   * **Transposizioni:**

     * `temp_classe00` → `temp_classe01d` (trasposizione di *data\_prescrizione* per ciascuna classe)

     * `temp_classe00` → `temp_classe01c` (trasposizione di *conf*)

   * **Merge e contatori:**

     * Unione in `temp_classe01` e calcolo dei contatori in `temp_counter_classe01` e `temp_counter_classe02` (definisce la macro variabile `limit_classe`)

   * **Elaborazione finale:**

     * Viene elaborato `temp_classe02` con il loop sui dati transposti per definire la data d'inizio d’acquisto per classe, che viene sintetizzata in `farma_classe` (se la somma raggiunge almeno 3\)

4. **Elaborazioni complementari:**

   * **Diagnosi:**

     * `dia_bpcodadiag` è creato da *EGTASK.DIAGNOSI\_AGGREGATE* (filtrando le diagnosi BPCO) unite alla coorte

   * **Esenzioni:**

     * Dalle tabelle *DWTSISSR.VFS\_ASSISTITI\_ESENTI* e *VDIZIONARIO\_ESENZIONI* si crea `temp_esenti00`

     * Successivamente, in base al codice di esenzione, si dividono in:

       * `temp_esenti01` → da cui si ottiene `dia_esenti` (la prima data d'esenzione per ciascun soggetto)

       * `temp_esclusi01` → da cui si ottiene `dia_esclusi` (soggetti da escludere)

   * **Codici SDO:**

     * Vengono selezionati i codici ICD9 relativi all’asma da *DIZ.DIZIONARIO\_BPCOINT* (in `temp_squery`)

     * Si esegue il merge di *egtask.sdo\_generica* e la coorte in `temp_sdo01`

     * Tramite una macro (%unica) viene generato `temp_sdo_single` che viene poi sintetizzato in `dia_sdo493` (se la condizione ICD9\_CM\_ASMA è soddisfatta)

5. **Dataset finale integrato:**

   * Il dataset finale (chiamato in base alla macro variabile \&dsoutput, ad esempio *temp\_integrata\_bpco* o *libout.\&nome.\_integrata\_bpco*) è ottenuto con il merge dei seguenti dataset:

     * **Coorte (\&coorte.)**

     * **farma\_classe**

     * **farma\_anno**

     * **dia\_esenti**

     * **dia\_esclusi**

     * **dia\_sdo493**

     * **dia\_BPCOdaDIAG** (che corrisponde a `dia_bpcodadiag`)

   * Viene calcolata la variabile `data_integrata_bpco` come il minimo tra le date provenienti da diagnosi, farma\_anno, farma\_classe ed esenzione, e in base a questa viene assegnata la variabile `integrata_bpco` e la provenienza (`bpco_from`).

# COVID

# FA

# IPERTENSIONE

# DISLIPIDEMIA

# ANEMIA

# Rischio CVMA

# ipercol familiare

# microalbuminuria

# obesità

# dialisi

dialisi

# ESAMI

# LABORATORIO

La tabella esami e’ una tabella formalmente di tipo L1 poiche’ derivazione diretta da tabelle L0, ma ci sono alcune tabelle costruite da varie operazioni di aggregazione sia tra diversi esami del DNLAB si acon esami estratti dal laboratorio di carionet. Ci sono in tutto 38\.

acr aer pcr per bnp probnp creatinina emoglobina calcio lipoproteina A ldl hdl tri col protu urine .

Gran Parte di essi e’ una semplice aggregazione di diversi codici di laboratorio in una sola categoria, per l’appunto l’esame. Alcuni subiscono un post processing semplice come la conversione delle unita’ di misura per avere un dataset omogeneo. Alcuni invece sono processati in piu’ fasi. 

Alcuni sono gia’ aggregati con gli esami estratti dal laboratorio di Cardionet.

# ECO ECG

# spirometrie

# fenotipo

# par. funzionali

# EVENTI

# Male

# Mace 3p 5p

# eventi ricovero

# emo maggiori

# PRESTAZIONI

# CUP altro

# CUP ecg

# CUP eco

# CUO ecovasco

# CUP ecocardio

# CUP tutte

# CUP spiro

# CUP visite

# prest Cardionet

# pronto soccorso

# TERAPIA

# farmaceutica terr.

## protocollo CLINICO

filtro date: t2.data\_indice \- 90 \<= t1.data\_prescrizione \<= t2.data\_indice \+ 180  
in output:

- [ ] potenza ipolipemizzanti \[compresi pcsk9 e inclisiran\]:  
      - [ ] indice  
      - [ ] fup \[il fup e’ il piu’ vicino all’LDL preso ad 1 anno dalla data indice, se non c’e’, si prende la potenza piu’ vicina a 12 mesi dalla data indice.  
- [ ] somma 6mesi, 12 mesi delle prescrizioni per persona, data\_indice e classe.  
- [ ] 0/1 classe fup 6 mesi   
- [ ] 0/1 classe anam 3 mesi

## protocollo PDTA

filtro date: (t2.data\_uscita\_indice\<=t1.data\_prescrizione\<=t2.data\_uscita\_indice+365\*2)  
in ouput:

- [ ] somma della copertura nell’anno per classe  
- [ ] colonne 0/1 delle classi  
- [ ] last\_AA, last\_ASA, last\_NAOTAO  
- [ ] somma atc su key\_anagrafe e data\_indice

## protocollo EPI4M

filtro date: (t2.data\_indice\<=t1.data\_prescrizione\<=t2.data\_indice+365)  
in output:

- [ ] potenza ipolipemizzanti;   
      - [ ] indice \[WHERE (t1.data\_indice \- 365 \< t1.data\_prescrizione \<= t1.data\_indice)\]   
      - [ ] fup \[(t2.data\_indice \+ 30\*9 \<= t2.data\_prescrizione \<= t2.data\_indice \+ 30\*12 )\]  
- [ ] Somma grezza degli ATC prescritti aggregati su key\_anagrafe

# cardio farma

[^1]:  Questi flussi sono accessibili solo da profilo SAS Aziendale.

[^2]:  PNLAB e’ il laboratorio di Pordenone che e’ stato importato nel DNLAB dal 2017\. Prima del 2017 il DNLAB non ha Pordenone.

[^3]:  AMBULATORIALE LV3 fa riferimento solo alla integrata DIALISI.