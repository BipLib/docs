# Bip Library Documentation

- [Introduction](#introduction)
- [Getting Started](#getting-started)
  - [Installation](#installation)
  - [Run Sample bot](#run-sample-bot)
  - [Create your own bot](#create-your-own-bot)
- [Components](#components)
  - [Bot](#bot)
  - [Stage](#stage)
  - [Node](#node)
  - [Routing System](#routing-system)
  - [Telegram Bot API](#telegram-bot-api)
  - [Webhook](#webhook)
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
in `bots/NotePad` there is a sample [NotePad](https://github.com/BipLib/bip/tree/main/bots/NotePad) bot that you can run it and see how it works.
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
    private function isSepehr() #this method is not Node (because it is private)
    {
        return $this->name == 'Sepehr';
    }
}
```
> **Note**
> Only public methods can be routed. and public properties automatically reassigned to the stage. if you don't want to automatically reassign a property private it. and if your method is not Node private it. for improving your database performance.

<a name="components"></a>
## Components
Bip has some components that you can use them in your bot. for example, you can use [Logger](#logger) to log your bot actions, or use [Webhook](#webhook) to get webhook data with Autocomplete IDE.

<a name="stage"></a>
### Stage
Stage is a class that contains [nodes](#node), [controller](#controller), [default](#default) method and [properties](#properties). that we will explain it in the following sections.
You can consider a stage is specific part of your bot. suppose you have a bot that has a specific part for shopping and another part for chatting you can create `ShopStage` and `ChatStage` class and use them in your bot.
Stage classes should be in `Stages/` directory of your bot.
for example in NotePad bot you can find `StartStage` classes in `Stages/` directory.

### Creating Stages :
in `Stages/` (or anything else) directory of your project create a php class file for your stage (file name should be `PascalCase` and The file name **must** be the same as the class name)
for example we create `MenuStage.php` file in `Stages/` directory and add this code to it :


```php
<?php
namespace Bot\<STAGE_DIRECTORY>; #read following note for more information

use Bip\App\Stage;

class MenuStage extends Stage
{
    public function controller()
    {
        route('menu')->onMessageText('/menu');
    }
    public function menu()
    {
        msg('This is menu');
        closeNode();
    }
    
}
```
and extends your stage from `Bip\App\Stage` class.

> **Note**
> for psr-4 autoload your stages you must add namespace for your stage classes. stage namespaces start with `namespace Bot\<STAGE_DIRECTORY>;` that `<STAGE_DIRECTORY>` is the path of stage directory. for example in NotePad bot `bots/NotePad/Stages/StartStage.php` stage has `namespace Bots\NotePad\Stages;`.

> **Note**
> `Bots` namespace point to `bots` directory in Bip root directory. you can change it in `composer.json` file.

<a name="node"></a>
### Node :
Nodes are **public** methods that are called when a specific event occurs. for example when a user sends a message to your bot, a node is called. a node is state of user for example we want to get two number from user and multiply them. we can create a node for getting first number and another node for getting second number and multiply them in a third node. for example :

```php
...
class CalculatorStage extends Stage
{
    #public properties are automatically reassigned to the stage
    public int $firstNumber = 0; 
    public int $secondNumber = 0;
    
    public function controller()
    {
        route('getFirstNumber')->onMessageText('/start');
    }
    #public methods are nodes
    public function getFirstNumber() 
    {
        msg('Enter first number');
        bindNode('saveFirstNumber');
    }
    public function saveFirstNumber()
    {
        $this->firstNumber = Webhook::getObject()->message->text;
        msg('Enter second number');
        bindNode('saveSecondNumber');
    }
    public function saveSecondNumber()
    {
        $this->secondNumber = Webhook::getObject()->message->text;
        msg("Result is : ".$this->firstNumber * $this->secondNumber);
        closeNode();
    }
}
```
> **Note**
> Only public methods can be routed. and public properties automatically reassigned to the stage. if you don't want to automatically reassign a property private it. and if your method is not Node private it. for improving your database performance.

<a name="controller"></a>
<a name="routing-system"></a>
### Controller and Routing System :
controller is **public** method that is called first of any node in each run.
you can use `route` method to route a node to a method. for example :

```php
...
public function controller()
{
    route('start')->onMessageText('/start');
    route('welcome')->onMessageText('/menu');
    route('profile')->onMessageText('Profile');
}
...
``` 

<a name="properties"></a>
#### Properties :
public properties in stage classes are automatically reassigned to the stage. it means that they are saved in database and in new requests they are reassigned to the stage. for example :

```php
...
class CounterStage extends Stage
{
    public int $counter = 0;
    public function controller()
    {
        route('increment')->onMessageText('/increment');
    }
    public function increment()
    {
        $this->counter++;
        msg("Counter is : $this->counter");
        closeNode();
    }
}
...
```
`CounterStage` is a stage that has a counter property. when a user sends `/increment` command to bot, the counter property is incremented and reassigned to the stage. so when the user sends `/increment` command again the counter property is incremented again and reassigned to the stage. and so on.
> **Note**
> non-public properties are not reassigned to the stage.

> **Note**
> highly recommended to use data type for properties. STM (Stage-Table Mapper) use data types for adding columns to the table. for example if you have a property that is `int` type, STM will add a column to the table with `int` data type. if you don't use data type for properties, STM will add a column to the table with `text` data type.

<a name="default"></a>
#### Default :
default method is a method that is called when no node is routed. for example :

```php
...
class CalculatorStage extends Stage
{
    public function controller()
    {
        route('getFirstNumber')->onMessageText('/start');
    }
    public function getFirstNumber() 
    {
        msg('Enter first number');
        bindNode('saveFirstNumber');
    }
    public function saveFirstNumber()
    {
        $this->firstNumber = Webhook::getObject()->message->text;
        msg('Enter second number');
        bindNode('saveSecondNumber');
    }
    public function saveSecondNumber()
    {
        $this->secondNumber = Webhook::getObject()->message->text;
        msg("Result is : ".$this->firstNumber * $this->secondNumber);
        closeNode();
    }
    public function default()
    {
        msg('Invalid command');
    }
}
```
when a user sends a message that is not routed to any node, the default method is called. for example if a user sends `/start` command to bot, the `getFirstNumber` node is called. but if a user sends a message that is not routed to any node, the `default` method is called.
> **Note**
> Bip Garbage Collector will delete `$this->firstNumber` and `$this->secondNumber` properties after the `saveSecondNumber` stage run ends and will not reassign them to the stage in next run.
