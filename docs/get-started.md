# Get started

## How to install?

```shell [npm]
npm install --save-dev husky
```

```shell [pnpm]
pnpm add --save-dev husky
```

```shell [yarn]
yarn add --dev husky
# Add pinst ONLY if your package is not private
yarn add --dev pinst
```

```shell [bun]
bun add --dev husky
```

## `husky init` (recommended)

* enables you
  * 💡simplifies setting up husky | project 💡-- via --
    * creating a `pre-commit` script | `.husky/`
    * updates the `prepare` script | `package.json`

```shell [npm]
npx husky init
```

```shell [pnpm]
pnpm exec husky init
```

```shell [yarn]
# Due to specific caveats and differences with other package managers,
# refer to the How To section.
```

```shell [bun]
bunx husky init
```

## Try it

Congratulations! You've successfully set up your first Git hook with just one command 🎉. 
Let's test it:

```shell
git commit -m "Keep calm and commit"
# test script will run every time you commit
```

## A few words...

### Scripting

* your hooks' use cases
  * run `npm run` or `npx` commands
  * script them -- via -- POSIX shell / CUSTOM workflows
    * _Example:_ lint your staged files | EACH commit
        ```shell
        # .husky/pre-commit
        prettier $(git diff --cached --name-only --diff-filter=ACMR | sed 's| |\\ |g') --write --ignore-unknown
        git update-index --again
        ```
    * see [lint-staged](https://github.com/lint-staged/lint-staged)

### Disabling hooks

* Husky does NOT force Git hooks 
* if you want to disable
  * globally -- via -- `HUSKY=0` 
* see [manual set-up](how-to)
