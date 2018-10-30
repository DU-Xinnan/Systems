#Shell Basics

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


### Zip and tar
`unzip something.zip -d foler/`  
`tar -cf something.tar something`  
`tar -xf something.tar`

### file system
`df -h`: disk usage  
`du -h -d level directory`: directory

### Pipe and command sequence
`&&`: if previous command returns 0  
`||`: if previous command doesn't returns 0

**CUT**  
`cut file -d 'dimimilator' -f number,number`  
`cut file -c n-m`: every line n-m char

**sort**  
`cat /etc/passwd | sort -t':' -k 3 -n` -t: split using ‘：’， -k sorting using number k segment, -n sort by number

**uniq**  
`uniq -dc`: output all duplicate lines with duplicate numbers

###Redirect
`0` `/dev/stdin`: standard input  
`1` `/dev/stdout`: standard output  
`2` `/dev/stderr`: standard error  

`>`: write in  
`>>`: append  
`2>&1`:

`tee: echo 'sth' | tee file`: print and redirect  

**create file descriptor**  
`exec 3>somefile`  
`cd /dev/fd/; ls -Al` to check existing descriptor  
`exec 3>&-`: close the descriptor
`/dev/null` is null descriptor, `cat sth 1>/dev/null 2>&1` will output nothing

### Processes and thread
`pstree`: see process structure 
`cat /proc/cpuinfo`: detailed information about CPUop

**Top**  

|content|explaination|
|---|---|
|  Cpu(s) 1.0%us | user space processes CPU usage  |
|  1.0% sy | kernal space CPU usage  |
| % id  |  availiable CPU usage |
| % wa | percentage of time waiting for input/output |
| Mem total | total physical memory |
| Mem used | used memory |
| Mem buffer | kernal buffer |
| Swap total | Swap total |
| cached | cached total swap, memory content swapped to disk and later swapped back to memory, but that part of disk haven't been overwrite |   

<br/>

|column name| meaning |
|---|---|
|PR| (dynamic) priority, (0 - 139), 0 - 99 are reserved for real-time processes, 100-139 are for users|
|NI| nice value(-20 - 19), smaller the higher priority, also refer to as static priority|
|VIRT| virtual memory used|
|RES|physical memory used|
|SHR|shared memory|
|S|state of the process|
|TIME|amount of time the process is active|

Interact with TOP

|command|meaning|
|---|---|
|P|sort according to COU usage|
|M|sort by resident memory|

**ps**
`ps aux, ps axjf` show all information
`ps -afxo user, ppid, pid, pgid,command`: customized display

`renice  -n pid`： change priority for a user program, 0-19

