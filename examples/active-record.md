```php
$user = User::create([
    'company'        => 'LdapRecord',
    'title'          => 'Web Developer',
    'password'       => 'P@ssw0rd',
    'samaccountname' => 'sbauman',
    'mail'           => 'sbauman@acme.org',
    'givenname'      => 'Steve',
    'sn'             => 'Bauman',
]);
```