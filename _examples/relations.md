```php
$user = User::find('cn=John Doe,dc=local,dc=com');

$groups = $user->groups()->get();

foreach ($groups as $group) {
    echo $group->getName() . PHP_EOL;
}
```