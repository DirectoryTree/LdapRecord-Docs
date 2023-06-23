---
title: Events
description: Listening for LDAP events in LdapRecord-Laravel
---

# Events

## Introduction

LdapRecord includes a robust event dispatcher that allows you to listen for various
events that occur, such as authentication and object creation / modification.

For example, you may wish to send a notification when an LDAP object is modified.
You can listen for the model `Saved` event and then send an email regarding the change.

## Creating the Listener

To get started, we will create an event listener in the `app/Ldap/Listeners`
directory and create a new file named `ObjectModified.php`. This will
contain a class that will listen for the `Saved` model event.

> You will have to create the `Ldap` and `Listeners` subdirectories.

```php
<?php

namespace App\Ldap\Listeners;

use LdapRecord\Models\Events\Saved;
use Illuminate\Support\Facades\Mail;

class ObjectModified
{
    public function handle(Saved $event)
    {
        $objectName = $event->getModel()->getName();

        // Send an email when the object has been modified.
        Mail::raw("Object [$objectName] has been modified.", function ($message) {
            $message->from('notifier@company.com', 'LDAP Notifier');
            $message->to('it-support@company.com');
            $message->subject('LDAP Object Modified');
        });
    }
}
```

For a list of all LdapRecord events, view the core [event's documentation](/docs/core/v3/events#list-of-events).

## Creating the Service Provider

Next, we will create a new Laravel service provider. This is where we will register our LDAP event
listeners. We will call it `LdapEventServiceProvider`. Execute the below command to generate it:

```bash
php artisan make:provider LdapEventServiceProvider
```

Add the provider to your `config/app.php` configuration file:

```php
// config/app.php

return [
    // ...

    'providers' => [
        // ...
        \App\Providers\LdapEventServiceProvider::class,
    ],
];
```

Then, in the generated provider we will update it to the following:

```php
<?php

namespace App\Providers;

use LdapRecord\Container;
use Illuminate\Support\ServiceProvider;

class LdapEventServiceProvider extends ServiceProvider
{
    /**
     * The LDAP event listener mappings for the application.
     *
     * @return array
     */
    protected $listen = [
        \LdapRecord\Models\Events\Saved::class => [
            \App\Ldap\ObjectModified::class
        ],
    ];

    /**
     * Register the application LDAP event listeners.
     *
     * @return void
     */
    public function boot()
    {
        $dispatcher = Container::getDispatcher();

        foreach ($this->listen as $event => $listeners) {
            foreach (array_unique($listeners) as $listener) {
                $dispatcher->listen($event, $listener);
            }
        }
    }
}
```

> We've removed the `register` method in the above generated class. We won't need it here.

As you can see above, we can add LdapRecord events to the `$listen` property as the key, and
the listeners as the value. This allows you to attach mulitple listeners to the same event.
