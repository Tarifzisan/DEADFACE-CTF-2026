# Git Good 

**Category:** Linux / Forensics / Privilege Escalation  
**Points:** 100  
**Author:** @syyntax  
**Target:** Charon  
**Initial User:** `svc-deploy`

## Challenge Description

The challenge provided access to the Charon Linux machine as `svc-deploy`.

After investigating the previous user's activity, we obtained access to the `deephax` account. This user maintained a small tooling project in their home directory and believed that all credentials had been removed from the source code.

The objective was to recover the next member's password from the Git repository, access their account, and retrieve their flag.

## 1. Accessing the Project

After switching to the `deephax` account, I navigated to the tooling project.

```bash
su - deephax
cd ~/kingmaker-scraper
```

I inspected the directory contents:

```bash
ls -la
```

### Output

```text
.git/
config.py
gh0st404_note.txt
scraper.py
```

The presence of the `.git` directory suggested that previous versions of the source code might contain sensitive information.

## 2. Inspecting Git History

I checked the repository's commit history.

```bash
git log --oneline --all --decorate
```

### Output

```text
b014c26 (HEAD -> master) move service pw out of source into env
49df4e8 initial scraper + service config
```

The latest commit message was particularly interesting:

```text
move service pw out of source into env
```

This suggested that a service password had previously existed in the source code and was later moved into an environment variable.

However, Git retains historical snapshots of committed files, even after changes are made in newer commits.

## 3. Identifying the Next User

I inspected the project note:

```bash
cat gh0st404_note.txt
```

### Output

```text
gh0st404 // scrape ops
The nightly collector under /opt/scrape feeds sunstalker's analyst pipeline.
Targets this cycle: TGRI exec calendars, Cronin Group filings.
```

This revealed the next target username:

```text
gh0st404
```

I also inspected the current configuration:

```bash
cat config.py
```

### Output

```python
import os

SERVICE_USER = "gh0st404"
SERVICE_PW = os.environ.get("SCRAPER_PW", "")
```

The password was no longer present in the current source code.

## 4. Recovering the Password from Git History

To investigate the previous version of the configuration file, I checked the initial commit.

```bash
git show 49df4e8:config.py
```

### Output

```python
# scraper service account
SERVICE_USER = "gh0st404"
SERVICE_PW = "Gh0stInTh3Sh3ll_404"
```

Success! The previous Git commit exposed the password that had been removed from the latest version.

### Recovered Credentials

| Field | Value |
|---|---|
| Username | `gh0st404` |
| Password | `Gh0stInTh3Sh3ll_404` |
| Commit | `49df4e8` |

## 5. Accessing the Next User

Using the recovered credentials, I switched to the `gh0st404` account.

```bash
su - gh0st404
```

After entering the recovered password, I verified the account:

```bash
whoami
pwd
```

### Output

```text
gh0st404
/home/gh0st404
```

Authentication was successful.

## 6. Retrieving the Flag

I inspected the user's home directory:

```bash
ls -la
```

### Output

```text
README.txt
flag.txt
```

Finally, I read the flag:

```bash
cat ~/flag.txt
```

### Output

```text
deadface{g1t_l0g_sp1lls_s3cr3ts}
```

## Flag

```text
deadface{g1t_l0g_sp1lls_s3cr3ts}
```

## Conclusion

The challenge demonstrated how sensitive credentials can remain exposed in Git history even after being removed from the current source code.

By inspecting the repository's previous commits, I recovered the password belonging to `gh0st404`, successfully accessed the account, and retrieved the flag.

### Key Takeaways

- Git stores historical versions of committed files.
- Removing credentials from the latest source code does not remove them from previous commits.
- `git log` helps identify interesting historical changes.
- `git show COMMIT:FILE` allows inspection of a specific file from an older commit.
- Secrets should never be committed to source control.
- Environment variables or dedicated secret-management systems are preferable for handling application credentials.

**Status:** Successfully Completed.
