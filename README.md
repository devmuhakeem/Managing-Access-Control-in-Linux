# Managing Access Control in Linux

A hands-on lab on identification, authentication, and authorization — creating local Linux users and groups, building a directory structure, reassigning ownership, configuring permissions, and verifying the restrictions by actually testing them from the restricted account.

## Scenario
Access control in Linux is built from a small set of command-line primitives: user/group creation, ownership assignment, and permission bits. This lab walked through all of them end to end, then proved the resulting restrictions actually held by switching into the restricted user's own shell.

## What I did

### 1. Created users and groups
```bash
useradd testuser01
passwd testuser01
useradd testuser02
passwd testuser02
groupadd admin
groupadd team
usermod -aG admin testuser01
usermod -aG team testuser02
```
- `useradd` creates the account; `passwd` sets its credential separately
- `-aG` (append to group) adds the user to the new group without removing existing group memberships — using plain `-G` instead would overwrite the user's whole group list
- Verified membership with `id testuser01` and `id testuser02`

### 2. Built a directory structure
```bash
mkdir testdirectory
mkdir testdirectory/SecProjects
mkdir testdirectory/TeamProjects
touch testdirectory/SecProjects/networkreport.txt
touch testdirectory/TeamProjects/workreport.txt
```
- Checked ownership with `ls -ld testdirectory` — owned by root by default, since it was created from a root shell

### 3. Reassigned ownership
```bash
chown -R testuser01:admin testdirectory/SecProjects
chown -R testuser02:team testdirectory/TeamProjects
```
- `user:group` syntax sets both owner and group in one command; `-R` applies it recursively so the file inside each directory is reassigned too
- Confirmed with `ls -ld` on each subdirectory

### 4. Set deliberate, differentiated permissions
```bash
chmod u=rwx,g=rwx,o-r testdirectory/SecProjects
chmod u=rwx,g=rwx,o=rx testdirectory/TeamProjects
```
- `SecProjects`: owner/group get full rwx, "others" explicitly denied read
- `TeamProjects`: owner/group get full rwx, "others" get read+execute but not write — viewable but not modifiable by outsiders

### 5. Verified the restriction from inside the restricted account
```bash
su testuser02
cd testdirectory/SecProjects
ls
```
- `testuser02` (member of `team`, not `admin`) was denied access to `SecProjects` content — confirmed from the actual restricted user's shell, not just inferred from the permission bits

## Key takeaways
- `useradd` and `passwd` are deliberately separate steps — creating an identity and granting it the ability to authenticate are two distinct actions
- Ownership is the prerequisite for security, not security itself — until `chown` reassigns it, group membership doesn't matter because the resource doesn't reference that group yet
- The `u/g/o` + `r/w/x` model is simple and covers most needs, but it's coarse — there's no way to grant one extra specific user access without widening the group or the "others" bucket; that gap is what ACLs exist to solve
- Configuring a permission is a claim; testing it from inside the restricted account (`su`) is the only real proof it holds
- The same underlying concepts (identity, group membership, owned resources, permission scope) appear in both Windows AD and Linux — just expressed through very different interfaces

## Tools
Linux (Bash), `useradd`, `groupadd`, `usermod`, `chown`, `chmod`, `su`

---
*Completed in an isolated training lab environment against a provisioned Linux machine.*
