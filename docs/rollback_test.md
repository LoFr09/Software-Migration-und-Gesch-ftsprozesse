# Rollback-Test

Hier teste ich, ob ich nach einer misslungenen Migration wieder zurück komme. Das ist wichtig, weil der Kunde keinen Datenverlust riskieren darf.

## Sicherung vor der Migration

Bevor ich migriert habe, habe ich von der EC2 zwei Sicherungen gemacht:

- einen **Snapshot** von der Festplatte
- ein **AMI** (ein komplettes Abbild der ganzen Instanz)

Unterschied: Ein Snapshot ist nur die Festplatte. Ein AMI ist die ganze Maschine. Mit dem AMI kann ich eine neue Instanz starten, die genau gleich ist wie vorher.

## Problem simulieren

Damit ich den Rollback wirklich teste, baue ich absichtlich einen Fehler ein. Ich gebe der Buchhaltung aus Versehen Zugriff auf den Ordner Personal:

```bash
sudo setfacl -R -m g:buchhaltung:rwx /srv/samba/personal
```

Jetzt sieht die Buchhaltung Daten, die sie nicht sehen darf. Genau der Moment, wo man zurück will.

## Rollback machen

Ich starte einfach eine neue Instanz aus dem AMI von vorher:

1. EC2 → AMIs → mein Image wählen → Launch instance from AMI
2. Gleiche Einstellungen wie vorher
3. Starten und kurz prüfen

## Ergebnis

- **Dauer:** ca. 5 Minuten
- **Daten:** alle wieder da
- **Datenverlust:** keiner, der falsche Zugriff war weg

## Cloud vs. On-Premise

| | Cloud (AMI) | On-Premise (Backup) |
|--|------------|---------------------|
| Wie | neue Instanz aus AMI | Backup zurückspielen |
| Dauer | paar Minuten | oft Stunden |
| Aufwand | klein | gross |

Der Cloud-Rollback ist viel schneller und einfacher.

## Lessons Learned

- Sicherung **vor** der Migration machen, nicht erst wenn was kaputt ist.
- Ein AMI ist für einen kompletten Rollback praktischer als ein Snapshot.
- Der Test zeigt: mein Plan funktioniert, ich komme in Minuten zurück. Das gibt dem Kunden Sicherheit.
