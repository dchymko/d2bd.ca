# site.ext

## Installation instructions

1. run `composer install`
2. run `php please make:user`
3. run `npm i` && `npm run dev`

## Environment file contents

### Development

```env
Dump your .env values here with sensitive data removed.
```

### Production


## NGINX config

Add the following to your NGINX config __inside the server block__ enable static resource caching:
```
expires $expires;
```

And this __outside the server block__:
```
map $sent_http_content_type $expires {
    default    off;
    text/css    max;
    ~image/    max;
    application/javascript    max;
    application/octet-stream    max;
}
```

## Deploy script Ploi

```bash
if [[ {COMMIT_MESSAGE} =~ "[BOT]" ]] && [[ {DEPLOYMENT_SOURCE} == "quick-deploy" ]]; then
    echo "Automatically committed on production. Nothing to deploy."
    {DO_NOT_NOTIFY}
    exit 0
fi

cd {SITE_DIRECTORY}
git pull origin {BRANCH}
composer install --no-interaction --prefer-dist --optimize-autoloader --no-dev

npm ci
npm run build

{RELOAD_PHP_FPM}

{SITE_PHP} artisan cache:clear
{SITE_PHP} artisan config:cache
{SITE_PHP} artisan route:cache
{SITE_PHP} artisan statamic:stache:warm
{SITE_PHP} artisan queue:restart
{SITE_PHP} artisan statamic:search:update --all
{SITE_PHP} artisan statamic:static:clear
{SITE_PHP} artisan statamic:static:warm --queue

echo "🚀 Application deployed!"
```

## Deploy script Forge

```bash
if [[ $FORGE_QUICK_DEPLOY == 1 ]]; then
    if [[ $FORGE_DEPLOY_MESSAGE =~ "[BOT]" ]]; then
        echo "Automatically committed on production. Nothing to deploy."
        exit 0
    fi
fi

cd $FORGE_SITE_PATH
git pull origin $FORGE_SITE_BRANCH
$FORGE_COMPOSER install --no-interaction --prefer-dist --optimize-autoloader --no-dev

npm ci
npm run build

( flock -w 10 9 || exit 1
    echo 'Restarting FPM...'; sudo -S service $FORGE_PHP_FPM reload ) 9>/tmp/fpmlock

$FORGE_PHP artisan cache:clear
$FORGE_PHP artisan config:cache
$FORGE_PHP artisan route:cache
$FORGE_PHP artisan statamic:stache:warm
$FORGE_PHP artisan queue:restart
$FORGE_PHP artisan statamic:search:update --all
$FORGE_PHP artisan statamic:static:clear
$FORGE_PHP artisan statamic:static:warm --queue
```
