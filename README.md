# 🚗 PUDIS Client

## Project Overview

This project provides a Python-based client library for **PUDIS**.

PUDIS is used at test benches to access vehicle diagnostic information and start diagnostic procedures, such as reading a vehicle protocol.

The goal of the project is to provide a reusable Python interface that simplifies and automates common PUDIS workflows for the test bench team.

The PUDIS Client can be integrated into an external Python application or test bench automation software.

---

# Project Goals

The main objective is to develop a reusable Python client for the test bench team.

The client should:

- Provide access to the PUDIS gRPC API
- Display important PUDIS states in a readable format
- Monitor status changes
- Start and close PUDIS
- Start a vehicle protocol readout
- Provide structured status information
- Support integration into external automation software
- Provide a modular foundation for future diagnostic functions

---

# Current Functionality

The following functions are currently implemented.

## Application Status Monitoring

The client can monitor the current PUDIS application state.

Supported states include:

- Application status unknown
- System integrity check running
- System integrity check failed
- System integrity checks completed
- Application ready
- Procedure running

The status can be monitored continuously or requested as a single status snapshot.

---

## Connection Status Monitoring

The client can monitor the current vehicle connection state.

Supported states include:

- Connection status unknown
- Not connected
- VCI detected
- DoIP active
- Vehicle connected

Additional connection information includes:

- Battery voltage
- Ignition state

---

## Login Status Monitoring

The client evaluates the authenticated user information provided by PUDIS.

The client can determine:

- Whether a user is logged in
- The username of the authenticated user
- Whether no user is currently logged in
- Information about the most recent login attempt, if available

The login status is determined through the following PUDIS field:

```proto
User authenticated_user = 5;
```

A user is considered logged in when `authenticated_user.name` contains a non-empty username.

---

## PUDIS Startup and Shutdown

The client can start and close PUDIS.

PUDIS is started through the official Windows shortcut:

```text
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\PUDIS Gen. 5.lnk
```

Using the official shortcut ensures that PUDIS starts with the required Windows application context.

The client can also close the running PUDIS process.

Available methods:

```python
client.start_pudis()
client.close_pudis()
```

---

## Handling an Unavailable PUDIS Server

The client can be started before PUDIS is running.

If the PUDIS gRPC interface is temporarily unavailable:

- The client does not terminate
- The status monitor continues running
- The client repeatedly attempts to reconnect
- Monitoring continues automatically when PUDIS becomes available

This allows external automation software to start the client before starting PUDIS.

---

## Waiting for PUDIS Readiness

The client provides a method that waits until:

- The PUDIS gRPC interface is reachable
- The application state is `READY`
- A user is logged in

Example:

```python
ready = client.wait_until_ready(
    max_wait_seconds=180,
    poll_interval_seconds=3,
)
```

The method returns:

```python
True
```

when PUDIS is ready, or:

```python
False
```

when the configured timeout is reached.

---

## Status Snapshot

The client can return the current PUDIS state as a structured Python dictionary.

Example:

```python
status = client.get_status_snapshot()
print(status)
```

Example result:

```python
{
    "application_state": 4,
    "application_state_text": "Application ready",
    "connection_state": 2,
    "connection_state_text": "VCI detected",
    "logged_in": True,
    "username": "A4013EM",
    "battery_voltage": 12.6,
    "ignition_on": True,
}
```

The structured result can be processed directly by external automation software.

---

## Vehicle Protocol Readout

The client can start a vehicle protocol readout through the PUDIS `ProcedureService`.

The following RPC method is used:

```text
StartReadVal
```

The public client method is:

```python
client.read_vehicle_protocol()
```

The Python client prepares the request and sends it to PUDIS. The actual diagnostic procedure is executed by the PUDIS server.

---

## VIN Handling

An important implementation detail concerns the VIN field in `StartReadValRequest`.

The client must not explicitly send an empty VIN:

```python
vin=""
```

If an empty VIN is explicitly included in the request, PUDIS may not use the automatically detected vehicle VIN.

Instead, the VIN field is omitted completely.

PUDIS then uses the VIN detected from the currently connected vehicle.

The request is therefore created without explicitly setting the VIN:

```python
request = procedure_control_pb2.StartReadValRequest()
```

Logical links are only added when values are provided.

---

## Running Procedure Monitoring

PUDIS provides the following RPC method for monitoring an active procedure:

```text
WatchRunningProcedure
```

A running procedure can provide:

- Procedure identifier
- Current diagnostic step
- Start time
- Progress percentage
- Vehicle type
- VIN
- Procedure status
- Generated vehicle protocol file path

Supported procedure states include:

- Running
- Finished
- Cancelled
- Failed

This functionality provides the technical foundation for displaying and processing the progress of a vehicle protocol readout.

---

# Integration into External Automation Software

The PUDIS Client is available as an installable Python library.

After installation, another Python application can import the public `PudisClient` class.

```python
from pudis_client import PudisClient

client = PudisClient()
```

The automation software can then call the required methods directly:

```python
client.start_pudis()

ready = client.wait_until_ready(
    max_wait_seconds=180,
    poll_interval_seconds=3,
)

if ready:
    status = client.get_status_snapshot()
    print(status)

    client.read_vehicle_protocol()
```

No interactive menu is required for this integration.

The file `library_test.py` provides a complete example and simulates the later call sequence of the test bench automation software.

---

# Available Public Methods

The public `PudisClient` interface currently provides:

```python
client.start_pudis()
client.close_pudis()
client.wait_until_ready()
client.get_status_snapshot()
client.read_vehicle_protocol()
```

## `start_pudis()`

Starts the locally installed PUDIS application through the configured Windows shortcut.

## `close_pudis()`

Closes the running PUDIS application.

## `wait_until_ready()`

Waits until PUDIS is reachable, the application state is `READY`, and a user is logged in.

## `get_status_snapshot()`

Returns the current application, connection, login, battery and ignition status as a Python dictionary.

## `read_vehicle_protocol()`

Starts the vehicle protocol readout through `StartReadVal`.

---

# Technical Background

## gRPC

PUDIS provides a gRPC API.

The PUDIS Client uses this API to:

- Request current status information
- Monitor state changes
- Start diagnostic procedures
- Monitor running procedures

The connection is currently configured as:

```text
Host: localhost
Port: 5050
```

PUDIS and the Python client therefore run on the same computer in the current environment.

---

# Role of the Proto Files

The `.proto` files define the PUDIS gRPC interface.

They describe:

- Available services
- RPC methods
- Request messages
- Response messages
- Status enums
- Data structures

Currently used services include:

```text
ApplicationStatusService
ProcedureService
```

Relevant RPC methods include:

```text
GetApplicationStatus
WatchApplicationStatus
GetConnectionStatus
WatchConnectionStatus
StartReadVal
GetRunningProcedure
WatchRunningProcedure
```

---

# Generated Python Code

Python files are generated from the `.proto` definitions.

The generated files include:

```text
application_status_pb2.py
application_status_pb2_grpc.py
procedure_control_pb2.py
procedure_control_pb2_grpc.py
```

The generated files provide:

- Python message classes
- Enums
- Request and response structures
- gRPC service stubs

The generated files should not be edited manually, except for package-specific import adjustments when required by the Python package structure.

The original `.proto` files are required during development and code generation.

They are not required at runtime when the generated Python modules are already included in the installed library.

---

# Architecture

The client follows a layered architecture.

## Public API Layer

The public API layer provides the `PudisClient` class.

File:

```text
src/pudis_client/pudis_client_api.py
```

This class is the main interface for external automation software.

---

## Service Layer

The service layer contains the workflow and business logic.

Examples:

- Checking whether a user is logged in
- Checking whether PUDIS is ready
- Starting a vehicle protocol
- Starting and closing PUDIS
- Monitoring status changes

Files:

```text
src/pudis_client/services/
```

---

## Client Layer

The client layer contains the direct technical gRPC calls.

Examples:

- `GetApplicationStatus`
- `GetConnectionStatus`
- `StartReadVal`
- `WatchRunningProcedure`

Files:

```text
src/pudis_client/clients/
```

---

## Mapper Layer

The mapper layer converts technical API values into human-readable status messages.

Examples:

```text
READY → Application ready
VCI_DETECTED → VCI detected
DOIP → Vehicle connected
```

Files:

```text
src/pudis_client/mappers/
```

---

## Generated Layer

The generated layer contains the Python modules generated from the PUDIS `.proto` files.

Files:

```text
src/pudis_client/generated/
```

---

# Project Structure

```text
pudis-client/
│
├─ README.md
├─ pyproject.toml
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
│       ├─ __init__.py
│       ├─ main.py
│       ├─ config.py
│       ├─ pudis_client_api.py
│       │
│       ├─ clients/
│       │   ├─ __init__.py
│       │   ├─ application_status_client.py
│       │   └─ procedure_client.py
│       │
│       ├─ services/
│       │   ├─ __init__.py
│       │   ├─ status_monitor_service.py
│       │   ├─ procedure_service.py
│       │   └─ pudis_lifecycle_service.py
│       │
│       ├─ mappers/
│       │   ├─ __init__.py
│       │   └─ status_mapper.py
│       │
│       └─ generated/
│           ├─ __init__.py
│           ├─ application_status_pb2.py
│           ├─ application_status_pb2_grpc.py
│           ├─ procedure_control_pb2.py
│           └─ procedure_control_pb2_grpc.py
│
├─ examples/
│   └─ automation_test.py
│
├─ tests/
├─ docs/
│
├─ build/
└─ dist/
    ├─ pudis_client-0.1.0-py3-none-any.whl
    └─ pudis_client-0.1.0.tar.gz
```

---

# Folder Description

## `protos/`

Contains the original PUDIS gRPC API definitions.

```text
application_status.proto
procedure_control.proto
```

---

## `generated/`

Contains the generated files used during development.

---

## `src/pudis_client/`

Contains the installable Python package.

This directory is the source used to build the Wheel and Source Distribution.

---

## `src/pudis_client/generated/`

Contains the generated gRPC modules included in the installable library.

---

## `src/pudis_client/clients/`

Contains the direct PUDIS gRPC communication.

---

## `src/pudis_client/services/`

Contains the workflow and business logic.

---

## `src/pudis_client/mappers/`

Contains status and output mappings.

---

## `examples/`

Contains example scripts that demonstrate how another Python application can call the PUDIS Client.

---

## `dist/`

Contains the built Python packages:

```text
pudis_client-0.1.0-py3-none-any.whl
pudis_client-0.1.0.tar.gz
```

The Wheel is the recommended installation format.

---

# Requirements

## Development Requirements

- Python 3.11 or newer
- pip
- Visual Studio or VS Code
- Access to PUDIS
- Access to the current PUDIS `.proto` files

Required Python packages:

```text
grpcio
grpcio-tools
protobuf
build
```

Installation:

```bash
python -m pip install grpcio grpcio-tools protobuf build
```

## Runtime Requirements

The installed Library requires:

```text
grpcio
protobuf
```

These dependencies are installed automatically when the Wheel is installed with `pip`.

The target computer also requires:

- PUDIS Gen. 5
- PUDIS gRPC interface on `localhost:5050`
- VCI drivers
- Test bench simulator or vehicle connection
- Required Windows permissions

---

# Generating the gRPC Code

Whenever a `.proto` file changes, the generated Python files must be regenerated.

Application status:

```powershell
python -m grpc_tools.protoc `
    -I./protos `
    --python_out=./generated `
    --grpc_python_out=./generated `
    ./protos/application_status.proto
```

Procedure control:

```powershell
python -m grpc_tools.protoc `
    -I./protos `
    --python_out=./generated `
    --grpc_python_out=./generated `
    ./protos/procedure_control.proto
```

After generation, the updated files must also be copied into:

```text
src/pudis_client/generated/
```

The package-specific imports in the generated gRPC files must then be verified.

---

# Building the Python Library

The package configuration is defined in:

```text
pyproject.toml
```

Install the build tool:

```powershell
python -m pip install --upgrade build
```

Build the Wheel and Source Distribution:

```powershell
python -m build
```

The generated packages are stored in:

```text
dist/
```

---

# Installing the Library

Install the Wheel with:

```powershell
python -m pip install .\dist\pudis_client-0.1.0-py3-none-any.whl
```

Verify the installation:

```powershell
python -m pip show pudis-client
```

Test the import:

```powershell
python -c "from pudis_client import PudisClient; print(PudisClient)"
```

---

# Testing the Installed Library

The file:

```text
library_test.py
```

provides a functional test for the installed PUDIS Client library.

Run:

```powershell
python .\library_test.py
```

The test performs the following steps:

1. Starts PUDIS
2. Waits until PUDIS is ready and a user is logged in
3. Reads the current status
4. Waits for the VCI and test bench simulator connection
5. Starts the vehicle protocol readout

---

# Important Notes

- PUDIS must be running for the gRPC API to be available.
- The client can be started before PUDIS and will wait for availability.
- PUDIS and the client currently use `localhost:5050`.
- Generated `pb2` files should not be modified manually.
- Changes to `.proto` files require regeneration of the Python code.
- The VIN must not be sent as an empty string.
- If the VIN field is omitted, PUDIS uses the VIN of the connected vehicle.
- A connected VCI and vehicle or simulator are required before starting the vehicle protocol.
- The interactive menu is only intended for development and manual testing.
- External automation software should use the public `PudisClient` methods.

---

# Future Extensions

Potential future extensions include:

- Additional diagnostic procedures
- Delete DTC
- Delete DTC with OBD
- Extended vehicle information
- Detailed precondition monitoring
- Additional PUDIS application settings
- Structured procedure result objects
- Defined exceptions for automation software
- Configurable PUDIS host and port
- Configurable PUDIS startup path
- Additional test bench automation workflows

---

# Summary

The PUDIS Client provides a reusable Python interface for PUDIS.

The current implementation supports:

- Starting and closing PUDIS
- Waiting for PUDIS readiness
- Application status monitoring
- Connection status monitoring
- Login status monitoring
- Structured status snapshots
- Starting a vehicle protocol readout
- Integration into external Python automation software
- Distribution as an installable Python Wheel

The modular architecture allows the client to be extended with additional diagnostic and automation functions in future versions.
- Running procedure monitoring
- Automatic PUDIS startup

The project is designed with a modular structure and can be extended with additional diagnostic and automation features in the future.
