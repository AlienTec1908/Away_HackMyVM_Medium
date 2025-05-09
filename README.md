# Away - HackMyVM Writeup

![Away VM Icon](Away.png)

Dieses Repository enthält das Writeup für die HackMyVM-Maschine "Away" (Schwierigkeitsgrad: Medium), erstellt von DarkSpirit. Ziel war es, initialen Zugriff auf die virtuelle Maschine zu erlangen und die Berechtigungen bis zum Root-Benutzer zu eskalieren.

## VM-Informationen

*   **VM Name:** Away
*   **Plattform:** HackMyVM
*   **Autor der VM:** DarkSpirit
*   **Schwierigkeitsgrad:** Medium
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Away](https://hackmyvm.eu/machines/machine.php?vm=Away)

## Writeup-Informationen

*   **Autor des Writeups:** Ben C.
*   **Datum des Berichts:** 05. Oktober 2022
*   **Link zum Original-Writeup (GitHub Pages):** [https://alientec1908.github.io/Away_HackMyVM_Medium/](https://alientec1908.github.io/Away_HackMyVM_Medium/)

## Kurzübersicht des Angriffspfads

Der Angriff auf die Away-Maschine umfasste folgende Schritte:

1.  **Reconnaissance:**
    *   Identifizierung der Ziel-IP (`192.168.2.121`) mittels `arp-scan`.
    *   Ein `nmap`-Scan offenbarte zwei offene Ports: SSH (22, OpenSSH 8.4p1) und HTTP (80, Nginx 1.18.0).
2.  **Web Enumeration:**
    *   `gobuster` fand nur eine `index.html`-Datei.
    *   Ein Hinweis (ASCII-Art Fingerprint) deutete auf den SSH-Benutzernamen `tula` hin (Quelle unklar, evtl. `index.html`).
3.  **Initial Access (SSH):**
    *   Ein Versuch, sich als `tula` per SSH anzumelden, schlug fehl (`Permission denied (publickey)`), was auf reine Schlüsselauthentifizierung hindeutete.
    *   Der öffentliche SSH-Schlüssel `id_ed25519.pub` wurde gefunden (Quelle unklar, vermutlich Webserver). Dieser enthielt im Kommentar die Passphrase des privaten Schlüssels: `Theclockisticking`.
    *   Der dazugehörige private SSH-Schlüssel `id_ed25519` wurde über `curl http://192.168.2.121/id_ed25519` vom Webserver heruntergeladen.
    *   Erfolgreicher SSH-Login als `tula` mit dem heruntergeladenen privaten Schlüssel und der bekannten Passphrase.
4.  **Privilege Escalation (tula -> lula via Webhook):**
    *   `sudo -l` für `tula` zeigte: `(lula) NOPASSWD: /usr/bin/webhook`.
    *   Die User-Flag (`/home/tula/user.txt`) wurde gelesen.
    *   Ein Webhook wurde konfiguriert (`hooks.json`), der bei Aufruf ein Skript (`/dev/shm/execute.sh`) ausführt. Dieses Skript enthielt einen Netcat-Reverse-Shell-Befehl.
    *   Der Webhook-Dienst wurde mit `sudo -u lula /usr/bin/webhook -hooks hooks.json -verbose` gestartet (lauscht auf Port 9000).
    *   Durch Aufrufen der URL `http://192.168.2.121:9000/hooks/redeploy-webhook` wurde der Hook ausgelöst und eine Reverse Shell als Benutzer `lula` auf dem Angreifer-System empfangen.
5.  **Privilege Escalation (lula -> root via SSH Key Leak):**
    *   Als `lula` konnte der private SSH-Schlüssel des Root-Benutzers (`/root/.ssh/id_ed25519`) mittels `/usr/bin/more` gelesen werden (gravierende Fehlkonfiguration der Berechtigungen).
    *   Der ausgelesene private Root-Schlüssel wurde auf dem Angreifer-System gespeichert.
    *   Erfolgreicher SSH-Login als `root` mit dem erbeuteten privaten Schlüssel.
6.  **Flags:**
    *   Die User-Flag wurde als `tula` gelesen.
    *   Die Root-Flag wurde aus `/root/ro0t.txt` gelesen.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `ssh`
*   `ssh-keyscan`
*   `vi` / `cat`
*   `curl`
*   `chmod`
*   `sudo`
*   `mkdir`
*   `nano`
*   `cp`
*   `webhook`
*   `nc` (netcat)
*   `id`
*   `more`
*   `ls`

## Identifizierte Schwachstellen (Zusammenfassung)

*   **Preisgabe von SSH-Benutzernamen und Schlüsselinformationen:** Ein Hinweis auf den Benutzer `tula` und die Passphrase des privaten Schlüssels im Kommentar des öffentlichen Schlüssels.
*   **Verfügbarkeit eines privaten SSH-Schlüssels über HTTP:** Der private Schlüssel für `tula` war öffentlich zugänglich.
*   **Unsichere `sudo`-Konfiguration (tula -> lula):** `tula` konnte `/usr/bin/webhook` als `lula` ohne Passwort ausführen. `webhook` konnte so konfiguriert werden, dass es beliebige Befehle ausführt.
*   **Fehlkonfigurierte Dateiberechtigungen (SSH Key Leak):** Der Benutzer `lula` hatte Leserechte auf den privaten SSH-Schlüssel des Root-Benutzers (`/root/.ssh/id_ed25519`).

## Flags

*   **User Flag (`/home/tula/user.txt`):** `HMVDULEMISPLCYDKEG`
*   **Root Flag (`/root/ro0t.txt`):** `HMVNYDWAPOQNDYUBNI`

---

**Wichtiger Hinweis:** Dieses Dokument und die darin enthaltenen Informationen dienen ausschließlich zu Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Techniken und Werkzeuge sollten nur in legalen und autorisierten Umgebungen (z.B. eigenen Testlaboren, CTFs oder mit expliziter Genehmigung) angewendet werden. Das unbefugte Eindringen in fremde Computersysteme ist eine Straftat und kann rechtliche Konsequenzen nach sich ziehen.

---
*Bericht von Ben C. - Cyber Security Reports*
