.htaccess Snippets Awesome
A collection of useful .htaccess snippets, all in one place.

NOTE: .htaccess files are for people that do not have rights to edit the main server configuration file. They are intrinsically slower and more complicated than using the main config. Please see the howto in the httpd documentation for further details.

Disclaimer: While dropping the snippet into an .htaccess file is most of the time sufficient, there are cases when certain modifications might be required. Use at your own risk.

IMPORTANT: Apache 2.4 introduces a few breaking changes, most notably in access control configuration. For more information, check the upgrading document as well as this issue.

Credits
What we are doing here is mostly collecting useful snippets from all over the interwebs (for example, a good chunk is from Apache Server Configs) into one place. While we’ve been trying to credit where due, things might be missing. If you believe anything here is your work and credits should be given, let us know, or just send a PR.

Table of Contents
Rewrite and Redirection
Force www
Force www in a Generic Way
Force non-www
Force non-www in a Generic Way
Force HTTPS
Force HTTPS Behind a Proxy
Force Trailing Slash
Remove Trailing Slash
Redirect a Single Page
Redirect Using RedirectMatch
Alias a Single Directory
Alias Paths to Script
Redirect an Entire Site
Alias "Clean" URLs
Exclude a URL from Redirection
Security
Deny All Access
Deny All Access Except Yours
Allow All Access Except Spammers'
Deny Access to Hidden Files and Directories
Deny Access to Backup and Source Files
Disable Directory Browsing
Disable Image Hotlinking
Disable Image Hotlinking for Specific Domains
Password Protect a Directory
Password Protect a File or Several Files
Block Visitors by Referrer
Prevent Framing the Site
Performance
Compress Text Files
Set Expires Headers
Turn eTags Off
Miscellaneous
Set PHP Variables
Custom Error Pages
Force Downloading
Prevent Downloading
Allow Cross-Domain Fonts
Auto UTF-8 Encode
Switch to Another PHP Version
Disable Internet Explorer Compatibility View
Serve WebP Images
Rewrite and Redirection
Note: It is assumed that you have mod_rewrite installed and enabled.

Force www
RewriteEngine on
RewriteCond %{HTTP_HOST} ^example\.com [NC]
RewriteRule ^(.*)$ http://www.example.com/$1 [L,R=301,NC]
Force www in a Generic Way
RewriteCond %{HTTP_HOST} !^$
RewriteCond %{HTTP_HOST} !^www\. [NC]
RewriteCond %{HTTPS}s ^on(s)|
RewriteRule ^ http%1://www.%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
This works for any domain. Source

Force non-www
It’s still open for debate whether www or non-www is the way to go, so if you happen to be a fan of bare domains, here you go:

RewriteEngine on
RewriteCond %{HTTP_HOST} ^www\.example\.com [NC]
RewriteRule ^(.*)$ http://example.com/$1 [L,R=301]
Force non-www in a Generic Way
RewriteEngine on
RewriteCond %{HTTP_HOST} ^www\.
RewriteCond %{HTTPS}s ^on(s)|off
RewriteCond http%1://%{HTTP_HOST} ^(https?://)(www\.)?(.+)$
RewriteRule ^ %1%3%{REQUEST_URI} [R=301,L]
Force HTTPS
RewriteEngine on
RewriteCond %{HTTPS} !on
RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI}

# Note: It’s also recommended to enable HTTP Strict Transport Security (HSTS)
# on your HTTPS website to help prevent man-in-the-middle attacks.
# See https://developer.mozilla.org/en-US/docs/Web/Security/HTTP_strict_transport_security
<IfModule mod_headers.c>
    # Remove "includeSubDomains" if you don't want to enforce HSTS on all subdomains
    Header always set Strict-Transport-Security "max-age=31536000;includeSubDomains"
</IfModule>
Force HTTPS Behind a Proxy
Useful if you have a proxy in front of your server performing TLS termination.

RewriteCond %{HTTP:X-Forwarded-Proto} !https
RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI}
Force Trailing Slash
RewriteCond %{REQUEST_URI} /+[^\.]+$
RewriteRule ^(.+[^/])$ %{REQUEST_URI}/ [R=301,L]
Remove Trailing Slash
This snippet will redirect paths ending in slashes to their non-slash-terminated counterparts (except for actual directories), e.g. http://www.example.com/blog/ to http://www.example.com/blog. This is important for SEO, since it’s recommended to have a canonical URL for every page.

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_URI} (.+)/$
RewriteRule ^ %1 [R=301,L]
Source

Redirect a Single Page
Redirect 301 /oldpage.html http://www.example.com/newpage.html
Redirect 301 /oldpage2.html http://www.example.com/folder/
Source

Redirect Using RedirectMatch
RedirectMatch 301 /subdirectory(.*) http://www.newsite.com/newfolder/$1
RedirectMatch 301 ^/(.*).htm$ /$1.html
RedirectMatch 301 ^/200([0-9])/([^01])(.*)$ /$2$3
RedirectMatch 301 ^/category/(.*)$ /$1
RedirectMatch 301 ^/(.*)/htaccesselite-ultimate-htaccess-article.html(.*) /htaccess/htaccess.html
RedirectMatch 301 ^/(.*).html/1/(.*) /$1.html$2
RedirectMatch 301 ^/manual/(.*)$ http://www.php.net/manual/$1
RedirectMatch 301 ^/dreamweaver/(.*)$ /tools/$1
RedirectMatch 301 ^/z/(.*)$ http://static.askapache.com/$1
Source

Alias a Single Directory
RewriteEngine On
RewriteRule ^source-directory/(.*) /target-directory/$1 [R=301,L]
Alias Paths to Script
FallbackResource /index.fcgi
This example has an index.fcgi file in some directory, and any requests within that directory that fail to resolve a filename/directory will be sent to the index.fcgi script. It’s good if you want baz.foo/some/cool/path to be handled by baz.foo/index.fcgi (which also supports requests to baz.foo) while maintaining baz.foo/css/style.css and the like. Get access to the original path from the PATH_INFO environment variable, as exposed to your scripting environment.

RewriteEngine On
RewriteRule ^$ index.fcgi/ [QSA,L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.fcgi/$1 [QSA,L]
This is a less efficient version of the FallbackResource directive (because using mod_rewrite is more complex than just handling the FallbackResource directive), but it’s also more flexible.

Redirect an Entire Site
Redirect 301 / http://newsite.com/
This way does it with links intact. That is www.oldsite.com/some/crazy/link.html will become www.newsite.com/some/crazy/link.html. This is extremely helpful when you are just “moving” a site to a new domain. Source

Alias “Clean” URLs
This snippet lets you use “clean” URLs -- those without a PHP extension, e.g. example.com/users instead of example.com/users.php.

RewriteEngine On
RewriteCond %{SCRIPT_FILENAME} !-d
RewriteRule ^([^.]+)$ $1.php [NC,L]
Source

Exclude URL from Redirection
This snippet allows you to exclude a URL from redirection. For example, if you have redirection rules setup but want to exclude robots.txt so search engines can access that URL as expected.

RewriteEngine On
RewriteRule ^robots.txt - [L]
Security
Deny All Access
## Apache 2.2
Deny from all

## Apache 2.4
# Require all denied
But wait, this will lock you out from your content as well! Thus introducing...

Deny All Access Except Yours
## Apache 2.2
Order deny,allow
Deny from all
Allow from xxx.xxx.xxx.xxx

## Apache 2.4
# Require all denied
# Require ip xxx.xxx.xxx.xxx
xxx.xxx.xxx.xxx is your IP. If you replace the last three digits with 0/12 for example, this will specify a range of IPs within the same network, thus saving you the trouble to list all allowed IPs separately. Source

Now of course there's a reversed version:

Allow All Access Except Spammers'
## Apache 2.2
Order deny,allow
Deny from xxx.xxx.xxx.xxx
Deny from xxx.xxx.xxx.xxy

## Apache 2.4
# Require all granted
# Require not ip xxx.xxx.xxx.xxx
# Require not ip xxx.xxx.xxx.xxy
Deny Access to Hidden Files and Directories
Hidden files and directories (those whose names start with a dot .) should most, if not all, of the time be secured. For example: .htaccess, .htpasswd, .git, .hg...

RewriteCond %{SCRIPT_FILENAME} -d [OR]
RewriteCond %{SCRIPT_FILENAME} -f
RewriteRule "(^|/)\." - [F]
Alternatively, you can just raise a “Not Found” error, giving the attacker no clue:

RedirectMatch 404 /\..*$
Deny Access to Backup and Source Files
These files may be left by some text/HTML editors (like Vi/Vim) and pose a great security danger if exposed to public.

<FilesMatch "(\.(bak|config|dist|fla|inc|ini|log|psd|sh|sql|swp)|~)$">
    ## Apache 2.2
    Order allow,deny
    Deny from all
    Satisfy All

    ## Apache 2.4
    # Require all denied
</FilesMatch>
Source

Disable Directory Browsing
Options All -Indexes
Disable Image Hotlinking
RewriteEngine on
# Remove the following line if you want to block blank referrer too
RewriteCond %{HTTP_REFERER} !^$

RewriteCond %{HTTP_REFERER} !^https?://(.+\.)?example.com [NC]
RewriteRule \.(jpe?g|png|gif|bmp)$ - [NC,F,L]

# If you want to display a “blocked” banner in place of the hotlinked image,
# replace the above rule with:
# RewriteRule \.(jpe?g|png|gif|bmp) http://example.com/blocked.png [R,L]
Disable Image Hotlinking for Specific Domains
Sometimes you want to disable image hotlinking from some bad guys only.

RewriteEngine on
RewriteCond %{HTTP_REFERER} ^https?://(.+\.)?badsite\.com [NC,OR]
RewriteCond %{HTTP_REFERER} ^https?://(.+\.)?badsite2\.com [NC,OR]
RewriteRule \.(jpe?g|png|gif|bmp)$ - [NC,F,L]

# If you want to display a “blocked” banner in place of the hotlinked image,
# replace the above rule with:
# RewriteRule \.(jpe?g|png|gif|bmp) http://example.com/blocked.png [R,L]
Password Protect a Directory
First you need to create a .htpasswd file somewhere in the system:

htpasswd -c /home/fellowship/.htpasswd boromir
Then you can use it for authentication:

AuthType Basic
AuthName "One does not simply"
AuthUserFile /home/fellowship/.htpasswd
Require valid-user
Password Protect a File or Several Files
AuthName "One still does not simply"
AuthType Basic
AuthUserFile /home/fellowship/.htpasswd

<Files "one-ring.o">
Require valid-user
</Files>

<FilesMatch ^((one|two|three)-rings?\.o)$>
Require valid-user
</FilesMatch>
Block Visitors by Referrer
This denies access for all users who are coming from (referred by) a specific domain. Source

RewriteEngine on
# Options +FollowSymlinks
RewriteCond %{HTTP_REFERER} somedomain\.com [NC,OR]
RewriteCond %{HTTP_REFERER} anotherdomain\.com
RewriteRule .* - [F]
Prevent Framing the Site
This prevents the website to be framed (i.e. put into an iframe tag), when still allows framing for a specific URI.

SetEnvIf Request_URI "/starry-night" allow_framing=true
Header set X-Frame-Options SAMEORIGIN env=!allow_framing
Performance
Compress Text Files
<IfModule mod_deflate.c>

    # Force compression for mangled headers.
    # https://developer.yahoo.com/blogs/ydn/pushing-beyond-gzipping-25601.html
    <IfModule mod_setenvif.c>
        <IfModule mod_headers.c>
            SetEnvIfNoCase ^(Accept-EncodXng|X-cept-Encoding|X{15}|~{15}|-{15})$ ^((gzip|deflate)\s*,?\s*)+|[X~-]{4,13}$ HAVE_Accept-Encoding
            RequestHeader append Accept-Encoding "gzip,deflate" env=HAVE_Accept-Encoding
        </IfModule>
    </IfModule>

    # Compress all output labeled with one of the following MIME-types
    # (for Apache versions below 2.3.7, you don't need to enable `mod_filter`
    #  and can remove the `<IfModule mod_filter.c>` and `</IfModule>` lines
    #  as `AddOutputFilterByType` is still in the core directives).
    <IfModule mod_filter.c>
        AddOutputFilterByType DEFLATE application/atom+xml \
                                      application/javascript \
                                      application/json \
                                      application/rss+xml \
                                      application/vnd.ms-fontobject \
                                      application/x-font-ttf \
                                      application/x-web-app-manifest+json \
                                      application/xhtml+xml \
                                      application/xml \
                                      font/opentype \
                                      image/svg+xml \
                                      image/x-icon \
                                      text/css \
                                      text/html \
                                      text/plain \
                                      text/x-component \
                                      text/xml
    </IfModule>

</IfModule>
Source

Set Expires Headers
Expires headers tell the browser whether they should request a specific file from the server or just grab it from the cache. It is advisable to set static content's expires headers to something far in the future.

If you don’t control versioning with filename-based cache busting, consider lowering the cache time for resources like CSS and JS to something like 1 week. Source

<IfModule mod_expires.c>
    ExpiresActive on
    ExpiresDefault                                      "access plus 1 month"

  # CSS
    ExpiresByType text/css                              "access plus 1 year"

  # Data interchange
    ExpiresByType application/json                      "access plus 0 seconds"
    ExpiresByType application/xml                       "access plus 0 seconds"
    ExpiresByType text/xml                              "access plus 0 seconds"

  # Favicon (cannot be renamed!)
    ExpiresByType image/x-icon                          "access plus 1 week"

  # HTML components (HTCs)
    ExpiresByType text/x-component                      "access plus 1 month"

  # HTML
    ExpiresByType text/html                             "access plus 0 seconds"

  # JavaScript
    ExpiresByType application/javascript                "access plus 1 year"

  # Manifest files
    ExpiresByType application/x-web-app-manifest+json   "access plus 0 seconds"
    ExpiresByType text/cache-manifest                   "access plus 0 seconds"

  # Media
    ExpiresByType audio/ogg                             "access plus 1 month"
    ExpiresByType image/gif                             "access plus 1 month"
    ExpiresByType image/jpeg                            "access plus 1 month"
    ExpiresByType image/png                             "access plus 1 month"
    ExpiresByType video/mp4                             "access plus 1 month"
    ExpiresByType video/ogg                             "access plus 1 month"
    ExpiresByType video/webm                            "access plus 1 month"

  # Web feeds
    ExpiresByType application/atom+xml                  "access plus 1 hour"
    ExpiresByType application/rss+xml                   "access plus 1 hour"

  # Web fonts
    ExpiresByType application/font-woff2                "access plus 1 month"
    ExpiresByType application/font-woff                 "access plus 1 month"
    ExpiresByType application/vnd.ms-fontobject         "access plus 1 month"
    ExpiresByType application/x-font-ttf                "access plus 1 month"
    ExpiresByType font/opentype                         "access plus 1 month"
    ExpiresByType image/svg+xml                         "access plus 1 month"
</IfModule>
Turn eTags Off
By removing the ETag header, you disable caches and browsers from being able to validate files, so they are forced to rely on your Cache-Control and Expires header. Source

<IfModule mod_headers.c>
    Header unset ETag
</IfModule>
FileETag None
Miscellaneous
Set PHP Variables
php_value <key> <val>

# For example:
php_value upload_max_filesize 50M
php_value max_execution_time 240
Custom Error Pages
ErrorDocument 500 "Houston, we have a problem."
ErrorDocument 401 http://error.example.com/mordor.html
ErrorDocument 404 /errors/halflife3.html
Force Downloading
Sometimes you want to force the browser to download some content instead of displaying it.

<Files *.md>
    ForceType application/octet-stream
    Header set Content-Disposition attachment
</Files>
Now there is a yang to this yin:

Prevent Downloading
Sometimes you want to force the browser to display some content instead of downloading it.

<FilesMatch "\.(tex|log|aux)$">
    Header set Content-Type text/plain
</FilesMatch>
Allow Cross-Domain Fonts
CDN-served webfonts might not work in Firefox or IE due to CORS. This snippet solves the problem.

<IfModule mod_headers.c>
    <FilesMatch "\.(eot|otf|ttc|ttf|woff|woff2)$">
        Header set Access-Control-Allow-Origin "*"
    </FilesMatch>
</IfModule>
Source

Auto UTF-8 Encode
Your text content should always be UTF-8 encoded, no?

# Use UTF-8 encoding for anything served text/plain or text/html
AddDefaultCharset utf-8

# Force UTF-8 for a number of file formats
AddCharset utf-8 .atom .css .js .json .rss .vtt .xml
Source

Switch to Another PHP Version
If you’re on a shared host, chances are there are more than one version of PHP installed, and sometimes you want a specific version for your website. The following snippet should switch the PHP version for you.

AddHandler application/x-httpd-php56 .php

# Alternatively, you can use AddType
AddType application/x-httpd-php56 .php
Disable Internet Explorer Compatibility View
Compatibility View in IE may affect how some websites are displayed. The following snippet should force IE to use the Edge Rendering Engine and disable the Compatibility View.

<IfModule mod_headers.c>
    BrowserMatch MSIE is-msie
    Header set X-UA-Compatible IE=edge env=is-msie
</IfModule>
Serve WebP Images
If WebP images are supported and an image with a .webp extension and the same name is found at the same place as the jpg/png image that is going to be served, then the WebP image is served instead.

RewriteEngine On
RewriteCond %{HTTP_ACCEPT} image/webp
RewriteCond %{DOCUMENT_ROOT}/$1.webp -f
RewriteRule (.+)\.(jpe?g|png)$ $1.webp [T=image/webp,E=accept:1]
