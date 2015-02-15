# Ansible FILES modules
### *Local copy of files modules*

---
### Requirements
 * See official Ansible docs

---
### Modules

  * [template - templates a file out to a remote server.](#template)
  * [unarchive - copies an archive to a remote location and unpack it](#unarchive)
  * [replace - replace all instances of a particular string in a file using a back-referenced regular expression.](#replace)
  * [copy - copies files to remote locations.](#copy)
  * [lineinfile - ensure a particular line is in a file, or replace an existing line using a back-referenced regular expression.](#lineinfile)
  * [acl - sets and retrieves file acl information.](#acl)
  * [synchronize - uses rsync to make synchronizing file paths in your playbooks quick and easy.](#synchronize)
  * [assemble - assembles a configuration file from fragments](#assemble)
  * [xattr - set/retrieve extended attributes](#xattr)
  * [stat - retrieve file or file system status](#stat)
  * [file - sets attributes of files](#file)
  * [ini_file - tweak settings in ini files](#ini_file)
  * [fetch - fetches a file from remote nodes](#fetch)

---

#### template
Templates a file out to a remote server.

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
```
# Example from Ansible Playbooks
- template: src=/mytemplates/foo.j2 dest=/etc/file.conf owner=bin group=wheel mode=0644

# Copy a new "sudoers" file into place, after passing validation with visudo
- template: src=/mine/sudoers dest=/etc/sudoers validate='visudo -cf %s'

```

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
```
# Example from Ansible Playbooks
- unarchive: src=foo.tgz dest=/var/lib/foo

# Unarchive a file that is already on the remote machine
- unarchive: src=/tmp/foo.zip dest=/usr/local/bin copy=no

```

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
Replace all instances of a particular string in a file using a back-referenced regular expression.

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
```
- replace: dest=/etc/hosts regexp='(\s+)old\.host\.name(\s+.*)?$' replace='\1new.host.name\2' backup=yes

- replace: dest=/home/jdoe/.ssh/known_hosts regexp='^old\.host\.name[^\n]*\n' owner=jdoe group=jdoe mode=644

- replace: dest=/etc/apache/ports regexp='^(NameVirtualHost|Listen)\s+80\s*$' replace='\1 127.0.0.1:8080' validate='/usr/sbin/apache2ctl -f %s -t'

```


---


#### copy
Copies files to remote locations.

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
```
# Example from Ansible Playbooks
- copy: src=/srv/myfiles/foo.conf dest=/etc/foo.conf owner=foo group=foo mode=0644

# Copy a new "ntp.conf file into place, backing up the original if it differs from the copied version
- copy: src=/mine/ntp.conf dest=/etc/ntp.conf owner=root group=root mode=644 backup=yes

# Copy a new "sudoers" file into place, after passing validation with visudo
- copy: src=/mine/sudoers dest=/etc/sudoers validate='visudo -cf %s'

```

#### Notes
- The "copy" module recursively copy facility does not scale to lots (>hundreds) of files. For alternative, see synchronize module, which is a wrapper around rsync.


---


#### lineinfile
Ensure a particular line is in a file, or replace an existing line using a back-referenced regular expression.

  * Synopsis
  * Options
  * Examples

#### Synopsis
 This module will search a file for a line, and ensure that it is present or absent.
 This is primarily useful when you want to change a single line in a file only. For other cases, see the M(copy) or M(template) modules.

#### Options

| Parameter     | required    | default  | choices    | comments |
| ------------- |-------------| ---------|----------- |--------- |
| insertbefore  |   no  |  | <ul> <li>BOF</li>  <li>*regex*</li> </ul> |  Used with C(state=present). If specified, the line will be inserted before the specified regular expression. A value is available; C(BOF) for inserting the line at the beginning of the file. May not be used with C(backrefs).  |
| dest  |   yes  |  | |  The file to modify.  |
| create  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Used with C(state=present). If specified, the file will be created if it does not already exist. By default it will fail if the file is missing.  |
| backrefs  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Used with C(state=present). If set, line can contain backreferences (both positional and named) that will get populated if the C(regexp) matches. This flag changes the operation of the module slightly; C(insertbefore) and C(insertafter) will be ignored, and if the C(regexp) doesn't match anywhere in the file, the file will be left unchanged. If the C(regexp) does match, the last matching line will be replaced by the expanded line parameter.  |
| state  |   no  |  | <ul> <li>present</li>  <li>absent</li> </ul> |  Whether the line should be there or not.  |
| others  |   no  |  | |  All arguments accepted by the M(file) module also work here.  |
| insertafter  |   no  |  | <ul> <li>EOF</li>  <li>*regex*</li> </ul> |  Used with C(state=present). If specified, the line will be inserted after the specified regular expression. A special value is available; C(EOF) for inserting the line at the end of the file. May not be used with C(backrefs).  |
| regexp  |   no  |  | |  The regular expression to look for in every line of the file. For C(state=present), the pattern to replace if found; only the last line found will be replaced. For C(state=absent), the pattern of the line to remove.  Uses Python regular expressions; see U(http://docs.python.org/2/library/re.html).  |
| line  |   no  |  | |  Required for C(state=present). The line to insert/replace into the file. If C(backrefs) is set, may contain backreferences that will get expanded with the C(regexp) capture groups if the regexp matches. The backreferences should be double escaped (see examples).  |
| backup  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Create a backup file including the timestamp information so you can get the original file back if you somehow clobbered it incorrectly.  |
| validate  |   no  |  | |  validation to run before copying into place. Use %s in the command to indicate the current file to validate. The command is passed securely so shell features like expansion and pipes won't work.  |

#### Examples
```
- lineinfile: dest=/etc/selinux/config regexp=^SELINUX= line=SELINUX=disabled

- lineinfile: dest=/etc/sudoers state=absent regexp="^%wheel"

- lineinfile: dest=/etc/hosts regexp='^127\.0\.0\.1' line='127.0.0.1 localhost' owner=root group=root mode=0644

- lineinfile: dest=/etc/httpd/conf/httpd.conf regexp="^Listen " insertafter="^#Listen " line="Listen 8080"

- lineinfile: dest=/etc/services regexp="^# port for http" insertbefore="^www.*80/tcp" line="# port for http by default"

# Add a line to a file if it does not exist, without passing regexp
- lineinfile: dest=/tmp/testfile line="192.168.1.99 foo.lab.net foo"

# Fully quoted because of the ': ' on the line. See the Gotchas in the YAML docs.
- lineinfile: "dest=/etc/sudoers state=present regexp='^%wheel' line='%wheel ALL=(ALL) NOPASSWD: ALL'"

- lineinfile: dest=/opt/jboss-as/bin/standalone.conf regexp='^(.*)Xms(\d+)m(.*)$' line='\1Xms${xms}m\3' backrefs=yes

# Validate a the sudoers file before saving
- lineinfile: dest=/etc/sudoers state=present regexp='^%ADMIN ALL\=' line='%ADMIN ALL=(ALL) NOPASSWD:ALL' validate='visudo -cf %s'

```


---


#### acl
Sets and retrieves file ACL information.

  * Synopsis
  * Options
  * Examples

#### Synopsis
 Sets and retrieves file ACL information.

#### Options

| Parameter     | required    | default  | choices    | comments |
| ------------- |-------------| ---------|----------- |--------- |
| name  |   yes  |  | |  The full path of the file or object.  |
| default  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  if the target is a directory, setting this to yes will make it the default acl for entities created inside the directory. It causes an error if name is a file.  |
| entity  |   no  |  | |  actual user or group that the ACL applies to when matching entity types user or group are selected.  |
| state  |   no  |  | <ul> <li>query</li>  <li>present</li>  <li>absent</li> </ul> |  defines whether the ACL should be present or not.  The C(query) state gets the current acl without changing it, for use in 'register' operations.  |
| entry  |   no  |  | |  DEPRECATED. The acl to set or remove.  This must always be quoted in the form of '<etype>:<qualifier>:<perms>'.  The qualifier may be empty for some types, but the type and perms are always requried. '-' can be used as placeholder when you do not care about permissions. This is now superceeded by entity, type and permissions fields.  |
| etype  |   no  |  | <ul> <li>user</li>  <li>group</li>  <li>mask</li>  <li>other</li> </ul> |  the entity type of the ACL to apply, see setfacl documentation for more info.  |
| follow  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  whether to follow symlinks on the path if a symlink is encountered.  |
| permissions  |   no  |  | |  Permissions to apply/remove can be any combination of r, w and  x (read, write and execute respectively)  |

#### Examples
```
# Grant user Joe read access to a file
- acl: name=/etc/foo.conf entity=joe etype=user permissions="r" state=present

# Removes the acl for Joe on a specific file
- acl: name=/etc/foo.conf entity=joe etype=user state=absent

# Sets default acl for joe on foo.d
- acl: name=/etc/foo.d entity=joe etype=user permissions=rw default=yes state=present

# Same as previous but using entry shorthand
- acl: name=/etc/foo.d entry="default:user:joe:rw-" state=present

# Obtain the acl for a specific file
- acl: name=/etc/foo.conf
  register: acl_info

```

#### Notes
- The "acl" module requires that acls are enabled on the target filesystem and that the setfacl and getfacl binaries are installed.


---


#### synchronize
Uses rsync to make synchronizing file paths in your playbooks quick and easy.

  * Synopsis
  * Options
  * Examples

#### Synopsis
 This is a wrapper around rsync. Of course you could just use the command action to call rsync yourself, but you also have to add a fair number of boilerplate options and host facts. You still may need to call rsync directly via C(command) or C(shell) depending on your use case. The synchronize action is meant to do common things with C(rsync) easily. It does not provide access to the full power of rsync, but does make most invocations easier to follow.

#### Options

| Parameter     | required    | default  | choices    | comments |
| ------------- |-------------| ---------|----------- |--------- |
| dirs  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Transfer directories without recursing  |
| src  |   yes  |  | |  Path on the source machine that will be synchronized to the destination; The path can be absolute or relative.  |
| delete  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Delete files that don't exist (after transfer, not before) in the C(src) path. This option requires C(recursive=yes).  |
| group  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Preserve group  |
| existing_only  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Skip creating new files on receiver.  |
| links  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Copy symlinks as symlinks.  |
| copy_links  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Copy symlinks as the item that they point to (the referent) is copied, rather than the symlink.  |
| dest  |   yes  |  | |  Path on the destination machine that will be synchronized from the source; The path can be absolute or relative.  |
| recursive  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Recurse into directories.  |
| compress  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Compress file data during the transfer. In most cases, leave this enabled unless it causes problems.  |
| rsync_timeout  |   no  |  | |  Specify a --timeout for the rsync command in seconds.  |
| rsync_path  |   no  |  | |  Specify the rsync command to run on the remote machine. See C(--rsync-path) on the rsync man page.  |
| perms  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Preserve permissions.  |
| rsync_opts  |   no  |  | |  Specify additional rsync options by passing in an array.  |
| checksum  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Skip based on checksum, rather than mod-time & size; Note that that "archive" option is still enabled by default - the "checksum" option will not disable it.  |
| owner  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Preserve owner (super user only)  |
| set_remote_user  |   |  | |  put user@ for the remote paths. If you have a custom ssh config to define the remote user for a host that does not match the inventory user, you should set this parameter to "no".  |
| times  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Preserve modification times  |
| mode  |   no  |  | <ul> <li>push</li>  <li>pull</li> </ul> |  Specify the direction of the synchroniztion. In push mode the localhost or delegate is the source; In pull mode the remote host in context is the source.  |
| archive  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Mirrors the rsync archive flag, enables recursive, links, perms, times, owner, group flags and -D.  |
| dest_port  |   |  | |  Port number for ssh on the destination host. The ansible_ssh_port inventory var takes precedence over this value.  |

#### Examples
```
# Synchronization of src on the control machine to dest on the remote hosts
synchronize: src=some/relative/path dest=/some/absolute/path

# Synchronization without any --archive options enabled
synchronize: src=some/relative/path dest=/some/absolute/path archive=no

# Synchronization with --archive options enabled except for --recursive
synchronize: src=some/relative/path dest=/some/absolute/path recursive=no

# Synchronization with --archive options enabled except for --times, with --checksum option enabled
synchronize: src=some/relative/path dest=/some/absolute/path checksum=yes times=no

# Synchronization without --archive options enabled except use --links
synchronize: src=some/relative/path dest=/some/absolute/path archive=no links=yes

# Synchronization of two paths both on the control machine
local_action: synchronize src=some/relative/path dest=/some/absolute/path

# Synchronization of src on the inventory host to the dest on the localhost in
pull mode
synchronize: mode=pull src=some/relative/path dest=/some/absolute/path

# Synchronization of src on delegate host to dest on the current inventory host
synchronize: >
    src=some/relative/path dest=/some/absolute/path
    delegate_to: delegate.host

# Synchronize and delete files in dest on the remote host that are not found in src of localhost.
synchronize: src=some/relative/path dest=/some/absolute/path delete=yes

# Synchronize using an alternate rsync command
synchronize: src=some/relative/path dest=/some/absolute/path rsync_path="sudo rsync"

# Example .rsync-filter file in the source directory
- var       # exclude any path whose last part is 'var'
- /var      # exclude any path starting with 'var' starting at the source directory
+ /var/conf # include /var/conf even though it was previously excluded

# Synchronize passing in extra rsync options
synchronize: src=/tmp/helloworld dest=/var/www/helloword rsync_opts=--no-motd,--exclude=.git 

```

#### Notes
- Inspect the verbose output to validate the destination user/host/path are what was expected.

- The remote user for the dest path will always be the remote_user, not the sudo_user.

- Expect that dest=~/x will be ~<remote_user>/x even if using sudo.

- To exclude files and directories from being synchronized, you may add C(.rsync-filter) files to the source directory.


---


#### assemble
Assembles a configuration file from fragments

  * Synopsis
  * Options
  * Examples

#### Synopsis
 Assembles a configuration file from fragments. Often a particular program will take a single configuration file and does not support a C(conf.d) style structure where it is easy to build up the configuration from multiple sources. M(assemble) will take a directory of files that can be local or have already been transferred to the system, and concatenate them together to produce a destination file. Files are assembled in string sorting order. Puppet calls this idea I(fragments).

#### Options

| Parameter     | required    | default  | choices    | comments |
| ------------- |-------------| ---------|----------- |--------- |
| src  |   yes  |  | |  An already existing directory full of source files.  |
| remote_src  |   no  |  | <ul> <li>True</li>  <li>False</li> </ul> |  If False, it will search for src at originating/master machine, if True it will go to the remote/target machine for the src. Default is True.  |
| dest  |   yes  |  | |  A file to create using the concatenation of all of the source files.  |
| delimiter  |   no  |  | |  A delimiter to separate the file contents.  |
| others  |   no  |  | |  all arguments accepted by the M(file) module also work here  |
| regexp  |   no  |  | |  Assemble files only if C(regex) matches the filename. If not set, all files are assembled. All "\" (backslash) must be escaped as "\\" to comply yaml syntax. Uses Python regular expressions; see U(http://docs.python.org/2/library/re.html).  |
| backup  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Create a backup file (if C(yes)), including the timestamp information so you can get the original file back if you somehow clobbered it incorrectly.  |

#### Examples
```
# Example from Ansible Playbooks
- assemble: src=/etc/someapp/fragments dest=/etc/someapp/someapp.conf

# When a delimiter is specified, it will be inserted in between each fragment
- assemble: src=/etc/someapp/fragments dest=/etc/someapp/someapp.conf delimiter='### START FRAGMENT ###'

```


---


#### xattr
set/retrieve extended attributes

  * Synopsis
  * Options
  * Examples

#### Synopsis
 Manages filesystem user defined extended attributes, requires that they are enabled on the target filesystem and that the setfattr/getfattr utilities are present.

#### Options

| Parameter     | required    | default  | choices    | comments |
| ------------- |-------------| ---------|----------- |--------- |
| follow  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  if yes, dereferences symlinks and sets/gets attributes on symlink target, otherwise acts on symlink itself.  |
| state  |   no  |  | <ul> <li>read</li>  <li>present</li>  <li>all</li>  <li>keys</li>  <li>absent</li> </ul> |  defines which state you want to do. C(read) retrieves the current value for a C(key) (default) C(present) sets C(name) to C(value), default if value is set C(all) dumps all data C(keys) retrieves all keys C(absent) deletes the key  |
| value  |   no  |  | |  The value to set the named name/key to, it automatically sets the C(state) to 'set'  |
| name  |   yes  |  | |  The full path of the file/object to get the facts of  |
| key  |   no  |  | |  The name of a specific Extended attribute key to set/retrieve  |

#### Examples
```
# Obtain the extended attributes  of /etc/foo.conf
- xattr: name=/etc/foo.conf

# Sets the key 'foo' to value 'bar'
- xattr: path=/etc/foo.conf key=user.foo value=bar

# Removes the key 'foo'
- xattr: name=/etc/foo.conf key=user.foo state=absent

```


---


#### stat
retrieve file or file system status

  * Synopsis
  * Options
  * Examples

#### Synopsis
 Retrieves facts for a file similar to the linux/unix 'stat' command.

#### Options

| Parameter     | required    | default  | choices    | comments |
| ------------- |-------------| ---------|----------- |--------- |
| path  |   yes  |  | |  The full path of the file/object to get the facts of  |
| get_md5  |   no  |  | |  Whether to return the md5 sum of the file  |
| follow  |   no  |  | |  Whether to follow symlinks  |

#### Examples
```
# Obtain the stats of /etc/foo.conf, and check that the file still belongs
# to 'root'. Fail otherwise.
- stat: path=/etc/foo.conf
  register: st
- fail: msg="Whoops! file ownership has changed"
  when: st.stat.pw_name != 'root'

# Determine if a path exists and is a directory.  Note we need to test
# both that p.stat.isdir actually exists, and also that it's set to true.
- stat: path=/path/to/something
  register: p
- debug: msg="Path exists and is a directory"
  when: p.stat.isdir is defined and p.stat.isdir == true

# Don't do md5 checksum
- stat: path=/path/to/myhugefile get_md5=no

```


---


#### file
Sets attributes of files

  * Synopsis
  * Options
  * Examples

#### Synopsis
 Sets attributes of files, symlinks, and directories, or removes files/symlinks/directories. Many other modules support the same options as the M(file) module - including M(copy), M(template), and M(assemble).

#### Options

| Parameter     | required    | default  | choices    | comments |
| ------------- |-------------| ---------|----------- |--------- |
| src  |   no  |  | <ul></ul> |  path of the file to link to (applies only to C(state=link)). Will accept absolute, relative and nonexisting paths. Relative paths are not expanded.  |
| force  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  force the creation of the symlinks in two cases: the source file does not exist (but will appear later); the destination exists and is a file (so, we need to unlink the "path" file and create symlink to the "src" file in place of it).  |
| selevel  |   no  |  | <ul></ul> |  level part of the SELinux file context. This is the MLS/MCS attribute, sometimes known as the C(range). C(_default) feature works as for I(seuser).  |
| seuser  |   no  |  | <ul></ul> |  user part of SELinux file context. Will default to system policy, if applicable. If set to C(_default), it will use the C(user) portion of the policy if available  |
| recurse  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  recursively set the specified file attributes (applies only to state=directory)  |
| state  |   no  |  | <ul> <li>file</li>  <li>link</li>  <li>directory</li>  <li>hard</li>  <li>touch</li>  <li>absent</li> </ul> |  If C(directory), all immediate subdirectories will be created if they do not exist, since 1.7 they will be created with the supplied permissions. If C(file), the file will NOT be created if it does not exist, see the M(copy) or M(template) module if you want that behavior.  If C(link), the symbolic link will be created or changed. Use C(hard) for hardlinks. If C(absent), directories will be recursively deleted, and files or symlinks will be unlinked. If C(touch) (new in 1.4), an empty file will be created if the c(path) does not exist, while an existing file or directory will receive updated file access and modification times (similar to the way `touch` works from the command line).  |
| serole  |   no  |  | <ul></ul> |  role part of SELinux file context, C(_default) feature works as for I(seuser).  |
| follow  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  This flag indicates that filesystem links, if they exist, should be followed.  |
| mode  |   no  |  | <ul></ul> |  mode the file or directory should be, such as 0644 as would be fed to I(chmod). As of version 1.8, the mode may be specified as a symbolic mode (for example, C(u+rwx) or C(u=rw,g=r,o=r)).  |
| path  |   yes  |  | |  path to the file being managed.  Aliases: I(dest), I(name)  |
| owner  |   no  |  | <ul></ul> |  name of the user that should own the file/directory, as would be fed to I(chown)  |
| group  |   no  |  | <ul></ul> |  name of the group that should own the file/directory, as would be fed to I(chown)  |
| setype  |   no  |  | <ul></ul> |  type part of SELinux file context, C(_default) feature works as for I(seuser).  |

#### Examples
```
- file: path=/etc/foo.conf owner=foo group=foo mode=0644
- file: src=/file/to/link/to dest=/path/to/symlink owner=foo group=foo state=link
- file: src=/tmp/{{ item.path }} dest={{ item.dest }} state=link
  with_items:
    - { path: 'x', dest: 'y' }
    - { path: 'z', dest: 'k' }

```

#### Notes
- See also M(copy), M(template), M(assemble)


---


#### ini_file
Tweak settings in INI files

  * Synopsis
  * Options
  * Examples

#### Synopsis
 Manage (add, remove, change) individual settings in an INI-style file without having to manage the file as a whole with, say, M(template) or M(assemble). Adds missing sections if they don't exist.
 Comments are discarded when the source file is read, and therefore will not show up in the destination file.

#### Options

| Parameter     | required    | default  | choices    | comments |
| ------------- |-------------| ---------|----------- |--------- |
| option  |   no  |  | |  if set (required for changing a I(value)), this is the name of the option.  May be omitted if adding/removing a whole I(section).  |
| dest  |   yes  |  | |  Path to the INI-style file; this file is created if required  |
| section  |   yes  |  | |  Section name in INI file. This is added if C(state=present) automatically when a single value is being set.  |
| value  |   no  |  | |  the string value to be associated with an I(option). May be omitted when removing an I(option).  |
| others  |   no  |  | |  all arguments accepted by the M(file) module also work here  |
| backup  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Create a backup file including the timestamp information so you can get the original file back if you somehow clobbered it incorrectly.  |

#### Examples
```
# Ensure "fav=lemonade is in section "[drinks]" in specified file
- ini_file: dest=/etc/conf section=drinks option=fav value=lemonade mode=0600 backup=yes

- ini_file: dest=/etc/anotherconf
            section=drinks
            option=temperature
            value=cold
            backup=yes

```

#### Notes
- While it is possible to add an I(option) without specifying a I(value), this makes no sense.

- A section named C(default) cannot be added by the module, but if it exists, individual options within the section can be updated. (This is a limitation of Python's I(ConfigParser).) Either use M(template) to create a base INI file with a C([default]) section, or use M(lineinfile) to add the missing line.


---


#### fetch
Fetches a file from remote nodes

  * Synopsis
  * Options
  * Examples

#### Synopsis
 This module works like M(copy), but in reverse. It is used for fetching files from remote machines and storing them locally in a file tree, organized by hostname. Note that this module is written to transfer log files that might not be present, so a missing remote file won't be an error unless fail_on_missing is set to 'yes'.

#### Options

| Parameter     | required    | default  | choices    | comments |
| ------------- |-------------| ---------|----------- |--------- |
| dest  |   yes  |  | |  A directory to save the file into. For example, if the I(dest) directory is C(/backup) a I(src) file named C(/etc/profile) on host C(host.example.com), would be saved into C(/backup/host.example.com/etc/profile)  |
| src  |   yes  |  | |  The file on the remote system to fetch. This I(must) be a file, not a directory. Recursive fetching may be supported in a later release.  |
| validate_md5  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Verify that the source and destination md5sums match after the files are fetched.  |
| fail_on_missing  |   no  |  | <ul> <li>yes</li>  <li>no</li> </ul> |  Makes it fails when the source file is missing.  |
| flat  |   |  | |  A  l  l  o  w  s     y  o  u     t  o     o  v  e  r  r  i  d  e     t  h  e     d  e  f  a  u  l  t     b  e  h  a  v  i  o  r     o  f     p  r  e  p  e  n  d  i  n  g     h  o  s  t  n  a  m  e  /  p  a  t  h  /  t  o  /  f  i  l  e     t  o     t  h  e     d  e  s  t  i  n  a  t  i  o  n  .        I  f     d  e  s  t     e  n  d  s     w  i  t  h     '  /  '  ,     i  t     w  i  l  l     u  s  e     t  h  e     b  a  s  e  n  a  m  e     o  f     t  h  e     s  o  u  r  c  e     f  i  l  e  ,     s  i  m  i  l  a  r     t  o     t  h  e     c  o  p  y     m  o  d  u  l  e  .        O  b  v  i  o  u  s  l  y     t  h  i  s     i  s     o  n  l  y     h  a  n  d  y     i  f     t  h  e     f  i  l  e  n  a  m  e  s     a  r  e     u  n  i  q  u  e  .  |

#### Examples
```
# Store file into /tmp/fetched/host.example.com/tmp/somefile
- fetch: src=/tmp/somefile dest=/tmp/fetched

# Specifying a path directly
- fetch: src=/tmp/somefile dest=/tmp/prefix-{{ ansible_hostname }} flat=yes

# Specifying a destination path
- fetch: src=/tmp/uniquefile dest=/tmp/special/ flat=yes

# Storing in a path relative to the playbook
- fetch: src=/tmp/uniquefile dest=special/prefix-{{ ansible_hostname }} flat=yes

```


---


---
Created by Jason Edelman. February 2015.
