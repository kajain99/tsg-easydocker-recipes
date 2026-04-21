# EasyDocker Recipes

Official recipe catalog for EasyDocker.

This repository contains the JSON recipes that EasyDocker uses to:

- build app forms
- show editable and recipe-managed values
- generate Docker Compose files
- deploy supported apps
- refresh recipes independently of app releases

## What A Recipe Does

Each recipe is the source of truth for three things:

1. the Compose template
2. the form fields shown to the user
3. the UI sections and help text used to present those fields

EasyDocker follows two important rules:

- every meaningful Compose value must come from a field placeholder
- every field must belong to a section defined in `ui.sections`

That means users can always see what the recipe is doing, whether a value is editable or read-only.

## Repository Structure

Recipes live in:

```text
recipes/
```

Important files:

- `recipes/index.json` - version map used by EasyDocker refresh
- `recipes/*.json` - individual app recipes
- `SCHEMA.md` - full recipe format
- `FIELDS.md` - field-by-field reference
- `recipe-template.json` - starter template for new recipes

## Current Recipe Model

An EasyDocker recipe includes:

- metadata such as `name`, `version`, and `description`
- Compose content such as `services`, `volumes`, `networks`, `configs`, or `secrets`
- a `fields` array defining every user-visible value
- a `ui` object defining section order, labels, and helper content
- optional `app_links` for launching apps after deploy

Typical section layout:

- `required` -> shown as `Minimum Required`
- `optional`
- `advanced`
- `readonly` -> shown as `Read-Only (Recipe Managed)`

Those section names are defined by the recipe itself in `ui.sections`.

## Help Text

Field help supports a small safe subset of HTML.

Useful examples:

- links with `<a>`
- inline examples with `<code>`
- line breaks with `<br>`
- light emphasis with `<strong>` or `<em>`

Keep help practical and short. Write it for users filling the form, not for developers reading the JSON.

## Authoring Rules

When creating or updating recipes:

- keep defaults beginner-friendly
- keep `Minimum Required` truly minimal
- use `select` for known choices when possible
- put expert-only inputs in `advanced`
- put recipe-managed values in `readonly` with `editable: false`
- do not hardcode meaningful Compose values directly in the Compose body
- bump the recipe `version` whenever the recipe changes

## For Contributors

If you add or update a recipe:

1. start from `recipe-template.json`
2. define `ui.sections` first
3. define fields for every meaningful Compose value
4. wire those fields into Compose placeholders
5. test the recipe in EasyDocker
6. update `recipes/index.json`

## For EasyDocker Users

You normally do not need to clone this repository manually.

EasyDocker can refresh recipes directly from the official source.

## License

This repository is licensed under the MIT License. See [LICENSE](LICENSE).
