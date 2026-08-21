# jsonschema 4.26.0 — advanced technical reference / feature and deployment catalog for LLM coding agents

## Version / source anchors

**Release anchor:** `jsonschema==4.26.0`  
**Release date:** 2026-01-07  
**Python floor:** Python `>=3.10`  
**License:** MIT  
**Status:** Production/Stable  
**Reference date:** 2026-08-20

This reference is intentionally pinned to the latest stable `jsonschema` release as of the reference date. It treats Draft 2020-12 as the default design target for new schemas unless an interoperability requirement explicitly requires an older draft.

### Source-of-truth hierarchy

Use sources in this order when exact behavior matters:

1. released `jsonschema 4.26.0` package and its public API;
2. version-pinned `jsonschema 4.26.0` documentation;
3. released package metadata / `pyproject.toml`;
4. official changelog through `v4.26.0`;
5. the JSON Schema specification for dialect semantics;
6. `referencing` documentation for reference-resolution behavior;
7. current `main` only for forward-looking information, never as released API.

Primary anchors:

- `jsonschema 4.26.0` docs: https://python-jsonschema.readthedocs.io/en/v4.26.0/
- Stable docs: https://python-jsonschema.readthedocs.io/en/stable/
- PyPI release: https://pypi.org/project/jsonschema/4.26.0/
- Source: https://github.com/python-jsonschema/jsonschema
- Changelog: https://github.com/python-jsonschema/jsonschema/blob/main/CHANGELOG.rst
- Validator API: https://python-jsonschema.readthedocs.io/en/stable/api/jsonschema/validators/
- Validator protocol: https://python-jsonschema.readthedocs.io/en/stable/api/jsonschema/protocols/
- Exceptions: https://python-jsonschema.readthedocs.io/en/stable/api/jsonschema/exceptions/
- Validator creation/extension: https://python-jsonschema.readthedocs.io/en/stable/creating/
- Referencing guide: https://python-jsonschema.readthedocs.io/en/stable/referencing/
- FAQ: https://python-jsonschema.readthedocs.io/en/stable/faq/
- JSON Schema specification: https://json-schema.org/specification

---

## Capability inventory

`jsonschema` provides:

- full validation support for JSON Schema Draft 2020-12;
- Draft 2019-09;
- Draft 7;
- Draft 6;
- Draft 4;
- Draft 3;
- schema metaschema validation;
- automatic validator-class selection from `$schema`;
- lazy iteration over all validation errors;
- single-error validation;
- boolean validity testing;
- structured, navigable validation errors;
- nested error contexts for combinators;
- best-error heuristics;
- error-tree construction;
- configurable format checking;
- custom format definitions;
- immutable type-checker customization;
- custom validation keywords;
- extension of existing validators;
- creation of entirely new validator dialects/classes;
- modern `$ref`, `$dynamicRef`, `$anchor`, and `$dynamicAnchor` resolution through `referencing`;
- in-memory schema registries;
- filesystem/database/network/custom schema retrieval;
- custom reference-resource loading;
- draft transitions during reference traversal;
- support for schemas and instances represented as Python objects;
- validation of JSON-compatible YAML/TOML-decoded data;
- a typed `Validator` protocol for application interfaces;
- integration with `check-jsonschema` for CLI/pre-commit workflows.

The library is a validator. It is **not**:

- a JSON parser;
- a schema authoring language;
- a data coercion system;
- a default-value application engine;
- a serializer;
- a canonicalizer;
- a database schema engine;
- a network policy engine;
- an automatic remote-schema downloader in the modern API;
- a general-purpose transformation framework.

---

## Package architecture and dependency roles

The released 4.26.0 package requires:

- `attrs>=22.2.0`
- `jsonschema-specifications>=2023.03.6`
- `referencing>=0.28.4`
- `rpds-py>=0.25.0`

Conceptual architecture:

```text
application
  |
  +--> jsonschema.Validator / validate()
          |
          +--> JSON Schema keyword implementations
          |
          +--> TypeChecker
          |
          +--> FormatChecker
          |
          +--> jsonschema-specifications
          |      -> official metaschemas / dialect resources
          |
          +--> referencing.Registry
                 -> Resource
                 -> Specification
                 -> $ref / anchors / dynamic scope
                 -> optional application-owned retrieval
```

`rpds-py` supplies persistent immutable data structures used internally by the modern stack. Application code normally should not couple itself to these internal representations.

**Agent rule:** treat `jsonschema` + `referencing` as separate responsibilities. `jsonschema` validates instances. `referencing` locates and interprets referenced schema resources.

---

# Comprehensive documentation map

## 0) Mental model and design rules
## 1) Installation, extras, dependencies, and version pinning
## 2) JSON data model and Python object mapping
## 3) Supported drafts and dialect selection
## 4) Fast-start validation patterns
## 5) `jsonschema.validate`
## 6) `validator_for`
## 7) Versioned validator classes
## 8) The `Validator` protocol
## 9) Validator construction and lifecycle
## 10) `check_schema`
## 11) `validate`, `iter_errors`, and `is_valid`
## 12) `evolve` and recursive validation
## 13) Draft 2020-12 keyword inventory
## 14) Boolean schemas
## 15) Type semantics and Python mappings
## 16) Numeric keywords
## 17) String keywords and regular expressions
## 18) Object keywords
## 19) Array keywords
## 20) Composition keywords
## 21) Conditional/dependency keywords
## 22) `unevaluatedProperties` and `unevaluatedItems`
## 23) Annotations versus assertions
## 24) `$id`, `$schema`, `$anchor`, `$dynamicAnchor`
## 25) `$ref` / `$dynamicRef` architecture
## 26) `referencing.Registry`
## 27) `referencing.Resource` and specification identity
## 28) In-memory schema bundles
## 29) Filesystem retrieval
## 30) HTTP/custom retrieval
## 31) YAML and non-JSON schema resources
## 32) Migrating from `RefResolver`
## 33) Format validation
## 34) Format extras and optional dependencies
## 35) Custom `FormatChecker`
## 36) `TypeChecker`
## 37) Custom Python type semantics
## 38) Validation errors
## 39) Error paths and JSON paths
## 40) Nested error context
## 41) `ErrorTree`
## 42) `best_match` and relevance
## 43) Creating validator classes
## 44) Extending validator classes
## 45) Custom keywords
## 46) Overriding existing keywords safely
## 47) Recursive keyword implementations and `descend`
## 48) Custom metaschemas/dialects
## 49) Default insertion and mutation
## 50) Non-JSON instances: YAML/TOML/etc.
## 51) Performance and validator reuse
## 52) Reference-registry performance
## 53) Concurrency and immutability
## 54) Security and resource governance
## 55) Network-reference security
## 56) Schema trust and custom code
## 57) CLI / `check-jsonschema`
## 58) Testing strategy
## 59) JSON Schema Test Suite
## 60) Draft migration
## 61) 4.x deprecations and compatibility traps
## 62) 4.26.0-specific behavior
## 63) Integration patterns
## 64) LLM-agent decision playbook
## 65) Anti-pattern inventory
## 66) Final deployment checklist

---

# 0) Mental model and design rules

A JSON Schema validator evaluates a Python object representing an **instance** against a Python object representing a **schema** interpreted under a particular JSON Schema dialect.

```text
serialized JSON/YAML/TOML/etc.
        |
        v
application parser / decoder
        |
        v
Python JSON-model objects
        |
        +--> schema -------------------+
        |                              |
        |                        dialect / registry
        |                              |
        +--> instance                  |
                |                      |
                +------> Validator <---+
                           |
                           +--> valid
                           |
                           +--> ValidationError(s)
```

Important consequences:

1. `jsonschema` does not normally parse JSON text itself.
2. Lexical properties of JSON are already gone by the time validation starts.
3. Duplicate JSON object keys cannot be detected after a normal `json.loads()` has collapsed them.
4. JSON number lexical spelling is not preserved.
5. Validation is defined around the JSON data model, not arbitrary Python objects.
6. `$ref` is special because resource identity and retrieval require URI-aware logic.
7. A schema's dialect is semantic state, not documentation decoration.
8. Format validation is opt-in.
9. Validation does not mutate the instance.
10. Modern reference resolution should use `referencing.Registry`, not `RefResolver`.

**Agent rule:** first decide whether the requirement is validation, parsing, normalization, transformation, or canonicalization. Do not make `jsonschema` own responsibilities outside validation unless an explicit custom extension is justified.

---

# 1) Installation, extras, dependencies, and version pinning

Basic install:

```bash
python -m pip install 'jsonschema==4.26.0'
```

With all current format dependencies:

```bash
python -m pip install 'jsonschema[format]==4.26.0'
```

With the project's non-GPL-oriented direct format dependency set:

```bash
python -m pip install 'jsonschema[format-nongpl]==4.26.0'
```

Python requirement:

```text
Python >= 3.10
```

Supported classifiers include CPython and PyPy, with Python 3.10 through 3.14 declared by the current project metadata.

## 1.1 Pinning posture

Pin exact versions when:

- validation output is persisted;
- schema compatibility is a protocol contract;
- error shape is machine-consumed;
- reference traversal behavior affects generated artifacts;
- format dependencies affect pass/fail behavior;
- schemas are externally supplied;
- reproducible builds are required.

At minimum pin `jsonschema`. For strongly reproducible validation, also lock the resolved versions of:

```text
jsonschema-specifications
referencing
rpds-py
format-extra dependencies used by the project
```

because those components can affect dialect resources, reference semantics, or format assertions.

---

# 2) JSON data model and Python object mapping

JSON Schema is fundamentally defined over:

```text
null
boolean
number
string
array
object
```

Typical Python mapping:

| JSON | Python |
|---|---|
| null | `None` |
| boolean | `bool` |
| integer/number | `int`, `float`, related numeric types as supported |
| string | `str` |
| array | `list` |
| object | `dict` |

For current Draft 2020-12 validation:

- `True` is a boolean, not an integer for schema purposes despite Python's `bool` subclassing `int`;
- integral floats such as `1.0` satisfy JSON Schema `"integer"` semantics;
- ordinary lists are arrays;
- ordinary dicts are objects.

## 2.1 Stay inside the JSON model

A Python object may exist which could never result from JSON decoding:

```python
{1: "non-string key"}
{("tuple",): "value"}
complex(1, 2)
set([1, 2])
```

The JSON Schema specification does not necessarily define behavior for such values.

**Agent rule:** preprocess/coerce application-domain objects into a JSON-model representation before validation when cross-language semantics matter.

---

# 3) Supported drafts and dialect selection

Built-in validator classes:

```python
from jsonschema import (
    Draft202012Validator,
    Draft201909Validator,
    Draft7Validator,
    Draft6Validator,
    Draft4Validator,
    Draft3Validator,
)
```

Recommended new-schema dialect:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema"
}
```

Always include `$schema` in authored schemas unless there is a strong reason not to.

## 3.1 Why explicit `$schema` matters

Without `$schema`:

- `validator_for()` falls back to a default;
- interpretation can become version-dependent;
- reference resources may need an externally supplied specification;
- ambiguous schemas may have different semantics between drafts.

Draft identity affects:

- available keywords;
- tuple validation;
- `$ref` siblings;
- dynamic references;
- integer semantics in historical drafts;
- dependency keywords;
- unevaluated keywords;
- identifier semantics.

---

# 4) Fast-start validation patterns

## 4.1 One-off validation

```python
from jsonschema import validate

schema = {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "required": ["name"],
    "properties": {
        "name": {"type": "string"},
    },
}

validate(instance={"name": "Ada"}, schema=schema)
```

Invalid:

```python
from jsonschema.exceptions import ValidationError

try:
    validate(instance={"name": 12}, schema=schema)
except ValidationError as exc:
    print(exc.message)
```

## 4.2 Repeated validation

Prefer an explicit validator object:

```python
from jsonschema import Draft202012Validator

Draft202012Validator.check_schema(schema)
validator = Draft202012Validator(schema)

for record in records:
    errors = list(validator.iter_errors(record))
```

This avoids repeatedly performing top-level schema-class discovery and metaschema validation for the same schema.

---

# 5) `jsonschema.validate`

Public API:

```python
jsonschema.validate(instance, schema, cls=None, *args, **kwargs)
```

Operational flow:

1. determine a validator class if `cls` is omitted;
2. validate the schema against its metaschema;
3. instantiate the validator;
4. validate the instance;
5. raise one `ValidationError` if invalid.

Example:

```python
from jsonschema import validate, Draft7Validator

validate(instance=data, schema=schema)
validate(instance=data, schema=schema, cls=Draft7Validator)
```

## 5.1 When to use it

Good:

- scripts;
- one-off checks;
- examples;
- tests with one instance.

Avoid in hot loops where the schema is unchanged.

## 5.2 Schema validation behavior

`validate()` validates the schema itself first. This is useful for safety and diagnostics, but means repeated one-off calls repeat work.

---

# 6) `validator_for`

```python
from jsonschema.validators import validator_for

ValidatorClass = validator_for(schema)
ValidatorClass.check_schema(schema)
validator = ValidatorClass(schema)
```

Selection is based on `$schema`.

Examples:

```python
schema_2020 = {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "integer",
}

schema_7 = {
    "$schema": "http://json-schema.org/draft-07/schema#",
    "type": "integer",
}
```

Without `$schema`, pass a deliberate default if fallback is acceptable:

```python
ValidatorClass = validator_for(schema, default=Draft7Validator)
```

Otherwise the latest supported draft is used.

**Agent rule:** explicit dialect beats implicit fallback.

---

# 7) Versioned validator classes

Current keyword implementation counts in 4.26.0:

| Validator | Implemented validation keywords |
|---|---:|
| `Draft202012Validator` | 36 |
| `Draft201909Validator` | 36 |
| `Draft7Validator` | 32 |
| `Draft6Validator` | 31 |
| `Draft4Validator` | 26 |
| `Draft3Validator` | 21 |

Draft 2020-12 validator keyword surface:

```text
$dynamicRef
$ref
additionalProperties
allOf
anyOf
const
contains
dependentRequired
dependentSchemas
enum
exclusiveMaximum
exclusiveMinimum
format
if
items
maxItems
maxLength
maxProperties
maximum
minItems
minLength
minProperties
minimum
multipleOf
not
oneOf
pattern
patternProperties
prefixItems
properties
propertyNames
required
type
unevaluatedItems
unevaluatedProperties
uniqueItems
```

Not every JSON Schema keyword is an assertion keyword implemented in `VALIDATORS`. Annotation/core keywords such as `$schema`, `$id`, `$defs`, `title`, `description`, `default`, and `examples` have different roles.

---

# 8) The `Validator` protocol

Type applications against the public protocol rather than concrete implementation internals:

```python
from jsonschema.protocols import Validator

def validate_record(validator: Validator, record: object) -> None:
    validator.validate(record)
```

Protocol class attributes include:

```text
META_SCHEMA
VALIDATORS
TYPE_CHECKER
FORMAT_CHECKER
ID_OF
```

Instance-level important state includes:

```text
schema
format_checker
registry / reference behavior
```

## 8.1 Do not subclass built-in validators

Subclassing validator classes is explicitly not a supported public extension API and has warned since 4.12.

Use:

```python
jsonschema.validators.extend(...)
jsonschema.validators.create(...)
```

instead.

---

# 9) Validator construction and lifecycle

Draft 2020-12 shape:

```python
Draft202012Validator(
    schema,
    resolver=None,              # deprecated path
    format_checker=None,
    *,
    registry=...,
)
```

Recommended:

```python
validator = Draft202012Validator(
    schema=schema,
    format_checker=Draft202012Validator.FORMAT_CHECKER,
    registry=registry,
)
```

The schema is assumed valid at construction time.

Validate it beforehand:

```python
Draft202012Validator.check_schema(schema)
```

## 9.1 Reuse

Validator instances are suitable for repeated validation under the same:

- schema;
- reference registry;
- format policy;
- type semantics.

Create separate validators when these policy boundaries differ.

---

# 10) `check_schema`

```python
Draft202012Validator.check_schema(schema)
```

Raises:

```python
jsonschema.exceptions.SchemaError
```

Use at:

- application startup;
- schema ingestion;
- schema registry publication;
- build time;
- before caching a validator.

Do not wait for arbitrary instance validation to discover that the schema itself is malformed.

---

# 11) `validate`, `iter_errors`, and `is_valid`

## 11.1 `validate`

```python
validator.validate(instance)
```

Raises a `ValidationError` when invalid.

Use when:
- fail-fast behavior is acceptable;
- only one representative error is needed.

## 11.2 `iter_errors`

```python
errors = validator.iter_errors(instance)
```

Lazily yields all discoverable errors.

```python
for error in validator.iter_errors(instance):
    print(error.json_path, error.message)
```

Use for:
- UI diagnostics;
- batch validation;
- schema-development tooling;
- structured error reports.

Do not convert to a list if early termination is enough.

## 11.3 `is_valid`

```python
if validator.is_valid(instance):
    ...
```

Returns boolean.

Use for:
- predicates;
- filtering;
- branch selection.

Do not call `is_valid()` and then `iter_errors()` on every invalid record if one traversal can directly produce the errors you need.

---

# 12) `evolve` and recursive validation

Create a related validator with changed state:

```python
other = validator.evolve(schema={"type": "number"})
```

`evolve()` preserves compatible policy such as reference behavior.

Important: the returned object satisfies the Validator protocol but is not guaranteed to be the same concrete class. A referenced schema can identify a different dialect.

## 12.1 Why `evolve` matters

Use it instead of deprecated second-schema arguments to validation methods.

Good:

```python
child_validator = validator.evolve(schema=subschema)
```

Avoid legacy patterns that pass another schema directly into methods which now deprecate that usage.

---

# 13) Draft 2020-12 keyword inventory

The following sections group the high-value assertion keywords by role.

---

# 14) Boolean schemas

JSON Schema allows the schema itself to be:

```json
true
```

or:

```json
false
```

Semantics:

- `true` accepts every instance;
- `false` accepts no instance.

Python:

```python
Draft202012Validator(True).is_valid(anything)   # True
Draft202012Validator(False).is_valid(anything)  # False
```

Use boolean schemas to simplify generated compositions and policy branches.

---

# 15) Type semantics and Python mappings

Example:

```json
{"type": "string"}
```

Multiple accepted types:

```json
{"type": ["string", "null"]}
```

Supported JSON Schema types:

```text
null
boolean
object
array
number
integer
string
```

## 15.1 Python specifics

Current type behavior intentionally accounts for Python quirks.

For example:

```python
True
```

is not treated as a JSON Schema integer merely because:

```python
isinstance(True, int)
```

is true in Python.

Similarly, Draft 2020-12 integer semantics consider mathematically integral values such as:

```python
1.0
```

integer-valued.

Do not write application code which reimplements these distinctions before validation unless coercion is an explicit separate stage.

---

# 16) Numeric keywords

Core numeric assertions:

```text
minimum
exclusiveMinimum
maximum
exclusiveMaximum
multipleOf
```

Example:

```json
{
  "type": "number",
  "minimum": 0,
  "exclusiveMaximum": 100,
  "multipleOf": 0.5
}
```

## 16.1 Floating-point caution

Python floating point can make decimal arithmetic unintuitive.

If exact decimal semantics are required:

- parse with `decimal.Decimal`;
- confirm custom numeric types behave as intended with the validator's TypeChecker;
- test `multipleOf` and bounds exhaustively around decimal edges.

## 16.2 Non-JSON numbers

`NaN`, `Infinity`, and `-Infinity` are not legal JSON numbers.

If Python objects containing such values reach the validator directly, you are already outside strict JSON parsing semantics.

Reject them at the input boundary when protocol fidelity matters.

---

# 17) String keywords and regular expressions

Core string assertions:

```text
minLength
maxLength
pattern
format
```

Example:

```json
{
  "type": "string",
  "minLength": 1,
  "maxLength": 64,
  "pattern": "^[A-Z][A-Za-z0-9_-]*$"
}
```

## 17.1 Regex dialect caution

JSON Schema recommends an ECMA-262-compatible regular-expression subset for portability.

Python `jsonschema` uses Python regular expressions for `pattern` / `patternProperties` behavior and for the `regex` format checker.

Therefore a pattern that works in Python may not be portable to another JSON Schema implementation.

**Agent rule:** if schemas are cross-language contracts, author regexes in a conservative cross-engine subset and test against the target implementations.

---

# 18) Object keywords

Major object assertions:

```text
properties
patternProperties
additionalProperties
required
propertyNames
minProperties
maxProperties
dependentRequired
dependentSchemas
unevaluatedProperties
```

## 18.1 `properties`

```json
{
  "type": "object",
  "properties": {
    "name": {"type": "string"},
    "age": {"type": "integer"}
  }
}
```

`properties` does not require a property to exist.

Add:

```json
{"required": ["name"]}
```

when presence matters.

## 18.2 `additionalProperties`

Closed object:

```json
{
  "type": "object",
  "properties": {
    "id": {"type": "string"}
  },
  "additionalProperties": false
}
```

Or constrain extra values:

```json
{
  "additionalProperties": {"type": "string"}
}
```

## 18.3 `patternProperties`

```json
{
  "patternProperties": {
    "^x-": {"type": "string"}
  }
}
```

Remember Python-regex portability constraints.

## 18.4 `propertyNames`

Validates property names themselves:

```json
{
  "propertyNames": {
    "pattern": "^[a-z][a-z0-9_]*$"
  }
}
```

---

# 19) Array keywords

Draft 2020-12 array assertions:

```text
prefixItems
items
contains
minContains
maxContains
minItems
maxItems
uniqueItems
unevaluatedItems
```

Not all keywords above appear directly in `VALIDATORS`; some are consumed through related implementations/spec semantics.

## 19.1 Tuple-like prefixes

Draft 2020-12:

```json
{
  "type": "array",
  "prefixItems": [
    {"type": "string"},
    {"type": "integer"}
  ],
  "items": false
}
```

This replaces the older tuple-schema form where `items` could itself be an array.

## 19.2 Homogeneous arrays

```json
{
  "type": "array",
  "items": {"type": "integer"}
}
```

## 19.3 `contains`

```json
{
  "type": "array",
  "contains": {"const": "primary"}
}
```

Drafts 2019-09/2020-12 support `minContains` / `maxContains` semantics.

## 19.4 `uniqueItems`

```json
{"uniqueItems": true}
```

JSON Schema equality differs from naive Python equality in important cases such as booleans versus `0`/`1`. The library has had explicit fixes to preserve specification equality semantics.

---

# 20) Composition keywords

```text
allOf
anyOf
oneOf
not
```

Examples:

```json
{
  "allOf": [
    {"type": "number"},
    {"minimum": 0}
  ]
}
```

```json
{
  "anyOf": [
    {"type": "string"},
    {"type": "integer"}
  ]
}
```

```json
{
  "oneOf": [
    {"const": "a"},
    {"const": "b"}
  ]
}
```

```json
{"not": {"type": "null"}}
```

## 20.1 Error contexts

Combinators frequently produce a top-level error whose `.context` contains the child failures which explain why branches failed.

Do not display only:

```text
is not valid under any of the given schemas
```

when a useful UI can inspect `.context`.

---

# 21) Conditional and dependency keywords

## 21.1 `if` / `then` / `else`

```json
{
  "if": {
    "properties": {
      "country": {"const": "US"}
    },
    "required": ["country"]
  },
  "then": {
    "required": ["state"]
  },
  "else": {
    "required": ["region"]
  }
}
```

## 21.2 `dependentRequired`

```json
{
  "dependentRequired": {
    "credit_card": ["billing_address"]
  }
}
```

## 21.3 `dependentSchemas`

```json
{
  "dependentSchemas": {
    "credit_card": {
      "required": ["billing_address"]
    }
  }
}
```

Older drafts use the combined `dependencies` keyword instead.

---

# 22) `unevaluatedProperties` and `unevaluatedItems`

These keywords depend on what other applicable schema branches have already evaluated.

Example:

```json
{
  "allOf": [
    {
      "properties": {
        "a": {"type": "integer"}
      }
    }
  ],
  "unevaluatedProperties": false
}
```

They are useful for composing closed schemas without manually repeating every property across subschema boundaries.

They are also among the trickier keywords for custom validators and extension logic because evaluation annotations matter.

4.24.0 fixed an `unevaluatedProperties` interaction with `additionalProperties`, which is a reminder to include these cases in upgrade regression suites.

---

# 23) Annotations versus assertions

Common annotation/core keywords include:

```text
title
description
default
examples
readOnly
writeOnly
deprecated
$content... metadata where applicable
$id
$schema
$defs
```

Not all keywords cause validation failure.

## 23.1 `default` does not mutate

```json
{
  "type": "object",
  "properties": {
    "enabled": {
      "type": "boolean",
      "default": true
    }
  }
}
```

Validating:

```python
obj = {}
validator.validate(obj)
```

does not add:

```python
{"enabled": True}
```

Mutation is separate from validation.

## 23.2 `readOnly` / `writeOnly`

These are annotations. They do not automatically know whether your application is in request or response mode.

Enforce application-specific read/write policy separately.

---

# 24) `$schema`, `$id`, `$anchor`, `$dynamicAnchor`

## 24.1 `$schema`

Identifies dialect:

```json
"$schema": "https://json-schema.org/draft/2020-12/schema"
```

## 24.2 `$id`

Establishes canonical resource identity/base URI:

```json
"$id": "https://schemas.example.com/customer"
```

Do not assume that an `$id` URI is fetchable over the network. URI identity and retrieval location are separate concepts.

## 24.3 `$anchor`

Named local reference target:

```json
"$anchor": "address"
```

Reference:

```json
"$ref": "#address"
```

## 24.4 `$dynamicAnchor`

Participates in dynamic reference scope and is paired with `$dynamicRef`.

Use only when recursive/extensible schema architecture truly requires dynamic scope.

---

# 25) `$ref` / `$dynamicRef` architecture

Modern reference resolution is delegated to `referencing`.

Application pattern:

```python
from jsonschema import Draft202012Validator
from referencing import Registry, Resource
```

Then:

```python
registry = Registry().with_resource(uri, resource)

validator = Draft202012Validator(
    root_schema,
    registry=registry,
)
```

## 25.1 Modern rule

Do not instantiate `RefResolver` for new code.

`RefResolver` and the validator constructor's `resolver=` path are deprecated since 4.18.

---

# 26) `referencing.Registry`

A registry represents an immutable collection of schema resources and optional retrieval behavior.

Typical methods from the companion library include:

```text
with_resource
with_resources
with_contents
contents
crawl
get_or_retrieve
combine
remove
resolver
resolver_with_root
```

The exact API is owned by `referencing`, not `jsonschema`.

## 26.1 Immutability

```python
registry2 = registry.with_resource(uri, resource)
```

does not mutate `registry`.

This makes registry composition safer for reuse and concurrency.

---

# 27) `referencing.Resource` and specification identity

A `Resource` couples:

```text
schema contents
+
JSON Schema specification/dialect
```

From self-identifying schema:

```python
from referencing import Resource

resource = Resource.from_contents(schema)
```

This inspects `$schema`.

When the schema lacks `$schema`, specify the dialect externally:

```python
from referencing.jsonschema import DRAFT202012

resource = DRAFT202012.create_resource(schema)
```

**Agent rule:** do not let schema dialect become accidental metadata.

---

# 28) In-memory schema bundles

Canonical pattern:

```python
from referencing import Registry, Resource

customer = Resource.from_contents({
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "$id": "urn:customer",
    "type": "object",
    "required": ["id"],
    "properties": {
        "id": {"type": "string"}
    },
})

registry = Registry().with_resource(
    "urn:customer",
    customer,
)

root = {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "array",
    "items": {"$ref": "urn:customer"},
}

validator = Draft202012Validator(root, registry=registry)
```

Prefer preloaded registries when schema sets are known at deployment time.

Benefits:

- no runtime network;
- deterministic resolution;
- low latency;
- easier security review;
- explicit schema inventory.

---

# 29) Filesystem retrieval

Application-owned retrieval:

```python
from pathlib import Path
import json

from referencing import Registry, Resource
from referencing.exceptions import NoSuchResource

SCHEMAS = Path("/srv/schemas").resolve()

def retrieve(uri: str):
    prefix = "https://schemas.example/"
    if not uri.startswith(prefix):
        raise NoSuchResource(ref=uri)

    relative = uri.removeprefix(prefix)
    path = (SCHEMAS / relative).resolve()

    if SCHEMAS not in path.parents and path != SCHEMAS:
        raise NoSuchResource(ref=uri)

    data = json.loads(path.read_text())
    return Resource.from_contents(data)

registry = Registry(retrieve=retrieve)
```

Security requirements:

- URI allowlist;
- path traversal prevention;
- size limit;
- schema dialect validation;
- controlled encodings;
- explicit errors.

---

# 30) HTTP/custom retrieval

Modern `referencing.Registry` does **not** automatically imply network retrieval.

If remote retrieval is required, define it explicitly.

Conceptual example:

```python
from referencing import Registry, Resource

def retrieve(uri: str):
    response = controlled_http_client.get(uri)
    response.raise_for_status()
    return Resource.from_contents(response.json())

registry = Registry(retrieve=retrieve)
```

Production retrieval must define:

```text
allowed schemes
allowed hosts
redirect policy
DNS/IP policy
timeouts
connection limits
maximum bytes
content-type policy
decompression bounds
cache policy
authentication
TLS policy
schema dialect policy
error handling
```

A schema `$id` is not evidence that it is safe to fetch.

---

# 31) YAML and non-JSON schema resources

A registry retrieval callback can deserialize YAML, TOML, database rows, or other storage formats into Python objects.

Example conceptual pipeline:

```text
URI
 -> application retrieval
 -> YAML safe loader
 -> JSON-compatible Python object
 -> Resource.from_contents(...)
```

However, only the JSON-compatible subset has well-defined JSON Schema semantics.

YAML mapping keys such as:

```yaml
1: foo
```

do not correspond to JSON object keys and may make keywords like `patternProperties` undefined from a JSON Schema perspective.

---

# 32) Migrating from `RefResolver`

Deprecated model:

```python
from jsonschema import RefResolver
```

or:

```python
Validator(schema, resolver=...)
```

Modern model:

```python
from referencing import Registry, Resource

registry = Registry(...)
Validator(schema, registry=registry)
```

Mapping:

| Old concept | Modern concept |
|---|---|
| resolver store | registry resources |
| handlers | `Registry(retrieve=...)` |
| automatic remote retrieval | explicit retrieve callback |
| mutable/scoped resolver assumptions | immutable registry + resolver internals |
| manual ref store | `with_resource(s)` |

Do not write compatibility abstractions which preserve unsafe implicit network retrieval.

---

# 33) Format validation

Critical rule:

```json
{"format": "date-time"}
```

does **not** automatically enforce date-time syntax.

Enable:

```python
validator = Draft202012Validator(
    schema,
    format_checker=Draft202012Validator.FORMAT_CHECKER,
)
```

or:

```python
from jsonschema import validate

validate(
    instance=value,
    schema=schema,
    format_checker=Draft202012Validator.FORMAT_CHECKER,
)
```

## 33.1 Unknown formats

`FormatChecker` treats unknown formats as valid.

This is by design.

If an application requires every referenced format name to be known, add a schema-governance check separately.

---

# 34) Format extras and optional dependencies

4.26.0 exposes:

```text
format
format-nongpl
```

Current `format` extra dependencies include support packages for:

```text
color
date-time
duration
hostname
idn-hostname
iri
iri-reference
json-pointer
relative-json-pointer
time
uri
uri-reference
uri-template
```

Built-in/no-extra checks also exist for formats such as:

```text
date
email
idn-email
ipv4
ipv6
regex
uuid
```

The validator's known format names include:

```text
date
date-time
duration
email
hostname
idn-email
idn-hostname
ipv4
ipv6
iri
iri-reference
json-pointer
regex
relative-json-pointer
time
uri
uri-reference
uri-template
uuid
```

## 34.1 Current extra dependency policy

Released metadata for `format` includes packages such as:

```text
fqdn
idna
isoduration
jsonpointer
rfc3339-validator
rfc3987
uri-template
webcolors
```

`format-nongpl` uses an alternate direct dependency set including:

```text
rfc3986-validator
rfc3987-syntax
```

for URI/IRI-related behavior.

4.25.0 specifically added `iri` and `iri-reference` support to `format-nongpl` via MIT-licensed `rfc3987-syntax`.

**Agent rule:** lock the selected extra dependency closure; missing optional dependencies can make a format assertion effectively succeed rather than fail.

---

# 35) Custom `FormatChecker`

Per-instance custom format:

```python
from jsonschema import FormatChecker

checker = FormatChecker()

@checker.checks("even-string-length")
def even_string_length(value):
    if not isinstance(value, str):
        return True
    return len(value) % 2 == 0
```

Use:

```python
validator = Draft202012Validator(
    {"type": "string", "format": "even-string-length"},
    format_checker=checker,
)
```

## 35.1 Raising checkers

A format checker can declare exceptions it catches:

```python
@checker.checks("custom-id", raises=ValueError)
def custom_id(value):
    ...
```

The captured exception becomes available through:

```python
ValidationError.cause
```

## 35.2 `conforms`

Programmatic check:

```python
checker.conforms(value, "uuid")
```

returns boolean.

## 35.3 `check`

```python
checker.check(value, "uuid")
```

raises `FormatError` when invalid.

## 35.4 Deprecated global registration

`FormatChecker.cls_checks` is deprecated.

Prefer instance-scoped:

```python
checker.checks(...)
```

This prevents global process state from silently changing unrelated validators.

---

# 36) `TypeChecker`

Each validator class owns a type checker:

```python
Draft202012Validator.TYPE_CHECKER
```

Public operations:

```text
is_type
redefine
redefine_many
remove
```

TypeChecker is immutable: modifications return a new checker.

Example:

```python
tc = Draft202012Validator.TYPE_CHECKER

new_tc = tc.redefine(
    "number",
    lambda checker, instance: isinstance(instance, MyNumber),
)
```

---

# 37) Custom Python type semantics

To integrate custom application types:

```python
from jsonschema import validators

base = Draft202012Validator

def is_number(checker, instance):
    return (
        base.TYPE_CHECKER.is_type(instance, "number")
        or isinstance(instance, MyNumber)
    )

type_checker = base.TYPE_CHECKER.redefine("number", is_number)

CustomValidator = validators.extend(
    base,
    type_checker=type_checker,
)
```

## 37.1 Preserve existing semantics when extending

Do not replace:

```python
number -> only MyNumber
```

unless the application genuinely wants to reject ordinary JSON numbers.

Delegate to the original TypeChecker when adding types.

## 37.2 Performance caution

Overly general Python ABC checks on every value can be slower than the library's optimized common-case checks.

Only widen types when domain objects actually require it.

---

# 38) Validation errors

Core exceptions:

```text
ValidationError
SchemaError
FormatError
UnknownType
UndefinedTypeCheck
```

`ValidationError` represents an invalid instance.

`SchemaError` represents an invalid schema under its metaschema.

## 38.1 Important `ValidationError` attributes

```text
message
validator
validator_value
schema
relative_schema_path
absolute_schema_path
schema_path
relative_path
absolute_path
json_path
path
instance
context
cause
parent
```

This structured state is significantly more useful than stringifying the exception.

---

# 39) Error paths and JSON paths

Example schema:

```json
{
  "type": "object",
  "properties": {
    "items": {
      "type": "array",
      "items": {
        "type": "integer"
      }
    }
  }
}
```

Invalid instance:

```json
{
  "items": [1, "bad"]
}
```

Relevant error state can identify:

```text
instance path: items -> 1
schema path: properties -> items -> items -> type
json_path: $.items[1]
```

Use paths for:

- API error locations;
- forms;
- IDE diagnostics;
- patch suggestions;
- test assertions.

4.24.1 fixed escaping of path segments in `ValidationError.json_path`, so avoid hand-constructing equivalent paths when the library already provides them.

---

# 40) Nested error context

For:

```text
anyOf
oneOf
allOf
if/then/else combinations
```

a high-level error can contain child errors in:

```python
error.context
```

Each child can point back through:

```python
error.parent
```

Diagnostic formatter pattern:

```python
def walk(error, indent=0):
    print(" " * indent, error.json_path, error.message)
    for child in error.context:
        walk(child, indent + 2)
```

Preserve context when machine consumers need to explain branch failures.

---

# 41) `ErrorTree`

Create:

```python
from jsonschema.exceptions import ErrorTree

tree = ErrorTree(validator.iter_errors(instance))
```

Useful features:

```python
tree.total_errors
tree[index_or_property]
```

It organizes errors by instance location and validation keyword.

Use when application code asks questions like:

- did field `name` fail?;
- which keywords failed at array index 3?;
- how many total errors exist under this subtree?

Do not use `ErrorTree` if a flat list with JSON paths already satisfies the consumer.

---

# 42) `best_match` and relevance

```python
from jsonschema.exceptions import best_match

error = best_match(validator.iter_errors(instance))
```

`best_match` uses a heuristic intended to pick a useful human-facing error.

General relevance behavior prefers:
- errors higher in the instance tree in many cases;
- stronger keywords over weak combinators such as `anyOf` / `oneOf`;
- context-aware choices.

The heuristic can change between releases.

**Agent rule:** do not make persisted machine contracts depend on the exact result of `best_match`.

For deterministic machine output, define your own sort policy over structured error fields.

---

# 43) Creating validator classes

Low-level factory:

```python
jsonschema.validators.create(
    meta_schema,
    validators=...,
    version=None,
    type_checker=...,
    format_checker=...,
    id_of=...,
    applicable_validators=...,
)
```

Use when implementing:

- a custom dialect;
- a substantially different metaschema;
- an application-specific validator vocabulary.

Keyword function signature:

```python
def keyword_validator(
    validator,
    keyword_value,
    instance,
    schema,
):
    ...
```

Keyword functions yield `ValidationError` objects for failures.

---

# 44) Extending validator classes

Common pattern:

```python
from jsonschema import Draft202012Validator, validators

Extended = validators.extend(
    Draft202012Validator,
    validators={
        "x-custom": custom_keyword,
    },
)
```

Additional parameters can replace:

```text
type_checker
format_checker
version
```

## 44.1 Important override behavior

If the mapping uses an existing keyword name:

```python
{"properties": my_properties}
```

the parent keyword implementation is silently replaced.

It is **not** automatically called.

If augmenting existing behavior:

```python
old = Draft202012Validator.VALIDATORS["properties"]

def my_properties(validator, value, instance, schema):
    # custom behavior
    for error in old(validator, value, instance, schema):
        yield error
```

---

# 45) Custom keywords

Example:

```python
from jsonschema import ValidationError, Draft202012Validator, validators

def is_even(validator, must_be_even, instance, schema):
    if not must_be_even:
        return
    if not isinstance(instance, int) or isinstance(instance, bool):
        return
    if instance % 2:
        yield ValidationError(f"{instance!r} is not even")

EvenValidator = validators.extend(
    Draft202012Validator,
    {"x-even": is_even},
)
```

Schema:

```json
{
  "type": "integer",
  "x-even": true
}
```

## 45.1 Vocabulary governance

For durable external schemas, do not casually add arbitrary unnamespaced keywords.

Prefer:

- documented extension vocabulary;
- custom metaschema;
- versioned URI identity;
- explicit supported-validator configuration.

---

# 46) Overriding existing keywords safely

If overriding a built-in keyword:

1. capture the original callable;
2. decide whether custom logic runs before or after original validation;
3. yield parent errors;
4. preserve instance/schema paths;
5. regression-test draft semantics;
6. test reference/evaluation interactions.

Particularly sensitive keywords:

```text
properties
items
prefixItems
$ref
$dynamicRef
allOf
anyOf
oneOf
unevaluatedProperties
unevaluatedItems
```

Changes can affect annotations and evaluated-location tracking beyond simple pass/fail.

---

# 47) Recursive keyword implementations and `descend`

When a custom keyword validates a subschema, use validator recursion facilities rather than manually constructing a new validator and flattening errors.

The project guidance is to use:

```python
validator.descend(...)
```

for subschema validation in keyword implementations.

Conceptual:

```python
for error in validator.descend(
    instance=subinstance,
    schema=subschema,
    path="child",
    schema_path="x-custom",
):
    yield error
```

This preserves:
- instance paths;
- schema paths;
- resolution behavior;
- nested contexts.

Do not call `iter_errors()` on a separate validator inside every custom keyword unless you intentionally want to sever those relationships.

---

# 48) Custom metaschemas and dialects

`validators.create()` accepts a metaschema and custom keyword mapping.

A serious custom dialect should define:

```text
dialect URI
metaschema
keyword vocabulary
identifier behavior
reference semantics
format policy
type policy
interoperability contract
```

For small application-only assertions, extending Draft 2020-12 is usually simpler.

## 48.1 Copy metaschemas before mutation

When extending and modifying:

```python
Extended.META_SCHEMA
```

copy it first.

Do not mutate the inherited metaschema object in place and accidentally affect the parent validator class.

---

# 49) Default insertion and mutation

JSON Schema's `default` is an annotation, not an instruction to mutate the instance.

If default insertion is required, a documented extension pattern wraps the built-in `properties` validator.

Conceptual implementation:

```python
from jsonschema import Draft202012Validator, validators

def extend_with_default(validator_class):
    validate_properties = validator_class.VALIDATORS["properties"]

    def set_defaults(validator, properties, instance, schema):
        if isinstance(instance, dict):
            for name, subschema in properties.items():
                if "default" in subschema:
                    instance.setdefault(name, subschema["default"])

        yield from validate_properties(
            validator,
            properties,
            instance,
            schema,
        )

    return validators.extend(
        validator_class,
        {"properties": set_defaults},
    )
```

## 49.1 Why mutation is risky

A schema can contain a default which is itself invalid under that schema.

Mutation also changes:
- idempotence;
- provenance;
- error timing;
- caller ownership of data.

For production pipelines, prefer:

```text
parse
-> normalize/default
-> validate normalized instance
```

as separate named stages unless schema-driven mutation is truly desired.

---

# 50) Non-JSON instances: YAML/TOML/etc.

`jsonschema` validates Python objects, so this is valid in principle:

```python
data = yaml.safe_load(text)
validator.validate(data)
```

or:

```python
data = tomllib.loads(text)
validator.validate(data)
```

if the result stays within the JSON data model.

## 50.1 Undefined edge cases

YAML can produce:
- non-string mapping keys;
- timestamps;
- sets;
- binary types;
- custom tags.

TOML can produce:
- `datetime`;
- `date`;
- `time`.

Preprocess these into explicit JSON-compatible representations before validation if portable JSON Schema semantics are required.

---

# 51) Performance and validator reuse

High-value rules:

1. call `check_schema()` once when ingesting a schema;
2. instantiate a validator once per stable policy;
3. reuse it across instances;
4. use `is_valid()` when only boolean validity is needed;
5. consume `iter_errors()` lazily if you may stop early;
6. avoid repeated `validate()` for the same schema;
7. preload referenced schemas;
8. avoid network retrieval in the validation hot path;
9. avoid expensive custom type/format functions unless necessary.

---

# 52) Reference-registry performance

Prefer:

```text
immutable prebuilt registry
+
in-memory resources
```

over:

```text
per-validation filesystem/network retrieval
```

For dynamic retrieval:

- cache resources at an application-controlled layer if appropriate;
- bound cache size;
- define invalidation;
- record schema version/hash;
- avoid recursive network storms;
- detect missing-resource failures distinctly.

`Registry` immutability makes it natural to build once and reuse.

---

# 53) Concurrency and immutability

The modern design is friendly to concurrent read-mostly use because:

- schemas are ordinary immutable-by-convention mappings;
- registries are immutable/persistent;
- type checkers are immutable;
- format checker configuration can be scoped per instance.

Nevertheless:

- do not mutate schema dicts during concurrent validation;
- do not globally register changing format handlers;
- do not mutate instance objects from custom validators unless explicitly synchronized;
- keep retrieval callbacks thread-safe;
- avoid shared mutable network clients unless the client itself supports intended concurrency.

When in doubt, construct immutable validation configuration at startup.

---

# 54) Security and resource governance

Validation of attacker-controlled instances can consume CPU/memory.

Risk areas include:

```text
very deep nesting
very large arrays/objects
large uniqueItems comparisons
expensive regexes
expensive custom formats
large combinator trees
contains / unevaluated scans
reference fan-out
recursive references
remote resource retrieval
```

The library does not provide universal resource budgets.

Applications should define:

```text
maximum input bytes
maximum decoded depth
maximum collection sizes
maximum schema bytes
maximum reference count
maximum referenced-resource bytes
maximum validation time where enforceable
maximum error count retained
```

---

# 55) Network-reference security

Never interpret:

```json
{"$ref": "http://169.254.169.254/..."}
```

as permission to fetch.

Explicit remote retrieval can create:

- SSRF;
- credential exposure;
- metadata-service access;
- internal network scanning;
- arbitrary large downloads;
- redirect bypass;
- DNS rebinding issues;
- decompression bombs.

Preferred policy:

```text
no network retrieval
```

unless the application genuinely needs it.

If enabled:

```text
allowlist schemes
allowlist hostnames
resolve and classify IPs
revalidate redirects
set short timeouts
bound response size
disable unexpected auth/cookies
verify TLS
cache deliberately
record provenance
```

---

# 56) Schema trust and custom code

A schema is data under standard keywords.

A custom validator or custom format turns parts of schema processing into application-defined code paths.

Treat schema-authored values as untrusted inputs to those functions.

Avoid:
- `eval`;
- dynamic imports;
- shell execution;
- arbitrary file paths;
- arbitrary network requests;
- unconstrained regex compilation;
- plugin loading directly from schema values.

Custom keyword handlers should be deterministic, bounded, and side-effect-minimal.

---

# 57) CLI / `check-jsonschema`

The old `jsonschema` CLI was deprecated/removed as the recommended command-line interface.

Current project documentation points users to:

```bash
pip install check-jsonschema
```

Use `check-jsonschema` for:

- CLI schema validation;
- pre-commit hooks;
- config-file validation workflows;
- SchemaStore-integrated use cases.

Do not expect:

```bash
jsonschema ...
```

to be the durable modern interface.

For application validation logic, use the Python API directly.

---

# 58) Testing strategy

Maintain three classes of tests.

## 58.1 Schema tests

```text
valid schema accepted by check_schema
invalid schema rejected
correct $schema URI
known metaschema
valid reference identifiers
```

## 58.2 Instance corpus

For every schema:

```text
positive examples
boundary examples
negative examples
nullability
empty structures
unexpected properties
nested invalid paths
combinator failures
format failures
```

## 58.3 Infrastructure tests

```text
all refs resolvable
network refs disabled unless expected
format dependencies installed
registry reproducible
custom keywords deterministic
custom types handled
error serializer stable
schema hashes pinned where required
```

---

# 59) JSON Schema Test Suite

For custom keyword/dialect work, use the official JSON Schema Test Suite concepts and upstream conformance expectations.

The `jsonschema` project itself tests against the specification suite.

If you create a new dialect or override core keywords, build compatibility tests which cover:

```text
reference behavior
type equality
number edge cases
uniqueItems equality
contains
unevaluated*
dynamic references
anchors
combinators
```

Do not rely only on happy-path application fixtures.

---

# 60) Draft migration

## 60.1 Draft 7 -> 2019-09 / 2020-12

Important areas:

```text
definitions -> $defs
dependencies -> dependentRequired / dependentSchemas
tuple items changes in 2020-12
additionalItems replaced by prefixItems/items model
recursiveRef -> dynamicRef evolution
unevaluatedProperties / unevaluatedItems
vocabulary model
format vocabulary semantics
$id / reference behavior
```

## 60.2 Keep `$schema` updated

Do not merely change the validator class while leaving:

```json
"$schema": "http://json-schema.org/draft-07/schema#"
```

inside the schema.

Migration is a schema-language change, not just an implementation setting.

---

# 61) 4.x deprecations and compatibility traps

Important modern deprecations/postures:

- `RefResolver` deprecated since 4.18;
- validator `resolver=` path deprecated with it;
- subclassing validator classes is not supported public API and warns;
- `FormatChecker.cls_checks` deprecated; use instance `.checks`;
- old global draft format-checker names are deprecated in favor of `DraftNValidator.FORMAT_CHECKER`;
- passing alternate schemas as legacy extra arguments to validator methods has been deprecated in favor of `evolve`;
- the old built-in CLI is not the recommended command-line surface;
- accessing `jsonschema.__version__` is deprecated; use `importlib.metadata.version("jsonschema")`.

Version query:

```python
from importlib.metadata import version

print(version("jsonschema"))
```

---

# 62) 4.26.0-specific behavior and recent release history

## 4.26.0

- reduces import time by delaying import of `urllib.request`.

This matters because legacy deprecated automatic remote retrieval was one of the remaining reasons to pull `urllib.request` into import-time state.

## 4.25.1

- fixes an incorrect required argument in the `Validator` protocol type annotations.

## 4.25.0

- `format-nongpl` gains `iri` and `iri-reference` support through MIT-licensed `rfc3987-syntax`.

## 4.24.1

- correctly escapes segments in `ValidationError.json_path`.

## 4.24.0

- fixes `unevaluatedProperties` handling when `additionalProperties` is present;
- drops Python 3.8 support.

## Upgrade rule

If machine consumers depend on:

```text
error paths
best-match selection
unevaluated behavior
reference traversal
format behavior
```

replay the validation corpus on upgrades.

---

# 63) Integration patterns

## 63.1 API request validation

```text
HTTP request bytes
 -> JSON decoder
 -> application normalization
 -> jsonschema validator
 -> structured ValidationError mapping
 -> domain logic
```

Do not use schema validation as the sole protection against:
- oversized HTTP bodies;
- duplicate keys;
- invalid encodings;
- authorization failures.

## 63.2 Configuration files

```text
YAML/TOML
 -> parser
 -> normalize to JSON model
 -> validator
 -> application config object
```

Use `check-jsonschema` in CI/pre-commit and the Python validator at runtime where useful.

## 63.3 Schema registry service

```text
schema upload
 -> parse
 -> enforce JSON-model rules
 -> check_schema
 -> ensure $schema
 -> validate references
 -> immutable version/hash
 -> publish
 -> prebuild Registry
```

## 63.4 Generated schemas

Schema generator should:
- emit `$schema`;
- emit stable `$id` where relevant;
- run `check_schema`;
- run positive/negative fixtures;
- round-trip via JSON serialization;
- avoid Python-only regex semantics if cross-language.

## 63.5 Multi-file local bundle

```text
load all schemas
 -> identify URIs
 -> create Resources
 -> build immutable Registry
 -> check roots
 -> instantiate validators
 -> no runtime filesystem/network reads
```

This is the recommended high-reliability architecture.

---

# 64) LLM-agent decision playbook

Before writing custom validation code, check whether a standard JSON Schema keyword already expresses the requirement.

## 64.1 Selection order

```text
standard JSON Schema assertion
    |
    v
standard annotation + application policy
    |
    v
FormatChecker
    |
    v
TypeChecker customization
    |
    v
custom keyword via validators.extend
    |
    v
custom dialect via validators.create
```

Use the lowest-customization level that correctly expresses the contract.

## 64.2 Reference resolution decision order

```text
inline schema
 -> $defs
 -> preloaded in-memory Registry
 -> controlled filesystem/database retrieve
 -> controlled HTTP retrieve only if necessary
```

## 64.3 Validation API decision order

```text
one-off -> validate()

repeated:
  validator_for(schema)
  -> check_schema once
  -> instantiate validator once
  -> reuse

need all errors -> iter_errors()
need boolean -> is_valid()
need fail-fast -> validate()
```

## 64.4 Error handling decision order

```text
machine integration -> structured ValidationError fields
UI/tree -> ErrorTree / context
single human error -> best_match
logs only -> str(error) as supplemental text, not sole representation
```

## 64.5 Agent pre-implementation checklist

Before implementing:

```text
[ ] identify schema draft
[ ] verify $schema is explicit
[ ] validate schema with check_schema
[ ] determine whether format assertions are intended
[ ] verify format dependencies
[ ] identify all $ref resources
[ ] choose Registry policy
[ ] prohibit or govern network retrieval
[ ] keep instance in JSON data model
[ ] choose error output contract
[ ] decide whether mutation/defaulting is separate
[ ] check standard keywords before custom code
[ ] add boundary/negative fixtures
```

---

# 65) Anti-pattern inventory

## Schema/dialect anti-patterns

- omitting `$schema` in durable schemas;
- validating Draft 7 syntax as 2020-12 without migration;
- assuming all implementations use identical regex semantics;
- inventing custom keywords where standard ones suffice;
- using annotations as though they were assertions.

## Runtime anti-patterns

- calling top-level `validate()` for millions of records under the same schema;
- skipping `check_schema()` before caching user-provided schemas;
- calling `is_valid()` then validating again only to collect errors;
- string-parsing `ValidationError` instead of using structured fields;
- relying on `best_match` as a stable machine contract.

## Reference anti-patterns

- new code using `RefResolver`;
- implicit automatic remote retrieval;
- treating `$id` as a download URL;
- fetching arbitrary schema URIs;
- building a new Registry per record;
- failing to pin schema resource versions.

## Format anti-patterns

- assuming `"format"` is validated automatically;
- installing format extras but forgetting to pass a FormatChecker;
- assuming an unavailable optional checker dependency will fail closed;
- globally registering format checkers in shared-process code;
- treating email format checking as proof of deliverability.

## Type anti-patterns

- validating arbitrary Python objects outside the JSON model without an explicit mapping;
- relying on Python's `bool`/`int` relationship instead of schema semantics;
- overriding `"number"` to accept only a custom class and accidentally rejecting ordinary numbers.

## Mutation anti-patterns

- expecting `default` to populate data;
- mutating instances during validation without documenting the side effect;
- coercing invalid values inside a validator and then claiming source data was valid.

## Security anti-patterns

- allowing `$ref` to access internal URLs;
- unconstrained custom format network calls;
- regex supplied by attackers without complexity governance;
- retaining unlimited validation errors from huge instances;
- assuming schema validation replaces parser/input-size controls.

---

# 66) Final deployment checklist

```text
DEPENDENCIES
[ ] jsonschema exact/compatible version pinned
[ ] Python >= 3.10
[ ] lockfile captures referencing/jsonschema-specifications/rpds-py
[ ] required format extra installed
[ ] license posture of optional dependencies reviewed

SCHEMA
[ ] $schema explicit
[ ] preferred dialect is Draft 2020-12 for new work
[ ] check_schema passes
[ ] $id policy defined
[ ] custom keywords documented/versioned
[ ] regex portability reviewed
[ ] defaults treated as annotations unless separate policy says otherwise

REFERENCES
[ ] modern referencing.Registry used
[ ] no RefResolver in new code
[ ] all known schemas preloaded where possible
[ ] retrieval policy explicit
[ ] filesystem traversal blocked
[ ] HTTP retrieval disabled or tightly governed
[ ] resource sizes bounded
[ ] schema provenance/version/hash recorded if needed

VALIDATION
[ ] validator constructed once per stable policy
[ ] format validation explicitly enabled/disabled
[ ] custom types tested
[ ] all-error versus fail-fast behavior intentional
[ ] machine diagnostics use structured error attributes
[ ] best_match used only for human-facing heuristics

INPUT MODEL
[ ] decoder guarantees understood
[ ] duplicate-key policy handled before dict creation if required
[ ] NaN/Infinity policy handled
[ ] non-string mapping keys prevented
[ ] YAML/TOML special types normalized

PERFORMANCE
[ ] schema validation not repeated in hot loop
[ ] validators reused
[ ] registries reused
[ ] references preloaded
[ ] validation corpus benchmarked
[ ] huge error collections bounded

SECURITY
[ ] input byte/depth/size limits exist
[ ] schema byte/complexity limits exist
[ ] custom validators are bounded and side-effect-minimal
[ ] custom formats do not perform uncontrolled I/O
[ ] remote refs cannot cause SSRF
[ ] regex complexity considered
[ ] logs do not leak sensitive instance data unnecessarily

TESTING
[ ] positive corpus
[ ] negative corpus
[ ] boundary corpus
[ ] reference corpus
[ ] format corpus
[ ] nested combinator/error-context corpus
[ ] unevaluated* corpus
[ ] draft migration corpus
[ ] upgrade replay gate
```

---

# Appendix A — Public top-level exports

The package's primary public top-level exports include:

```python
Draft201909Validator
Draft202012Validator
Draft3Validator
Draft4Validator
Draft6Validator
Draft7Validator
FormatChecker
SchemaError
TypeChecker
ValidationError
validate
```

Validator creation/selection utilities live under:

```python
jsonschema.validators
```

Structured exceptions/helpers live under:

```python
jsonschema.exceptions
```

Typing protocol:

```python
jsonschema.protocols.Validator
```

---

# Appendix B — Validator-class decision matrix

| Situation | Recommended API |
|---|---|
| new schema | `Draft202012Validator` |
| schema declares `$schema` | `validator_for(schema)` |
| legacy Draft 7 contract | `Draft7Validator` |
| one-off validation | `validate()` |
| repeated instances | instantiate versioned validator |
| schema ingestion | `ValidatorClass.check_schema()` |
| boolean answer | `.is_valid()` |
| all errors | `.iter_errors()` |
| fail-fast | `.validate()` |
| same policies/new schema | `.evolve(schema=...)` |
| custom reference sources | `registry=Registry(...)` |
| custom format | `FormatChecker().checks(...)` |
| custom Python type | `TYPE_CHECKER.redefine(...)` + `validators.extend()` |
| custom keyword | `validators.extend()` |
| custom dialect/metaschema | `validators.create()` |

---

# Appendix C — Draft 2020-12 keyword family cheat sheet

## Core / identity / reference

```text
$schema
$id
$ref
$anchor
$dynamicRef
$dynamicAnchor
$vocabulary
$comment
$defs
```

## Applicator

```text
prefixItems
items
contains
additionalProperties
properties
patternProperties
dependentSchemas
propertyNames
if
then
else
allOf
anyOf
oneOf
not
```

## Unevaluated

```text
unevaluatedItems
unevaluatedProperties
```

## Validation

```text
type
const
enum
multipleOf
maximum
exclusiveMaximum
minimum
exclusiveMinimum
maxLength
minLength
pattern
maxItems
minItems
uniqueItems
maxContains
minContains
maxProperties
minProperties
required
dependentRequired
```

## Annotation / metadata

```text
title
description
default
deprecated
readOnly
writeOnly
examples
```

## Format/content-related vocabulary

```text
format
contentEncoding
contentMediaType
contentSchema
```

Not every keyword above is implemented as a direct callable in `Draft202012Validator.VALIDATORS`; core/annotation semantics can be handled through other layers or may remain informational.

---

# Appendix D — Error serialization recommendation

A durable application-facing error DTO should prefer structured fields:

```python
def serialize_error(error):
    return {
        "message": error.message,
        "keyword": error.validator,
        "keyword_value": error.validator_value,
        "instance_path": list(error.absolute_path),
        "schema_path": list(error.absolute_schema_path),
        "json_path": error.json_path,
        "children": [serialize_error(e) for e in error.context],
        "cause": None if error.cause is None else str(error.cause),
    }
```

Do not serialize:
- arbitrary entire instance fragments if they may contain secrets;
- arbitrary schema objects if very large;
- raw exception `repr` as the only contract.

Version your error DTO independently from `jsonschema`'s human-readable message text.

---

# Appendix E — Recommended production bootstrap

```python
from jsonschema import Draft202012Validator
from referencing import Registry, Resource

def build_validator(root_schema, resources):
    Draft202012Validator.check_schema(root_schema)

    pairs = []
    for uri, schema in resources.items():
        Draft202012Validator.check_schema(schema)
        pairs.append((uri, Resource.from_contents(schema)))

    registry = Registry().with_resources(pairs)

    return Draft202012Validator(
        root_schema,
        registry=registry,
        format_checker=Draft202012Validator.FORMAT_CHECKER,
    )
```

For a high-assurance system, add:

```text
schema hash verification
URI uniqueness
dialect allowlist
resource-count limit
resource-size limit
no-network policy
format dependency self-test
fixture replay
```

before publishing the validator to application traffic.

---

# Appendix F — Upgrade review checklist

For every `jsonschema` upgrade:

```text
[ ] read jsonschema changelog
[ ] read referencing changelog if resolved version changes
[ ] read jsonschema-specifications change if resolved version changes
[ ] compare format-extra dependency closure
[ ] replay schema metaschema tests
[ ] replay valid/invalid instance corpus
[ ] compare machine error DTOs
[ ] inspect changed json_path values
[ ] replay anyOf/oneOf nested errors
[ ] replay unevaluatedProperties/Items
[ ] replay $ref/$dynamicRef
[ ] replay custom keywords
[ ] replay custom formats
[ ] replay custom types
[ ] confirm no deprecated APIs remain
[ ] benchmark representative workloads
```

A changed error message alone need not be a semantic break. A changed validation decision, reference target, error path, or persisted machine contract deserves explicit review.
