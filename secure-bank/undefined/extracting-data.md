# Extracting Data

## Lab 8: SQL Injection - Extracting data with SQL injection

<figure><img src="../../.gitbook/assets/image (442).png" alt=""><figcaption></figcaption></figure>

Go to User Lookup page [http://127.0.0.1/index.php?page=user-info.php](http://127.0.0.1/index.php?page=user-info.php)

Injection this payload into username field: `admin' OR 1=1 —` we can dump all DB users

<figure><img src="../../.gitbook/assets/image (443).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (444).png" alt=""><figcaption></figcaption></figure>
