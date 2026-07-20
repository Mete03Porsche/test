# 🚗 PUDIS Client

## Project Overview

This project develops a Python-based client for **PUDIS**.

PUDIS is used at test benches to generate vehicle diagnostic protocols and provide information about the current diagnostic environment.

The goal of this project is to create a client that supports the test bench team by simplifying and automating common workflows.

The client provides a user-friendly interface for monitoring system states and triggering diagnostic processes through the PUDIS gRPC API.

---

# Project Goals

The main objective is to develop a client for the test bench team.

The client should:

- Display important PUDIS states in a readable way
- Monitor status changes
- Bundle multiple diagnostic functions in one tool
- Simplify workflows at test benches
- Provide a foundation for future automation features

---

# Technical Background

PUDIS provides a **gRPC API**.

The Python client communicates with this API and retrieves information from PUDIS.

The API is defined through `.proto` files.

From these `.proto` files, Python gRPC code is automatically generated and used by the client.

---

# Current Functionality

The client currently supports the following features:

## Application Status Monitoring

The client can monitor:

- System self-checks
- Application ready state
- Procedure running state
- System errors

---

## Connection Status Monitoring

The client can monitor:

- Not connected
- VCI detected
- Vehicle connected

---

## Login Status Monitoring

The client checks the authenticated user information provided by PUDIS.

The client can determine whether:

- A user is logged in
- No user is logged in

---

## Vehicle Protocol Readout (VAL)

The client can start the process of generating a vehicle protocol.

The implementation uses the PUDIS ProcedureService and the `StartReadVal` method.

Important implementation detail:

- The VIN must not be sent as an empty string.
- If the VIN field is omitted completely, PUDIS automatically uses the VIN of the connected vehicle.

The vehicle protocol readout process is currently working successfully.

---

## Running Procedure Monitoring

The client can monitor running procedures through:

```text
WatchRunningProcedure
```

The following information can be displayed:

- Procedure status
- Current diagnostic step
- Progress percentage
- VIN (if available)
- Generated VAL file path

---

## PUDIS Startup

The client can start PUDIS automatically.

This allows the client to be started even if PUDIS is currently closed.

If PUDIS is unavailable:

- The client remains running
- The application does not crash
- The user can start PUDIS directly from the client

PUDIS is started using the official Windows shortcut.

---

# Architecture

The application is divided into multiple layers.

## PUDIS

PUDIS acts as the server and exposes the gRPC API.

## Generated Layer

The generated layer contains gRPC code created from the `.proto` files.

## Client Layer

The client layer handles communication with the PUDIS API.

## Service Layer

The service layer contains the business logic.

Examples:

- Status monitoring
- Vehicle protocol readout
- PUDIS lifecycle handling

## Application Layer

The application layer contains:

```text
main.py
```

which acts as the entry point of the client.

---

# Role of the Proto Files

The `.proto` files define:

- Services
- RPC methods
- Request messages
- Response messages

Examples:

```text
ApplicationStatusService
ProcedureService
GetApplicationStatus
GetConnectionStatus
StartReadVal
WatchRunningProcedure
```

The `.proto` files are only required for code generation.

The generated Python files are used during runtime.

---

# Project Structure

```text
pudis-client/
│
├─ README.md
├─ requirements.txt
├─ .gitignore
│
├─ protos/
│   ├─ application_status.proto
│   └─ procedure_control.proto
│
├─ generated/
│   ├─ application_status_pb2.py
│   ├─ application_status_pb2_grpc.py
│   ├─ procedure_control_pb2.py
│   └─ procedure_control_pb2_grpc.py
│
├─ src/
│   └─ pudis_client/
│       ├─ main.py
│       ├─ config.py
│       │
│       ├─ clients/
│       │   ├─ application_status_client.py
│       │   └─ procedure_client.py
│       │
│       ├─ services/
│       │   ├─ status_monitor_service.py
│       │   ├─ procedure_service.py
│       │   └─ pudis_lifecycle_service.py
│       │
│       └─ mappers/
│           └─ status_mapper.py
│
├─ scripts/
│   └─ checkmk/
│
├─ tests/
└─ docs/
```

---

# Folder Description

## protos/

Contains the original gRPC API definitions.

Examples:

```text
application_status.proto
procedure_control.proto
```

---

## generated/

Contains the generated Python gRPC code.

Examples:

```text
application_status_pb2.py
application_status_pb2_grpc.py
```

These files are automatically generated and should not be edited manually.

---

## src/pudis_client/

Contains the main client implementation.

### main.py

Entry point of the application.

### clients/

Contains the API communication layer.

Examples:

- GetApplicationStatus
- GetConnectionStatus
- StartReadVal

### services/

Contains the business logic.

Examples:

- Status monitoring
- Vehicle protocol handling
- PUDIS startup handling

### mappers/

Converts technical API values into human-readable text.

Example:

```text
READY → Application Ready
VCI_DETECTED → VCI Detected
```

---

# Requirements

Required software:

- Python 3.11 or 3.12
- pip
- Visual Studio or VS Code
- Access to PUDIS
- Required `.proto` files

Required Python packages:

```text
grpcio
grpcio-tools
protobuf
```

Installation:

```bash
python -m pip install grpcio grpcio-tools protobuf
```

---

# Generating gRPC Code

Whenever a `.proto` file changes, the Python code must be regenerated.

Example:

```bash
python -m grpc_tools.protoc ^
-I./protos ^
--python_out=./generated ^
--grpc_python_out=./generated ^
./protos/application_status.proto
```

The same process applies to:

```text
procedure_control.proto
```

---

# Important Notes

- PUDIS must be running for the gRPC API to be available.
- If PUDIS is unavailable, the client remains active and continues trying to reconnect.
- Generated `pb2` files must never be modified manually.
- Changes to `.proto` files require regeneration of the Python code.
- PUDIS can be started directly from the client.
- Vehicle protocol generation is implemented using `StartReadVal`.
- If the VIN field is omitted, PUDIS uses the VIN from the connected vehicle automatically.

---

# Future Extensions

Potential future features include:

- Delete DTC
- Delete DTC with OBD
- Additional procedure handling
- Vehicle information monitoring
- Extended application settings
- Additional automation workflows
- Enhanced test bench functions

---

# Summary

The project aims to provide a user-friendly client for PUDIS.

Current functionality includes:

- Application status monitoring
- Connection status monitoring
- Login status monitoring
- Vehicle protocol generation
- Running procedure monitoring
- Automatic PUDIS startup

The project is designed with a modular structure and can be extended with additional diagnostic and automation features in the future.
