HikVision Telegram Camera Bot
=============================
Bot which sends snapshots from your HikVision camera(s).

Installation
------------

To install bot, simply `clone` repo and install dependencies using `pip3`.
Make sure you have [Python 3](https://www.python.org/downloads/) installed.


```
git clone https://github.com/tropicoo/hikvision-camera-bot.git
pip3 install Pillow python-telegram-bot requests watchdog xmltodict
```

Configuration
-------------
First of all you need to [create Telegram Bot](https://core.telegram.org/bots#6-botfather)
 and obtain its token.

Before starting HikVision Telegram Camera Bot needs to be configured.
Configuration is simply stored in [JSON](https://spring.io/understanding/JSON#structure) 
format.

Copy default configuration file `config-template.json` with name you like e.g.
to `config.json` and edit, which comes with default template:
```json
{
  "telegram": {
    "token": "",
    "allowed_user_ids": []
  },
  "watchdog": {
    "enabled": false,
    "directory": ""
  },
  "log_level": "INFO",
  "camera_list": {
    "cam_1": {
      "description": "Kitchen Camera",
      "api": {
        "host": "",
        "auth": {
          "user": "",
          "password": ""
        },
        "endpoints": {
          "picture": "/Streaming/channels/102/picture?snapShotImageType=JPEG",
          "motion_detection": "/MotionDetection/1/",
          "alert_stream": "/ISAPI/Event/notification/alertStream"
        },
        "stream_timeout": 300
      },
      "motion_detection": {
        "alert": {
          "enabled": false,
          "delay": 20,
          "fullpic": true
        }
      }
    },
    "cam_2": {
      "description": "Basement Camera",
      "api": {
        "host": "",
        "auth": {
          "user": "",
          "password": ""
        },
        "endpoints": {
          "picture": "/Streaming/channels/102/picture?snapShotImageType=JPEG",
          "motion_detection": "/MotionDetection/1/",
          "alert_stream": "/ISAPI/Event/notification/alertStream"
        },
        "stream_timeout": 300
      },
      "motion_detection": {
        "alert": {
          "enabled": false,
          "delay": 20,
          "fullpic": true
        }
      }
    }
  }
}
```

To get things done follow the next steps:
1. Put the obtained bot token to `token` key as string.
2. [Find](https://stackoverflow.com/a/32777943) your Telegram user id
and put it to `allowed_user_ids` list as integer value. Multiple ids can
be used, just separate them with a comma.
3. If you wish to monitor directory and send files to yourself when created,
enable `watchdog` by setting `enabled` to `true` and specify `directory`
path to be monitored e.g. `/tmp/watchdir`. Make sure that directory exists.
For example configure your camera to take and put snapshot on move detection
through FTP to watched folder. Watchdog looks for `on_create` events, sends
created file and deletes it.
4. HikVision camera settings are placed inside the `camera_list` section. Template
comes with two cameras. Preferable names of cameras are `cam_1`,
`cam_2`, `cam_3` and so on with any description.
5. Write authentication credentials in appropriate keys: `user` and `password`
for every camera you want to use.
6. Same for `host`, which should include protocol e.g. `http://192.168.10.10`
7. In `motion_detection` section you can enable sending picture on alert.
Configure `delay` setting in seconds between pushing alert pictures.
To send resized picture change `fullpic` to `false`.

**Example configuration**
```json
{
  "telegram": {
    "token": "23546745:VjFIo2q34fjkKdasfds0kgSLnh",
    "allowed_user_ids": [
      1011111,
      5462243
    ]
  },
  "watchdog": {
    "enabled": true,
    "directory": "/tmp/watchdir"
  },
  "log_level": "INFO",
  "camera_list": {
    "cam_1": {
      "description": "Kitchen Camera",
      "api": {
        "host": "http://192.168.10.10",
        "auth": {
          "user": "admin",
          "password": "kjjhthOogv"
        },
        "endpoints": {
          "picture": "/Streaming/channels/102/picture?snapShotImageType=JPEG",
          "motion_detection": "/MotionDetection/1/",
          "alert_stream": "/ISAPI/Event/notification/alertStream"
        },
        "stream_timeout": 300
      },
      "motion_detection": {
        "alert": {
          "enabled": false,
          "delay": 20,
          "fullpic": true
        }
      }
    }
  }
}
```

Usage
=====
Simply run and see for welcome message in Telegram client.
> Note: This will log the output to the stdout/stderr (your terminal). Closing
the terminal will shutdown the bot.
```bash
python3 bot.py -c config.json

# Or make the script executable by adding 'x' flag
chmod +x bot.py
./bot.py -c config.json
```

If you want to run the bot in the background use the following commands
```bash
# With writing to the log file
nohup python3 bot.py -c config.json &>/tmp/camerabot.log &

# Without writing to the log file
nohup python3 bot.py --config config.json &>- &
```

### Misc
If you're on the Raspberry Pi, you can easily add bot execution to the startup.

Simply edit the `/etc/rc.local` add the following line before the last one
(which is `exit 0`):
```bash
# The path '/home/pi/hikvision-camera-bot/config.json' is an absolute path to the config file (same for 'bot.py')
nohup python3 /home/pi/hikvision-camera-bot/bot.py --config /home/pi/hikvision-camera-bot/config.json &>- &
```