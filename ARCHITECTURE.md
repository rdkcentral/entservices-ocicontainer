# OCIContainer Plugin Architecture

## Overview

The OCIContainer plugin is a WPEFramework/Thunder plugin that provides container management capabilities for RDK platforms. It acts as a bridge between Thunder framework and the Dobby container runtime, exposing container lifecycle management through a standardized COM-RPC interface.

## System Architecture

### High-Level Component Stack

```
┌─────────────────────────────────────────────────┐
│         Thunder Framework / Client Apps         │
└─────────────────────┬───────────────────────────┘
                      │ JSON-RPC / COM-RPC
┌─────────────────────▼───────────────────────────┐
│          OCIContainer Thunder Plugin             │
│  ┌──────────────────────────────────────────┐   │
│  │       OCIContainer (JSONRPC Layer)       │   │
│  └─────────────────┬────────────────────────┘   │
│                    │                             │
│  ┌─────────────────▼────────────────────────┐   │
│  │    OCIContainerImplementation (Core)     │   │
│  └─────────────────┬────────────────────────┘   │
│                    │                             │
│  ┌─────────────────▼────────────────────────┐   │
│  │        DobbyInterface (Adapter)          │   │
│  └─────────────────┬────────────────────────┘   │
└────────────────────┼─────────────────────────────┘
                     │ D-Bus IPC
┌────────────────────▼─────────────────────────────┐
│           Dobby Container Runtime                │
│  (OCI Container Management & Execution)          │
└──────────────────────────────────────────────────┘
                     │
┌────────────────────▼─────────────────────────────┐
│    Linux Kernel (cgroups, namespaces, etc.)      │
└──────────────────────────────────────────────────┘
```

## Core Components

### 1. OCIContainer (JSONRPC Layer)
**File:** `plugin/OCIContainer.h`, `plugin/OCIContainer.cpp`

**Responsibilities:**
- Thunder plugin lifecycle management (Initialize, Deinitialize)
- JSON-RPC method registration and dispatching
- Connection management for client registrations
- Event notification distribution to registered clients

**Key Interfaces:**
- Implements `PluginHost::IPlugin` for Thunder integration
- Implements `PluginHost::JSONRPC` for JSON-RPC support
- Uses `Exchange::IOCIContainer` interface for container operations

### 2. OCIContainerImplementation (Business Logic)
**File:** `plugin/OCIContainerImplementation.h`, `plugin/OCIContainerImplementation.cpp`

**Responsibilities:**
- Core container management logic
- Client notification registration/unregistration
- Event dispatching to multiple clients
- Thread-safe access to notification callbacks
- Container state management

**Key Features:**
- Multi-client notification support via observer pattern
- Thread-safe operations using mutex locks
- Asynchronous event dispatching via worker pool
- COM-RPC service registration for out-of-process communication

### 3. DobbyInterface (Dobby Adapter)
**File:** `plugin/DobbyInterface.h`, `plugin/DobbyInterface.cpp`

**Responsibilities:**
- Abstraction layer for Dobby container runtime
- D-Bus communication with Dobby daemon
- Container lifecycle operation implementations
- State change event handling and propagation
- OMI (OCI Metadata Interface) integration

**Container Operations:**
- `listContainers()` - Enumerate running containers
- `getContainerInfo()` - Retrieve container metadata
- `getContainerState()` - Query container execution state
- `startContainer()` - Launch container from OCI bundle
- `startContainerFromDobbySpec()` - Launch with Dobby-specific config
- `stopContainer()` - Terminate container (graceful/forced)
- `pauseContainer()` / `resumeContainer()` - Suspend/resume execution
- `hibernateContainer()` / `wakeupContainer()` - Deep suspend/restore
- `executeCommand()` - Run command inside container
- `annotate()` / `removeAnnotation()` - Metadata management
- `mount()` / `unmount()` - Runtime filesystem operations

**Event Handling:**
- `onContainerStarted()` - Container launch notification
- `onContainerStopped()` - Container termination notification
- `onContainerStateChanged()` - Runtime state transitions
- `onVerityFailed()` - Integrity verification failures (OMI)

## Data Flow

### Container Start Operation
```
Client Request (JSON-RPC)
    ↓
OCIContainer::startContainer()
    ↓
OCIContainerImplementation::StartContainer()
    ↓
DobbyInterface::startContainer()
    ↓
D-Bus IPC → Dobby Daemon
    ↓
OCI Runtime (runc/crun)
    ↓
Container Process Creation
    ↓
← Event: onContainerStarted()
    ↓
← OCIContainerImplementation::dispatchEvent()
    ↓
← Notify all registered clients
```

### Event Notification Flow
```
Dobby State Change
    ↓
D-Bus Signal
    ↓
DobbyProxy Event Listener
    ↓
DobbyInterface::stateListener() [static callback]
    ↓
DobbyInterface::onContainerStateChanged()
    ↓
IEventHandler::dispatchEvent()
    ↓
OCIContainerImplementation::Dispatch()
    ↓
Worker Pool Job
    ↓
Iterate registered INotification callbacks
    ↓
Client event handlers (onContainerStarted/Stopped/StateChanged)
```

## Threading Model

- **Main Thunder Thread:** Handles plugin initialization, JSON-RPC requests
- **IPC Service Thread:** Manages D-Bus communication with Dobby
- **Worker Pool Threads:** Asynchronous event dispatch to clients
- **Synchronization:** Mutex locks protect client notification lists

## Dependencies

### External Libraries
- **WPEFramework/Thunder:** Plugin framework, COM-RPC, JSON-RPC
- **Dobby:** OCI container runtime and management daemon
- **libsystemd (sd-bus):** D-Bus communication
- **OMI Client Library:** OCI Metadata Interface for secure containers

### Internal Dependencies
- **helpers/UtilsLogging.h:** Logging macros (LOGINFO, LOGERR, etc.)
- **helpers/UtilsJsonRpc.h:** JSON-RPC helper utilities

## Configuration

**Product Config Directory:** `/etc/entservices/`

**Plugin Configuration:**
- `PLUGIN_OCICONTAINER_STARTUPORDER` - Plugin initialization order

**Build Options:**
- `PLUGIN_OCICONTAINER` - Enable/disable plugin build
- `COMCAST_CONFIG` - Platform-specific configurations
- `RDK_SERVICE_L2_TEST` - Enable L2 testing support

## Security Considerations

- Container isolation via Linux kernel namespaces and cgroups
- D-Bus communication secured via system bus permissions
- OMI integration for verified container execution
- Optional security token enforcement (DISABLE_SECURITY_TOKEN)

## Testing Framework

**L1 Tests:** Unit tests for core functionality (`Tests/L1Tests/`)
- Container operation validation
- Error handling verification

**L2 Tests:** Integration tests with Dobby runtime (`Tests/L2Tests/`)
- End-to-end container lifecycle
- Multi-client notification scenarios
- State transition verification

## Error Handling

- Comprehensive error codes via `Core::hresult`
- Detailed error reasons returned to clients
- Graceful degradation on Dobby unavailability
- Logging at all critical paths for debugging
