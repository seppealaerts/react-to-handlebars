# react-to-handlebars

Convert React components to Handlebars templates. Write email (or other) templates in React/JSX with `Handlebars.If`, `Handlebars.Each`, and `Handlebars.Val`; run the CLI to emit `.handlebars` files you can use with any Handlebars-based stack.

### Why?

The email provider I use for a certain project (Mandrill) only accepts Handlebars templates, but I wanted to author emails in React (e.g. with [React Email](https://react.email)) for a better developer experience: components, reusability, TypeScript, and a live preview. This package lets you build templates in React and compile them to Handlebars so they work with Mandrill (or any Handlebars-based handler).

## Install

```bash
npm install react-to-handlebars
```

Peer dependency: **React** (^18 or ^19).

## CLI

Build all `.tsx` / `.jsx` files in a directory to `.handlebars` files (output next to sources):

```bash
npx react-to-handlebars <dir>
```

Example:

```bash
npx react-to-handlebars source/emails
```

- `dir` is resolved relative to the current working directory.
- Each component file can export optional `PreviewProps` for the build (placeholder data when rendering to static HTML).

### Preview (Handlebars → HTML)

Compile `.handlebars` templates with JSON data to static HTML for previews. For each template, the script looks for a matching JSON file (same path with `.json` suffix, or same base name with `.json`):

```bash
npx react-to-handlebars-preview <sourceDir> [outputDir]
```

- `sourceDir` — directory containing `.handlebars` files (required).
- `outputDir` — where to write `.html` files (default: `previews`).

Example: `npx react-to-handlebars-preview source/emails previews` reads `source/emails/welcome.handlebars` and `source/emails/welcome.handlebars.json` (or `welcome.json`) and writes `previews/welcome.html`.

## Library usage

Use the `Handlebars` helpers in your React components so the build can turn them into Handlebars blocks and variables.

### Setup

Wrap your template tree with `Handlebars.Provider` and pass the data you use in conditions and variables (for React preview only; the build uses `PreviewProps` or markers):

```tsx
import { Handlebars } from "react-to-handlebars";

const previewData = { name: "Jane", items: [{ title: "Hello" }] };

<Handlebars.Provider data={previewData}>
  <MyEmail />
</Handlebars.Provider>;
```

### API

| Component         | Handlebars equivalent     | Purpose                          |
| ----------------- | ------------------------- | -------------------------------- |
| `Handlebars.If`   | `{{#if condition}}`       | Conditional block                |
| `Handlebars.Else` | `{{else}}`                | Else branch (child of `If`)      |
| `Handlebars.Each` | `{{#each array}}`         | Loop over an array               |
| `Handlebars.Val`  | `{{path}}` / `{{this.x}}` | Output a variable (path or name) |

- **Conditions** in `Handlebars.If` are strings: `"order.id"`, `"items.length"`, `"a && b"`, `"x \|\| y"`, `"!flag"`. They are evaluated against the `Provider` data in React and turned into Handlebars `{{#if ...}}` at build time.
- **Each**: `array` is the Handlebars path (e.g. `"items"`). Optional `itemVar` gives the block param name (e.g. `"item"` → `{{#each items as \|item\|}}`). Inside `Each`, `Handlebars.Val` with `name="title"` becomes `{{this.title}}`.
- **Val**: `name` is the Handlebars path (or property name inside an each). Optional `value` is used in React preview when not building.

### Example component

```tsx
import { Handlebars } from "react-to-handlebars";

export const PreviewProps = {
  userName: "Alex",
  items: [{ label: "First" }, { label: "Second" }],
};

export default function MyEmail() {
  return (
    <div>
      <p>
        Hello, <Handlebars.Val name="userName" value="Alex" />.
      </p>
      <Handlebars.If condition="items.length">
        <ul>
          <Handlebars.Each array="items" itemVar="item">
            <li>
              <Handlebars.Val name="label" />
            </li>
          </Handlebars.Each>
        </ul>
        <Handlebars.Else>
          <p>No items.</p>
        </Handlebars.Else>
      </Handlebars.If>
    </div>
  );
}
```

After running `react-to-handlebars` on the directory containing this file, you get a `.handlebars` file with classic Handlebars `{{#if}}`, `{{#each}}`, `{{variable}}`, and `{{else}}`.

## License

MIT
