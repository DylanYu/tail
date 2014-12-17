# File orgnization
src - source code
tail.jar - executable file
README.md - readme file
testFile - test data

# Usage

This project have used some libs only supported in **Java 7**, so please make sure you have **JDK 1.7** or higher installed

The program is tested in **Windows** platform and performed well. 
Some inconsistencies may occur in Linux and Mac because underline file system implementation is different.

This program is still fast when the file is large because we read from tail.

## Execute jar file

We provide -n and -f options to run the program, value for -n could be negative (default) or positive. 
You can execute like:

java -jar tail.jar -n 10 -f testFile

java -jar tail.jar -n -5 testFile

java -jar tail.jar testFile

With -f option, any appended data will be printed out to console.

You can open a text file editor like Notepad++ to add some lines to
the file and watch the changes in console.

If -n option is added, we assume the value for -n is 10 (or -10)

If you want to end the monitor, press CTRL + C.

The number for -n option could be *positive* like

java -jar tail.jar -n +10 -f testFile

You will get date from line 10 of the file and continue to monitor changes.

## Run as an Eclipse project

Import (or copy) all the source code into an Eclipse project and run from Cmd.java, 
before configuring the *Program arguments* in Run Configuration like "-n 10 -f testFile". 
Make sure you have a file named "testFile" located in you project folder.

# Design consideration

## Input file size can vary from few MBs to tens of GB

We seek to tail of the file and reverse scan the file so large 
file will not slow our program as we only read what we need.

## Two cases for value of N: A) N is small to fit in memory, B) N is large to fit in memory

When N is small when reverse scan the file we can store what is read and
print after finished reading. 

But when N is large we cannot store all the data we read, so when we just seek back to the 
right start position and not store anything, then start from the start position we read forward 
again to tail and print anything we meet. This just doubles the running time, which is acceptable.

# Implementation detail

## Overall

We first print the tail or head of the file according to user's input, if the 
user want to monitor the appended data, we register a watch service to keep an
eye on the file and print anything appended.

## Print the tail or head of the file

When the value for -n option is positive, we just skip the first (n-1) lines and
print anything left in the file.

When the value for -n option is negative, there're two ways to find all tail lines
and print. 

A) keep the latest N lines while reading forward the data

B) start from the end and go backwards until we read the Nth line

I chosen B. For most cases N is much smaller than overall lines so read all the data is a waste 
of time, which makes B a good choice.


## Monitor appended data in file

Use Java RandomAccessFile to seek to the end of file, and use Java WatchService to monitor
the file. When the file is modified, we move forward the point until we meet the new end of file.