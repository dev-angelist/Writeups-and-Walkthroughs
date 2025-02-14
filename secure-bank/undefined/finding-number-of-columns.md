# Finding Number of Columns

## Lab 9: SQL Injection - Finding Number of Columns

<figure><img src="../../.gitbook/assets/image (440).png" alt=""><figcaption></figcaption></figure>

Go to User Lookup page [http://127.0.0.1/index.php?page=user-info.php](http://127.0.0.1/index.php?page=user-info.php)

in this case we need to find the number of columns, the idea is to user the SQL operator 'ORDER BY' to simulate a sorting of column, if we'll have a SQL error for the column with number columns\_number + 1, it means that the columns\_number tested was correct

Payload -> `' ORDER BY columns_number+1 #`&#x20;

<figure><img src="../../.gitbook/assets/image (445).png" alt=""><figcaption></figcaption></figure>

No SQL error, than 5 columns isn't the correct answer, continuing 10 is the correct answer because `' ORDER BY 11 #` payload generates a SQL error:

<figure><img src="../../.gitbook/assets/image (446).png" alt=""><figcaption></figcaption></figure>
