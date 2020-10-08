# Mininet Walkthrough
## Part 1: Everyday Mininet Usage
First, a (perhaps obvious) note on command syntax for this walkthrough:

$ preceeds Linux commands that should be typed at the shell prompt
mininet> preceeds Mininet commands that should be typed at Mininet’s CLI,
#preceeds Linux commands that are typed at a root shell prompt
In each case, you should only type the command to the right of the prompt (and then press return, of course!)

### Display Startup Options
Let’s get started with Mininet’s startup options.
Type the following command to display a help message describing Mininet’s startup options:
```bash
farhan@Arctic:~$ sudo mn -h
```
This walkthrough will cover typical usage of the majority of options listed.

### Start Wireshark
To view control traffic using the OpenFlow Wireshark dissector, first open wireshark in the background:
