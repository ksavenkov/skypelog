skypelog
========

Read Skype messages even if they are removed by senders.

OK, small how-to to start with.

On MacOS, Skype stores all messages in SQLite database. The obvious idea is to create a table to store all messages
inserted in that table. When someone removes his message, the table with messages will be updated with an empty message.
But our copy won't. Hooray!

Walkthrough:

1. Login to the skype DB on your local machine:

```
> sqlite3 /Users/$USERNAME/Library/Application\ Support/Skype/$SKYPE_LOGIN/main.db
```

2. Create a table to store our skype log (I decided to keep just a few fields):

```
sqlite> CREATE TABLE skypelog (author TEXT, from_dispname TEXT, timestamp INTEGER, body_xml TEXT);
```

3. Create an update trigger:

```
sqlite> CREATE TRIGGER update_skypelog AFTER UPDATE ON Messages
...> BEGIN
...> INSERT INTO skypelog (author, from_dispname, timestamp, body_xml)
...> values (new.author, new.from_dispname, new.timestamp, new.body_xml);
...> END;
```

4. That's all, folks! Whenever you see some message is removed and want to read it, type the following:

```
sqlite> SELECT * FROM skypelog WHERE author = $MESSAGE_AUTHOR_SKYPE_LOGIN;
```

KNOWN BUGS: several identical entries per message are inserted in the skypelog table, probably due to status is updated 
after the message is read etc. Probably need to replace INSERT in the trigger with INSERT_OR_UPDATE.