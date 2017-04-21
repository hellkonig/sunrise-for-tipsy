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

By referring [Sunrise's wiki](https://bitbucket.org/lutorm/sunrise/wiki/
Compiling) and [Ben Keller's blog](http://www.physics.mcmaster.ca/~keller
bw/posts/sunrise-on-docker.html),
