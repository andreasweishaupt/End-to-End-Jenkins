# Jenkins Setup mit Docker Compose

Diese Anleitung beschreibt, wie Sie Jenkins mit Docker Compose einrichten, zwei Agenten hinzufügen und eine Pipeline konfigurieren.

## Voraussetzungen

- Docker und Docker Compose müssen auf Ihrem System installiert sein.

## Schritte

1) Build agent
    1) Clone repo: https://github.com/jenkinsci/docker-agent
    2) Run in cloned repo: `docker build -t test:ubuntu -f ubuntu/Dockerfile .`

### 1. Jenkins mit Docker Compose starten

Führen Sie den folgenden Befehl aus, um Jenkins zu starten:

```bash
docker-compose up -d
```

### 2. Jenkins initialisieren und einrichten

3) Admin Passwort abfragen und speicher (kann nach Neustart nicht mehr erhalten werden): `docker compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword`

1. Öffnen Sie einen Webbrowser und gehen Sie zu `http://localhost:8060`, um auf die Jenkins-Weboberfläche zuzugreifen.
2. Folgen Sie den Anweisungen zur Ersteinrichtung von Jenkins.
3. Installieren Sie die empfohlenen Plugins.

### 3. Agenten anlegen

#### Agent 1

1. Gehen Sie zu **Verwalten Jenkins** > **Verwalten von Nodes und Clouds**.
2. Klicken Sie auf **Neuer Node**.
3. Geben Sie den Namen `agent-1` ein und wählen Sie **Statischer Agent**.
4. Klicken Sie auf **OK**.
5. Füllen Sie das Feld **Stammverzeichnis in entferntem Dateisystem** mit `./volumes/agent-1`.
6. Speichern Sie die Konfiguration.

#### Agent 2

Wiederholen Sie die obigen Schritte für `agent-2`, aber verwenden Sie `./volumes/agent-2` als Stammverzeichnis.

### 4. Geheimnis in docker-compose.yml kopieren

1. Nach der Erstellung der Agenten wird in der Detailansicht des Agenten ein Geheimnis (Secret) angezeigt.
2. Kopieren Sie das Secret für jeden Agenten in Ihre `docker-compose.yml` Datei unter `JENKINS_SECRET`.

Node master bearbeiten und Anzahl der Build-Prozessoren auf 0 setzen

### 5. Container neu starten

Führen Sie die folgenden Befehle aus, um die Container neu zu starten:

```bash
docker-compose down
docker-compose up -d
```

### 6. Berechtigungen setzen

Bevor die Pipeline ausgeführt werden kann, müssen noch die Rechte gesetzt werden:
Führen Sie dazu mit den folgenden Befehlen die notwendigen Befehle im Jenkins-Container aus:
```bash
docker exec -u root e2e-jenkins chown -R jenkins:jenkins /var/jenkins_home/workspace
docker exec -u root e2e-jenkins chmod -R 775 /var/jenkins_home/workspace
```

### 7. Pipeline anlegen

1. Gehen Sie zu **Element anlegen** in der Jenkins-Weboberfläche.
2. Geben Sie den Namen `e2e-pipeline` ein und wählen Sie den Typ **Pipeline**.
3. Mit ok bestätigen
3. Scrollen Sie nach unten zum Abschnitt **Pipeline script** und fügen Sie den folgenden Code ein:

```groovy
node('agent-1') {
    def agentName = env.NODE_NAME
    echo agentName
    def jenkinsFilePath = '/var/jenkins_home/workspace/file/Jenkinsfile'
    load jenkinsFilePath
}
```

4. Klicken Sie auf Speichern.

### 8. Pipeline ausführen

Klicken Sie auf **Jetzt ausführen**, um die Pipeline zu starten. Dies wird den definierten Build-Prozess in Gang setzen und die Schritte im `Jenkinsfile` auf dem angegebenen Agenten ausführen.
