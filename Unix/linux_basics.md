#Linux Basics

### Env Variables
create variable `tmp`:  

    $ declare tmp
    tmp = duxinn
    echo $tmp

`temp` is only valid in current cell.

**three command that are related to Env Variable**:
`set` > `env` > 	`export`  
only those are valid in child process are Env Variable	
`/etc/bashrc` stores variable and `/etc/profile` stores environment variable  
we can use export to create a temporary variable  

### Searching in shell
- `where is` only searches binary file (-b), help file (-m), source codes (-s)
- `locate` searches in “ /var/lib/mlocate/mlocate.db ”, use `updatedb` to force update.
- `which` searches in PATH
- `find` is the most powerful one `sudo find /etc/ -name interfaces`, the format is find [path] [option] [action], `-atime` last visit time, `-ctime` last modify, `mtime` last modify property time. `-newer file` will list all the files that are newer than it. `-mtime n` means file modified between -n-1 to -n, `-mtime +n` means file modified before n days. `mtime -n` list file modified within n days.