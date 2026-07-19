## ADDED Requirements

### Requirement: Plugin Manifest Format
Each plugin SHALL ship a static manifest declaring its identity (name, unique ID), version, target schema/interface version, and the set of network hosts it requires access to.

#### Scenario: Manifest declares required fields
- **WHEN** the host parses a plugin's manifest
- **THEN** the host requires plugin ID, version, target interface version, and declared network hosts to be present, and refuses to register the plugin if any is missing

#### Scenario: Manifest declares network capabilities
- **WHEN** a plugin's manifest lists the hostnames it needs to call
- **THEN** the host uses exactly that list to scope the plugin's HTTP fetch capability at load time

### Requirement: Plugin Discovery
The host SHALL discover bundled first-party plugins and user-installed plugins from designated plugin directories at startup without requiring manual configuration for the bundled set.

#### Scenario: Bundled plugin is discovered at startup
- **WHEN** the app starts with a first-party plugin present in the bundled plugin directory
- **THEN** the host registers the plugin and it becomes available without user action

#### Scenario: User-installed plugin is discovered
- **WHEN** a plugin package is placed in the user plugin directory
- **THEN** the host discovers it on next startup and registers it alongside bundled plugins

### Requirement: Plugin Enable/Disable State
The host SHALL allow a registered plugin to be individually enabled or disabled without uninstalling it, and disabled plugins SHALL NOT be loaded or invoked.

#### Scenario: User disables a plugin
- **WHEN** a user disables a registered plugin
- **THEN** the host stops invoking that plugin for data requests while keeping it registered for later re-enabling

#### Scenario: Disabled plugin is not loaded on startup
- **WHEN** the app starts and a registered plugin is marked disabled
- **THEN** the host does not instantiate that plugin's WASM module

### Requirement: Plugin Version Tracking
The registry SHALL track the installed version of each plugin and detect when a newer version is available for plugins that declare an update source.

#### Scenario: Plugin update is available
- **WHEN** a plugin declares an update source and a newer version exists there than the installed version
- **THEN** the registry surfaces that an update is available for that plugin
