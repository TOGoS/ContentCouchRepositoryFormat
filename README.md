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

## Default repository resolution

There is often a single repository being used by any given system.  Standard rules for finding it are as follows:
- If a local repository is explicitly specified by a command-line argument (or config file), the program should use that.
- If ```CCOUCH_REPO_DIR``` (or one of [several fallbacks](#environment-variables)) environment variable is set and non-empty, use that
  (and use ```CCOUCH_REPO_NAME``` as the name of that repository, unless
  a configuration file within the repository overrides that)
- Otherwise, check for ".ccouch" in the user's home directory.  There is no default repository name,
  and anything that depends on it being named (such as writing head files)
  should throw an error.

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
(though lines beginning with ```#``` followed by a word with no whitespace between may be given
[special meaning](https://github.com/TOGoS/M3UExtensions)
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

## Environment variables

Not all versions of ContentCouch actually look at these variables!
Nevertheless, if you are storing this information in environment variables,
e.g. for use by scripts that run ccouch, or other utilities,
try to use them as indicated below:

- `CCOUCH_REPO_DIR` (fallbacks: `ccouch_repo_dir`, `ccouch_dir`, `ccouch_repo_path`) - path of directory containing 'heads', 'data', etc
- `CCOUCH_REPO_NAME` (fallbacks: `ccouch_repo_name`) - name of your local repository
- `CCOUCH_STORE_SECTOR` (fallbacks: `ccouch_store_sector`) - name of sector into which explicitly-stored files should
  be stored/uploaded, e.g. "pictures", 'share', etc)
- `CCOUCH_CACHE_SECTOR` (fallbacks: `ccouch_cache_sector`, `CCOUCH_STORE_SECTOR`, `ccouch_store_sector`) -
  name of sector within which blobs cached
  as a side-effect of other operations (i.e. when the goal is not
  specifically to store them) should be stored.
  Caching should fall back to ccouch_store_sector if this is not set.
  
Not used directly by CCouch, but for my own purposes:
  
- `TOG_STUFF_DIR` (fallbacks: `tog_stuff_dir`) - contains TOG's smaller documents -- `docs`, `proj`, etc.
- `TOG_DATASTORE_DIR` (fallbacks: `datastore_root`; read-only fallbacks: `TOG_STUFF_DIR`, `tog_stuff_dir`, `"D:"`) - contains collections of large files; 'archives', 'incoming', 'share', 'music/work', etc
- `TOG_MUSIC_WORK_DIR` (should default to `TOG_STUFF_DIR + "/music/work"`)

Changes:
- 2019-11-19 - `ccouch_repo_dir` replaces `ccouch_dir` and/or `ccouch_repo_path`
- 2022-06-23 - changing to prefer CAPITALIZED_VARIABLE_NAMES instead of lowercase_ones,
  the idea being that lowercase signifiy local variables used by scripts,
  whereas capitalized ones represent configuration used across the system,
  such as `TERM`, `EDITOR`, `PATH`, `NO_COLOR`, etc.
