# __:simple-linux:{.lg .top} Linux__


In Unix and Unix-like operating systems, `/usr/bin` and `/usr/local/bin` are directories where executable binary files are stored. They are part of the system's PATH, which is a list of directories that the system searches when trying to find a command to execute. The difference between the two directories is based on the management and distribution scope of the software installed.

1. `/usr/bin`: This directory contains binary executables for all users that are managed by the system's package manager. The software in this directory is considered to be official, and is managed and updated by the system's package manager. It's also part of the Unix Filesystem Hierarchy Standard (FHS).

2. `/usr/local/bin`: This directory is intended for software that is local to the machine. This is where you should install software that you compile and install from source, or software not managed by the system's package manager. This allows you to keep locally installed software separate from the system software. It's also part of the FHS and is intended for use by the system administrator.

In general, when a user types a command, the system will search directories in the PATH in order and execute the first matching command it finds. Since `/usr/local/bin` is typically searched before `/usr/bin`, locally installed software will take precedence over system software.


## Common Comands

??? "[`telnet`](https://man7.org/linux/man-pages/man1/telnet.1.html)"

    ```{ .bash }
    Syntax:     telnet [options] [host [port]]

    # Basic example to connect to a host on a specified port:
    telnet example.com 80

    # Example for checking if a mail server is running on port 25:
    telnet mail.example.com 25

    # Example with a timeout option, Login Username:
    telnet -e q -l user 10.0.0.1 23
    ```

    - **`telnet`** is a network protocol used to connect to remote machines over TCP. It allows interaction with remote systems through command-line interfaces and is useful for testing connectivity or interacting with services running on specific ports.
    - **Host**: The remote server you want to connect to.
    - **Port**: The port number on which the service is running (e.g., HTTP on port 80, SMTP on port 25).
