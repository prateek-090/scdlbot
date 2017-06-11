# [Music Downloader Telegram Bot](https://t.me/scdlbot)

[![PyPI version](https://badge.fury.io/py/scdlbot.svg)](https://pypi.python.org/pypi/scdlbot)
[![Updates](https://pyup.io/repos/github/gpchelkin/scdlbot/shield.svg?token=376ffde2-5188-4912-bf3c-5f316e52d43f)](https://pyup.io/repos/github/gpchelkin/scdlbot/)
[![GitHub license](https://img.shields.io/badge/license-GPLv3-green.svg)](https://raw.githubusercontent.com/gpchelkin/scdlbot/master/LICENSE.txt)
[![Telegram Bot](https://img.shields.io/badge/telegram-bot-blue.svg)](https://t.me/scdlbot)


## Bot Usage

Send `/start` or `/help` command to [bot](https://t.me/scdlbot) or refer directly to the [help message](scdlbot/messages/help.tg.md).

### Supported sites and used packages

- [**Telegram Bot API**](https://core.telegram.org/bots/api): [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot)
- [**SoundCloud**](https://soundcloud.com): [scdl](https://github.com/flyingrub/scdl)
- [**Bandcamp**](https://bandcamp.com): [bandcamp-dl](https://github.com/iheanyi/bandcamp-dl)
- [**YouTube**](https://www.youtube.com/), [**Mixcloud**](https://www.mixcloud.com/), etc.: [youtube-dl](https://rg3.github.io/youtube-dl)
- Use [SoundScrape](https://github.com/Miserlou/SoundScrape) in the future?

## Development

### TODO
- [Dokku webhooks support](https://github.com/python-telegram-bot/python-telegram-bot/wiki/Webhooks#using-haproxy-with-one-subdomain-per-bot)
- Async download and send
- Ask in groups only
- Check all links for Bandcamp type
- Something cool with Botan

### Installation

#### Requirements
Those should be available in your `PATH`:
- [**Python 3.6**](https://www.python.org/) ([pyenv](https://github.com/pyenv/pyenv) recommended)
- [**FFmpeg**](https://ffmpeg.org/download.html) for running locally (fresh builds for [Windows](https://ffmpeg.zeranoe.com/builds/) and [Linux](https://johnvansickle.com/ffmpeg/) recommended)
- [**Heroku CLI**](https://cli.heroku.com/) is recommended

#### Install from [PyPI](https://pypi.python.org/pypi/scdlbot) (preferred)
```
pip3 install scdlbot
```

#### Install from Git source
```
git clone https://github.com/gpchelkin/scdlbot.git
cd scdlbot
pip3 install --requirement requirements.txt

# If you want to install system-wide, not recommended:
python3 setup.py install
```

### Configuration

Copy config file sample and set up config environment variables in it:
```
# if you installed from PyPI, download sample:
wget https://raw.githubusercontent.com/gpchelkin/scdlbot/master/.env.sample

cp .env.sample .env
nano .env
```

##### Required
- `TG_BOT_TOKEN`: Telegram Bot API Token, [obtain here](https://t.me/BotFather), also diable privacy mode if you want
- `STORE_CHAT_ID`: Chat ID for storing audios for inline mode
- `SC_AUTH_TOKEN`: SoundCloud Auth Token, [obtain here](https://flyingrub.github.io/scdl/)

##### Optional
- `USE_WEBHOOK`: use webhook for bot updates: `1`, use polling (default): `0`, [more info](https://core.telegram.org/bots/api#getting-updates)
- `PORT`: Heroku sets this automatically for web dynos if you are using webhook
- `APP_URL`: Heroku App URL like `https://<appname>.herokuapp.com/`, required for webhook
- `BOTAN_TOKEN`: [Botan.io](http://botan.io/) [token](http://appmetrica.yandex.com/)
- `NO_CLUTTER_CHAT_IDS` — Comma-separated chat IDs with no replying and caption hashtags
- `BIN_PATH` — Custom directory where `scdl` and `bandcamp-dl` binaries are available, e.g. `~/.pyenv/shims/` if you use pyenv, default: empty
- `DL_DIR` — Parent directory for MP3 download directory, default: ~ (user's home directory)


### Running Locally

#### Using [Heroku Local](https://devcenter.heroku.com/articles/heroku-local#run-your-app-locally-using-the-heroku-local-command-line-tool) (preferred)
You need [Heroku CLI](https://cli.heroku.com/).
```
# if you installed from PyPI, download Procfile:
wget https://raw.githubusercontent.com/gpchelkin/scdlbot/master/Procfile

# for long polling:
heroku local worker
# for webhooks (you will also need to set up some NGINX with SSL):
heroku local web
```

#### Using just Python
```
export $(cat .env | xargs)
python3 -m scdlbot
# or just:
env $(cat .env | xargs) python3 -m scdlbot
```


### Deploying to [Heroku](https://heroku.com/)

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)

When app is deployed you **must** set only one dyno working on "Resources" tab in your app settings depending on [which way of getting updates](https://core.telegram.org/bots/api#getting-updates) you have chosen and set in config variables.


#### Manually
You can do the same as the button above but using [Heroku CLI](https://cli.heroku.com/), not much of a fun. Assuming you are in `scdbot` repository directory:

```
heroku login
# Create app with Python3 buildpack and set it for upcoming builds:
heroku create --buildpack heroku/python
heroku buildpacks:set heroku/python
# Add FFmpeg buildpack needed for youtube-dl:
heroku buildpacks:add --index 1 https://github.com/laddhadhiraj/heroku-buildpack-ffmpeg.git --app scdlbot
# Deploy app to Heroku:
git push heroku master
# Set config vars automatically from your .env file
heroku plugins:install heroku-config
heroku config:push
# Or set them one by one:
heroku config:set TG_BOT_TOKEN="<TG_BOT_TOKEN>" STORE_CHAT_ID="<STORE_CHAT_ID>" ...
```

If you use webhook, start web dyno and stop worker dyno:
```
heroku ps:scale web=1 worker=0
heroku ps:stop worker
```

If you use polling, start worker dyno and stop web dyno:
```
heroku ps:scale worker=1 web=0
heroku ps:stop web
```

Some useful commands:
```
# Attach to logs:
heroku logs -t
# Test run ffprobe
heroku run "ffprobe -version"
```

### Deploying to [Dokku](https://github.com/dokku/dokku)

Use Dokku and their docs on your own server. App is tested and fully ready for deployment with polling (no webhooks yet).

```
scp .env dokku.pchelk.in:~
ssh dokku.pchelk.in
dokku apps:create scdlbot
dokku config:set scdlbot $(cat .env | xargs)
# Ctrl+D
git remote add dokku dokku@dokku.pchelk.in:scdlbot
git push dokku master
ssh dokku.pchelk.in
dokku ps:scale scdlbot worker=1 web=0
dokku ps:restart
```
