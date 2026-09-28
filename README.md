# Bash Scripting — Complete Comprehensive Guide

A detailed reference and use-case guide for Bash: core concepts, syntax, commands, and real-world scripting patterns. Written for beginners through intermediate users who want a single reference document.

---

## Table of Contents

1. [What is Bash](#1-what-is-bash)
2. [Shebang, Permissions & Running Scripts](#2-shebang-permissions--running-scripts)
3. [Variables & Data Types](#3-variables--data-types)
4. [Quoting Rules](#4-quoting-rules)
5. [Special Variables & Parameters](#5-special-variables--parameters)
6. [Operators & Arithmetic](#6-operators--arithmetic)
7. [String Manipulation](#7-string-manipulation)
8. [Arrays](#8-arrays)
9. [Conditionals (if / test / [[ ]])](#9-conditionals-if--test---)
10. [Loops](#10-loops)
11. [Case Statements](#11-case-statements)
12. [Functions](#12-functions)
13. [Input/Output & Redirection](#13-inputoutput--redirection)
14. [Pipes & Command Substitution](#14-pipes--command-substitution)
15. [Working with Files & Directories](#15-working-with-files--directories)
16. [Text Processing (grep, sed, awk, cut, sort)](#16-text-processing-grep-sed-awk-cut-sort)
17. [Process & Job Control](#17-process--job-control)
18. [Exit Codes & Error Handling](#18-exit-codes--error-handling)
19. [Reading User Input & Arguments](#19-reading-user-input--arguments)
20. [Working with JSON, CSV & Dates](#20-working-with-json-csv--dates)
21. [Cron & Scheduling](#21-cron--scheduling)
22. [Common Use Cases (End-to-End Scripts)](#22-common-use-cases-end-to-end-scripts)
23. [Best Practices](#23-best-practices)
24. [Quick Reference Cheat Sheet](#24-quick-reference-cheat-sheet)

---

## 1. What is Bash

Bash (**B**ourne **A**gain **SH**ell) is the default command-line shell and scripting language on most Linux distributions and macOS (through Catalina; newer macOS defaults to `zsh` but Bash is still available). It's used both interactively (typing commands) and as a scripting language (`.sh` files) for automation, system administration, and glue code between other programs.

Check your Bash version:
```bash
bash --version
echo $BASH_VERSION
```

---

## 2. Shebang, Permissions & Running Scripts

Every script should start with a **shebang** line telling the OS which interpreter to use:
```bash
#!/bin/bash
```
or, more portably (uses whichever `bash` is first in `$PATH`):
```bash
#!/usr/bin/env bash
```

### Making a script executable
```bash
chmod +x myscript.sh
./myscript.sh
```

### Running without execute permission
```bash
bash myscript.sh
sh myscript.sh          # runs with /bin/sh (may be dash, not full bash features)
source myscript.sh      # runs in the CURRENT shell (variables/functions persist after)
. myscript.sh           # same as source
```

The difference between `./script.sh` and `source script.sh` matters: the former runs in a **subshell** (its `cd`, exported variables, etc. don't affect your terminal after it exits); the latter runs in your **current shell**.

---

## 3. Variables & Data Types

Bash has no strict typing — everything is fundamentally a string, though it supports integer arithmetic contexts.

```bash
name="Elmor"
age=35
readonly PI=3.14159      # constant, cannot be reassigned
unset name                # delete a variable
```

**No spaces around `=`.** `name = "Elmor"` is a syntax error (Bash parses it as running a command called `name`).

### Using variables
```bash
echo $name
echo "$name"
echo "${name}"           # braces avoid ambiguity, e.g. "${name}_suffix"
```

### Export (make available to child processes)
```bash
export PATH="$PATH:/opt/mytools"
```

### Default values / parameter expansion
```bash
echo "${name:-Guest}"        # use "Guest" if name is unset or empty (doesn't change $name)
echo "${name:=Guest}"        # same, but ALSO assigns Guest to $name if unset
echo "${name:?Error: not set}"  # print error and exit if unset
echo "${name:+Set}"          # print "Set" if name IS set (non-empty), else nothing
```

### Command substitution into a variable
```bash
current_dir=$(pwd)
files_count=$(ls | wc -l)
today=`date +%F`           # backtick syntax — legacy, prefer $() instead
```

---

## 4. Quoting Rules

| Quote | Behavior |
|---|---|
| `"double quotes"` | Variables and `$(...)` are expanded; most special characters are literal |
| `'single quotes'` | Nothing is expanded — fully literal |
| `` `backticks` `` | Command substitution (legacy form of `$(...)`) |
| `\` (backslash) | Escapes the next single character |

```bash
name="World"
echo "Hello, $name"     # Hello, World
echo 'Hello, $name'     # Hello, $name
echo "Path: \$HOME"     # Path: $HOME  (escaped $)
```

**Always quote variable expansions** (`"$var"`) when using them in commands, especially file paths — unquoted variables undergo word-splitting and glob expansion, which breaks on filenames with spaces.

---

## 5. Special Variables & Parameters

| Variable | Meaning |
|---|---|
| `$0` | Script name |
| `$1`, `$2`, ... | Positional arguments |
| `$#` | Number of arguments |
| `$@` | All arguments, each as a separate word (use `"$@"`) |
| `$*` | All arguments as a single word |
| `$?` | Exit status of the last command |
| `$$` | PID of the current shell/script |
| `$!` | PID of the last background job |
| `$-` | Current shell option flags |
| `$IFS` | Internal Field Separator (default: space/tab/newline) |
| `$RANDOM` | Random integer (0–32767) |
| `$SECONDS` | Seconds since the shell started |

`"$@"` vs `"$*"` matters a lot: with args `a "b c" d`, `"$@"` gives you three separate words (`a`, `b c`, `d`); `"$*"` gives one combined string (`a b c d`). Use `"$@"` when forwarding arguments to another command.

---

## 6. Operators & Arithmetic

### Arithmetic expansion
```bash
result=$((5 + 3))
result=$((10 / 3))       # integer division = 3
result=$((10 % 3))       # modulus = 1
((count++))
((count += 5))
```

### `let` and `expr` (older styles, `$(( ))` is preferred)
```bash
let "x = 5 + 3"
x=$(expr 5 + 3)
```

### Comparison (numeric — inside `[ ]` or `[[ ]]`)
| Operator | Meaning |
|---|---|
| `-eq` | Equal |
| `-ne` | Not equal |
| `-gt` / `-ge` | Greater than / or equal |
| `-lt` / `-le` | Less than / or equal |

### Comparison (string)
| Operator | Meaning |
|---|---|
| `=` or `==` | Equal (use `==` inside `[[ ]]`) |
| `!=` | Not equal |
| `<` / `>` | Lexicographic less/greater (must be inside `[[ ]]` or escaped in `[ ]`) |
| `-z` | String is empty |
| `-n` | String is non-empty |

### Logical
```bash
[ "$a" -eq 1 ] && [ "$b" -eq 2 ]     # AND
[ "$a" -eq 1 ] || [ "$b" -eq 2 ]     # OR
[[ $a -eq 1 && $b -eq 2 ]]           # AND, inside [[ ]]
```

---

## 7. String Manipulation

```bash
str="Hello, World!"

echo ${#str}                # length -> 13
echo ${str:7}                # substring from index 7 -> "World!"
echo ${str:7:5}               # substring, 5 chars -> "World"
echo ${str,,}                  # lowercase -> "hello, world!"
echo ${str^^}                  # uppercase -> "HELLO, WORLD!"
echo ${str/World/Bash}         # replace first match -> "Hello, Bash!"
echo ${str//o/0}               # replace ALL matches -> "Hell0, W0rld!"
echo ${str#Hello, }            # remove shortest match from the START -> "World!"
echo ${str%, World!}           # remove shortest match from the END -> "Hello"

# Concatenation
greeting="Hello"
name="Elmor"
full="$greeting, $name!"

# Splitting a string into an array
IFS=',' read -ra parts <<< "a,b,c"
echo "${parts[1]}"             # b
```

---

## 8. Arrays

### Indexed arrays
```bash
fruits=("Apple" "Banana" "Cherry")
echo "${fruits[0]}"             # Apple
echo "${fruits[@]}"             # all elements
echo "${#fruits[@]}"            # length -> 3
fruits+=("Mango")               # append
unset fruits[1]                 # remove Banana

for f in "${fruits[@]}"; do
    echo "$f"
done
```

### Associative arrays (Bash 4+, like a dictionary)
```bash
declare -A person
person[name]="Elmor"
person[role]="Engineer"

echo "${person[name]}"
for key in "${!person[@]}"; do
    echo "$key = ${person[$key]}"
done
```

---

## 9. Conditionals (if / test / `[[ ]]`)

```bash
if [ "$age" -ge 18 ]; then
    echo "Adult"
elif [ "$age" -ge 13 ]; then
    echo "Teen"
else
    echo "Child"
fi
```

### `[[ ]]` (Bash extended test — preferred over `[ ]` in Bash scripts)
```bash
if [[ -f "$file" && -r "$file" ]]; then
    echo "File exists and is readable"
fi

if [[ "$str" == a* ]]; then       # pattern matching works inside [[ ]]
    echo "Starts with a"
fi

if [[ "$str" =~ ^[0-9]+$ ]]; then  # regex matching
    echo "Is a number"
fi
```

### File test operators
| Test | Meaning |
|---|---|
| `-e` | Exists |
| `-f` | Is a regular file |
| `-d` | Is a directory |
| `-r` / `-w` / `-x` | Readable / Writable / Executable |
| `-s` | Exists and is non-empty |
| `-L` | Is a symbolic link |
| `file1 -nt file2` | file1 newer than file2 |
| `file1 -ot file2` | file1 older than file2 |

---

## 10. Loops

### For loop
```bash
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

for i in {1..10}; do echo "$i"; done
for i in {1..10..2}; do echo "$i"; done     # step by 2

for f in *.txt; do
    echo "Processing $f"
done

for ((i=0; i<5; i++)); do
    echo "$i"
done
```

### While loop
```bash
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Reading a file line by line (the correct, robust way)
while IFS= read -r line; do
    echo "Line: $line"
done < "input.txt"
```

### Until loop
```bash
count=1
until [ $count -gt 5 ]; do
    echo "$count"
    ((count++))
done
```

### Break / Continue
```bash
for i in {1..10}; do
    [ $i -eq 3 ] && continue
    [ $i -eq 7 ] && break
    echo "$i"
done
```

---

## 11. Case Statements

```bash
read -p "Enter a fruit: " fruit
case "$fruit" in
    apple|Apple)
        echo "It's an apple"
        ;;
    banana)
        echo "It's a banana"
        ;;
    *.pdf)
        echo "Looks like a filename pattern, not a fruit"
        ;;
    *)
        echo "Unknown fruit"
        ;;
esac
```

---

## 12. Functions

```bash
greet() {
    local name="$1"           # 'local' scopes the variable to the function
    echo "Hello, $name!"
}

greet "Elmor"

# Return values: bash functions "return" only exit codes (0-255).
# For actual data, echo it and capture with $()
add() {
    local sum=$(( $1 + $2 ))
    echo "$sum"
}
result=$(add 5 7)
echo "Result: $result"

# Return an exit status explicitly
is_even() {
    (( $1 % 2 == 0 ))
    return $?
}
if is_even 4; then echo "Even"; fi
```

---

## 13. Input/Output & Redirection

```bash
command > file.txt          # redirect stdout, overwrite
command >> file.txt         # redirect stdout, append
command 2> error.log        # redirect stderr only
command > out.log 2>&1      # redirect both stdout and stderr to the same file
command &> all.log          # shorthand for the above (bash-specific)
command < input.txt         # redirect stdin from a file
command 2>/dev/null         # discard stderr entirely
command > /dev/null 2>&1    # discard everything (silent mode)
```

### Here-documents & here-strings
```bash
cat <<EOF
Multiple
lines of text
with $variable expansion
EOF

cat <<'EOF'
No expansion happens here, $variable stays literal
EOF

grep "pattern" <<< "$variable"   # here-string
```

---

## 14. Pipes & Command Substitution

```bash
ls -la | grep ".txt"
cat file.txt | sort | uniq -c | sort -nr
ps aux | grep nginx | awk '{print $2}'

# Command substitution
file_count=$(ls | wc -l)
now=$(date +"%Y-%m-%d %H:%M:%S")

# Process substitution (treats command output as a file)
diff <(ls dir1) <(ls dir2)
```

---

## 15. Working with Files & Directories

```bash
pwd                              # print working directory
cd /path/to/dir                  # change directory
cd -                             # go to previous directory
ls -la                           # list all, long format
mkdir -p a/b/c                   # create nested directories
rmdir emptydir                   # remove empty directory
rm file.txt                      # delete a file
rm -rf directory/                # force-delete directory recursively (careful!)
cp file.txt backup.txt
cp -r sourcedir/ destdir/
mv old.txt new.txt               # rename or move
touch newfile.txt                # create empty file / update timestamp
find . -name "*.log"             # find files by name
find . -type f -mtime +30        # files older than 30 days
find . -type f -size +10M        # files larger than 10MB
du -sh *                         # size of each item in current dir
df -h                            # disk free space, human-readable
ln -s /path/to/target linkname   # create a symbolic link
stat file.txt                    # detailed file metadata
chmod 755 script.sh              # set permissions
chown user:group file.txt        # change ownership
```

---

## 16. Text Processing (grep, sed, awk, cut, sort)

### grep — search text
```bash
grep "error" logfile.txt
grep -i "error" logfile.txt          # case-insensitive
grep -r "TODO" ./src                 # recursive
grep -v "debug" logfile.txt          # invert match (lines NOT containing)
grep -c "error" logfile.txt          # count matches
grep -n "error" logfile.txt          # show line numbers
grep -E "error|warning" logfile.txt  # extended regex (multiple patterns)
```

### sed — stream editor (find & replace, line editing)
```bash
sed 's/foo/bar/' file.txt            # replace first occurrence per line
sed 's/foo/bar/g' file.txt           # replace all occurrences
sed -i 's/foo/bar/g' file.txt        # edit the file in place
sed -n '5,10p' file.txt              # print only lines 5-10
sed '/pattern/d' file.txt            # delete lines matching pattern
```

### awk — field/column processing
```bash
awk '{print $1}' file.txt                 # print first column
awk -F',' '{print $2}' file.csv           # use comma as field separator
awk '{sum += $3} END {print sum}' file.txt   # sum a column
awk '$3 > 100 {print $0}' file.txt        # filter rows where col 3 > 100
awk 'NR==1{print} NR>1{print $0}' file.txt   # print header + rest
```

### cut — extract columns
```bash
cut -d',' -f1,3 file.csv        # fields 1 and 3, comma-delimited
cut -c1-10 file.txt              # characters 1-10 of each line
```

### sort / uniq
```bash
sort file.txt
sort -n file.txt                 # numeric sort
sort -r file.txt                 # reverse
sort -k2 file.txt                # sort by 2nd column
sort file.txt | uniq             # unique lines (must be sorted first)
sort file.txt | uniq -c          # count occurrences
```

### Others worth knowing
```bash
wc -l file.txt         # line count
wc -w file.txt          # word count
tr 'a-z' 'A-Z' < file.txt   # translate characters (lowercase to uppercase)
tee output.txt          # write to file AND stdout, from a pipe
head -n 20 file.txt
tail -n 20 file.txt
tail -f logfile.txt      # follow a growing log file live
```

---

## 17. Process & Job Control

```bash
ps aux                       # list all running processes
ps aux | grep myapp
top                           # interactive process viewer
kill 1234                     # send SIGTERM to PID 1234
kill -9 1234                  # force kill (SIGKILL)
killall processname           # kill by name
pkill -f "pattern"            # kill matching a command-line pattern

command &                     # run in background
jobs                          # list background jobs of current shell
fg %1                         # bring job 1 to foreground
bg %1                         # resume job 1 in background
wait                          # wait for all background jobs to finish
nohup command &                # run immune to hangup (survives terminal close)
disown                        # detach a job from the shell so it isn't killed on exit
```

---

## 18. Exit Codes & Error Handling

By convention, `0` = success, non-zero = failure.

```bash
exit 0        # success
exit 1        # generic failure
echo $?       # check exit status of the last command
```

### Safe script settings (put near the top of every script)
```bash
set -e          # exit immediately if any command fails
set -u          # error on use of an undefined variable
set -o pipefail # a pipeline fails if ANY command in it fails, not just the last
set -x          # print each command before running it (debug mode)

# Common combined form:
set -euo pipefail
```

### Trap — run cleanup code on exit/error/signal
```bash
cleanup() {
    echo "Cleaning up temp files..."
    rm -f /tmp/myscript.*
}
trap cleanup EXIT
trap 'echo "Interrupted"; exit 1' INT TERM
```

### Manual error checking
```bash
if ! command -v git &> /dev/null; then
    echo "git is not installed" >&2
    exit 1
fi

mkdir /some/dir || { echo "Failed to create directory" >&2; exit 1; }
```

---

## 19. Reading User Input & Arguments

### Prompting for input
```bash
read -p "Enter your name: " name
echo "Hello, $name"

read -s -p "Enter password: " password   # -s hides input
echo
```

### Script arguments
```bash
#!/bin/bash
echo "Script name: $0"
echo "First arg: $1"
echo "All args: $@"
echo "Arg count: $#"

if [ $# -lt 1 ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi
```

### Parsing flags with `getopts`
```bash
while getopts "n:a:vh" opt; do
    case $opt in
        n) name="$OPTARG" ;;
        a) age="$OPTARG" ;;
        v) verbose=true ;;
        h) echo "Usage: $0 -n name -a age [-v]"; exit 0 ;;
        \?) echo "Invalid option: -$OPTARG"; exit 1 ;;
    esac
done

# Run as: ./script.sh -n Elmor -a 35 -v
```

---

## 20. Working with JSON, CSV & Dates

### JSON with `jq` (install: `apt install jq` / `brew install jq`)
```bash
echo '{"name":"Elmor","role":"Engineer"}' | jq '.name'
curl -s https://api.example.com/user | jq '.data.email'
jq '.items[] | select(.price > 100)' data.json
jq -r '.name' data.json          # -r strips quotes from string output
```

### CSV (plain Bash, simple cases)
```bash
while IFS=',' read -r name role years; do
    echo "$name works as $role ($years years)"
done < employees.csv
```

### Dates
```bash
date                              # current date/time
date +"%Y-%m-%d"                  # 2026-09-28
date +"%Y-%m-%d %H:%M:%S"
date -d "yesterday" +"%Y-%m-%d"   # GNU date: relative dates
date -d "+7 days" +"%Y-%m-%d"
date -d "@1234567890"             # convert Unix timestamp to date
date +%s                          # current Unix timestamp
```
(On macOS, `date` is BSD-based and uses `-v` instead of `-d` for relative dates, e.g. `date -v-1d`.)

---

## 21. Cron & Scheduling

Cron runs scripts on a schedule. Edit your crontab with `crontab -e`.

```
# Format: minute hour day month weekday command
0 8 * * *        /home/user/scripts/daily_report.sh      # every day at 8:00 AM
*/15 * * * *     /home/user/scripts/healthcheck.sh        # every 15 minutes
0 0 * * 0        /home/user/scripts/weekly_backup.sh      # every Sunday at midnight
0 9 1 * *        /home/user/scripts/monthly_invoice.sh    # 9 AM on the 1st of every month
```

Useful commands:
```bash
crontab -l          # list current user's cron jobs
crontab -e           # edit
crontab -r           # remove all cron jobs (careful!)
```
Always use **full absolute paths** in cron scripts — cron runs with a minimal environment/PATH, unlike your interactive shell.

---

## 22. Common Use Cases (End-to-End Scripts)

### 22.1 Backup a directory with a timestamp
```bash
#!/bin/bash
set -euo pipefail
SOURCE="/home/user/data"
DEST="/backups/data_$(date +%Y%m%d_%H%M%S).tar.gz"
tar -czf "$DEST" -C "$SOURCE" .
echo "Backup created at $DEST"
```

### 22.2 Find and delete files older than N days
```bash
#!/bin/bash
DAYS=30
DIR="/var/log/myapp"
find "$DIR" -type f -name "*.log" -mtime +"$DAYS" -exec rm -f {} \;
echo "Deleted logs older than $DAYS days"
```

### 22.3 Monitor disk usage and send an alert
```bash
#!/bin/bash
THRESHOLD=90
df -h | awk 'NR>1 {print $5, $6}' | while read -r usage mount; do
    pct=${usage%\%}
    if [ "$pct" -ge "$THRESHOLD" ]; then
        echo "WARNING: $mount is at ${pct}% usage"
    fi
done
```

### 22.4 Bulk-rename files
```bash
#!/bin/bash
for f in *.jpeg; do
    [ -e "$f" ] || continue
    mv -- "$f" "${f%.jpeg}.jpg"
done
```

### 22.5 Simple website/service health check with retries
```bash
#!/bin/bash
URL="https://example.com/health"
MAX_RETRIES=3
for i in $(seq 1 $MAX_RETRIES); do
    if curl -sf "$URL" -o /dev/null; then
        echo "Service is up"
        exit 0
    fi
    echo "Attempt $i failed, retrying..."
    sleep 5
done
echo "Service is DOWN after $MAX_RETRIES attempts" >&2
exit 1
```

### 22.6 Parse a log file for errors and summarize
```bash
#!/bin/bash
LOG="/var/log/app.log"
echo "Error summary for $LOG:"
grep -oE "ERROR: [A-Za-z_]+" "$LOG" | sort | uniq -c | sort -nr
```

### 22.7 Batch-convert CSV rows into user-creation commands
```bash
#!/bin/bash
while IFS=',' read -r username fullname; do
    [ "$username" = "username" ] && continue   # skip header row
    useradd -c "$fullname" "$username"
    echo "Created user: $username"
done < newusers.csv
```

### 22.8 Deploy script: pull latest code, install deps, restart service
```bash
#!/bin/bash
set -euo pipefail
APP_DIR="/opt/myapp"
cd "$APP_DIR"
git pull origin main
npm install --production
sudo systemctl restart myapp
echo "Deployment complete at $(date)"
```

### 22.9 Watch a directory for new files and process them
```bash
#!/bin/bash
WATCH_DIR="/data/incoming"
PROCESSED_DIR="/data/processed"
for file in "$WATCH_DIR"/*; do
    [ -f "$file" ] || continue
    echo "Processing: $file"
    # ... do work on $file ...
    mv "$file" "$PROCESSED_DIR/"
done
```
(For real-time watching instead of polling, use `inotifywait` from the `inotify-tools` package: `inotifywait -m "$WATCH_DIR" -e create | while read -r dir action file; do ...; done`.)

### 22.10 Reusable logging function pattern
```bash
#!/bin/bash
LOG_FILE="/var/log/myscript.log"

log() {
    local level="$1"; shift
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] [$level] $*" | tee -a "$LOG_FILE"
}

log INFO "Script started"
if ! some_command; then
    log ERROR "some_command failed"
    exit 1
fi
log INFO "Script completed successfully"
```

---

## 23. Best Practices

- **Always start scripts with `#!/usr/bin/env bash`** and `set -euo pipefail` for safer, fail-fast behavior.
- **Quote your variables** (`"$var"`, `"$@"`) to avoid word-splitting and glob-expansion bugs, especially with filenames containing spaces.
- **Use `[[ ]]` instead of `[ ]`** for conditionals in Bash scripts — it's safer (no word-splitting on unquoted variables) and supports pattern/regex matching.
- **Use `local`** for variables inside functions to avoid polluting the global scope.
- **Prefer `$(...)` over backticks** for command substitution — it's more readable and nests cleanly.
- **Check command success explicitly** (`if ! command; then ...`) rather than assuming things worked.
- **Avoid parsing `ls` output** — use globs (`for f in *.txt`) or `find` instead; filenames can contain characters that break naive parsing.
- **Use `trap` for cleanup** (temp files, locks) so scripts clean up even on error or interruption.
- **Lint scripts with `shellcheck`** — it catches an enormous number of quoting/logic bugs before they bite you.
- **Use full absolute paths in cron jobs and other non-interactive contexts**, since `$PATH` and environment there is minimal.
- **Comment non-obvious logic** and add a usage/help block for any script that takes arguments.
- **Test destructive commands (`rm -rf`, bulk renames) with `echo` first** or a dry-run flag before running for real.

---

## 24. Quick Reference Cheat Sheet

| Task | Command |
|---|---|
| List files | `ls -la` |
| Change directory | `cd path` |
| Print working dir | `pwd` |
| Read file | `cat file` |
| Search text | `grep pattern file` |
| Find/replace in file | `sed 's/old/new/g' file` |
| Column processing | `awk '{print $1}' file` |
| Copy/Move/Delete | `cp` / `mv` / `rm` |
| Find files | `find . -name "*.txt"` |
| List processes | `ps aux` |
| Kill process | `kill -9 PID` |
| Run in background | `command &` |
| Redirect stdout/stderr | `command > out.log 2>&1` |
| Pipe commands | `cmd1 \| cmd2` |
| Command substitution | `$(command)` |
| Loop over items | `for i in list; do ...; done` |
| Conditional | `if [[ condition ]]; then ...; fi` |
| Function | `name() { ...; }` |
| Script arguments | `$1`, `$2`, `$@`, `$#` |
| Exit status of last command | `$?` |
| Compress/extract | `tar -czf out.tar.gz dir/` / `tar -xzf out.tar.gz` |
| Check disk space | `df -h` |
| Schedule a job | `crontab -e` |
| Lint a script | `shellcheck script.sh` |

---

*This guide covers core Bash syntax and the most commonly used commands across file handling, text processing, process control, and automation. For a deep dive on any single area (advanced `awk`/`sed`, `systemd` service scripts, or a specific script you're building), ask and it can be expanded into its own section.*
