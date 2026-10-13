# turbo-repro-boundaries-jsx-in-js

Reproduction for a Turborepo bug: `turbo boundaries` and `turbo query` file tracing fail to parse JSX in `.js` files.

Created with `npx create-turbo@canary -e with-shell-commands -m pnpm`. The only additions are in `packages/pkg-a/src`:

- `greeting.js` has JSX in a `.js` file.
- `greeting-control.jsx` is an identical copy with a `.jsx` extension.
- `helper.js` is imported by both.

## Reproduce

```sh
pnpm install
pnpm turbo boundaries
pnpm turbo query 'query { file(path: "packages/pkg-a/src/greeting.js") { dependencies { files { items { path } } errors { items { message } } } } }'
pnpm turbo query 'query { file(path: "packages/pkg-a/src/greeting-control.jsx") { dependencies { files { items { path } } errors { items { message } } } } }'
```

## Expected

Both files parse. `turbo boundaries` reports no issues, and both queries list `packages/pkg-a/src/helper.js` as a dependency.

## Actual (turbo 2.11.8-canary.2)

```
$ pnpm turbo boundaries
Checking packages...
  x failed to parse file /repo/packages/pkg-a/src/greeting.js: parser panicked

Checked 3 files in 5 packages, 1 issue found
```

Querying `greeting.js` returns no dependencies and the error `failed to parse file: Unexpected JSX expression`. Querying `greeting-control.jsx` returns `packages/pkg-a/src/helper.js`.

`turbo boundaries` passes on `2.8.13-canary.10` and fails starting with `2.8.13-canary.11`.
