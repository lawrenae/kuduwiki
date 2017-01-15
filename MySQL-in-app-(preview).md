The feature enables running [MySql](http://www.mysql.com/) within the site itself.  One does not need to provision database explicitly and the data is always backed up along with the site contents.   The key benefit is ease of use/setup and network performance (MySql running locally within the site).   The limitation is it does not support multiple instances, auto scale nor slots.

## Scenarios

Any sites using MySql as database namely [WordPress](https://wordpress.org/), [Joomla](https://www.joomla.org/), etc.

## Enable/disable feature 

To enable the feature, see [this](https://blogs.msdn.microsoft.com/appserviceteam/2016/08/18/announcing-mysql-in-app-preview-for-web-apps/).  To verify if MySql is actually accessible by the site, create the following file `d:\home\site\wwwroot\phpmysql.php` with below content and browse to `http://<sitename>.azurewebsites.net/phpmysql.php`.

```php
<?php
$servername = "";
$username = "";
$password = "";
$dbname = "";

// Parsing connnection string
foreach ($_SERVER as $key => $value) {
    if (strpos($key, "MYSQLCONNSTR_") !== 0) {
        continue;
    }
    
    $servername = preg_replace("/^.*Data Source=(.+?);.*$/", "\\1", $value);
    $dbname = preg_replace("/^.*Database=(.+?);.*$/", "\\1", $value);
    $username = preg_replace("/^.*User Id=(.+?);.*$/", "\\1", $value);
    $password = preg_replace("/^.*Password=(.+?)$/", "\\1", $value);
}

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);
// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
} else {
    echo "connection successful<br/>";
}
$conn->close();
?>
```

## Internals, Diagnostics and Loggings

#### Where can I find MySql Credentials (connection string)?

The connection string flows to your application as an env variable `MYSQLCONNSTR_localdb`.  Beware that we are *not* using the default MySql port (3306).  In fact, the port number may vary for each application life cycle depending on its availability at startup time.  The port info is also available as an env variable `WEBSITE_MYSQL_PORT` to your site.
  
#### How to use phpMyAdmin with MySql in-app?

[phpMyAdmin](https://www.phpmyadmin.net/) is enabled by default with the feature.  You can access it thru `https://<sitename>.scm.azurewebsites.net/phpMyAdmin/`.  Unlike phpMyAdmin from SiteExtenions gallery, this phpMyAdmin is aware of MySql credentials and will connect automatically.  The *caveat* is since MySql is only started with the main site if one were to access phpMyAdmin (which is part of SCM site) *without* the main site (and MySql) running, it may confusingly prompt for password.   We are working toward improving the phpMyAdmin experience to better inform users in this scenario.

<strong>Important:</strong> If you previously have phpMyAdmin installed via SiteExtension gallery, you will have to uninstall it.  Since this phpMyAdmin from SiteExtension gallery will take precedent and it is *not* MySql In-App aware, it will not work with MySql In-App. 

#### What about MySql command line tools from SCM Kudu Console?

Among other things, `MySql.exe` or `MySqlAdmin.exe` tools are available at `d:\program files (x86)\MySql\5.7.9.0\bin`.  Like phpMyAdmin above, make sure the main site is running before using the tool in SCM Kudu Console.  The credential is available at `D:\home\data\mysql\MYSQLCONNSTR_localdb.txt`.   If you are running WebJobs and need to get connection string programmatically, [this sample](https://github.com/suwatch/phpmyadmin/blob/96d3e349de0e17b3fe2f01f3d9e1e8b4497b86ed/config.inc.php#L49-L72) shows how to do so.   Do make sure you set AlwaysOn for your site to ensure MySql is up and running. 

Due to limited spaces, we may not have all the complete MySql tool set.  Let us know (via [forum](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowsazurewebsitespreview)) if any other tools would be useful.

#### Where is MySql Data directory?

MySql data is stored at `d:\home\data\mysql` directory.

#### Where is MySql Server log?

MySql server log is stored at `d:\home\logfiles\mysql` directory.   This will be where you start investigating any MySql issues.  Beware that the logfile names are dated and, if the number exceeds the limit (50 log files), old ones will be removed to free up spaces.  

To turn on [general log](http://dev.mysql.com/doc/refman/5.7/en/query-log.html), simply add appSettings `WEBSITE_MYSQL_GENERAL_LOG = 1`.  The log file is also stored at `d:\home\logfiles\mysql` directory (look for log file with `general` suffix).

To turn on [slow query log](http://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html), simply add appSettings `WEBSITE_MYSQL_SLOW_QUERY_LOG = 1`.  The log file is also stored at `d:\home\logfiles\mysql` directory (look for log file with `slowquery` suffix).

#### How do I customize MySql server (MySqld.exe arguments)?

You can specify extra arguments for MySqld.exe via `WEBSITE_MYSQL_ARGUMENTS` appSettings.

#### Is my database backed up and restored along with site backup/restore feature?

Yes.  During site backup, we simply run `mysqldump.exe` tool against In-App MySql.  The output script `LocalMySqlDatabase.sql` was saved along with the site content - you can see it in the backup zip file.  During restore, we simply run `mysql.exe` on the `LocalMySqlDatabase.sql` script. 

#### Can I import and export database by myself?

Yes. You could simply use phpMyAdmin (see how to access above).   Programatically, you could POST to `https://<sitename>.scm.azurewebsites.net/mysqlutils/dump` (with empty body) and `https://<sitename>.scm.azurewebsites.net/mysqlutils/execute` to export and import sql scripts respectively.  Note that, this is SCM of your site, you will need to provide publishing credentials.

#### Can I customize the database, username and password to be used?

Yes.  The connection string is stored at `D:\home\data\mysql\MYSQLCONNSTR_localdb.txt`.   The application (such as wordpress) reads from this file for what database, username and password to use.  This also applies to what to backup and restore provided by Azure WebApps.   If you want to customize the database, username and password, after you have created a new database, add new username or update password, simply modify `D:\home\data\mysql\MYSQLCONNSTR_localdb.ini`
, remove `D:\home\data\mysql\MYSQLCONNSTR_localdb.txt` and restart the WebApps.

#### Issue: Cannot connect to mysql invalid settings or wrong username/password?

Try delete `D:\home\data\mysql\MYSQLCONNSTR_localdb.txt` and restart the WebApps.

#### How do I report issues?

[Forum](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowsazurewebsitespreview) is a good starting point.  Do title it with `MySql in-app` keyword for faster references.
