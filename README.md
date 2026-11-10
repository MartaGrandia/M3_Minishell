# 🐚 Minishell

> A POSIX-compliant UNIX shell interpreter built from scratch in C, modeling core Bash behaviors, process lifecycle management, pipeline redirection, and signal propagation.

## 📖 Overview

`Minishell` is a low-level systems programming project focused on recreating the core mechanics of an interactive UNIX command line interpreter. The application implements an end-to-end execution lifecycle: reading raw input via `readline`, lexical analysis (tokenization), syntax parsing into executable command structures, environment variable resolution, file descriptor redirection, and parallel process execution connected through system pipelines.

## ⚙️ Technical Constraints & Architecture

- **Language:** C (compiled with `gcc -Wall -Wextra -Werror`).
- **Allowed POSIX Syscalls:** Low-level process & file system primitives (`fork`, `execve`, `pipe`, `dup2`, `wait`, `waitpid`, `open`, `close`, `read`, `write`, `getcwd`, `chdir`, `stat`, `unlink`, `signal`, `sigaction`, `tcsetattr`, `tcgetattr`).
- **Memory Safety:** 100% leak-free execution across infinite REPL loops, process forks, and syntax error paths (audited using Valgrind).
- **Standards:** Compliant with 42's **Norminette** coding standard.

---

## 🏛️ Pipeline & Modular Architecture

Minishell processes user input through four decoupled engineering layers:

### 1. Lexical Analysis & Parsing
- **Quote Handling:** 
  - Single quotes (`'`) preserve the literal value of all characters within.
  - Double quotes (`"`) preserve literal text while allowing variable expansions (`$VAR`).
- **Syntax Validation:** Detects unclosed quotes, invalid redirection placements, or orphan pipe tokens before entering the execution phase.

### 2. Environment Management
- Maintains a dynamic, heap-allocated copy of environment variables (`envp`).
- Evaluates `$VAR` expansion dynamically and handles `$?` to return the exit status of the most recently executed pipeline.

### 3. Execution & Process Handling
- **External Binaries:** Searches executable binary paths listed inside the environment's `$PATH` variable or executes local binaries via relative/absolute paths.
- **Pipelines (`|`):** Connects `stdout` of process $N$ to `stdin` of process $N+1$ using system pipes (`pipe()`) and file descriptor duplication (`dup2()`).
- **Redirections:**
  - `<` Redirects input stream from a file.
  - `>` Redirects output stream to a file (truncate).
  - `>>` Redirects output stream to a file (append).
  - `<<` (*Here-document*) Reads input until a line containing the specified delimiter is seen.

### 4. Interactive Signal Propagation
- **Interactive Mode:**
  - `Ctrl-C` (`SIGINT`): Prints a new prompt on a fresh line.
  - `Ctrl-\` (`SIGQUIT`): Ignored.
  - `Ctrl-D` (`EOF`): Exits the shell REPL cleanly.
- **Child Execution Mode:** Signals are restored to default behaviors so child processes respond correctly to system interrupts.

---

## 🛠️ Built-in Commands

Reimplemented from scratch to run within the main shell process (avoiding unnecessary subshell forks where environment mutation is required):

| Built-in | Functionality | Options / Scope |
| :--- | :--- | :--- |
| `echo` | Prints arguments to standard output. | Supports `-n` flag (suppress trailing newline). |
| `cd` | Changes current working directory. | Supports relative and absolute paths, updates `PWD` and `OLDPWD`. |
| `pwd` | Prints absolute path of current working directory. | Uses POSIX `getcwd()`. |
| `export` | Sets environment variables. | Exports variables to child processes or lists current exports. |
| `unset` | Removes environment variables. | Removes variable definitions from dynamic internal store. |
| `env` | Prints current environment variables. | Outputs formatted `KEY=VALUE` list. |
| `exit` | Terminates shell process. | Accepts optional integer exit status codes (`0` - `255`). |

---

## 🛠️ Quickstart & Execution

### Prerequisites

Minishell requires the GNU `readline` development library installed on your system:

```bash
# macOS (via Homebrew)
brew install readline

# Ubuntu / Debian
sudo apt-get install libreadline-dev
