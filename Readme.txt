MongoDB è un database
Humongous --> si posso immagazinare molti dati e recuperarli velocemnte
E' un db server che permette di creare diversi database
I database contengono le collections
Nelle collections ci sono i documenti
Ogni documento è un file json
Non c'è lo schema. Ogni documento può avere diverse chiavi
I file Json, come al solito sono liste di chiavi-value
Il server salva i Json in Bson, che è la versione serializzata
E' efficiente quando i dati che normalmente andrebero creati usando una join, stanno tutti in un unico documento

Generalmente abbiamo meno collection. Perchè i documenti contengono le ralazioni
C'è anche la versione mobile per salvare i dati offline
Altri strumenti:
Stich: a grandi linea è un tool per interrogare MongoDB con delle query API
Funzioni Serverless: come le lambda
database triggers: come dice il nome, dei trigger che scattano quando succede qualcosa sul db e permettono di svolgere delle operazioni
Real-time Sync: per sincronizzare il db mobile

Per installare la mongosh sul Mac, scaricare la versione compressa dal sito,
    estrarre l'archivio, prendere i due file che ci sono e meerli insieme al resto
    di mongo usr/local/bin

Per vedre le cartelle nascoste. cmd+shft+.

I database e le collection sono create automaticamente quando iniziamo a inserire i documenti

Quando inserisco un documento è possibile specificare il _id, l'importante è che siano univoci.
L'id avrà il tipo specificato nell'inserimento. Nellìesempio qui sotto è una stringa. Es:
db.flightData.InsertOne({_id: "txl-lhr-1", departureAirport: "TXL", arrivalAirpot: "LHR"})

Create: insertOne(data, options), insertMany(data, options)
    per insertMany va passato un array

Read: find(filter, options), findOne(filter, options)
    db.flightData.find(intercontinental: true)
    db.flightData.find({distace: {$gt: 10000}})

    in realtà find ritorna un cursore, per evitare di ritornare troppi dati
    si può usare poi il cursore per ciclare sui dati. Es:

    db.passengers.find().forEach( (passData)=> {printjson(passData)}) -- sembra anche che forEach sia efficiente

Update: updateOne(filter, data, options), updateMany(filter, data, options). Es:
    db.flightData.updateOne({distance: 12000, {$set: {marker: "delete"}}}) -- mette la proprietà delete la documento che ha distamce a 12000
    db.flightData.updateMany({}, {$set: {marker: "toDelete"}}) -- cambia il campo marker di tutti i documenti
    db.flightData.update({_id: ObjectId("613f0fc764aff84e18a43fd3")}, {delayed: false}) -- sostituisce l'intero documento. Ma è più chiaro usare replace

    db.flightData.replaceOne({_id: ObjectId("613f0fc764aff84e18a43fd3")},  {
        "departureAirport": "MUC",
        "arrivalAirport": "SFO",
        "aircraft": "Airbus A380",
        "distance": 12000,
        "intercontinental": true
    })

Delete: deleteOne(filter, options), deleteMany(filter, options). Es
    db.flightData.deleteMany({}) -- cancella tutto
    db.flightData.deleteOne({marker: "toDelete"})

Projection (applicate server side. _id è sempre incluso di default. Se lo si vuole escludere va fatto _id: 0 )
    db.passengers.find({}, {name: 1}) -- significa nei dati estratti includi name 

Embedded (nested) Documents sono documenti inclusi in altri documenti. Si possono avere fino a
100 livelli (inutile dire che è eccessivo) e il documento con tutti i suoi sotto documenti
può pesare 16MB max
    db.flightData.updateMany({}, {$set: {status: {description: "on-time", lastUpdated: "1 hour ago"}}})

    db.flightData.find({"status.description": "on-time"})


Inserire un array:
    db.passengers.updateOne({_id: ObjectId("613f18c764aff84e18a43fe9")}, {$set: {hobbies: ["sports", "cooking"]}})

Accedere a un array
    db.passengers.findOne({_id: ObjectId("613f18c764aff84e18a43fe9")}).hobbies

Recuperare i documenti che hanno un certo valore in un certo array:
    db.passengers.find({hobbies: "sports"}) -- trova il singolo valore che nell'array hobbies ha il valore sport


Cancellare un database
    use databaseName
    db.dropDatabase()

Cancellare una collection
    db.myCollection.drop()

Schemas & Data Modelling
Di per se MongoDb non ha lo schema, ma dal punto di vista logico i documenti in una collection
dovranno avere lo stesso schema. Lo scenario più probabile è che i documenti abbiano le stesse chiavi, 
al massimo alcune chiavi mancano.
Se vogliamo avere la chiave, ma senza valore:
    db.products.insertOne({name: "A book", price: 12.99, details: null})

Anche se lo stile di MongoDB sarebbe quello di non avere details

Tipi di dati

Text. Es: "max". Non ha limiti, se non il fatto che un documento arriva al massimo a 16MB
Boolean. soliti true o false
Number. Ha 3 sotto tipi :
    NumberInt (Int32), NumberLong(Long), Double (64 bit, usato di default dalla shell. se il mio numero è più grande c'è overflow), NumberDecimal (floating point a alta precisione)
ObjectId. Una stringa con un elemento temporale. Oggetti creati uno dopo l'altro, avranno gli id in sequenza
ISODate. 2018-09-09
Timestamp. Usato internamente. Due documenti non hanno lo stesso timestamp
Embedded Document
Array 

es:
    db.companies.insertOne({name: "Fresh Apples Inc.", isStartup: true, employees: 33, funding: 12345678901234567890, details: {ceo: "Mark Super"}, tags: [{title: "super"}, {title: "perfetct"}], fundingDate: new Date(), insertedAt: new Timestamp()})


Data Schemas & Data Modelling
Which Data does my App need or generate? --> User Information, Product, Information, Orders, .... --> Define the Fields you'll need (and how they relate)
Where do I need my Data --> Welcome Page, Products List Page, Order Page --> Defines your required collections + fields grouping
Which kind  of Data or Information do I want to display? --> Welcome Page, Products List Page, Order Page --> Defines which queries you'll need
How often do I fetch my data --> For every page reload --> Defines whether you should optimize for easy fetching (duplicare le collection per aiutare la ricerca)
How often do I write or change my data? --> Order ==> Often, Product Data ==> Rarely ==> non duplicare le collection
In pratica i dati vanno immagazzinati così come vanno fatti vedere nella App
Evitare le join

Relazioni
Abbiamo la possibilità di salvare la relazione come Embedded Document, oppure come reference
Embedded Document: con una sola query ho tutti i dati. Ma ho duplicazione, e se cambio qualcosa relativo al documento Embedded, potrei doverli cercare tutti e aggiornarli
Reference: mi servono più query per ottenere il documento finale, ma non ho ridondanza di dati

