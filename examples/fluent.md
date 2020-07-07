```php
User::whereEnabled()
    ->whereMemberOf('cn=Managers,ou=Groups,dc=local,dc=com')
    ->whereNotContains('company', 'Acme')
    ->get()
    ->each(function ($user) {
        $user->company = 'Acme Organization';
        
        $user->save();
    });
```