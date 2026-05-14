# custom-nginx
renaming nginx module created by Justin Hoffman to avoid conflicts with Webmin core nginx module

Webmin introduced their own native nginx module since Webmin version 2.640. Unfortunately this overwrites the 3rd party module created by Justin Hoffman.
https://www.justindhoffman.com/project/nginx-webmin-module

To safely rename your custom, third-party nginx module on Debian so it does not conflict with Webmin’s native nginx module, execute the following commands in your server terminal, follow next steps.

(If you already have the native nginx module installed. Remove first. Webmin->Webmin Configuration->Webmin Modules->Delete, Select the nginx module and Delete selected module (nginx).)

SSH into your server.

## Stop the Webmin Service ##
Stop Webmin to prevent any active configuration read/write conflicts.

```
systemctl stop webmin
```

## Move the Core Module Directory ##
On Debian installations, Webmin modules reside inside /usr/share/webmin. Move your third-party folder to its new name:
```
cd /usr/share/webmin
```
```
mv nginx custom-nginx
```

## Modify Internal Module Code References ##
Webmin scripts often load library files relative to their directory name (e.g., nginx-lib.pl). You must update internal references inside the module's folder: 
```
cd /usr/share/webmin/custom-nginx
```
Rename the core library file:
```
mv nginx-lib.pl custom-nginx-lib.pl
```
Update all file scripts to import the newly renamed library:
```
sed -i 's/nginx-lib.pl/custom-nginx-lib.pl/g' *.cgi *.pl
```

## Move the Module Configuration Directory ##
Webmin retains user configurations, permissions, and paths in /etc/webmin. Rename this directory to match your new module identifier:
```
cd /etc/webmin
```
```
mv nginx custom-nginx
```

## Update Permissions and Clear Module Cache ##
Webmin caches installed modules and maps access permissions per user. You need to point access from nginx to custom-nginx. 
Open your Webmin access control file:
```
nano /etc/webmin/webmin.acl
```
Locate your username (e.g., root:) and look through the string of modules. Change nginx to custom-nginx. Save and exit (Ctrl+O, Enter, Ctrl+X).

Clear out the cached system module registry so Webmin re-scans the directories:
```
rm -f /etc/webmin/module.infos.cache
```

## Start Webmin and Refresh ##
Start the service up again:
```
systemctl start webmin
```
Log into your Webmin browser interface.
In the left-hand menu, navigate to Webmin -> Refresh Modules.



       
