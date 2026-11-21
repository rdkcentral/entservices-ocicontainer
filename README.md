# OCI Container Plugin Documentation

This repository contains the ENT / RDK OCI container service subsystem documented under the plugin area.

## Generated subsystem docs
- [docs/plugin-OCIContainer.md](docs/plugin-OCIContainer.md)

## Subsystem overview
The selected subsystem is the Thunder plugin at [plugin](plugin). It exposes OCI container lifecycle RPC operations, bridges them to Dobby, and emits state and lifecycle events back to callers.

## Related source locations
- [plugin/OCIContainer.h](plugin/OCIContainer.h)
- [plugin/OCIContainer.cpp](plugin/OCIContainer.cpp)
- [plugin/OCIContainerImplementation.h](plugin/OCIContainerImplementation.h)
- [plugin/OCIContainerImplementation.cpp](plugin/OCIContainerImplementation.cpp)
- [plugin/DobbyInterface.h](plugin/DobbyInterface.h)
- [plugin/DobbyInterface.cpp](plugin/DobbyInterface.cpp)
- [plugin/CMakeLists.txt](plugin/CMakeLists.txt)
- [Tests/L1Tests/tests/test_OCIContainer.cpp](Tests/L1Tests/tests/test_OCIContainer.cpp)

## Notes
This repository root README is intentionally concise; the full subsystem documentation is in the dedicated Markdown under [docs/plugin-OCIContainer.md](docs/plugin-OCIContainer.md).
