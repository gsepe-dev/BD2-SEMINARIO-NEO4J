# Neo4j - Seminario BD2

## 1. Installazione

Per le istruzioni complete di installazione fare riferimento alla documentazione ufficiale:

[https://neo4j.com/docs/operations-manual/current/installation/](https://neo4j.com/docs/operations-manual/current/installation/)

## 2. Avvio del server
```bash
sudo neoj4 start
```

Una volta avviato, l'interfaccia web è disponibile all'indirizzo:

[http://localhost:7474](http://localhost:7474)

## 3. Nodi

### 3.1 Creazione

- Nodo semplice
```
CREATE (sample);
```

- Nodi multipli
```
CREATE (sample1), (sample2);
```

- Nodo con etichetta
```
CREATE (sample:player);
```

- Nodo con più etichette
```
CREATE (sample:person:player);
```

- Nodo con proprietà
```
CREATE (Dhawan:player {name: "Shikar Dhawan", YOB: 1985, POB: "Delhi"});
```

### 3.2 Recupero
```
MATCH (n) RETURN n;
```

### 3.3 Modifica

- Aggiungere una proprietà
```
MATCH (Dhawan:player {name: "Shikar Dhawan", YOB: 1985, POB: "Delhi"}) 
SET Dhawan.highestscore = 187 
RETURN Dhawan;
```

- Cambiare proprietà esistenti
```
MATCH (Dhawan:player {name: "Shikar Dhawan", YOB: 1985, POB: "Delhi"}) 
SET Dhawan.name = "Rayindra Jadela" 
RETURN Dhawan;
```

- Aggiungere etichette
```
MATCH (Anderson {name: "James Anderson", YOB: 1982, POB: "Burnely"}) 
SET Anderson:player 
RETURN Anderson;
```

### 3.4 Cancellazione

- Nodo specifico
```
MATCH (Ishant:player {name: "Ishant Sharma", YOB: 1988, POB: "Delhi"})
DELETE Ishant;
```

- Proprietà
```
MATCH (Dhoni:player {name: "Mahendra Dhoni", YOB: 1981, POB: "Ranchi"})  
REMOVE Dhoni.POB 
RETURN Dhoni;
```

- Etichetta
```
MATCH (Dhoni:player {name: "Mahendra Dhoni", YOB: 1981}) 
REMOVE Dhoni:player 
RETURN Dhoni;
```

- Tutti i nodi e relazioni
```
MATCH (n) DETACH DELETE n;
```

Attenzione: per grandi volumi di dati si consiglia di cancellare direttamente tutto il database cancellando il file **data/graph.db**

## 4. Relazioni

### 4.1 Creazione

- Relazione diretta
```
CREATE (Dhawan:player)-[:TOP_SCORER_OF]->(Ind:country {name: "India"});
```

- Relazione con MERGE
```
MATCH (a:country {name: "India"})
MATCH (b:tournament {name: "Champions Trophy 2019"})
MERGE (a)-[r:WINNERS_OF]->(b)
RETURN a, r, b;
```

### 4.2 Gestione proprietà con MERGE
```
MERGE (Jadeja:player {name: "Ravindra Jadeja", YOB: 1988, POB: "NavagamGhed"})   
    ON CREATE SET Jadeja.isCreated = "true"
    ON MATCH SET Jadeja.isFound = "true"
RETURN Jadeja;
```

## 5. Percorsi e FOREACH

- Creare un percorso
```
CREATE p = (Dhawan:player {name:"Shikar Dhawan"})-[:TOPSCORER_OF]->
   (Ind:country {name: "India"})-[:WINNER_OF]->
   (CT2019:tournament {name: "Champions Trophy 2019"}) 
RETURN p;
```

- Aggiornare tutti i nodi di un percorso
```
MATCH p = (Dhawan)-[*]->(CT2019) 
WHERE Dhawan.name = "Shikar Dhawan" AND CT2019.name = "Champions Trophy 2019" 
FOREACH (n IN nodes(p) | SET n.marked = TRUE);
```

## 6. Query e Filtri

### 6.1 MATCH

- Generale
```
MATCH (n) RETURN n;
MATCH (n:player {name: "Shikar Dhawan"}) RETURN n;
```

- Tutti i nodi associati a un nodo
```
MATCH (Ind:country {name:"India"}) -- (n)
RETURN n.name;
```

- Tutti i nodi associati a un nodo in base alla direzione
```
MATCH (Ind:country {name:"India"}) --> (n)
RETURN n;
```

```
MATCH (Ind:country {name:"India"}) <-- (n)
RETURN n;
```

### 6.2 OPTIONAL MATCH
```
MATCH (a:tournament {name: "Champions Trophy 2019"}) 
OPTIONAL MATCH (a)-->(x) 
RETURN x;
```

### 6.3 Filtrare nodi tramite relazioni
```
MATCH (n) 
WHERE (n)-[:TOP_SCORER_OF]->( {name: "India", result: "Winners"})
RETURN n;
```

## 7. Aggregazioni e Ordinamento

### 7.1 COUNT
```
MATCH (n {name: "India", result: "Winners"})--(x) 
RETURN n, count(*);
```

### 7.2 ORDER BY e LIMIT
```
MATCH (n:player)  
RETURN n.name, n.YOB 
ORDER BY n.YOB DESC
LIMIT 2;
```

### 7.3 SKIP
```
MATCH (n:player) 
RETURN n.name, n.YOB 
ORDER BY n.YOB DESC
SKIP 1 LIMIT 1;
```

### 7.4 WITH e COLLECT
```
MATCH (n:player) 
WITH n 
ORDER BY n.name DESC LIMIT 2 
RETURN collect(n.name)
```

## 8. Funzioni ausiliarie
```
MATCH (n:player) RETURN toUpper(n.name), n.YOB, n.POB;
MATCH (n:player) RETURN substring(n.name, 0, 5), n.YOB, n.POB;
MATCH (n:player) RETURN min(n.YOB);
```

## 9. Liste e Unione

### 9.1 UNWIND
```
UNWIND [1, 2, 3, null] AS x 
RETURN x, 'val' AS y;
```

### 9.2 UNION / UNION ALL
```
MATCH (n:country)
RETURN n.name AS name
UNION ALL
MATCH (n:player)
RETURN n.POB AS name;
```

## 10. Indici e Vincoli

- Creazione indice
```
CREATE INDEX player_name_yob_index
FOR (p:player)
ON (p.name, p.yob);
```

- Cancellazione indice
```
DROP INDEX player_name_yob_index;
```

- Mostrare tutti gli indici
```
SHOW INDEXES;
```

- Creazione vincolo di unicità
```
CREATE CONSTRAINT player_id_unique IF NOT EXISTS
FOR (n:player)
REQUIRE n.id IS UNIQUE;
```

- Cancellazione vincolo di unicità
```
DROP CONSTRAINT player_id_unique IF EXISTS;
```