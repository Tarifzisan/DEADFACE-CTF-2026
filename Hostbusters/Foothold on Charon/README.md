# Foothold on Charon 

**Category:** Linux / Forensics / Privilege Escalation  
**Points:** 10  
**Author:** @syyntax  
**Target:** `charon`  
**Initial User:** `svc-deploy`

## Challenge Description

DEADFACE operates an internal operations and staging machine named **Charon**, used by its crew to coordinate their influence operations.

We were provided initial access to the machine through a web-based shell as the `svc-deploy` service account. The objective was to investigate the system and retrieve the flag proving our foothold.

## 1. Initial Access

After opening the Hostbusters web-shell, I was provided access to the Charon machine as `svc-deploy`.

I started by checking the current user, privileges, hostname, and working directory.

```bash
whoami
id
hostname
pwd
```

### Output

```text
svc-deploy
uid=1001(svc-deploy) gid=1001(svc-deploy) groups=1001(svc-deploy),1001(svc-deploy)
charon
/home/svc-deploy
```

This confirmed that I had successfully obtained access to the target machine as the `svc-deploy` user.

## 2. Enumerating the Home Directory

Next, I inspected the current user's home directory, including hidden files.

```bash
ls -la
```

### Output

```text
total 16
drwxr-x---    2 svc-depl svc-depl      4096 Oct 18 01:14 .
drwxr-xr-x    1 root     root          4096 Oct 18 01:14 ..
-rw-------    1 svc-depl svc-depl       196 Oct 18 01:14 .bash_history
-rw-r-----    1 svc-depl svc-depl        33 Oct 18 01:14 flag.txt
```

The listing revealed an interesting file named `flag.txt` in the home directory.

The file was readable by the current user, so I decided to inspect its contents.

## 3. Retrieving the Foothold Flag

I used the `cat` command to read the file.

```bash
cat flag.txt
```

### Output

```text
deadface{f00th0ld_p4st_th3_v31l}
```

## Flag

**Foothold Flag:**

```text
deadface{f00th0ld_p4st_th3_v31l}
```

## Conclusion

The challenge provided an initial foothold on the Charon Linux machine through a web-shell session running as `svc-deploy`.

By performing basic Linux enumeration and inspecting the user's home directory, I discovered `flag.txt` and successfully retrieved the foothold flag.

**Key Takeaways:**
- Identifying the current user and group permissions.
- Understanding basic Linux filesystem enumeration.
- Inspecting hidden files and user home directories.
- Retrieving flags through terminal-based investigation.

**Status:** Initial Foothold Obtained.
