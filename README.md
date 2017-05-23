# Laravel Permit

Laravel Permit is an authorization and ACL package for laravel. Its fast and more customizable.
You can easily handle role based ACL or specific user wise permission. So, Lets start a journey with Laravel Permit.

## Installation

You can start it from composer. Go to your terminal and run this command from your project root directory.

```shell
composer require nahid/permit
```

Wait for a while, its download all dependencies.

## Configurations

After installation complete successfully you have to configure it. First copy these line paste it in `config/app.php` where `providers` array is exists.

```php
Nahid\Permit\PermitServiceProvider::class,
```

and add the line for facade support

```php
'Permit'    => Nahid\Permit\Facades\Permit::class,
```

hmm, Now you have to run this command to publish necessary files.

```shell
php artisan vendor:publish --provider=Nahid\Permit\PermitServiceProvider
```

and then go to `config/permit.php` and edit with your desire credentials.

```php
return [
    "users" => [
        'model' => \App\User::class,
        'table' => 'users'
    ]

];
```

Now run this command for migrations

```shell
php artisan migrate
```

You are all most done, just add this trait `Nahid\Permit\Users\Permitable` in you `User` model. Example

```php
namespace App;

use Illuminate\Database\Eloquent\Model;
use Nahid\Permit\Users\Permitable;

class User extends Model
{
    use Permitable;
}
```

Yeh, its done.

## How does it work?

Its a common question. But first you have to learn about our database architecture.
When you run migrate command then we create a table 'permissions' with field 'role_name' and 'permission', and
add two column 'role' and 'permissions' in `users` table. `role` column store users role and `permissions` column store user specific controls.
Here `role` column has a relation with `permissions.role_name` column with its controls. `permissions.permission` handle role based control.

We store permissions as JSON format with specific service and controls.

```json
{
    "user": {
        "create": true,
        "update": true
    },
    "post": {
        "create": false,
        "update": true,
        "delete": false
    }
}
```

Here `user` and `post` is a service/module name and `create`, `update` and `delete` or others is an event.

### Set User Role

##### Syntax

`bool Permit::setUserRole(int $user_id, string $role_name)`

##### Example

```php
Permit::setUserRole(1, 'admin');
```

### Set User Permission

##### Syntax

`bool Permit::setUserPermission(int $user_id, string $service, array $events)`

##### Example

```php
Permit::setUserPermission(1, 'post', ['create'=>true, 'update'=>true]);
```


### Set Role Permission

##### Syntax

`bool Permit::setRolePermission(string $role_name, string $service, array $events)`

##### Example

```php
Permit::setRolePermission('admin', 'post', ['create'=>true, 'update'=>true]);
```


## How to Authorize an event?

### Check user ability

```php
$user = User::find(1);

if (Permit::userCan($user, 'post:create')) {
    //do something
}
```
In `post:create` is an event with module/service. Here `post` is a module and `create` is an event.

So if the user is authorized with post create event then the user passed.

`Permit::userCan()` method return boolean. If you want to throw Unauthorized exception you may use

`Permit::userAllows()` with same parameters.

### Check user role ability

```php
$user = User::find(1);

if (Permit::roleCan($user, 'post:create')) {
    //do something
}
```

Here when given users role allowed this event then its passed. Here is a similar method for throw exception

`Permit::roleAllows()`

### User ability

User ability with role and specific permissions. Here we check both(user and role) permission but if user specific permission is priority first.

```php
$user = User::find(1);

if (Permit::can($user, 'post:create')) {
    //do something
}
```

and here is a alternate method for throw exception

`Permit::allows()`


### Helper function

