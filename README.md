# books

[B]ooks - which is only one of the names this program goes by - is a front-end for accessing a locally accessible libgen / libgen_fiction database instance, offering versatile search and download directly from the command line. The included update_libgen tool is used to keep the database up to date - if the database is older than a user-defined value it is updated before the query is executed. This generally only takes a few seconds, but it might take longer on a slow connection or after a long update interval. Updating can be temporarily disabled by using the '-x' command line option. To refresh the database(s) from a dump file use the included refresh_libgen program, see 'update_libgen vs refresh_libgen' below for more information on which tool to use.

Books comes in three main flavours:

* *books / books-all / fiction*: CLI search interface which dumps results to the terminal, download through MD5
* *nbook / nfiction*: text-based browser offering limited preview and download
* *xbook / xfiction*: gui-based browser offering preview and download


The *book* tools are based on the *libgen* database, the *fiction* tools use the *libgen_fiction* database. Apart from the fact that the *fiction* tools do not support all the search criteria offered by the 'book' tools due to differences in the database layout, all programs share the same interface.

The database can be searched in two modes, per-field (the default) and fulltext (which, of course, only searches book metadata, not the actual book contents). The current implementation for fulltext search is actually a pattern match search on a number of concatenated database columns, it does not use MySQL's native fulltext search. The advantage of this implementation is that it does not need a full-text index (which is not part of the libgen dump and would need to be generated locally), the disadvantage is that it does not offer more advanced natural language search options. Given the limited amount of 'natural language' available in the database the latter does not seem to be much of a disadvantage and the implementation performs well.

In the (default) per-field search mode the database can be searched for patterns (SQL 'like' operator with leading and trailing wildcards) using lower-case options and/or exact matches using upper-case options. The fulltext search by necessity always uses pattern matching over the indicated fields ('title' and 'author' if no other fields are specified).

Publications can be downloaded using torrents or from libgen download mirror servers by selecting them in the result list or by using the 'Download' button in the preview window, the *books* and *fiction* tools can be used to download publications based on their MD5 hash (use `-J ...`). When using the gui-based tools in combination with the 'yad' tool, double-clicking a row in the result list shows a preview, the other tools generate previews for selected publications using the '-w' command line option.

##How to use *books* et al.

I'll let the programs themselves do the talking:

```txt
$ books -h
books version 0.6

Use: books OPTIONS [like] [<PATTERN>]

(...)

SEARCH BY FIELD:

This is the default search mode. If no field options are given this searches
the Title field for the PATTERN. Capital options (-A, -T, etc) for exact match,
lower-case (-a, -t, etc) for pattern match.

FULLTEXT SEARCH (-f):

Performs a pattern match search over all fields indicated by the options. If no
field options are given, perform a pattern match search over the Author and
Title fields.

Depending on which name this program is executed under it behaves differently:

    books: query database and show results, direct download with md5
    books-all: query database and show results (exhaustive search over all tables, slow)

    nbook: select publications for download from list (terminal-based)
    xbook: select publications for download from list (GUI)

    fiction: query database and show results (using 'fiction' database), direct download with md5

    nfiction: select publications for download from list (terminal-based, use 'fiction' database)
    xfiction: select publications for download from list (GUI, use 'fiction' database)

OPTIONS

    -z, -Z	search on LOCATOR
    -y, -Y	search on YEAR
    -v, -V	search on VOLUMEINFO
    -t, -T	search on TITLE
    -s, -S	search on SERIES
    -r, -R	search on PERIODICAL
    -q, -Q	search on OPENLIBRARYID
    -p, -P	search on PUBLISHER
    -o, -O	search on TOPIC_DESCR
    -n, -N	search on ASIN
    -m		search on MD5
    -l, -L	search on LANGUAGE
    -i, -I	search on ISSN
    -g, -G	search on TAGS
    -e, -E	search on EXTENSION
    -d, -D	search on EDITION
    -c, -C	search on CITY
    -b, -B	search on IDENTIFIERWODASH
    -a, -A	search on AUTHOR

    -f		fulltext search
 		searches for the given words in the fields indicated by the other options.
 		when no other options are given this will perform a pattern match search
 		for the given words over the Author and Title fields.

    -w		preview publication info before downloading (cover preview only in GUI tools)
 		select one or more publication to preview and press enter/click OK.

 		double-clicking a result row also shows a preview irrespective of this option,
 		but this only works when using the yad gui tool

    -= DIR	set download location to DIR

    -$		use extended path when downloading:
    		    nonfiction/[topic/]author[/series]/title
    		    fiction/language/author[/series]/title

    -u BOOL	use bittorrent (-u 1 or -u y) or direct download (-u 0 or -u n)
 		this parameter overrides the default download method
 		bittorrent download depends on an external helper script
 		to interface with a bittorrent client

    -U MD5	print torrent path (torrent#/md5) for given MD5

    -j MD5	print filename for given MD5

    -J MD5	download file for given MD5
                can be combined with -u to download with bittorrent

    -M MD5	fast path search on md5, only works in _books_ and _fiction_
    		can be combined with -F FIELDS to select fields to be shown
    		output goes directly to the terminal (no pager)

    -F FIELDS	select which fields to show in pager output

    -# LIMIT	limit search to LIMIT hits (default: 1000)

    -x		skip database update
 		(currently only the 'libgen' database can be updated)

    -@		use torsocks to connect to the libgen server(s). You'll need to install
    		torsocks before using this option; try this in case your ISP
    		(or a transit provider somewhere en-route) blocks access to libgen

    -k		install symlinks for all program invocations

    -h		show this help message

EXAMPLES

Do a pattern match search on the Title field for 'ilias' and show the results in the terminal

  $ books like ilias


Do an exact search on the Title field for 'The Odyssey' and show the results in the terminal

  $ books 'the odyssey'


Do an exact search on the Title field for 'The Odyssey' and the Author field for 'Homer', showing
the result in the terminal

  $ books -T 'The Odyssey' -A 'Homer'


Do the same search as above, showing the results in a list on the terminal with checkboxes to select
one or more publications for download

  $ nbook -T 'The Odyssey' -A 'Homer'


A case-insensitive pattern search using an X11-based interface; use bittorrent (-u y or -u 1) when downloading files

  $ xbook -u y -t 'the odyssey' -a 'homer'


Do a fulltext search over the Title, Author, Series, Periodical and Publisher fields, showing the
results in a terminal-based checklist for download after preview (-w)

  $ nbook -w -f -t -a -s -r -p 'odyssey'


Walk over a directory of publications, compute md5 and use this to generate file names:

  $ find /path/to/publications -type f|while read f; do books -j $(md5sum "$f"|awk '{print $1}');done


As above, but print torrent number and path in torrent file

  $ find /path/to/publications -type f|while read f; do books -U $(md5sum "$f"|awk '{print $1}');done


Find publications by author 'thucydides' and show their md5,title and year in the terminal

  $ books -a thucydides -F md5,title,year


Get data on a single publication using fast path MD5 search, show author, title and extension

  $ books -M 51b4ee7bc7eeb6ed7f164830d5d904ae -F author,title,extension


Download a publication using its MD5 (-J MD5), using bittorrent (-u y or -u 1) to download

  $ books -u y -J 51b4ee7bc7eeb6ed7f164830d5d904ae

```

```txt
$ update_libgen -h
update_libgen version 0.6

Usage: update_libgen OPTIONS

    -l LIMIT	get updates in blocks of LIMIT entries
    -v		be verbose about what is being updated; repeat for more verbosity:
                -v: 	show basic info (number of updates, etc)
                -vv:	show ID, Title and TimeLastModified for each update
    -n		do not update database. Use together with -v or -vv to show
    		how many (-v) and which (-vv) titles would be updated.
    -j FILE	dump (append) json to FILE
    -s FILE	dump (append) sql to FILE
    -u URL	use URL to access the libgen API (overrides default)
    -t DATETIME	get updates since DATETIME (ignoring TimeLastModified in database)
     		use this option together with -s to create an sql update file to update
    		non-networked machines
    -i ID       get updates from ID

    -H DBHOST	database host
    -P DBPORT	database port
    -U DBUSER	database user
    -p DBPASS	database password (use empty string to get a password prompt)
    -D DATABASE	database name

    -a APIHOST	use APIHOST as API server
    -@		use tor (through torsocks) to connect to libgen API server
    -q		don't warn about missing fields in database or api response
    -h		this help message
```

```txt
$ refresh_libgen -h
refresh_libgen version 0.6.1

Usage: refresh_libgen OPTIONS

Performs a refresh from a database dump file for the chosen libgen databases.

    -n		do not refresh database
 		use together with '-v' to check if recent dumps are available
    -f		force refresh, use this on first install
    -v		be verbose about what is being updated
    -d DAYS	only use database dump files no older than DAYS days (default: 5)
    -u DBS	refresh DBS databases (default: compact fiction libgen)

    -H DBHOST	database host (localhost)
    -P DBPORT	database port (3306)
    -U DBUSER	database user (libgen)
    -R REPO	dump repository (http://gen.lib.rus.ec/dbdumps/)
    -c 		create a config file using current settings (see -H, -P, -U, -R)
    -e		edit config file

    -p DBPASS	database password (cache password for this session)
 		use empty string ("") to get password prompt

    -@		use tor (through torsocks) to connect to libgen server
    -k		keep downloaded files after exit
    -h		this help message
```

##Torrents, direct download...

*Books* (et al) can download files either from torrents (using `-u y` or `-u 1`) or from one of the libgen download mirrors (default, use`-u n` or `-u 0` in case torrent download is set as default). To limit the load on the download servers it is best to use torrents whenever possible. The latest publications are not yet available through torrents since those are only created for batches of 1000 publications. The feasibility of torrent download also depends on whether the needed torrents are seeded. Publications which can not be downloaded through torrents can be downloaded directly.

###Torrent download process
Torrent download works by selecting individual files for download from the 'official' torrents, i.e. it is *not* necessary to download the whole torrent for a single publication. This process is automated by means of a helper script which is used to interface *books* with a torrent client. Currently the only torrent client for which a helper script is available is *transmission-daemon*, the script uses the related *transmission-remote* program to interface with the daemon. Writing a helper script should not be that hard for other torrent clients as long as these can be controlled through the command line or via an API.

When downloading through torrents *books* first tries to download the related torrent file from the 'official' repository, if this fails it gives up and suggests using direct download instead. Once the torrent file has been downloaded it is checked to see whether it contains the required file. If this check passes the torrent is submitted to the torrent client with only the required file selected for download. A job script is created which can be used to control the torrent job, if the `torrent_cron_job` parameter in the PREFERENCES section or the config file is set to `1` it is submitted as a cron job. The task of this script is to copy the downloaded file from the torrent client download directory (`torrent_download_directory` in books.conf or the PREFERENCES section) to the target directory (preference `target_directory`) under the correct name. Once the torrent has finished downloading the job script will copy the file to that location and remove the cron job. If `torrent_cron_job` is not set (or is set to `0`) the job script can be called 'by hand' to copy the file, it can also be used to perform other tasks like retrying the download from a libgen download mirror server (use `-D`, this will cancel the torrent and cron job for this file) or to retry the torrent download (use `-R`). The script has the following options:

```txt
$ XYZ.job -h
Use: bash jobid.job [-s] -[i] [-r] [-R] [-D] [-h] [torrent_download_directory]

Copies file from libgen/libgen_fiction torrent to correct location and name

    -S	show job status
    -s	show torrent status (short)
    -i	show torrent info (long)
    -I	show target file name
    -r	remove torrent and cron jobs
    -R	restart torrent download (does not restart cron job)
    -D	direct download (removes torrent and cron jobs)
    -h	show this help message
```

###The torrent helper script interface
The torrent helper script (here named `ttool`) needs to support the following commands:

* `ttool add-selective <torrent_file> <md5>`
  download file <md5> from torrent <torrent_file>
* `ttool torrent-hash <torrent_file>`
  get btih (info-hash) for <torrent_file>
* `ttool torrent-files <torrent_file>`
  list files in <torrent_file>
* `ttool remove <btih>`
  remove active torrent with info-hash <btih>
* `ttool ls <btih>`
  show download status for active torrent with info-hash <btih>
* `ttool info <btih>`
  show extensive info (files, peers, etc) for torrent with info-hash <btih>
* `ttool active <btih>`
  return `true` if the torrent is active, `false` otherwise

Output should be the requested data without any headers or other embellishments. Here is an example using the (included) `tm` helper script for the *transmission-daemon* torrent client, showing all required commands:

```txt
$ tm torrent-files r_2412000.torrent 
2412000/00b3c21460499dbd80bb3a118974c879
2412000/00b64be1207c374e8719ee1186a33c4d
2412000/00c4f3a075d3af0813479754f010c491
...
... (994 files omitted for brevity)
...
2412000/ff2473a3b8ec1439cc459711fb2a4b97
2412000/ff913204c002f19ed2ee1e2bdfd236d4
2412000/ffb249ae5d148639d38f2af2dba6c681

$ tm torrent-hash r_2412000.torrent 
e73d4bc21d0f91088c174834840f7da232330b4d

$ tm add-selective r_2412000.torrent 00c4f3a075d3af0813479754f010c491
... (torrent client output omitted)

$ tm ls 6934f632c06a91572b4401e5b4c96eec89d311d7
    ID   Done       Have  ETA           Up    Down  Ratio  Status       Name
    25     0%       None  Unknown      0.0     0.0   None  Idle         762000
Sum:                None               0.0     0.0

(output from transmission-daemon, format is client-dependent)

$ tm info 6934f632c06a91572b4401e5b4c96eec89d311d7
... (torrent client output omitted)

$ tm active 6934f632c06a91572b4401e5b4c96eec89d311d7; echo "torrent is $([[ $? -gt 0 ]] && echo "not ")active"
torrent is active

$ if tm active 6934f632c06a91572b4401e5b4c96eec89d311d7; then echo "torrent is active"; fi
torrent is active

$ tm active d34db33f; echo "torrent is $([[ $? -gt 0 ]] && echo "not ")active"
torrent is not active
```

##Installation
Download this repository (or a tarball) and copy the four scripts - `books`, `update_libgen`, `refresh_libgen` and `tm` (only needed when using the transmission-daemon torrent client) - into a directory which is somewhere on your $PATH ($HOME/bin would be a good spot). Run `books -k`to create symlinks to the various names under which the program can be run:

* `books`
* `books-all`
* `fiction`
* `nbook`
* `xbook`
* `nfiction`
* `xfiction`

Create a database on a mysql server somewhere within reach of the intended host. Either open *books* in an editor to configure the database details (look for `CONFIGURE ME` below) and anything else (eg. `target_directory` for downloaded books, `max_age` before update, `language` for topics, MD5 in filenames, tools, etc) or add these settings to the (optional) config file `books.conf` in $XDG_CONFIG_HOME (usually $HOME/.config). The easiest way to create the config file is to run `refresh_libgen` with the required options. As an example, the following command sets the database server to `base.example.org`, the database port to `3306` and the database username to `genesis`:

```bash
$ refresh_libgen -H base.example.org -P 3306 -U genesis -c`
```

Make sure to add the `-c` option *at the end* of the command or it won't work. Once the config file has been created it can be edited 


```bash
main () {
        # PREFERENCES
        config=${XDG_CONFIG_HOME:-$HOME/.config}/books.conf

        # target directory for downloaded publications
        target_directory="${HOME}/Books"      <<<<<< ... CONFIGURE ME ... >>>>>>
        # when defined, subdirectory of $target_directory) for torrents
        torrent_directory="torrents"
        # when defined, location where files downloaded with torrent client end up
        # torrent_download_directory="/net/p2p/incoming" <<<<<< ... ENABLE/CONFIGURE ME ... >>>>>>
        # when true, launch cron jobs to copy files from torrent download directory
        # to target directory using the correct name
        torrent_cron_job=1
        # default limit on queries
        limit=1000
        # maximum database age (in minutes) before attempting update
        max_age=120
        # topics are searched/displayed in this language ("en" or "ru")
        language="en"       <<<<<<<<<<<< ... CONFIGURE ME ..... >>>>>>>>>>>>>>>>
        # database host
        dbhost="localhost"  <<<<<<<<<<<< ... CONFIGURE ME ..... >>>>>>>>>>>>>>>>
        # database port
        dbport="3306"       <<<<<<<<<<<< ... CONFIGURE ME ..... >>>>>>>>>>>>>>>>
        # database user
        dbuser="libgen"     <<<<<<<<<<<< ... CONFIGURE ME ..... >>>>>>>>>>>>>>>>
        # default fields for fulltext search
        default_fields="author,title"
        # window/dialog heading for dialog and yad/zenity
        list_heading="Select publication(s) for download:"

        # add md5 to filename? Possibly superfluous as it can be derived from the file contents but a good guard against file corruption
        filename_add_md5=0

        # tool preferences, list preferred tool first
        gui_tools="yad|zenity"
        tui_tools="dialog|whiptail"
        dl_tools="curl|wget"
        parser_tools="xidel|hxwls"
        pager_tools="less|more"

        # torrent helper tools need to support the following commands:
        # ttool add-selective <torrent_file> <md5>  # downloads file <md5> from torrent <torrent_file>
        # ttool torrent-hash <torrent_file>         # gets btih for <torrent_file>
        # ttool torrent-files <torrent_file>        # lists files in <torrent_file>
        torrent_tools="tm"   <<<<<<<<<<<< ... CONFIGURE ME ..... >>>>>>>>>>>>>>>>

        # database names to use:
        #   books, books-all, nbook, xbook and xbook-all use the main libgen database
        #   fiction, nfiction and xfiction use the 'fiction' database
        declare -A programs=(     
                [books]=libgen        <<<<<<<<<<<<<<<< ... CONFIGURE ME ..... >>>>>>>>>>>>>>>>
                [books-all]=libgen    <<<<<<<<<<<<<<<< ... CONFIGURE ME ..... >>>>>>>>>>>>>>>>
                [nbook]=libgen        <<<<<<<<<<<<<<<< ... CONFIGURE ME ..... >>>>>>>>>>>>>>>>
                [xbook]=libgen        <<<<<<<<<<<<<<<< ... CONFIGURE ME ..... >>>>>>>>>>>>>>>>
                [fiction]=libgen_fiction       <<<<<<< ... CONFIGURE ME ..... >>>>>>>>>>>>>>>>
                [nfiction]=libgen_fiction     <<<<<<<< ... CONFIGURE ME ..... >>>>>>>>>>>>>>>>
                [xfiction]=libgen_fiction     <<<<<<<< ... CONFIGURE ME ..... >>>>>>>>>>>>>>>>
                [libgen_preview]=libgen # the actual database to use for preview is passed as a command line option
        )
```

The same goes for the 'PREFERENCES' sections in `update_libgen` and `refresh_libgen`. In most cases the only parameters which might need change are `dbhost`, `dbuser`, `torrent_download_directory` and possibly `torrent_tools`. Since all programs use a common `books.conf` config file it is usually sufficient to add these parameters there:

```bash
$ cat $HOME/.config/books.conf
dbhost="base.example.org"
dbuser="exampleuser"
torrent_download_directory="/net/p2p/incoming"
torrent_tools="tm"
```

Please note that there is no option to enter a database password as that would be rather insecure. Either use a read-only, password-free mysql user to access the database or enter your database details in $HOME/.my.cnf, like so:

```ini
[mysql]
user=exampleuser
password=zooperzeekret
```

Make sure the permissions on $HOME/.my.cnf are sane (eg. mode 640 or 600), see http://dev.mysql.com/doc/refman/5.7/en/ ... files.html for more info on this subject.

Install symlinks to all tools by calling books with the -k option:

```
 $ books -k
```

##*update_libgen* vs. *refresh_libgen*

If you regularly use books, nbook and/or xbook, the main (or compact) database should be kept up to date automatically. In that case it is only necessary to use *refresh_libgen* to refresh the database when you get a warning from *update_libgen* about unknown columns in the API response.

If you have not used any of these tools for a while it can take a long time - and a lot of data transfer - to update the database through the API (which is what *update_libgen* does). Especially when using the compact database it can be quicker to use *refresh_libgen* to just pull the latest dump instead of waiting for *update_libgen* to do its job.

The *fiction* database can not be updated through the API (yet), so for that databases *refresh_libgen* is currently the canonical way to get the latest version.

##Dependencies

These tools have the following dependencies (apart from a locally available libgen/libgen_fiction instance on MySQL/MariaDB), sorted in order of preference:

* all: bash 4.x or higher - the script relies on quite a number of bashisms


* `books`/`fiction`: less | more (use less!)
* `nbook`/`nfiction`: dialog | whiptail (whiptail is buggy, use dialog!)
* `xbook`/`xfiction`: yad | zenity (more functionality with yad, but make sure your yad supports --html - you might have to build it yourself (use --enable-html during ./configure). If in doubt about the how and why of this, just use Zenity)


Preview/Download has these dependencies:

* awk (tested with mawk, nawk and gawk)
* stdbuf (part of GNU coreutils)
* xidel | hxwls (html parser tools, used for link extraction)
* curl | wget


`update_libgen` has the following dependencies:

* jq (CLI json parser/mangler)
* awk (tested with mawk, nawk and gawk)


`refresh_libgen` has these dependencies:

* w3m
* wget
* unrar
* pv (only needed when using the verbose (-v) option

`tm` has these dependencies:

* transmission-remote

