# Aufgabe: Kafka Cluster mit drei Brokern ohne Zookeeper (KRaft)

## Ziel
Erstelle einen Kafka-Cluster mit drei Brokern auf einer Maschine. Nutze die KRaft-Architektur.

---

## Hinweise & Schritte

### 1. **Vorbereitung**
- Stelle sicher, dass Java 11 (oder höher) installiert ist.  
  → `java -version` prüfen. Falls nicht vorhanden, JDK installieren.
- Lade Kafka herunter (mind. Version 3.0):  
  [https://kafka.apache.org/downloads](https://kafka.apache.org/downloads)

---

### 2. **Verzeichnisstruktur**
- Entpacke Kafka in ein Verzeichnis, z. B. `C:\kafka`.
- Erstelle pro Broker ein separates Datenverzeichnis:
    - `C:\kafka\data\broker1`
    - `C:\kafka\data\broker2`
    - `C:\kafka\data\broker3`

---

### 3. **Konfiguration der Broker**
- Öffne die Datei `server.properties` im `config`-Ordner.
- Erstelle für jeden Broker eine separate Kopie:
    - `server-broker1.properties`
    - `server-broker2.properties`
    - `server-broker3.properties`

#### Wichtige Konfigurationen pro Broker
Bearbeite folgende Parameter individuell pro Broker:

- **Allgemeine Einstellungen:**
  ```properties
  broker.id=1  # Für Broker 2: 2, Für Broker 3: 3
  log.dirs=C:\kafka\data\broker1  # Broker 2: broker2, Broker 3: broker3
  listeners=PLAINTEXT://localhost:9092  # Broker 2: 9093, Broker 3: 9094
  inter.broker.listener.name=PLAINTEXT
  listener.security.protocol.map=PLAINTEXT:PLAINTEXT,CONTROLLER:PLAINTEXT
  process.roles=broker,controller  # Nur für Broker 1
  controller.listener.names=CONTROLLER
  num.network.threads=3
  num.io.threads=8
  log.retention.hours=168
  log.segment.bytes=1073741824
  log.retention.check.interval.ms=300000
  ```

- **Controller-Quorum:** (Identisch für alle Broker)
  ```properties
  controller.quorum.voters=1@localhost:9092,2@localhost:9093,3@localhost:9094
  ```

- **Broker 2 und Broker 3:** Ändere `process.roles` auf:
  ```properties
  process.roles=broker
  ```

---

### 4. **Cluster-ID erstellen**
1. Generiere eine Cluster-ID (nur einmal ausführen):
   ```bash
   .\bin\windows\kafka-storage.bat random-uuid
   ```
   Merke dir die generierte ID, z. B.: `P6XBz1RPRg2mbAhZ4e4O2g`.

2. Initialisiere jeden Broker mit der Cluster-ID:
   ```bash
   .\bin\windows\kafka-storage.bat format -t <CLUSTER_ID> -c .\config\server-broker1.properties
   .\bin\windows\kafka-storage.bat format -t <CLUSTER_ID> -c .\config\server-broker2.properties
   .\bin\windows\kafka-storage.bat format -t <CLUSTER_ID> -c .\config\server-broker3.properties
   ```

---

### 5. **Broker starten**
Starte die Broker nacheinander in separaten Terminals:
```bash
.\bin\windows\kafka-server-start.bat .\config\server-broker1.properties
.\bin\windows\kafka-server-start.bat .\config\server-broker2.properties
.\bin\windows\kafka-server-start.bat .\config\server-broker3.properties
```

Prüfe, ob alle Broker laufen:
```bash
tasklist | findstr kafka
```

---

### 6. **Cluster testen**
1. Erstelle ein Test-Topic:
   ```bash
   .\bin\windows\kafka-topics.bat --create --topic test-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
   ```

2. Liste die Topics auf:
   ```bash
   .\bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092
   ```

3. Sende und empfange Nachrichten:
    - Producer starten:
      ```bash
      .\bin\windows\kafka-console-producer.bat --topic test-topic --bootstrap-server localhost:9092
      ```
    - Consumer starten:
      ```bash
      .\bin\windows\kafka-console-consumer.bat --topic test-topic --from-beginning --bootstrap-server localhost:9092
      ```
