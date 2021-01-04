# Removing DRM from Kindle eBooks

## Requirements
Requirements are surprisingly light on necessary technical knowledge! Just some simple software but there are some things like **maximum** versions you need to be aware of, or the whole thing blows up.

The big one is: if you're using the Kindle desktop software, *"DRM can only be removed from KFX format files downloaded with Kindle for PC/Mac 1.26 or earlier."* ...according to the DeDRM developer/repo. Contrastingly, you're going to want the latest version of all the other pieces. (DeDRM and Calibre both migrated from python2 to python3 a few years ago, which does mean there are some old versions that aren't compatible.)

* A desktop computer (mac/pc/linux maybe!)
* The free/opensource Calibre software (mac/pc/linux)
* A source for your eBooks:
  * a Kindle with books on it (+ usb cable), or
  * the Kindle-for-PC/Mac software (with downloaded books)
* Two plugins for Calibre (.zip files):
  * the KFX Input plugin
  * the DeDRM for Calibre plugin
  
## Where to get the software

* [Calibre](https://calibre-ebook.com/download) developer's website
* [Kindle for PC](http://www.oldversion.com/windows/kindle-for-pc/) old-versions archive
* Kindle for Mac 
  * just google for old versions (no convenient site I know of)
* [DeDRM_tools](https://github.com/apprenticeharper/DeDRM_tools) github repo (download from 'releases')
* [KFX Input plugin](https://www.mobileread.com/forums/showthread.php?t=291290) obnoxious forum thread (sorry)

## Lazy directions

1. Install Calibre on your computer
2. Install the KFX Input plugin for Calibre
   * find the Preferences for Calibre (this can literally be difficult, depending on version and your screen width)
   * go the 'plugins' section of preferences
   * use the "load plugin from file" button
   * target the *zip* file for the plugin (no need to unzip it!)
3. Install the DeDRM plugin for Calibre
   * do like above
   * modern versions are *just* the zip you need
   * older versions had zips for multiple tools inside another zip
   * you (probably?) need to use the customize button on this plugin, and add the serial number of your Kindle ereader
4. Prep your ebook source
   * a usb-connected Kindle, or
   * an "old enough" version of Kindle-for-PC/Mac, with books downloaded locally after installing the *old* version.  ...AND make sure it's set to NOT auto-update in its settings! Otherwise it'll almost immediately update itself.
5. Simply find and add the ebook file, and "add" it to Calibre
   * use the "add" button on the toolbar
   * if it fails, it'll import a "nothing" book with no info and no art
   * on a mac, desktop Kindle ebooks are in `~/Library/Application Support/Kindle/My Kindle Content/`
   * on a PC,  desktop Kindle ebooks are in `%userprofile%\Documents\My Kindle Content`



## old notes -- ignore probably

Initial setup

If you already have Kindle for PC installed
Disable automatic updates in settings
Uninstall it
Skip all of step 2 except 2b, obviously
Install old version of Kindle For PC
DISABLE YOUR INTERNET CONNECTION
Install it and open it
Disable automatic updates in settings
Re-enable network
Sign-in to your Amazon account
Install Calibre
Install “KFX Input” plugin for Calibre
Preferences > Plugins > Add from file > .zip file for plugin
Install “kindle unpack” plugin for Calibre
Preferences > Plugins > Add from file > .zip file for plugin
Install DeDRM tools
Unzip the dedrm tools archive
Install the “for calibre” plugin same as prev plugins
Input your Kindle serial number in the plugin settings


Not initial setup

Ideally download your ebook to your computer using Kindle for PC
If you already have the books on your device you don’t strictly need to but you’ll probably get better access to multiple formats via PC
Open Calibre
Use “Add books” toolbar icon
Point it to the book file(s) on your device, OR
Point it to the book file(s) on your computer: ~/Documents/My Kindle Content
Calibre should automatically add a DRM-free copy to your Calibre library
You can rightclick > show location to expose the file in Explorer


Also not initial setup

Some amazon book file formats can be “unpacked” via the KindleUnpack plugin to expose, for example, 
Embedded PDF or ePub files
These exposed files are lossless, with 100% original formatting
If you want to use a drm-free book on a device that can’t read a specific format, you’ll need to “convert” the file to, say, .mobi or .ePub
Converting isn’t a “lossless” operation: formatting may change or be lost
If you get a DRM error trying to view the ebook in Calibre, make sure
You have the KFX Input plugin
You have your Kindle(s) serial number saved in the DeDRM plugin’s settings
For books originating from Kindle-for-PC, DeDRM automatically grabs unlock keys from k4pc but only if k4pc is logged in to your amazon account
Don’t try to use the DeDRM “stand-alone” python script: it’s currently broken


