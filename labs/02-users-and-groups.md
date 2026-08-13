# Lab 02 - Users and Groups

## Objective

Practice Linux user and group administration on `ubuntu-server01`.

The lab covers:

* Identifying users and group memberships
* Understanding Linux account files
* Creating and modifying users
* Managing passwords
* Switching between users
* Creating and managing groups
* Primary and supplementary groups
* Effective group membership
* Locking and unlocking accounts
* Removing groups

---

## Environment

| Component           | Details                   |
| ------------------- | ------------------------- |
| Hostname            | `ubuntu-server01`         |
| Operating System    | Ubuntu Server 24.04.4 LTS |
| Administrative User | `hanlong`                 |
| Test User           | `alice`                   |
| Test Group          | `developers`              |
| Remote Access       | SSH                       |
| Server IP           | `10.10.10.11`             |

---

## 1. Investigating the Current User

Checked the currently logged-in user:

```bash
whoami
```

Result:

```text
hanlong
```

Displayed the user's UID, primary GID, and group memberships:

```bash
id
```

The account had:

```text
UID:            1000
Primary GID:    1000
Primary Group:  hanlong
```

Displayed all groups associated with the current session:

```bash
groups
```

The account was a member of several groups, including:

```text
hanlong adm cdrom sudo dip plugdev lxd
```

The `sudo` supplementary group provides the account with administrative privileges through `sudo`.

---

## 2. Examining `/etc/passwd`

Displayed the account entry for `hanlong`:

```bash
grep '^hanlong:' /etc/passwd
```

A `/etc/passwd` entry contains seven colon-separated fields:

```text
username:x:UID:GID:GECOS:home:shell
```

| Field    | Purpose                                         |
| -------- | ----------------------------------------------- |
| Username | Login/account name                              |
| `x`      | Password information is stored in `/etc/shadow` |
| UID      | User ID                                         |
| GID      | Primary group ID                                |
| GECOS    | Account information/comment                     |
| Home     | User's home directory                           |
| Shell    | Login shell                                     |

For `hanlong`, the home directory was:

```text
/home/hanlong
```

and the login shell was:

```text
/bin/bash
```

---

## 3. Examining `/etc/group`

Group information was inspected using:

```bash
cat /etc/group
```

Specific groups can be queried more efficiently using:

```bash
getent group sudo
```

The `hanlong` account appeared as a member of the `sudo` group.

This demonstrated the difference between the user's primary group and supplementary groups.

---

## 4. Examining `/etc/shadow`

Attempted to read the shadow password database as a normal user:

```bash
cat /etc/shadow
```

Result:

```text
Permission denied
```

Compared the permissions of the account databases:

```bash
ls -l /etc/passwd /etc/shadow /etc/group
```

`/etc/passwd` is readable by normal users because applications need access to basic account information.

`/etc/shadow` is significantly more restricted because it contains password hashes and password-aging information.

The account's shadow entry could be inspected with administrative privileges:

```bash
sudo grep '^hanlong:' /etc/shadow
```

Sensitive shadow entries were not included in the documentation.

---

## 5. Creating a User

Verified that the test account did not already exist:

```bash
id alice
```

Created the account:

```bash
sudo useradd -m -s /bin/bash alice
```

The options used were:

```text
-m    Create the user's home directory
-s    Specify the user's login shell
```

The resulting account was verified using:

```bash
id alice
getent passwd alice
```

Alice was assigned:

```text
UID:            1001
Primary GID:    1001
Primary Group:  alice
Home:           /home/alice
Shell:          /bin/bash
```

---

## 6. User Home Directory and `/etc/skel`

Verified that Alice's home directory was created:

```bash
ls -ld /home/alice
```

Inspected its contents:

```bash
sudo ls -la /home/alice
```

Compared them with:

```bash
ls -la /etc/skel
```

Files such as the following were present:

```text
.bashrc
.profile
.bash_logout
```

### Observation

When `useradd -m` creates a home directory, files from `/etc/skel` are used to populate the new user's home directory.

`/etc/skel` therefore provides the default starting environment for newly created users.

---

## 7. Password Status

Checked Alice's password status:

```bash
sudo passwd -S alice
```

Initially, the status showed:

```text
alice L ...
```

`L` indicated that password authentication was locked.

A password was assigned using:

```bash
sudo passwd alice
```

The status was checked again:

```bash
sudo passwd -S alice
```

The status changed to:

```text
alice P ...
```

where `P` indicates that a usable password has been configured.

---

## 8. Switching Users

Switched from `hanlong` to Alice using:

```bash
su - alice
```

Verified the new identity and environment:

```bash
whoami
id
pwd
groups
```

Results confirmed:

```text
User: alice
Home: /home/alice
```

Alice was unable to perform administrative operations such as:

```bash
sudo apt update
```

because she was not a member of the `sudo` group.

Returned to the original account using:

```bash
exit
```

---

## 9. `su` vs `su -`

Both forms were tested:

```bash
su alice
```

and:

```bash
su - alice
```

### Observation

`su alice` changes the user identity while largely preserving the existing environment.

`su - alice` starts a login shell for Alice and provides an environment similar to Alice logging in directly.

The `-` does not specify the username. `alice` specifies the target user.

---

## 10. Creating Groups

Created two test groups:

```bash
sudo groupadd developers
sudo groupadd operations
```

Verified their existence using:

```bash
getent group developers
getent group operations
```

---

## 11. Adding Users to a Supplementary Group

Added Alice to the `developers` group:

```bash
sudo usermod -aG developers alice
```

Added `hanlong`:

```bash
sudo usermod -aG developers hanlong
```

The options mean:

```text
-G    Specify supplementary groups
-a    Append instead of replacing existing supplementary groups
```

Using `-aG` prevents existing supplementary group memberships from being removed.

Membership was verified using:

```bash
id alice
id hanlong
getent group developers
```

The resulting group contained:

```text
developers: ... :alice,hanlong
```

---

## 12. Group Changes and Login Sessions

After adding `hanlong` to `developers`, running:

```bash
groups
```

did not initially display the new group.

However:

```bash
id hanlong
```

and:

```bash
getent group developers
```

showed that the account database had been updated.

After logging out of SSH and reconnecting:

```powershell
ssh ubuntu-server01
```

the new group appeared when running:

```bash
groups
```

### Observation

Changes to supplementary group membership do not necessarily affect an already-running login session.

A new login session loads the updated group memberships.

---

## 13. Primary and Effective Groups

Created a file while using the normal login environment:

```bash
touch ~/primary-group-test.txt
```

Checked its ownership:

```bash
ls -l ~/primary-group-test.txt
```

The file was owned by:

```text
hanlong hanlong
```

The first value represents the owner and the second represents the group owner.

Changed the effective group by starting a new shell:

```bash
newgrp developers
```

Verified the effective group:

```bash
id -gn
```

Created another file:

```bash
touch ~/developer-test.txt
```

Its ownership showed:

```text
hanlong developers
```

Exited the `newgrp` shell:

```bash
exit
```

The effective group returned to:

```text
hanlong
```

### Observation

Being a member of a supplementary group does not automatically make it the group owner of every newly created file.

`newgrp developers` starts a new shell using `developers` as the effective group.

New files created within that shell can therefore receive `developers` as their group ownership.

---

## 14. Locking and Unlocking an Account

Checked Alice's initial status:

```bash
sudo passwd -S alice
```

The account showed:

```text
P
```

Locked password authentication:

```bash
sudo passwd -l alice
```

The status changed to:

```text
L
```

Attempting to authenticate using:

```bash
su - alice
```

failed while the password was locked.

The password was unlocked using:

```bash
sudo passwd -u alice
```

The account returned to:

```text
P
```

---

## 15. Removing a Group

The unused `operations` group was removed.

The normal command for removing a group is:

```bash
sudo groupdel operations
```

The group was verified as removed using:

```bash
getent group operations
```

No output was returned, confirming that the group no longer existed.

---

## Final Lab State

The following accounts and groups were retained for the next lab:

```text
Users
├── hanlong
└── alice

developers
├── hanlong
└── alice
```

Alice remains a normal non-administrative user.

Both users are members of the `developers` supplementary group.

These accounts will be used in the next lab to configure shared directory ownership and permissions.

---

## Key Takeaways

From this lab I learned:

* Linux identifies users and groups using UIDs and GIDs.
* `/etc/passwd` contains general account information.
* `/etc/shadow` contains protected password information.
* `/etc/group` stores group information.
* `/etc/skel` provides default files for new home directories.
* Creating a user does not necessarily create a usable password.
* Primary and supplementary groups serve different purposes.
* `usermod -aG` safely appends supplementary group membership.
* Existing login sessions may not immediately receive new group memberships.
* `newgrp` can change the effective group for a new shell.
* Normal users do not automatically receive administrative privileges.
* Password authentication can be locked and unlocked without deleting an account.
* Administrative changes should be verified after they are made.

---

## Result

Successfully created and administered Linux users and groups on `ubuntu-server01`.

The lab demonstrated account creation, password management, group membership, login environments, account locking, effective groups, and verification of administrative changes.

The retained `developers` group and `alice` account will be used in the next lab for file ownership and permissions.
