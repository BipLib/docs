# Bip Library Documentation

- [Introduction](#introduction)
- [Getting Started](#getting-started)
  - [Installation](#installation)
  - [Run Sample bot](#run-sample-bot)
  - [Create your own bot](#create-your-own-bot)
- [Components](#components)
  - [Introduction](#components-introduction)
  - [Bot](#bot)
  - [Stage](#stage)
  - [Node](#node)
  - [Routing System](#routing-system)
  - [Telegram Bot API](#telegram-bot-api)
  - [Helpers](#helpers)
  - [Config](#config)
  - [Database](#database)
  - [Logger](#logger)


<a name="introduction"></a>
## Introduction
Bip (bot in php) is open source library for creating stage base and powerful telegram bots. It is written in PHP and uses [Telegram Bot API](https://core.telegram.org/bots/api) to communicate with Telegram servers.

Bip uses stages to implement the bots. Each [stage](#stage) is a class that contains Nodes, [Node](#node) is method of stage that do actions. these Nodes are handled by [Routing System](#routing-system). You can use [Helpers](#helpers) to make your code more readable and easy to use. and use [Call](#telegram-bot-api) for calling telegram bot api methods with IDE Autocomplete.

<a name="getting-started"></a>
## Getting Started
For getting started you need to get bip library and install it on your server.

<a name="installation"></a>
### Installation
It is recommended to use [composer](https://getcomposer.org) to install the Bip :

```bash
composer require biplib/bip
```

<a name="run-sample-bot"></a>
### Run Sample bot
in `bots/NotePad` there is a sample NotePad bot that you can run it and see how it works.
For running this bot you need to edit `bots/NotePad/config.php` and set your bot token that you get from [BotFather](https://t.me/BotFather).

Then copy url of `bots/NotePad/index.php` and set it as webhook url by calling this url in your browser (replace `<YOUR_BOT_TOKEN>` with your bot token and `<YOUR_SERVER_URL>` with your server url) :

`https://api.telegram.org/bot<YOUR_BOT_TOKEN>/setWebhook?url=<YOUR_SERVER_URL>/bip/bots/NotePad/index.php`
> **Note**
> Telegram needs ssl certificate for webhook url, so you need to use https url. if you want only test Bip use [telegram-bot-api](https://github.com/tdlib/telegram-bot-api) for testing your bot on local machine.

if you get `{"ok":true,"result":true,"description":"Webhook was set"}` as response, your bot is ready to use. go to telegram and start your bot and enjoy it.

<a name="create-your-own-bot"></a>
### Create your own bot
For creating your own bots you need to create a directory for each bot in `bots` directory.
Name of directory is the name of your bot must be `PascalCase` for example `NotePad` or `MyBot` for PSR-4 autoload compatibility.

Then create a `index.php` file for start point of bot in bot directory and add this code to it :

```php
<?php

require __DIR__."/../../vendor/autoload.php"; #load autoloader
require 'config.php'; #load config file

use Bip\Bot; #use Bot class
use Bots\<BOT_DIRECTORY>\Stages\StartStage; #use StartStage class replace <BOT_DIRECTORY> with your bot directory



Bot::init(new StartStage()); #init bot with StartStage
Bot::run(); #run bot
```

Then create a `config.php` file in bot directory and add this code to it :

```php
<?php

use Bip\App\Config;
use Bip\Database\LazyJsonDatabase;

Config::add([
    'token'     => '<YOUR_BOT_TOKEN>', #set your bot token
    'admins'    => [12345678,123456789],#set admins id
    'database'  => new LazyJsonDatabase('database.json.db'), #set database LazyJsonDatabase is simple file-based database
]);

```

Now you can create [stages](#stage) for your bot in `Stages` directory of your bot.
for example create `StartStage.php` in `Stages` directory and add this code to it :

```php
<?php
namespace Bots\<BOT_DIRECTORY>\Stages; #replace <BOT_DIRECTORY> with your bot directory.


use Bip\App\Stage;

class StartStage extends Stage
{
    public function controller()
    {
        route('welcome')->onMessageText('/start');
    }
    public function welcome()
    {
        msg('Hello World');
        closeNode();
    }

}
```

`controller` method is called when bot is started, and it is used for [routing](#routing-system). in this example we route `/start` command to `welcome` method. overriding `controller` method is required for each stage.
Don't forget to call `closeNode()` at the end of each node. if you don't call `closeNode()`, bot will not continue to next node and will be stuck in current node. with `bindNode()` you can bind a node to a method and call it later. for example :

```php
...
class StartStage extends Stage
{
    public string $name = 'none'; #public properties are automatically reassigned to the stage
    public function controller() #controller is method that is called first.
    {
        route('setName')->onMessageText('/setName');
        route('getName')->onMessageText('/getName');
    }
    public function setName() #this method is Node
    {
        msg('What is your name?');
        bindNode('saveName');
    }
    public function saveName() #this method is Node
    {
        $this->name = Webhook::getObject()->message->text;
        msg("You'd like me to call you by the name $this->name");
        closeNode();
    }
    public function getName() #this method is Node
    {
        msg("Your name is $this->name");
        closeNode();
    }
    private function isSepehr() #this method is not Node
    {
        return $this->name == 'Sepehr';
    }
}
```
> **Note**
> Only public methods can be routed. and public properties automatically reassigned to the stage. if you don't want to automatically reassign a property private it. and if your method is not Node private it. for improving your database performance.

