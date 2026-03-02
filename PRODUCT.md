# OCIContainer Plugin - Product Documentation

## Product Overview

The OCIContainer plugin enables **containerized application management** on RDK (Reference Design Kit) platforms. It provides a standards-based interface for launching, managing, and monitoring OCI-compliant containers, enabling secure and isolated execution of applications and services on set-top boxes, smart TVs, and other RDK devices.

## Key Features

### Container Lifecycle Management
- **Launch Containers:** Start containers from OCI bundles or Dobby-specific configurations
- **Stop Containers:** Graceful or forced termination of running containers
- **State Monitoring:** Real-time tracking of container execution states
- **List Operations:** Enumerate all active containers with metadata

### Advanced Operations
- **Pause/Resume:** Suspend and restore container execution without termination
- **Hibernate/Wakeup:** Deep suspend with memory snapshot for rapid recovery
- **Exec Support:** Execute commands inside running containers
- **Dynamic Mounting:** Add/remove filesystem mounts at runtime

### Event Notification System
- **Real-time Events:** Instant notifications on container state changes
- **Multi-Client Support:** Multiple applications can subscribe to container events
- **Comprehensive Events:** Started, stopped, state changed, and error notifications

### Metadata Management
- **Container Annotations:** Add custom key-value metadata to containers
- **Runtime Configuration:** Query and modify container properties
- **State Inspection:** Detailed container status and configuration retrieval

## Use Cases

### 1. Application Isolation
**Scenario:** Running untrusted third-party applications safely on the device

**Solution:** Each application runs in its own container with:
- Resource limits (CPU, memory, I/O)
- Network isolation
- Restricted filesystem access
- Controlled device permissions

**Benefits:**
- Enhanced security posture
- System stability protection
- Simplified application management

### 2. Service Orchestration
**Scenario:** Managing multiple micro-services on a single device

**Solution:** Deploy each service as a container:
- Independent lifecycle management
- Service discovery and coordination
- Resource allocation and prioritization
- Crash isolation and auto-restart

**Benefits:**
- Modular system architecture
- Independent service updates
- Fault tolerance

### 3. Development and Testing
**Scenario:** Rapid application development and testing cycles

**Solution:** Containerized development environments:
- Consistent runtime across dev/test/production
- Quick container start/stop for testing
- Snapshot and restore for debugging
- Isolated test environments

**Benefits:**
- Faster iteration cycles
- Reproducible test scenarios
- Reduced system configuration drift

### 4. Content Protection
**Scenario:** Secure media playback and DRM operations

**Solution:** Verified container execution with OMI:
- Integrity-checked container images
- Tamper detection
- Secure key material handling
- Encrypted container content

**Benefits:**
- Content security compliance
- Reduced piracy risk
- Licensing enforcement

## API Capabilities

### Core API Methods (via Thunder JSON-RPC)

#### Container Management
```
startContainer(containerId, bundlePath, command, westerosSocket)
startContainerFromDobbySpec(containerId, dobbySpec, command, westerosSocket)
stopContainer(containerId, force)
listContainers()
getContainerInfo(containerId)
getContainerState(containerId)
```

#### Execution Control
```
pauseContainer(containerId)
resumeContainer(containerId)
hibernateContainer(containerId, options)
wakeupContainer(containerId)
executeCommand(containerId, options, command)
```

#### Configuration & Metadata
```
annotate(containerId, key, value)
removeAnnotation(containerId, key)
mount(containerId, source, target, type, options)
unmount(containerId, target)
```

### Event Notifications
```
onContainerStarted(descriptor, name)
onContainerStopped(descriptor, name, reason)
onContainerStateChanged(descriptor, name, state)
```

## Integration Benefits

### For System Integrators
- **Standards-Based:** Uses OCI container specifications for broad compatibility
- **Thunder Framework Integration:** Seamless integration with RDK Thunder ecosystem
- **Well-Defined APIs:** Clear, documented interfaces for all operations
- **Flexible Deployment:** Supports various container configurations and runtime options

### For Application Developers
- **Simple Container Management:** High-level APIs abstract container complexity
- **Event-Driven Architecture:** React to container state changes in real-time
- **Rich Metadata Support:** Add custom annotations for application-specific needs
- **Multi-Instance Support:** Run multiple instances with isolated configurations

### For Device Manufacturers
- **Proven Technology:** Built on mature Linux container technologies
- **Resource Efficient:** Minimal overhead compared to VM-based solutions
- **Scalable:** Handles multiple containers on resource-constrained devices
- **Secure:** Leverages kernel namespaces, cgroups, and security modules

## Performance Characteristics

### Resource Footprint
- **Memory Overhead:** ~10-20MB per container (varies with workload)
- **CPU Impact:** Minimal overhead; near-native performance
- **Storage:** Layered filesystem with efficient space utilization
- **Network:** Native network stack performance with configurable isolation

### Scalability
- **Concurrent Containers:** Supports 10+ simultaneous containers (hardware-dependent)
- **Startup Time:** Container launch in <500ms (warm start)
- **Event Latency:** Real-time notifications with <50ms delay
- **API Throughput:** Handles 100+ API calls/second

### Reliability
- **Failure Isolation:** Container crashes don't affect host or other containers
- **Automatic Recovery:** Dobby daemon manages container lifecycle and restarts
- **Graceful Degradation:** System remains functional if individual containers fail
- **Comprehensive Logging:** Detailed logs for troubleshooting and audit

## Platform Requirements

### Minimum System Requirements
- **OS:** Linux kernel 3.18+ with namespace and cgroup support
- **Memory:** 256MB available RAM (beyond base system)
- **Storage:** 100MB for runtime components + container images
- **CPU:** ARM or x86 architecture

### Required Services
- **Dobby Daemon:** Container runtime manager (must be running)
- **D-Bus System Bus:** Inter-process communication
- **Thunder Framework:** Plugin host environment

### Optional Components
- **OMI Proxy:** For verified container execution
- **Westeros Compositor:** For graphical container support

## Deployment Scenarios

### Typical Deployment
```
┌────────────────────────────────────────┐
│     RDK Platform (Set-Top Box/TV)      │
│  ┌──────────────────────────────────┐  │
│  │    Thunder Framework             │  │
│  │  ┌────────────────────────────┐  │  │
│  │  │  OCIContainer Plugin       │  │  │
│  │  └────────────┬───────────────┘  │  │
│  └───────────────┼──────────────────┘  │
│                  │                      │
│  ┌───────────────▼──────────────────┐  │
│  │    Dobby Container Runtime       │  │
│  │  ┌────────┐ ┌────────┐ ┌──────┐ │  │
│  │  │ App 1  │ │ App 2  │ │ DRM  │ │  │
│  │  │        │ │        │ │      │ │  │
│  │  └────────┘ └────────┘ └──────┘ │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

## Security & Compliance

- **OCI Specification Compliance:** Adheres to Open Container Initiative standards
- **Kernel Security Modules:** Compatible with SELinux, AppArmor
- **Content Security:** Integrates with OMI for verified execution
- **Access Control:** D-Bus policy enforces service access restrictions
- **Audit Logging:** Comprehensive event logging for security analysis

## Future Roadmap Considerations

- Enhanced resource controls (QoS, bandwidth limiting)
- GPU/hardware acceleration passthrough
- Advanced networking modes (overlay, macvlan)
- Container image management APIs
- Snapshot and migration capabilities
- Health monitoring and auto-recovery policies
