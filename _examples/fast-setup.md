```php
use LdapRecord\Connection;

$connection = new Connection([
	'hosts' => ['127.0.0.1'],
  	'base_dn' => 'dc=local,dc=com',
	'username' => 'cn=user,dc=local,dc=com',
  	'password' => 'SuperSecret',
]);

$connection->connect();

$users = $connection->query()
                    ->where('title', '=', 'Accountant')
                    ->get();
```
