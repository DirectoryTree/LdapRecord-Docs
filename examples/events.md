```php
$dispatcher = Container::getEventDispatcher();

$dispatcher->listen(Saved::class, function (Saved $event) {
    Mail::send(
        new LdapObjectModified($event->getModel())
    );
});
```