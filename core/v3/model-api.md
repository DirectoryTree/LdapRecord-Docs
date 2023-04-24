---
title: Model API
description: A list of all available LdapRecord model methods.
---

# Available Model Methods (API)

## Method Listing

#### `addAttribute`

Immediately inserts a new attribute value on the model.

Performs an `ldap_mod_add` under the hood.

> **Important**: This does not create attributes that do not exist on your LDAP schema for the object.

```php
$model->addAttribute('telephonenumber', '+1 555 555 1717');
```

#### `addAttributeValue`

Add a value into an array of attribute values:

```php
$model->proxyaddresses = ['SMTP:sbauman@microsoft.com'];

$model->addAttributeValue('proxyaddresses', 'smtp:sbauman@local.com');

// Displays:
// [
//     'SMTP:sbauman@microsoft.com',
//     'smtp:sbauman@local.com'
// ]
var_dump($model->proxyaddresses);
```

#### `addHidden`

Add an attribute to hide when encoding a model using `json_encode`:

```php
$model->addHidden('userpassword');

$model->addHidden(['userpassword', 'mail']);

// 'userpassword' and 'mail' will be omitted:
$attributes = json_encode($model);
```

#### `addModification`

Add a batch modification to the model to be executed upon `save()`:

```php
// Using an array...
$modification = [
    'attrib'  => 'telephoneNumber',
    'modtype' => LDAP_MODIFY_BATCH_ADD,
    'values'  => ['+1 555 555 1717'],
];

$model->addModification($modification);

// Using a BatchModification...
$mod = new \LdapRecord\Models\BatchModification(
    $attrib = 'telephoneNumber',
    $modType = LDAP_MODIFY_BATCH_ADD,
    $values = ['+1 555 555 1717']
);

$model->addModification($mod);

$model->save();
```

#### `addVisible`

Add an attribute to include that is being hidden when encoding a model using `json_encode`:

```php
$model->addVisible('userpassword');

$model->addVisible(['userpassword', 'mail']);
```

#### `ancestors`

Begin querying the direct ancestors of the model:

```php
$ancestors = $model->ancestors()->get();
```

#### `asDateTime`

Convert an LDAP timestamp to a `Carbon\Carbon` instance:

```php
$carbon = $model->asDateTime('20200508184557Z', 'ldap');

$carbon = $model->asDateTime('20200508184533.0Z', 'windows');

$carbon = $model->asDateTime(132334371140000000, 'windows-int');
```

#### `attributesToArray`

Convert all the models attributes to their JSON encodable value:

```php
$attributes = $model->attributesToArray();
```

> **Important**: LDAP date attributes specified via the `$dates` model property will be converted in the returned array.

#### `convert`

Convert a model into another by copying its attributes, connection and distinguished name:

> **Important**: This will also set `$model->exists` property to `true` if the **model being converted** exists.

```php
$into = new \LdapRecord\Models\ActiveDirectory\User();

// Returns instance of \LdapRecord\Models\ActiveDirectory\User
$user = $model->convert($into);
```

#### `countAttributes`

Get the total number of attributes on a model:

> **Important**: This does not count attribute values.

```php
$model->fill([
    'cn' => 'Steve Bauman',
    'sn' => 'Bauman',
]);

// Returns '2'
$model->countAttributes();
```

#### `createAttribute`

This method was renamed to [addAttribute](#addattribute) in v3.0.

#### `delete`

Immediately delete an existing model instance:

```php
$model->delete();

// Returns false.
$model->exists;
```

To delete nested objects contained in the model recursively, pass in `true`:

```php
$model->delete($recursive = true);
```

#### `deleteAttributes`

This method was renamed to [removeAttributes](#removeattributes) in v3.0.

#### `descendants`

Begin querying the direct descendants of the model:

```php
$descendants = $model->descendants()->get();
```

#### `escape`

Prepare a value to be escaped:

```php
// Returns \LdapRecord\Models\Attributes\EscapedValue;
$escapedValue = $model->escape($value, $ignore = '', $flags = 0);

// Cast to string for immediate return of escaped value:
$escapedValue = (string) $model->escape($value, $ignore = '', $flags = 0);
```

#### `fill`

Fill a model with attributes and values:

```php
$model->fill([
    'cn' => 'Steve Bauman',
    'proxyAddresses' => ['foo', 'bar'],
]);

$model->save();
```

#### `fresh`

Get a fresh **new** instance of the existing model.

The model will be re-retrieved from the LDAP directory. The existing model will not be affected:

```php
$fresh = $model->fresh();
```

#### `fromDateTime`

Convert a `DateTime`, `Carbon` or `string` to the specified LDAP timestamp type:

```php
$date = new \DateTime('now');

$ldapTimestamp = $model->fromDateTime('ldap', $date);

$windowsTimestamp = $model->fromDateTime('windows', $date);

$windowsIntTimestamp = $model->fromDateTime('windows-int', $date);
```

#### `getAnrAttributes`

Get an array of ANR attributes defined on the model:

```php
$attributes = $model->getAnrAttributes();

// Displays:
// [
//    'cn',
//    'sn',
//    'uid',
//    'name',
//    'mail',
//    'givenname',
//    'displayname'
// ]
var_dump($attributes);
```

#### `getAppends`

Get the accessors being appended to the models array form:

```php
$model->setAppends(['foo-bar']);

// Displays:
// [
//    'foo-bar',
// ]
var_dump($model->getAppends());
```

#### `getAttribute`

Get the value of the specified attribute.

This will return an `array` if the attribute exists and `null` if non-existent:

> **Important**: If a mutator exists for the attribute (such as a date,
> or custom attribute [mutator method](//docs/core/v2/model-mutators/#defining-a-mutator)),
> it's value will be returned instead.

```php
// Returns array:
$values = $model->getAttribute('cn');

// Returns first value:
$value = $model->getAttribute('cn')[0];

// Returns null:
$null = $model->getAttribute('non-existent');
```

#### `getAttributeValue`

Performs the same as above.

#### `getAttributes`

Get all the models raw attributes:

> **Important**: Mutator attribute values will not be included in this array.

```php
$attributes = $model->getAttributes();

foreach ($attributes as $attribute => $values) {
    //
}
```

#### `getConnection`

Resolve the underlying `LdapRecord\Connection`:

```php
$connection = $model->getConnection();

$config = $connection->getConfiguration();
```

#### `getConnectionName`

Get the connection name from the model:

> **Important**: If no connection is set on the model, `null` will be returned.

```php
class Entry extends Model
{
    protected ?string $connection = 'domain-b';
}

// Returns 'domain-b'
$connectionName = (new Entry)->getConnectionName();
```

#### `getConvertedGuid`

Get the models string GUID:

```php
// Example: bf9679e7-0de6-11d0-a285-00aa003049e2
$guid = $model->getConvertedGuid();
```

#### `getCreatableDn`

Get the models creatable Distinguished Name:

```php
$model = new Entry(['cn' => 'John Doe']);

// Returns: 'cn=John Doe,dc=local,dc=com'
$creatableDn = $model->getCreatableDn();

$model->cn = 'Jane Doe';

// Returns: 'cn=Jane Doe,dc=local,dc=com'
$creatableDn = $model->getCreatableDn();
```

#### `getCreatableRdn`

Get the models creatable relative Distinguished Name:

```php
$model = new Entry(['cn' => 'John Doe']);

// Returns: 'cn=John Doe'
$creatableDn = $model->getCreatableRdn();

$model->cn = 'Jane Doe';

// Returns: 'cn=Jane Doe'
$creatableDn = $model->getCreatableRdn();
```

#### `getDateFormat`

Get the format that dates are serialized to:

```php
// Returns: 'Y-m-d\TH:i:sO'
$model->getDateFormat();
```

#### `getDates`

Get an array of attributes that should be mutated to dates:

```php
$dates = $model->getDates();

// Displays:
// [
//    'createtimestamp' => 'ldap',
//    'modifytimestamp' => 'ldap',
// ]
var_dump($dates);
```

#### `getDirty`

Get the attributes that have been changed:

```php
$model->cn = 'Changed';

foreach ($model->getDirty() as $attribute => $values) {
    // Displays: 'cn'
    echo $attribute;

    // Displays: ['Changed']
    var_dump($values);
}
```

#### `getDn`

Get the models Distinguished Name:

```php
// Displays: 'cn=John Doe,dc=local,dc=com'
echo $model->getDn();
```

#### `getFirstAttribute`

Get the first value of an attribute:

```php
$model->proxyaddresses = ['first', 'second'];

// Returns: 'first'
$value = $model->getFirstAttribute('proxyaddresses');
```

#### `getGlobalScopes`

Get the global scopes set on the model:

```php
Model::addGlobalScope('my-scope', function () {
    // ...
});

// Returns: ['my-scope' => \Closure]
$scopes = $model->getGlobalScopes();
```

#### `getGuidKey`

Get the attribute key that contains the objects GUID:

```php
// Returns: 'objectguid'
$model->getGuidKey();
```

#### `getHidden`

Get the hidden attributes for the model:

```php
$model->addHidden('cn');

// Returns: ['cn']
$model->getHidden();
```

#### `getModifications`

Get the models batch modifications to be processed:

```php
$model->cn = 'Changed';

// Returns:
// [
//      [
//      'attrib' => 'cn',
//      'modtype' => 1,
//      'values' => ['Changed'],
//     ]
// ]
$mods = $model->getModifications();
```

#### `getName`

Get the name of a model:

```php
$model->setDn('cn=John Doe,dc=local,dc=com');

// Returns: 'John Doe'
$name = $model->getName();
```

#### `getObjectGuid`

Get the raw binary object GUID:

> **Important**:
>
> -   The **raw binary** GUID will be returned when connecting to **Active Directory**.
> -   The **raw string** GUID will be returned with **other LDAP directories**.

```php
$rawBinary = $model->getObjectGuid();
```

#### `getOriginal`

Get the original, un-modified attributes on a model:

```php
$model = Model::findBy('cn', 'Steve Bauman');

$model->cn = 'Changed';

// Returns:
// [
//    'cn' => ['Steve Bauman']
//    ...
// ]
$model->getOriginal();
```

#### `getParentDn`

Get the direct parent Distinguished Name of a model:

```php
$model = Model::find('cn=John Doe,dc=local,dc=com');

// Returns: 'dc=local,dc=com'
$model->getParentDn();

// Returns: 'ou=Users,dc=local,dc=com'
$model->getParentDn('cn=Steve Bauman,ou=Users,dc=local,dc=com');
```

#### `getRawAttribute`

Get a model's raw value from an attribute:

```php
$timestamp = $model->getRawAttribute('modifytimestamp');
```

#### `getRdn`

Get the models Relative Distinguished Name:

```php
$model = Model::find('cn=John Doe,dc=local,dc=com');

// Returns: 'cn=John Doe'
$model->getRdn();

// Returns: 'cn=Steve Bauman'
$model->getRdn('cn=Steve Bauman,ou=Users,dc=local,dc=com');
```

#### `getVisible`

Get the attributes that should be visible when encoding a model using `json_encode`:

```php
$model->addVisible('cn', 'sn');

// Returns: ['cn', 'sn']
$visibleAttributes = $model->getVisible();
```

#### `hasAppended`

Determine if the model has an accessor attribute being appended:

```php
$model->setAppends(['foo-bar']);

// Returns: true
$model->hasAppended('foo-bar');
```

#### `hasAttribute`

Determine if the model has an attribute with a value:

```php
$model = Model::findBy('cn', 'Steve Bauman');

// Returns: true
$model->hasAttribute('cn');

// Returns: false
$model->hasAttribute('non-existent');
```

#### `hasGetMutator`

Determine if the model has a 'get' mutator for the given attribute:

```php
class Entry extends Model
{
    public function getCnAttribute(array $values): mixed
    {
        // ...
    }
}

$model = new Entry();

// Returns: true
$model->hasGetMutator('cn');
```

#### `hasSetMutator`

Determine if the model has a 'set' mutator for the given attribute:

```php
class Entry extends Model
{
    public function setCnAttribute($value): void
    {
        // ...
    }
}

$model = new Entry();

// Returns: true
$model->hasSetAttribute('cn');
```

#### `inside`

Set the container that the model should be **created** inside:

> **Important**: Calling `inside()` on an existing model will not perform any
> move / rename operation. Use [move](#move) or [rename](#rename) instead.

```php
$model = new Model();

// ...

$model->inside('ou=Container,dc=local,dc=com');

$model->save();
```

#### `is`

Determine if a model is the same by comparing their Distinguished Names and connections:

```php
// Returns: bool
$model->is($another);
```

#### `isAncestorOf`

Determine if a model is an ancestor of another:

```php
$user = User::find('cn=John Doe,ou=Accounting,ou=Accounts,dc=local,dc=com');
$ou = OrganizationalUnit::find('ou=Accounts,dc=local,dc=com');

// Returns: true
$ou->isAncestorOf($user);
```

#### `isChildOf`

Determine if a model is an **immediate** child of another:

```php
$ou = OrganizationalUnit::find('ou=Accounts,dc=local,dc=com');
$user = User::find('cn=John Doe,ou=Accounts,dc=local,dc=com');

// Returns: true
$user->isChildOf($ou);
```

#### `isDateAttribute`

Determine if given attribute is a date attribute on the model:

```php
class Entry extends Model
{
    protected array $dates = ['whenchanged' => 'windows'];
}

$model = new Entry();

// Returns: true
$model->isDateAttribute('whenchanged');
```

#### `isDescendantOf`

Determine if a model is a descendent of another:

```php
$ou = OrganizationalUnit::find('ou=Accounts,dc=local,dc=com');
$user = User::find('cn=John Doe,ou=Accounting,ou=Accounts,dc=local,dc=com');

// Returns: true
$user->isDescendantOf($ou);
```

#### `isDirty`

Determine if the given attribute has been changed:

```php
$model = Model::findBy('cn', 'Steve Bauman');

// Returns: false
$model->isDirty('cn');

$model->cn = 'Changed';

// Returns: true
$model->isDirty('cn');
```

#### `isParentOf`

Determine if a model is an **immediate** parent of another:

```php
$ou = OrganizationalUnit::find('ou=Accounts,dc=local,dc=com');
$user = User::find('cn=John Doe,ou=Accounts,dc=local,dc=com');

// Returns: true
$ou->isParentOf($user);
```

#### `makeHidden`

Make the given, typically visible, attributes hidden on the model:

```php
class Entry extends Model
{
    protected array $visible = ['cn'];
}

// This will override the above:
$model->makeHidden('cn');
```

#### `makeVisible`

Make the given, typically hidden, attributes visible on the model:

```php
class Entry extends Model
{
    protected array $hidden = ['cn'];
}

// This will override the above:
$model->makeVisible('cn');
```

#### `move`

Move a model into the specified container:

```php
$user = User::find('cn=Steve Bauman,dc=local,dc=com');

$ou = OrganizationalUnit::find('ou=Office Users,dc=local,dc=com');

$user->move($ou);

// Returns: 'cn=Steve Bauman,ou=Office Users,dc=local,d=com'
$user->getDn();
```

#### `newBatchModification`

Create a new `LdapRecord\Models\BatchModification` instance:

```php
// Returns instance of: BatchModification
$mod = $model->newBatchModification(
    'cn', LDAP_MODIFY_BATCH_ADD, ['Steve Bauman']
);
```

#### `newCollection`

Create a new `Tightenco\Collect\Support\Collection`:

```php
$collection = $model->newCollection($items = []);
```

#### `newInstance`

Create a **new** model instance:

```php
$model = Model::findByAnr('sbauman');

$new = $model->newInstance();
```

#### `newQuery`

Create a **new** model query:

> **Important**: Global scopes and object classes **will be applied** to this query.

```php
$results = $model->newQuery()->where('cn', 'contains', 'Steve')->get();
```

#### `newQueryWithoutScopes`

Create a new model query **without** global scopes:

> **Important**: Global scopes and object classes **will not be applied** to this query.

```php
$results = $model->newQueryWithoutScopes()->get();
```

#### `refresh`

Refresh a models attributes by re-retrieving them from your LDAP directory.

This will affect the current model instance:

```php
$model->refresh();
```

#### `removeAttribute`

Immediately remove an attribute on a model.

Performs an `ldap_mod_del` under the hood.

```php
// Remove one value from a single-value attribute:
$model->removeAttribute('telephonenumber');

// Remove all values from a multi-value attribute:
$model->removeAttribute('memberuid');
```

#### `removeAttributes`

Immediately remove multiple attributes on a model.

Performs an `ldap_mod_del` under the hood.

```php
// Removing one value from an attribute:
$model->removeAttribute(["memberuid" => "jdoe"]);

// Removing multiple values from an attribute:
$model->removeAttribute([
    "memberuid" => ["jdoe", "jsmith"]
]);

// Removing all values from an attribute:
$model->removeAttribute(["memberuid" => []]);
```

#### `rename`

Rename a model and keep its container location:

> **Important**: When renaming is successful, the models distinguished name is
> automatically updated to reflect its new name in the directory, so you may
> run further operations on it during the same request.

```php
$user = Model::find('cn=John Doe,dc=local,dc=com');

$user->rename('Jane Doe');

// Returns: 'cn=Jane Doe,dc=local,dc=com'
$user->getDn();
```

#### `replaceAttribute`

Immediately replaces the entire attribute value on the model.

Performs an `ldap_mod_replace` under the hood.

```php
$model->updateAttribute('proxyaddresses', ['foo', 'bar']);
```

#### `save`

Persist the new or existing model to the directory:

```php
// Save a new model:
$model = new Model(['cn' => 'John Doe']);

$model->save();

// Updating an existing model:
$model = Model::findBy('cn', 'John Doe');

$model->cn = 'Jane Doe';

$model->save();
```

You may also pass an array of attributes to persist to your LDAP directory:

```php
$model->save(['cn' => 'Steve Bauman']);
```

#### `setAppends`

Set the accessors to append to model arrays:

```php
$model->setAppends(['foo-bar']);

// Displays:
// [
//     'foo-bar',
// ]
var_dump($model->getAppends());
```

#### `setAttribute`

Set the value of an existing or non-existing attribute:

```php
$model->setAttribute('attribute', 'value');

$model->setAttribute('attribute', ['foo', 'bar']);
```

#### `setConnection`

Set the name of the connection for the model to use:

```php
$model = new Model();

// ...

$model->setConnection('domain-b');

// Model will be saved to 'domain-b'
$model->save();
```

#### `setDateFormat`

Set the date format to use when serializing LDAP dates:

```php
$model = Model::findByAnr('sbauman');

$model->setDateFormat('Y-m-d H:i');

$attributes = json_encode($model);
```

#### `setDn`

Set the Distinguished Name of the model to be created with:

```php
$model = new Model();

$model->setDn('cn=John Doe,dc=local,dc=com');

$model->save();
```

#### `setFirstAttribute`

Set the first value of an existing or non-existing attribute:

```php
$model = new Model();

$model->setFirstAttribute('cn', 'John Doe');

// Returns ['cn' => ['John Doe']]
$model->getAttributes();

$model->proxyaddresses = ['foo', 'bar'];

// Overwrites 'foo' with 'baz':
$model->setFirstAttribute('proxyaddresses', ['baz']);
```

#### `setHidden`

Clear defined hidden attributes and set the attributes
that should be hidden during serialization:

```php
$model->setHidden(['cn', 'sn']);

// Attributes 'cn' and 'sn' will be removed:
$attributes = json_encode($model);
```

#### `setModifications`

Set the models batch modifications to be processed upon save:

```php
$mods = [
    [
        'attrib'  => 'telephoneNumber',
        'modtype' => LDAP_MODIFY_BATCH_ADD,
        'values'  => ['+1 555 555 1717'],
    ]
];

$model->setModifications($mods);

$model->save();
```

#### `setVisible`

Clear defined visible attributes and set the attributes
that should be visible during serialization:

```php
$model->setVisible(['cn', 'sn']);

// Only attributes 'cn' and 'sn' will be included:
$attributes = json_encode($model);
```

#### `siblings`

Create a new query to retrieve a models siblings:

> **Important**: The existing model instance will be included in the query results.

```php
$siblings = $model->siblings()->get();
```

#### `update`

Persist the changes of a model to the LDAP directory.

> **Important**: The [save](#save) method should be used instead
> of `update` to persist new or existing models. If the model
> does not exist in the directory, an exception will be thrown.

```php
$model->cn = 'John Doe';

$model->update();
```

You may also provide an array of attributes to persist to your LDAP directory:

```php
$model->update(['cn' => 'John Doe']);
```

#### `updateAttribute`

This method was renamed to [replaceAttribute](#replaceattribute) in v3.0.
