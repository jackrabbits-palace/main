# SSH Tunneling

How could we enable friendos to ssh into slime.church or serious.town via jackrabits-palace from anywhere in the world?

We dont' know yet. But so far we've come up with the following 2 shit solutions. (We'd prefer something where friendos don't have to do extra config to just type `ssh friendo@slime.church`).

## shit solution 1

enable tunnel for friendo on slime.church:

``` shell
$ ssh -R 222:localhost:22 jackrabbits-palace.local
```

frindo can now ssh to slime.church from anywhere, using:

``` shell
$ ssh friendo@jackrabbitspal.us:222
```

friendo, may optionally (for added chill vibez), edit their ssh config thusly:

``` shell
Host slimechurch
    HostName jackrabbitspal.us
    Port 222
```

then, friendo, can ssh into slime.church like this instead:

``` shell
$ ssh slimechurch
```


## shit solution 2

friendo *must* edit ssh config to say:

``` shell
Host jackrabbit
  HostName jackrabbitspal.us

Host slimechurch
  ProxyCommand ssh -q jackrabbit nc -q0 %h 22
```

then friendo may ssh to slime church as before:

``` shell
$ ssh slimechurch
```

## question to answer about shit solutions

in either shit solution, may friendo ssh onto jackrabbit and move around the file system? (we hope not!!!)

in shit solution 2: must we set up a user for friendo on jackrabbit for it to work?
