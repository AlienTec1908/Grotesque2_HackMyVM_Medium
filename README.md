# Grotesque2 - HackMyVM (Medium) - Writeup

![Grotesque2 Icon](Grotesque2.png)

Dieses Repository enthält einen zusammengefassten Bericht über die Kompromittierung der HackMyVM-Maschine "Grotesque2" (Schwierigkeitsgrad: Medium).

## Metadaten

*   **Maschine:** Grotesque2 (HackMyVM - Medium)
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Grotesque2](https://hackmyvm.eu/machines/machine.php?vm=Grotesque2)
*   **Autor des Writeups:** DarkSpirit
*   **Datum:** 16. November 2022
*   **Original Writeup:** [https://alientec1908.github.io/Grotesque2_HackMyVM_Medium/](https://alientec1908.github.io/Grotesque2_HackMyVM_Medium/)

## Zusammenfassung des Angriffspfads

Die initiale Reconnaissance mittels `arp-scan` identifizierte das Ziel. Standard-Web-Scans (`gobuster`, `nikto`) auf Port 80 scheiterten oder lieferten keine verwertbaren Ergebnisse, was auf eine ungewöhnliche Konfiguration hindeutete.

Ein systematisches Scannen von Ports mittels einer `wget`-Schleife deckte einen versteckten Webserver auf Port 258 auf. Die Webseite auf diesem Port enthielt eine Liste potenzieller SSH-Benutzernamen (`satan, raphael, angel, distress, greed or lust`). Weiterhin wurde in einem Bild auf dieser Seite durch starke Vergrößerung (Steganographie) ein MD5-Passwort-Hash entdeckt. Nach einer unklaren Modifikation dieses Hashes (basierend auf einem Hinweis auf der Webseite) wurde er mittels `john` und `rockyou.txt` erfolgreich zum Passwort `solomon1` geknackt.

Ein `hydra`-Angriff auf den SSH-Dienst (Port 22) unter Verwendung der Benutzernamenliste und des Passworts `solomon1` identifizierte die gültige Kombination `angel:solomon1`. Dies ermöglichte den initialen SSH-Zugriff als Benutzer `angel`.

Die lokale Enumeration als `angel` führte zum Fund der Datei `/rootcreds.txt` im Root-Verzeichnis (`/`). Diese Datei war für alle les- und schreibbar und enthielt die Klartext-Zugangsdaten für den `root`-Benutzer (`root:sweetchild`). Mittels `su root` und dem geleakten Passwort wurde Root-Zugriff erlangt. Anschließend wurden die User- und Root-Flags gelesen.

## Verwendete Tools (Auswahl)

*   `arp-scan`
*   `gobuster`
*   `nikto`
*   `wget`
*   `grep`
*   `ll` (alias for `ls -l`)
*   `cat`
*   `echo`
*   `john`
*   `hydra`
*   `ssh`
*   `sudo` (erfolglos versucht)
*   `su`
*   `cd`
*   `ls`

## Angriffsschritte (Übersicht)

1.  **Reconnaissance:** Ziel-IP (`arp-scan`), Hostname (`gro.hmv` in `/etc/hosts`). Initiale Webscans auf Port 80 (`gobuster`, `nikto`) scheitern/sind unproduktiv.
2.  **Hidden Port Discovery:** Systematischer Portscan mit `wget`-Schleife -> Port 258 identifiziert abweichenden Inhalt.
3.  **Credential Discovery:** Webseite auf Port 258 -> Liste von SSH-Usernames. Bild auf Port 258 -> MD5-Hash via Steganographie. Hash modifizieren (basierend auf Seitenhinweis).
4.  **Password Crack:** Modifizierten MD5-Hash (`b6e7...0b3`) mit `john --format=Raw-MD5 ... rockyou.txt` knacken -> Passwort `solomon1`.
5.  **Initial Access:** `hydra` mit Userliste und Passwort `solomon1` gegen SSH -> `angel:solomon1` gefunden. Login via `ssh angel@gro.hmv`.
6.  **User Flag:** `cat /home/angel/user.txt`.
7.  **Privilege Escalation Vector:** `ls -la /` -> Datei `/rootcreds.txt` mit unsicheren Berechtigungen (`rwxrwxrwx`) gefunden.
8.  **Leaked Credentials:** `cat /rootcreds.txt` -> Inhalt `root:sweetchild`.
9.  **Root Access:** `su root` mit Passwort `sweetchild`.
10. **Root Flag:** `cat /root/root.txt`.

## Flags

*   **User Flag (`/home/angel/user.txt`):** `8B7C6AC34997997C91125A08E01AEF57`
*   **Root Flag (`/root/root.txt`):** `DC6F071EF98631030FD300DE4A556CBF`

---

## Disclaimer

Die hier dargestellten Informationen und Techniken dienen ausschließlich Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Methoden dürfen nur auf Systemen angewendet werden, für die eine ausdrückliche Genehmigung vorliegt (z.B. in CTF-Umgebungen wie HackMyVM, Penetrationstests mit schriftlicher Erlaubnis). Das unbefugte Eindringen in fremde Computersysteme ist illegal und strafbar. Die Autoren übernehmen keine Haftung für missbräuchliche Verwendung der bereitgestellten Informationen. Handeln Sie stets legal und ethisch verantwortlich.

---
