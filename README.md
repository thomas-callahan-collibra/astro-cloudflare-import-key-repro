# `@astrojs/cloudflare` dep scan hoists `import:` object keys

Minimal reproduction: in dev, `@astrojs/cloudflare`'s frontmatter dependency-scan plugin treats an object key named `import` at the start of a line, for example `import.meta.glob`'s `import: "default"` option, as an import statement. It hoists the key out of the options object, Rolldown fails to parse the result, and Vite skips dependency pre-bundling.

## Steps

```sh
pnpm install
rm -rf node_modules/.vite   # the scan only runs when Vite's deps cache is empty or stale
pnpm dev
```

Then open http://localhost:4321/ (the page itself renders fine).

## Actual

```
(!) Failed to run dependency scan. Skipping dependency pre-bundling. Error:   Failed to scan for dependencies from entries:
  src/pages/index.astro

  Build failed with 1 error:

[PARSE_ERROR] Expected `from` but found `:`
   ╭─[ src/pages/index.astro:1:7 ]
   │
 1 │ import: "default",
   │       ┬
   │       ╰── `from` expected
───╯
```

## Expected

No error. The `import` key stays inside the `import.meta.glob` options object, and dependency pre-bundling runs.

## Where

`packages/integrations/cloudflare/src/rolldown-plugin-astro-frontmatter.ts`. `IMPORT_STMT_RE` (`/^\s*import\b…/m`) matches `  import: "default",`, and `isImportExpression()` only excludes `import.` and `import(`. The trigger is [`src/pages/index.astro`](src/pages/index.astro).

Tested with `astro` 7.4.4 and `@astrojs/cloudflare` 14.4.3.
