---
title: Events
description: Listening to events in LdapRecord
---

# Events

## Introduction

LdapRecord events provide a method of listening for certain LDAP actions
that are called and execute tasks for that specific event.

The LdapRecord event dispatcher was actually derived from the
[Laravel Framework](https://github.com/laravel/framework) with
Broadcasting & Queuing omitted to remove extra dependencies
that would be required with implementing those features.

If you've worked with Laravel's event system before, this will feel very familiar.

## Registering Listeners

Before we get to registering listeners, it's crucial to know that events throughout
LdapRecord are fired irrespective of the current connection or provider in use.

This means that when using multiple LDAP connections, the same events will be fired. This
allows you to set listeners on events that occur for all LDAP connections you utilize.

If you are required to determine which events are fired from alternate connections, see [below](#determining-the-connection).

To register a listener on an event, retrieve the event dispatcher and call the `listen()` method:

```php
use LdapRecord\Container;
use LdapRecord\Auth\Events\Binding;

$dispatcher = Container::getDispatcher();

$dispatcher->listen(Binding::class, function (Binding $event) {
    $event->connection; // LdapRecord\Connections\Ldap instance
    $event->username; // 'jdoe@acme.org'
    $event->password; // 'super-secret'
});
```

The first argument is the event name you would like to listen for, and the
second is either a closure or class name that should handle the event.

### Using a Class Based Listener

> When using just a class name, the class must contain a public `handle()` method that will handle the event.

```php
use LdapRecord\Container;
use LdapRecord\Auth\Events\Binding;

$dispatcher = Container::getDispatcher();

$dispatcher->listen(Binding::class, MyApp\BindingEventHandler::class);
```

```php
namespace MyApp;

use LdapRecord\Auth\Events\Binding;

class BindingEventHandler
{
    public function handle(Binding $event): void
    {
        // Handle the event...
    }
}
```

## Model Events

Model events are handled the same way as authentication events.

Simply call the event dispatcher `listen()` method with the model event you are wanting to listen for:

```php
use LdapRecord\Container;
use LdapRecord\Models\Events\Saving;

$dispatcher = Container::getDispatcher();

$dispatcher->listen(Saving::class, function (Saving $event) {
    // Returns the model instance being saved,
    // eg. `LdapRecord\Models\Entry`
    $event->getModel();
});
```

## Wildcard Event Listeners

You can register listeners using the `*` as a wildcard parameter to catch multiple events with the same listener.

Wildcard listeners will receive the event name as their first argument, and the entire event data array as their second argument:

```php
use LdapRecord\Container;

$dispatcher = Container::getDispatcher();

// Listen for all model events.
$dispatcher->listen('LdapRecord\Models\Events\*', function ($eventName, array $data) {
    // Returns 'LdapRecord\Models\Events\Updating'
    echo $eventName;

    // Returns [0] => (object) LdapRecord\Models\Events\Updating;
    var_dump($data);
});

$connection = Container::getDefaultConnection();

$user = $connection->query()->find('cn=User,dc=local,dc=com');

$user->company = 'New Company';

$user->save();
```

## Determining the Connection

If you're using multiple LDAP connections and you require the ability to determine which events belong
to a certain connection, you can do so by verifying the host of the LDAP connection.

Here's an example:

```php
use LdapRecord\Container;
use LdapRecord\Models\Events\Creating;

$dispatcher = Container::getDispatcher();

$dispatcher->listen(Creating::class, function ($event) {
    $connection = $event->model->getConnection();

    $host = $connection->getHost();

    echo $host; // Displays 'ldap://192.168.1.1:386'
});
```

Example with authentication events:

```php
use LdapRecord\Container;
use LdapRecord\Auth\Events\Binding;

$dispatcher = Container::getDispatcher();

$dispatcher->listen(Binding::class, function ($event) {
    $connection = $event->connection;

    $host = $connection->getHost();

    echo $host; // Displays 'ldap://192.168.1.1:386'
});
```

## List of Events

### Authentication Events

There are several events that are fired during initial and subsequent binds
to your configured LDAP server. Here is a list of all events that are fired:

<table>
    <thead>
        <tr>
            <th>Event</th>
            <th>Fired</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td markdown="1"><code>LdapRecord\Auth\Events\Attempting</code></td>
            <td>
                When any authentication attempt is called via:
                <ul><li markdown="1"><code>$connection->auth()->attempt()</code></li></ul>
            </td>
        </tr>
        <tr>
            <td markdown="1"><code>LdapRecord\Auth\Events\Passed</code></td>
            <td>
                When any authentication attempts pass via:
                <ul><li markdown="1"><code>$connection->auth()->attempt()</code></li></ul>
            </td>
        </tr>
        <tr>
            <td markdown="1"><code>LdapRecord\Auth\Events\Failed</code></td>
            <td>
                When any authentication attempts fail via:
                <ul>
                    <li markdown="1"><code>$connection->auth()->attempt()</code></li>
                    <li markdown="1"><code>$connection->auth()->bind()</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td markdown="1"><code>LdapRecord\Auth\Events\Binding</code></td>
            <td>
                When any LDAP bind attempts occur via:
                <ul>
                    <li markdown="1"><code>$connection->auth()->attempt()</code></li>
                    <li markdown="1"><code>$connection->auth()->bind()</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td markdown="1"><code>LdapRecord\Auth\Events\Bound</code></td>
            <td>
                When any LDAP bind attempts are successful via:
                <ul>
                    <li markdown="1"><code>$connection->auth()->attempt()</code></li>
                    <li markdown="1"><code>$connection->auth()->bind()</code></li>
                </ul>
            </td>
        </tr>
    </tbody>
</table>

### Model Events

There are several events that are fired during the creation, updating and deleting of all models.

Here is a list of all events that are fired:

<table>
    <thead>
        <tr>
            <th>Event</th>
            <th>Fired</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td markdown="1"><code>LdapRecord\Models\Events\Saving</code></td>
            <td>
                When a model is in the process of being saved via:
                <ul>
                    <li markdown="1"><code>$model->save()</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td markdown="1"><code>LdapRecord\Models\Events\Saved</code></td>
            <td>
                When a model has been successfully saved via:
                <ul>
                    <li markdown="1"><code>$model->save()</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td markdown="1"><code>LdapRecord\Models\Events\Creating</code></td>
            <td>
                When a model is being created via:
                <ul>
                    <li markdown="1"><code>$model->save()</code></li>
                    <li markdown="1"><code>$model->create()</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td markdown="1"><code>LdapRecord\Models\Events\Created</code></td>
            <td>
                When a model has been successfully created via:
                <ul>
                    <li markdown="1"><code>$model->save()</code></li>
                    <li markdown="1"><code>$model->create()</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td markdown="1"><code>LdapRecord\Models\Events\Updating</code></td>
            <td>
                When a model is being updated via:
                <ul>
                    <li markdown="1"><code>$model->save()</code></li>
                    <li markdown="1"><code>$model->update()</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td markdown="1"><code>LdapRecord\Models\Events\Updated</code></td>
            <td>
                When a model has been successfully updated via:
                <ul>
                    <li markdown="1"><code>$model->save()</code></li>
                    <li markdown="1"><code>$model->update()</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td markdown="1"><code>LdapRecord\Models\Events\Deleting</code></td>
            <td>
                When a model is being deleted via:
                <ul>
                    <li markdown="1"><code>$model->delete()</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td markdown="1"><code>LdapRecord\Models\Events\Deleted</code></td>
            <td>
                When a model has been successfully deleted via:
                <ul>
                    <li markdown="1"><code>$model->delete()</code></li>
                </ul>
            </td>
        </tr>
    </tbody>
</table>
