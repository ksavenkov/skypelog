skypelog
========

Read Skype messages even if they are removed by senders.

OK, small how-to to start with.

On MacOS and Linux, Skype stores all messages in SQLite database. The obvious idea is to create a table to store all messages inserted in that table. When someone removes his message, the table with messages will be updated with an empty message.
But our copy won't. Hooray!

If you're a Linux user, you can skip walkthrough and use the shell scripts:

* `install.sh <YOUR SKYPE LOGIN>` for setting logger up;

* `read.sh <YOUR SKYPE LOGIN> <SKYPE LOGIN OF MESSAGE AUTHOR>` for reading edited/removed messages of specified user;

* `uninstall.sh <YOUR SKYPE LOGIN>` for logger removal.

Walkthrough:

1. Login to the skype DB on your local machine:

	* MacOS

		```
		> sqlite3 /Users/$USERNAME/Library/Application\ Support/Skype/$SKYPE_LOGIN/main.db
		```

	* Linux

		```
		> sqlite3 ~/.Skype/$SKYPE_LOGIN/main.db
		```

2. Create a table to store our skype log (I decided to keep just a few fields):

	```
	sqlite> CREATE TABLE skypelog (author TEXT, from_dispname TEXT, timestamp INTEGER, body_xml TEXT);
	```

3. Create an update trigger:

	```
	sqlite> CREATE TRIGGER update_skypelog AFTER UPDATE ON Messages
	...> BEGIN
	...> INSERT INTO skypelog
	...> SELECT new.author, new.from_dispname, new.timestamp, new.body_xml
	...> WHERE (SELECT count(*) FROM skypelog 
	...> WHERE timestamp = new.timestamp AND body_xml = new.body_xml) = 0;
	...> END;
	```

4. That's all, folks! Whenever you see some message is removed and want to read it, type the following:

	* Short version displaying all messages:

		```
		sqlite> SELECT * FROM skypelog WHERE author = $MESSAGE_AUTHOR_SKYPE_LOGIN;
		```

	* Extended version displaying only edited/deleted messages:

		```
		sqlite> SELECT l.from_dispname, l.body_xml FROM skypelog l LEFT JOIN Messages m
		sqlite> ON l.timestamp = m.timestamp and l.body_xml = m.body_xml
		sqlite> WHERE l.author = $MESSAGE_AUTHOR_SKYPE_LOGIN AND m.timestamp is null;
		```