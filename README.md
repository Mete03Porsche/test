# 🚗 PUDIS Client – Projektdokumentation

> ℹ️ **Zweck dieser Seite**  
> Diese Seite beschreibt **die Projektidee**, **das Zielbild**, **die Voraussetzungen**, **das technische Grundprinzip** und **die empfohlene Projektstruktur** des PUDIS-Clients.  
> Die Seite ist bewusst **allgemein gehalten** und **nicht** als Ablauf- oder Fortschrittsdokumentation gedacht.

---

## 🎯 1. Projektidee

An den Prüfständen werden über PUDIS **Fahrzeugprotokolle** für Fahrzeuge gezogen.  
Das Team vor Ort arbeitet dabei direkt mit der PUDIS-Anwendung.

Ziel dieses Projekts ist es, einen **Python-basierten Client** zu entwickeln, der das Team bei diesen Aufgaben unterstützt.

Der Client soll langfristig als **zentrale Arbeitsoberfläche bzw. technisches Hilfswerkzeug** dienen, damit wiederkehrende Aufgaben **nicht jedes Mal direkt manuell in PUDIS** durchgeführt werden müssen.-!

### 🧩 Vereinfacht gesagt

- **PUDIS** stellt Funktionen und Zustände bereit.
- **Der Client** greift auf diese Funktionen zu.
- **Das Team** soll später möglichst über den Client arbeiten können.

---

## 🏁 2. Ziel des Projekts

Der Client soll so aufgebaut werden, dass er **erweiterbar** ist und nach und nach weitere Aufgaben übernehmen kann.

### ✅ Geplantes Zielbild

Der Client soll künftig dabei helfen,

- Zustände aus PUDIS abzufragen,
- Änderungen live zu überwachen,
- Arbeitsschritte für das Team zu vereinfachen,
- wiederkehrende Aufgaben zu bündeln,
- spätere Funktionen modular zu ergänzen.

### 🔍 Aktueller fachlicher Fokus

Der erste sinnvolle Baustein ist die Überwachung von:

- **ApplicationStatus**
- **ConnectionStatus**

Damit lässt sich bereits erkennen,

- ob die Anwendung bereit ist,
- ob gerade eine Prozedur läuft,
- ob eine Verbindung besteht,
- ob ein VCI erkannt wurde.

### 🚀 Perspektivisch mögliche Erweiterungen

Je nach Abstimmung mit dem Team kann der Client später zusätzlich unterstützen bei:

- Authentifizierung
- Überwachung von Prozeduren
- Nutzung weiterer PUDIS-Services
- Logging / Nachverfolgung
- Konfigurierbarkeit
- zusätzlicher Automatisierungslogik

---

## 🧠 3. Fachlicher Hintergrund

PUDIS stellt eine **gRPC-API** bereit.  
Über diese API kann ein externer Client Informationen abfragen oder Zustandsänderungen beobachten.

### 📡 Beispiele für relevante Informationen

#### ApplicationStatus
Zeigt den Zustand der Anwendung, z. B.:
- bereit
- Prozedur läuft
- weitere definierte Zustände

#### ConnectionStatus
Zeigt den Zustand der Verbindung, z. B.:
- nicht verbunden
- VCI erkannt
- DoIP aktiv

Die Schnittstelle wird über **`.proto`-Dateien** beschrieben.  
Aus diesen Dateien wird der Python-Code erzeugt, der für die gRPC-Kommunikation benötigt wird.

---

## ⚙️ 4. Technisches Grundprinzip

### 👥 4.1 Rollenverständnis

- **PUDIS = Server**
- **Python-Projekt = Client**

Der Client sendet Anfragen an PUDIS oder beobachtet Änderungen live.

### 📄 4.2 Rolle der `.proto`-Dateien

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

### 🏗️ 4.3 Generierter Code

Aus den `.proto`-Dateien werden Python-Dateien erzeugt:

- `*_pb2.py`
- `*_pb2_grpc.py`

Diese Dateien sind **generierter Code** und werden **nicht manuell angepasst**.

### 🧱 4.4 Logischer Aufbau des Clients

Der Client lässt sich in mehrere Ebenen unterteilen:

1. **Proto-Ebene** → beschreibt die API  
2. **Generated Code** → technische Python-Schnittstelle  
3. **Client-Schicht** → direkte Kommunikation mit PUDIS  
4. **Service-Schicht** → fachliche Logik  
5. **Startpunkt / Main** → Ausführung des Tools  

---

## 🛠️ 5. Voraussetzungen

Damit ein neuer Praktikant oder Entwickler den Client lokal einrichten und ausführen kann, werden folgende Voraussetzungen benötigt.

### 💻 5.1 Software

- **Python 3.11 oder 3.12**
- **pip**
- **Visual Studio** oder **VS Code**
- Zugriff auf die **PUDIS-Anwendung**
- Zugriff auf die **aktuellen `.proto`-Dateien**

### 📦 5.2 Benötigte Python-Bibliotheken

Aktuell werden mindestens folgende Bibliotheken benötigt:

- `grpcio`
- `grpcio-tools`
- `protobuf`

### ⬇️ 5.3 Installation der Bibliotheken

```bash
python -m pip install grpcio grpcio-tools protobuf
```

### 🌐 5.4 Laufzeitvoraussetzungen

Vor dem Start des Clients muss bekannt sein:

- auf welcher Adresse der gRPC-Server läuft,
- welcher Port verwendet wird.

Aktuell bekannt:

- **Host:** `localhost`
- **Port:** `5050`

### ❗ 5.5 Wichtiger Hinweis

PUDIS muss laufen, damit sich der Client verbinden kann.  
Wenn PUDIS geschlossen ist, ist der gRPC-Server nicht erreichbar.

---

## 🚀 6. Einrichtung für neue Praktikanten / Entwickler

### 1️⃣ Projekt lokal bereitstellen
Repository bzw. Projektordner lokal öffnen.

### 2️⃣ Python installieren
Empfohlen: Python 3.11 oder 3.12

### 3️⃣ Python-Abhängigkeiten installieren

```bash
python -m pip install grpcio grpcio-tools protobuf
```

oder – falls vorhanden – über eine `requirements.txt`:

```bash
python -m pip install -r requirements.txt
```

### 4️⃣ `.proto`-Dateien prüfen
Sicherstellen, dass die aktuellen und richtigen `.proto`-Dateien vorhanden sind.

### 5️⃣ gRPC-Code generieren
Beispiel für `application_status.proto`:

```bash
python -m grpc_tools.protoc -I./protos --python_out=./generated --grpc_python_out=./generated ./protos/application_status.proto
```

### 6️⃣ PUDIS starten
Ohne laufende PUDIS-Anwendung kann der Client nicht funktionieren.

### 7️⃣ Konfiguration prüfen
Host und Port kontrollieren.

### 8️⃣ Client starten
Beispiel:

```bash
python src/main.py
```

---

## 🗂️ 7. Empfohlene Projektstruktur

Damit das Projekt mit weiteren Anforderungen wachsen kann, sollte es modular aufgebaut werden.

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
│   ├─ authentication_pb2.py
│   ├─ authentication_pb2_grpc.py
│   ├─ procedure_control_pb2.py
│   ├─ procedure_control_pb2_grpc.py
│   ├─ data_distributions_pb2.py
│   └─ data_distributions_pb2_grpc.py
│
├─ src/
│   ├─ main.py
│   ├─ config.py
│   │
│   ├─ clients/
│   │   ├─ grpc_channel.py
│   │   ├─ application_status_client.py
│   │   ├─ authentication_client.py
│   │   ├─ procedure_client.py
│   │   └─ data_distribution_client.py
│   │
│   ├─ services/
│   │   ├─ application_status_service.py
│   │   ├─ connection_status_service.py
│   │   ├─ authentication_service.py
│   │   ├─ procedure_service.py
│   │   └─ data_distribution_service.py
│   │
│   ├─ mappers/
│   │   ├─ application_status_mapper.py
│   │   ├─ connection_status_mapper.py
│   │   └─ procedure_status_mapper.py
│   │
│   └─ utils/
│       ├─ console_output.py
│       ├─ logger.py
│       └─ helpers.py
│
├─ tests/
│   ├─ manual_test_cases.md
│   ├─ application_status_tests.md
│   ├─ connection_status_tests.md
│   └─ procedure_tests.md
│
└─ docs/
    ├─ setup.md
    ├─ architecture.md
    └─ api_notes.md
```

---

## 🧭 8. Erklärung der Struktur

### `README.md`
Kurzer Einstieg ins Projekt.

Soll enthalten:
- worum es im Projekt geht,
- was das Ziel ist,
- welche Voraussetzungen nötig sind,
- wie das Projekt gestartet wird.

### `protos/`
Enthält die originalen `.proto`-Dateien.

### `generated/`
Enthält den automatisch generierten Python-Code aus den `.proto`-Dateien.

### `src/`
Enthält den selbst geschriebenen Python-Code.

#### `src/main.py`
Startpunkt des Clients.

#### `src/config.py`
Zentrale Konfiguration, z. B. Host und Port.

#### `src/clients/`
Direkte technische gRPC-Kommunikation.

#### `src/services/`
Fachliche Logik, z. B.:
- Status bewerten
- lesbare Meldungen erzeugen
- Regeln für Abläufe formulieren

#### `src/mappers/`
Zuordnung technischer Zustände zu lesbaren Texten.

#### `src/utils/`
Hilfsfunktionen, z. B. für Logging oder Konsolenausgabe.

### `tests/`
Testfälle und ggf. spätere automatisierte Tests.

### `docs/`
Ausführlichere technische Dokumentation.

---

## 📌 9. Wichtige Hinweise

- Wenn PUDIS geschlossen ist, ist der Server nicht erreichbar.
- Streams können denselben Zustand mehrfach senden.
- Solche Zustände sollten im Client dedupliziert werden.
- Wenn eine Methode im generierten Python-Code fehlt, passt meist die `.proto`-Datei nicht zum generierten Stand.
- Änderungen an `.proto`-Dateien erfordern eine erneute Generierung des Python-Codes.

---

## ❓ 10. Offene Punkte

Da die finalen Anforderungen noch mit dem Team abgestimmt werden, bleiben aktuell noch Fragen offen.

Beispiele:

- Welche Aufgaben soll der Client final vollständig unterstützen?
- Soll der Client später nur Informationen anzeigen oder auch Aktionen starten?
- Wird zusätzlich Authentifizierung benötigt?
- Welche weiteren Services sollen eingebunden werden?
- Soll Logging oder Export vorgesehen werden?

---

## 📝 11. Zusammenfassung

Der Client ist als **erweiterbares Hilfswerkzeug für das Team an den Prüfständen** gedacht.

Das Ziel ist, wiederkehrende Aufgaben rund um PUDIS **zentral über einen eigenen Client** abbilden zu können, anstatt alle Arbeitsschritte direkt manuell in PUDIS auszuführen.

Diese Dokumentation beschreibt deshalb bewusst:

- den fachlichen Zweck,
- das Zielbild,
- die Voraussetzungen,
- das technische Grundprinzip,
- und die empfohlene Projektstruktur.

