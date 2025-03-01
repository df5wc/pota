# A POTA tool

This repository contains a small tool written in python that allows to query the POTA API from the command line. You can for example query current spots and filter them by mode, band or other stuff. It allows to query call profiles, park statistics and more. See the help text below for a more detailed explanation. The help text is also output by `pota help`.

Some information bits and caveats:

* This is command line only! If you aren't someone doing most of your work on the command line, it is probably not for you.
* The tool was written in Python. I doesn't use anything fancy but I cannot say which python version is actually required. I'm using python 3.8 which works.
* The Python `requests` library is required. It is usually a separate package in your Linux distribution or may be installed using `pip install requests`.
* Linux support only. I haven't written this for windows nor do I have any plans for windows support. You may be able to use it in WSL however.
* The program uses the *unofficial* POTA API because there is no official one. It may stop working every day because of changes to the API.
* The program uses caching and rate limiting in order to behave friendly to the pota servers. This behavior may be changed from the command line. Please act responsibly.
* I'm no python programmer, so the code is probably awful to read for anybody who is one. Be prepared.
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

activators [options]
  Print a list of all activators, the number of activations and the number of
  contacts.
    -a, --activations           Sort by number of activations instead of call
    -c <regex>, --call <regex>  Extract calls that match the given regex
    -m <modes>, --mode <modes>  Filter by modes (see below)
    -t, --totals                Add a line with totals
    -T, --TOTALS                Print just one line with totals
    -y <years>, --year <years>  Limit data to the given year (or years)
  The regular expression given to '-c' is a real regex that must match the
  full call sign. It behaves as if it were enclosed in '^...$'. This allows
  to prefix a call filter regex with '^' which negates it. Multiple call
  filters may be used which are 'and' combined. So
    pota activators -a -y 2025 -c 'd[a-r].*' -c '^df5.*'
  prints the list of activators in 2025 with german call signs but excludes
  the ones starting with 'DF5'.
  Entries can be filtered by modes and number of QSOs in this mode. Valid modes
  are 'cw', 'data' or 'phone'. A mode string may just contain a mode name or
  include a comparison operator and a number appended. If there is an
  additional percent sign, the percentage of QSOs in this mode is compared. If
  just a mode is given, a QSO count > 0 is tested. So '-m cw>100' filters all
  activators with more than 100 cw QSOs. '-m cw>50%' applies to all activators
  with more than 50% of their total QSOs in cw. As usual, a match can be
  inverted by prefixing the filter with '^'. More than one mode filter can be
  specified by separating the expressions using commas. In this case the
  filters are 'or' combined. Filters in separate '-m' arguments are 'and'
  combined.
  The '-y' option accepts a single year or a comma separated list of years.
  Example that outputs data from 2024 and 2025 for all call signs starting
  with 'DF5':
    pota activators -y 24,2025 -c 'df5.*'
  Output activators with german call signs from 2025 that have cw qsos:
    pota activators -y 2025 -c 'd[a-r].*' -m cw
  Same as above but with cw only:
    pota activators -y 2025 -c 'd[a-r].*' -m cw -m ^data -m ^phone
  Another way to say the same:
    pota activators -y 2025 -c 'd[a-r].*' -m '^cw<100%'
  Ranking (most activations first) for 2024 excluding data operators:
    pota activators -a -y 2024 -m ^data
  Output all activators that do mostly cw:
    pota activators -a -m 'cw>90%'

awards <call>
  Show the awards for a call.

cleanup
  Cleanup the cache. Remove all outdated data.

config name[=value]
  Query or set configuration variables.
  Currently supported:
    general.callsign    The callsign of the user. Used when spotting.

  The configuration is stored in a file named '.pota' in the users home
  directory.

countries
  Print all countries.

leaderboard <park>
  Print the leader board for a park.

mayor <call|park|region(*)|country(*)>
  Print the mayor for a park, all parks in a region or country or print all
  parks where a user is the mayor (*).
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

spot [options] call@park freq [mode]
  Post a spot. Command options:
    -c text, --comment text     Post using this text as comment.
  The call@park, freq and mode arguments can be specified in any order. They
  are detected from their syntax. If no mode is given, SSB is assumed. The
  frequency must be specified in kilohertz and may include trailing digits.

  The spotters callsign is taken from the configuration. See the config
  command on how to set it.

  Example:
    pota spot df5wc@de-0008 14032.1 cw -c "thanks, 73!"


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
