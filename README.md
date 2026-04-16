# ts-json

Serialize a TypeScript source file into a clean JSON representation of its AST.

## What it does

Parses a `.ts` file with the TypeScript compiler API and walks the resulting AST, emitting a structural JSON tree with:

- `SyntaxKind` numeric enums resolved to their string names (`"FunctionDeclaration"` instead of `257`)
- Circular and internal fields stripped (`parent`, `symbol`, `flags`, `pos`, `end`, and friends)
- Only structural properties kept — roughly 130 in an allowlist

The result is stable, readable, and safe to `JSON.stringify` without running into cycles.

## What it isn't

Not a transformer, not a compiler, not a type checker. It reads, it serializes, it exits. If you want to rewrite code, look at [ts-morph](https://github.com/dsherret/ts-morph) or a proper [TransformerFactory](https://github.com/madou/typescript-transformer-handbook).

## Install / Run

Requires [Deno](https://deno.com).

```sh
deno run --allow-read https://jsr.io/@dustinboston/ts-json ./example.ts > example.json
```

## Example

```ts
// example.ts
export function greet(name: string): string {
  return `Hello, ${name}`;
}
```

```json
{
  "kind": "SourceFile",
  "statements": [
    {
      "kind": "FunctionDeclaration",
      "modifiers": [{ "kind": "ExportKeyword" }],
      "name": { "kind": "Identifier", "text": "greet" },
      "parameters": [
        {
          "kind": "Parameter",
          "name": { "kind": "Identifier", "text": "name" },
          "type": { "kind": "StringKeyword" }
        }
      ],
      "type": { "kind": "StringKeyword" },
      "body": { "kind": "Block", "statements": [ ... ] }
    }
  ]
}
```

*(Shape is illustrative — actual output includes the full set of allowlisted properties.)*

## Why the allowlist

The TypeScript AST carries a lot of baggage you don't want in JSON: back-pointers to `parent` (circular), resolver state (`symbol`, `resolvedSymbol`), source positions (`pos`, `end`), and bitmask fields (`flags`, `transformFlags`, `modifierFlagsCache`) that are meaningless without the enums that decode them.

Rather than strip these at emit time, `ts-json` keeps an explicit allowlist of the ~130 properties that make up the structural shape of the tree. Adding a property is a deliberate choice; everything else is ignored.

See [`props.ts`](./props.ts) for the full list and [`mod.ts`](./mod.ts) for the 40-line visitor.

## Use cases

- Diffing ASTs across commits without caring about source positions
- Feeding a structural view of a file to an LLM that doesn't need raw syntax
- Inspecting what the compiler actually sees — sometimes easier than reading through the `typescript.d.ts` type definitions

## License

[GPL-3.0](./LICENSE)
