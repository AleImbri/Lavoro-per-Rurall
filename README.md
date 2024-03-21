### Lettura dei dati
- Ho usato la libreria Geopandas per leggere il dataset in input (resa_girasole_2022.gpkg)

### EDA
- Ho cambiato il tipo di alcune colonne in modo consono a ciò che rappresentano
- Ho riordinato il dataset dalla data più vecchia a quella più recente
- Ho analizzato alcune statistiche del dataset
- Ho riempito i dati mancanti usando la mediana dei valori della colonna in questione
- Ho notato che nel dataset comparivano 3 giorni di raccolta (31/08/2022, 03/09/2022, 05/09/2022), così ho deciso di mostrare alcuni grafici separatamente per ciascun gruppo
- Ho mostrato in particolare il grafico in funzione del tempo delle features che comparivano nel dataset (longitudine, latitudine, resa, velocità, area, umidità)
- Ho deciso poi di mostrare anche il percorso della macchina agricola (latitudine in funzione della longitudine) con la corrispondente resa per ciasun punto (usando una color map)
- Per quest'ultimo caso, ho deciso di mostrare anche il grafico del dataset totale oltre a quello dei 3 singoli gruppi

### Interpolazione spaziale dei dati
- 
