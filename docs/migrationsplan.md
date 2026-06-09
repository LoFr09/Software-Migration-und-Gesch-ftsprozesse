# Migrationsplan: Windows-Fileserver → Linux/Samba

**Kunde:** Treuhandfirma, 18 Leute, ca. 800 GB
**Ziel:** Server von Windows auf Linux umstellen. Alles bleibt gleich für die Mitarbeitenden.

---

## a) Ist-Zustand

- Windows Server 2019, ESU läuft bald aus
- 4 Ordner: Mandate, Buchhaltung, Personal, Vorlagen
- ca. 800 GB Daten
- Rechte über Windows-Gruppen (NTFS)

**Berechtigungsmatrix:** `RW` = Lesen+Schreiben, `R` = Lesen, `✗` = kein Zugriff

| Ordner | Treuhänder | Buchhaltung | Personal | GL |
|--------|:----------:|:-----------:|:--------:|:--:|
| Mandate | RW | ✗ | ✗ | R |
| Buchhaltung | ✗ | RW | ✗ | R |
| Personal | ✗ | ✗ | RW | R |
| Vorlagen | R | R | R | R |

---

## b) Soll-Zustand

- Linux Server (Debian/Ubuntu) mit Samba
- Gleiche 4 Ordner, gleiche Rechte
- Rechte über Linux-Gruppen statt Windows-Gruppen
- Clients verbinden gleich wie vorher

---

## c) Strategie

- **Wann:** am Wochenende, ausserhalb der Steuerperiode
- **Reihenfolge:** Vorlagen → Buchhaltung → Personal → Mandate
- **Tool:** Ich würde Robocopy nehmen oder rsync (nicht sicher ob mit Windows kompatibel) zum Kopieren der Daten

---

## d) Risiken

| Risiko | Lösung |
|--------|--------|
| Rechte falsch | vorher mit Testusern prüfen |
| Umlaute kaputt | Samba auf UTF-8 |
| Zu wenig Platz | Speicher vorher prüfen |
| Migration scheitert | Rollback-Plan |

---

## e) Rollback

- Alter Windows-Server bleibt an.
- Daten nur kopieren, nichts löschen.
- Bei Problemen: Netzlaufwerk zurück auf alten Server.
- Alten Server erst nach 2 Wochen abschalten.

---

## f) Kommunikation

| Wann | Wer | Was |
|------|-----|-----|
| 2 Wochen vorher | Chef | Termin freigeben |
| 1 Woche vorher | Alle | Info-Mail |
| Montag | Alle | Anleitung + Support-Nummer |

---

## g) Cloud-Variante

Statt eigenem Server geht es auch in der Cloud (AWS).

| | On-Premise | Cloud (AWS) |
|--|-----------|-------------|
| Wartung | selbst | AWS (EC2 mit Samba oder AWS FSx)|
| Kosten | einmalig | monatlich |
| Internet | nein | ja, immer |


**Fazit:** Für dieses KMU ist On-Premise am günstigsten und einfachsten. Cloud nur, wenn keine eigene Hardware mehr gewünscht ist.