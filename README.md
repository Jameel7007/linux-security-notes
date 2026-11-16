# Linux & Security Fundamentals

My learning log as I build hands-on cybersecurity skills, starting with the Linux command line.

**Started:** November 2026
**Current focus:** [OverTheWire Bandit](https://overthewire.org/wargames/bandit/), a wargame that teaches Linux and security basics through puzzles on a remote server.

> Following OverTheWire's rules, this repo contains **no passwords and no level-by-level solutions**. It documents the skills and concepts, organised by topic.

## Progress

| Wargame | Status |
|---|---|
| Bandit | Levels 0–9 complete, working on level 10 |

## Skills learned so far

### 1. Connecting to a remote server with SSH

```bash
ssh user@host -p 2220
```

- `-p` sets the port. SSH normally uses port 22; this server uses a custom one.
- On first connection, SSH asks you to trust the server's **fingerprint** and saves it to `~/.ssh/known_hosts`. If that fingerprint ever changes, SSH warns you, which protects against impersonation.
- Each user account has its own password and its own permissions.

### 2. Moving around and looking at files

| Command | What it does |
|---|---|
| `pwd` | Show which folder I'm in |
| `ls` | List files |
| `ls -la` | List all files, including hidden ones, with details |
| `cd folder` | Move into a folder |
| `cat file` | Print a file's contents |
| `file ./*` | Identify the type of every file without opening it |

**Hidden files:** any name starting with `.` is hidden from plain `ls`. `.` means "this folder" and `..` means "the folder above."

### 3. Reading `ls -la` output

```
-rw-r----- 1 bandit5 bandit4   33 Oct 25 13:37 -file00
```

| Part | Meaning |
|---|---|
| `-rw-r-----` | Permissions (`d` at the start = folder) |
| `1` | Link count |
| `bandit5` | Owner (user) |
| `bandit4` | Group |
| `33` | Size in bytes |
| `Oct 25 13:37` | Last modified |
| `-file00` | Name |

### 4. Awkward filenames

The shell splits commands at spaces and treats a leading `-` as an option.

| Problem | Fix |
|---|---|
| Name is or starts with `-` | Use a path: `cat ./-filename` |
| Name contains spaces | Quote it: `cat "my file"` or escape: `cat my\ file` |
| Both | Combine: `cat ./"--name with spaces--"` |

**Tab completion** fills in names for you and handles the escaping.

### 5. Searching with `find`

```bash
find . -size 1033c                                     # exact size in bytes
find / -user someuser -group somegroup -size 33c 2>/dev/null
```

- `.` starts the search here; `/` starts at the top of the whole system.
- Rules can be combined: `-user`, `-group`, `-size`, `-type`, `-name`.
- `2>/dev/null` throws away error messages (like "Permission denied"), leaving only real results.

### 6. Searching inside files and combining commands

| Command | What it does |
|---|---|
| `grep word file` | Show only lines containing a word |
| `sort file` | Sort lines so duplicates sit together |
| `uniq -u` | Keep only lines that appear once |
| `strings file` | Pull readable text out of a binary file |

**Pipes (`|`)** send one command's output into the next:

```bash
sort file | uniq -u          # find the one unique line
strings file | grep ===      # find readable text after a marker
```

### 7. File types and binary data

- `file` guesses a file's type from its bytes. It's a hint, not proof.
- **ASCII text** is clean, readable text. Labels like "data" or "extended-ASCII ... escape sequences" usually mean binary junk.
- If binary output scrambles the terminal, `reset` fixes it.

## Terminal productivity (macOS)

| Keys | Action |
|---|---|
| `↑` | Bring back the last command |
| `Tab` | Autocomplete a file or folder name |
| `Option + click` | Move the cursor to that spot |
| `Ctrl + A` / `Ctrl + E` | Jump to start / end of line |
| `Option + ← / →` | Jump word by word |
| `Ctrl + W` / `Ctrl + U` | Delete a word / everything before the cursor |
| `Ctrl + C` | Cancel the current command |

## Key takeaways

1. The shell has rules (spaces split words, `-` means option, `.` means hidden). Most "weird" problems come from those rules.
2. Check what something is before you open it.
3. Small tools chained with pipes can do powerful things.
4. AI can recall syntax, but understanding the system is what lets me judge whether a command or its output is right.

## Next up

- Bandit level 9 onward: encoding (`base64`), character translation (`tr`), compressed archives (`tar`, `gzip`, `bzip2`), and hex dumps (`xxd`).
