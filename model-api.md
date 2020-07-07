---
title: Model API
description: A list of all available LdapRecord model methods.
extends: _layouts.documentation
section: content
---

# Available Model Methods (API)

<div class="api-method-list">
<p>
<a href="#addAttributeValue">addAttributeValue</a>
<a href="#addHidden">addHidden</a>
<a href="#addModification">addModification</a>
<a href="#addVisible">addVisible</a>
<a href="#ancestors">ancestors</a>
<a href="#asDateTime">asDateTime</a>
<a href="#attributesToArray">attributesToArray</a>
<a href="#convert">convert</a>
<a href="#countAttributes">countAttributes</a>
<a href="#createAttribute">createAttribute</a>
<a href="#delete">delete</a>
<a href="#deleteAttribute">deleteAttribute</a>
<a href="#descendants">descendants</a>
<a href="#escape">escape</a>
<a href="#fill">fill</a>
<a href="#fresh">fresh</a>
<a href="#fromDateTime">fromDateTime</a>
<a href="#getAnrAttributes">getAnrAttributes</a>
<a href="#getAttribute">getAttribute</a>
<a href="#getAttributeValue">getAttributeValue</a>
<a href="#getAttributes">getAttributes</a>
<a href="#getConnection">getConnection</a>
<a href="#getConnectionName">getConnectionName</a>
<a href="#getConvertedGuid">getConvertedGuid</a>
<a href="#getCreatableDn">getCreatableDn</a>
<a href="#getCreatableRdn">getCreatableRdn</a>
<a href="#getDateFormat">getDateFormat</a>
<a href="#getDates">getDates</a>
<a href="#getDirty">getDirty</a>
<a href="#getDn">getDn</a>
<a href="#getFirstAttribute">getFirstAttribute</a>
<a href="#getGlobalScopes">getGlobalScopes</a>
<a href="#getGuidKey">getGuidKey</a>
<a href="#getHidden">getHidden</a>
<a href="#getModifications">getModifications</a>
<a href="#getName">getName</a>
<a href="#getObjectGuid">getObjectGuid</a>
<a href="#getOriginal">getOriginal</a>
<a href="#getParentDn">getParentDn</a>
<a href="#getRdn">getRdn</a>
<a href="#getVisible">getVisible</a>
<a href="#hasAttribute">hasAttribute</a>
<a href="#hasGetMutator">hasGetMutator</a>
<a href="#hasSetMutator">hasSetMutator</a>
<a href="#inside">inside</a>
<a href="#is">is</a>
<a href="#isAncestorOf">isAncestorOf</a>
<a href="#isChildOf">isChildOf</a>
<a href="#isDateAttribute">isDateAttribute</a>
<a href="#isDescendantOf">isDescendantOf</a>
<a href="#isDirty">isDirty</a>
<a href="#isParentOf">isParentOf</a>
<a href="#makeHidden">makeHidden</a>
<a href="#makeVisible">makeVisible</a>
<a href="#move">move</a>
<a href="#newBatchModification">newBatchModification</a>
<a href="#newCollection">newCollection</a>
<a href="#newInstance">newInstance</a>
<a href="#newQuery">newQuery</a>
<a href="#newQueryWithoutScopes">newQueryWithoutScopes</a>
<a href="#rename">rename</a>
<a href="#save">save</a>
<a href="#setAttribute">setAttribute</a>
<a href="#setConnection">setConnection</a>
<a href="#setDateFormat">setDateFormat</a>
<a href="#setDn">setDn</a>
<a href="#setFirstAttribute">setFirstAttribute</a>
<a href="#setHidden">setHidden</a>
<a href="#setModifications">setModifications</a>
<a href="#setVisible">setVisible</a>
<a href="#siblings">siblings</a>
<a href="#synchronize">synchronize</a>
<a href="#update">update</a>
<a href="#updateAttribute">updateAttribute</a>
</p>
</div>

## Method Listing

#### `addAttributeValue` {#addAttributeValue}

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

#### `addHidden` {#addHidden}

Add an attribute to hide when encoding a model using `json_encode`:

```php
$model->addHidden('userpassword');

$model->addHidden(['userpassword', 'mail']);

// 'userpassword' and 'mail' will be omitted:
$attributes = json_encode($model);
```

#### `addModification` {#addModification}

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

#### `addVisible` {#addVisible}

Add an attribute to include that is being hidden when encoding a model using `json_encode`:

```php
$model->addVisible('userpassword');

$model->addVisible(['userpassword', 'mail']);
```

#### `ancestors` {#ancestors}

Begin querying the direct ancestors of the model:

```php
$ancestors = $model->ancestors()->get();
```

#### `asDateTime` {#asDateTime}

Convert an LDAP timestamp to a `Carbon\Carbon` instance:

```php
$carbon = $model->asDateTime('ldap', '20200508184557Z');

$carbon = $model->asDateTime('windows', '20200508184533.0Z');

$carbon = $model->asDateTime('windows-int', 132334371140000000);
```

#### `attributesToArray` {#attributesToArray}

Convert all the models attributes to their JSON encodable value:

```php
$attributes = $model->attributesToArray();
```

> LDAP date attributes specified via the `$dates` model property will be converted in the returned array.

#### `convert` {#convert}

Convert a model into another by copying its attributes, connection and distinguished name:

> This will also set `$model->exists` property to `true` if the **model being converted** exists.

```php
$into = new \LdapRecord\Models\ActiveDirectory\User();

// Returns instance of \LdapRecord\Models\ActiveDirectory\User
$user = $model->convert($into);
```

#### `countAttributes` {#countAttributes}

Get the total number of attributes on a model:

> This does not count attribute values.

```php
$model->fill([
    'cn' => 'Steve Bauman',
    'sn' => 'Bauman',
]);

// Returns '2'
$model->countAttributes();
```

#### `createAttribute` {#createAttribute}

Immediately inserts a new attribute value on the model.

Performs an `ldap_mod_add` under the hood.

> This does not create attributes that do not exist on your LDAP schema for the object.

```php
$model->createAttribute('telephonenumber', '+1 555 555 1717');
```

#### `delete` {#delete}

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

#### `deleteAttribute` {#deleteAttribute}

Immediately delete an attribute on a model.

Performs an `ldap_mod_del` under the hood.

```php
$model->deleteAttribute('telephonenumber');

$model->telephonenumber; // Returns null
```

#### `descendants` {#descendants}

Begin querying the direct descendants of the model:

```php
$descendants = $model->descendants()->get();
```

#### `escape` {#escape}

Prepare a value to be escaped:

```php
// Returns \LdapRecord\Models\Attributes\EscapedValue;
$escapedValue = $model->escape($value, $ignore = '', $flags = 0);

// Cast to string for immediate return of escaped value:
$escapedValue = (string) $model->escape($value, $ignore = '', $flags = 0);
```

#### `fill` {#fill}

Fill a model with attributes and values:

```php
$model->fill([
    'cn' => 'Steve Bauman',
    'proxyAddresses' => ['foo', 'bar'],
]);

$model->save();
```

#### `fresh` {#fresh}

Get a fresh **new** instance of the existing model.

The model will be re-retrieved from the LDAP directory. The existing model will not be affected:

```php
$fresh = $model->fresh();
```

#### `fromDateTime` {#fromDateTime}

Convert a `DateTime`, `Carbon` or `string` to the specified LDAP timestamp type:

```php
$date = new \DateTime('now');

$ldapTimestamp = $model->fromDateTime('ldap', $date);

$windowsTimestamp = $model->fromDateTime('windows', $date);

$windowsIntTimestamp = $model->fromDateTime('windows-int', $date);
```

#### `getAnrAttributes` {#getAnrAttributes}

Get an array of ANR attributes defined on the model:

```php
$attributes = $model->getAnrAttributes();

// Displays: [
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

#### `getAttribute` {#getAttribute}

Get the value of the specified attribute.

This will return an `array` if the attribute exists and `null` if non-existent:

> If a mutator exists for the attribute (such as a date, or custom attribute
> [mutator method](//docs/model-mutators/#defining-a-mutator)), it's value will be returned instead.

```php
// Returns array:
$values = $model->getAttribute('cn');

// Returns first value:
$value = $model->getAttribute('cn')[0];

// Returns null:
$null = $model->getAttribute('non-existent');
```

#### `getAttributeValue` {#getAttributeValue}

Performs the same as above.

#### `getAttributes` {#getAttributes}

Get all the models raw attributes:

> Mutator attribute values will not be included in this array.

```php
$attributes = $model->getAttributes();

foreach ($attributes as $attribute => $values) {
    //
}
```

#### `getConnection` {#getConnection}

Resolve the underlying `LdapRecord\Connection`:

```php
$connection = $model->getConnection();

$config = $connection->getConfiguration();
```

#### `getConnectionName` {#getConnectionName}

Get the connection name from the model:

> If no connection is set on the model, `null` will be returned.

```php
class Entry extends Model
{
    protected $connection = 'domain-b';
}

// Returns 'domain-b'
$connectionName = (new Entry)->getConnectionName();
```

#### `getConvertedGuid` {#getConvertedGuid}

Get the models string GUID:

```php
// Example: bf9679e7-0de6-11d0-a285-00aa003049e2
$guid = $model->getConvertedGuid();
```

#### `getCreatableDn` {#getCreatableDn}

Get the models creatable Distinguished Name:

```php
$model = new Entry(['cn' => 'John Doe']);

// Returns: 'cn=John Doe,dc=local,dc=com'
$creatableDn = $model->getCreatableDn();

$model->cn = 'Jane Doe';

// Returns: 'cn=Jane Doe,dc=local,dc=com'
$creatableDn = $model->getCreatableDn();
```

#### `getCreatableRdn` {#getCreatableRdn}

Get the models creatable relative Distinguished Name:

```php
$model = new Entry(['cn' => 'John Doe']);

// Returns: 'cn=John Doe'
$creatableDn = $model->getCreatableRdn();

$model->cn = 'Jane Doe';

// Returns: 'cn=Jane Doe'
$creatableDn = $model->getCreatableRdn();
```

#### `getDateFormat` {#getDateFormat}

Get the format that dates are serialized to:

```php
// Returns: 'Y-m-d\TH:i:sO'
$model->getDateFormat();
```

#### `getDates` {#getDates}

Get an array of attributes that should be mutated to dates:

```php
$dates = $model->getDates();

// Displays: [
//    'createtimestamp' => 'ldap',
//    'modifytimestamp' => 'ldap',
// ]
var_dump($dates);
```

#### `getDirty` {#getDirty}

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

#### `getDn` {#getDn}

Get the models Distinguished Name:

```php
// Displays: 'cn=John Doe,dc=local,dc=com'
echo $model->getDn();
```

#### `getFirstAttribute` {#getFirstAttribute}

Get the first value of an attribute:

```php
$model->proxyaddresses = ['first', 'second'];

// Returns: 'first'
$value = $model->getFirstAttribute('proxyaddresses');
```

#### `getGlobalScopes` {#getGlobalScopes}

Get the global scopes set on the model:

```php
Model::addGlobalScope('my-scope', function () {
    // ...
});

// Returns: ['my-scope' => \Closure]
$scopes = $model->getGlobalScopes();
```

#### `getGuidKey` {#getGuidKey}

Get the attribute key that contains the objects GUID:

```php
// Returns: 'objectguid'
$model->getGuidKey();
```

#### `getHidden` {#getHidden}

Get the hidden attributes for the model:

```php
$model->addHidden('cn');

// Returns: ['cn']
$model->getHidden();
```

#### `getModifications` {#getModifications}

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

#### `getName` {#getName}

Get the name of a model:

```php
$model->setDn('cn=John Doe,dc=local,dc=com');

// Returns: 'John Doe'
$name = $model->getName();
```

#### `getObjectGuid` {#getObjectGuid}

Get the raw binary object GUID:

> The raw binary object GUID will be returned when connecting to Active Directory.
> <br/><br/>
> The raw string GUID will be returned with other LDAP directories.

```php
$rawBinary = $model->getObjectGuid();
```

#### `getOriginal` {#getOriginal}

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

#### `getParentDn` {#getParentDn}

Get the direct parent Distinguished Name of a model:

```php
$model = Model::find('cn=John Doe,dc=local,dc=com');

// Returns: 'dc=local,dc=com'
$model->getParentDn();

// Returns: 'ou=Users,dc=local,dc=com'
$model->getParentDn('cn=Steve Bauman,ou=Users,dc=local,dc=com');
```

#### `getRdn` {#getRdn}

Get the models Relative Distinguished Name:

```php
$model = Model::find('cn=John Doe,dc=local,dc=com');

// Returns: 'cn=John Doe'
$model->getRdn();

// Returns: 'cn=Steve Bauman'
$model->getRdn('cn=Steve Bauman,ou=Users,dc=local,dc=com');
```

#### `getVisible` {#getVisible}

Get the attributes that should be visible when encoding a model using `json_encode`:

```php
$model->addVisible('cn', 'sn');

// Returns: ['cn', 'sn']
$visibleAttributes = $model->getVisible();
```

#### `hasAttribute` {#hasAttribute}

Determine if the model has an attribute with a value:

```php
$model = Model::findBy('cn', 'Steve Bauman');

// Returns: true
$modle->hasAttribute('cn');

// Returns: false
$model->hasAttribute('non-existent');
```

#### `hasGetMutator` {#hasGetMutator}

Determine if the model has a 'get' mutator for the given attribute:

```php
class Entry extends Model
{
    public function getCnAttribute($values)
    {
        // ...
    }
}

$model = new Entry();

// Returns: true
$model->hasGetMutator('cn');
```

#### `hasSetMutator` {#hasSetMutator}

Determine if the model has a 'set' mutator for the given attribute:

```php
class Entry extends Model
{
    public function setCnAttribute($values)
    {
        // ...
    }
}

$model = new Entry();

// Returns: true
$model->hasSetAttribute('cn');
```

#### `inside` {#inside}

Set the container that the model should be **created** inside:

> Calling `inside()` on an existing model will not perform any move / rename operation.
> <br/><br/>
> Use [move](#move) or [rename](#rename) instead.

```php
$model = new Model();

// ...

$model->inside('ou=Container,dc=local,dc=com');

$model->save();
```

#### `is` {#is}

Determine if a model is the same by comparing their Distinguished Names and connections:

```php
// Returns: bool
$model->is($another);
```

#### `isAncestorOf` {#isAncestorOf}

Determine if a model is an ancestor of another:

```php
$user = User::find('cn=John Doe,ou=Accounting,ou=Accounts,dc=acme,dc=org');
$ou = OrganizationalUnit::find('ou=Accounts,dc=acme,dc=org');

// Returns: true
$ou->isAncestorOf($user);
```

#### `isChildOf` {#isChildOf}

Determine if a model is an **immediate** child of another:

```php
$ou = OrganizationalUnit::find('ou=Accounts,dc=acme,dc=org');
$user = User::find('cn=John Doe,ou=Accounts,dc=acme,dc=org');

// Returns: true
$user->isChildOf($ou);
```

#### `isDateAttribute` {#isDateAttribute}

Determine if given attribute is a date attribute on the model:

```php
class Entry extends Model
{
    protected $dates = ['whenchanged' => 'windows'];
}

$model = new Entry();

// Returns: true
$model->isDateAttribute('whenchanged');
```

#### `isDescendantOf` {#isDescendantOf}

Determine if a model is a descendent of another:

```php
$ou = OrganizationalUnit::find('ou=Accounts,dc=acme,dc=org');
$user = User::find('cn=John Doe,ou=Accounting,ou=Accounts,dc=acme,dc=org');

// Returns: true
$user->isDescendantOf($ou);
```

#### `isDirty` {#isDirty}

Determine if the given attribute has been changed:

```php
$model = Model::findBy('cn', 'Steve Bauman');

// Returns: false
$model->isDirty('cn');

$model->cn = 'Changed';

// Returns: true
$model->isDirty('cn');
```

#### `isParentOf` {#isParentOf}

Determine if a model is an **immediate** parent of another:

```php
$ou = OrganizationalUnit::find('ou=Accounts,dc=acme,dc=org');
$user = User::find('cn=John Doe,ou=Accounts,dc=acme,dc=org');

// Returns: true
$ou->isParentOf($user);
```

#### `makeHidden` {#makeHidden}

Make the given, typically visible, attributes hidden on the model:

```php
class Entry extends Model
{
    protected $visible = ['cn'];
}

// This will override the above:
$model->makeHidden('cn');
```

#### `makeVisible` {#makeVisible}

Make the given, typically hidden, attributes visible on the model:

```php
class Entry extends Model
{
    protected $hidden = ['cn'];
}

// This will override the above:
$model->makeVisible('cn');
```

#### `move` {#move}

Move a model into the specified container:

```php
$user = User::find('cn=Steve Bauman,dc=local,dc=com');

$ou = OrganizationalUnit::find('ou=Office Users,dc=local,dc=com');

$user->move($ou);

// Returns: 'cn=Steve Bauman,ou=Office Users,dc=local,d=com'
$user->getDn();
```

#### `newBatchModification` {#newBatchModification}

Create a new `LdapRecord\Models\BatchModification` instance:

```php
// Returns instance of: BatchModification
$mod = $model->newBatchModification(
    'cn', LDAP_MODIFY_BATCH_ADD, ['Steve Bauman']
);
```

#### `newCollection` {#newCollection}

Create a new `Tightenco\Collect\Support\Collection`:

```php
$collection = $model->newCollection($items = []);
```

#### `newInstance` {#newInstance}

Create a **new** model instance:

```php
$model = Model::findByAnr('sbauman');

$new = $model->newInstance();
```

#### `newQuery` {#newQuery}

Create a **new** model query:

> Global scopes and object classes **will be applied** to this query.

```php
$results = $model->newQuery()->where('cn', 'contains', 'Steve')->get();
```

#### `newQueryWithoutScopes` {#newQueryWithoutScopes}

Create a new model query **without** global scopes:

> Global scopes and object classes **will not be applied** to this query.

```php
$results = $model->newQueryWithoutScopes()->get();
```

#### `rename` {#rename}

Rename a model and keep it's container location:

> When renaming is successful, the models distinguished name is automatically
> updated to reflect its new name in the directory, so you may run further
> operations on it during the same request.

```php
$user = Model::find('cn=John Doe,dc=local,dc=com');

$user->rename('cn=Jane Doe');

// Returns: 'cn=Jane Doe,dc=local,dc=com'
$user->getDn();
```

#### `save` {#save}

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

#### `setAttribute` {#setAttribute}

Set the value of an existing or non-existing attribute:

```php
$model->setAttribute('attribute', 'value');

$model->setAttribute('attribute', ['foo', 'bar']);
```

#### `setConnection` {#setConnection}

Set the name of the connection for the model to use:

```php
$model = new Model();

// ...

$model->setConnection('domain-b');

// Model will be saved to 'domain-b'
$model->save();
```

#### `setDateFormat` {#setDateFormat}

Set the date format to use when serializing LDAP dates:

```php
$model = Model::findByAnr('sbauman');

$model->setDateFormat('Y-m-d H:i');

$attributes = json_encode($model);
```

#### `setDn` {#setDn}

Set the Distinguished Name of the model to be created with:

```php
$model = new Model();

$model->setDn('cn=John Doe,dc=local,dc=com');

$model->save();
```

#### `setFirstAttribute` {#setFirstAttribute}

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

#### `setHidden` {#setHidden}

Clear defined hidden attributes and set the attributes
that should be hidden during serialization:

```php
$model->setHidden(['cn', 'sn']);

// Attributes 'cn' and 'sn' will be removed:
$attributes = json_encode($model);
```

#### `setModifications` {#setModifications}

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

#### `setVisible` {#setVisible}

Clear defined visible attributes and set the attributes
that should be visible during serialization:

```php
$model->setVisible(['cn', 'sn']);

// Only attributes 'cn' and 'sn' will be included:
$attributes = json_encode($model);
```

#### `siblings` {#siblings}

Create a new query to retrieve a models siblings:

> The existing model instance will be included in the query results.

```php
$siblings = $model->siblings()->get();
```

#### `synchronize` {#synchronize}

Refresh a models attributes by re-retrieving them from your LDAP directory.

This will affect the current model instance:

```php
$model->synchronize();
```

#### `update` {#update}

Persist the changes of a model to the LDAP directory.

> The [save](#save) method should be used instead of `update` to persist new or existing models.
> <br/><br/>
> If the model does not exist in the directory, an exception will be thrown.

```php
$model->cn = 'John Doe';

$model->update();
```

You may also provide an array of attributes to persist to your LDAP directory:

```php
$model->update(['cn' => 'John Doe']);
```

#### `updateAttribute` {#updateAttribute}

Immediately updates an attribute value on the model.

Performs an `ldap_mod_replace` under the hood.

```php
$model->updateAttribute('proxyaddresses', ['foo', 'bar']);
```
