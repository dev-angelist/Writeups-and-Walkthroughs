---
description: >-
  https://portswigger.net/web-security/deserialization/exploiting#java-serialization-format
icon: '2'
---

# Java Insecure Deserialization

## **Java Insecure Deserialization**

Insecure deserialization in Java occurs when an application deserializes untrusted data without proper validation, allowing an attacker to manipulate the serialized object and achieve remote code execution (RCE), privilege escalation, or other malicious activities.

#### **Key Concepts**

**Gadgets and Gadget Libraries**

* **Gadget**: A property or method inside an object that can be leveraged for exploitation when deserialized.
* **Gadget Library**: Some Java libraries contain pre-existing gadgets that attackers can use to construct an exploit. Examples include:
  * **Apache CommonsCollections** (versions 1-6)
  * **Spring Framework**
  * **JDK classes (e.g., `java.rmi.server.UnicastRemoteObject`)**
  * **Jackson, Fastjson, and XStream**

Even though these libraries are not inherently vulnerable, if an application deserializes untrusted input while these libraries are present in the classpath, an attacker can exploit them to create a **gadget chain**, leading to successful exploitation.

**How Java Deserialization Works**

Serialization in Java is the process of converting an object into a byte stream that can be stored or transmitted.

* `ObjectOutputStream.writeObject(obj)`: Serializes an object.
* `ObjectInputStream.readObject()`: Deserializes the byte stream into an object.

If the input passed to `readObject()` is attacker-controlled, it can be exploited to execute arbitrary code.

### **Exploitation and Attack Vectors**

**Exploiting Java Insecure Deserialization**

An attacker can exploit insecure deserialization by sending a crafted malicious serialized object to a vulnerable application. Common attack vectors include:

* **Cookies** → Sending serialized payloads via HTTP cookies.
* **Web Requests** → Injecting payloads through POST/GET parameters.
* **Files** → Uploading serialized payloads as files.
* **Inter-Process Communication (IPC)** → Sending malicious objects through network sockets, RMI, or JNDI.

**Ysoserial Tool**

[ysoserial](https://github.com/frohoff/ysoserial) is a popular tool for generating exploit payloads using known gadget libraries.\
Example command:

```bash
bashCopiaModificajava -jar ysoserial.jar CommonsCollections1 "whoami" > payload.ser
```

* This generates a payload that executes `whoami` upon deserialization.
* The payload is then sent to the vulnerable application, which executes the system command when deserialized.

#### **Exploit Example**

If an application deserializes objects from an untrusted source, an attacker can craft a malicious object like:

```java
javaCopiaModificaimport java.io.*;

public class MaliciousPayload implements Serializable {
    private void readObject(ObjectInputStream in) throws Exception {
        Runtime.getRuntime().exec("calc.exe");  // Arbitrary code execution
    }
}
```

When deserialized, this object spawns a calculator on Windows or executes a system command.

### **Mitigation Strategies**

* **Avoid deserialization of untrusted data.**
*   **Use `ObjectInputFilter` (Java 9+)** to allowlist safe classes:

    ```java
    javaCopiaModificaObjectInputFilter filter = info -> info.serialClass().getName().startsWith("trusted.package") 
        ? ObjectInputFilter.Status.ALLOWED 
        : ObjectInputFilter.Status.REJECTED;
    ```
* **Use safer data formats like JSON or protobuf** instead of Java serialization.
* **Keep dependencies updated** to avoid known gadget chains.
* **Restrict available classes** in the classpath to prevent gadget chains.

***

## Labs 🔬

* aaa
* bbb
