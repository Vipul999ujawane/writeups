## Circus
### Category : Web

TLDR;
git repository leak, and PHP Type Juggling (magic hash)


Used Git Dumper to leak the repository.

In the .gitignore, we see
```
backup.sh
backups
```
So, went to https://circus.tjctf.org/backup.sh

Found :

```
#!/bin/bash

mysqldump -u root -h mysql -p circus users > ./backups/users.sql
```

Figured, we can access the DB file from here. So, went to https://circus.tjctf.org/backups/users.sql

And we can!.

Got the users db with 1000 users and password hashes. 


Found index.php, 
```
<?php 
	require __DIR__ . "/../include/flag.php";

	session_start();
	$error = "";
	mysqli_report(MYSQLI_REPORT_ERROR | MYSQLI_REPORT_STRICT);
	if (isset($_POST["username"]) && isset($_POST["password"]))
	{
		try
		{
			$mysqli = new mysqli(NULL, NULL, NULL, "circus");
			$stmt = $mysqli->prepare("SELECT * FROM users WHERE username = ?");
			$stmt->bind_param("s", $_POST["username"]);
			$stmt->execute();
			$res = $stmt->get_result();
			if ($res->num_rows === 0)
			{
				$error = "Invalid credentials";
			}
			else
			{
				$row = $res->fetch_assoc();
				if (hash('sha256', $_POST["password"]) == $row["password"])
				{
					$_SESSION["id"] = $row["id"];
					$_SESSION["name"] = $row["fname"];
				}
			}
		}
		catch (Exception $e)
		{
			$error = "Error signing in";
		}
	}
?>
```
We find a weak compairson between the hash and the password! 

Time for using a [magic hash](https://github.com/spaze/hashes) (type juggling). 

In users.sql, we find a the username with the hash that matches the regex : `^0e\d+$`

We find the entry :
```
(773,'Andon1956','0e75759761935916943951971647195794671357976597614357959761597165','Rosie','Kelly')
```

We login with the username `Andon1956` and a magic hash keyword  

![](Circus_flag.jpeg)