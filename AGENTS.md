# Notion Rehype

A TypeScript Unified parser plugin that converts Notion API blocks into a HAST tree for use in the rehype ecosystem.

## Stack

- TypeScript in strict mode with ESM and ES2020 output.
- Unified, VFile, HAST, and the Notion SDK.
- Prettier for formatting.
- pnpm 10.

## Key Commands

- `pnpm build`: clean and compile the package with TypeScript.
- `pnpm build:watch`: compile in watch mode.
- `pnpm clean`: remove generated build output.
- `pnpm exec prettier --check "src/**/*.ts"`: check source formatting.
- `pnpm typecheck`: type-check without emitting build output.
- Match CI locally in this order: formatting, typecheck, then build.
- There is currently no automated test suite. Do not describe a change as tested unless a real test command was run.

## File Structure

```text
src/
|-- index.ts              Unified plugin entry point and input parsing
|-- notion-to-hast.ts     Conversion context, task queue, and root assembly
|-- types.ts              Options and Notion-derived block types
|-- utils.ts              Shared HAST, metadata, list, and footnote helpers
`-- handlers/             Block-specific and shared rendering handlers
docs/
`-- CONTRIBUTING.md       Contributor and maintainer release process
```

## Code Style

- Follow `.prettierrc.cjs`: 88-column width, semicolons, single quotes, two spaces, and ES5 trailing commas.
- Use `import type` for type-only imports.
- Include `.js` extensions on local TypeScript imports because the package is published as ESM.
- Prefer straightforward control flow and descriptive intermediate variables over dense expressions.
- Keep modules focused and avoid new abstractions or dependencies unless they reduce concrete complexity.
- Preserve Notion API field names such as `rich_text` and `plain_text` when representing SDK data.

## Naming Conventions

- Files and directories use kebab-case.
- Variables, functions, and options use camelCase.
- Types and interfaces use PascalCase.
- Handler modules usually define a local `handler`, export it as default, and are imported as `handleX`.
- Public names should describe the Notion or HAST concept they represent; avoid generic names without domain context.

## Documentation

- Add JSDoc or TSDoc to new public APIs when the TypeScript signature does not fully explain the contract.
- Explain why a non-obvious conversion constraint exists; do not restate self-explanatory code.
- Keep TODO and FIXME comments actionable and specific.

## Implementation Notes

- `src/index.ts` installs a custom Unified parser. It accepts blocks through `VFile.data` or parses them from the input string.
- `src/notion-to-hast.ts` uses a LIFO task stack. `Context.addTasks` pushes arrays in reverse so handlers preserve source order when tasks are popped.
- Handlers return HAST for individual Notion block types. Keep block dispatch centralized in `src/handlers/index.ts`.
- `footnoteReference` is shallow-cloned before conversion because footnote processing mutates it.
- `dist/` is generated publication output. Change `src/` and rebuild instead of editing generated files.

## Quality Checklist

- Formatting, typecheck, and build pass in CI order.
- New local imports include `.js` extensions and type-only imports use `import type`.
- Names follow repository conventions and public API changes are documented.
- Conversion changes preserve block order, nested children, list grouping, and footnote behavior.
- Unsupported Notion block types continue to fail safely rather than corrupting the HAST tree.
- If behavior needs regression coverage, add a real test runner and focused tests rather than a no-op test script.

## Releases

- Contributors do not change package versions or create release tags.
- For an explicit maintainer release task, follow `docs/CONTRIBUTING.md`; it is the canonical versioning, publishing, verification, and recovery procedure.
