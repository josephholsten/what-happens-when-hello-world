# What happens when you write a hello world program?

When I first learned to program, it drove me crazy that so much was
going on that was totally unexplained. It's twenty years later, I may as
well write down how it actually works.

We're going to to write and run a program. It's probably the most famous
source code in history, and it's originally from Brian Kernighan,
[Programming in C: A Tutorial]

This is the program:

    main( ) {
        printf("hello, world");
    }

Here is how to program it and run it.

## 1. Create a directory `helloworld`

    mkdir helloworld\n

This is handled by the current shell, and probably readline. something
about gets() into a buffer, which is then passed to a fork-exec to run
the `*char` `["mkdir","helloworld"]`.

the kernel runs the fork, and the parent process passes the stdin,
stdout, and stderr fds. it then waits for the child to exit.

this execution code is defined in the `bash` source. the implementation
of this is in the `bash` shell, in particular apple's fork of the
[`bash` source code]. It quite a few bits to handle this.

in the the child process, the kernel immediately runs `execlp()` (?)
with the `execlp("mkdir","helloworld")`

the impl of `execlp()` is in mac whose function header is [Libc
`include/unistd.h:445`] and whose impl is [Libc
`gen/FreeBSD/exec.c:112`]

it finds the program `mkdir` in `/bin/mkdir`

the source of this program can be find in opensource.apple.com's sources
[`file_cmds` `mkdir/mkdir.c`].

it starts with a `main(argc="mkdir",argv=["helloworld"])`.

it will make an operating system call `mkdir(2)` with
`mkdir("helloworld",S_IRWXU | S_IRWXG | S_IRWXO]` to create the
directory.

The source of `mkdir(2)` is implemented in [`xnu`
`bsd/vfs/vfs_syscalls.c:7282`].

`mkdir1at` is where it gets interesting, and I stop understand at all.

virtual filesystem creates a new file and a link to it in the current
directory

the physical filesystem translates this into the exact addresses that
need to change in memory

the kernel will modify the disk

it will communicate with the disk with a disk driver it will communicate
over some bus plugged into the bus will be a disk controller which
translates it for the internals of a SSD on the disk, the changes look
like ...

Then the mkdir program will exit

And the shell's parent process will acknowledge the exit and return to
waiting for keyboard input

## 2. Change directory to `helloworld/`

    cd helloworld\n

This updates the environment variable `PWD` of the current shell

The implementation of this is in the [`bash` `bash-3.2/builtins/cd.c`]

## 3. Edit the file main.c

    vi main.c\n

This `execlp()`s the program `vi`, which expands to `/usr/bin/vi`

This program is compiled from apple's fork of [`vim` source code]

`vim` does quite a bit but ultimately sets things up in `main()`,
handles `argv` of `["main.c"]`

it then presents a buffer waiting for user input

## 4. Enter the program into the editor

    i\n
    main( ) {\n
        printf("hello, world");\n
    } <esc>

this executes the `i` command, switching mode to insert. the keypresses
then are handled and both the internal buffer and the visual
representation is updated. When `<esc>` is pressed, `vi` returns to
visual mode.

## 5. Save the file to `main.c`, and exit the editor

    :x\n

vi handles the `:` keypress and switches to command mode, taking the
command `x`, and signaling completion with `\n`.

vi identifies this as an `:xit` command.

this causes it to `open()` and write the file `main.c`

the kernel accepts this write

virtual filesystem creates a new file and a link to it in the current
directory

the physical filesystem translates this into the exact addresses that
need to change in memory

see above for how messages are sent to the hard disk

## 6. Compile into the program `a.out`

    cc main.c\n

This `execlp()`s the program `cc`, which expands to `/usr/bin/cc`

the source of this is probably something related to clang

it'll `open()` the `main.c` file, parse it into assembler and assemble
into an executable file and write to `a.out`

## 7. Run the program

    ./a.out

This `execlp()`s the program `./a.out`.

this just runs the program from above

the program calls the printf()

## 8. View the output of the program

## How you are entering keys into the computer?

see alex/what-happens-when

    there is a keyboard
    this keyboard is recognizing signals from switches under each key
    it sends these signals accross to the computer over bluetooth
    the computer has a bluetooth device
    which sends interrupts to the kernel
    which handles these messages by a keyboard driver
    which passes to a windowing system?
    which passes to an app
    which happens to be a virtual terminal
    which responds to connections by creating a virtual pty
    which sets up the terminal
    which responds to inputs
    which starts a shell
    which may directly handle your commands
    or may fork-exec into a new process
    which may exit to return to the shell

## How are you seeing this?

in addition to attaching the keyboard interrupts to the terminal
application the terminal application can also display its state

Because `Terminal.app` is closed source, we will be using `iTerm 2` as a
reference implementation.

## How might one maintain this program?

GNU Hello provides an example of what useful information should be
available along with program source code.

It uses tools like

-   make
-   automake
-   m4
-   autoconf
-   perl
-   gnulib
-   gettext
-   getopt\_long
-   help2man
-   texinfo

https://www.gnu.org/software/hello://www.gnu.org/software/hello/

https://github.com/aosm/xnu

  [Programming in C: A Tutorial]: http://cm.bell-labs.com/cm/cs/who/dmr/ctut.pdf
  [`bash` source code]: https://github.com/aosm/bash
  [Libc `include/unistd.h:445`]: https://github.com/aosm/Libc/blob/master/include/unistd.h#L445
  [Libc `gen/FreeBSD/exec.c:112`]: https://github.com/aosm/Libc/blob/master/gen/FreeBSD/exec.c#L112
  [`file_cmds` `mkdir/mkdir.c`]: https://github.com/aosm/file_cmds/blob/master/mkdir/mkdir.c
  [`xnu` `bsd/vfs/vfs_syscalls.c:7282`]: https://github.com/aosm/xnu/blob/master/bsd/vfs/vfs_syscalls.c#L7282
  [`bash` `bash-3.2/builtins/cd.c`]: https://github.com/aosm/bash/blob/master/bash-3.2/builtins/cd.c
  [`vim` source code]: https://github.com/aosm/vim
