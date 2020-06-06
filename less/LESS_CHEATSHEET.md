# Less Cheatsheet

`less` is a command line tool that allows you to view (but not change) the contents of a file. `less` performs well when handling very large files.

### Table of Contents
**[1. Example Log File](#example-log-file)**<br>
**[2. Getting Help](#getting-help)**<br>
**[3. Opening a File](#opening-a-file)**<br>
**[4. Closing a File](#closing-a-file)**<br>
**[5. Navigating Around a File](#navigating-around-a-file)**<br>
**[6. View Live File Updates](#view-live-file-updates)**<br>
**[7. Searching for a Term](#searching-for-a-term)**<br>
**[8. Marking a Line](#marking-a-line)**<br>
**[9. Editing Files](#editing-files)**<br>
**[10. Examining Multiple Files](#examining-multiple-files)**<br>

## Example Log File

For this example, we need to play around with a very large file so that we can get used to navigating over tens of thousands of lines of log statements quickly. I am going to use a subset of an Apache web server log file from [here](http://www.almhuette-raith.at/apache-log/access.log) as my test log file (warning, the file is over 1GB in size).

You can find the log file here [/resources/access.log](./resources/access.log) which has been reduced in size to be only 50 thousand lines long and 11MB in size.

If you would like to download and reduce the size of the log file yourself, follow these instructions:
```shell
# Pipe the curl of the log file into head and take the first 50000 lines before writing to access.log
curl http://www.almhuette-raith.at/apache-log/access.log | head -n 50000 > access.log
# Verify access.log has 50,000 lines
wc -l access.log
```

## Getting Help

If you need some help or a reminder of possible commands and arguments, use `less -?` or `less --help` to see a summary. You can also see the `less` man-pages [here](https://www.man7.org/linux/man-pages/man1/less.1.html).

Once you are in `less`, you can use `h` or `H` to see the summary of `less` commands.

## Opening a File

The simplest way to open a file using `less` is to run `less ./resources/access.log` however we can also pipe data into `less`. For example: `ps | less` will pipe the result of `ps` which are details of running processes into `less`.

You can use the `-N` argument when invoking `less` to see line numbers, e.g. `less -N ./resources/access.log`.

To see the useful metadata about the file which is currently open in `less` such as the filename, which line numbers are on screen and the number of bytes through the file you are, use `Ctrl + G` which will give you a result like below:

```shell
less ./resources/access.log
Ctrl + G
> access.log lines 1-32/50000 byte 8083/11749760 0%  (press RETURN)
```

## Closing a File

If you have opened a file with `less`, close it by pressing one of (`q`, `:q`, `Q`, `:Q`, `ZZ`). If you are watching live logs, you may see the following `Waiting for data... (interrupt to abort)` which means you can interrupt using `Ctrl + C` and then quit less using `q`.

There are some other ways of exiting `less`:

|          Key Command         |                                 Action                                |
|:----------------------------:|:---------------------------------------------------------------------:|
|      -e or --quit-at-eof     |  Less will automatically exit the second time it reaches end of file  |
|      -E or --QUIT-AT-EOF     |   Less will automatically exit the first time it reaches end of file  |
|  -F or --quit-if-one-screen  |    Less will automatically exit if entire file fits on first screen   |


## Navigating Around a File

Use the table below to help you navigate the `access.log` file.

|          Key Command          |                          Action                           |
|:-----------------------------:|:---------------------------------------------------------:|
|   Down Arrow, enter, e or j   |                     Move down one line                    |
|        Up Arrow, y or k       |                      Move up one line                     |
|      Space bar or Pg Down     |                     Move down one page                    |
|           b or Pg Up          |                      Move up one page                     |
|             d or u            |             Forward or backward half a window             |
|      Left or Right Arrow      |                    Scroll horizontally                    |
|           g or Home           |                    Go to the first line                   |
|            G or End           |                      Go to last line                      |
|              10g              |                    Go to the 10th line                    |
|           10j or 10k          |                Go 10 lines forward or back                |
|           50p or 50%          |       Go to percentage position, 50 would be halfway      |

Firstly, open the [/resources/access.log](./resources/access.log) file using `less`.

```shell
# By default, the start of the file is shown
less ./resources/access.log
# Go to the end of the file
G
# Go up a page
Pg Up
# Go backwards half a window
u
# Go back to the top of the file
g
# Move 1000 lines forward
1000j
# Use right arrow key to see the end of long lines
->
```

## View Live File Updates

If `access.log` was an actual log file on an Apache server logging incoming requests, seeing those incoming requests create new logs at the end of the log file would be very useful. This can be done with the following command:

```shell
less ./resources/access.log
Ctrl + F
# You will see this message: Waiting for data... (interrupt to abort)
# Screen will now be following the bottom of the log file with new logs being shown
Waiting for data... (interrupt to abort)
# Use Ctrl + C to abort the live data
Ctrl + C
```

## Searching for a Term

One of the most useful features of `less` is the ability to do advanced searches on the file very quickly. The administrator of the Apache server may want to quickly check for any errors in the logs, which they could easily do using the following:

```shell
less ./resources/access.log
# Pressing enter after the below will show and highlight the next occurrence of 'error'
/error
# Press n to go to the next occurrence
n
# Press N to go to the previous occurrence
N
# Using ? instead of / will take you to the previous occurrence
?error
```

If you have multiple files open in `less` (see below for details) you can search for a term until a match is found, moving onto the next file if no match is found in the first.

```shell
less ./resources/access.log
# Open another log file
:e
Examine: ./resources/access2.log
# Go to the first file
:p
# To search until match is found on multiple files, use /* which will show the EOF-ignore prompt
/*
EOF-ignore/SearchTerm
```

### Searching for a Term using a Regex

A REGEX will work in the regular search term input.

```shell
less ./resources/access.log
# Basic REGEX for an IP Address
/[0-9]+(\.[0-9]+)(\.[0-9]+)(\.[0-9]+)
```

### Ignoring Lines Containing a Term

It might be useful to search for lines which do not include a specific pattern. The administrator might be interested to know how many users access the website on a device that does not run Android. The Apache logs contain the browser and operating system for each request so a search would involve looking for lines that do not include Android, this can be done in the following way:

```shell
less ./resources/access.log
# Typing in ! will cause the Non-match message to display
/!Android
Non-match /Android
# Go to next line that does not include Android
n
```

### Only See Lines Which Include Search Term

This is very different to searching using `/searchTerm` as in this scenario, we do not want to see any lines that do not include the particular term. This could be useful if there was an error with a POST request and so we want to ignore all GET requests. To do this, do the following:

```shell
less ./resources/access.log
&POST
# An & will appear below, indicating lines are being ignored, can continue searching normally for the error
& :/error
```

## Marking a Line

If a line is of particular interest and you may wish to come back to it later, you can mark it by doing the following:

```shell
less ./resources/access.log
/media/error
# Typing just m will bring up the mark prompt which accepts one letter
m
mark: E
# The line matching media/error is now marked with the letter E
/python-requests
m
mark: P
# The line matching python-requests is marked with P
# Use an apostrophe to go to a specific mark
'
goto mark: P
'
goto mark: E
```

## Editing Files

You cannot directly edit files using `less`, however pressing `v` whilst looking at a file in `less` will open the file in the configured editor and then return to `less` once the text editor has been exited.

## Examining Multiple Files

`less` allows you to have multiple files open that you can switch between quite easily.

|  Key Command  |                      Action                      |
|:-------------:|:------------------------------------------------:|
|       :e      |  Examine the new file from user input file name  |
|       :n      |               Examine the next file              |
|       :p      |             Examine the previous file            |
|       :x      |      Examine the first file in list of files     |
|       :d      |      Remove current file from list of files      |

Firstly, make sure you have more log files to examine:

```shell
# Add two more log files
cp ./resources/access.log ./resources/access2.log
cp ./resources/access.log ./resources/access3.log
less ./resources/access.log
# After typing in :e the Examine prompt will show
:e
Examine: ./resources/access2.log
# This message details the current file and number of opened files
# ./resources/newAccess.log (file 2 of 2)
:e
Examine: ./resources/access3.log
# ./resources/access3.log (file 3 of 3)
# Examine the first
:x
# ./resources/access.log (file 1 of 3)
:n
# ./resources/access2.log (file 2 of 3)
# Remove the 2nd file (access2.log) from he list of files
:d
# ./resources/access.log (file 1 of 2)
```
