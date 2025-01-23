# Aufgabe: mTLS für Kafka-Broker einrichten

## Ziel
Richte mTLS (mutual TLS) für deinen Kafka-Cluster ein, um sichere Kommunikation zwischen Clients und Brokern sowie zwischen den Brokern selbst zu gewährleisten. Du hast bereits folgende Ressourcen:

- Keystores:
    - `kafka-1.jks`
    - `kafka-2.jks`
    - `kafka-3.jks`
- Truststore:
    - `root-ca.jks`

---

## Anforderungen
- **Keystores und Truststore** müssen auf den jeweiligen Broker-Servern verfügbar sein.
- Kafka muss so konfiguriert werden, dass es TLS für interne und externe Kommunikation nutzt.

---

## Schritte

### 1. **Keystore und Truststore auf Broker kopieren**
1. Stelle sicher, dass die Keystores (`kafka-1.jks`, `kafka-2.jks`, `kafka-3.jks`) und der Truststore (`root-ca.jks`) auf den jeweiligen Broker-Servern verfügbar sind.
2. Kopiere die Dateien z. B. in das Verzeichnis `/opt/kafka/config/ssl/` auf jedem Broker.

---

### 2. **Kafka-Server konfigurieren**
Bearbeite die `server.properties`-Datei für jeden Broker.

#### Allgemeine TLS-Einstellungen
Füge die folgenden Konfigurationsoptionen hinzu oder passe sie an:

```properties
# Broker-ID (ändern für jeden Broker)
broker.id=1

# Listener für externe und interne Kommunikation
listeners=SSL://localhost:9092
inter.broker.listener.name=SSL

# SSL Konfiguration für Broker
ssl.keystore.location=/opt/kafka/config/ssl/kafka-1.jks
ssl.keystore.password=<KESTORE_PASSWORT>
ssl.key.password=<KEY_PASSWORT>
ssl.truststore.location=/opt/kafka/config/ssl/root-ca.jks
ssl.truststore.password=<TRUSTSTORE_PASSWORT>
ssl.client.auth=required
```

#### Broker-Spezifische Anpassungen
Für Broker 2 und 3:
- Ändere `broker.id` entsprechend (`2` für Broker 2, `3` für Broker 3).
- Passe den Pfad des Keystores an (`kafka-2.jks` und `kafka-3.jks`).

---

### 3. **Client-Konfiguration anpassen**
Clients müssen ebenfalls für TLS konfiguriert werden, um mit den Brokern zu kommunizieren.

#### Beispiel-Client-Eigenschaften (z. B. `client.properties`):

```properties
security.protocol=SSL
ssl.keystore.location=/path/to/client-keystore.jks
ssl.keystore.password=<KESTORE_PASSWORT>
ssl.key.password=<KEY_PASSWORT>
ssl.truststore.location=/path/to/root-ca.jks
ssl.truststore.password=<TRUSTSTORE_PASSWORT>
```

---

### 4. **Broker starten**
Starte jeden Broker mit den aktualisierten Konfigurationen:
```bash
bin/kafka-server-start.sh config/server.properties
```

---

### 5. **Testen der TLS-Verbindung**

#### 5.1 **Überprüfung der Broker**
- Stelle sicher, dass alle Broker korrekt gestartet wurden.
- Prüfe die Logs, um sicherzustellen, dass keine SSL-Fehler auftreten:
  ```bash
  cat logs/server.log | grep SSL
  ```

#### 5.2 **Verbindung testen (Client zu Broker)**
- Verwende die Kafka-Tools, um eine Verbindung zu testen:

1. Erstelle ein Test-Topic:
   ```bash
   bin/kafka-topics.sh --create --topic test-topic --bootstrap-server localhost:9092 --command-config client.properties
   ```

2. Sende Nachrichten:
   ```bash
   bin/kafka-console-producer.sh --topic test-topic --bootstrap-server localhost:9092 --producer.config client.properties
   ```

3. Empfange Nachrichten:
   ```bash
   bin/kafka-console-consumer.sh --topic test-topic --bootstrap-server localhost:9092 --consumer.config client.properties --from-beginning
   ```
