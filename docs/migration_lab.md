# Migration Lab: Windows-Fileserver auf Linux/Samba

Hier dokumentiere ich, wie ich den Fileserver von Windows auf Linux mit Samba migriert habe. Ziel war: gleiche Ordner, gleiche Rechte, aber auf einem Linux-Server in der AWS-Cloud (EC2).

## Was ich aufgesetzt habe

Als Ziel habe ich eine EC2-Instanz mit Ubuntu Server genommen (Typ t3.micro). Darauf laeuft Samba als Fileserver. In der Security Group habe ich nur Port 22 (SSH) und 445 (SMB) fuer meine eigene IP freigegeben, damit der Server nicht offen im Netz steht.

Die vier Ordner sind die gleichen wie beim Kunden: Mandate, Buchhaltung, Personal, Vorlagen.

## Die Quelle (Windows)

Auf der Windows-Seite habe ich die vier Ordner mit ein paar Testdateien angelegt und die NTFS-Rechte gesetzt. Dazu habe ich lokale Gruppen erstellt (treuhand, buchhaltung, personal, gl) und die Rechte mit `icacls` vergeben. Beispiel fuer Mandate:

```powershell
icacls C:\Fileserver-Quelle\Mandate /grant "treuhand:(OI)(CI)M" "gl:(OI)(CI)R"
```

`M` heisst Lesen+Schreiben, `R` heisst nur Lesen. Genau so wie in der Berechtigungsmatrix.

Den eigentlichen Kopiervorgang von der Windows-VM auf die EC2 konnte ich nicht machen, weil die VM kein Internet hatte (Netzwerk-Problem in VMware). Darum habe ich die Daten direkt auf der Linux-Seite mit rsync migriert. Der Befehl von Windows aus haette so ausgesehen:

```powershell
scp -i samba-key.pem -r Mandate Buchhaltung Personal Vorlagen ubuntu@<EC2-IP>:/home/ubuntu/upload/
```

## Samba auf EC2 einrichten

Zuerst Samba installiert und die Gruppen + Test-User angelegt:

```bash
sudo apt install -y samba acl smbclient

sudo groupadd treuhand
sudo groupadd buchhaltung
sudo groupadd personal
sudo groupadd gl

for u in u_treuhand u_buchhaltung u_personal u_gl; do
  sudo useradd -M -s /usr/sbin/nologin $u
done

sudo usermod -aG treuhand u_treuhand
sudo usermod -aG buchhaltung u_buchhaltung
sudo usermod -aG personal u_personal
sudo usermod -aG gl u_gl
```

Danach fuer jeden User ein Samba-Passwort gesetzt mit `sudo smbpasswd -a <user>`.

Dann die Ordner angelegt und die Rechte mit ACLs gesetzt. ACLs brauche ich, weil pro Ordner mehrere Gruppen unterschiedliche Rechte haben. Beispiel Mandate (Treuhand darf alles, GL nur lesen):

```bash
sudo setfacl -R -m g:treuhand:rwx -m g:gl:rx /srv/samba/mandate
sudo setfacl -dR -m g:treuhand:rwx -m g:gl:rx /srv/samba/mandate
```

Das `-d` sorgt dafuer, dass neue Dateien die Rechte automatisch erben.

Die Freigaben selbst stehen in der `smb.conf` (siehe `config/smb.conf`). Pro Share habe ich mit `valid users` festgelegt, wer ueberhaupt rein darf. Zum Schluss:

```bash
testparm
sudo systemctl restart smbd
```

`testparm` hat "Loaded services file OK" gemeldet, also passt die Config.

## Daten kopieren

Die Migration habe ich mit rsync gemacht. rsync ist gut, weil es genau zeigt was kopiert wird und man es einfach wiederholen kann:

```bash
sudo rsync -av ~/quelle/Mandate/ /srv/samba/mandate/
```

(gleich fuer die anderen drei Ordner)

Mit `getfacl /srv/samba/mandate` habe ich geprueft, dass die Rechte stimmen. Da steht `group:treuhand:rwx` und `group:gl:r-x` drin, also genau richtig.

## Rechte testen

Ich habe pro User mit `smbclient` getestet, ob die Rechte stimmen:

- Treuhand kommt auf Mandate rein und darf schreiben. Passt.
- Buchhaltung kommt NICHT auf Mandate. Zugriff verweigert. Passt.
- GL kommt auf Mandate rein, kann aber nichts speichern (nur lesen). Passt.
- Auf Vorlagen duerfen alle lesen, aber keiner schreiben. Passt.

Die Matrix wird also sauber umgesetzt.

## Sonderfaelle

- **Umlaute:** Ordner "Schäden Müller" und Datei `prüfung_ä_ö_ü.txt` werden korrekt angezeigt, weil Samba auf UTF-8 steht.
- **Grosse Datei (>4 GB):** Eine 5-GB-Datei mit `fallocate -l 5G` erstellt. Kommt ohne Probleme rueber.
- **Langer Pfad (>260 Zeichen):** Unter Windows geht das nur mit der Long-Path-API, unter Linux ist es kein Problem.

## Fazit

Samba laeuft auf der EC2, die Ordner und Rechte sind gleich wie auf dem Windows-Server. Ich habe die Zugriffe pro User getestet und die Sonderfaelle geprueft. Damit ist das Lab ein funktionierender Proof of Concept fuer eine echte Migration.
