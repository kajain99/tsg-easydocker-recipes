# Defining New EasyDocker Recipes

This guide is the authoring contract for adding recipes to `tsg-easydocker-recipes`.

Use it with:

- `recipe-template.json` as the starting JSON shape
- `SCHEMA.md` as the full format reference
- `FIELDS.md` as the field property reference

## Where Recipes Live

Each app recipe is one JSON file in:

```text
recipes/<recipe-name>.json
```

The recipe must also be listed in:

```text
recipes/index.json
```

`index.json` is the version map used by EasyDocker when refreshing recipes.

## Required Top-Level Fields

Every recipe must define these top-level keys:

| Key | Type | Required | Purpose |
|---|---|---:|---|
| `name` | string | Yes | Unique recipe id. Should match the filename without `.json`. |
| `version` | integer | Yes | Recipe version. Increment when the recipe changes. |
| `description` | string | Yes | Short catalog/form description. |
| `services` | object | Yes | Docker Compose services template. |
| `fields` | array | Yes | User-visible and recipe-managed form fields. |
| `ui` | object | Yes | Form sections, tags, and explanatory copy. |

Most current recipes also define:

| Key | Type | Required | Purpose |
|---|---|---:|---|
| `app_links` | array | When the app has a browser UI | Post-deploy launch buttons. |
| `networks` | object/string template | When needed | Compose network definitions. |

Recipes may include other valid Docker Compose top-level sections such as `volumes`, `configs`, or `secrets` when needed.

## Reserved Recipe Keys

These top-level keys are EasyDocker metadata and are not copied into generated Compose YAML:

- `name`
- `version`
- `description`
- `fields`
- `ui`
- `app_links`

All other top-level keys are treated as Compose template content.

## Compose Template Rules

Compose values must be driven by placeholders:

```json
"image": "${APP_IMAGE}"
```

The app validates Compose leaves strictly:

- every Compose leaf value must be a string template
- every string template must contain at least one `${FIELD_NAME}` placeholder
- every placeholder must reference a field in `fields`, `PROJECT_NAME`, or a supported derived placeholder

Objects and arrays are allowed as Compose structure, but their final leaf values still need placeholders.

Good:

```json
"ports": [
  "${APP_PORT}:${APP_INTERNAL_PORT}"
]
```

Not accepted:

```json
"restart": "unless-stopped"
```

Instead, define a field such as `RESTART_POLICY` and use:

```json
"restart": "${RESTART_POLICY}"
```

If an exact placeholder resolves to an empty value, EasyDocker prunes it where possible. Empty list items, empty objects, and empty environment entries are removed from generated Compose.

Relative bind mounts beginning with `./` are resolved under the deployed app's config folder.

## Field Rules

Every item in `fields` must define:

| Key | Type | Required | Purpose |
|---|---|---:|---|
| `name` | string | Yes | Placeholder name. Must be unique in the recipe. |
| `label` | string | Yes | User-facing label. |
| `section` | string | Yes | Must match a section in `ui.sections`. |
| `input_type` | string | Yes | `text`, `password`, `textarea`, or `select`. |
| `required` | boolean | Yes | Whether the browser form requires a value. |
| `default` | any/string | Yes in current recipes | Default value used for display and Compose generation. |
| `help` | string | Yes in current recipes | Help shown beside the field. |

Common optional field keys:

| Key | Used For |
|---|---|
| `editable: false` | Show recipe-managed values as read-only. |
| `options` | Static choices for `select` fields. |
| `options_source` | Dynamic choices. Currently supports `docker_networks`. |
| `data_type` | Special parsing. Supports `yaml` and `docker_network`. |
| `rows` | Textarea height. |
| `docs_url` / `docs_label` | Extra official documentation link under a field. |
| `visible_when` / `hidden_when` | Conditional field visibility. |
| `omit_when_inactive` | Remove conditional field value from Compose when hidden. |
| `availability_path` | Disable or annotate hardware-dependent fields when a device was not detected. |
| `detection_required_message` | Custom message for unavailable hardware fields. |

Use uppercase snake case for field names, for example `PLEX_PORT` or `GLUETUN_IMAGE`.

## Standard Sections

Every current recipe uses these four sections:

```json
"sections": [
  { "name": "required", "label": "Minimum Required" },
  { "name": "optional", "label": "Optional", "collapsed": true },
  { "name": "advanced", "label": "Advanced", "collapsed": true },
  { "name": "readonly", "label": "Read-Only (Recipe Managed)", "collapsed": true }
]
```

Use them this way:

- `required`: minimum user input needed for a working deployment
- `optional`: common user choices with safe defaults
- `advanced`: image tags, command overrides, networking, resources, and expert settings
- `readonly`: recipe-managed values shown for transparency, usually with `editable: false`

EasyDocker injects missing CPU and memory limit fields into the `advanced` section for each service at runtime, so every recipe should keep an `advanced` section.

## Required UI Fields

Every current recipe's `ui` object includes:

| Key | Type | Required | Purpose |
|---|---|---:|---|
| `sections` | array | Yes | Ordered form section definitions. |
| `tags` | array | Yes in current recipes | Catalog badges. |
| `overview` | string | Yes in current recipes | Short app explanation. |
| `what_happens` | array | Yes in current recipes | Bullets shown before deployment. |
| `data_location` | string | Yes in current recipes | Explains where data/config will live. |

Each section must define `name`; `label` is recommended. `collapsed` is optional.

## App Links

Use `app_links` when the deployed app exposes a web UI:

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

`port`, `path`, and `scheme` support placeholders. Links with no resolved port are skipped.

## Special Field Types

### `data_type: "yaml"`

Use this when a field should become a real YAML value rather than a string.

Good uses:

- `command`
- `entrypoint`
- healthcheck toggles
- lists of published ports
- extra environment maps

If parsing fails, EasyDocker keeps the submitted value as a string.

### `data_type: "docker_network"`

Use this with:

```json
"options_source": "docker_networks"
```

The field can resolve to one of these modes:

- empty or `__create_bridge__`: use the default Compose bridge network
- `__host__`: set `network_mode: host`
- `__external__:<network-name>`: attach the service to an existing Docker network

For a field named `APP_NETWORK_MODE`, EasyDocker also exposes derived placeholders:

- `${APP_NETWORK_MODE_MODE}`
- `${APP_NETWORK_MODE_SERVICE_NETWORKS}`
- `${APP_NETWORK_MODE_DEFINITIONS}`

These are used to template `network_mode`, service `networks`, and top-level `networks`.

## Conditional Fields

Use `visible_when` and `hidden_when` to show fields only for relevant choices.

```json
"visible_when": {
  "VPN_TYPE": ["openvpn"]
}
```

```json
"hidden_when": {
  "APP_NETWORK_MODE": ["__host__"]
}
```

Add `omit_when_inactive: true` when a hidden field should resolve to `null` and be pruned from generated Compose.

## Extra Environment Maps

Inside a service, `x-easydocker-extra-environment` can be used to merge a YAML object into the final `environment` map:

```json
"x-easydocker-extra-environment": "${APP_EXTRA_ENV}"
```

The backing field should usually use:

```json
"data_type": "yaml"
```

## Recipe-Managed Values

If the recipe controls a value but users should see it, define it as a read-only field:

```json
{
  "name": "APP_IMAGE",
  "label": "Image",
  "section": "readonly",
  "input_type": "text",
  "required": true,
  "default": "example/app:latest",
  "editable": false,
  "help": "Recipe-managed image."
}
```

This keeps the generated Compose transparent without making beginner users edit internals.

## New Recipe Checklist

1. Copy `recipe-template.json` to `recipes/<recipe-name>.json`.
2. Set `name`, `version`, and `description`.
3. Define `ui.sections`, `ui.tags`, `ui.overview`, `ui.what_happens`, and `ui.data_location`.
4. Add the Compose `services` template.
5. Create fields for every meaningful Compose value.
6. Use placeholders for every Compose leaf value.
7. Put beginner-required inputs in `required`.
8. Put image tags, networking, command overrides, and expert choices in `advanced`.
9. Put recipe-managed internals in `readonly` with `editable: false`.
10. Add `app_links` if the app exposes a web UI.
11. Add or update the recipe entry in `recipes/index.json`.
12. Load the recipe in EasyDocker and verify the generated Compose before deploying.

## Minimal Shape

```json
{
  "name": "exampleapp",
  "version": 1,
  "description": "Short app description",
  "services": {
    "exampleapp": {
      "container_name": "${APP_CONTAINER_NAME}",
      "image": "${APP_IMAGE}",
      "restart": "${RESTART_POLICY}"
    }
  },
  "fields": [
    {
      "name": "RESTART_POLICY",
      "label": "Restart Policy",
      "section": "optional",
      "input_type": "select",
      "required": true,
      "default": "unless-stopped",
      "options": [
        { "label": "Unless Stopped", "value": "unless-stopped" },
        { "label": "Always", "value": "always" },
        { "label": "No", "value": "no" }
      ],
      "help": "Controls whether the container restarts automatically."
    },
    {
      "name": "APP_CONTAINER_NAME",
      "label": "Container Name",
      "section": "readonly",
      "input_type": "text",
      "required": true,
      "default": "${PROJECT_NAME}",
      "editable": false,
      "help": "Recipe-managed container name."
    },
    {
      "name": "APP_IMAGE",
      "label": "Image",
      "section": "readonly",
      "input_type": "text",
      "required": true,
      "default": "example/app:latest",
      "editable": false,
      "help": "Recipe-managed image."
    }
  ],
  "ui": {
    "sections": [
      { "name": "required", "label": "Minimum Required" },
      { "name": "optional", "label": "Optional", "collapsed": true },
      { "name": "advanced", "label": "Advanced", "collapsed": true },
      { "name": "readonly", "label": "Read-Only (Recipe Managed)", "collapsed": true }
    ],
    "tags": ["example"],
    "overview": "ExampleApp is a sample app.",
    "what_happens": [
      "One container will be created"
    ],
    "data_location": "Stored relative to this app's config folder unless you enter an absolute path."
  }
}
```
