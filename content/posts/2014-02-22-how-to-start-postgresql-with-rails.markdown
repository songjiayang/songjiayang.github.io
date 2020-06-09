---
title: How to start postgresql with rails
date: 2014-02-22 00:24:32 +0800
tags: [ruby, postgresql] 
---

#### Step 1: Install postgresql server with homebrew

```
brew install postgresql
```

After installed, Typing `brew info postgresql` in Terminal ,you will get some important infomation:

```
postgresql: stable 9.3.2 (bottled)
http://www.postgresql.org/
Conflicts with: postgres-xc
/usr/local/Cellar/postgresql/9.3.2 (2924 files, 39M) *
  Poured from bottle
From: https://github.com/Homebrew/homebrew/commits/master/Library/Formula/postgresql.rb
==> Dependencies
Required: readline ✔
Recommended: ossp-uuid ✔
==> Options
--32-bit
	Build 32-bit only
--enable-dtrace
	Build with DTrace support
--no-perl
	Build without Perl support
--no-tcl
	Build without Tcl support
--without-ossp-uuid
	Build without ossp-uuid support
--without-python
	Build without python support
==> Caveats
If builds of PostgreSQL 9 are failing and you have version 8.x installed,
you may need to remove the previous version first. See:
  https://github.com/Homebrew/homebrew/issues/issue/2510

To migrate existing data from a previous major version (pre-9.3) of PostgreSQL, see:
  http://www.postgresql.org/docs/9.3/static/upgrading.html

When installing the postgres gem, including ARCHFLAGS is recommended:
  ARCHFLAGS="-arch x86_64" gem install pg

To install gems without sudo, see the Homebrew wiki.

To reload postgresql after an upgrade:
    launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
```  

#### Step 2: Install pg gem

```
ARCHFLAGS="-arch x86_64" gem install pg
```

if errors, just try:

```
$ whitch pg_config
  /usr/local/bin/pg_config
$ gem install pg -- --with-pg-config=/usr/local/bin/pg_config
```
