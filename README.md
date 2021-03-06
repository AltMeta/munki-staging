# Introduction

This is a script that utilises a Trello board to manage the promotion of
Munki items through development to production and then, if desired to
archival.  You can use any number of steps between development and
production, but by default the script will expect 3 Munki Catalogs:

* development
* testing
* production

and this introduction will focus on this example. Taking these
catalogs as a basis, you would create 5 Trello boards:

* To Development: Items placed in this list will be moved to the development catalog when the script next runs.
* Development: Items in here are in the Development list. Do not place items directly in here, the script will manage the addition / removal of items to the list.
* To Testing: Items placed in this list will be moved to the testing catalog when the script next runs.
* Testing: Items here are in testing.
* To Production: Items here will be moved into production on the next run.

When items are moved into production, they are (by default) moved to a
dated list, so you can have a history of when items were placed into
production. One list will be made per day. However, you can turn off
this behaviour if you wish. This dated behaviour can also be enabled on
development and production.

# Usage

## Setup

It is recommended that this script is run under a service Trello account rather than a real persons, so you can separate the changes made by the script from normal users. This user will need to have access to the Trello board you're using. You will need to know the board ID - the board ID is the part after ``/b`` and before the name of your board (with a URL or https://trello.com/b/AbCdEfGh/my-trello-board, __AbCdEfGh__ would be the board ID.)

You will need an [API key](https://trello.com/app-key). Make note of the key and then head over to [Trello's instructions](https://trello.com/docs/gettingstarted/#token) for creating a user token. Choose how long you want to issue to token for - using the value of 'never' will stop the token from expiring. The only required option is read and write access to the Trello account. The name can be anything you like, it's how you will identify the token in future.

```
https://trello.com/1/authorize?key=substitutewithyourapplicationkey&name=munki-trello&expiration=never&response_type=token&scope=read,write
```

You will be given a 64 character string that you will need to take note of.

You will also need to install the trello module:

```
$ sudo easy_install trello
```

## Running the script

You can run the script manually on a machine that has the Munki makecatalogs command installed (this will run on OS X or Linux, Windows isn't tested).
Note that unlike the upstream version, there is no Docker setup.

In order to get the maximum flexibility from the script, you will need
to use a configuration file; however most options for the development,
testing and production setup above are available on the command line.

### Example

```
$ python munki-trello.py --boardid 12345 --key myverylongkey --token myevenlongertoken --repo-path /Volumes/my-repo
```

### Command line Options

* ``--boardid``: Required. The ID of your Trello board.
* ``--key``: Required. Your Trello API key.
* ``--token``: Required. Your Trello User Token.
* ``--config``: Optional. A file to read configuration settings from. 
* ``--to-dev-list``: Optional. The name of your 'To Development' list. Defaults to ``To Development``.
* ``--dev-list``: Optional. The name of your 'Development' list. Defaults to ``Development``.
* ``--to-test-list``: Optional. The name of your 'To Testing' list. Defaults to ``To Testing``.
* ``--test-list``: Optional. The name of your 'Testing' list. Defaults to ``Testing``.
* ``--to-prod-list``: Optional. The name of your 'To Production' list. Defaults to ``To Production``.
* ``--prod-suffix`` or ``--suffix``: Optional. The suffix that will be put after the dated 'Production' lists. Defaults to ``Production``; if unset packages will be added to the production list.
* ``--prod-list``: Optional. The name of your 'Production' list. Defaults to ``Production``; only used when ``--prod-suffix`` is unset.
* ``--repo-path``: Optional. The path to your Munki repository. Defaults to ``/Volumes/Munki``.
* ``--makecatalogs``: Optional. The path to ``makecatalogs``. Defaults to ``/usr/local/munki/makecatalogs``.
* ``--date-format``: Optional. The date format to use when creating dated lists. See strftime(1) for details of the formating options.  Defaults to ``%d/%m/%y``.
* ``--dev-stage-days``: Optional. Set the due date for autostaging; as packages are added into development, this will set the card due date to the current time plus the about of time given. You will need to seperately turn on staging, which is independent of this option.  Default: 0 (no due date set).
* ``--stage-test``: Optional. Automatically promote packages with a due date set from the development list into the testing list. Note: there is a seperate option to enable the setting of the due date.  Default: False (no auto promotion to test).
* ``--test-stage-dates``: Optional. Set the due date for autostaging; as packages are added into test, this will set the card due date to the current time plus the about of time given. You will need to seperately turn on staging, which is independent of this option.  Default: 0 (no due date set).
* ``--stage-prod``: Optional. Automatically promote packages with a due date set from the testing list into the production list. Note: there is a seperate option to enable the setting of the due date.  Default: False (no auto promotion to test).

## Configuration file

You can give all of the command line options in a configuration file,
which will be read first. The default configuration file
locations are:
    /etc/munki-trello/munki-trello.cfg
    ./munki-trello.cfg
and these will always be checked. You can also add an extra config
file location by using the --config command line option.

N.B. Configuration files will be processed *before* command line options,
and not all configuration items have a command line equivalent.

Options on the command line will be used in preference to those in the
configuration file. An example configuration file is in
munki-trello.cfg-template.

The configuration file has several sections:
  * the `[main]` section with some global defaults
  * the optional `[rssfeeds]` section 
  * Munki repository sections (`[munk_repo_<name>]`)
  * Munki catalog sections (`[munki_catalog_<name>]`)

#### The `[main]` section

The main section contains global configuration items; the data aboud
the Trello board, the path to makecatalogs and the date_format to be
used.

The full options are:

* ``boardid``: The ID of your Trello board.
* ``key``:  Your Trello API key.
* ``token``:  Your Trello User Token.
* ``makecatalogs``: the path to the munki makecatalogs script. Defaults to ``/usr/local/munki/makecatalogs``.
* ``date-format``: The date format to use when creating dated lists. See strftime(1) for details of the formatting options (note that the script assumes use of numeric options only).  Defaults to ``%d/%m/%y``.

Note that the script requires that `boardid`, `key` and `token are set
either in the configuration file or on the command line.

#### The `[rssfeeds]` section

If present, this section configures the output of RSSFeeds of packages
in each catalog, that you can publish so that people know which
version of a package is in a catalog, and when the software available
changes. 

In order to use RSSfeeds you will need to install:
```
$ sudo easy_install PyRSS2Gen
```
You will also need to configure the following; there are no defaults:

*``rssdir``: the directory to publish the RSSFeeds to (one file per catalog, named after the catalog)
*``rss_link_template``: the link in the RSSFeeds for the item; can use the following templates: `%(name)s', '%(version)s`, `%(catalog)s`
*``guid_link_template``: a unique link to this version of the package (this will be used by RSS Readers to track the package entry) ; can use the following templates: `%(name)s', '%(version)s`, `%(catalog)s`
# Have %(catalog)s
*``catalog_link_template``: a link to information about the catalog; can use the following template: `%(catalog)s`
*``description_template``: the description of the RSS Channel; can use the following template: `%(catalog)s`
*``icon_url_template``: a link to the Munki icons;  can use the follo
wing template: `%(icon_path)s` - the on disk path to the Munki icon


As an example, a complete RSS Feed configuration is:

```
[rssfeeds]
rssdir=/srv/www/site.orchardox.ac.uk/htdocs/rssfeeds
rss_link_template=https://site.orchard.ox.ac.uk/packages/%(name)s
guid_link_template=https://site.orchard.ox.ac.uk/packages/%(name)s/%(version)s
catalog_link_template=https://site.orchard.ox.ac.uk/catalogs/%(catalog)s
description_template='Software packages in Orchard %(catalog)s catalog'
icon_url_template=https://site.orchard.ox.ac.uk/munki/%(icon_path)s
```

#### The Munki catalog sections `[munki_catalog_<name>]`

The Munki catalog sections contain information about different Munki
catalogs and their related Trello lists. This information includes any
configuration of autostaging, and the setting of due dates.

The name in the section title is not used, but it is suggested that
this follow the name of the Munki catalog, as this will aid
readability of the configuaration file.

The full options are:

*``list``: ''Required''. The name of the list in the trello board; when using dated lists this is also the suffix used after the date.
*``to_list``: The name of the list in trello in which to put packages to be migrated into this catalog; defaults to 'To <list>'.
is as 
*``catalog``: ''Required''. The name of the Munki catalog that this list is used for.
*``stage_days``: Default: unset. The number of days that a package remains in this catalog before being autostaged/promoted to the stage_to catalog.  Note: autostaging must be enabled in order for package staging to occur.
*``stage_to``: The name of the munki repository/config section to
stage packages to (if auto staging).
*``autostage``: Default: 0 (off). Whether or not to automatically promote packages based on the Trello card due date.
*``munki_repo``: The name of the underlying Munki repository (if using more than one Munki repository.
*``dated_lists``: Default: 0 (off). If new Trello lists are created when packages are moved into this catalog, based on the date the packages are moved.

#### The Munki repository sections `[munki_catalog_<name>]`

These sections document the Munki repository or repositories that
packages live in. You will require at least one repository. However,
you can have as may repositories as you would like; an use case for
this is to have a 'main' repository and a 'retired' repository, and
migrate older versions of packages into the 'retired' repository,
which could be on slower storage.

There is one required parameter:

*``repo_path``: The path to the Munki repository


# Troubleshooting

> I'm seeing items that won't move to the next stage no matter how often I move them.

Make sure the combination of ``name`` and ``version`` is unique. For speed, the initial ingest of Munki data is done via your ``all`` catalog rather than traversing your pkgsinfo files. If you have two pkgsinfo files that have the same version / name combination as anther, this script won't touch anything after the first. Once the duplicate(s) have been removed, the item will be promoted to the next stage.

# Copyright 

Copyright (c) 2015 University of Oxford

This is based on an original script by Graham Gilbert
<graham@grahamgilbert.com> with significant rewriting by the Unix
Services Team within IT Services, University of Oxford.


