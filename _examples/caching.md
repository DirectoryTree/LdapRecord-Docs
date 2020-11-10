```php
$connection = Container::getDefaultConnection();

$connection->setCache($myAppCache);

$until = new \DateTime('tomorrow');

$users = User::cache($until)->get();
```