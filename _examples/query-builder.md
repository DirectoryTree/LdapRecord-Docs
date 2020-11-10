```php
public function index()
{
    $users = User::in('ou=office,dc=local,dc=com')
                ->whereCompany('ACME')
                ->whereMemberOf('cn=Accounting,dc=local,dc=com')
                ->get();

    return view('ldap.users.index', ['users' => $users]);
}
```