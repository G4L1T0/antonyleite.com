---
title: Introduction to Insecure Deserialization in PHP
date: 2021-06-18 00:00:00 -0300
tags: [insecure-deserialization-php]
---

# Introduction
Since the beginning of the year in my trajectory in the Pentest as a Service (PTaaS) team at Conviso, I have been following the operations related to pentesters. So, I chose some topics to research and to delve deeper. Today I'm going to introduce the topic of Insecure Deserialization in PHP.

# Serialization Process
The serialization process aims to transform an object into a format that can be easily stored, transferred and restored when necessary, without changing its attributes and values. Serialization simplifies and facilitates some processes, for example:

- Data storage in files or databases;
- Sending data over a network or in API calls.

Below is a diagram that demonstrates how serialization and deserialization (the reverse process of serialization) happen.

![](https://i.imgur.com/9GygQVS.png)

# PHP serialization
The PHP (Hypertext Preprocessor) programming language has built-in serialization functions. When necessary to use the serialization process, use serialize() and unserialize() for deserialization. Below is an example of serializing the ConvisoPerson object.

```php
<?php
Class  ConvisoPerson
{
    public $username = 'Antony';
    public $team = 'PTaaS';
    public $age = 17;
    public $office = 'Intern';
    public $accountAdmin =  False;
    public  function  validateAdmin(){
    if ($this->accountAdmin){
        echo  ' [+] '  .  $this->username .  ' is administrator\n';
    } else {
        echo  ' [+] '  .  $this->username .  ' not is administrator\n';
        }
    }
}
$convisoPerson =  new  ConvisoPerson();
echo  serialize($convisoPerson);
?>
```

When executing this code, you will have a String - a series of characters - that represents the ConvisoPerson object:
```
O:13:"ConvisoPerson":5:{s:8:"username";s:6:"Antony";s:4:"team";s:5:"PTaaS";s:3:"age";i:17;s:6:"office";s:6:"Intern";s:12:"accountAdmin";b:0;}
```

# Understanding Serialized Object
Let's inspect the output from the code above. The structure of the serialized object in PHP is “DATA\_TYPE, DATA”. 

O -> Represents an object:
```
O:13:"ConvisoPerson":5:
    O: QUANTITY_OF_CHARACTERS: "NAME_OF_CLASS": NUMBER_OF_PROPERTIES:
```

s -> Represents a string:
```
s:8:"username";
    s: QUANTITY_OF_CHARACTERS: NAME_STRING;
```

b -> Represents a boolean:
```
b:0;
    b: BOOLEAN_VALUE;
```

The structure of the serialized object has 5 properties. The first is "username" with the value "Antony", the second "team" with the value "PTaaS", the third "age" with the value 17, the fourth "office" with the value "Intern" and the last "AccountAdmin" with the value 0 (meaning the boolean value False).

# Deserialization

To perform deserialization, use the `unserialize()` function on a serialized object. The purpose of this function is to restore the object that was serialized with all its properties.

```php
<?php
Class  ConvisoPerson
{
    public $username =  'Antony';
    public $team =  'PTaaS';
    public $age =  17;
    public $office =  'Intern';
    public $accountAdmin =  False;
    public  function  validateAdmin(){
    if ($this->accountAdmin){
        echo  ' [+] '  .  $this->username .  ' is administrator\n';
    } else {
        echo  ' [+] '  .  $this->username .  ' not is administrator\n';
        }
    }
}


$admin = new ConvisoPerson();
$admin_serialize = serialize($admin);


$admin_unserialize = unserialize($admin_serialize);
echo $admin_unserialize->validateAdmin();
```

# PHP magic methods
Magic methods are special functions preceded by \_\_. They modify some default PHP behaviors when executed on an object.

Learn more about these methods at:

- [https://www.php.net/manual/en/language.oop5.magic.php](https://www.php.net/manual/en/language.oop5.magic.php)

Some interesting methods in the context o serialization are `__sleep()` and `__wakeup()`.

The `__sleep()` function is called implicitly when we perform a serialization using `serialize()` function, this method must return an array with the name of the object properties that will be serialized.

The `__wakeup()` function is used to restore a possible communication with the database that may have been lost due to the serialization process and to perform other restart tasks.

# Exploiting deserialization in PHP

A potentially dangerous scenario is when the user has control of the serialized object. This allows an attacker to use existing application code in a harmful way, resulting in vulnerabilities such as code execution, privilege escalation, arbitrary file access, broken access control, and denial of service attacks.

An example of broken access control exploiting a variable manipulation, where the goal is to acquire administrator privileges, is the [PortSwigger Modifying Serialized Objects](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects) lab. After logging into the lab with the credentials provided in the challenge description, you will receive a base64 encoded session cookie.

![](https://i.imgur.com/vbSoLaL.png)

When you decode this base64, we will have the following string:

`O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}`

We can see that this is a serialized object. An interesting field to be manipulated is the administrator, which is a field with a boolean value.

First, let's change its value to 1

`O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}`

Next, let's take our modified string and encode in base64

`Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czo1OiJhZG1pbiI7YjoxO30=`

When you're done modifying the base64 serialized object, just remove the old session cookie value, insert what you changed and refresh the page.

![](https://i.imgur.com/uPNpKz9.png)

And it's done! The request has been accepted and you have admin privileges, we can verify these privileges with the Admin Panel option that appears on the upper right side.

# References

[A8 - Insecure Deserialization](https://ftp.registro.br/pub/gts/gts33/tutorial/A8%20-%20Insecure%20Deserialization.pdf)

[Intro to PHP Deserialization/Object Injection](https://www.youtube.com/watch?v=HaW15aMzBUM)

[Insecure deserialization](https://portswigger.net/web-security/deserialization)

[Exploiting PHP deserialization](https://medium.com/swlh/exploiting-php-deserialization-56d71f03282a)

[PHP Object Injection](https://owasp.org/www-community/vulnerabilities/PHP_Object_Injection)

[Diving into unserialize()](https://medium.com/swlh/diving-into-unserialize-3586c1ec97e)

[PHP: Serializing an unserializing the simple way](https://itnext.io/php-serializing-an-unserializing-the-simple-way-da25c0d9340d)
