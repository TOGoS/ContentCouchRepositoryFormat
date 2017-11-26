# ContentCouch Repository Format

This is here so that I can maintain documentation on the format of ```.ccouch``` directories in one place,
since there are multiple programs that would like to be interoperable:

- [TPFetcher](//github.com/TOGoS/TPFetcher.git)
- [PHPN2R](//github.com/TOGoS/PHPN2R.git)
- [ContentCouch](https://github.com/TOGoS/ContentCouch.git)
- [ContentCouch3](https://github.com/TOGoS/ContentCouch3.git)
- and probably some others

The purpose of this document is to assign specific meanings and encodings to specific files within a repository,
but not every repository is expected to contain all the files mentioned.
Programs should have some reasonable default behavior when some files are missing.

## Directory structure

- ```.ccouch/```
  - ```data/``` - content stored here
    - _sector name_ - one directory for each arbitrary 'sector', which can be used to separate different sets of data
      - _first 2 digits of base32-encoded hash_ -
        base32 as in RFC-3548 to match [the encoding of hash-based URNs](http://www.nuke24.net/docs/2015/HashURNs.html)
        - _base32-encoded hash_ - lowest common denominator is SHA-1,
	  but any hash algorithm could be used so long as chance of collision remains low.
	  For encoding a bitprint, the dot should be included, to match the post-scheme part of a bitprint URN.
  - ```heads/``` - encoded commit objects are stored here by name
    - _source repository name_
      - _project name/path_ - e.g. "archives/images/TOGoS"
        - _commit number_ - should match that of the source repository
  - ```cache/``` - cached information;
    nothing should be stored in this directory that can't be recreated on demand
  - ```remote-repos.lst``` - a list of remote repository pseudo-URLs (see below)
  - ```repo-name.txt``` - single-line text file containing the name of this repository

## remote-repos.lst

A list of repositories, one per line.
Blank lines and lines starting with ```#``` are ignored
(though lines beginning with ```#``` followed by a word with no whitespace between may be given special meaning
which may be ignored)
Repositories may be specified in one of several formats:

- _hostname_ - ```http://``` prefix and ```/uri-res/N2R?``` postfix are asssumed
- ```http://```_hostname_ - ```/uri-res/N2R?``` postfix is assumed
- ```http://```_hostname_```/```_resolver-path_ - path to ```N2R```-like endpoint.
  If _resolver-path_ does not end with ```?``` or ```/```, ```?``` is assumed to separate
  the path from the URL.

TODO: some way to indicate high or low-priority repositories,
or ones that are only intermittently available and therefore should only
be checked as a last resort.
Maybe lines like ```#priority:N``` and ```#transient``` to indicate properties
of repositories listed.
Also need a way to indicate names of remote repositories; maybe just ```#name:```_name_.
