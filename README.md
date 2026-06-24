
# 🚗 PUDIS / Checkmk Client

## Worum geht es in diesem Projekt?

Dieses Projekt entwickelt eine kombinierte Lösung aus **Client** und **Skripten** für PUDIS bzw. Checkmk.

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

## 🔍 Aktueller fachlicher Fokus

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

## 🚀 Perspektivisch mögliche Erweiterungen

Je nach Abstimmung mit dem Team kann das Projekt später zusätzlich erweitert werden um:

- Authentifizierung
- Überwachung oder Steuerung von Prozeduren
- Nutzung weiterer PUDIS-Services
- Logging / Nachverfolgung
- Konfigurierbarkeit
- Checkmk-spezifische Skripte
- zusätzliche Automatisierungslogik

---

## 🧠 Fachlicher Hintergrund

PUDIS stellt eine **gRPC-API** bereit.  
Über diese API kann ein externer Client Informationen abfragen oder Zustandsänderungen beobachten.

### 📡 Beispiele für relevante Informationen

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

## ⚙️ Technisches Grundprinzip

### 👥 Rollenverständnis

- **PUDIS = Server / API-Anbieter**
- **Client = Benutzernahe Lösung für das Prüfstandteam**
- **Skripte = Technische Lösung für Checkmk / PUDIS-Stationen**

### 📄 Rolle der `.proto`-Dateien

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

### 🏗️ Generierter Code

Aus den `.proto`-Dateien werden Python-Dateien erzeugt:

- `*_pb2.py`
- `*_pb2_grpc.py`

Diese Dateien sind **generierter Code** und werden **nicht manuell angepasst**.

### 🧱 Logischer Aufbau des Projekts

Das Projekt lässt sich in mehrere Ebenen unterteilen:

1. **Proto-Ebene** → beschreibt die API  
2. **Generated Code** → technische Python-Schnittstelle  
3. **Client-Schicht** → direkte Kommunikation mit PUDIS  
4. **Service-Schicht** → fachliche Logik für den Client  
5. **Script-Schicht** → scriptbasierte Funktionen für Checkmk / PUDIS-Stationen  
6. **Startpunkt / Main** → Ausführung des Clients oder der Skripte  

---

## 🛠️ Voraussetzungen

Damit ein neuer Praktikant oder Entwickler den Client lokal einrichten und ausführen kann, werden folgende Voraussetzungen benötigt.

### 💻 Software

- **Python 3.11 oder 3.12**
- **pip**
- **Visual Studio** oder **VS Code**
- Zugriff auf die **PUDIS-Anwendung**
- Zugriff auf die **aktuellen `.proto`-Dateien**
- falls relevant: Zugriff auf **Checkmk** bzw. die benötigte Umgebung

### 📦 Benötigte Python-Bibliotheken

Aktuell werden mindestens folgende Bibliotheken benötigt:

- `grpcio`
- `grpcio-tools`
- `protobuf`

### ⬇️ Installation der Bibliotheken

```bash
python -m pip install grpcio grpcio-tools protobuf
