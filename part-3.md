```php
if ($_REQUEST['email']) {
    $masterEmail = $_REQUEST['email'];
}

$masterEmail = isset($masterEmail) && $masterEmail
    ? $masterEmail
    : array_key_exists('masterEmail', $_REQUEST) && $_REQUEST["masterEmail"]
    ? $_REQUEST['masterEmail'] : 'unknown';

echo 'The master email is ' . $masterEmail . '\n';

$conn = mysqli_connect('localhost', 'root', 'sldjfpoweifns', 'my_database');
$res = mysqli_query($conn, "SELECT * FROM users WHERE email='" . $masterEmail . "'");
$row = mysqli_fetch_row($res);

echo $row['username'] . "\n";
```

We have a lot of issues here

1. If we don't have email parameter in HTTP request we will have an error since we use incorrect check.
Correct check can be following

```php
if (!empty($_REQUEST['email'])) {
    $masterEmail = $_REQUEST['email'];
}
```

or

```php
if (array_key_exists('email', $_REQUEST) && $_REQUEST["email"]) {
    $masterEmail = $_REQUEST['email'];
}
```

2. We have a SQL injection vulnerability here since we don't escape $email value in SQL query. We should use
prepared statements to be protected. Sequence is `prepare(), bind_param(), execute()`
   
3. With mysqli functions we are coupled with concrete database - MySQL. If we decide to switch to PostgreSQL
or some other database we will have to rewrite our queries. To solve this issue in PHP PDO is used.
   
4. We fix mix several concerns - working with http request, working with database, rendering (echo statements)
   and even a bit of business logic (masterEmail and email substitution). These parts should be decoupled to separate classes and files.
   
5. We provide sensitive information - database credentials in database connection. We should get database connection
settings from configuration file and configuration file should take sensitive information from .env file that 
   should not be stored under version control system. In this case information is better protected, and it's easy to 
   set up connection in different environments without affecting source code.
   
