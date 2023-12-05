# Backup strategy

## Overview

### Client
> Client snapshots are saved to a drive formatted with XFS.  This is because it is faster for this usecase than Btrfs and is less likely to experience inode exhaustion using it's default config than other file systems.

A backup script using rsync to make snapshots every 2 hours to a remote host storing the snapshots by datestamp directory in a directory named the same as the client host.  This script is installed via salt-minion.  A list of excludes is read from ~/.config/backup/excludes.txt and mostly contains caches, ISO files, and flatpack blobs.

### Server
> The backup destination drive mounted at /srv/backup is formatted with Btrfs

> [!note]
> I'm working on a script to use SQLite to store file hashes.  The reason is that each file, at each path (including every hardlink in the multitude of snapshots) requires about 2 GiB of space.  That's per client, per day.  See below for more detail.

1. Scan /srv/snapshots to get all client hostname folders
3. Run, in parallel
	1. Generate a hash file for each client hostname folder's contents using find | xargs b3sum >> /srv/backup/<client_hostname>_<datestamp>.b3sum
	2. lrztar | blkar each folder to /srv/backup/<client_hostname>_<datestamp>.lrz.blkar
4. Rsync the tools used to the /srv/backup/tools folder
5. Run pruning script
6. Run filesystem defragmentation
7. Run bees
8. Run borgmatic to borg backup the backup drive at /srv/backup to borgbase

## Borg Backup
Borg is used to keep offsite backup sizes small.  It is not meant for convenience.  Because of how I've structured the backup folder and it's contents, it would be trivial to extract the b3sum files and parse them to find the host and file/directory I'm looking for so I can get the correct path for a borg export-tar command in the event my local sources are not available.  In the latter case, it is also assumed I'll probably need everything anyway.

## File Hashes
I want to keep this for a few reasons.
1. Redundnat validation of files post extraction from an archive
2. An easy way to find files/directories that I want to have restored

I'm thinking I could have a table for archives, a table for hashes of each file with the file name, and a table correlating files and archives with the path of the file.  I'd need to write some code to do that though.  

| file_hash | file_name |
|----------|-----------|
| 871eba0d6c704855e8cbd324b8affcdb9fe4932d39b5502a7a1b713f252c93cf | file1 |
| 0ec92b972f8d3970102714e68a7b80fe0d57c8c7087074a97180affee9b0aa35 | file2 |
| db8089b0e1f677054bcc2692738ccdcccb97167a98cd96401d1b20334b14cea3 | file3 |

| archive_file | file_name | file_path | file_hash |
|--------------|-----------|-----------|-----------|
| archive1 | file1 | full/path/to/file1 | 871eba0d6c704855e8cbd324b8affcdb9fe4932d39b5502a7a1b713f252c93cf |
| archive2 | file1 | full/path/to/file1 | 871eba0d6c704855e8cbd324b8affcdb9fe4932d39b5502a7a1b713f252c93cf |
| archive2 | file1 | alternate/path/to/file1 | 871eba0d6c704855e8cbd324b8affcdb9fe4932d39b5502a7a1b713f252c93cf |
| archive3 | file2 | full/path/to/file2 | 0ec92b972f8d3970102714e68a7b80fe0d57c8c7087074a97180affee9b0aa35 |
| etc | etc | etc | etc |

