# EasyDocker Field Reference

This document explains the properties available for each item inside a recipe's `fields` array.

If `SCHEMA.md` explains the full recipe format, this file focuses only on field definitions.

---

## Field Properties

Each field can contain the properties below.

### `name`

Internal variable name used by placeholders.

Example:

```json
"name": "TS_AUTHKEY"
```

This matches Compose placeholders such as:

```json
"TS_AUTHKEY": "${TS_AUTHKEY}"
```

---

### `label`

User-facing label shown in the form.

Example:

```json
"label": "Auth Key"
```

---

### `section`

Controls where the field appears in the EasyDocker form.

The section name must match one of the entries in `ui.sections`.

Common section pattern:

- `required`
- `optional`
- `advanced`
- `readonly`

Typical labels:

- `required` -> `Minimum Required`
- `optional` -> `Optional`
- `advanced` -> `Advanced`
- `readonly` -> `Read-Only (Recipe Managed)`

Example:

```json
"section": "required"
```

---

### `input_type`

Controls how the field is rendered.

Currently supported:

- `text`
- `password`
- `textarea`
- `select`

Example:

```json
"input_type": "password"
```

---

### `required`

Whether the user must provide a value.

Example:

```json
"required": true
```

Note: a required field with a usable default can still submit without user input.

---

### `default`

Default value used in the form and during Compose generation.

Example:

```json
"default": "./data"
```

Defaults may also reference placeholders, for example:

```json
"default": "${PROJECT_NAME}_server"
```

---

### `help`

Help text shown when the user clicks the `?` button.

Example:

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

Examples:

```json
"help": "Optional timezone such as <code>Asia/Dubai</code> or <code>Etc/UTC</code>."
```

```json
"help": "Read the <a href=\"https://example.com\" target=\"_blank\" rel=\"noopener noreferrer\">official docs</a>."
```

---

### `editable`

Whether the user can change the field.

Example:

```json
"editable": false
```

Behavior:

- omitted means editable
- `false` means visible but read-only
- recipe-managed values usually belong in the `readonly` section and set `editable: false`

---

### `rows`

Optional. Used only with `textarea`.

Example:

```json
"rows": 2
```

---

### `data_type`

Optional. Controls how EasyDocker interprets the submitted value.

Current useful value:

- `yaml`

Example:

```json
"data_type": "yaml"
```

Use this for advanced fields such as:

- `command`
- `entrypoint`

When `data_type` is `yaml`, the field may accept:

- a plain string
- a YAML list
- a YAML object

---

### `options`

Optional. Used only when `input_type` is `select`.

Each option should contain:

- `label`
- `value`

Example:

```json
"options": [
  {
    "label": "Yes",
    "value": "true"
  },
  {
    "label": "No",
    "value": "false"
  }
]
```

---

## Quick Reference Table

| Property | Meaning | Required | Example |
|---|---|---:|---|
| `name` | Internal placeholder name | Yes | `TS_AUTHKEY` |
| `label` | User-facing form label | Yes | `Auth Key` |
| `section` | Section key from `ui.sections` | Yes | `required` |
| `input_type` | Field render type | Yes | `text`, `password`, `textarea`, `select` |
| `required` | Whether user must provide a value | Yes | `true` |
| `default` | Default field value | Usually | `./data` |
| `help` | Help shown in UI | Recommended | `Host path where data is stored.` |
| `editable` | Whether the value can be changed | Optional | `false` |
| `rows` | Textarea height | Optional | `2` |
| `data_type` | Parse mode for value | Optional | `yaml` |
| `options` | Dropdown options | Only for `select` | list of label/value pairs |

---

## Example Field Definitions

### Password Field

```json
{
  "name": "TS_AUTHKEY",
  "label": "Auth Key",
  "section": "required",
  "input_type": "password",
  "required": true,
  "default": "",
  "help": "Visit <a href=\"https://login.tailscale.com/admin/settings/keys\" target=\"_blank\" rel=\"noopener noreferrer\">the Tailscale auth keys page</a>, generate a key, and paste it here."
}
```

### Text Field

```json
{
  "name": "TS_HOSTNAME",
  "label": "Advertised Hostname",
  "section": "optional",
  "input_type": "text",
  "required": false,
  "default": "",
  "help": "Name you want to give this device in Tailscale.<br>Example: <code>UgreenDXP</code>."
}
```

### Textarea Field

```json
{
  "name": "TS_EXTRA_ARGS",
  "label": "Extra tailscale up Arguments",
  "section": "advanced",
  "input_type": "textarea",
  "rows": 2,
  "required": false,
  "default": "",
  "help": "Optional extra arguments passed to <code>tailscale up</code>."
}
```

### Select Field

```json
{
  "name": "TS_ACCEPT_DNS",
  "label": "Accept DNS",
  "section": "optional",
  "input_type": "select",
  "required": true,
  "default": "true",
  "options": [
    { "label": "Yes", "value": "true" },
    { "label": "No", "value": "false" }
  ],
  "help": "Use Tailscale DNS so MagicDNS names work on this device."
}
```

### Read-Only Field

```json
{
  "name": "TS_STATE_DIR",
  "label": "State Directory Inside Container",
  "section": "readonly",
  "input_type": "text",
  "required": true,
  "default": "/var/lib/tailscale",
  "editable": false,
  "help": "Recipe-managed internal state directory used by the container."
}
```

### YAML-Aware Advanced Field

```json
{
  "name": "TAILSCALE_COMMAND",
  "label": "Compose Command Override",
  "section": "advanced",
  "input_type": "textarea",
  "rows": 2,
  "required": false,
  "default": "",
  "data_type": "yaml",
  "help": "Optional command override.<br>Can be a plain string or YAML list."
}
```

---

## CSV-Style Mental Model

If JSON feels too dense, think of each field as one spreadsheet row with these columns:

```text
name,label,section,input_type,required,default,help,editable,rows,data_type,options
```

Example:

```text
TS_AUTHKEY,Auth Key,required,password,true,,Generate a Tailscale auth key and paste it here.,true,,,
TS_ROUTES,Advertised Routes,optional,text,false,,Optional subnet routes to advertise,true,,,
TS_USERSPACE,Userspace Networking,advanced,select,true,false,Usually leave this disabled when /dev/net/tun is available.,true,,,Disabled=false|Enabled=true
TS_STATE_DIR,State Directory Inside Container,readonly,text,true,/var/lib/tailscale,Recipe-managed internal state directory,false,,,
```

The actual recipe format remains JSON. This is just the mental shortcut.

---

## Authoring Advice

- define `ui.sections` first
- keep `required` minimal
- use `select` for fixed choices
- use `advanced` for expert options
- use `readonly` plus `editable: false` for recipe-managed values
- use `data_type: "yaml"` for `command` and `entrypoint`
- make help practical and user-focused
- remember that every meaningful Compose value should be backed by a field

---
