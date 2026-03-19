student.bash_env
=================

An Ansible role that configures default initialization files for the Bash shell used for newly created users. This role creates skeleton files in `/etc/skel/` that are copied to new user home directories, including `.bashrc`, `.bash_profile`, and `.vimrc` configuration files.

Requirements
------------

- Ansible 2.9 or higher
- Target hosts must be running a Linux distribution with Bash shell

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

### `default_prompt` (default: `'[student prompt \W]\$ '`)

The Bash prompt string to use for new users. This variable accepts standard Bash prompt escape sequences such as:
- `\u` - username
- `\h` - hostname
- `\W` - basename of current working directory
- `\$` - displays `$` for regular users or `#` for root

Example values:
```yaml
default_prompt: '[student prompt \W]\$ '
default_prompt: '[\u on \h in \W dir]\$ '
default_prompt: '[\u@\h \W]\$ '
```

### `prompt_color` (default: `normal`)

The color to use for the Bash prompt. Valid values are:
- `red` - Red colored prompt
- `yellow` - Yellow colored prompt
- `blue` - Blue colored prompt
- `normal` - No color (default)

Example:
```yaml
prompt_color: blue
```

### `skel_dir` (default: `'/etc/skel/'`)

The directory where skeleton files are placed. This is typically `/etc/skel/` on Linux systems. Files created in this directory are copied to new user home directories when accounts are created.

### `bash_user` (optional, default: `root`)

The user owner for the skeleton files. If not specified, defaults to `root`.

### `bash_group` (optional, default: `root`)

The group owner for the skeleton files. If not specified, defaults to `root`.

Dependencies
------------

None.

Example Playbook
----------------

### Basic usage

```yaml
---
- name: Configure bash environment for new users
  hosts: all
  become: true
  roles:
    - student.bash_env
```

### Custom prompt and color

```yaml
---
- name: Configure bash environment with custom prompt
  hosts: all
  become: true
  vars:
    default_prompt: '[\u on \h in \W dir]\$ '
    prompt_color: blue
  roles:
    - student.bash_env
```

### Using in a playbook with user creation

```yaml
---
- name: Use student.bash_env role playbook
  hosts: webservers
  become: true
  vars:
    default_prompt: '[\u on \h in \W dir]\$ '
    prompt_color: normal
  pre_tasks:
    - name: Ensure test user does not exist
      ansible.builtin.user:
        name: student2
        state: absent
        force: true
        remove: true

  roles:
    - student.bash_env

  post_tasks:
    - name: Create the test user
      ansible.builtin.user:
        name: student2
        state: present
        password: "{{ 'redhat' | password_hash }}"
```

What This Role Does
-------------------

This role creates the following files in `/etc/skel/`:

1. **`.bashrc`** - Contains:
   - Source of global bashrc definitions
   - Color definitions (RED, BLUE, YELLOW, NORMAL)
   - PS1 prompt configuration using `default_prompt` and `prompt_color` variables

2. **`.bash_profile`** - Contains:
   - Sources `~/.bashrc`
   - Sets up PATH to include `$HOME/.local/bin` and `$HOME/bin`

3. **`.vimrc`** - Contains:
   - Basic vim configuration (cursorline and cursorcolumn settings)

When new user accounts are created on the system, these skeleton files are automatically copied to the new user's home directory, providing a consistent Bash environment configuration.

License
-------

BSD

Author Information
------------------

This role is used for Red Hat training purposes.
