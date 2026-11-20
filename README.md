# Linux & Security Fundamentals

My learning log as I build hands-on cybersecurity skills, starting with the Linux command line.

**Started:** November 2026
**Current focus:** [OverTheWire Bandit](https://overthewire.org/wargames/bandit/), a wargame that teaches Linux and security basics through puzzles on a remote server.

> Following OverTheWire's rules, this repo contains **no passwords and no level-by-level solutions**. It documents the skills and concepts, organised by topic.

## Progress

| Wargame | Status |
|---|---|
| Bandit | Levels 0–19 complete, working on level 20 |

## Projects

- [Home network security assessment](home-network-assessment.md): mapping and assessing the devices on my own Wi-Fi with `arp`, `nmap` and `curl`

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

### 8. Encoding is not encryption: base64

```bash
base64 -d file      # decode base64 back to normal text
```

- base64 turns any data into plain letters, numbers, `+`, `/`, often ending in `=`.
- It hides nothing: anyone can decode it instantly. Encoding changes how data *looks*; encryption needs a secret key.

### 9. Letter substitution with `tr` (ROT13)

```bash
cat file | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

- `tr` swaps characters one-for-one: the Nth character of the first list becomes the Nth of the second.
- ROT13 shifts every letter 13 places. Doing it twice gets you back where you started.

### 10. Hexdumps and layered compression

Some files are wrapped like Russian dolls. The method is a loop:

1. `file name` to identify the real type (it reads the contents, not the name)
2. `mv` to give it the matching extension
3. Open it with the matching tool
4. Repeat until `file` says ASCII text

| `file` says | Rename to | Open with |
|---|---|---|
| gzip compressed | `.gz` | `gzip -d name.gz` |
| bzip2 compressed | `.bz2` | `bzip2 -d name.bz2` |
| POSIX tar archive | `.tar` | `tar -xvf name.tar` |
| ASCII text | | `cat name` |

Other tools from this level:

```bash
mktemp -d            # make a private, randomly named work folder in /tmp
cp ~/file .          # copy a file to here
xxd -r hex.txt > out # turn a hexdump back into the original bytes
mv old new           # rename (or move) a file
```

- gzip and bzip2 **compress** a single file and need the right extension. tar **bundles** files into one box without compressing.
- A file's extension is just a label. `file` tells the truth.

### 11. SSH keys instead of passwords

An SSH key comes in two halves:

| Half | Lives on | Analogy |
|---|---|---|
| Public key | The server | The lock |
| Private key | My computer | The key, which must stay secret |

```bash
scp -P 2220 user@host:remote-file .          # secure copy from a server to here (capital -P for port)
chmod 600 private-key                         # only I can read/write it
ssh -i private-key user@host -p 2220          # log in with the key, no password
```

**Permissions with `chmod`:** each digit is owner / group / others, adding read (4), write (2), execute (1).
`600` = owner read+write, nobody else anything → `-rw-------`.
SSH refuses to use a private key that others can read ("UNPROTECTED PRIVATE KEY FILE").

### 12. Good habits on shared systems

- `/tmp` is a shared scratch space that anyone can write to and that gets wiped. Use `mktemp -d` for an unguessable folder.
- Clean up when done (`rm -rf` your own work folder, carefully). Leftover files leak secrets and are where investigators and attackers both look.

### 13. Ports and talking to network services with `nc`

A server is like a building with 65,535 numbered doors (**ports**). Different services wait behind different doors: SSH on 22, HTTPS on 443, and so on.

```bash
nc localhost 30000    # open a raw connection to port 30000 on this machine
```

- `nc` (netcat) opens a plain connection so you can type straight to a service and read its reply.
- `localhost` means "this same machine."
- `Ctrl+C` closes the connection.

### 14. Encrypted connections with SSL/TLS

`nc` sends plain text, like a postcard anyone on the network can read. TLS puts the conversation in a locked envelope.

```bash
openssl s_client -connect localhost:30001   # open a TLS-encrypted connection, then type your message
```

How TLS works:
1. **ID check:** the server presents a certificate to prove who it is.
2. **Handshake:** both sides agree on a shared secret key.
3. **Encrypted conversation:** everything after is scrambled with that key.

Reading the output:
- `self-signed certificate` means the server signed its own ID card. That's fine for practice, but on the real web it's a warning sign; trusted sites use certificates signed by a recognised authority.
- `Protocol: TLSv1.3` and the cipher line show how strong the encryption is.
- Never send passwords over unencrypted connections. That's why websites use `https://`.

How I'd find this tool on my own: read the goal, notice the constraint ("must use SSL/TLS"), try the tool I know (`nc`) and see why it isn't enough, then use hints, `man`, search, or AI to find the right one.

### 15. Port scanning and service detection with `nmap`

```bash
nmap localhost -p 31000-32000                 # which ports in this range are open?
nmap -sV localhost -p 31046,31518,31790       # what service is behind each open port?
```

- A scan only says which doors are **open**. `-sV` (service/version detection) talks to each one to identify it, so it's much slower.
- Read the clues: `echo` services just repeat what you send; a TLS service that replies "Wrong! Please enter the correct password" is the one that wants something.
- Pressing the space bar during a long nmap scan prints progress.

**Ethics:** only scan systems you have written permission to scan (wargames, labs, your own devices, bug bounty scopes). Unauthorised scanning can get you reported, blocked, or worse, and it damages trust in a field built on trust.

### 16. Debugging a copied SSH key

```bash
pbpaste > key.file        # macOS: save the clipboard to a file
head -1 key.file          # check the first line
tail -1 key.file          # check the last line
echo >> key.file          # append a newline (>> adds, > overwrites)
ssh-keygen -y -f key.file # test that a private key is valid
```

- `Load key ...: invalid format` often means the file is missing its final newline. zsh hints at this by printing `%` right after the last line.
- The clipboard holds only the last thing copied.

### 17. Comparing files with `diff`

```bash
diff old.txt new.txt
```

```
42c42
< line from the first file
---
> line from the second file
```

- `42c42` means line 42 **c**hanged (`a` = added, `d` = deleted).
- `<` lines come from the first file, `>` lines from the second.
- Real-world use: spotting what changed in config files, code, or logs, for example after a suspected intrusion.

### 18. Running a single command over SSH

```bash
ssh user@host -p 2220 cat readme   # run one command remotely, print the result, disconnect
```

- A normal login starts an interactive shell, which runs startup files like `.bashrc`. If that file is booby-trapped (e.g. logs you out instantly), you can still run a command directly without opening the interactive shell.
- Real-world use: quick remote checks and automation scripts (`ssh server uptime`), and a reminder that startup files are a common place for attackers to hide persistence.

### 19. setuid: a program that runs as its owner

```
-rwsr-x---  1  bandit20  bandit19  ...  bandit20-do
```

- Normally a program runs with **my** permissions. A **setuid** program (`s` instead of `x` in the owner's permissions) runs with its **owner's** permissions, whoever starts it.
- Analogy: a hotel housekeeper with a master card does one task for a guest whose own card can't open that room.

```bash
whoami                                       # bandit19
./bandit20-do whoami                         # bandit20
./bandit20-do cat /etc/bandit_pass/bandit20  # read a file only bandit20 can read
```

- Real-world use: `passwd` is setuid root, so ordinary users can change their own password in a root-only file.
- Security angle: a careless setuid program that runs *any* command is a gift to attackers. If it's owned by root, it means full **privilege escalation**. Checking for setuid files is a standard step when auditing a Linux system.
- Discovery habit: read the goal, look closely (`ls -l`), notice what's unusual (an `s`), let the program explain itself (run it with no arguments or `--help`), and look up anything new.

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
2. Check what something is before you open it. A file's name is just a label.
3. Small tools chained with pipes can do powerful things.
4. Leave shared systems clean, and keep secrets (like private keys) locked down.

## Memorising commands vs understanding concepts in the age of AI

A question I asked myself while learning: do I need to memorise all these commands when AI tools exist?

My conclusion: **mostly understand the concepts, but keep a small core automatic.**

**Know by heart** (it comes naturally from use): `ls`, `cd`, `pwd`, `cat`, `cp`, `mv`, `rm`, `grep`, `find`, `|`, `ssh`, `chmod`, `file`, plus Tab, Up arrow and Ctrl+C.
These get typed constantly, and AI isn't always available: offline servers, secure workplaces, live incidents, interviews and certifications.

**Understand, but look up the details:** `tar` flags, `tr` ranges, `xxd`, exact `find` options. Knowing the tool exists and what it does matters; the exact syntax can come from `man`, `--help` or AI.

**What AI can't replace:**

| Skill | Why it still matters |
|---|---|
| Knowing what's possible | You can't ask for a tool you don't know exists |
| Breaking a problem into steps | e.g. identify → choose tool → apply → repeat |
| Reading output and errors | Understanding "Permission denied" or "unknown suffix" |
| Judging AI's answers | AI can be confidently wrong, and `rm -rf` has no undo |
| Security instincts | Don't trust file names, protect keys, clean up after yourself |

My test: could I explain *why* each command works, even if I'd look up the exact spelling? If yes, I've learned what matters.

## Next up

- Bandit level 13 onward: SSH keys and network services.
