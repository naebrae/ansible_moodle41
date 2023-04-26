# ansible_moodle

## Description

Ansible playbook to install Moodle 4.1 latest

> Includes a Samba Active Directory setup on Alpine Linux 3.17 for LDAP testing.

## Environment

- Red Hat Enterprise Linux 8
- Red Hat Enterprise Linux 9

## Usage

```
vagrant up
```

```
sudo su -c 'echo "192.168.1.141    moodle41.lab.home" >> /etc/hosts'
```

https://moodle41.lab.home

> It is a self-signed certificate so it will report a certificate error NET::ERR_CERT_AUTHORITY_INVALID


### Example inventory file

> Taken from file vagrant creates

```
moodle41 moodle_site=https://moodle41.lab.home moodle_ldaphost=ldap://dc1.ad.lab.home moodle_ldapbase=DC=ad,DC=lab,DC=home moodle_adminpass=Passw0rd! moodle_adminemail=root@localhost.localdomain moodle_fullname='Moodle Four Point One' moodle_shortname='Moodle 4.1' usegit=true
```

> Uses git clone if 'usegit=true' otherwise it downloads the lastest tgz. Will use tgz from files if found.

## Accounts

Local Accounts

| account | password |
|---|---|
| admin | Passw0rd! |

LDAP Accounts

| account | password |
|---|---|
| user1 | Passw0rd |
| user2 | Passw0rd |
| 1001A | Passw0rd |
| 1002B | Passw0rd |
| 20001 | Passw0rd |
| 20002 | Passw0rd |


## STACK plugin

The STACK question type adds a sophisticated assessment in mathematics and related disciplines, with emphasis on formative assessment underpinned by computer algebra.

### Installation

#### Maxima rpmbuild on Red Hat Enterprise Linux 8 and 9

Maxima: A Computer Algebra System (https://maxima.sourceforge.io/)

The maxima 5.44 rpm available from sourceforge won't install on RHEL 8 or 9 because of the updated readline. A solution is to rebuild the package.

##### Enable coderead-build repository

- Red Hat Enterprise Linux 8
```
sudo subscription-manager repos --enable rhel-8-server-optional-rpms --enable codeready-builder-for-rhel-8-x86_64-rpms
```

- Red Hat Enterprise Linux 9
```
sudo subscription-manager repos --enable rhel-9-server-optional-rpms --enable codeready-builder-for-rhel-9-x86_64-rpms
```

- AlmaLinux 8
```
sudo dnf config-manager --set-enabled powertools
```

- AlmaLinux 9
```
sudo dnf config-manager --set-enabled crb
```

##### Install texinfo

```
sudo dnf -y install texinfo
```

##### Install GCL

GCL - an implementation of Common Lisp
https://www.gnu.org/software/gcl/

```
# Latest version Jan 2023
curl -O https://mirror.freedif.org/GNU/gcl/gcl-2.6.14.tar.gz
tar xf gcl-2.6.14.tar.gz
cd gcl-2.6.14
./configure
make
sudo make install
```

##### rpmbuild Maxima

```
# Moodle stack plugin in Jan 2023 only supports Maxima 5.44.
curl -L -o maxima-5.44.0-1.src.rpm -O https://sourceforge.net/projects/maxima/files/Maxima-Linux/5.44.0-Linux/maxima-5.44.0-1.src.rpm/download
rpm -i maxima-5.44.0-1.src.rpm
sed -i '/^Release/ s/$/%{?dist}/' rpmbuild/SPECS/maxima-5.44.0.spec
rpmbuild -ba -D 'debug_package %{nil}' rpmbuild/SPECS/maxima-5.44.0.spec
```

##### Ansible to install Maxima, the STACK plugin, and dependents.

```
- hosts: all
  become: true
  tasks:
  - block:
    - copy: src=maxima/maxima-5.44.0-1.el{{ ansible_distribution_major_version|int }}.x86_64.rpm dest=/tmp
    - copy: src=maxima/maxima-exec-gcl-5.44.0-1.el{{ ansible_distribution_major_version|int }}.x86_64.rpm dest=/tmp
    - yum:
        name:
        - /tmp/maxima-5.44.0-1.el{{ ansible_distribution_major_version|int }}.x86_64.rpm
        - /tmp/maxima-exec-gcl-5.44.0-1.el{{ ansible_distribution_major_version|int }}.x86_64.rpm
        state: installed
        disable_gpg_check: yes
    when: ansible_distribution_major_version|int >= 8

  # https://moodle.org/plugins/qtype_stack/versions
  - unarchive: src=qtype_stack_moodle41_2023010400.zip dest=/web/moodle/question/type/ owner=root group=root
  # https://moodle.org/plugins/qbehaviour_adaptivemultipart/versions
  - unarchive: src=qbehaviour_adaptivemultipart_moodle40_2022092200.zip dest=/web/moodle/question/behaviour/ owner=root group=root
  # https://moodle.org/plugins/qbehaviour_dfcbmexplicitvaildate/versions
  - unarchive: src=qbehaviour_dfcbmexplicitvaildate_moodle40_2022092200.zip dest=/web/moodle/question/behaviour/ owner=root group=root
  # https://moodle.org/plugins/qbehaviour_dfexplicitvaildate/versions
  - unarchive: src=qbehaviour_dfexplicitvaildate_moodle40_2022092200.zip dest=/web/moodle/question/behaviour/ owner=root group=root
```

### Create STACK maxima image

At the bottom of the following page

```
Site adminstration / Plugins / Question types / STACK / healthcheck script
```

Select the **Create Maxima image**
