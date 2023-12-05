# Add Icon for strfry relay

## Edit file landing page
~~~
sudo -iu strfry
nano /home/strfry/strfry/src/tmpls/landing.tmpl
# Add Lines
<link rel="icon" type="image/x-icon" href="https://satsdays.com/bangkok.png">

# Compile strfry again, stop strfry and copy new compiled strfry to /usr/local/bin then start strfry with new icon image.

~~~
