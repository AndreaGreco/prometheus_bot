# prometheus_bot

This bot is designed to alert messages from [alertmanager](https://github.com/prometheus/alertmanager).

## Compile

[GOPATH related doc](https://golang.org/doc/code.html#GOPATH).
```bash
export GOPATH="your go path"
make clean
make
```

## Usage

1. Create Telegram bot with BotFader, it will return your bot token

2. Specify telegram token in ```config.yaml```:

    ```yml
    telegram_token: "token goes here"
    # ONLY IF YOU USING TEMPLATE required for test

    template_path: "template.tmpl" 
    time_zone: "Europe/Rome"
    split_token: "|"    

    # ONLY IF YOU USING DATA FORMATTING FUNCTION, NOTE for developer: important or test fail
    template_time_outdata: "02/01/2006 15:04:05" 
    ```

3. Run ```telegram_bot```. See ```prometheus_bot --help``` for command line options

### Configuring alert manager

Alert manager configuration file:

```yml
- name: 'admins'
  webhook_configs:
  - send_resolved: True
    url: http://127.0.0.1:9087/alert/-chat_id
```

Replace ```-chat_id``` with the number you got from your bot, with ```-```. To use multiple chats just add more receivers.

## Test

To run tests with `make test` you have to:

- Create `config.yml` with a valid telegram API key and timezone in the project directory
- Create `prometheus_bot` executable binary in the project directory
- Define chat ID with `TELEGRAM_CHATID` environment variable
- Ensure port `9087` on localhost is available to bind to

```bash
export TELEGRAM_CHATID="-YOUR TELEGRAM CHAT ID"
make test
```
### Create your own test
When alert manager send alert to telegram bot, *only debug flag ```-d```* Telegram bot will dump json in that generate alert, in stdout.
You can copy paste this from json for your test, by creating new .json.
Test will send ```*.json``` file into ```testdata``` folder

or

```sh
TELEGRAM_CHATID="-YOUR TELEGRAM CHAT ID" make test
```

## Customizing messages with template

This bot support [go templating language](https://golang.org/pkg/text/template/).
Use it for customizing your message.

For enable template you must set this two settings in your ```config.yaml``` or template will skip.
```yml
telegram_token: "token here"
template_path: "template.tmpl" # your template file name
time_zone: "Europe/Rome" # your time zone check it out from WIKI
split_token: "|" # token used for split measure label.
```

You can also pass template path with `-t` command line argument, it has higher priority than the config option.

[WIKI List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

Best way for build your custom template is:
-    Enable bot with ```-d``` flag
-    Catch some of your alerts in json, then copy it from bot STDOUT
-    Save json in testdata/yourname.json
-    Launch ```make test```

```-d``` options will enable ```debug``` mode and template file will reload every message, else template is load once on startup.

Is provided as [default template file](testdata/default.tmpl) with all possibile variable.
Remember that telegram bot support HTML tag. Check [telegram doc here](https://core.telegram.org/bots/api#html-style) for list of aviable tags.

### Template extra functions
Template language support many different functions for text, number and data formatting.

#### Support this functions list

-   ```str_UpperCase```: Convert string to uppercase
-   ```str_LowerCase```: Convert string to lowercase
-   ```str_Title```: Convert string in Title, "title" --> "Title" fist letter become Uppercase
-   DEPRECATED  ```str_Format_Byte```: Convert number expressed in ```Byte``` to number in related measure unit. It use ```strconv.ParseFloat(..., 64)``` take look at go related doc for possible input format, usually every think '35.95e+06' is correct converted.
Example:
    -    35'000'000 [Kb] will converter to '35 Gb'
    -    89'000 [Kb] will converter to '89 Mb'
-   ```str_Format_MeasureUnit```: Convert string to scaled number and add append measure unit label. For add measure unit label you could add it in prometheus alerting rule. Example of working: 8*e10 become 80G. You cuold also start from a different scale, example kilo:"s|g|3". Check production example for complete implemetation. Require ```split_token: "|"``` in conf.yaml
-   ```HasKey```: Param:dict map, key_search string Search in map if there requeted key

-    ```str_FormatDate```: Convert prometheus string date in your preferred date time format, config file param ```time_outdata``` could be used for setup your favourite format
Require more setting in your cofig.yaml
```yaml
template_time_zone: "Europe/Rome"
template_time_outdata: "02/01/2006 15:04:05"
```
[WIKI List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

## Production example

Production example contains a example of how could be a real template.

```testdata/production_example.json```
```testdata/production_example.tmpl```

It could be a base, for build a real tempalte, or simply copy some part, check-out how to use functions.
Sysadmin usually love copy.

