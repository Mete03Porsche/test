# PUDIS Client

## Project Overview

The PUDIS Client is an installable Python library for interacting with the PUDIS gRPC API.

PUDIS is used at test benches to access diagnostic information and start diagnostic procedures, such as reading a vehicle protocol.

The library provides a reusable interface that can be integrated into external Python applications and test bench automation software.

---

## Project Goals

The main goals of the PUDIS Client are:

- Provide access to the PUDIS gRPC API
- Monitor important PUDIS states
- Retrieve structured status information
- Detect the current login status
- Start and close PUDIS
- Wait until PUDIS is ready
- Start a vehicle protocol readout
- Support integration into external automation software
- Provide a modular foundation for future diagnostic functions

---

## Current Functionality

### Application Status

The client can read and describe the current PUDIS application state.

Supported states include:

- Application status unknown
- System integrity check running
- System integrity check failed
- System integrity checks completed
- Application ready
- Procedure running

### Connection Status

The client can read the current vehicle connection state.

Supported states include:

- Connection status unknown
- Not connected
- VCI detected
- DoIP active
- Vehicle connected

Additional information includes:

- Battery voltage
- Ignition state

### Login Status

The client evaluates the authenticated user information provided by PUDIS.

A user is considered logged in when:

```proto
authenticated_user.name
```

contains a non-empty username.

The client can provide:

- Login state
- Authenticated username
- Information about the most recent login attempt, if available

### PUDIS Startup and Shutdown

The client can start and close PUDIS.

Available methods:

```python
client.start_pudis()
client.close_pudis()
```

PUDIS is started through the configured Windows shortcut:

```text
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\PUDIS Gen. 5.lnk
```

Using the official shortcut ensures that PUDIS starts with the required Windows application context.

### Handling an Unavailable PUDIS Server

The client can be started before PUDIS is running.

If the PUDIS gRPC interface is temporarily unavailable:

- The client remains active
- The status monitor does not terminate
- The client continues trying to reconnect
- Monitoring continues automatically when PUDIS becomes available

### Waiting for PUDIS Readiness

The client can wait until:

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

The method returns `True` when PUDIS is ready. It returns `False` if the configured timeout is reached.

### Status Snapshot

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
    "username": "example_user",
    "battery_voltage": 12.6,
    "ignition_on": True,
}
```

The structured result can be processed directly by external automation software.

### Vehicle Protocol Readout

The client can start a vehicle protocol readout through the PUDIS `ProcedureService`.

The following RPC method is used:

```text
StartReadVal
```

The public client method is:

```python
client.read_vehicle_protocol()
```

The Python client prepares the request and sends it to PUDIS. The actual diagnostic procedure is executed by PUDIS.

---

## VIN Handling

The client must not explicitly send an empty VIN when calling `StartReadVal`.

The following approach must not be used:

```python
vin = ""
```

If the VIN field is omitted completely, PUDIS uses the VIN detected from the connected vehicle.

The request is therefore created without explicitly setting the VIN:

```python
request = procedure_control_pb2.StartReadValRequest()
```

Logical links are only added when values are provided.

---

## Installation

The PUDIS Client is available as an installable Python Wheel.

Install the Wheel with:

```powershell
python -m pip install .\pudis_client-0.1.0-py3-none-any.whl
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

## Usage

After installation, the library can be imported into another Python application:

```python
from pudis_client import PudisClient

client = PudisClient()
```

Example:

```python
from pudis_client import PudisClient


def main():
    client = PudisClient()

    client.start_pudis()

    ready = client.wait_until_ready(
        max_wait_seconds=180,
        poll_interval_seconds=3,
    )

    if not ready:
        print("PUDIS did not become ready in time.")
        return

    status = client.get_status_snapshot()
    print(status)

    client.read_vehicle_protocol()


if __name__ == "__main__":
    main()
```

The file `library_test.py` provides a complete functional example and simulates the call sequence of external test bench automation software.

---

## Public Interface

The public `PudisClient` interface currently provides the following methods:

```python
client.start_pudis()
client.close_pudis()
client.wait_until_ready()
client.get_status_snapshot()
client.read_vehicle_protocol()
```

### `start_pudis()`

Starts the locally installed PUDIS application.

### `close_pudis()`

Closes the running PUDIS application.

### `wait_until_ready()`

Waits until PUDIS is reachable, the application state is `READY`, and a user is logged in.

### `get_status_snapshot()`

Returns application, connection, login, battery and ignition information as a structured Python dictionary.

### `read_vehicle_protocol()`

Sends a `StartReadVal` request to PUDIS to start the vehicle protocol readout.

---

## Technical Background

PUDIS provides a gRPC API.

The PUDIS Client uses this API to:

- Request application status information
- Request connection status information
- Evaluate the current login status
- Start diagnostic procedures

The current connection settings are:

```text
Host: localhost
Port: 5050
```

PUDIS and the Python client therefore run on the same computer in the current environment.

---

## Proto Files

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

## Generated Python Code

The following Python files are generated from the `.proto` definitions:

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

The original `.proto` files are required during development and code generation.

They are not required at runtime when the generated Python files are already included in the installed library.

---

## Architecture

The project follows a layered architecture.

### Public API Layer

The public API layer provides the `PudisClient` class.

```text
src/pudis_client/pudis_client_api.py
```

This class is the main interface for external automation software.

### Service Layer

The service layer contains workflow and business logic.

Examples:

- Checking whether a user is logged in
- Checking whether PUDIS is ready
- Starting a vehicle protocol
- Starting and closing PUDIS
- Monitoring PUDIS states

```text
src/pudis_client/services/
```

### Client Layer

The client layer contains the direct technical gRPC calls.

Examples:

- `GetApplicationStatus`
- `GetConnectionStatus`
- `StartReadVal`
- `WatchRunningProcedure`

```text
src/pudis_client/clients/
```

### Mapper Layer

The mapper layer converts technical API values into human-readable status messages.

Examples:

```text
READY → Application ready
VCI_DETECTED → VCI detected
DOIP → DoIP active
```

```text
src/pudis_client/mappers/
```

### Generated Layer

The generated layer contains the Python modules created from the PUDIS `.proto` files.

```text
src/pudis_client/generated/
```

---

## Project Structure

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
├─ scripts/
│
├─ tests/
└─ docs/
```

---

## Folder Description

### `protos/`

Contains the original PUDIS gRPC API definitions.

### `src/pudis_client/`

Contains the installable Python package.

### `src/pudis_client/generated/`

Contains the generated gRPC modules included in the installable library.

### `src/pudis_client/clients/`

Contains the direct gRPC communication with PUDIS.

### `src/pudis_client/services/`

Contains workflow and business logic.

### `src/pudis_client/mappers/`

Contains human-readable status mappings.

### `examples/`

Contains example scripts that demonstrate how another Python application can call the PUDIS Client.

---

## Requirements

### Development Requirements

- Python 3.11 or newer
- pip
- Visual Studio or Visual Studio Code
- Access to PUDIS
- Access to the current PUDIS `.proto` files

Required development packages:

```text
grpcio
grpcio-tools
protobuf
build
```

Installation:

```powershell
python -m pip install grpcio grpcio-tools protobuf build
```

### Runtime Requirements

The installed library requires:

```text
grpcio
protobuf
```

The target computer also requires:

- PUDIS Gen. 5
- PUDIS gRPC interface on `localhost:5050`
- VCI drivers
- Test bench simulator or vehicle connection
- Required Windows permissions

---

## Generating the gRPC Code

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

## Building the Python Library

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

Example:

```text
pudis_client-0.1.0-py3-none-any.whl
pudis_client-0.1.0.tar.gz
```

---

## Testing

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

## Important Notes

- PUDIS must be running for the gRPC API to be available.
- The client can be started before PUDIS and will wait for availability.
- PUDIS and the client currently communicate through `localhost:5050`.
- Changes to `.proto` files require regeneration of the Python code.
- The VIN must not be sent as an empty string.
- If the VIN field is omitted, PUDIS uses the VIN of the connected vehicle.
- A connected VCI and vehicle or simulator are required before starting the vehicle protocol.
- The interactive menu is intended only for development and manual testing.
- External automation software should use the public `PudisClient` methods.

---

## Future Extensions

Potential future features include:

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

## Summary

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
