# AWS RDS: Managed Database

Bisher habe ich Datenbanken selbst auf einem Server installiert und gewartet. Mit RDS übernimmt AWS die Wartung (Patches, Backups, Verfügbarkeit). Hier setze ich eine RDS-Instanz auf, importiere eine Test-Datenbank und vergleiche das mit der self-hosted Variante.

> Stellen mit `<...>` musst du mit deinen echten Werten ersetzen.

## 1. RDS-Instanz erstellen

In der AWS Console (Region **eu-central-1**):

1. RDS → **Create database** → **Standard create**
2. Engine: **MySQL**
3. Template: **Free tier**
4. Instance Class: **db.t3.micro**
5. Master username + Passwort setzen (notieren!)
6. Connectivity:
   - VPC: deine VPC aus Block 2.6
   - **Public access: No** (die DB soll nicht öffentlich erreichbar sein, sie liegt im private Subnet)
   - Security Group: eine SG, die Port **3306** von deiner EC2-Instanz erlaubt
7. **Create database** und warten, bis Status "Available" ist.

Den **Endpoint** findest du danach in den Details der Instanz (sieht aus wie `meine-db.xxxx.eu-central-1.rds.amazonaws.com`).

## 2. Test-Datenbank erstellen und exportieren

Auf der EC2 eine lokale DB als Quelle aufsetzen:


```bash
sudo apt install -y mariadb-server mysql-client

sudo mysql -e "CREATE DATABASE kundendb;"
sudo mysql kundendb -e "
CREATE TABLE kunden (id INT PRIMARY KEY, name VARCHAR(50), ort VARCHAR(50));
INSERT INTO kunden VALUES (1,'Muster AG','St. Gallen'),(2,'Beispiel GmbH','Zürich');
"
```

Mit `mysqldump` exportieren:

```bash
sudo mysqldump kundendb > kundendb.sql
```

## 3. Dump in RDS importieren

Von der EC2 aus (gleiche VPC) auf die RDS verbinden. Zuerst die Datenbank anlegen, dann den Dump einspielen:

```bash
mysql -h <RDS-ENDPOINT> -u <master-user> -p -e "CREATE DATABASE kundendb;"
mysql -h <RDS-ENDPOINT> -u <master-user> -p kundendb < kundendb.sql
```

Prüfen, ob die Daten da sind:

```bash
mysql -h <RDS-ENDPOINT> -u <master-user> -p -e "USE kundendb; SELECT * FROM kunden;"
```

Hier sollten die zwei Kunden-Zeilen erscheinen.

_(Output hier einfügen, sobald du es ausgeführt hast)_

## 4. Connection-String

```
Host:        <RDS-ENDPOINT>
Port:        3306
User:        <master-user>
Datenbank:   kundendb
```

## 5. Vergleich: Self-hosted vs. RDS

| Aufgabe | Self-hosted (MariaDB, Block 2.4) | RDS (Managed) |
|---------|----------------------------------|---------------|
| Patches / Updates | selbst machen | AWS |
| Backups | selbst einrichten | automatisch (AWS) |
| Failover / Hochverfügbarkeit | selbst aufbauen | AWS (Multi-AZ) |
| Monitoring | selbst | AWS (CloudWatch) |
| Schema / Tabellen / Queries | selbst | selbst |
| Security Groups / IAM | selbst | selbst |
| Kosten | nur der Server | höher (Aufpreis fürs Managen) |

**Was AWS managt:** Patches, Backups, Failover, Monitoring – also der ganze Betrieb.
**Was ich selbst mache:** Datenmodell, Queries, wer darf zugreifen (Security Groups, IAM).

**Wann lohnt sich was:**
- **Self-hosted:** wenn ich volle Kontrolle will, es günstig sein soll und ich die Wartung selbst übernehmen kann. Gut für kleine Setups.
- **RDS:** wenn ich mich nicht um Wartung und Backups kümmern will und Verfügbarkeit wichtig ist. Kostet mehr, spart aber Zeit und Risiko.

## Fazit

RDS nimmt mir den ganzen Wartungsaufwand ab. Dafür zahle ich mehr und gebe ein Stück Kontrolle ab. Für die Treuhandfirma mit wenig IT-Personal wäre RDS sinnvoll, wenn sie später eine Datenbank brauchen, weil sie sich dann nicht um Backups und Updates kümmern müssen.
