# OCVTS — Documentazione del Repository

Manuale di dominio del sistema **OCVTS** (Osservatorio Cardiovascolare del Friuli Venezia
Giulia). Spiega **come**, a partire dai dati grezzi del Repository Epidemiologico Regionale
(RER), vengono costruite le variabili cliniche ed epidemiologiche usate negli studi.

## 📄 Documento

➡️ **[OCVTS.md](OCVTS.md)** — il documento completo (con sommario navigabile e diagrammi di
flusso). GitHub renderizza nativamente i diagrammi Mermaid.

## Com'è organizzato

Il documento è in tre parti, in ordine **top‑down** (dal generale al particolare):

1. **Introduzione** — cos'è il RER, le fonti, i livelli dati L0–L4 e i tre protocolli di
   studio (CLINICO, PDTA, EPI4M).
2. **Builders** — come dai dati grezzi **L0** si costruiscono le aggregazioni **L1 → L2**
   (SDO, C@rdioNet, diagnosi aggregate, esami di laboratorio, classi farmacologiche,
   dizionari).
3. **Datamart** — come le variabili L0–L4 diventano le **colonne finali della coorte**:
   diagnosi integrate, esami, eventi, prestazioni, score e terapia.

## Come leggere

- I dati sono raffinati in livelli, da **L0** (grezzo RER) a **L4** (classi di rischio).
- Ogni capitolo apre con un **diagramma di flusso** semplificato: tabelle colorate per
  livello, trasformazioni (esagoni) con i filtri, finestre temporali sugli archi.
- I nomi coorte‑specifici sono scritti senza il prefisso `&nome.` (es. `integrata_irc`).
- La nota **⚠︎ Limite** segnala i punti in cui la ricetta non è ricostruibile staticamente.

## Come è stato generato

Contenuto ricostruito **staticamente** dalla pipeline di lineage sugli EGP e sui codici SAS
del repository di produzione, **senza** accesso al server SAS. Formule, filtri, soglie e
liste di variabili sono estratti dal **codice reale**.
