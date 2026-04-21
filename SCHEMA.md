# EasyDocker Recipe Schema

This document defines the JSON recipe format used by EasyDocker.

An EasyDocker recipe describes:

1. recipe metadata
2. Docker Compose template content
3. form fields
4. UI section layout
5. optional post-deploy app links

The current recipe model has two core rules:

- every meaningful Compose value must come from a field placeholder
- every field must belong to a section declared in `ui.sections`

---

## Top-Level Structure

```json
{
  "name": "appname",
  "version": 1,
  "description": "Short app description",

  "services": {},
  "volumes": {},
  "networks": {},
  "configs": {},
  "secrets": {},

  "app_links": [],
  "fields": [],
  "ui": {}
}
```

---

## Reserved Top-Level Keys

These keys are metadata and are not copied directly into the generated Compose file:

- `name`
- `version`
- `description`
- `fields`
- `ui`
- `app_links`

All other top-level keys are treated as Compose content and included in generated YAML.

That means recipes may include valid Compose sections such as:

- `services`
- `volumes`
- `networks`
- `configs`
- `secrets`
- `name`
- other valid Compose keys

---

## Required Metadata

### `name`

```json
"name": "tailscale"
```

- required
- unique recipe name
- used in routing and config folder naming
- should be lowercase and filename-safe

### `version`

```json
"version": 6
```

- required
- integer only
- increase whenever the recipe changes
- EasyDocker refresh updates only when the remote version is higher

### `description`

```json
"description": "Secure remote access to your NAS"
```

- required
- short human-readable description

---

## Compose Template Rules

Inside Compose sections, use placeholders like:

```json
"${FIELD_NAME}"
```

These placeholders are replaced using submitted form values.

### Important Rule

Every meaningful Compose value must come from a field placeholder.

That includes things like:

- image names
- container names
- restart policy
- internal mount targets
- internal ports
- capabilities
- devices
- command and entrypoint

If a value matters, define it as a field and reference it from Compose.

### Example

```json
"image": "${TAILSCALE_IMAGE}"
```

with a corresponding field:

```json
{
  "name": "TAILSCALE_IMAGE",
  "label": "Image",
  "section": "readonly",
  "input_type": "text",
  "required": true,
  "default": "tailscale/tailscale:latest",
  "editable": false,
  "help": "Recipe-managed Tailscale image."
}
```

### Empty Placeholder Behavior

If a placeholder is the entire value and resolves to empty, EasyDocker omits it where possible.

Example:

```json
"command": "${APP_COMMAND}"
```

If `APP_COMMAND` is blank, `command` is omitted.

### Combined Templates

Placeholders can also be combined inside longer strings:

```json
"${APP_PORT}:${APP_INTERNAL_PORT}"
```

or:

```json
"${EXIT_NODE_EXTRA_ARGS} ${TS_EXTRA_ARGS}"
```

---

## `fields` Schema

`fields` defines everything the user can see in the form.

That includes:

- editable values
- optional values
- advanced values
- recipe-managed read-only values

Example:

```json
"fields": [
  {
    "name": "TS_AUTHKEY",
    "label": "Auth Key",
    "section": "required",
    "input_type": "password",
    "required": true,
    "default": "",
    "help": "Create a Tailscale auth key and paste it here."
  }
]
```

### Field Properties

#### `name`

Internal placeholder name.

```json
"name": "TS_AUTHKEY"
```

- required
- must match placeholder names used in Compose

#### `label`

User-facing field label.

```json
"label": "Auth Key"
```

- required

#### `section`

Section key used by the UI.

```json
"section": "required"
```

- required
- must match one of the section names defined in `ui.sections`

#### `input_type`

Field render type.

Currently supported:

- `text`
- `password`
- `textarea`
- `select`

#### `required`

Whether the field must be filled.

```json
"required": true
```

#### `default`

Default value used in the form and compose generation.

```json
"default": "./data"
```

Defaults may themselves contain placeholders, for example:

```json
"default": "${PROJECT_NAME}_server"
```

#### `help`

Help text shown in the UI.

```json
"help": "Host path where app data will be stored."
```

Help supports a small safe subset of HTML.

Useful tags:

- `<a>`
- `<br>`
- `<code>`
- `<strong>`
- `<em>`
- `<ul>`
- `<ol>`
- `<li>`
- `<p>`

#### `editable`

Whether the user can change the field.

```json
"editable": false
```

- optional
- omitted means editable
- `false` means visible but read-only

#### `rows`

Optional. Used with `textarea`.

```json
"rows": 2
```

#### `data_type`

Optional. Controls how the submitted value is parsed.

Current useful value:

- `yaml`

```json
"data_type": "yaml"
```

Use this for advanced fields such as:

- `command`
- `entrypoint`

#### `options`

Optional. Used only with `select`.

Each option contains:

- `label`
- `value`

Example:

```json
"options": [
  { "label": "Yes", "value": "true" },
  { "label": "No", "value": "false" }
]
```

---

## `ui` Schema

The `ui` object defines how the form is grouped and explained.

Example:

```json
"ui": {
  "sections": [
    {
      "name": "required",
      "label": "Minimum Required"
    },
    {
      "name": "optional",
      "label": "Optional",
      "collapsed": true
    },
    {
      "name": "advanced",
      "label": "Advanced",
      "collapsed": true
    },
    {
      "name": "readonly",
      "label": "Read-Only (Recipe Managed)",
      "collapsed": true
    }
  ],
  "overview": "Tailscale lets you securely access your NAS from anywhere.",
  "what_happens": [
    "A Tailscale container will be created",
    "Its state will be saved in the folder you choose"
  ],
  "data_location": "Stored relative to this app's config folder unless you enter an absolute path."
}
```

### UI Properties

#### `sections`

- required
- ordered list of sections rendered by the form
- every field's `section` must match one of these names

Each section object supports:

- `name` - internal section key
- `label` - heading shown in the UI
- `collapsed` - optional boolean for initial collapsed state

#### `overview`

- short app explanation

#### `what_happens`

- list of bullets shown to the user

#### `data_location`

- short note explaining where data/config goes

---

## `app_links` Schema

Optional. Used to show app launch links after deploy.

Example:

```json
"app_links": [
  {
    "label": "Open Immich",
    "service": "immich-server",
    "port": "${IMMICH_PORT}",
    "path": "/",
    "scheme": "http"
  }
]
```

### Link Properties

- `label` - user-facing button text
- `service` - service name
- `port` - host port
- `path` - optional URL path
- `scheme` - optional, usually `http` or `https`

---

## Example Recipe

```json
{
  "name": "exampleapp",
  "version": 1,
  "description": "Example Docker app",
  "services": {
    "exampleapp": {
      "container_name": "${APP_CONTAINER_NAME}",
      "image": "${APP_IMAGE}",
      "restart": "${RESTART_POLICY}",
      "ports": [
        "${APP_PORT}:${APP_INTERNAL_PORT}"
      ],
      "environment": {
        "APP_TOKEN": "${APP_TOKEN}"
      },
      "volumes": [
        "${DATA_LOCATION}:${APP_DATA_TARGET}"
      ],
      "command": "${APP_COMMAND}"
    }
  },
  "fields": [
    {
      "name": "APP_PORT",
      "label": "App Port",
      "section": "required",
      "input_type": "text",
      "required": true,
      "default": "8080",
      "help": "Port used to open the app in your browser."
    },
    {
      "name": "APP_TOKEN",
      "label": "App Token",
      "section": "required",
      "input_type": "password",
      "required": true,
      "default": "",
      "help": "Secret token used by the app."
    },
    {
      "name": "DATA_LOCATION",
      "label": "Data Folder",
      "section": "required",
      "input_type": "text",
      "required": true,
      "default": "./data",
      "help": "Host path where app data will be stored."
    },
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
        { "label": "On Failure", "value": "on-failure" },
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
      "help": "Recipe-managed image name."
    },
    {
      "name": "APP_INTERNAL_PORT",
      "label": "Internal Port",
      "section": "readonly",
      "input_type": "text",
      "required": true,
      "default": "8080",
      "editable": false,
      "help": "Recipe-managed container port."
    },
    {
      "name": "APP_DATA_TARGET",
      "label": "Data Path Inside Container",
      "section": "readonly",
      "input_type": "text",
      "required": true,
      "default": "/data",
      "editable": false,
      "help": "Recipe-managed internal data path."
    },
    {
      "name": "APP_COMMAND",
      "label": "Compose Command Override",
      "section": "advanced",
      "input_type": "textarea",
      "rows": 2,
      "required": false,
      "default": "",
      "data_type": "yaml",
      "help": "Optional command override. Can be a string or YAML list."
    }
  ],
  "ui": {
    "sections": [
      {
        "name": "required",
        "label": "Minimum Required"
      },
      {
        "name": "optional",
        "label": "Optional",
        "collapsed": true
      },
      {
        "name": "advanced",
        "label": "Advanced",
        "collapsed": true
      },
      {
        "name": "readonly",
        "label": "Read-Only (Recipe Managed)",
        "collapsed": true
      }
    ],
    "overview": "ExampleApp is a sample self-hosted app.",
    "what_happens": [
      "One container will be created",
      "Its data will be stored in the folder you choose"
    ],
    "data_location": "Stored relative to this app's config folder unless you enter an absolute path."
  }
}
```

---

## Authoring Rules

### Keep required fields minimal

The `required` section should contain only the minimum values needed to get a working deployment.

### Use `select` for known choices

For booleans or fixed options, use `select` instead of free text.

### Use `advanced` for expert settings

Examples:

- `command`
- `entrypoint`
- daemon flags
- healthcheck toggles
- low-level tuning values

### Use `editable: false` for recipe-managed values

If a value should be visible but not editable, define it as a field and set:

```json
"editable": false
```

### Keep help practical

Good help text explains:

- what the field does
- expected format
- a short example if useful

### Do not hardcode meaningful Compose values

If a Compose value matters, define a field for it and reference that field from Compose.

---

## Reusable Prompt For Another ChatGPT Session

Use this prompt to generate an EasyDocker-compatible recipe:

> Create an EasyDocker-compatible recipe JSON.  
> Requirements:
> - valid JSON only
> - include `name`, `version`, `description`, `services`, `fields`, and `ui`
> - define `ui.sections` and make every field use one of those section names
> - use placeholders like `${FIELD_NAME}`
> - make every meaningful Compose value come from a field placeholder
> - define user-editable values in `fields`
> - use `editable: false` for recipe-managed fields that should be shown but not changed
> - use `data_type: "yaml"` for advanced fields like `command` or `entrypoint` when needed
> - include `app_links` if the app exposes a web UI
> - keep defaults beginner-friendly
> - output only the final recipe JSON

---
