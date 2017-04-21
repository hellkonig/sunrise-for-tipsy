# sunrise-for-tipsy

[Sunrise](https://bitbucket.org/lutorm/sunrise/overview) is a popular 
"Monte-Carlo Radiation Transfer code for calculating absorption and 
scattering of light in astrophysical situations". 
Unfortunately, it is notoriously finnicky to get installed, as it relies 
on specific point release versions of nearly a dozen different libraries. 
[Ben Keller](https://github.com/bwkeller) introduces a way to build 
Sunrise on [Docker](https://docs.docker.com/) in his blog. However, his 
instruction require root permission which does not work for people who 
is working on the server. 

By referring [Sunrise's wiki] (https://bitbucket.org/lutorm/sunrise/wiki/Compiling) 
and [Ben Keller's blog](http://www.physics.mcmaster.ca/~kellerbw/posts/sunrise-on-docker.html),
this instruction will build Sunrise as a normal on your home directory 
on server.

## Prerequisites

### ICC compiler

Unfortunately, sunrise will not compile with any free compilers. You will 
need to get the Intel C++ compil on your server. 
If the ICC compiler does not work on your server, you can get it from intel
and you may need to pay for it though and you have to consume your precious
time on installing it. If you do not want to spend time on this step, 
please contact the administrator to solve this problem.

### Set up Mercurial

Mercurial is a free, distribute source control management tool like git. In the
following instructions, you need it to clone projects rather than git so that
Mercurial is highly recommended to install.

You can find full descriptions for Mercurial setting up on Mercurial's 
[webpage](https://www.mercurial-scm.org/wiki/Download). 
Since Mercurial is written in Python, it is much easier to get Mercurial working
on your server by using pip or conda as a result, just execute 
```
pip install Mercurial
```
or 
```
conda install -c conda-forge mercurial
```
This shoudl work on all platform of Linux, even on OSX and Windows. If you meet
the permission problem by using pip, you can add the option `--user' to have a
try.
