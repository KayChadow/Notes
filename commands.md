# Bash commands

Some useful commands when working with the linux terminal, and when doing CTF or hacking.

---
```sh
$0
```
- Name of current shell or script
```sh
$1
```
- $n is the n-th parameter
## A
```sh
alias sayhi="echo hi"
```
- Create shortcuts for (complex) commands
## B
```sh
binwalk
```
- Hiding files in files
## C
```sh
cat
```
- Print file contents
- Without arguments, it uses stdin. Can be used to have stdin as input after piping payload to executable like this: `(echo "payload"; cat;) | ./runme`
```sh
checksec
```
- Check the properties of a binary file
## D
```sh
df
```
- Disk free: disk space of file system
```sh
display
```
- Image viewer
## E
```sh
echo -e -n
```
- Echo text to terminal. Can be used with netcat (nc) command to send input
- e to allow `\` escapes
- n to not print trailing newline
```sh
evil-winrm
```
- Evil pentest version of Window remote management tool that uses WS-Management protocol.
```sh
exiftool
```
- Get file info and metadata
- Create polyglot (php/jpg) files with `exiftool -Comment="<?php payload; ?>" image.jpg -o polyglot.php`
```sh
expand -i -t 4 file1 > file1
```
- Will replace all leading tabs in the file with 4 spaces
```sh
expr
```
- Mathematics in the terminal, use space between operators and numbers!
## F
```sh
feh
```
- Image viewer
```sh
find . (options)
```
- Find files 
- Use -name or -size …(c) to specify
```sh
free
```
- Display amount of free memory
```sh
ftp
```
- FTP connection on port 20,21
- SFTP is secure version
## G
```sh
gdb
```
- Debugger
- Can be used to disassemble ELF
- When debugging pre-compiled ELF file, use: b *0xDEADBEEF, to set breakpoints on instructions
```sh
git
```
- GOATED
- Pull -> add -> commit -> push
```sh
gobuster
```
- Brute force for directories on web server for example
- gobuster dir -u <url\> -w </usr/share/wordlists/w.txt\>
```sh
grep -E
```
- Get matching characters on file or piped input
- Use `-E` for RegEx
- Use `|` as "or"
```sh
gzip -d
```
- Unzip a .gz file
## H
```sh
hexdump -C
```
- Get hex dump of file
```sh
hexedit
```
- Edit hex version of file
- Tab to edit ascii
## I
```sh
id
```
- View current UID and GID, and other related ID's or groups
```sh
impacket-<package>
```
- Collection of python3 classes focused on providing access to network packets
- Use the package `mssqlclient <user>:<pass>@<host> -windows-auth` to connect to Microsoft database with credentials
## J
```sh
john -w=/usr/share/wordlists/rockyou.txt <passhashfile>
```
- John the ripper, brute force hashes for example
## K
```sh
kill
```
- Kill a process by process id
## L
```sh
less
```
- Nice and handy file viewer
```sh
ln (target) (linkname)
```
- Used to create hard links
- Symbolic links with `-s`
```sh
locate
```
- Used to find files using database
- Database is automatically updated ~once a day
- Update database manually with `updatedb`
```sh
logger
```
- Write log to the /var/log/messages file
```sh
ltrace
```
- Used to see what library calls are made during binary execution
## M
```sh
mktemp
```
- Make temp file in /tmp/
- Use `-d` for temp directory
```sh
more
```
- Display file in terminal
```sh
mysql -h <host> -P <port> -u <user>
```
- Connect to some mysql version server
- mysql -h <host\> -P <port\> -u <user\>
- When in, do `show databases` and `show tables`
## N
```sh
nc <host> <port>
```
- TCP (or UDP) connection with host
- Pipe a file into it as input
- For payload input use `echo -e "$(<my_payload.txt)" | nc host port`
- Can be used to connect to a bindshell (shell in post hack)
```sh
nmap <ip>
```
- Network scanning and enumeration
- Use `-sV` for service version
- Use `-A` to enable some script scan
- Use `-v` for verbose output
- Use `-p` to clarify ports or "-" for all
## O
```sh
objcopy -I binary -O binary --reverse-bytes=n inputfile.bin outputfile.bin
```
- Swap endianness, "n" is 2 or 4 (word length or something)
```sh
objdump -d
```
- Get the disassembly code of any ELF program, use -M x86-64 to change assembly syntax
- Use `-h` to display the sections of the ELF file, and see where writable memory is (.bss)
- Use `-T` to display the dynamic symbol table of an executable or shared object (like libc.so.6), output format:
\<address_offset> \<linkage_type(?)> \<DefinedFunction (DF)> \<size> \<version> \<name>
- `-R` displays the dynamic relocation table of an ELF (Executable and Linkable Format) file
## P
```sh
pidof
```
- Get process id of a process
```sh
pngcheck
```
- Check if file is good png (-v) for verbose
```sh
ps
```
- Process status: display running processes in current shell
## R
```sh
redis-cli
```
- Connect to redis database on port 6379
```sh
rot13
```
- Alias of: `tr 'A-Za-z' 'N-ZA-Mn-za-m'`
- Added this alias myself, to do rot13 decrypt/encrypt in terminal easily
## S
```sh
sed -i “(line num (range) optional) s/findstr/replacestr/(g for all occurrences)” file.smth
```
- To find and replace and write to the file without need of any text editor programs
```sh
sha256sum
```
- Get the sha256 checksum to check the hash
```sh
smbclient 
```
- SMB connect on port 445
- Do `//<server>/<share>` to connect to a share
```sh
stat
```
- Get basic file info
```sh
strace
```
- A lot of info (idk what) about the running file or something
```sh
strings
```
- Returns all readable strings in a file
```sh
su
```
- Switch user
```sh
sudo
```
- Super User DO
- Do `-l` to view sudo permissions
## T
```sh
tar -xvf (file)
```
- Unpack a .tar file
```sh
telnet
```
- Telnet connection for simple communication, port 23
```sh
top
```
- See updating overview of all running processes
```sh
tr
```
- Translate files based on characters
```sh
tty
```
- Displays the file name of the terminal connected to the standard input, to identify the terminal you are currently using
- (Because cron jobs do not have a terminal)
## U
```sh
unzip
```
- Unzip a .zip file
## V
```sh
vim
```
- Very nice
- Press `:shell` to get shell
- Can set variables with `:set shell=/bin/sh`
## X
```sh
xxd
```
- Hex dump



## Template
```sh
command
```
- Context
