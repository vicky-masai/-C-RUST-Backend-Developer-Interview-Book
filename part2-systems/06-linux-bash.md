# Chapter 6 — Linux, Ubuntu & Bash Scripting

> JD **must-have**: "Ubuntu and Linux commands", "Bash script and Shell Script". Interviews here are practical: "how would you find X / automate Y on a server?"

## 6.1 Filesystem hierarchy (know where things live)

| Path | Contains |
|---|---|
| `/etc` | system configuration |
| `/var/log` | logs (`syslog`, app logs) |
| `/home`, `/root` | user homes |
| `/usr/bin`, `/usr/local/bin` | executables |
| `/opt` | third-party apps |
| `/tmp` | temporary (cleared on reboot) |
| `/proc`, `/sys` | virtual: kernel & process info |
| `/dev` | devices (`/dev/null`!) |

## 6.2 The everyday command toolkit

```bash
# Navigation & files
ls -lah              # list with sizes, hidden files
find /var/log -name "*.log" -mtime -1     # logs modified in last day
du -sh */            # disk usage per directory
df -h                # free disk space
tar -czf app.tar.gz app/    # create archive; -xzf to extract

# Viewing & following
cat file; less file
tail -f /var/log/app.log         # follow a log live — daily backend task
head -n 50 file
wc -l file                        # count lines

# Processes & resources (ties to Ch 5)
ps aux | grep myapp
top / htop
kill -TERM <pid>      # graceful; kill -9 = SIGKILL last resort
nohup ./server &      # keep running after logout
jobs, fg, bg          # job control

# Networking — essential for backend debugging
ss -tlnp              # which process listens on which port (modern netstat)
curl -v http://localhost:8080/health    # test an API (Ch 8)
ping host; ip addr
scp file user@host:/path              # copy over SSH
ssh user@host
```

## 6.3 Permissions & ownership

```
-rwxr-xr--  1 alice devs 4096 Jul 1 10:00 deploy.sh
 │││││││││└ others: read only
 ││││└┴┴ group: read+execute
 │└┴┴ user: read+write+execute
 └ type: - file, d directory, l symlink
```

```bash
chmod 754 deploy.sh     # rwx r-x r-- (7=rwx, 5=r-x, 4=r--)
chmod +x script.sh      # make executable
chown alice:devs file
sudo <cmd>              # run as root
```

**Interview one-liner:** "Octal digits are user/group/other; each is read=4 + write=2 + execute=1."

## 6.4 Pipes, redirection & text processing (the power trio)

```bash
cmd > out.txt          # stdout to file (overwrite);  >> append
cmd 2> err.txt         # stderr (fd 2)
cmd > all.txt 2>&1     # both to one file
cmd1 | cmd2            # pipe stdout into stdin
cmd < input.txt        # stdin from file

# grep — search
grep -rn "ERROR" /var/log/app/          # recursive, line numbers
grep -i "timeout" app.log | wc -l       # case-insensitive count
grep -E "5[0-9]{2}" access.log          # regex: HTTP 5xx

# awk — column processing
awk '{print $1, $9}' access.log         # print columns 1 and 9
awk '$9 == 500 {c++} END {print c}' access.log   # count 500s

# sed — stream editing
sed 's/old/new/g' file                  # replace (add -i to edit in place)
sed -n '10,20p' file                    # print lines 10-20

# sort/uniq — the classic "top N" pipeline
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
# → top 10 IPs by request count — a VERY common interview task
```

```mermaid
flowchart LR
    A[access.log] --> B["awk: extract IP column"] --> C[sort] --> D["uniq -c: count"] --> E["sort -rn: rank"] --> F["head: top 10"]
```

## 6.5 Ubuntu administration essentials

```bash
# Packages (apt)
sudo apt update && sudo apt upgrade
sudo apt install postgresql
apt list --installed | grep postgres

# Services (systemd)
sudo systemctl status myapp
sudo systemctl start|stop|restart|enable myapp   # enable = start on boot
journalctl -u myapp -f          # follow a service's logs
journalctl -u myapp --since "1 hour ago"

# Scheduling
crontab -e
# ┌─min ┌─hour ┌─day ┌─month ┌─weekday
#   0     3     *     *      *    /opt/scripts/backup.sh   # daily 03:00

# Environment
export DATABASE_URL=postgres://...   # env var for current shell
echo $PATH
which cargo                           # where does a command resolve from
```

A minimal **systemd unit** (deployment questions love this):

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Rust backend
After=network.target postgresql.service

[Service]
ExecStart=/opt/myapp/server
Restart=on-failure
Environment=RUST_LOG=info
User=appuser

[Install]
WantedBy=multi-user.target
```

## 6.6 Bash scripting — the essentials

```bash
#!/usr/bin/env bash
set -euo pipefail    # THE professional header — explain each flag:
# -e: exit on any command failure
# -u: error on undefined variables
# -o pipefail: a pipeline fails if ANY stage fails

# Variables (no spaces around =) and quoting (ALWAYS quote expansions)
name="world"
echo "Hello, ${name}"

# Command substitution & arithmetic
today=$(date +%F)
count=$(( 3 + 4 ))

# Arguments & exit codes
# $0 script name, $1..$n args, $# arg count, $@ all args, $? last exit code
if [[ $# -lt 1 ]]; then
    echo "usage: $0 <env>" >&2
    exit 1
fi

# Conditionals
if [[ -f "/etc/app.conf" ]]; then echo "config exists"; fi
if [[ "$1" == "prod" && -n "${TOKEN:-}" ]]; then echo "deploying"; fi
# -f file exists, -d dir exists, -z empty, -n non-empty, -x executable

# Loops
for f in logs/*.log; do gzip "$f"; done
while read -r line; do echo "got: $line"; done < input.txt

# Functions
log() { echo "[$(date +%T)] $*"; }
log "starting deployment"

# Error handling pattern
trap 'echo "failed at line $LINENO" >&2' ERR
```

### A realistic script interviewers might ask you to sketch

```bash
#!/usr/bin/env bash
# health-check.sh — restart service if the health endpoint fails
set -euo pipefail

URL="http://localhost:8080/health"
SERVICE="myapp"
MAX_RETRIES=3

for i in $(seq 1 "$MAX_RETRIES"); do
    if curl -sf --max-time 5 "$URL" > /dev/null; then
        echo "healthy"
        exit 0
    fi
    echo "check $i/$MAX_RETRIES failed" >&2
    sleep 2
done

echo "unhealthy — restarting $SERVICE" >&2
sudo systemctl restart "$SERVICE"
```

### Bash pitfalls (trap questions)

| Pitfall | Fix |
|---|---|
| `rm -rf $DIR/` with unset DIR → `rm -rf /` | `set -u` and quote: `"${DIR:?}"` |
| Unquoted `$var` splits on spaces | always `"$var"` |
| `cmd1 && cmd2 \|\| cmd3` isn't if/else | use a real `if` |
| Parsing `ls` output | use globs or `find -print0 \| xargs -0` |
| `[ ]` vs `[[ ]]` | prefer `[[ ]]` in bash (safer, regex `=~`) |

Name-drop: **shellcheck** — the linter for shell scripts; run it in CI.

---

## 🎯 Chapter 6 Interview Q&A

**Q1. Find all ERROR lines from the last hour across rotated logs?**
`journalctl -u app --since "1 hour ago" | grep ERROR` or `find /var/log/app -mmin -60 -name '*.log' -exec grep -Hn ERROR {} +`.

**Q2. A port is already in use — how do you find the culprit?**
`ss -tlnp | grep :8080` (or `lsof -i :8080`), then inspect/kill that PID.

**Q3. What does `2>&1` mean?**
Redirect fd 2 (stderr) to wherever fd 1 (stdout) currently points — order matters: `> file 2>&1`, not the reverse.

**Q4. Difference between `&&`, `;`, and `\|` ?**
`&&` runs next only on success; `;` runs regardless; `\|` pipes stdout of one into stdin of the next.

**Q5. What is `set -euo pipefail`?**
Fail fast on errors, on undefined variables, and on any failure inside a pipeline — baseline for production scripts.

**Q6. How do you keep a process running after you log out?**
`nohup cmd &`, `tmux`/`screen`, or properly: a systemd service with `Restart=on-failure`.

**Q7. Disk is full — walk me through your steps.**
`df -h` find the filesystem → `du -sh /* 2>/dev/null | sort -h` drill down → usually `/var/log`: truncate/compress, configure logrotate. Check deleted-but-open files: `lsof +L1`.

**Q8. grep vs awk vs sed — one line each?**
grep filters lines by pattern; awk processes columns/fields with logic; sed transforms text streams (substitutions).

**Q9. What are soft vs hard links?**
Hard link: another directory entry for the same inode (survives original deletion, same filesystem only). Symlink: a small file containing a path (can dangle, can cross filesystems).

**Q10. How do environment variables propagate?**
Children inherit the parent's exported environment at exec time. `export` marks a var for inheritance; changes never flow back to the parent shell.

**Q11. Server is slow — first five commands you run?**
`top`/`htop` (CPU, load, memory), `free -h` (swap?), `df -h` (disk full?), `iostat`/`vmstat 1` (I/O or context-switch storm), `ss -s` + app logs (`journalctl -u app`). Then dig where the anomaly is.

**Q12. Write a one-liner: top 5 most frequent words in a file.**
`tr -s ' ' '\n' < file | sort | uniq -c | sort -rn | head -5`
