# A POTA tool

This repository contains a small tool written in python that allows to query the POTA API from the command line. See below for the help text which is also output by `pota help`.

Some information bits and caveats:

* This is command line only! If you aren't someone doing most of your work on the command line, it is probably not for you.
* The tool was written in Python. I doesn't use anything fancy but I cannot say which python version is actually required. I'm using python 3.8 which works.
* Linux support only. I haven't written this for windows nor do I have any plans for windows support. You may be able to use it in WSL however.
* The program uses the *unofficial* POTA API because there is no official one. It may stop working every day because of changes to the API.
* The program uses caching and rate limiting in order to behave friendly to the pota servers. This behavior may be changed from the command line. Please act responsibly.
* I'm no python programmer, so the code is probably awful to read for anybody who one. Be prepared.
* The original version is not hosted on github. Github is just used for distribution. So instead of writing issues you may prefer to contact me directly.
* This software was written as a playground for me. So there might be changes and additions here and there, but there's no concrete plan for anything.
* The tool is able to output json so you may use it in other scripts that postprocess the json data.

Ok, so here's a copy of the help:

```
Usage: pota [global options] cmd [cmd options] [arguments]

Please note that order is important on the command line!

Available global options are:
  -c <dir>, --cachedir <dir>    Use this directory for cache files
  -f, --fromcache               Work only with the cache, no downloads
  -d, --debug                   Enable debug output
  -j, --json                    Output json data instead of text
  -l <sec>, --ratelimit <sec>   Rate limit API calls
  -n, --nocache                 Do not read or write cached data
  -r, --nocacheread             Do not read from the cache
  -w, --nowritecache            Do not write to the cache

Options -r/-w and -f cannot be used together. If -f is given, the cached data
is used regardless of its age. Commands with lots of API calls will
automatically enable rate limiting if not disabled on the command line.

Available commands and their arguments are:

activations [options] <call|park>
  Print activations for a park or call. For a park, all activations are shown,
  for a call only up to 25 recent ones. Command options:
    -u, --unsuccessful          Include unsuccessful activations

awards <call>
  Show the awards for a call.

cleanup
  Cleanup the cache. Remove all outdated data.

countries
  Print all countries.

leaderboard <park>
  Print the leader board for a park.

mayor <call|park>
  Print the mayor for a park or print all parks where a user is the mayor (*).
  For a call, only the parks from the last 25 activations are considered.
  Command options:
    -q, --qsos                  Use QSO count instead of activations

parks [country|region]
  Print parks sorted by park reference. Output may be limited to a single
  country or a region. Command options:
    -a, --activations           Sort by number of activations
    -f, --first                 Add first activation info (*)
    -q, --qsos                  Sort by number of qsos

profile <call|park>
  Print the profile for a call or park.

regions [country]
  Print regions. Output may be limited to a single country.

spots [options]
  Print currently active spots. Command options:
    -f f, --filter f            Add a filter for the displayed spots.
    -q, --qrt                   Include spots marked as qrt.
    -r, --rbn                   Print all spotters including RBN (*)
    -s, --spotters              Print all spotters w/o RBN (*)
  Filter tokens can be bands, calls, modes, parks, regions or countries. All
  filter tokens are compared in full, so you cannot use 'DE-00' to include
  just a subset of parks (those starting with 'DE-00').
  Multiple tokens can be added to one filter separated by commas. In this case
  they are "or" combined. Multiple filters specified with -f are "and"
  combined. So
    pota spots -f 20m,cw
  prints all spots for 20m and all spots using mode CW, while
    pota spots -f 20m -f cw
  prints all spots for 20m that are using mode CW.
  Filter tokens may be negated using '^' as prefix. So
    pota spots -f ^ft8
  shows all spots except those using FT8. Please note that using negations
  with token lists is almost useless.
    pota spots -f ^ft4,ft8
  will show all spots that aren't FT4 *OR* FT8 which actually means that all
  spots will be shown.

stats <call|park>
  Print statistics for a call or park.

version
  Print the software version.

Options marked with (*) will cause additional API calls and will automatically
apply rate limiting which increases command execution times. It is possible to
override rate limiting by adding '-l 0' to the command line but please note
that this stresses the POTA servers and is rather unfriendly.
```


