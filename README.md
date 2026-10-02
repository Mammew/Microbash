# Micro-bash

A minimal Unix shell written in C. It reads a command line, parses it, and runs the commands in child processes, with support for pipes, I/O redirection, environment variables and a built-in `cd`.

The skeleton of the program (v2.2) was written by Giovanni Lagorio for the *Sistemi di Elaborazione e Trasmissione dell'Informazione* lab, Computer Science degree, University of Genoa (a.y. 2022/2023). The sections marked `TO BE DONE` in the source were implemented as the lab assignment.

## Features

- **Interactive prompt** showing the current working directory (e.g. `/home/user $ `), built with GNU Readline
- **Pipelines** with `|`: `cmd1 | cmd2 | cmd3`
- **Input redirection** with `<file` and **output redirection** with `>file`
- **Environment variables**: an argument starting with `$` is replaced by the value of that variable (or by an empty string if it is not set)
- **Built-in `cd`** to change the shell's working directory
- **Child status reporting**: if a command exits with a non-zero status, or is killed by a signal, the shell prints its PID and the exit status or signal

## Requirements

- A C compiler (e.g. `gcc`)
- The GNU Readline development library (e.g. `libreadline-dev` on Debian/Ubuntu)

## Build

```bash
gcc -o microbash microbash.c -lreadline
```

To print the parsed command line before each execution, build with the `DEBUG` flag:

```bash
gcc -DDEBUG -o microbash microbash.c -lreadline
```

## Run

```bash
./microbash
```

Exit the shell with `Ctrl+D` (end of input).

## Usage examples

```bash
/home/user $ ls -l | grep .c | wc -l
/home/user $ sort <names.txt >sorted.txt
/home/user $ cat <input.txt | grep error >errors.txt
/home/user $ echo $HOME
/home/user $ cd /tmp
/tmp $
```

## Syntax rules and restrictions

- Arguments are separated by spaces or tabs. There is no support for quoting or escaping.
- A redirection operator must be attached to its path, with no space in between: `<input.txt` and `>output.txt` are valid, `< input.txt` is not.
- Each command can have at most one input and one output redirection.
- Only the **first** command of a pipeline can have input redirection, and only the **last** one can have output redirection.
- Output files are created if missing and truncated if they exist (created with `rwx` permissions for the owner only).
- `cd` must be the only command on the line, takes exactly one argument and cannot have redirections.
- `$VAR` expansion applies only when `$` is the first character of an argument, and the whole argument is replaced.
- Not supported: background execution (`&`), command chaining (`;`, `&&`, `||`), globbing, and quoting.

## How it works

1. `readline` reads a line, using the current directory as the prompt.
2. `parse_line` splits the line on `|` into commands, and `parse_cmd` splits each command into arguments and redirections, expanding `$VAR` arguments.
3. `check_redirections` and `check_cd` validate the line and print an error if it breaks the rules above.
4. `execute_line` opens the redirection files, creates the pipes (with `FD_CLOEXEC`), and starts each command with `run_child`, which does `fork`, redirects the file descriptors with `dup2`, and calls `execvp`.
5. `wait_for_children` waits for all the children and reports abnormal terminations.
6. `free_line` releases all the memory used by the parsed line.

## License

Copyright (C) 2020-2022 Giovanni Lagorio.
Distributed under the GNU General Public License, version 2 or (at your option) any later version.
