# Vim Cheat sheet
###Moving Cursor  
`h` left  
`l` right  
`j` down  
`k` up  
`w` next word  
`b`  previous word  

### Deleting in Vim
`dd` delete entire row  
`D` delete to row end  
`d^` delete to row start  
`dG` delete to the end of file  
`d1G` delete to start of the file  
`u` undo  
`Ctrl + r` redo

### Quit Vim
`w <filepath>` save as  
`:x` save and exit  
`Shift + z` save and exit

### Navigating cursor in vim
`.` repeat previous operation  
`n Shift g` go to line n, use `:set nu` to show line number  
`gg` go to first line  
`G` go to last line  
`0` go to the beginning of the line 
`$` go to the end of the line

### Copy paste and cut
`y` is use for copy, `yy` copy whole line, `y0` to the beginning of the line. `y$` to the end of the line  
`p` for paste

### Indentation
`>>` indent to right  
`<<` indent to left

### Searching
`/` search downwards  
`?` search upwards

### Selecting in vim
`v` select char by char  
`V` select line  
then `d` to delete and `y` to copy
