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

### Correlazione tra i dati iniziali e l'indice vegetativo NDVI
- Ho letto con Pandas i risultati scaricati: usando un timeframe di 16 giorni per il periodo richiesto, ho ottenuto 24 date diverse per ogni coppia di longitudine e latitudine che avevo nel dataset di partenza (1962 punti), per un totale di 24 x 1962 = 47088 righe
- Ho notato che in realtà le date partivano dal 29/08/2021 e non dal 31/08/2021, probabilmente perchè è la data più vicina a quella richiesta tra tutte quelle in cui il satellite in questione ha acquisito i dati, ma non è rilevante ai fini del compito
- Ho filtrato i risultati in modo da tenere solo le colonne che mi interessavano (ID, Latitude, Longitude, Date e MOD13Q1_061__250m_16_days_NDVI)
- Ho raggruppato i dati usando la funzione groupby, in modo da ottenere un NDVI medio per ogni coppia di longitudine e latitudine, e rinominato alcune colonne
- Usando la funzione merge di Pandas, ho fatto una left join tra i dati originali e la nuova colonna ottenuta "NDVI medio", usando come chiave la coppia longitudine-latitudine
- Ho guardato se ci fosse correlazione lineare (correlazione di Pearson) tra NDVI medio e resa, scoprendo di no (-0.01)
- Ho cercato un'eventuale correlazione non lineare tra la variabile resa e le features NDVI medio, longitudine e latitudine: per farlo, ho cercato una relazione tra quelle variabili usando una random forest, ottenendo un MAPE (Mean Absolute Percentage Error) del 21.7% sul set di validazione e del 24.1% sul set di test
- Ho stampato un grafico che rappresenta la resa predetta dal modello in funzione della resa reale: sembra disporsi in modo simile a una retta inclinata a 45°, suggerendo che il modello prevede le varie rese correttamente (anche se con un errore percentuale medio intorno al 12% per il dataset totale)
- Avendo trovato una relazione, ciò potrebbe suggerire una correlazione (in questo caso non lineare) tra resa e NDVI (anche se con l'aggiunta delle features longitudine e latitudine)
- Guardando l'importanza delle caratteristiche della random forest, ho scoperto però che l'indice NDVI ha contribuito solo per un 2% a prevedere il valore di resa (contro un 71% della longitudine e un 27% della latitudine), confermando quindi che i dati del NDVI scaricati non sembrano essere correlati alla resa
- Il motivo potrebbe essere che una risoluzione spaziale di 250 metri non è sufficientemente precisa per quel terreno (dai dati raggruppati si nota infatti che tendono a ripetersi più volte gli stessi identici valori di NDVI al variare della coppia longitudine-latitudine, suggerendo che è necessaria una risoluzione spaziale migliore, ad esempio di 50 metri o 10 metri)

### Random forest
- Ho filtrato i dati in modo da tenere solo le colonne che mi interessavano (NDVI medio, longitudine, latitudine e resa)
- Ho suddiviso in X e y rispettivamente le colonne delle features (NDVI medio, longitudine e latitudine) e la colonna del target (resa)
- Ho lanciato una prima random forest con iperparametri arbitrari solo per farmi una prima idea delle performance
- Ho lanciato una grid search per provare tante combinazioni di iperparametri a mia scelta
- Ho usato 3 metriche di performance: MSE (Mean Squared Error), MAPE (Mean Absolute Percentage Error) e MAE (Mean Absolute Error)
- Ho utilizzato anche dei pesi per calcolare una media pesata di queste 3 metriche, scegliendo di dare più priorità al MAPE (con un peso di 5, contro un peso di 1 per gli altri due)
- Mi sono inventato un sistema automatizzato per creare una cartella per ogni grid search lanciata, salvando al suo interno i modelli migliori per ogni metrica (in un file pickle) e le varie stampe a video che si vedevano per quella particolare grid search (in un file .txt)
- Ho reso inoltre possibile "zoommare" un modello precedente per trovare iperparametri ancora migliori: è un metodo che ho inventato per trovare perlomeno un minimo locale migliore rispetto a quelli trovati precedentemente, anzichè limitarsi a una sola grid search o cambiare completamente gli iperparametri. Nel caso si voglia zoommare, bisogna specificare nelle variabili "zoom_cartella" e "zoom_modello" quali sono il numero della cartella e il numero del modello per cui avviene lo zoom; poi bisogna porre la variabile "zoom" uguale a "f' zoom_{zoom_cartella}_{zoom_modello}'" in modo che il nome della cartella cambi di conseguenza, e ovviamente bisogna cambiare manualmente gli iperparametri della grid search in modo da restringerli nell'intorno della migliore combinazione su cui si sta zoommando. Se non si tratta di uno zoom, basta lasciare vuota la stringa nella variabile "zoom"
- Ho anche aggiunto la possibilità di "completamento" di una grid search non terminata: è un altro metodo che ho inventato per evitare, in caso di spegnimento accidentale del PC o di impossibilità di proseguire con la grid search per motivi di tempo, di ricominciare tutto da capo. Nel caso si voglia riprendere una grid search interrotta, è sufficiente specificare nella variabile "numero_cartella_completamento" qual è il numero della cartella corrispondente alla grid search che si vuole terminare; nella variabile "completamento" bisogna mettere "f' completamento {numero_cartella_completamento}'" in modo che il nome della cartella cambi di conseguenza; nella variabile "iterazione_di_partenza" bisogna specificare da quale iterazione/combinazione bisogna partire (la prima è la numero 1, quindi, trattandosi di un completamento, sarà un numero maggiore di 1). Se non si tratta di un completamento, basta lasciare vuota la stringa nella variabile "completamento"

### Performance della random forest
- Ho definito una funzione per ottenere il percorso del file pickle corrispondente al modello che mi interessa usare a partire dal numero della cartella in cui si trova (numero_cartella_scelta) e dal numero del modello (numero_modello_scelto)
- Ho mostrato a scopo illustrativo come sono fatti alcuni alberi della random forest scelta
- Ho mostrato le performance del modello scelto sul training set, validation set, test set e sui dati dati totali, usando come metriche MSE, MAPE, MAE, R^2, media aritmetica (di MSE, MAPE e MAE) e media pesata (di MSE, MAPE e MAE)
- Ho mostrato l'importanza delle features del modello scelto
- Ho mostrato il grafico della resa predetta in funzione di quella reale, che idealmente sarebbe quindi una retta inclinata a 45° (che è ciò che avremmo se la resa predetta fosse sempre identica a quella reale)

### Possibili miglioramenti
- Si potrebbe utilizzare una rete neurale anzichè una random forest per vedere se le performance migliorano
- Si dovrebbe usare un satellite, o magari un drone, con una risoluzione spaziale migliore (meno di 250 metri) e magari anche una frequenza di campionamento migliore (meno di 16 giorni)
- In realtà non c'è un motivo preciso per cui la resa dovrebbe dipendere dai particolari valori di longitudine e latitudine in cui si effettua la raccolta; eventualmente dipende solo dall'indice vegetativo medio dell'anno precedente. Tuttavia ho aggiunto comunque quelle features a scopo illustrativo
- Nell'interpolazione spaziale, si possono testare altri metodi di interpolazione; inoltre, anzichè creare una griglia in un rettangolo, si può creare una griglia che stia tutta all'interno del dominio di longitudine e latitudine originale (che nel nostro caso è più simile a un triangolo rettangolo)
