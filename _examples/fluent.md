```php
User::in('ou=office,dc=local,dc=com')
    ->whereEnabled()
    ->whereMemberOf('cn=managers,ou=groups,dc=local,dc=com')
    ->whereNotContains('company', 'acme')
    ->get()
    ->each(function ($user) {
        $user->company = 'Acme Organization';
        
        $user->save();
    });
```