# Linux Basics

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

##shell script
command inside `` will has higher priority to be executed  

**()**  

 code inside `()` will executed as child process, parent process is not able to access variables in side `()`
  `()` can also be used to init array `arr=(1,2,3,4,5); echo ${arr[1]}`

**variable**
`readonly varname` will define a const
`$0 $1...${10}` represent parameters pased in

**operator**  

number operator  

|operator|meaning|
|---|---|
|-eq|euqal|
|-ne| not equal|
|-gt|greater than|
|-lt|less than|
|-ge|greater or equal|
|-le|less or equal|
|||

example:

	a=10
	b=20

	if [ $a -eq $b ]
	then
	   echo "$a -eq $b : a == b"
	else
	   echo "$a -eq $b: a != b"
	fi
	
	if [[ $a -lt 100 && $b -gt 100 ]]
	then
	   echo "return true"
	else
	   echo "return false"
	fi



String Operator   

|operator|meaning|
|---|--|
|=|equal|
|!=|not equal|
|-z|true if length is 0|
|-n|true if length > 0|
|str|true if not empty|

file operator   

|operator|meaning|
|---|---|
|-e|file exist|
|-f|is a file, not directory or system file|
|-s|file size is not 0|
|-d|is a directory|
|-b| is a device|
|-p|is a pipe|
|-r|has read permission or not|
|-w|has write permission|
|-x|is executable or not|
|f1 -nt f2|f1 newer than f2|
|f1 -ot f2|f1 older than f2|
|f1 -ef f2|f1 and f2 is the same hard link|

**for loop**
	  
	for var in item1 item2 ... itemN
	do
	    command1
	    command2
	    ...
	    commandN
	done
	
**while loop**

	int=1
	while(( $int<=5 ))
	do
	    echo $int
	    let "int++"
	done

**case**  

	echo 'Enter a number between 1 and 4:'
	echo 'The number you entered is:'
	read aNum
	case $aNum in
	    1)  echo 'You have chosen 1'
	    ;;
	    2)  echo 'You have chosen 2'
	    ;;
	    3)  echo 'You have chosen 3'
	    ;;
	    4)  echo 'You have chosen 4'
	    ;;
	    *)  echo 'You did not enter a number between 1 and 4'
	    ;;
	esac

**function**  

	#!/bin/bash
	funWithParam(){
	    echo "The first parameter is $1 !"
	    echo "The second parameter is $2 !"
	    echo "The tenth parameter is $10 !"
	    echo "The tenth parameter is ${10} !"
	    echo "The eleventh parameter is ${11} !"
	    echo "The total number of parameters is $# !"
	    echo "Outputs all parameters as a string $* !"
	}
	# call the function
	funWithParam 1 2 3 4 5 6 7 8 9 34 73

