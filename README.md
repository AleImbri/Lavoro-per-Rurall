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
- Usando la funzione mgrid di Numpy, ho creato una griglia regolare di longitudini e latitudini (1000 punti x 1000 punti) che coprissero l'intera area di interesse (per farlo ho definito come estremi della griglia 4 vertici di un rettangolo, che in particolare sono i minimi e i massimi delle longitudini e latitudini che compaiono nel dataset)
- Ho usato la funzione griddata per interpolare i valori di resa sui punti della griglia: in particolare ho usato il metodo "nearest", che assegna a ogni punto della griglia il valore del punto originale più vicino
- Ho creato una heatmap usando la funzione imshow di Matplotlib: la heatmap visualizza i valori di resa come colori, con aree di resa più alta rappresentate da colori più scuri

### Download dei valori dell'indice vegetativo NDVI (Normalized Difference Vegetation Index)
- Dopo alcune ricerche online, ho trovato un sito delle NASA da cui scaricare i dati che mi interessavano: https://appeears.earthdatacloud.nasa.gov/task/point
- Tramite Python, ho esportato il dataset originale in un file .csv usando solo le colonne richieste dal sito (ID, Category, Latitude, Longitude)
- Ho caricato il file .csv sul sito, nell'apposita sezione
- Sempre sul sito, ho inserito il periodo per cui mi serviva sapere l'indice vegetativo: dato che serviva l'anno precedente al periodo di raccolta, ho scelto come periodo quello dal 31/08/2021 al 30/08/2022
- Sotto la dicitura "Select the layers to include in the sample" ho cercato "NDVI"
- Ho scelto la voce "Terra MODIS Vegetation Indices (NDVI & EVI)" perchè era quello con risoluzione migliore (250 metri anzichè 500 o 1000) e timeframe minore (16 giorni anzichè 30)
- Ho cliccato su "Submit" e atteso la mail per il download dei dati richiesti
- Tornando a Python, ho letto i risultati scaricati usando Pandas: usando un timeframe di 16 giorni per il periodo richiesto, ho ottenuto 24 date diverse per ogni coppia di longitudine e latitudine che avevo nel dataset di partenza (1962 punti), per un totale di 24 x 1962 = 47088 righe
- Ho notato che in realtà le date partivano dal 29/08/2021 e non dal 31/08/2021, probabilmente perchè è la data più vicina a quella richiesta tra tutte quelle in cui il satellite in questione ha acquisito i dati, ma non è rilevante ai fini del compito
- Ho filtrato i risultati in modo da tenere solo le colonne che mi interessavano (ID, Latitude, Longitude, Date e MOD13Q1_061__250m_16_days_NDVI)
- Ho raggruppato i dati usando la funzione groupby, in modo da ottenere un NDVI medio per ogni coppia di longitudine e latitudine, e rinominato alcune colonne
- Usando la funzione merge di Pandas, ho fatto una left join tra i dati originali e la nuova colonna ottenuta "NDVI medio", usando come chiave la coppia longitudine-latitudine
- Ho guardato se ci fosse correlazione lineare (correlazione di Pearson) tra NDVI medio e resa, scoprendo di no (-0.01)
- Ho guardato se sembrasse esserci un'eventuale correlazione non lineare tra la variabile resa e le features NDVI medio, longitudine e latitudine: per farlo, ho cercato la relazione matematica tra quelle variabili usando una random forest, e ottenendo un MAPE (Mean Absolute Percentage Error) intorno al 22%
- Ho stampato un grafico che rappresenta la resa predetta dal modello in funzione della resa reale: sembra disporsi in modo simile a una retta inclinata a 45°, suggerendo che il modello prevede le varie rese correttamente (anche se con un errore percentuale medio del 22%)
- Avendo trovato una relazione, ciò significa che sembra esserci anche una correlazione (in questo caso non lineare) tra resa e NDVI (anche se con l'aggiunta delle features longitudine e latitudine)


### Random forest
- 

### Possibili miglioramenti
