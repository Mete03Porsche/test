# 🚗 PUDIS / Checkmk Client

## Worum geht es in diesem Projekt?

Dieses Projekt entwickelt eine kombinierte Lösung aus **Client** und **Skripten** für **PUDIS** bzw. **Checkmk**.

An den Prüfständen werden über PUDIS **Fahrzeugprotokolle** für Fahrzeuge gezogen.  
Das Team vor Ort arbeitet dabei mit einer bestehenden Software um bestimmte Funktionen wie z.B die Status-Überwachung durchzuführen.

Ziel ist es, einen **Client** zu entwickeln, der das Prüfstandteam bei diesen Aufgaben unterstützt und bestimmte Abläufe vereinfacht.  
Das Team soll später bestimmte Funktionen vereinfacht **über den Client** nutzen können.

Parallel dazu werden bestimmte Funktionen auch **scriptbasiert** umgesetzt.  
Der Hintergrund ist, dass **für Checkmk** bestimmte Informationen oder Funktionen **scriptbasiert** bereitgestellt oder angezeigt werden müssen.

---

## Ziel des Projekts

Das Projekt verfolgt zwei Richtungen:

### 1. Client-basiert für die Prüfstände

Ein Client soll für das Prüfstandteam entwickelt werden, über den bestimmte Aufgaben rund um PUDIS vereinfacht und zentral ausgeführt werden können.

Der Client ist für Benutzer gedacht und kann später z. B.:

- Zustände verständlich anzeigen
- Abläufe überwachen
- Funktionen bündeln
- mehrere PUDIS-Services verwenden
- Workflows für das Prüfstandteam vereinfachen

### 2. Script-basiert für Checkmk / PUDIS-Stationen

Zusätzlich sollen bestimmte Funktionen parallel scriptbasiert umgesetzt werden.

Diese Skripte sollen vom Checkmk-Agenten bzw. auf PUDIS-Stationen ausgeführt werden und eine Checkmk-kompatible Ausgabe liefern.

Beispiel:

```text
0 "PUDIS Login Status" - Eingeloggt: A4013EM
```

oder:

```text
2 "PUDIS Login Status" - Nicht eingeloggt
```

---



## Fachlicher Hintergrund

PUDIS stellt eine **gRPC-API** bereit.  
Über diese API kann ein externer Client Informationen abfragen oder Zustandsänderungen beobachten.

Beispiele:

### ApplicationStatus

Der ApplicationStatus beschreibt den Zustand der PUDIS-Anwendung.

Beispiele:

- System prüft sich
- Checks abgeschlossen
- Application ist bereit
- Prozedur läuft
- Fehler im System

Zusätzlich enthält der ApplicationStatus Informationen zum aktuell authentifizierten Benutzer:

```proto
User authenticated_user = 5;
LastLoginAttempt last_login_attempt = 6;
```

Diese Felder werden für die Loginprüfung verwendet.

### ConnectionStatus

Der ConnectionStatus beschreibt den Zustand der Verbindung.

Beispiele:

- nicht verbunden
- VCI erkannt

---

## Technisches Grundprinzip

### Rollenverständnis

- **PUDIS = Server / API-Anbieter**
- **Client = Benutzernahe Lösung für das Prüfstandteam**
- **Skripte = Technische Lösung für Checkmk / PUDIS-Stationen**
- **Checkmk-Agent = führt das Skript auf der Zielstation aus**

---

## Rolle der `.proto`-Dateien

Die `.proto`-Dateien definieren die gRPC-Schnittstelle.

Darin steht:

- welche Services verfügbar sind,
- welche Methoden aufgerufen werden können,
- welche Datenstrukturen und Zustände existieren.

Beispiele:

```text
ApplicationStatusService
GetApplicationStatus
WatchApplicationStatus
GetConnectionStatus
WatchConnectionStatus
```

Aus den `.proto`-Dateien werden Python-Dateien generiert:

```text
application_status_pb2.py
application_status_pb2_grpc.py
```

Diese Dateien sind **generierter Code** und werden **nicht manuell angepasst**.

Wichtig:

Die `.proto`-Datei wird nur für die Entwicklung bzw. Generierung benötigt.  
Der Checkmk-Agent benötigt zur Laufzeit nicht die `.proto`-Datei, sondern nur die generierten Python-Dateien.

---

## Projektstruktur

Die aktuelle Projektstruktur ist so aufgebaut, dass Client- und Skriptlogik getrennt sind.

```text
pudis-client/
│
├─ README.md
├─ requirements.txt
├─ .gitignore
│
├─ protos/
│   └─ application_status.proto
│
├─ generated/
│   ├─ application_status_pb2.py
│   └─ application_status_pb2_grpc.py
│
├─ src/
│   └─ pudis_client/
│       ├─ main.py
│       ├─ config.py
│       │
│       ├─ clients/
│       │   └─ application_status_client.py
│       │
│       ├─ services/
│       │   └─ status_monitor_service.py
│       │
│       └─ mappers/
│           └─ status_mapper.py
│
├─ scripts/
│   ├─ checkmk/
│   │   └─ check_pudis_login.py
│   │
│   └─ pudis_station/
│
├─ deployment/
│   └─ checkmk_agent/
│       └─ pudis_login_check/
│           ├─ check_pudis_login.py
│           └─ lib/
│               ├─ application_status_pb2.py
│               └─ application_status_pb2_grpc.py
│
├─ tests/
└─ docs/
```

---

## Erklärung der wichtigsten Ordner

### `protos/`

Hier liegen die originalen `.proto`-Dateien.

Diese Dateien beschreiben die API und dienen als Grundlage für die Generierung des Python-gRPC-Codes.

Beispiel:

```text
protos/application_status.proto
```

---

### `generated/`

Hier liegen die automatisch generierten Python-Dateien.

Beispiel:

```text
generated/application_status_pb2.py
generated/application_status_pb2_grpc.py
```

Diese Dateien werden nicht manuell bearbeitet.

Wenn sich die `.proto` ändert, müssen diese Dateien neu generiert werden.

---

### `src/pudis_client/`

Hier liegt der clientbasierte Teil für das Prüfstandteam.

Dieser Teil ist für eine spätere benutzernahe Anwendung gedacht.

#### `main.py`

Startpunkt des Clients.

#### `clients/`

Technische Kommunikation mit der PUDIS-gRPC-API.

#### `services/`

Fachliche Logik, z. B.:

- Status überwachen
- Änderungen erkennen
- Ausgaben erzeugen

#### `mappers/`

Übersetzung technischer Werte in verständliche Texte.

Beispiel:

```text
READY → Application ist bereit
VCI_DETECTED → VCI erkannt
```

---

### `scripts/checkmk/`

Hier liegen Skripte für Checkmk.

Diese Skripte laufen nicht dauerhaft, sondern werden vom Checkmk-Agenten ausgeführt.

Ein Skript soll:

1. einmal laufen,
2. PUDIS abfragen,
3. eine Checkmk-konforme Zeile ausgeben,
4. sich beenden.

---

### `deployment/checkmk_agent/`

Hier liegt ein isoliertes Paket für den Checkmk-Agenten.

Dieses Paket ist dafür gedacht, an das Team bzw. an den Agenten weitergegeben zu werden.

Der Agent benötigt nicht die vollständige Projektstruktur, sondern nur:

```text
check_pudis_login.py
application_status_pb2.py
application_status_pb2_grpc.py
```

Empfohlene Struktur:

```text
pudis_login_check/
├─ check_pudis_login.py
└─ lib/
   ├─ application_status_pb2.py
   └─ application_status_pb2_grpc.py
```

---


## Unterschied zwischen Client und Checkmk-Skript

### Client

Der Client ist für das Prüfstandteam gedacht.

Eigenschaften:

- läuft dauerhaft,
- zeigt Statusänderungen lesbar an,
- ist benutzerorientiert,
- kann später mehrere Funktionen bündeln,
- kann erweitert werden.

Beispiel:

```text
src/pudis_client/main.py
```

### Checkmk-Skript

Das Skript ist für Checkmk bzw. den Agenten gedacht.

Eigenschaften:

- läuft einmal,
- gibt eine Statuszeile zurück,
- beendet sich direkt,
- ist monitoringorientiert,
- ist nicht für direkte Bedienung durch Benutzer gedacht.

Beispiel:

```text
scripts/checkmk/check_pudis_login.py
```

---

## Voraussetzungen

Damit das Projekt lokal entwickelt und getestet werden kann, werden folgende Voraussetzungen benötigt:

- Python 3.11 oder 3.12
- pip
- Visual Studio oder VS Code
- Zugriff auf PUDIS
- Zugriff auf die aktuellen `.proto`-Dateien

Benötigte Python-Bibliotheken für Entwicklung:

```text
grpcio
grpcio-tools
protobuf
```

Installation:

```bash
python -m pip install grpcio grpcio-tools protobuf
```

Für den Checkmk-Agenten werden zur Laufzeit nur benötigt:

```text
grpcio
protobuf
```

`grpcio-tools` wird nur zur Generierung der gRPC-Dateien benötigt.

---

## gRPC-Code generieren

Wenn sich die `.proto`-Datei ändert, muss der generierte Python-Code neu erstellt werden.

Befehl im Projekt-Hauptordner:

```bash
python -m grpc_tools.protoc -I./protos --python_out=./generated --grpc_python_out=./generated ./protos/application_status.proto
```

Danach werden aktualisiert:

```text
generated/application_status_pb2.py
generated/application_status_pb2_grpc.py
```

Nach der Generierung sollte geprüft werden, ob das benötigte Feld vorhanden ist:

```text
authenticated_user
```

---

## Deployment für Checkmk-Agent

Für den Checkmk-Agenten wird ein isoliertes Paket erstellt.

Beispiel:

```text
deployment/checkmk_agent/pudis_login_check/
├─ check_pudis_login.py
└─ lib/
   ├─ application_status_pb2.py
   └─ application_status_pb2_grpc.py
```

Die `.proto`-Datei wird nicht mitgegeben, da sie nur zur Codegenerierung benötigt wird.

Wichtig:

Nur `check_pudis_login.py` soll vom Agenten ausgeführt werden.  
Die Dateien im `lib/`-Ordner werden nur importiert.

---

## Wichtige Hinweise

- Wenn PUDIS geschlossen ist, ist der gRPC-Server nicht erreichbar.
- Wenn eine Methode oder ein Feld im Python-Code fehlt, ist meist der generierte Code veraltet.
- Nach Änderungen an `.proto`-Dateien muss der Code erneut generiert werden.
- Die generierten `pb2`-Dateien dürfen nicht manuell bearbeitet werden.
- Der Checkmk-Agent benötigt nicht die gesamte Projektstruktur.
- Für den Agenten sollte ein isoliertes Deployment-Paket verwendet werden.

---

---

## Zusammenfassung

Das Projekt verfolgt zwei Ziele:

1. **Einen Client für das Prüfstandteam aufbauen**, der bestimmte Aufgaben rund um PUDIS vereinfacht.
2. **Parallele scriptbasierte Funktionen bereitstellen**, damit Informationen oder Zustände über Checkmk bzw. PUDIS-Stationen genutzt werden können.


Damit entsteht eine kombinierte Lösung aus:

- benutzernahem Client für das Prüfstandteam
- technischen Skripten für Checkmk / PUDIS-Stationen
