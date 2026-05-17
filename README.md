# PP4

## Goal

In this exercise you will:

* Use SSH to connect to remote servers from WSL, macOS, or Linux shells, understanding the handshake and authentication process.
* Generate an Ed25519 SSH key pair and explain the concept of digital signatures.
* Configure your local SSH client via the `~/.ssh/config` file for streamlined access.
* Securely copy files between local and remote hosts using `scp`, including local-to-remote, remote-to-local, and remote-to-remote transfers.
* Automate startup tasks on the remote server by writing a shell script that runs at login and explaining the role of `~/.bashrc` vs. `~/.profile`.

**Important:** Start a stopwatch when you begin and work uninterruptedly for **90 minutes**. Once time is up, stop immediately and record exactly where you paused.

---

## Workflow

1. **Fork** this repository
2. **Modify & commit** your solution
3. **Submit your link for Review**

---

## Prerequisites

* Several starter repos are available here:
  [https://github.com/orgs/STEMgraph/repositories?q=SSH%3A](https://github.com/orgs/STEMgraph/repositories?q=SSH%3A)
* Consult the SSH and SCP man-pages for detailed options and explanations:

  * `man ssh`
  * `man scp`

---

## Tasks

### Task 1: SSH Login

**Objective:** Establish an SSH connection and observe each stage of the process.

1. From your local shell (WSL, macOS Terminal, or Linux), log into the `vorlesungsserver` (or any other remote machine of your choice, e.g. your own raspberry pi):

   ```bash
   ssh -v youruser@remotehost
   ```
2. Carefully observe and note each step:

   * **TCP connection** to port 22 on `remotehost`.
   * **SSH protocol handshake**: key exchange and algorithm negotiation.
   * **Authentication**: public-key or password exchange.
   * **Shell allocation**: your remote session starts.
3. After login, exit the session with `exit`.

**Provide:**

```bash
# 1) The exact ssh command you ran
# 2) A detailed, step-by-step explanation of what happened at each stage
```
1) ssh -v adnan_rihawi@128.140.85.215
2) Beim SSH‑Login wird zuerst eine TCP‑Verbindung aufgebaut. Danach handeln Client und Server im SSH‑Handshake die Verschlüsselungs‑ und Schlüsselaustauschverfahren aus. Anschließend authentifiziert sich der Server über seinen Host‑Key, der mit der Datei known_hosts abgeglichen wird. Zum Schluss authentifiziert sich der Benutzer. Da keine privaten Schlüssel vorhanden waren, erfolgte die Anmeldung per Passwort.


---

### Task 2: Ed25519 Key Pair

**Objective:** Create a secure key pair and explain how digital signatures verify identity.

1. Generate an Ed25519 SSH key pair:

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

   * Accept the default file location (`~/.ssh/id_ed25519`). Or provide the `-f <filepath>` option additionally.
   * Enter a passphrase when prompted (optional).
2. Locate and inspect your `id_ed25519` (private key) and `id_ed25519.pub` (public key).
3. Install your key on the remote machine (e.g. `vorlesungsserver`.
4. Explain in writing:

   * How the **private key** is used to sign challenges.
   * Der SSH‑Server sendet beim Login eine zufällige Challenge an den Client.
Der Client nimmt diese Challenge und erzeugt mit seinem privaten Ed25519‑Schlüssel eine digitale Signatur.
Der private Schlüssel verlässt dabei niemals das Gerät — er wird nur intern benutzt, um die Signatur zu berechnen.
Die Signatur beweist dem Server, dass der Client den privaten Schlüssel besitzt.
   * How the **public key** on the server verifies signatures without revealing the private key.
   * Der Server besitzt den öffentlichen Schlüssel in der Datei ~/.ssh/authorized_keys.
Mit diesem öffentlichen Schlüssel kann der Server mathematisch prüfen, ob die Signatur gültig ist.
Wichtig: Der öffentliche Schlüssel kann die Signatur prüfen, aber niemals den privaten Schlüssel rekonstruieren.
So kann der Server sicherstellen, dass der richtige Benutzer sich anmeldet, ohne dass ein Passwort übertragen wird.
   * Why Ed25519 is preferred (performance, security).
   * Ed25519 ist heute der Standard in OpenSSH, weil es sehr schnell, sehr sicher und resistent gegen Seitenkanalangriffe ist.
Die Schlüssel sind deutlich kürzer als RSA‑Schlüssel, bieten aber höhere Sicherheit.
Außerdem ist Ed25519 weniger fehleranfällig, benötigt keine großen Schlüsselgrößen und ist optimal für moderne Hardware optimiert.
   * 

**Provide:**

```bash
# 1) The ssh-keygen command you ran
ssh-keygen -t ed25519 -C "adnan_rihawi"
# 2) The file paths of the generated keys
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
# 3) Your written explanation (3–5 sentences) of the signature process
Ein Ed25519‑Schlüsselpaar besteht aus einem privaten und einem öffentlichen Schlüssel.
Beim SSH‑Login sendet der Server eine zufällige Challenge an den Client.
Der Client signiert diese Challenge mit seinem privaten Schlüssel, der niemals übertragen wird.
Der Server prüft die Signatur mit dem öffentlichen Schlüssel, der in authorized_keys gespeichert ist.
Wenn die Signatur gültig ist, beweist der Client, dass er den privaten Schlüssel besitzt, und der Login wird ohne Passwort erlaubt.
```

---

### Task 3: SSH Config File

**Objective:** Simplify SSH commands via `~/.ssh/config`.

1. Open (or create) `~/.ssh/config` in `vim`.
2. Add entries for your hosts, for example:

   ```text
   Host my-remote
       HostName remote.example.com
       User youruser
       IdentityFile ~/.ssh/id_ed25519

   Host backup-server
       HostName backup.example.com
       User backupuser
       Port 2222
       IdentityFile ~/.ssh/id_ed25519_backup
   ```
3. Save and close the file, then test:

   ```bash
   ssh my-remote
   ssh backup-server
   ```
4. Explain:

   * How SSH reads `~/.ssh/config` and matches hosts.
     SSH liest die Datei ~/.ssh/config von oben nach unten und sucht nach dem ersten Host‑Eintrag, der zum eingegebenen Befehl passt. Wenn du z. B. ssh vorlesung        eingibst, vergleicht SSH den Namen „vorlesung“ mit allen Host‑Mustern und wendet dann die darunterstehenden Einstellungen (User, HostName, IdentityFile, Port)      automatisch an.
   * The difference between `HostName` and `Host`.
     Host ist ein Alias, den du selbst erfindest. Er existiert nur auf deinem eigenen Computer und dient als Kurzname.
     HostName ist die echte Adresse des Servers (IP oder Domain), zu dem SSH sich verbinden soll.
   * How aliases prevent long commands.
     Aliase verhindern lange SSH‑Befehle, weil ein kurzer Name wie vorlesung automatisch alle gespeicherten Einstellungen (Benutzername, IP‑Adresse, Port und            Schlüssel) ersetzt und dadurch das Tippen komplexer Befehle überflüssig macht.

**Provide:**

```text
# 1) The full contents of your ~/.ssh/config
Host vorlesung
    HostName 128.140.85.215
    User adnan_rihawi
    IdentityFile C:\Users\Adnan\.ssh\id_ed25519
    Port 22
# 2) A short explanation (3–4 sentences) of how the config simplifies connections
    SSH liest die Datei ~/.ssh/config von oben nach unten und wählt den ersten Host‑Eintrag aus, der zum eingegebenen Alias passt. Der Wert bei Host ist nur ein selbst gewählter Kurzname, während HostName die echte IP‑Adresse oder Domain des Servers ist. Wenn ich ssh vorlesung eingebe, übernimmt SSH automatisch alle gespeicherten Einstellungen wie Benutzername, Schlüsseldatei und Port.Dadurch muss ich keine langen SSH‑Befehle mehr tippen,was Verbindungen schneller,einfacher und weniger fehleranfällig macht.
```

---

### Task 4: SCP File Transfers

**Objective:** Practice copying files securely using `scp`.

1. **Local → Remote**:

   ```bash
   scp /path/to/localfile.txt youruser@remotehost:~/destination/
   ```
2. **Remote → Local**:

   ```bash
   scp youruser@remotehost:~/remotefile.log ./local_destination/
   ```
3. **Remote → Remote** (between two directories on the same remote host):

   ```bash
   scp -r youruser@remotehost:/path/dir1 youruser@remotehost:/path/dir2
   ```
4. For each command:

   * Verify file timestamps and sizes after transfer, using `ls -la`
   * Note any flags you used (e.g., `-r`, `-P` for port).
5. Explain:

   * How `scp` initiates an SSH session for each transfer.
     scp startet für jede einzelne Dateiübertragung automatisch eine neue SSH‑Sitzung. Das bedeutet, dass jede Übertragung dieselben Sicherheitsmechanismen nutzt        wie ein normaler SSH‑Login (Authentifizierung, Schlüsselprüfung, Host‑Verifikation).
   * The role of encryption in protecting data in transit.
     Da scp vollständig auf SSH basiert, werden alle übertragenen Daten verschlüsselt, bevor sie das Netzwerk verlassen. Dadurch können Dritte weder Inhalte noch        Zugangsdaten mitlesen oder manipulieren, selbst wenn der Datenverkehr abgefangen wird.

**Provide:**

```bash
# 1) Each scp command you ran
scp C:\Users\Adnan\test.txt adnan_rihawi@128.140.85.215:~/uploads/
scp adnan_rihawi@128.140.85.215:~/server.log C:\Users\Adnan\Downloads\
scp -r adnan_rihawi@128.140.85.215:/home/adnan_rihawi/source \
       adnan_rihawi@128.140.85.215:/home/adnan_rihawi/backup/
# 2) Any flags or options used
-r   # rekursiv kopieren (für Ordner)
# 3) A brief explanation (2–3 sentences) of scp’s mechanism
scp startet für jede Dateiübertragung automatisch eine neue SSH‑Sitzung, wodurch dieselben Sicherheitsmechanismen wie beim normalen Einloggen gelten. Die gesamte Kommunikation wird verschlüsselt, sodass weder Dateien noch Zugangsdaten unterwegs mitgelesen oder verändert werden können. Dadurch ist SCP eine sichere Methode,um Daten zwischen lokalen und entfernten Systemen zu übertragen.
```

---

### Task 5: Login Shell Script & Profile Explanation

**Objective:** Automate commands at login and understand shell initialization files.

1. On the **remote** server, create a script `~/login_tasks.sh` containing at least three commands you find useful (e.g., `echo "Welcome $(whoami)"`, `uptime`, `ls ~/projects`). You may either use `vim` or try the following to create a file from your commandline directely:

   ```bash
   cat << 'EOF' > ~/login_tasks.sh
   #!/usr/bin/env bash
   echo "Welcome $(whoami)! Today is $(date)."
   uptime
   ls ~/projects
   EOF
   chmod +x ~/login_tasks.sh
   ```

> The files content should be something akin to:
> ```bash
> #!/usr/bin/env bash
> echo "Welcome $(whoami)! Today is $(date)."
> uptime
> ls ~/projects
> ```

2. Append to your `~/.bashrc` (or `~/.profile` if using a login shell) a line to source this script on each new session:

   ```bash
   echo "source ~/login_tasks.sh" >> ~/.bashrc
   ```
3. Log out and log back in to trigger the script.
4. Explain:

   * The difference between `~/.bashrc` and `~/.profile` (interactive vs. login shells).
     ~/.profile wird von Login‑Shells gelesen, also z. B. wenn du dich per SSH auf einem Server anmeldest.
     ~/.bashrc wird von interaktiven, nicht‑Login‑Shells geladen, z. B. jedes Mal, wenn du ein neues Terminalfenster öffnest oder bash startest.
   * Why and when each file is read.
     ~/.profile wird nur einmal beim Einloggen ausgeführt, weil dort Umgebungsvariablen und Startskripte stehen sollen, die nur einmal gesetzt werden müssen.
     ~/.bashrc wird jedes Mal ausgeführt, wenn eine neue Bash‑Shell startet, damit Aliase, Funktionen und Einstellungen immer verfügbar sind.
     Darum gehören Login‑Tasks in .profile und interaktive Einstellungen in .bashrc.
   * How sourcing differs from executing.
     Beim Sourcen (source datei.sh) wird der Inhalt der Datei im aktuellen Shell‑Prozess ausgeführt — Variablen, Funktionen und Änderungen bleiben erhalten.
     Beim Ausführen (./datei.sh) startet Bash einen neuen, separaten Prozess, der nach dem Ende wieder verschwindet und keine Änderungen an deiner aktuellen Shell       hinterlässt.
    Darum werden Startskripte wie login_tasks.sh immer gesourct, nicht ausgeführt.
     

**Provide:**

```bash
# 1) The contents of login_tasks.sh
#!/bin/bash
echo "Willkommen, Adnan! Login-Tasks wurden ausgeführt."
date
# 2) The lines you added to ~/.bashrc or ~/.profile
source ~/login_tasks.sh
# 3) Your explanation (3–5 sentences) of shell init files and sourcing vs. executing
~/.profile wird von Login‑Shells gelesen, zum Beispiel wenn man sich per SSH auf einem Server anmeldet.
~/.bashrc wird dagegen von interaktiven Shells geladen, also jedes Mal, wenn ein neues Terminalfenster geöffnet wird.
Beim Sourcen (source datei.sh) wird der Inhalt der Datei im aktuellen Shell‑Prozess ausgeführt, sodass Variablen, Funktionen und Änderungen erhalten bleiben.
Beim Ausführen (./datei.sh) startet Bash einen neuen Prozess, der nach dem Ende wieder verschwindet und keine Änderungen an der aktuellen Shell hinterlässt.
Darum werden Startskripte wie login_tasks.sh in .profile eingebunden und gesourct, nicht ausgeführt.
```

---

**Remember:** Stop working after **90 minutes** and record where you stopped.
