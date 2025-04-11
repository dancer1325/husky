# How To

## Adding a NEW Hook

* == creat a file | ".husky/"

## Startup files

* | BEFORE running hooks, execute local commands / placed |
  - `$XDG_CONFIG_HOME/husky/init.sh`
  - `~/.config/husky/init.sh`
  - `~/.huskyrc` (deprecated)
  - | Windows, `C:\Users\yourusername\.config\husky\init.sh`

## Skipping Git Hooks

### / 1! Command

* ways
  * `-n/--no-verify` option
    ```sh
    git commit -m "..." -n # Skips Git hooks
    ```
    * ALLOWED |
      * MOST Git commands  
  * HUSKY=0 git someSpecificGitCommand
   ```shell
   HUSKY=0 git ... # Temporarily disables all Git hooks
   git ... # Hooks will run again
   ```

### / MULTIPLE commands

* use case
  * | extended period (e.g., during rebase/merge)
   ```shell
   export HUSKY=0 # Disables ALL Git hooks
   git ...
   git ...
   unset HUSKY # Re-enables hooks
   ```

### / GUI

* |GUI client
  ```sh
  # ~/.config/husky/init.sh
  export HUSKY=0 # Husky won't install and won't run hooks on your machine
  ```

## CI server & Docker

* ways
  * use `HUSKY=0
    * _Example:_ | GitHub Actions
      ```yml
      # https://docs.github.com/en/actions/learn-github-actions/variables
      env:
        HUSKY: 0
      ```
    * recommended
      * Reason: 🧠 avoid installing Git Hooks 🧠
  * | "package.json", adjust `prepare` script 
    ```json
    // package.json
    "prepare": "husky || true"
    ```
    * use case
      * if installing ONLY `dependencies` (!= `devDependencies`) -> `"prepare": "husky"` script failS
  * create `.husky/install.mjs` & use | "package.json"'s `prepare` script 
    * use case
      * if PREVIOUS solution, you get a `command not found` error message
    ```js
    // Skip Husky install in production and CI
    if (process.env.NODE_ENV === 'production' || process.env.CI === 'true') {
      process.exit(0)
    }
    const husky = (await import('husky')).default
    console.log(husky())
    ```

    ```json
    "prepare": "node .husky/install.mjs"
    ```

## Testing Hooks -- WITHOUT -- Committing

* | hook script, add `exit 1`
  ```shell
  # .husky/pre-commit
  
  # Your WIP script
  # ...
  
  exit 1
  ```

```shell
git commit -m "testing pre-commit code"
# A commit will not be created
```

## Project Not in Git Root Directory

* TODO:
Husky doesn't install in parent directories (`../`) for security reasons. However, you can change the directory in the `prepare` script.

Consider this project structure:

```
.
├── .git/
├── backend/  # No package.json
└── frontend/ # Package.json with husky
```

Set your prepare script like this:

```json
"prepare": "cd .. && husky frontend/.husky"
```

In your hook script, change the directory back to the relevant subdirectory:

```shell
# frontend/.husky/pre-commit
cd frontend
npm test
```

## Non-shell hooks

In order to run scripts that require the use of a scripting language, use the following pattern for each applicable hook:

(Example using hook `pre-commit` and NodeJS)
1. Create an entrypoint for the hook:
    ```shell
    .husky/pre-commit
    ```
2. In the file add the following
    ```shell
    node .husky/pre-commit.js
    ```
3. in `.husky/pre-commit.js`
   ```javascript
   // Your NodeJS code
   // ...
   ```

## Bash

Hook scripts need to be POSIX compliant to ensure best compatibility as not everyone has `bash` (e.g. Windows users).

That being said, if your team doesn't use Windows, you can use Bash this way:

```shell
# .husky/pre-commit

bash << EOF
# Put your bash script inside
# ...
EOF
```

## Node Version Managers and GUIs

If you're using Git hooks in GUIs with Node installed via a version manager (like `nvm`, `n`, `fnm`, `asdf`, `volta`, etc...), you might face a `command not found` error due to `PATH` environment variable issues.

### Understanding `PATH` and Version Managers

`PATH` is an environment variable containing a list of directories. Your shell searches these directories for commands. If it doesn't find a command, you get a `command not found` message.

Run `echo $PATH` in a shell to view its contents.

Version managers work by:
1. Adding initialization code to your shell startup file (`.zshrc`, `.bashrc`, etc.), which runs each time you open a terminal.
2. Downloading Node versions to a directory in your home folder.

For example, if you have two Node versions:

```shell
~/version-manager/Node-X/node
~/version-manager/Node-Y/node
```

Opening a terminal initializes the version manager, which picks a version (say `Node-Y`) and prepends its path to `PATH`:

```shell
echo $PATH
# Output
~/version-manager/Node-Y/:...
```

Now, node refers to `Node-Y`. Switching to `Node-X` changes `PATH` accordingly:

```shell
echo $PATH
# Output
~/version-manager/Node-X/:...
```

The issue arises because GUIs, launched outside a terminal, don't initialize the version manager, leaving `PATH` without the Node install path. Thus, Git hooks from GUIs often fail.

### Solution

Husky sources `~/.config/husky/init.sh` before each Git hook. Copy your version manager initialization code here to ensure it runs in GUIs.

Example with `nvm`:

```shell
# ~/.config/husky/init.sh
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

Alternatively, if your shell startup file is fast and lightweight, source it directly:

```shell
# ~/.config/husky/init.sh
. ~/.zshrc
```

## Manual setup

Git needs to be configured and husky needs to setup files in `.husky/`.

Run the `husky` command once in your repo. Ideally, include it in the `prepare` script in `package.json` for automatic execution after each install (recommended).

::: code-group

```json [npm]
{
  "scripts": {
    "prepare": "husky" // [!code hl]
  }
}
```

```json [pnpm]
{
  "scripts": {
    "prepare": "husky" // [!code hl]
  }
}
```

```json [yarn]
{
  "scripts": {
    // Yarn doesn't support prepare script
    "postinstall": "husky",
    // Include this if publishing to npmjs.com
    "prepack": "pinst --disable",
    "postpack": "pinst --enable"
  }
}
```

```json [bun]
{
  "scripts": {
    "prepare": "husky" // [!code hl]
  }
}
```

:::

Run `prepare` once:

::: code-group

```sh [npm]
npm run prepare
```

```sh [pnpm]
pnpm run prepare
```

```sh [yarn]
# Yarn doesn't support `prepare`
yarn run postinstall
```

```sh [bun]
bun run prepare
```

:::

Create a `pre-commit` file in the `.husky/` directory:

::: code-group

```shell [npm]
# .husky/pre-commit
npm test
```

```shell [pnpm]
# .husky/pre-commit
pnpm test
```

```shell [yarn]
# .husky/pre-commit
yarn test
```

```sh [bun]
# .husky/pre-commit
bun test
```

:::
