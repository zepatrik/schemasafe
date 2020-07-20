# `@exodus/schemasafe`

A code-generating [JSON Schema](https://json-schema.org/) validator that attempts to be reasonably secure.

Supports [draft-04/06/07/2019-09](doc/Specification-support.md).

[![Node CI Status](https://github.com/ExodusMovement/schemasafe/workflows/Node%20CI/badge.svg)](https://github.com/ExodusMovement/schemasafe/actions)
[![npm](https://img.shields.io/npm/v/@exodus/schemasafe.svg)](https://www.npmjs.com/package/@exodus/schemasafe)
[![codecov](https://codecov.io/gh/ExodusMovement/schemasafe/branch/master/graph/badge.svg)](https://codecov.io/gh/ExodusMovement/schemasafe)
[![Total alerts](https://img.shields.io/lgtm/alerts/g/ExodusMovement/schemasafe.svg?logo=github)](https://lgtm.com/projects/g/ExodusMovement/schemasafe/alerts/)

## Features

* Converts schemas to self-contained JavaScript files, can be used in the build process.\
  _Integrates nicely with bundlers, so one won't need to generate code in runtime and that works with CSP._
* Optional `requireValidation: true` mode enforces full validation of the input object.\
  **Using `mode: "strong"` is recommended which combines that option with other extra schema safety checks.**
* Does not fail open on unknown or unprocessed keywords — instead throws at build time if schema
  was not fully understood.
  _That is implemented by tracking processed keywords and ensuring that none remain uncovered._
* Does not fail open on schema problems -- instead throws at build time.\
  _E.g. it will detect mistakes like `{type: "array", "maxLength": 2}`._
* Under 1700 lines of code, non-minified.
* Uses secure code generation approach to prevent data from schema from leaking into the generated
  code without being JSON-wrapped.
* 0 dependencies
* Very fast
* Supports JSON Schema draft-04/06/07/2019-09 and the `discriminator` OpenAPI keyword.
* Can assign defaults and/or remove additional properties when schema allows to do that safely.
  Throws at build time if those options are used with schemas that don't allow to do that safely.

## Installation

```sh
npm install --save @exodus/schemasafe
```

## Usage

Simply pass a schema to compile it

```js
const { validator } = require('@exodus/schemasafe')

const validate = validator({
  type: 'object',
  required: ['hello'],
  properties: {
    hello: {
      type: 'string'
    }
  }
})

console.log('should be valid', validate({ hello: 'world' }))
console.log('should not be valid', validate({}))
```

## Custom formats

`@exodus/schemasafe` supports the formats specified in JSON schema v4 (such as date-time).
If you want to add your own custom formats pass them as the formats options to the validator

```js
const validate = validator({
  type: 'string',
  format: 'only-a'
}, {
  formats: {
    'only-a': /^a+$/
  }
})

console.log(validate('aa')) // true
console.log(validate('ab')) // false
```

## External schemas

You can pass in external schemas that you reference using the `$ref` attribute as the `schemas` option

```js
const ext = {
  type: 'string'
}

const schema = {
  $ref: 'ext#' // references another schema called ext
}

// pass the external schemas as an option
const validate = validator(schema, { schemas: { ext: ext }})

console.log(validate('hello')) // true
console.log(validate(42)) // false
```

## Enabling errors shows information about the source of the error

When the `includeErrors` option is set to `true`, `@exodus/schemasafe` also outputs:

- `keywordLocation`: a JSON pointer string as an URI fragment indicating which sub-schema failed, e.g.
  `#/properties/item/type`
- `instanceLocation`: a JSON pointer string as an URI fragment indicating which property of the object
  failed validation, e.g. `#/item`

```js
const schema = {
  type: 'object',
  required: ['hello'],
  properties: {
    hello: {
      type: 'string'
    }
  }
}
const validate = validator(schema, { includeErrors: true })

validate({ hello: 100 });
console.log(validate.errors)
// [ { keywordLocation: '#/properties/hello/type', instanceLocation: '#/hello' } ]
```

Only the first error is reported by default unless `allErrors` option is also set to `true` in
addition to `includeErrors`.

See [Error handling](./doc/Error-handling.md) for more information.

## Generate Modules

To compile a validator function to an IIFE, call `validate.toModule()`:

```js
const { validator } = require('@exodus/schemasafe')

const schema = {
  type: 'string',
  format: 'hex'
}

// This works with custom formats as well.
const formats = {
  hex: (value) => /^0x[0-9A-Fa-f]*$/.test(value),
}

const validate = validator(schema, { formats })

console.log(validate.toModule())
/** Prints:
 * (function() {
 * 'use strict'
 * const format0 = (value) => /^0x[0-9A-Fa-f]*$/.test(value);
 * return (function validate(data) {
 *   if (data === undefined) data = null
 *   if (!(typeof data === "string")) return false
 *   if (!format0(data)) return false
 *   return true
 * })})();
 */
```

## Performance

`@exodus/schemasafe` uses code generation to turn a JSON schema into javascript code that is easily
optimizeable by v8.

See [Performance](./doc/Performance.md) for information on options that might affect performace
both ways.

## Previous work

This is based on a heavily rewritten version of the amazing (but outdated)
[is-my-json-valid](https://github.com/mafintosh/is-my-json-valid) by
[@mafintosh](https://github.com/mafintosh/is-my-json-valid).

Compared to `is-my-json-valid`, `@exodus/schemasafe` adds security-first design, many new features,
newer spec versions support, slimmer and more maintainable code, 0 dependencies, self-contained JS
module generation, fixes bugs and adds better test coverage, and drops support for outdated Node.js
versions.

## License

MIT
