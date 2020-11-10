```php
use LdapRecord\Models\Model;

class User extends Model
{
    protected $objectClasses = [
        'top',
        'person',
        'organizationalperson',
        'user',
    ];
}
```