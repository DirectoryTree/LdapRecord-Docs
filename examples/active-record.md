```php
$user = new User();

$user->cn = 'Steve Bauman';
$user->title = 'Developer';
$user->samaccountname = 'sbauman';
$user->company = 'Acme';
$user->mail = 'sbauman@acme.org';
$user->givenname = 'Steve';
$user->sn = 'Bauman';
$user->password = 'Super-Secret';

$user->save();
```