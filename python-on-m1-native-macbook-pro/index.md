# Python on M1 native Macbook Pro 2021


## What have we got

After upgrading to the new Macbook Pro with the M1 Pro processor at the end of 2021, it was time to install a newer version of Python.

```Shell
$ /echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

$ which python
/usr/bin/python

$ python --version
Python 2.7.18
```


While MacOS includes a version of Python I wanted a newer version and to keep it seperate from what Apple was using the other version for, and perhaps to have multiple environments for development. 

Git seems to be installed in MacOS Monterey and is a recent enough version to leave it as is for now.

```Shell
$ git --version
git version 2.32.0 (Apple Git-132)
```

I have decided to use Homebrew as the package manager for tools not included by Apple, and there will be many of these tools that can be easily installed when required.  Homebrew has support for the Apple Silicon CPU and hence the best optoin for my M1 Pro processor.

## Homebrew
### Installing Homebrew

The Instructions are easy enough to follow, with the requirements showing that Xcode will need to be installed. 

The only other main point is that it wants Borne-again shell (i.e. bash) to be the shell used for installation.  I did not have issues using the now default zsh shell, other than having to rebuild the *hash* table (see note below). It seems to call bash to interpret the downloaded script anyway.

Follow the instructions, and run the `bash curl URL/install.sh` command, which will do all the hard work and install everything under the path /opt/homebrew for Apple Silicon CPUs.

> Go forth and use the Homebrew installation docs:
>
> https://docs.brew.sh/Installation

When opening up a new terminal, the zsh profile `~/.zprofile` should set up the correct path from the installation.

```Shell
$ cat ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
alias ll="ls -l"
```

That last alias gives me Linux love to my Mac and helps with my typing muscle memory.

## Python
### Installing Python 3.9.9 or higher

```Shell
brew install python
```

It really is that easy.


## Checking the Python installation

```Shell
echo $PATH
/opt/homebrew/bin:/opt/homebrew/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

% for FILE in `which -a python3`; do echo -n "$FILE is "; $FILE --version; done
/opt/homebrew/bin/python3 is Python 3.9.9
/usr/bin/python3 is Python 3.8.9

% for FILE in `which -a python`; do echo -n "$FILE is "; $FILE --version; done
/opt/homebrew/bin/python is Python 3.9.9
/usr/bin/python is Python 2.7.18
```

`Note:`  As mentioned, because I ran python earlier or its path was already cached in the zsh hash table of names, I was not seeing the new install version and was seeing the Apple version of python version.  The hash table needs to be emptied and rebuilt, which is done easily with the following.

```Shell
hash -r
```

Now we can see inside zsh that it is finding the intended version (not cashed in the hash).
```Shell
% for FILE in `which -a python3`; do echo -n "$FILE is "; $FILE --version; done
/opt/homebrew/bin/python3 is Python 3.9.9
/usr/bin/python3 is Python 3.8.9

% for FILE in `which -a python`; do echo -n "$FILE is "; $FILE --version; done
/opt/homebrew/bin/python is Python 3.9.9
/usr/bin/python is Python 2.7.18
```

### Change some links

The last thing I did was to assume we are always running the command "python3" even when we just run "python".  I have switched my thinking to be in python3 unless I specify the path to Python 2.  So I have linked it as below.

```Shell
$ cd /opt/homebrew/bin
$ ls -l

$ ln -s python3 python
$ ln -s python3-config python-config
$ ln -s pip3 pip
```

Dont forget to re-hash again 
```Shell
$ hash -r
```

So now we have the everything

```Shell
% which python3
/opt/homebrew/bin/python3

% which python 
/opt/homebrew/bin/python

$ for FILE in `which -a python3`; do echo -n "$FILE is "; $FILE --version; done
/opt/homebrew/bin/python3 is Python 3.9.9
/usr/bin/python3 is Python 3.8.9

% for FILE in `which -a python`; do echo -n "$FILE is "; $FILE --version; done 
/opt/homebrew/bin/python is Python 3.9.9
/usr/bin/python is Python 2.7.18
```

Thats is it, time to go do some Python 3 programming!


