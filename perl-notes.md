---
layout: default
title: Perl notes
description: Some caveats in Perl system scripting.
tags: perl beginner tips programming system
---

* Table of contents
{:toc}

## Executing system commands synchronously

### Things to remember

1. The backticks capture output.
2. The system command does not capture output
3. A successful exec never returns
4. IPC::System::Simple handles most of the shell and error details for you.
5. IPC::Run3 allows you to work with input and output at the same time

### [Fire and forget with system](https://www.effectiveperlprogramming.com/2010/04/when-perl-isnt-enough)

[perlfunc](http://perldoc.perl.org/functions/system.html)

If you don’t care about the output, you can use system to start a command. It’s output goes to the same place your Perl output goes to (e.g. to the screen):

```pl
system( 'ls', '-l' )
	and die 'something went horribly wrong';
```

Your perl program waits (“blocks”) while the external command runs, and your perl program only continues once the external command completes.

The return value from system is the exit value from the external command. Each command can define its own exit values, but most commonly they return 0 for success or a non-zero number to indicate a particular sort of failure. At first glance the _and_ looks odd, but that’s because an error from ls returns a value that Perl considers true.

### [``]()

[perlop](http://perldoc.perl.org/perlop.html)

Backticks capture standard output of the command run and perl program execution is blocked.

Backticks also trigger the same interpolation that double-quotes do, so you can place variables in the backticks and they will be converted into text before Perl executes the command.

The backtick operator does variable interpolation without regard to the presence of single quotes in the command.

If you use the backticks in scalar context, Perl returns all of the standard output (but not standard error!) as a single, possibly multi-line string. If you use it in list context, then Perl returns a list that has one line per element:

```pl
# Full directory listing in a single scalar
my $output = `ls -l`;

# Full directory listing, one line per list item
my @output = `ls -l`;
```

The backticks have a generalized quoting syntax too. The qx{} quoting is the same as the literal `` but allows you to choose the delimiter (see perlop). These are all the same code:

```pl
my $output = `ls -l`;

my $output = qx{ls -l};

my $output = qx(ls -l);
```

### Use IPC::System::Simple to make life easier

Running external commands from within a Perl program is
easy enough, but you can easily get confused about whether or not those
commands succeeded or failed. IPC::System::Simple provides its own versions of system that dies when it encounters an error:

```pl
use IPC::System::Simple qw(system);
use Try::Tiny;

try {
	system( 'ls', '-l', $ARGV[0] );
	}
catch {
	print $_;
	};
```

When you run this command, you see that IPC::System::Simple figures out the success or failure for you (much like autodie):

```pl
% perl ipc_system_simple.pl /does/exist
-rw-rw-r-- 1 buster  staff  555 Jan  1 00:00 /does/exist
% perl ipc_system_simple.pl /does/not/exist
ls: /does/not/exist: No such file or directory
"ls" unexpectedly returned exit value 1 at ipc_system_simple.pl line 5
```

IPC::System::Simple also has a replacement for backticks, which it calls capture. You could run the ps command to get a list of running processes then look for the line with your program ($0 is the name of the currently running Perl program):

```pl
use IPC::System::Simple qw(capture);

my @processes = capture( 'ps', 'ux' );

print grep { /$0/o } @processes;
```

This program just prints the ps line that applies to itself:

```pl
% perl ipc_system_simple_capture.pl
buster 74442   0.0  0.1   601344   2876 s000  S+   11:08 AM 0:00.04 perl ipc_system_simple_capture.pl
```

IPC::System::Simple‘s system and capture commands behave much like Perl’s built-in system command, so they might invoke an underlying shell. If you want to prevent an shell interaction (i.e. don’t interpret metacharacters), you can use the systemx and capturex versions instead. This prevents you from accidently starting a command that does more than you wanted:

```pl
use IPC::System::Simple qw(systemx);
systemx("ps ux | rm -rf /");
```

The systemx thinks its argument a file with the literal characters you specified in the argument, and fails instead of trying to remove all of your files:

```pl
% perl ipc_system_simple_systemx.pl
"ps ux | rm -rf /" failed to start: "No such file or directory" at ipc_system_simple_system x.pl line 2
```

### Use IPC::Run3 to connect filehandles to external processes

Backticks, system, exec, and IPC::System::Simple are all fine ways of running processes in Perl; however, they don’t do a great job in working with output streams. What if you want to write to an external program, or if you want to both read and write to the same external process? In those cases, you want the IPC::Run3 module. This module is the modern successor to the core module IPC::Open3, which though more flexible, is also more complex to use.

IPC::Run3 is best served by an example. In this example you are going to send some input to the sort command and capture the standard input, and standard output.

With IPC::Run3, you can define a scalar variable ($stdin) that represents the input, and the scalar variables where it should put the output ($stdout and $stderr):

```pl
use IPC::Run3;
my @command = qw(sort -n);

my $stdin   = "5\n34\n32\n12\nhi\n";
my ( $stdout, $stderr ) = ( '', '' );

run3 \@command, \$stdin, \$stdout, \$stderr;

die "something went horribly wrong" if $?;

print "STDOUT\n$stdout\nSTDERR\n$stderr\n";
```

You can even do fancy things like bypass writing to a scalar and directly write to a file.

## [Perl traps](http://mojolicious.org/perldoc/perltrap)

Practicing Perl Programmers should take note of the following:

* Remember that many operations behave differently in a list context than they do in a scalar one. See perldata for details.

* Avoid barewords if you can, especially all lowercase ones. You can't tell by just looking at it whether a bareword is a function or a string. By using quotes on strings and parentheses on function calls, you won't ever get them confused.

* You cannot discern from mere inspection which builtins are unary operators (like chop() and chdir()) and which are list operators (like print() and unlink()). (Unless prototyped, user-defined subroutines can only be list operators, never unary ones.) See perlop and perlsub.

* People have a hard time remembering that some functions default to $_, or @ARGV, or whatever, but that others which you might expect to do not.

* The <FH> construct is not the name of the filehandle, it is a readline operation on that handle. The data read is assigned to $_ only if the file read is the sole condition in a while loop:

```pl
while (<FH>)*   { }
while (defined($_ = <FH>)) { }..
<FH>;  # data discarded!
```

* Remember not to use = when you need =~; these two constructs are quite different:

```pl
$x =  /foo/;
$x =~ /foo/;
```

* The do {} construct isn't a real loop that you can use loop control on.

* Use my() for local variables whenever you can get away with it (but see perlform for where you can't). Using local() actually gives a local value to a global variable, which leaves you open to unforeseen side-effects of dynamic scoping.

* If you localize an exported variable in a module, its exported value will not change. The local name becomes an alias to a new value but the external name is still an alias for the original.

As always, if any of these are ever officially declared as bugs, they'll be fixed and removed.

### Shell traps

* The arguments are available via @ARGV, not $1, $2, etc.

* The environment is not automatically made available as separate scalar variables.

* The shell's test uses "=", "!=", "<" etc for string comparisons and "-eq", "-ne", "-lt" etc for numeric comparisons. This is the reverse of Perl, which uses eq, ne, lt for string comparisons, and ==, != < etc for numeric comparisons.

