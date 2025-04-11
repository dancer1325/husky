# Troubleshoot

## Command not found

* see [How To](how-to)

## Hooks NOT running

1. Verify the [file name is correct](https://git-scm.com/docs/githooks)  
   1. _Example:_ `precommit` or `pre-commit.sh` are invalid names
2. Run `git config core.hooksPath` / 's output == `.husky/_` OR your CUSTOM hooks directory
3. Git version `2.9+`

## `.git/hooks/` Not Working After Uninstall

* TODO: If hooks in `.git/hooks/` don't work post-uninstalling `husky`, execute `git config --unset core.hooksPath`.

## Yarn | Windows

* TODO:
Git hooks might fail with Yarn on Windows using Git Bash (`stdin is not a tty`). For Windows users, implement this workaround:

1. Create `.husky/common.sh`:

```shell
command_exists () {
  command -v "$1" >/dev/null 2>&1
}

# Workaround for Windows 10, Git Bash, and Yarn
if command_exists winpty && test -t 1; then
  exec < /dev/tty
fi
```

2. Source it where Yarn commands are run:

```shell
# .husky/pre-commit
. .husky/common.sh

yarn ...
```
