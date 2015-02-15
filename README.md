# Dynamically Generate Markdown Docs for Ansible Modules
The module `ansible_docstring` was built to assist in the generation of Ansible-like web docs mainly for the purpose of having offline web docs for custom modules.  However, it can also be used to generate web docs for Ansible core modules as can be seen from the example.

Unlike Ansible's `make webdocs` functionality, this can generate a single markdown file for just ONE module.  There is no need to generate complete web docs for every module when they aren't needed.

There are more error checks that could be built into the module and Jinja2 template to possibly make this more user friendly and more robust.  If there are any suggestions, please feel free to open an issue or PR.

### Example Playbook (as can also be seen in the examples dir):
```
---

- name: create local markdown doc
  hosts: localhost
  connection: local
  gather_facts: no

  vars:
    heading: 'Ansible FILES modules'
    subheading: 'Local copy of files modules'
    requirements:
      - See official Ansible docs

  tasks:

    - name: get docs and examples for modules
      ansible_docstring: path=/usr/share/ansible/files/
      register: modules

    - name: build web/markdown ansible docs
      template: src=templates/ansible-docs.j2 dest=web/ansiblefilesdoc.md
```

###Running this playbook:

```
jason@ansible:~/ansible/gendoc$ ansible-playbook -i hosts create-ansible-webdoc.yml

PLAY [create local markdown doc] ********************************************** 

TASK: [get docs and examples for modules] ************************************* 
ok: [localhost]

TASK: [build web/markdown ansible docs] *************************************** 
changed: [localhost]

PLAY RECAP ******************************************************************** 
localhost                  : ok=2    changed=1    unreachable=0    failed=0   

jason@ansible:~/ansible/gendoc$ 
```


### Generates the following markdown file:
Slight modifications made for the Examples section to be able to show the markdown in a markdown a file.
```
# Ansible FILES modules
### *Local copy of files modules*

---
### Requirements
 * See official Ansible docs

---
### Modules

  * [template - templates a file out to a remote server](#template)
  * [unarchive - copies an archive to a remote location and unpack it](#unarchive)
  * [replace - replace all instances of a particular string in a file using a back-referenced regular expression](#replace)
  * [copy - copies files to remote locations](#copy)
  * [lineinfile - ensure a particular line is in a file, or replace an existing line using a back-referenced regular expression](#lineinfile)
  * [acl - sets and retrieves file acl information](#acl)
  * [synchronize - uses rsync to make synchronizing file paths in your playbooks quick and easy](#synchronize)
  * [assemble - assembles a configuration file from fragments](#assemble)
  * [xattr - set/retrieve extended attributes](#xattr)
  * [stat - retrieve file or file system status](#stat)
  * [file - sets attributes of files](#file)
  * [ini_file - tweak settings in ini files](#ini_file)
  * [fetch - fetches a file from remote nodes](#fetch)
---

#### template
Templates a file out to a remote server

  * Synopsis
  * Options
  * Examples

#### Synopsis
 Templates are processed by the Jinja2 templating language (U(http://jinja.pocoo.org/docs/)) - documentation on the template formatting can be found in the Template Designer Documentation (U(http://jinja.pocoo.org/docs/templates/)).
 Six additional variables can be used in templates: C(ansible_managed) (configurable via the C(defaults) section of C(ansible.cfg)) contains a string which can be used to describe the template name, host, modification time of the template file and the owner uid, C(template_host) contains the node name of the template's machine, C(template_uid) the owner, C(template_path) the absolute path of the template, C(template_fullpath) is the absolute path of the template, and C(template_run_date) is the date that the template was rendered. Note that including a string that uses a date in the template will result in the template being marked 'changed' each time.

#### Options

| Parameter     | required    | default  | choices    | comments |
| ------------- |-------------| ---------|----------- |--------- |
                      | dest  |   yes  |  | |  Location to render the template to on the remote machine.  |
    | src  |   yes  |  | |  Path of a Jinja2 formatted template on the local server. This can be a relative or absolute path.  |
    | validate  |   no  |  | |  The validation command to run before copying into place.  The path to the file to validate is passed in via '%s' which must be present as in the visudo example below.  validation to run before copying into place. The command is passed securely so shell features like expansion and pipes won't work.  |
    | backup  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Create a backup file including the timestamp information so you can get the original file back if you somehow clobbered it incorrectly.  |
      
#### Examples

# Example from Ansible Playbooks
- template: src=/mytemplates/foo.j2 dest=/etc/file.conf owner=bin group=wheel mode=0644

# Copy a new "sudoers" file into place, after passing validation with visudo
- template: src=/mine/sudoers dest=/etc/sudoers validate='visudo -cf %s'


#### Notes
- Since Ansible version 0.9, templates are loaded with C(trim_blocks=True).


---


#### unarchive
Copies an archive to a remote location and unpack it

  * Synopsis
  * Options
  * Examples

#### Synopsis
 The M(unarchive) module copies an archive file from the local machine to a remote and unpacks it.

#### Options

| Parameter     | required    | default  | choices    | comments |
| ------------- |-------------| ---------|----------- |--------- |
                      | dest  |   yes  |  | |  Remote absolute path where the archive should be unpacked  |
    | src  |   yes  |  | |  Local path to archive file to copy to the remote server; can be absolute or relative.  |
    | copy  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Should the file be copied from the local to the remote machine?  |
    | creates  |   no  |  | |  a filename, when it already exists, this step will B(not) be run.  |
      
#### Examples

# Example from Ansible Playbooks
- unarchive: src=foo.tgz dest=/var/lib/foo

# Unarchive a file that is already on the remote machine
- unarchive: src=/tmp/foo.zip dest=/usr/local/bin copy=no


#### Notes
- requires C(tar)/C(unzip) command on target host

- can handle I(gzip), I(bzip2) and I(xz) compressed as well as uncompressed tar files

- detects type of archive automatically

- uses tar's C(--diff arg) to calculate if changed or not. If this C(arg) is not supported, it will always unpack the archive

- does not detect if a .zip file is different from destination - always unzips

- existing files/directories in the destination which are not in the archive are not touched.  This is the same behavior as a normal archive extraction

- existing files/directories in the destination which are not in the archive are ignored for purposes of deciding if the archive should be unpacked or not


---


#### replace
Replace all instances of a particular string in a file using a back-referenced regular expression

  * Synopsis
  * Options
  * Examples

#### Synopsis
 This module will replace all instances of a pattern within a file.
 It is up to the user to maintain idempotence by ensuring that the same pattern would never match any replacements made.

#### Options

| Parameter     | required    | default  | choices    | comments |
| ------------- |-------------| ---------|----------- |--------- |
                  | dest  |   yes  |  | |  The file to modify.  |
    | replace  |   no  |  | |  The string to replace regexp matches. May contain backreferences that will get expanded with the regexp capture groups if the regexp matches. If not set, matches are removed entirely.  |
    | others  |   no  |  | |  All arguments accepted by the M(file) module also work here.  |
    | regexp  |   yes  |  | |  The regular expression to look for in the contents of the file. Uses Python regular expressions; see U(http://docs.python.org/2/library/re.html). Uses multiline mode, which means C(^) and C($) match the beginning and end respectively of I(each line) of the file.  |
    | validate  |   no  |  | |  validation to run before copying into place  |
    | backup  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Create a backup file including the timestamp information so you can get the original file back if you somehow clobbered it incorrectly.  |
      
#### Examples

- replace: dest=/etc/hosts regexp='(\s+)old\.host\.name(\s+.*)?$' replace='\1new.host.name\2' backup=yes

- replace: dest=/home/jdoe/.ssh/known_hosts regexp='^old\.host\.name[^\n]*\n' owner=jdoe group=jdoe mode=644

- replace: dest=/etc/apache/ports regexp='^(NameVirtualHost|Listen)\s+80\s*$' replace='\1 127.0.0.1:8080' validate='/usr/sbin/apache2ctl -f %s -t'


---


#### copy
Copies files to remote locations

  * Synopsis
  * Options
  * Examples

#### Synopsis
 The M(copy) module copies a file on the local box to remote locations.

#### Options

| Parameter     | required    | default  | choices    | comments |
| ------------- |-------------| ---------|----------- |--------- |
                      | src  |   no  |  | |  Local path to a file to copy to the remote server; can be absolute or relative. If path is a directory, it is copied recursively. In this case, if path ends with "/", only inside contents of that directory are copied to destination. Otherwise, if it does not end with "/", the directory itself with all contents is copied. This behavior is similar to Rsync.  |
    | directory_mode  |   no  |  | |  When doing a recursive copy set the mode for the directories. If this is not set we will default the system defaults.  |
    | force  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  the default is C(yes), which will replace the remote file when contents are different than the source.  If C(no), the file will only be transferred if the destination does not exist.  |
    | dest  |   yes  |  | |  Remote absolute path where the file should be copied to. If src is a directory, this must be a directory too.  |
    | selevel  |   no  |  | <ul></ul> |  level part of the SELinux file context. This is the MLS/MCS attribute, sometimes known as the C(range). C(_default) feature works as for I(seuser).  |
    | seuser  |   no  |  | <ul></ul> |  user part of SELinux file context. Will default to system policy, if applicable. If set to C(_default), it will use the C(user) portion of the policy if available  |
    | recurse  |   no  |  | |  Copy all contents in the source directory recursively.  This will be slightly inefficient compared to the 'synchronize' module and should not be used for large directory trees.  |
    | serole  |   no  |  | <ul></ul> |  role part of SELinux file context, C(_default) feature works as for I(seuser).  |
    | group  |   no  |  | <ul></ul> |  name of the group that should own the file/directory, as would be fed to I(chown)  |
    | content  |   no  |  | |  When used instead of 'src', sets the contents of a file directly to the specified value.  |
    | setype  |   no  |  | <ul></ul> |  type part of SELinux file context, C(_default) feature works as for I(seuser).  |
    | mode  |   no  |  | <ul></ul> |  mode the file or directory should be, such as 0644 as would be fed to I(chmod). As of version 1.8, the mode may be specified as a symbolic mode (for example, C(u+rwx) or C(u=rw,g=r,o=r)).  |
    | owner  |   no  |  | <ul></ul> |  name of the user that should own the file/directory, as would be fed to I(chown)  |
    | follow  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  This flag indicates that filesystem links, if they exist, should be followed.  |
    | validate  |   no  |  | |  The validation command to run before copying into place.  The path to the file to validate is passed in via '%s' which must be present as in the visudo example below. The command is passed securely so shell features like expansion and pipes won't work.  |
    | backup  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Create a backup file including the timestamp information so you can get the original file back if you somehow clobbered it incorrectly.  |
      
#### Examples

# Example from Ansible Playbooks
- copy: src=/srv/myfiles/foo.conf dest=/etc/foo.conf owner=foo group=foo mode=0644

# Copy a new "ntp.conf file into place, backing up the original if it differs from the copied version
- copy: src=/mine/ntp.conf dest=/etc/ntp.conf owner=root group=root mode=644 backup=yes

# Copy a new "sudoers" file into place, after passing validation with visudo
- copy: src=/mine/sudoers dest=/etc/sudoers validate='visudo -cf %s'


#### Notes
- The "copy" module recursively copy facility does not scale to lots (>hundreds) of files. For alternative, see synchronize module, which is a wrapper around rsync.

---
---
Created by Jason Edelman. February 2015.
```

## Viewing the results in a Browser
Check out the web directory in this repo.