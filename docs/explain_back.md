# Explain-Back

## Frage 1 – Warum auf Linux wechseln?

Drei Gründe: Kosten sparen (keine Windows-Lizenz und keine ESU-Gebühren mehr, Linux ist gratis), Sicherheit (der alte Windows Server bekommt bald keine Updates mehr, Linux schon) und Wartung (mit Samba merken die Mitarbeitenden keinen Unterschied).

Nachteil: Die Umstellung braucht Aufwand und Know-how, und dabei kann etwas schiefgehen (falsche Rechte, fehlende Dateien). Darum braucht es einen Rollback-Plan: Falls die Migration scheitert, kommt man schnell zurück auf den alten Stand, ohne Datenverlust.

## Frage 2 – Warum ist "kein Rollback nötig" gefährlich?

Weil man Fehler oft erst nach dem Go-Live merkt, wenn schon mit den Daten gearbeitet wird. Ohne Rollback gibt es dann keinen Weg zurück. Drei Szenarien:

1. Berechtigungen falsch übertragen jemand sieht Daten, die er nicht sehen darf.
2. Dateien mit Umlauten oder zu langen Pfaden werden nicht korrekt kopiert, einzelne fehlen.
3. Strom- oder Hardwareausfall mitten im Kopieren, Daten unvollständig oder beschädigt.

## Frage 3 – Was ist BPMN und warum modelliert man Prozesse?

BPMN zeichnet einen Geschäftsprozess als standardisiertes Diagramm. Vorteil gegenüber Text: Man sieht den Ablauf auf einen Blick, statt einen langen Text lesen zu müssen. Lanes zeigen, wer für welchen Schritt zuständig ist. Gateways zeigen Entscheidungen (z.B. "Berechtigung ok? Ja/Nein"). So gibt es weniger Missverständnisse und alle reden über den gleichen Ablauf.

## Frage 4 – Vorteile von RDS, was gibt man auf?

RDS nimmt einem die Wartung ab: Updates, Backups, Failover und Monitoring macht AWS. Das spart Zeit und erhöht die Verfügbarkeit. Dafür gibt man Kontrolle ab (kein Zugriff aufs Betriebssystem), macht sich von AWS abhängig und zahlt mehr als bei einer selbst gehosteten Datenbank.

---


# Selbsteinschätzung

A = sicher/selbstständig, B = machbar mit gelegentlicher Hilfe, C = unsicher in der Praxis, D = Thema noch nicht geläufig

| Bereich | Note (A–D) |
|---------|------------|
| BPMN-Modellierung (Lanes, Gateways, Datenobjekte) | B |
| Migrationsplanung (Ist/Soll, Risiken, Rollback) | B |
| Samba-Konfiguration und Linux-Berechtigungen | C |
| Daten-Migration (rsync, Berechtigungsprüfung) | C |
| Endbenutzer-Dokumentation (verständlich schreiben) | B |
| AWS EC2 Snapshots/AMI (Cloud-basierter Rollback) | B |
| AWS RDS (Managed Database, Import, Vergleich) | B |


---

# Schwierigkeiten 
Ich habe dieses Projekt als eher anspruchsvoll empfunden, da ich so etwas noch nie zuvor gemacht habe. Ich habe durch dieses Projekt gelernt, wie ich mit Netzwerkproblemen in einer VM umgehe und wie eine solche Migration richtig abläuft. Claud hat mich mit Hinweisen und bei der Textverbesserung sowie beim Troubleshooting unterstützt.
