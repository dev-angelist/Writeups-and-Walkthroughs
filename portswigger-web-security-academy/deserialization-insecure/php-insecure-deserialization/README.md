---
description: >-
  https://portswigger.net/web-security/deserialization/exploiting#php-serialization-format
icon: '3'
---

# PHP Insecure Deserialization

## **PHP Insecure Deserialization**

PHP serialization allows objects, arrays, and values to be converted into a storable string format using `serialize()`. However, when `unserialize()` is used on untrusted data, it can lead to arbitrary code execution, data manipulation, or unauthorized object injection.

#### **How PHP Serialization Works**

```php
phpCopiaModificaclass User {
    public $username;
    public function __construct($name) {
        $this->username = $name;
    }
}

$user = new User("admin");
$serialized = serialize($user);
echo $serialized;
```

**Output:**

```
cssCopiaModificaO:4:"User":1:{s:8:"username";s:5:"admin";}
```

This serialized string can be stored in a database, session, or sent over a network.

### **Magic Methods & Exploitation**

In PHP, special **magic methods** can be abused during deserialization:

* **`__wakeup()`** → Executes code when an object is unserialized.
* **`__sleep()`** → Executes code before serialization.
* **`__destruct()`** → Executes when an object is destroyed.
* **`__toString()`** → Can be used to trigger code execution via string conversion.

If a PHP application unserializes untrusted input, an attacker can inject a malicious object that triggers one of these methods.

#### **Example of PHP Object Injection Attack**

A vulnerable PHP application:

```php
phpCopiaModificaif(isset($_GET['data'])) {
    $obj = unserialize($_GET['data']);
}
```

An attacker can craft a malicious payload:

```php
phpCopiaModificaclass Malicious {
    public function __destruct() {
        system("whoami");
    }
}

$payload = serialize(new Malicious());
echo urlencode($payload); 
```

Example payload:

```
cssCopiaModificaO:9:"Malicious":0:{}
```

Sending this payload via `?data=O:9:"Malicious":0:{}` executes `whoami` on the server when the object is destroyed.

### **Mitigation Strategies**

* **Never use `unserialize()` on untrusted input.**
*   **Use JSON instead of serialization.**

    ```php
    phpCopiaModifica$json = json_encode($object);
    $object = json_decode($json);
    ```
*   **Implement allowlisting** to only accept expected classes:

    ```php
    phpCopiaModificaunserialize($data, ["allowed_classes" => ["SafeClass"]]);
    ```
* **Use Web Application Firewalls (WAFs)** to detect and block serialized attack payloads.

***

## Labs 🔬

* aaa
* bbb
