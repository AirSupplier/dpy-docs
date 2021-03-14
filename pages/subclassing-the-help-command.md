---
layout: default
title: Help Command
description: Walkthrough guide on subclassing HelpCommand
---

# A basic guide on subclassing HelpCommand
This guide will walkthrough the ways to create a custom help command by subclassing HelpCommand.


## Brief explanation of subclassing
[`Skip ahead if you already know this.`](#start)

In simple terms, a subclass is a way to inherit a class behaviour/attributes from another class. Here's how you would subclass
a class in Python.
```py
class A:
    def __init__(self, attribute1):
        self.attribute1 = attribute1

    def method1(self):
        print("method 1")

    def method2(self):
        print("method 2")


class B(A):
    def __init__(self, attribute1, attribute2):
        super().__init__(attribute1) # This calls A().__init__ magic method.
        self.attribute2 = attribute2

    def method1(self): # Overrides A().method1
        print("Hi")
```
Given the example, the variable `instance_a` contains the instance of class `A`. As expected, the output will be this.
```py
>>> instance_a = A(1)
>>> instance_a.attribute1
1
>>> instance_a.method1()
"method 1"
```
How about `B` class? `instance_b` contains the instance of class `B`, it inherits attributes/methods from class `A`. Meaning
it will have everything that class `A` have, but with an additional attribute/methods.
```py
>>> instance_b = B(1, 2)
>>> instance_b.attribute1
1
>>> instance_b.attribute2
2
>>> instance_b.method1()
"Hi"
>>> instance_b.method2()
"method 2"
```
Make sure to look and practice more into subclassing classes to understand fully on what it is before diving into subclassing 
HelpCommand.

## <a name="start"></a>Getting started
With that out of the way, let's get started. For subclassing HelpCommand, first, you would need to know the types of `HelpCommand`. 
Where each class has their own usage.
### Types of HelpCommand class
There are a few types of HelpCommand classes that you can choose;
1. [`DefaultHelpCommand`][defhelplink] a help command that is given by default.
2. [`MinimalHelpCommand`][minhelplink] a slightly better help command.
3. [`HelpCommand`][helplink] an empty class that is the base class for every HelpCommand you see. On its own, it will 
   not do anything.
   
By default, help command is using the class `DefaultHelpCommand`. This is stored in [`bot.help_command`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.Bot.help_command). This attribute 
will **ONLY** accept instances that subclasses `HelpCommand`. Here is how you were to use the `DefaultHelpCommand` instance.
```py
from discord.ext import commands
bot = commands.Bot(command_prefix="uwu ")
bot.help_command = commands.DefaultHelpCommand()

# OR

bot = commands.Bot(command_prefix="uwu ", help_command=commands.DefaultHelpCommand())
# Both are equivalent
```
#### Here's an example of what that looks like.
![img.png](https://media.discordapp.net/attachments/784735050071408670/787570202593460224/unknown.png)
_Now, of course, this is done by default. I'm only showing you this as a demonstration. Don't scream at me_ 

Let's do the same thing with `MinimalHelpCommand` next.

```py
from discord.ext import commands
bot = commands.Bot(command_prefix="uwu ")
bot.help_command = commands.MinimalHelpCommand()
```
#### This is how that would look like:

![minhelpcommand.png](https://cdn.discordapp.com/attachments/784735050071408670/787571374742962228/unknown.png)

## Embed MinimalHelpCommand
Now say, you want the content to be inside an embed. But you don't want to change the content of
`DefaultHelpCommand`/`MinimalHelpCommand` since you want a simple HelpCommand with minimal work. There is a short code 
from `?tag embed help example` by `gogert` in [discord.py](https://discord.gg/dpy) server, a sample code you can follow 
shows this;
```py
import discord
from discord.ext import commands
bot = commands.Bot(command_prefix="uwu ")

class MyNewHelp(commands.MinimalHelpCommand):
    async def send_pages(self):
        destination = self.get_destination()
        for page in self.paginator.pages:
            emby = discord.Embed(description=page)
            await destination.send(embed=emby)

bot.help_command = MyNewHelp()
```
The resulting code will show that it have the content of `MinimalHelpCommand` but in an embed.

![embedminimalhelp.png](https://cdn.discordapp.com/attachments/784735050071408670/787574637117833236/unknown.png)

### How does this work?
Looking over the `MinimalHelpCommand` source code, [here](https://github.com/Rapptz/discord.py/blob/master/discord/ext/commands/help.py#L1083-L1339).
Every method that is responsible for `<prefix>help <argument>` will call [`MinimalHelpCommand.send_pages`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.MinimalHelpCommand.send_pages)
when it is about to send the content. This makes it easy to just override `send_pages` without having to override any
other method there are in `MinimalHelpCommand`.

# HelpCommand
## Basic methods to override
If you want to use `HelpCommand` class, we need to understand the basic of subclassing HelpCommand. 
Here are a list of HelpCommand relevant methods, and it's responsibility.

1. [`HelpCommand.send_bot_help(mapping)`][sendbothelp]
   Gets called with `<prefix>help`
2. [`HelpCommand.send_command_help(command)`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.HelpCommand.send_command_help)
   Gets called with `<prefix>help <command>`
3. [`HelpCommand.send_group_help(group)`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.HelpCommand.send_group_help)
   Gets called with `<prefix>help <group>`
4. [`HelpCommand.send_cog_help(cog)`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.HelpCommand.send_cog_help) 
   Gets called with `<prefix>help <cog>`
   
## Useful attributes
1. [`HelpCommand.context`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.HelpCommand.context)
   the Context object in the help command.
2. [`HelpCommand.clean_prefix`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.HelpCommand.clean_prefix)
   a cleanup prefix version that remove any mentions.

Seems simple enough? Now let's see what happens if you override one of the methods. Here's an example code of how you 
would do that. This override will say `"hello!"` when you type `<prefix>help` to demonstrate on what's going on. 

We'll use
[`HelpCommand.get_destination()`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.HelpCommand.get_destination)
to get the [`abc.Messageable`](https://discordpy.readthedocs.io/en/latest/api.html#discord.abc.Messageable)
instance for sending a message to the correct channel.

#### Code Example
```py
from discord.ext import commands
bot = commands.Bot(command_prefix="uwu ")

class MyHelp(commands.HelpCommand):
    async def send_bot_help(self, mapping):
        channel = self.get_destination()
        await channel.send("hello!")

bot.help_command = MyHelp()
```
#### Output

![hellohelpcommand.png](https://media.discordapp.net/attachments/784735050071408670/787584311221813248/unknown.png)

Keep in mind, using `HelpCommand` class will require overriding every `send_x_help` methods. For example, `<prefix>help jsk` is 
a command that should call `send_command_help` method. However, since `HelpCommand` is an empty class, it will not say 
anything.

## help command
Let's work our way to create a `<prefix> help`. 
Given the documentation, [`await send_bot_help(mapping)`][sendbothelp] method receives `mapping(Mapping[Optional[Cog], List[Command]])`
as its parameter. `await` indicates that it should be an async function.
### What does this mean?
* `Mapping[]` is a [`collections.abc.Mapping`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Mapping),
  for simplicity’s sake, this usually refers to a dictionary since it's under `collections.abc.Mapping`.
* `Optional[Cog]` is a [`Cog`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.Cog)
object that has a chance to be `None`.
* `List[Command]` is a list of [`Command`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.Command)
objects.
* `Mapping[Optional[Cog], List[Command]]` means it's a map object with `Optional[Cog]` as it's key and `List[Command]` as
its value.
  
_All of these are typehints in the `typing` module. You can learn more about it [here](https://docs.python.org/3/library/typing.html#module-typing)._

Now, for an example, we will use this `mapping` given in the parameter of `send_bot_help`. For each of the command, we'll
use [`HelpCommand.get_command_signature(command)`][Hgetcommandsig] to get the command signature of a command in an `str`
form.

#### Example Code
```py
import discord
from discord.ext import commands
bot = commands.Bot(command_prefix="uwu ")

class MyHelp(commands.HelpCommand):
    async def send_bot_help(self, mapping):
        embed = discord.Embed(title="Help")
        for cog, commands in mapping.items():
           command_signatures = [self.get_command_signature(c) for c in commands]
           if command_signatures:
                cog_name = getattr(cog, "qualified_name", "No Category")
                embed.add_field(name=cog_name, value="\n".join(command_signatures), inline=False)

        channel = self.get_destination()
        await channel.send(embed=embed)

bot.help_command = MyHelp()
```
### How does it work?
1. Create an embed.
2. Use dict.items() to get an iterable of `(Cog, list[Command])`.
3. Each element in `list[Command]`, we will call `self.get_command_signature(command)` to get the proper signature of 
   the command.
4. If the list is empty, meaning, no commands is available in the cog, we don't need to show it, hence 
   `if command_signatures:`.
5. `cog` has a chance to be `None`, this refers to No Category. We'll use [`getattr`](https://docs.python.org/3/library/functions.html#getattr)
   to avoid getting an error to get cog's name through [`Cog.qualified_name`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.Cog.qualified_name).
6. Using [`str.join`](https://docs.python.org/3/library/stdtypes.html#str.join) each command will be displayed on a separate line.
7. Once all of this is finished, display it.

#### The result

![samplehelp.png](https://cdn.discordapp.com/attachments/784735050071408670/787610952690040852/unknown.png)

I agree, this does not look pretty. But this demonstrate how to create a help command with minimal effort. The reason why
it looks ugly, is that we're using [`HelpCommand.get_command_signature(command)`][Hgetcommandsig].
Let's override that method to make it a little more readable.

We'll borrow codes from [`MinimalHelpCommand.get_command_signature`](https://github.com/Rapptz/discord.py/blob/v1.5.1/discord/ext/commands/help.py#L1144-L1145).
Optionally, we can subclass `MinimalHelpCommand` instead of copying codes. I'm doing this as a demonstration of 
overriding other methods.

We'll also use [`HelpCommand.filter_commands`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.HelpCommand.filter_commands),
this method will filter commands by removing any commands that the user cannot use. It is a handy method to use.

#### The Example
```py
import discord
from discord.ext import commands
bot = commands.Bot(command_prefix="uwu ")

class MyHelp(commands.HelpCommand):
   def get_command_signature(self, command):
        return '%s%s %s' % (self.clean_prefix, command.qualified_name, command.signature)

    async def send_bot_help(self, mapping):
        embed = discord.Embed(title="Help")
        for cog, commands in mapping.items():
           filtered = await self.filter_commands(commands, sort=True)
           command_signatures = [self.get_command_signature(c) for c in filtered]
           if command_signatures:
                cog_name = getattr(cog, "qualified_name", "No Category")
                embed.add_field(name=cog_name, value="\n".join(command_signatures), inline=False)

        channel = self.get_destination()
        await channel.send(embed=embed)

bot.help_command = MyHelp()
```
#### The resulting output

![betterhelpcommand.png](https://cdn.discordapp.com/attachments/784735050071408670/787614011759919114/unknown.png)

This looks more readable than the other one. While this should cover most of your needs, you may want to know more helpful
attribute that is available on `HelpCommand` in the official documentation.

## help [argument] command
Now that the hard part is done, let's take a look at `<prefix>help [argument]`. The method responsible for this is as 
follows;
1. `send_command_help`
2. `send_cog_help`
3. `send_group_help`

As a demonstration, let's go for `send_command_help` this method receive a `Command` object. For this, it's simple, all 
you have show is the attribute of the command.

For example, this is your command code, your goal is you want to show the `help`,`aliases` and the `signature`.
#### Command Code
```py
@bot.command(help="Shows all bot's command usage in the server on a sorted list.",
             aliases=["br", "brrrr", "botranks", "botpos", "botposition", "botpositions"])
async def botrank(ctx, bot: discord.Member):
   pass
```

Then it's simple, you can display each of the attribute by [`Command.help`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.Command.help)
and [`Command.aliases`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.Command.aliases).

For the signature, instead of using the previous `get_command_signature`, we're going to subclass `MinimalHelpCommand`.

#### Help Code

```py
import discord
from discord.ext import commands
bot = commands.Bot(command_prefix="uwu ")

class MyHelp(commands.MinimalHelpCommand):
    async def send_command_help(self, command):
        embed = discord.Embed(title=self.get_command_signature(command))
        embed.add_field(name="Help", value=command.help)
        alias = command.aliases
        if alias:
            embed.add_field(name="Aliases", value=", ".join(alias), inline=False)

        channel = self.get_destination()
        await channel.send(embed=embed)

bot.help_command = MyHelp()
```
#### What you get

![commandhelpcommand.png](https://cdn.discordapp.com/attachments/784735050071408670/787629706358292490/unknown.png)

As you can see, it is very easy to create `<prefix>help [argument]`. The class already handles the pain of checking whether
the given argument is a command, a cog, or a group command. It's up to you on how you want to display it, whether it's through 
a plain message, an embed or even using [`discord.ext.menus`](https://github.com/Rapptz/discord-ext-menus).

## Command Attributes
Let's say, someone is spamming your help command. For a normal command, all you have to do to combat this is using a 
[`cooldown`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.cooldown) decorator 
and slap that thing above the command declaration. Or, what about if you want an alias? Usually, you would put an 
aliases kwargs in the [`command`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.Bot.command)
decorator. However, HelpCommand is a bit special, It's a god damn class. You can't just put a decorator on it and expect 
it to work. 


That is when [`HelpCommand.command_attrs`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.HelpCommand.command_attrs)
come to the rescue. This attribute can be set during the HelpCommand declaration, or a direct attribute assignment. 
According to the documentation, it accepts exactly the same thing as a [`command`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.Bot.command)
decorator in a form of a dictionary.

For example, we want to rename the help command as `"hell"` instead of `"help"` for whatever reason. We also want to make
an alias for `"help"` so users can call the command with `"hell"` and `"help"`. Finally, we want to put a cooldown,
because help command messages are big, and we don't want people to spam those. So, what would the code look like?

#### Example Code
```py
from discord.ext import commands

attributes = {
   'name': "hell",
   'aliases': ["help", "helps"],
   'cooldown': commands.Cooldown(2, 5.0, commands.BucketType.user)
}

# During declaration
help_object = commands.MinimalHelpCommand(command_attrs=attributes)

# OR through attribute assignment
help_object = commands.MinimalHelpCommand()
help_object.command_attrs = attributes

bot = commands.Bot(command_prefix="uwu ", help_command=help_object)
```
### How does it work?
1. sets the name into `"hell"` is refers to here `'name': "hell"`.
2. sets the aliases by passing the list of `str` to the `aliases` key, which refers to here `'aliases': ["help", "helps"]`. 
3. sets the cooldown through the `"cooldown"` key by passing in a [`Cooldown`](https://github.com/Rapptz/discord.py/blob/v1.5.1/discord/ext/commands/cooldowns.py#L70-L134)
   object. This object will make a cooldown with a rate of 2, per 5 with a bucket type [`BucketType.user`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.discord.ext.commands.BucketType),
   which in simple terms, for every [`discord.User`](https://discordpy.readthedocs.io/en/latest/api.html#discord.User), 
   they can call the command twice, every 5 seconds.
4. We're going to use `MinimalHelpCommand` as the `HelpCommand` object.

#### The result
![cooldownhelp.png](https://cdn.discordapp.com/attachments/784735050071408670/789471505145659412/unknown.png)

As you can see, the name of the help command is now `"hell"`, and you can also trigger the help command by `"help"`. It 
will also raise an [`OnCommandCooldown`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.CommandOnCooldown)
error if it was triggered 3 times in 5 seconds due to our `Cooldown` object. Of course, I didn't show that in the result,
but you can try it yourself. You should handle the error in an error handler when an `OnCommandCooldown` is raised.

## Error handling for HelpCommand
### Basic error message override
What happens when `<prefix>help command` fails to get a command/cog/group? Simple, `HelpCommand.send_error_message` will
be called. HelpCommand will not call `on_command_error` when it can't find an existing command. It will also give you an 
`str` instead of an error instance.

```py
import discord
from discord.ext import commands
bot = commands.Bot(command_prefix="uwu ")

class MyHelp(commands.HelpCommand):
    async def send_error_message(self, error):
        embed = discord.Embed(title="Error", value=error)
        channel = self.get_destination()
        await channel.send(embed=embed)

bot.help_command = MyHelp()
```
`error` is a string that will only contain the message, all you have to do is display the message.

#### The output:

![errorhelp.png](https://cdn.discordapp.com/attachments/784735050071408670/787639990515269662/unknown.png)

### How about a local error handler?

Indeed, we have it. [`HelpCommand.on_help_command_error`](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.HelpCommand.on_help_command_error),
this method is responsible for handling any error just like any other local error handler.

#### Code
```py
import discord
from discord.ext import commands
bot = commands.Bot(command_prefix="uwu ")

class MyHelp(commands.HelpCommand):
    async def send_bot_help(self, mapping):
        raise commands.BadArgument("Something broke")

    async def on_help_command_error(self, ctx, error):
        if isinstance(error, commands.BadArgument):
            embed = discord.Embed(title="Error", description=str(error))
            await ctx.send(embed=embed)
        else:
            raise error

bot.help_command = MyHelp()
```
Rather basic, just raise an error that subclasses `commands.CommandError` such as `commands.BadArgument`. The error raised 
will cause `on_help_command_error` to be invoked. The code shown will catch this `commands.BadArgument` instance that is 
stored in `error` variable, and show the message.

#### Output:

![errorhandler.png](https://cdn.discordapp.com/attachments/784735050071408670/787645587004588052/unknown.png)

To be fair, you should create a proper error handler through this official documentation. [Here](https://discordpy.readthedocs.io/en/latest/ext/commands/commands.html#ext-commands-error-handler).

There is also a lovely example by `Mysty` on error handling in general. [Here](https://gist.github.com/EvieePy/7822af90858ef65012ea500bcecf1612).
This example shows how to properly create a global error handler and local error handler.
## The end
I hope that reading this walkthrough will assist you and give a better understanding on how to subclass [`HelpCommand`][helplink]. 
All the example code given are to demonstrate the feature of `HelpCommand` and feel free to try it. There are lots of 
creative things you can do to create a `HelpCommand`. 

#### Here is how my help command looks like as an inspiration.
![myhelp.png](https://cdn.discordapp.com/attachments/784735050071408670/789480296055177226/unknown.png)

Here's my code if you want to take a look. [my help command](https://github.com/InterStella0/stella_bot/blob/master/cogs/helpful.py).

Now, of course, any question regarding `HelpCommand` should be asked in the [discord.py](https://discord.gg/dpy) server 
because I don't really check this gist as much, and because there is a lot of helpful discord.py helpers if you're nice 
enough to them. :D


[defhelplink]: https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.DefaultHelpCommand
[minhelplink]: https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.MinimalHelpCommand
[helplink]: https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.HelpCommand
[sendbothelp]: https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.HelpCommand.send_bot_help
[Hgetcommandsig]: https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.HelpCommand.get_command_signature

# Credits
Credits to [Stella](https://github.com/InterStella0)
