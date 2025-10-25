To run:

```
../hugo server -D  --disableFastRender -b=http://nishnik.com/ --appendPort=false
# run this from NishantBlog
# then move the public to /var/www/html
sudo rm -r /var/www/html/
sudo mv public /var/www/html
# Also change the image folder to have all `_` and add a `/` - look at other blogs to see it.
```
