[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE.md) 
![Unit Tests](https://github.com/OpenVoiceOS/ovos-core/actions/workflows/unit_tests.yml/badge.svg)
[![codecov](https://codecov.io/gh/OpenVoiceOS/ovos-config/branch/dev/graph/badge.svg?token=CS7WJH4PO2)](https://codecov.io/gh/OpenVoiceOS/ovos-config)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)
[![Chat](https://img.shields.io/matrix/openvoiceos-general:matrix.org)](https://matrix.to/#/#OpenVoiceOS-general:matrix.org)
[![GitHub Discussions](https://img.shields.io/github/discussions/OpenVoiceOS/OpenVoiceOS?label=OVOS%20Discussions)](https://github.com/OpenVoiceOS/OpenVoiceOS/discussions)

# OVOS-config

helper package to interact with mycroft config


## Configuration Structure

### ovos.conf
The `ovos_config` package determines which config files to load based on `ovos.conf`.
`get_ovos_config` will return default values that load `mycroft.conf` unless otherwise configured.

`ovos.conf` files are loaded in the following order, with later files taking priority over earlier ones in the list:
1. /etc/OpenVoiceOS/ovos.conf
2. /etc/mycroft/ovos.conf (Deprecated)
3. `XDG_CONFIG_DIRS` + /OpenVoiceOS/ovos.conf
3. /etc/xdg/OpenVoiceOS/ovos.conf
4. `XDG_CONFIG_HOME` (default ~/.config) + /OpenVoiceOS/ovos.conf

A simple `ovos_config` should have a structure like:

```yaml
xdg: True
base_folder: mycroft
config_filename: mycroft.conf
default_config_path: <Path to Installed Core>/configuration/mycroft.conf
module_overrides: {}
submodule_mappings: {}
```

Non-Mycroft modules may specify alternate config paths. A call to `get_ovos_config` from 
`neon_core` or `neon_messagebus` will return a configuration like:

```yaml
xdg: True
base_folder: neon
config_filename: neon.yaml
default_config_path: /etc/example/config/neon.yaml
module_overrides:
  neon_core:
    xdg: true
    base_folder: neon
    config_filename: neon.yaml
    default_config_path: /etc/example/config/neon.yaml
submodule_mappings:
  neon_core: neon_core
  neon_core.skills.skill_manager: neon_core
  neon_messagebus: neon_core
  neon_speech: neon_core
  neon_audio: neon_core
  neon_gui: neon_core
```

If `get_ovos_config` was called from `mycroft` with the same configuration file as the last example,
the returned configuration would be:

```yaml
xdg: True
base_folder: mycroft
config_filename: mycroft.conf
default_config_path: <Path to Installed Core>/configuration/mycroft.conf
module_overrides:
  neon_core:
    xdg: true
    base_folder: neon
    config_filename: neon.yaml
    default_config_path: /etc/example/config/neon.yaml
submodule_mappings:
  neon_core: neon_core
  neon_core.skills.skill_manager: neon_core
  neon_messagebus: neon_core
  neon_speech: neon_core
  neon_audio: neon_core
  neon_gui: neon_core
```

## Configuration
`ovos_config.config.Configuration` is a singleton object that loads a single config
object. The configuration files loaded are determined by `ovos.conf` as described above.
Using the above example, if `Configuration()` is called from `neon_speech`, the
following configs would be loaded in this order:

1. /etc/example/config/neon.yaml
2. `os.environ.get('MYCROFT_SYSTEM_CONFIG')` or /etc/neon/neon.yaml
3. `os.environ.get('MYCROFT_WEB_CACHE')` or `XDG_CONFIG_PATH`/neon/web_cache.json (unless `disable_remote_config == True` in earlier config)
4. ~/.neon/neon.yaml (Deprecated)
3. `XDG_CONFIG_DIRS` + /neon/neon.yaml
3. /etc/xdg/neon/neon.yaml
4. `XDG_CONFIG_HOME` (default ~/.config) + /neon/neon.yaml
