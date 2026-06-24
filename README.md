# 🚗 PUDIS / Checkmk Client

## Worum geht es in diesem Projekt?

Dieses Projekt entwickelt eine kombinierte Lösung aus **Client** und **Skripten** für **PUDIS** bzw. **Checkmk**.

An den Prüfständen werden über PUDIS **Fahrzeugprotokolle** für Fahrzeuge gezogen.  
Das Team vor Ort arbeitet dabei mit einer bestehenden Software und nutzt PUDIS im Hintergrund für bestimmte Funktionen.

Ziel ist es, einen **Client** zu entwickeln, der das Prüfstandteam bei diesen Aufgaben unterstützt und bestimmte Abläufe vereinfacht.  
Das Team soll später bestimmte Funktionen **nicht mehr direkt manuell in PUDIS**, sondern **über den Client** nutzen können.

Parallel dazu werden bestimmte Funktionen auch **scriptbasiert** umgesetzt.  
Der Hintergrund ist, dass an den **PUDIS-Stationen bzw. über Checkmk** bestimmte Informationen oder Funktionen **nur scriptbasiert** bereitgestellt oder angezeigt werden können.

---

## Ziel des Projekts

Das Projekt verfolgt zwei Richtungen:

### 1. Client-basiert für die Prüfstände
Ein Client soll für das Prüfstandteam entwickelt werden, über den bestimmte Aufgaben rund um PUDIS vereinfacht und zentral ausgeführt werden können.

### 2. Script-basiert für Checkmk / PUDIS-Stationen
Zusätzlich sollen bestimmte Funktionen parallel scriptbasiert umgesetzt werden, damit Zustände oder Informationen über Checkmk bzw. an PUDIS-Stationen angezeigt oder verarbeitet werden können.

---

## Aktueller fachlicher Fokus

Der erste sinnvolle Baustein ist die Überwachung von:

- **ApplicationStatus**
- **ConnectionStatus**

Damit lässt sich bereits erkennen,

- ob die Anwendung bereit ist,
- ob gerade eine Prozedur läuft,
- ob eine Verbindung besteht,
- ob ein VCI erkannt wurde,
- ob das System sich gerade prüft,
- ob Checks abgeschlossen wurden,
- ob beim Hochfahren Fehler aufgetreten sind.

---

## Perspektivisch mögliche Erweiterungen

Je nach Abstimmung mit dem Team kann das Projekt später zusätzlich erweitert werden um:

- Authentifizierung
- Überwachung oder Steuerung von Prozeduren
- Nutzung weiterer PUDIS-Services
- Logging / Nachverfolgung
- Konfigurierbarkeit
- Checkmk-spezifische Skripte
- zusätzliche Automatisierungslogik

---

## Fachlicher Hintergrund

PUDIS stellt eine **gRPC-API** bereit.  
Über diese API kann ein externer Client Informationen abfragen oder Zustandsänderungen beobachten.

### Beispiele für relevante Informationen

#### ApplicationStatus
Zeigt den Zustand der Anwendung, z. B.:

- System prüft sich
- Checks abgeschlossen
- bereit
- Prozedur läuft
- Check fehlgeschlagen

#### ConnectionStatus
Zeigt den Zustand der Verbindung, z. B.:

- nicht verbunden
- VCI erkannt
- DoIP aktiv

Die Schnittstelle wird über **`.proto`-Dateien** beschrieben.  
Aus diesen Dateien wird der Python-Code erzeugt, der für die gRPC-Kommunikation benötigt wird.

---

## Technisches Grundprinzip

### Rollenverständnis

- **PUDIS = Server / API-Anbieter**
- **Client = Benutzernahe Lösung für das Prüfstandteam**
- **Skripte = Technische Lösung für Checkmk / PUDIS-Stationen**

### Rolle der `.proto`-Dateien

Die `.proto`-Dateien definieren:

- welche Services verfügbar sind,
- welche Methoden aufgerufen werden können,
- welche Datenstrukturen und Zustände es gibt.

Beispiele:

- `ApplicationStatusService`
- `GetApplicationStatus`
- `WatchApplicationStatus`
- `GetConnectionStatus`
- `WatchConnectionStatus`

### Generierter Code

Aus den `.proto`-Dateien werden Python-Dateien erzeugt:

- `*_pb2.py`
- `*_pb2_grpc.py`

Diese Dateien sind **generierter Code** und werden **nicht manuell angepasst**.

### Logischer Aufbau des Projekts

Das Projekt lässt sich in mehrere Ebenen unterteilen:

1. **Proto-Ebene** → beschreibt die API  
2. **Generated Code** → technische Python-Schnittstelle  
3. **Client-Schicht** → direkte Kommunikation mit PUDIS  
4. **Service-Schicht** → fachliche Logik für den Client  
5. **Script-Schicht** → scriptbasierte Funktionen für Checkmk / PUDIS-Stationen  
6. **Startpunkt / Main** → Ausführung des Clients oder der Skripte  

---

## Voraussetzungen

Damit ein neuer Praktikant oder Entwickler das Projekt lokal einrichten und ausführen kann, werden folgende Voraussetzungen benötigt.

### Software

- **Python 3.11 oder 3.12**
- **pip**
- **Visual Studio** oder **VS Code**
- Zugriff auf die **PUDIS-Anwendung**
- Zugriff auf die **aktuellen `.proto`-Dateien**
- falls relevant: Zugriff auf **Checkmk** bzw. die benötigte Umgebung

### Benötigte Python-Bibliotheken

Aktuell werden mindestens folgende Bibliotheken benötigt:

- `grpcio`
- `grpcio-tools`
- `protobuf`

### Installation der Bibliotheken

```bash
python -m pip install grpcio grpcio-tools protobuf
```

### Laufzeitvoraussetzungen

Vor dem Start muss bekannt sein:

- auf welcher Adresse der gRPC-Server läuft,
- welcher Port verwendet wird.

Aktuell bekannt:

- **Host:** `localhost`
- **Port:** `5050`

### Wichtiger Hinweis

PUDIS muss laufen, damit sich der Client verbinden kann.  
Wenn PUDIS geschlossen ist, ist der gRPC-Server nicht erreichbar.

---

## Einrichtung für neue Praktikanten / Entwickler

### 1. Projekt lokal bereitstellen
Repository bzw. Projektordner lokal öffnen.

### 2. Python installieren
Empfohlen: Python 3.11 oder 3.12

### 3. Python-Abhängigkeiten installieren

```bash
python -m pip install grpcio grpcio-tools protobuf
```

oder – falls vorhanden – über eine `requirements.txt`:

```bash
python -m pip install -r requirements.txt
```

### 4. `.proto`-Dateien prüfen
Sicherstellen, dass die aktuellen und richtigen `.proto`-Dateien vorhanden sind.

### 5. gRPC-Code generieren
Beispiel für `application_status.proto`:

```bash
python -m grpc_tools.protoc -I./protos --python_out=./generated --grpc_python_out=./generated ./protos/application_status.proto
```

### 6. PUDIS starten
Ohne laufende PUDIS-Anwendung kann der Client nicht funktionieren.

### 7. Konfiguration prüfen
Host und Port kontrollieren.

### 8. Client oder Skripte starten
Je nach Anwendungsfall wird entweder der Client oder ein scriptbasierter Aufruf verwendet.

---

## Empfohlene Projektstruktur

Damit das Projekt sowohl für den Client als auch für scriptbasierte Funktionen wachsen kann, sollte es modular aufgebaut werden.

```text
pudis-client/
│
├─ README.md
├─ requirements.txt
├─ .gitignore
│
├─ protos/
│   ├─ application_status.proto
│   ├─ authentication.proto
│   ├─ procedure_control.proto
│   └─ data_distributions.proto
│
├─ generated/
│   ├─ application_status_pb2.py
│   ├─ application_status_pb2_grpc.py
│   └─ ...
│
├─ src/
│   ├─ main.py
│   ├─ config.py
│   │
│   ├─ clients/
│   ├─ services/
│   ├─ mappers/
│   └─ utils/
│
├─ scripts/
│   ├─ checkmk/
│   ├─ pudis_station/
│   └─ common/
│
├─ tests/
└─ docs/
```

---

## Erklärung der Struktur

### `src/`
Enthält den **client-basierten Python-Code** für das Prüfstandteam.

### `scripts/`
Enthält **scriptbasierte Lösungen** für Checkmk bzw. PUDIS-Stationen.

### `protos/`
Enthält die originalen `.proto`-Dateien.

### `generated/`
Enthält den automatisch generierten Python-Code aus den `.proto`-Dateien.

### `tests/`
Testfälle und ggf. spätere automatisierte Tests.

### `docs/`
Dokumentation und technische Notizen.

---

## Wichtige Hinweise

- Wenn PUDIS geschlossen ist, ist der Server nicht erreichbar.
- Streams können denselben Zustand mehrfach senden.
- Solche Zustände sollten im Client dedupliziert werden.
- Wenn eine Methode im generierten Python-Code fehlt, passt meist die `.proto`-Datei nicht zum generierten Stand.
- Änderungen an `.proto`-Dateien erfordern eine erneute Generierung des Python-Codes.
- Für Checkmk-spezifische Anforderungen können zusätzliche scriptbasierte Lösungen notwendig sein.

---

## Offene Punkte

Da die finalen Anforderungen noch mit dem Team abgestimmt werden, bleiben aktuell noch Fragen offen.

Beispiele:

- Welche Aufgaben soll der Client final vollständig für das Prüfstandteam unterstützen?
- Welche Funktionen sollen parallel scriptbasiert für Checkmk bereitgestellt werden?
- Welche weiteren Services sollen angebunden werden?
- Soll der Client später nur Informationen anzeigen oder auch Aktionen starten?
- Welche Teile gehören fachlich in den Client und welche in die Skript-Schicht?

---

## Zusammenfassung

Das Projekt verfolgt zwei Ziele:

1. **Einen Client für das Prüfstandteam aufbauen**, der bestimmte Aufgaben rund um PUDIS vereinfacht.  
2. **Parallele scriptbasierte Funktionen bereitstellen**, damit Informationen oder Zustände auch über Checkmk bzw. PUDIS-Stationen genutzt werden können.

Damit entsteht eine kombinierte Lösung aus:

- **benutzernahem Client für das Team**
- **technischen Skripten für Checkmk / PUDIS-Stationen**
