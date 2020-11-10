```php
$user = User::create([
    'company' => 'Acme',
    'cn' => 'Steve Bauman',
    'password' => 'P@ssw0rd',
]);

$administrators = Group::find('cn=Admins,dc=local,dc=com');

$user->groups()->attach($administrators);
```