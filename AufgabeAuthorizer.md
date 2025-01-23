# Aufgabe: StandardAuthorizer für Kafka einrichten und ACLs testen

## Ziel
Richte den StandardAuthorizer für deinen Kafka-Cluster ein, konfiguriere ACLs (Access Control Lists) und teste die Zugriffssteuerung für einen Benutzer. Du hast bereits:

- **Keystores:**
    - `admin.keystore.jks` (für den Admin, der Superuser ist)
    - `user.keystore.jks` (für den Benutzer)
- **Broker-Kommunikation:** Broker untereinander sind ebenfalls Superuser.

---

## Anforderungen
- Kafka-Cluster mit mTLS (mutual TLS) aktiv.
- `ssl.principal.mapping.rules` für die Benutzer-Mapping-Regeln.

---

## Schritte

### 1. **StandardAuthorizer aktivieren**
Bearbeite die `server.properties`-Datei für jeden Broker und füge die folgenden Konfigurationen hinzu:

```properties
# Aktivieren des StandardAuthorizer
authorizer.class.name=org.apache.kafka.metadata.authorizer.StandardAuthorizer

# SSL Principal Mapping Rules
ssl.principal.mapping.rules=RULE:^CN=(.*?),.*$/$1/,DEFAULT

# Superuser-Konfiguration
super.users=User:kafka-1;User:kafka-2;User:kafka-3;User:admin

# ACL-Optionen
allow.everyone.if.no.acl.found=False

# Client-Authentifizierung für Controller-Listener
listener.name.controller.ssl.client.auth=required
```
- **`super.users`:** Enthält die Superuser (z. B. `admin` und Broker).
- **`ssl.principal.mapping.rules`:** Extrahiert die Benutzer-ID aus dem CN (Common Name) des Zertifikats.

> Hinweis: Passe die CN-Werte der Broker entsprechend deiner Umgebung an.

---

### 2. **Admin-Tool konfigurieren**
Erstelle eine Konfigurationsdatei für den Admin (`admin.properties`):

```properties
security.protocol=SSL
ssl.keystore.location=/path/to/admin.keystore.jks
ssl.keystore.password=<ADMIN_KEYSTORE_PASS>
ssl.key.password=<ADMIN_KEY_PASS>
ssl.truststore.location=/path/to/root-ca.jks
ssl.truststore.password=<TRUSTSTORE_PASS>
```

---

### 3. **Benutzer-ACLs konfigurieren**
Verwende das Kafka-Admin-Tool, um die ACLs für den Benutzer festzulegen.

#### 3.1 **ACL für Topic-Zugriff**
Erlaube dem Benutzer `user` das Lesen und Schreiben auf ein bestimmtes Topic (`test-topic`):

```bash
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --command-config admin.properties \
  --add \
  --allow-principal User:user \
  --operation Read \
  --operation Write \
  --topic test-topic
```

#### 3.2 **ACL für den Consumer-Group-Zugriff**
Erlaube dem Benutzer `user` den Zugriff auf eine Consumer-Group (`test-group`):

```bash
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --command-config admin.properties \
  --add \
  --allow-principal User:user \
  --operation Read \
  --group test-group
```

---

### 4. **Benutzer-Tool konfigurieren**
Erstelle eine Konfigurationsdatei für den Benutzer (`user.properties`):

```properties
security.protocol=SSL
ssl.keystore.location=/path/to/user.keystore.jks
ssl.keystore.password=<USER_KEYSTORE_PASS>
ssl.key.password=<USER_KEY_PASS>
ssl.truststore.location=/path/to/root-ca.jks
ssl.truststore.password=<TRUSTSTORE_PASS>
```

---

### 5. **Testen der ACLs**

#### 5.1 **Nachrichten senden (Producer)**
Verwende den Benutzer `user`, um Nachrichten an das Topic `test-topic` zu senden:

```bash
bin/kafka-console-producer.sh --bootstrap-server localhost:9092 \
  --producer.config user.properties \
  --topic test-topic
```

#### 5.2 **Nachrichten konsumieren (Consumer)**
Verwende den Benutzer `user`, um Nachrichten aus dem Topic `test-topic` zu lesen:

```bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
  --consumer.config user.properties \
  --topic test-topic \
  --group test-group \
  --from-beginning
```

#### 5.3 **Test mit fehlenden Berechtigungen**
- Versuche, mit einem anderen Benutzer oder auf ein anderes Topic zuzugreifen, um sicherzustellen, dass die ACLs korrekt funktionieren.
- Prüfe die Logs der Broker auf Zugriffsdaten.
