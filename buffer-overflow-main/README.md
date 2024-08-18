In this assignment, you will explore buffer overflows. Make sure
that you test everything using the official course docker image.

## Security Mechanisms

 * The C compiler uses a security mechanism called "Stack Guard",
   which detects and prevents buffer overflows. We have provided
   you with a Makefile that compiles your code with this disabled
   by default.

 * Additionally, the operating system performs address space
   randomization (ASLR), which does not prevent buffer overflows, but
   makes it much harder for them to result in a successful
   exploit. You will need to work around this for this project!  As a
   hint, ASLR will move things around in blocks, so stack and heap
   offsets will differ between runs, but the fundamental structure
   will not be changed, so if you know the distance (in bytes) between
   two values on the stack, that distance will remain constant from
   one run to another.

## Exploiting a Vulnerable Program

We have provided you with a vulnerable program in `stack.c` -- you are
not to modify this. The Makefile will compile this as needed to allow
us to exploit it, including making it setuid-root. To compile, you
should run the `build.sh` script, which will do the compilation in a
container running the `baseline` image.

This program has a function `bof()`, which includes a buffer-overflow
vulnerability. The program reads from the network, does some useless
work with the provided data, and echoes back what you sent it. It
echoes the received message using the `sprintf(s,fmt,...)` function,
which behaves like `printf(fmt,...)`, except that the output is
written to the string `s` rather than standard output. This is a great
way to create formatted strings in C. You will be attacking it over
the network.

We have provided you with an initial skeleton for `exploit_1.c`.  The
term shellcode literally refers to code that starts a command shell,
like bash. However, nowadays shellcode is considered to be any byte
code that can be inserted into an exploit to accomplish a particular
objective. The shellcode provided in `exploit_1.c` is designed to
execute the command `cat /var/secret/token`. When run normally, this
command results in permission denied since `/var/secret` is accessible
only to root. You will attempt to execute this command by exploiting
the buffer overflow vulnerability in `stack.c`

As a tip, you *will* need to send more than one message to the `stack`
program in order for your attack to succeed. Take a careful look at
the function `bof()` in `stack.c`, and look for the potentially
exploitable bugs.

### What to do:

Write a program named `exploit_1.c` that creates and sends a buffer
overflow to `stack.c`. The message you send should include:

 - shellcode
 - target address in the stack to which control should go
   when bof() returns
 - NOP instructions to increase the chance of success

`exploit_1` takes no command line arguments. If your exploit is
implemented correctly, then when `bof()` returns it will execute your
shellcode, printing the contents of `/var/secret/token`.

Running the code in a container is a bit more complex than running it
locally. To help with this, we have provided the following scripts:

 * `run_stack.sh` starts a container with `stack` running
 * `run_exploit.sh` runs `exploit_1` in the same container
 * `run_baseline.sh` starts an essentially idle container

You can also start an interactive container, so that you can run the
programs multiple times without restarts, as well as using `gdb`. (You
can use `run_baseline.sh` to accomplish this, too.) To do this, run:

    docker run --name stack -ti --rm -v "$(pwd):/opt" baseline

Once you have a container running, you can start interactive shells in
it (from other host terminals) with:

    docker exec -ti stack bash

This runs the program `bash` (that is, the shell) on the container
named `stack`, and runs it interactively.

Because the container captures the output from your programs, if you've used
the `run_stack.sh` script you can view stdout and stderr with:

    docker logs stack

You will also need to clean up the container when you are done:

    docker rm stack

### How to submit your assignment:

The only file you should need to modify is `exploit_1.c`. Add, commit, and
push your solution, so that it is uploaded to the server:

    git add exploit_1.c
    git commit
    git push origin main

### Tips

#### Make sure you can run `stack` and `exploit_1` as provided

You will need to be able to run both in a single container, so that
they can communicate. You can use `run_stack.sh` and `run_exploit.sh`
for this.  Don't forget to run `docker kill stack` and `docker rm
stack` when you're finished with the container.

#### Make sure you can run `stack` in `gdb`

For this, you will want to use `run_baseline.sh` instead of
`run_stack.sh`.  Otherwise, you will end up with two copies of `stack`
running, which will likely behave oddly.

#### Examine the behavior of `stack` in normal running conditions

In `gdb`, take a look at the memory layout. What information can you
gather, and what can you infer from it?

#### Identify the vulnerability or vulnerabilities in `stack.c`

The networking code is not the vulnerable part --- that would just be
cruel on our part.

#### Use the provided attack buffer in `exploit_1.c`

This buffer is large enough for your attack, and comes pre-filled with
NOPs. Just overwrite the parts that need to be different.

