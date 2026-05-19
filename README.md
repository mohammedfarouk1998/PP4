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
## Aufgabe 1: SSH-Login-Protokollanalyse (Task 1)

### 1) Der exakte SSH-Befehl:
ssh -v MohammedS1998@128.140.85.215

### 2) Detaillierte Schritt-für-Schritt-Erklärung der SSH-Phasen:

* **Schritt 1: Verbindungsaufbau (TCP-Verbindung)**
  * **Log-Auszug:** `debug1: Connecting to 128.140.85.215 [...] port 22.` -> `debug1: Connection established.`
  * **Erklärung:** Mein lokaler Rechner (Client) initiiert eine TCP-Verbindung zum entfernten Server auf dem Standard-SSH-Port 22. Der Server antwortet und die grundlegende Netzwerkverbindung wird erfolgreich hergestellt (`Connection established`).

* **Schritt 2: Protokoll- und Banner-Austausch (Handshake)**
  * **Log-Auszug:** `Local version string SSH-2.0-OpenSSH_for_Windows_9.5` -> `Remote protocol version 2.0, remote software version OpenSSH_10.0p2 Debian-7`
  * **Erklärung:** In dieser Phase lernen sich die beiden Rechner kennen. Sie tauschen ihre Software-Versionen und Betriebssystem-Informationen aus (Windows OpenSSH 9.5 auf Client-Seite und Debian OpenSSH 10.0p2 auf Server-Seite), um sich auf eine gemeinsame Protokollversion (SSH-2.0) zu einigen.

* **Schritt 3: Authentifizierungsversuch (Authenticating)**
  * **Log-Auszug:** `debug1: Authenticating to [...] as 'MohammedS1998'` -> `debug1: Next authentication method: publickey, password`
  * **Erklärung:** Der Server versucht nun, meine Identität für den Benutzer `MohammedS1998` zu überprüfen. Zuerst wird die sichere Public-Key-Methode versucht. Da diese fehlschlägt (z. B. weil der Schlüssel noch nicht hinterlegt war), schaltet das System auf die nächste verfügbare Methode um: die Passworteingabe (`Next authentication method: publickey, password`).

* **Schritt 4: Erfolgreiche Anmeldung und Sitzungsstart (Session Allocation)**
  * **Log-Auszug:** `Authenticated to [...] using "password".` -> `debug1: channel 0: new session [client-session]`
  * **Erklärung:** Nach der Eingabe des korrekten Passworts akzeptiert der Server die Authentifizierung (`Authenticated`). Anschließend wird ein neuer sicherer Kanal (`channel 0`) geöffnet und die interaktive Shell-Sitzung (`client-session`) auf dem Remote-Server erfolgreich gestartet.
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
   * How the **public key** on the server verifies signatures without revealing the private key.
   * Why Ed25519 is preferred (performance, security).

**Provide:**

```bash
# 1) The ssh-keygen command you ran
ssh-keygen -t ed25519 -C "mohamed@farouk.de" 
# 2) The file paths of the generated keys
/home/MohammedS1998/task2
/home/MohammedS1998/task2.pub
# 3) Your written explanation (3–5 sentences) of the signature process
Der Server schickt dem Client eine mathematische Herausforderung. Dann schickt der Client seine Signatur zurück. Der Server hat bereits den Schlüssel, vergleicht beide miteinander und wenn es passt, ist es richtig. Ed25519 wird genutzt, weil es effizienter, schneller und
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
   * The difference between `HostName` and `Host`.
   * How aliases prevent long commands.

**Provide:**

```text
# 1) The full contents of your ~/.ssh/config
# 2) A short explanation (3–4 sentences) of how the config simplifies connections
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
   * The role of encryption in protecting data in transit.

**Provide:**

```bash
# 1) Each scp command you ran
# 2) Any flags or options used
# 3) A brief explanation (2–3 sentences) of scp’s mechanism
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
   * Why and when each file is read.
   * How sourcing differs from executing.

**Provide:**

```bash
# 1) The contents of login_tasks.sh
# 2) The lines you added to ~/.bashrc or ~/.profile
# 3) Your explanation (3–5 sentences) of shell init files and sourcing vs. executing
```

---

**Remember:** Stop working after **90 minutes** and record where you stopped.
