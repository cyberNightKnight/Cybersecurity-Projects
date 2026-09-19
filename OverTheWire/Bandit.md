# OverTheWire: Bandit

Bandit is an OverTheWire lab that focuses on the shell.

I used a Kali Linux VM to solve all the challenges.

## Level 0

I connected to the lab using this command:

```$ ssh bandit0@bandit.labs.overthewire.org -p 2220``` 

ssh is used for remote connections. Here I'm connecting to bandit.labs.overthewire.org using port 2220 and with username bandit0.

(picture)

According to the instructions, the password is bandit0.

After entering the password, we're ready to go to the next Level.

(picture)

## Level 0 -> Level 1

Next level's password is stored in a file called readme inside the home directory.

In order to visualize current directory's contents, we can use  ```ls ```:

(picture)

This displays the readme file. To see what's inside it, we can use  ```cat```, which is used, among other things, to display file contents:
```$ cat readme```

This outputs the password.

To access the next level, we first exit and then run the same command as we did at Level 0, but using bandit1 with the new password:
```$ exit```
```$ ssh bandit1@bandit.labs.overthewire.org -p 2220```

If we wish to empty the terminal, we can use the command ```clear```.

## Level 1 -> Level 2

To find the file located in "-":

We make sure it's in the current working directory.
```$ ls```

We visualize its contents, which in this case is the password.
```$ cat ./-```

In Linux, dashes (-) are used to specify options for commands. If we were to use the dash alone with ```cat```, the command would remain executing.

```./``` is the file's path, where ```.``` is the current directory, so here we're declaring that we're dealing with a file.

## Level 2 -> Level 3

Now the password is stored in a file called --spaces in this filename--.

Similar to the single dash, double hyphens are also used to specify options. Particularly, double hyphens are often used as the longer written form of the single dash one.

This time we also have to deal with spaces; typically the command line will take spaces as different arguments. In order to counter this, we surround the filename in quotes.
```$ cat ./"--spaces in this filename--"```

This gives us this level's password.

## Level 3 -> Level 4

First we visualize the current directory:
```$ ls```

Here, we have two choices: 

1. We go to the ```inhere``` directory using ```cd```:

```cd inhere```

In this level, the password is stored in a hidden file. To show its name, we use the ```-a``` (all) flag after ```ls```.

```$ ls -a```

There, we find a file named ```...Hiding-From-You```.

Finally, we display the contents using ```cat```:
```$ cat ...Hiding-From-You```

2. We can print the file's contents without changing directories using paths:

We visualize ```inhere```'s contents:
```$ ls -a inhere```

Then we output the desired file's contents:
```cat inhere/...Hiding-From-You```

## Level 4 -> Level 5

Next level's password is stored in the only human-readable file in inhere.

```cd inhere```
```find . -type f -printf "\n\n" -exec cat {} \;```

## Level 5 -> Level 6

Password is stored in the ```inhere``` directory, is human-readable, is 1033 bytes, and isn't executable

```find . -type f ! -executable -size 1033c -printf "\n\n" -exec cat {} \;```

## Level 6 -> Level 7

File is stored somewhere owned by user bandit7, owned by group bandit6, 33 bytes in size

```find / -type f -size 33c -group bandit6 -user bandit7 -printf "\n\n" -exec cat {} \;```


## Level 7 -> Level 8
Password is found in data.txt next to the word "millionth"

```grep data.txt -e "millionth"```

## Level 8 -> Level 9
The password is in data.txt and is the only unique line of text

```sort data.txt | uniq -u```

## Level 9 -> Level 10
Password is in data.txt, in a human-readable string, preceded by several "="

```strings data.txt | grep -e '.=='```

## Level 10 -> Level 11
Next level's password is stored in data.txt, which contains base64 data.

```base64 -d data.txt```

## Level 11 -> Level 12
The password is stored in data.txtm where lowercase and uppercase were rotated by 13 positions

```cat data.txt | tr 'a-zA-Z' 'n-za-mN-ZA-M'```
## Level 12 -> Level 13
data.txt is now a hexdump that has been compressed several times. Creating a directory is useful

```xxd -r data.txt data.bin```
To turn the hexdump into binary

Analyze the file 
```file data.bin``` 
(picture)
```mv data.bin data.gz```
```file data```
data: bzip2 compressed data, block size = 900k

```mv data data.bz2```
```bunzip2 data.bz2```
```file data```
data: gzip compressed data, was "data4.bin", last modified: Wed Jun 24 14:58:46 2026, max compression, from Unix, original size modulo 2^32 20480

```mv data data.gz```
```unzip data.gz```
```file data```
data: POSIX tar archive (GNU)

```man tar```
```mv data data.tar```
```tar -xvf data.tar```
```file data5.bin```
data5.bin: POSIX tar archive (GNU)

```tar -xvf data5.bin```
```file data6.bin```
data6.bin: bzip2 compressed data, block size = 900k

```bunzip2 data6.bin```
```file data6.bin.out```
data6.bin.out: POSIX tar archive (GNU)

```tar -xvf data6.bin.out```
```file data8.bin```
data8.bin: gzip compressed data, was "data9.bin", last modified: Wed Jun 24 14:58:46 2026, max compression, from Unix, original size modulo 2^32 49

```gunzip data8.bin```
```mv data8.bin data8.gz```
```gunzip data8.gz```
```file data8```
data8: ASCII text

```cat data8```
The password

## Level 13 -> Level 14
Next level's password is in /etc/bandit_pass/bandit14 and can only be read by user bandit14


```
