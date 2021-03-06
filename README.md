**What does?**

Implements simple and complex routing operations.

**What doesn't?**

Doesn't implements routes as same as somewhere like general MVC architectures but could be integrated easily, at least hoping so.. :)

<hr size="1" />

Before beginning;

- Set your autoloader properly
- Use PHP >= 5.4
- Handle errors with `try/catch` blocks

### Indicators

All provided operators will be setted as a parameter. If named, then could be called by its name, otherwise must be called by integer index.

Let's what operators produce and if you need more, simply use pure RegExp statements...

```text
# shortcuts
{%d}   = Digit -> (\d+)
{%w}   = Words -> (\w+)
{%x}   = Hexes -> ([a-f0-9]+)
{%ac}  = a-c   -> ([a-c]+)
{%az}  = a-z   -> ([a-z]+)
{%az*} = a-z   -> ([a-z]*)
{%az-} = a-z   -> ([a-z])

# named (1)
:uid = All   -> (?<uid>[^/]+)
# named (2)
{uid} = uid  -> (uid)
{followers|followees} = (followers|followees)
# named (2) with params
{uid} = uid  -> (?<uid>uid) # if params set as [uid]
{followers|followees} = (?<tab>(followers|followees)) # if params set as [tab]
```

```php
$route = new \Router\Route();
# This method is useful if you need to remove base URI
# E.g: http://dev.local/router/user
# $route->removeUriBase('/router');
```

Note: After all `add()` calls you need to call `run()` method.

### Simple
```php
# matches: /user
$route->add('/user', [
    '_name_' => 'user',
    '_file_' => '/routes/user.php',
]);

...

# Finally call matcher method after all route rules added
$route->run();

# If don't want to run router for "/" uri
if (!$route->isRoot()) {
    $route->run();
}

# Get route file
$file = $route->getFile();
# Check file (or you can define your file checker / shomething else)
if (is_file($file)) {
    include($file);
}
```

### With Params
```php
# You can use (retrieve) params anytime if needed
$uid = $route->getParam('uid');

# matches: /user/(\d+)
$route->add('/user/{%d}', [
    '_name_' => 'user',
    '_file_' => '/routes/user.php',
    'params' => ['uid']
]);

# Also you can filter them at first
$route->add('/user/{%d}', [
    '_name_' => 'user',
    '_file_' => '/routes/user.php',
    'params' => ['uid'],
    'params_filter' => function ($params) {
        foreach ($params as $key => $value) {
            $params[$key] = (int) $value;
            # or
            if ($key == 'uid') {
                $params[$key] = (int) $value;
            }
        }
    }
]);

# matches: /user/(\d+)/(followers|followees)
$route->add('/user/{%d}/{followers|followees}', [
    '_name_' => 'user',
    '_file_' => '/routes/user.php',
    'params' => ['uid', 'tab']
]);

# Or use param(s) as a file rename tool
$route->add('/user/{%d}/{followers|followees}', [
    '_name_' => 'user',
    # At this line, params.tab goes to rename user file
    '_file_' => '/routes/user-$tab.php',
    'params' => ['uid', 'tab']
]);
# Here is the same actions
$route->add('/user/{login|logout|register}', [
    '_name_' => 'user',
    '_file_' => '/routes/user-$page.php',
    'params' => ['page']
]);
$route->add('/{403|404}', [
    '_name_' => 'error',
    '_file_' => '/routes/errors/$code.php',
    'params' => ['code']
]);
```

### Named Params
```php
# matches: /user/(?<uid>[^/]+)
$route->add('/user/:uid', [
    '_name_' => 'user',
    '_file_' => '/routes/user.php',
]);

# matches: /user/(?<uid>[^/]+)/(?<tab>[^/]+)
$route->add('/user/:uid/:tab', [
    '_name_' => 'user',
    '_file_' => '/routes/user-$tab.php', # <- rename
]);

# matches: /user/(?<uid>[^/]+)/message/(?<mid>[^/]+)
$route->add('/user/:uid/message/:mid', [
    '_name_' => 'user',
    '_file_' => '/routes/user-message.php',
]);

# matches: /user/(?<uname>[^/]+)/message/(\d+)
$route->add('/user/:uname/message/{%d}', [
    '_name_' => 'user',
    '_file_' => '/routes/user-message.php',
]);
# Or name the digit part too
$route->add('/user/:uname/message/{%d}', [
    '_name_' => 'user',
    '_file_' => '/routes/user-message.php',
    'params' => ['', 'mid']
]);
```

### Shortcut Patterns
```php
$route->addShortcutPattern('digits', '(\d+)');
$route->addShortcutPattern('username', '(?<username>[a-z]{1}[a-z0-9-]{2,10})');

# matches: /user/(\d+)/message/(\d+)
$route->add('/user/$digits/message/$digits', [
    '_name_' => 'user',
    '_file_' => '/routes/user-message.php',
    'params' => ['uid', 'mid']
]);

# matches: /user/(?<username>[a-z]{1}[a-z0-9-]{2,10})
$route->add('/user/$username', [
    '_name_' => 'user',
    '_file_' => '/routes/user.php'
]);
# matches: /user/(?<username>[a-z]{1}[a-z0-9-]{2,10})/(?<tab>followers|followees)
$route->add('/user/$username/{followers|followees}', [
    '_name_' => 'user',
    '_file_' => '/routes/user-$tab.php',
    'params' => ['tab']
]);
```

### Manual RegExp's (pure for geeks)
```php
$route->add('/user/(\d+)', [
    '_name_' => 'user',
    '_file_' => '/routes/user.php',
]);

# With shortcut
# matches: /user/(?<username>[a-z]{1}[a-z0-9-]{2,10})/(?<tab>followers|followees)
$route->add('/user/$username/(?<tab>followers|followees)', [
    '_name_' => 'user',
    '_file_' => '/routes/user.php',
]);

# Idle, without named params set in source
$route->add('/user/(\d+)/(followers|followees)', [
    '_name_' => 'user',
    '_file_' => '/routes/user.php',
    'params' => ['uid', 'tab']
]);
# Idle, without params set
$route->add('/user/(?<uid>\d+)/(?<tab>followers|followees)', [
    '_name_' => 'user',
    '_file_' => '/routes/user.php',
]);
```

### Extras
```php
# Adding extras
$route->add('/foo/{bar|baz}', [
    '_name_' => 'foo',
    '_file_' => '/files/$file',
    'params' => ['file'],
    # here say router to not use header,footer files
    'extras' => ['head' => false, 'foot' => false]
]);

# Diagram
# Match route?      -> yes
# Found route file? -> yes

# Check to including header file or not
if (false !== $route->getExtra('head')) {
    include('/files/header.php');
}

# Include route file
include($route->getFile());

# Check to including footer file or not
if (false !== $route->getExtra('foot')) {
    include('/files/footer.php');
}
```
