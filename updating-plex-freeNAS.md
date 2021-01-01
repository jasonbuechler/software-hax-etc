### Updating the Plex server running on FreeNAS
#### ...SEEMS like a giant ass-pain; but in concept it's very simple

---
### What *are* Plex server updates? (on FreeBSD)
* a folder full of files compressed into a single archive similar to a .zip
* this compressed new folder is literally just a replacement for the folder holding "the old version"
* we'll call this old/new key folder the "guts" of Plex Media Server
* *the actual "update" process is **almost** exclusively just replacing the old folder with a new one*

The update download itself looks like: 
* `PlexMediaServer-1.21.1.3830-6c22540d5-FreeBSD-amd64.tar.bz2`
* `PlexMediaServer-<version>-<release>-<platform/arch>.tar.bz2`
* The `.tar.bz2` suffix implies it was "archived" first into a .tar; then compressed into a .bz2 archive.

I said "almost" above, because after you "correctly" swap the new folder into place, you'll need to create a "linux shortcut" for one folder. I'll cover that below. 

---
### Where Plex lives: your FreeNAS jail vs your NAS pool

You really only need to care about the "guts" folder. The standard name of this folder is `plexmediaserver` unless you paid for PlexPass; in which case, it will be named `plexmediaserver-plexpass`.

You can "find" this folder by browsing deep into your storage pool's file system, or through "entering" the jail (via a console shell). *Note: if you choose the latter, the filesystem hierarchy "starts" at jail's root level.*

Actual "guts" of Plex server:
```
<yourPool>/iocage/jails/<plexJail>/root/usr/local/share/plexmediaserver/
    [inside the jail filesystem] -->   /usr/local/share/plexmediaserver/
```

If you ever need the logs, they're in this deceptively-named directory: 
```
<yourPool>/iocage/jails/<plexJail>/root/Plex Media Server/
    [inside the jail filesystem] -->   /Plex Media Server/
```

The 'iocage' jail system can be configured to use a different path, but for FreeNAS, I believe the default is as shown here.

---
### Instructions (sort of)

Semantic steps:
1. uncompress/extract the .tar.bz2 archive
2. remove (or just rename) the old "plexmediaserver" directory
3. rename the extracted directory to "plexmediaserver" (from "PlexMediaServer....")
4. make sure the "new" files are owned by the same owner/group as the "old" files
5. create a shortcut named "Plex_Media_Server" that points to "Plex Media Server"
6. restart the jail (or just the service)

Although everything above (except restarting) can be accomplished on a Windows machine using 7zip and WinSCP, it's plenty easy to do exclusively via command line, if that's your cup of tea. Note: these aren't suppose to match up exactly with the above. And they don't.

Steps for command-line:
1. get WHATEVER.tar.bz2 into the jail filesystem "somehow" (e.g. wget/scp/etc)
2. `cd <yourPool>/iocage/jails/<plexJail>/root/usr/local/share`
3. `bunzip2 WHATEVER.tar.bz2 | tar -xf -`
4. `mv plexmediaserver old_PMS_dir && mv PlexMediaServer* plexmediaserver`
5. `ln -s "Plex Media Server" plexmediaserver`
6. `chown -R root:wheel plexmediaserver`
7. `service plexmediaserver restart`

---
### What's in the update archive/the "guts" folder?

I'm putting this at the bottom because except for the shortcut, you don't need to know what's in there. But if you're curious:
```
plexmediaserver
├── CrashUploader
├── lib/
├── Plex Commercial Skipper
├── Plex DLNA Server
├── Plex Media Fingerprinter
├── Plex Media Scanner
├── Plex Media Server
├── Plex Relay
├── Plex Script Host
├── Plex Transcoder
├── Plex Tuner Service
├── Plex_Media_Server -> Plex\ Media\ Server   ## this is the shortcut/symlink that you need to make
├── Resources/
└── start.sh
```
