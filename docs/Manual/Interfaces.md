# Interfaces

Interfaces in the Kanal ecosystem are special objects that inherit `Kanal::Core::Interfaces::Interface`
and provide input/output management for router with... something from outside :)

What does it mean? Well, we have core, router, input, output, condition etc but...

What does spawn an input for router? Where goes output after router created it?

The anwer is: Kanal interfaces!

Interfaces usually contain code that connects your router to some external service, library, etc.

For example, there can be `TelegramInterface`. (actually it does already exists, you can use it, read more in Ecosystem part of documentation)

# Possible manual for interfaces out there

All interfaces should inherit the core interface class of the Kanal: `Kanal::Core::Interfaces::Interface`.

(more about that in interface development section)

Meanin that they should have basic configuration values passed in constructor or
in some method (e.g. bot_token in Telegram) and method `start`.

They also require core, provided by you.

Often interfaces have some plugins for Kanal, written specifically for external service in use in those interfaces.

Meaning core you provided would be used to register those plugins, as well as input parameters, output parameters.


`start` method should be the starting point for interfaces. For example for TelegramInterface, `start`
method would connect to telegram api, using additional library for this (e.g. telegram-bot-ruby).

After connecting, interface would get messages from telegram and create input for router, with
registered for this parameters.

After getting output from your router, interface would use telegram library to send router output,
converting it into compatible response for telegram library.
