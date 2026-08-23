# EasyDocker Recipe Requirements

Use this document as the mandatory checklist when Codex creates or reviews an EasyDocker recipe.

A recipe is not complete unless every requirement below is satisfied.

## Required File Rules

- The recipe must be valid JSON.
- The recipe file must live at `recipes/<recipe-name>.json`.
- The filename must match the recipe `name`.
- The recipe `name` must be lowercase and filename-safe.
- The recipe must be listed in `recipes/index.json`.
- The `recipes/index.json` entry must use the same `name` and `version` as the recipe file.

## Required Top-Level Keys

Every recipe must include:

```json
{
  "name": "",
  "version": 1,
  "description": "",
  "services": {},
  "fields": [],
  "ui": {}
}
```

| Key | Requirement |
|---|---|
| `name` | Unique recipe id. Must match the filename without `.json`. |
| `version` | Integer. Start at `1`; increment on any recipe change. |
| `description` | Short human-readable description for the catalog and recipe page. |
| `services` | Docker Compose services template. Must contain at least one service. |
| `fields` | All editable and recipe-managed values used by placeholders. |
| `ui` | Sections, tags, and explanatory copy for the form. |

Add `app_links` when the app exposes a browser UI.

Add `networks`, `volumes`, `configs`, or `secrets` only when the generated Compose needs them.

## Required UI Keys

Every recipe `ui` object must include:

```json
{
  "sections": [],
  "tags": [],
  "overview": "",
  "what_happens": [],
  "data_location": ""
}
```

| Key | Requirement |
|---|---|
| `sections` | Ordered list of form sections. Must include every field section used by `fields`. |
| `tags` | Catalog tags such as `media`, `networking`, `photos`, or `utility`. |
| `overview` | One short explanation of what the app does. |
| `what_happens` | Bullets explaining what EasyDocker will create or change. |
| `data_location` | Short explanation of where app data/config will be stored. |

Use these standard sections unless there is a strong reason not to:

```json
[
  { "name": "required", "label": "Minimum Required" },
  { "name": "optional", "label": "Optional", "collapsed": true },
  { "name": "advanced", "label": "Advanced", "collapsed": true },
  { "name": "readonly", "label": "Read-Only (Recipe Managed)", "collapsed": true }
]
```

## Required Field Keys

Every field in `fields` must include:

```json
{
  "name": "",
  "label": "",
  "section": "",
  "input_type": "text",
  "required": true,
  "default": "",
  "help": ""
}
```

| Key | Requirement |
|---|---|
| `name` | Unique uppercase snake-case placeholder name. |
| `label` | Clear user-facing label. |
| `section` | Must match one `ui.sections[].name`. |
| `input_type` | Must be `text`, `password`, `textarea`, or `select`. |
| `required` | Boolean. Set `true` only when the form must have a value. |
| `default` | Default value used for display and Compose generation. Use `""` if blank. |
| `help` | Practical help text for the person filling the form. |

For `select` fields, include `options` unless using `options_source`.

For `textarea` fields, include `rows`.

For recipe-managed values, include `"editable": false` and put the field in the `readonly` section.

## Compose Placeholder Rules

Every meaningful Compose value must come from a field placeholder.

Valid:

```json
"image": "${APP_IMAGE}"
```

Valid:

```json
"ports": [
  "${APP_PORT}:${APP_INTERNAL_PORT}"
]
```

Invalid:

```json
"image": "example/app:latest"
```

Invalid:

```json
"restart": "unless-stopped"
```

Rules:

- Every Compose leaf value must be a string template.
- Every Compose leaf string must contain at least one `${FIELD_NAME}` placeholder.
- Every placeholder must reference a field in `fields`, `PROJECT_NAME`, or a supported derived placeholder.
- Do not hardcode meaningful Compose values directly in `services`, `networks`, `volumes`, `configs`, or `secrets`.
- If a value should be fixed by the recipe, create a read-only field for it.

## Required Common Fields

Most single-service recipes should define these concepts, using app-specific names:

| Concept | Typical Field | Section |
|---|---|---|
| Host web port | `APP_PORT` | `required` |
| Data/config host path | `APP_DATA_PATH` or `APP_CONFIG_PATH` | `required` |
| Timezone | `TZ` | `required` or `optional` |
| PUID | `PUID` | `required` or `optional` |
| PGID | `PGID` | `required` or `optional` |
| Restart policy | `RESTART_POLICY` | `optional` |
| Container image | `APP_IMAGE` | `advanced` or `readonly` |
| Container name | `APP_CONTAINER_NAME` | `readonly` |
| Internal container port | `APP_INTERNAL_PORT` | `readonly` |
| Internal data/config path | `APP_DATA_TARGET` or `APP_CONFIG_TARGET` | `readonly` |

Only include fields that the app actually needs.

## App Links

If the deployed app has a web UI, include `app_links`.

```json
"app_links": [
  {
    "label": "Open App",
    "service": "app",
    "port": "${APP_PORT}",
    "path": "/",
    "scheme": "http"
  }
]
```

Requirements:

- `label` must be clear.
- `port` must use a placeholder.
- `path` should start with `/`.
- `scheme` should usually be `http` or `https`.

## Docker Network Fields

If the recipe lets the user choose networking, use:

```json
{
  "name": "APP_NETWORK_MODE",
  "label": "Network Mode",
  "section": "advanced",
  "input_type": "select",
  "required": true,
  "default": "__create_bridge__",
  "data_type": "docker_network",
  "options_source": "docker_networks",
  "help": "Choose the default bridge network, host networking, or an existing Docker network."
}
```

Then use the derived placeholders where needed:

```json
"network_mode": "${APP_NETWORK_MODE_MODE}",
"networks": "${APP_NETWORK_MODE_SERVICE_NETWORKS}"
```

and top-level:

```json
"networks": "${APP_NETWORK_MODE_DEFINITIONS}"
```

## Conditional Fields

If a field is only valid for some choices, use `visible_when` or `hidden_when`.

If the hidden value must not appear in Compose, also set:

```json
"omit_when_inactive": true
```

Example:

```json
{
  "hidden_when": {
    "APP_NETWORK_MODE": ["__host__"]
  },
  "omit_when_inactive": true
}
```

## YAML-Typed Fields

Use `"data_type": "yaml"` when the field should become a YAML list, object, boolean, or number rather than a string.

Good uses:

- `command`
- `entrypoint`
- healthcheck options
- extra port lists
- extra environment maps

## Hardware-Dependent Fields

For new recipes, prefer `availability_from` with an allowlisted detected variable:

```json
{
  "name": "APP_DRI_DEVICE",
  "label": "Hardware Acceleration Device",
  "section": "advanced",
  "input_type": "text",
  "required": false,
  "default": "",
  "availability_from": "detected.HAS_DRI",
  "help": "Optional hardware acceleration device."
}
```

Use `default_from` when a detected scalar should provide a field's initial value, for example `"default_from": "detected.CPU_THREADS"`. User-submitted values take precedence. Supported references are maintained by EasyDocker; recipes cannot execute detectors.

For a select field, `options_from` can use a detected list of strings such as `detected.SERIAL_DEVICES`, `detected.VIDEO_DEVICES`, or `detected.DRI_DEVICES`. EasyDocker turns each detected path into a label/value option without allowing recipe-defined detection commands.

Use `input_type: "detected"` when the recipe must consume a typed detected array. Detected fields must declare an allowlisted `detected_source`, a supported mode (`all`, `select_one`, or `select_many`), and a supported value type. Values are resolved or validated again on the server during Compose generation. A full-array placeholder must occupy the complete Compose value so it remains a YAML list rather than becoming a string.

Existing recipes may continue using `availability_path` when a device path depends on host hardware.

Example:

```json
{
  "name": "APP_DRI_DEVICE",
  "label": "Hardware Acceleration Device",
  "section": "advanced",
  "input_type": "text",
  "required": false,
  "default": "",
  "availability_path": "/dev/dri",
  "help": "Optional hardware acceleration device."
}
```

Use `detection_required_message` when the default unavailable-hardware message is not clear enough.

## Help Text Requirements

Every field must have useful `help`.

Good help explains:

- what the field controls
- expected format
- a safe example when useful
- when to leave it blank

Allowed HTML in help:

- `<a>`
- `<br>`
- `<code>`
- `<strong>`
- `<em>`
- `<ul>`
- `<ol>`
- `<li>`
- `<p>`

Do not put long documentation inside `help`. Use `docs_url` and `docs_label` for external docs.

## Final Codex Review Checklist

Before returning a recipe, Codex must confirm:

- JSON parses successfully.
- `name` matches the filename.
- `version` is an integer.
- `description` is present and short.
- `services` contains at least one service.
- `fields` is not empty.
- `ui.sections` is present.
- `ui.tags`, `ui.overview`, `ui.what_happens`, and `ui.data_location` are present.
- Every field has `name`, `label`, `section`, `input_type`, `required`, `default`, and `help`.
- Every field name is unique.
- Every field section exists in `ui.sections`.
- Every `select` field has `options` or `options_source`.
- Every Compose leaf contains a placeholder.
- Every placeholder references a field, `PROJECT_NAME`, or a supported derived placeholder.
- Recipe-managed fixed values are fields with `editable: false`.
- `app_links` exists when the app has a web UI.
- `recipes/index.json` includes the recipe name and version.
