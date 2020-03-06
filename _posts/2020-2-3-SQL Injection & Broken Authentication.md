---
layout: post
title:SQL Injection & Broken Authentication
---
# SQL Injection
SQL injection is an attack that takes advantage of improper user validation to exploit the database layer of an application. Additionally, user-supplied data when passed into the database engine without sanitization can cause leakage of secret information and possibly database corruption. This attack will begin by identifying a location in a webpage that has some form of input. At this point, simple SQL commands can be used as input to identify whether the webpage is effectively validating input. If the input is not properly validated, more targeted commands can be run to ultimately have the webpage return sensitive information such as user credentials from the database.

### Exercise

1. Go to (<http://sbr2020.herokuapp.com/>). You will see a login page. Enter `ash@yahoo.com` for the username and `pass123` for the password and click 'Submit'.

    ![Normal login](/images/week4_login.png)

    As the username and password are correct, the app will show the to-do lists.

    ![Todo lists](/images/week4_loggedin.png)

2. Go back to the login page by clicking 'Login' on the navigation bar. Now enter `ash@yahoo.com` for the username and `pass123` for the password and click 'Submit'. As the password is incorrect, you remain on the login page and are not allowed to see the to-do lists.

    ![Normal login](/images/week4_login.png)

3. We will now inject SQL using the Username input field in an attempt to bypass authentication. Enter `' OR 1=1;--` as the username and `a` as the password. Click 'Submit'.

    ![SQL Injection](/images/week4_injection.png)

    The app takes us to the page with the to-do lists even though we didn't enter valid login credentials. This is because the text we enter is parsed as SQL by the database. This is a typical SQL injection attack.

    ![Todo lists](/images/week4_loggedin.png)
 
**[Video of SQL Injection on Vulnerable Web App](https://media.oregonstate.edu/media/t/0_fsxkppgy/151221042)**

### Analysis

The relevant section of code which constructs and executes the SQL query is as follows:

```
if(email && password) {
    db.client.query(`SELECT * FROM users WHERE email='${email}' AND password='${password}'`, (err, result, rows)=>{
      if (result.rows.length === 0) {
        res.redirect('back');
        return;
      } else {
        console.log("AUTHENICATED!!!");
        res.redirect("/lists")
      }
    });
  }
```

The `db.client.query` uses a [Javascript template string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) as an argument. After substitution, the query expands to `SELECT * FROM users WHERE email='ash@yahoo.com' OR 1=1;--' AND password='a'`. The injected `OR 1=1` clause causes the authentication check to pass successfully even though valid credentials are not supplied.

This can be fixed by using a [PostgreSQL parameterized query](https://node-postgres.com/features/queries) instead of a Javascript template string. The `db.client.query` call can be modified as follows:
```
db.client.query({"text":"SELECT * FROM users WHERE email=$1 AND password=$2", "values":[email, password]}, (err, result, rows)=>{
```
With this simple change, Javascript no longer performs the substitution. The SQL template string and the user-supplied values are separately supplied to PostgreSQL which can safely do the substitution with full knowledge that the user-supplied strings should never be interpreted as SQL clauses. You can test the hardened version at (<http://sbr2020secure.herokuapp.com/>). The SQL injection technique described above will no longer bypass authentication.

**[Video of SQL Injection on Secure Web App](https://media.oregonstate.edu/media/t/0_qy3v7ymo/151221042)**

# Broken Authentication
Broken authentication is a form of vulnerability that an adversary can use to authenticate into a web page and impersonate a legitimate user. This can be done in many ways such as taking advantage of a web application that fails to lock out accounts after multiple failed attempts to brute force a weak password. Adversaries can also use information retrieved from the webpage after a failed login attempts to identify whether a username or password is correct. Improperly handling sessions by allowing users remain logged in for a long period of time despite leaving the computer unattended can enable an adversary to access the user’s session on that computer.     
### Exercise
For this exercise you will need to download and install Burp Suite Community Edition (<https://portswigger.net/burp/communitydownload>) a free cybersecurity tool.

1. Create a text file called username.txt and enter the following commonly used emails:
- <default@hotmail.com>
- <admin@msn.com>
- <user123@gmail.com>
- <janedoe@yahoo.com>
- <sql@aol>     

2. Create a text file called passwords.txt and enter the following commonly used passwords:
- admin
- P@ssword
- user123
- default
- monkey     

3. Open Burp Suite and go through the following menus    

   **- NEXT**

   ![Burp, Openning Page](/images/BurpOpen.JPG)    

   **- START BURP**     

   ![Burp, Start New Project Page](/images/BurpStart.JPG)        

   **- PROXY > OPTIONS**  Make sure there is a proxy listener set up for 127.0.0.1:8080 and that it is running.     

   ![Burp, Openning Page](/images/BurpProxyOptions.JPG)     

4. Open Firefox and click on the three horizontal lines on the upper right hand corner and click on OPTIONS or PREFERENCES (depending on what version you are on)      

   ![Firefox, Options Menu](/images/FirefoxOptions.JPG)     

   **- At the bottom of the page find Network Settings and click on SETTINGS.**     

   ![Firefox, Network Settings](/images/FirefoxNetwork.JPG)      

   **- Before you make any changes here make sure you make a note of the original settings. Now change the settings to Manual Proxy Configuration to match the picture below.**   

   ![Firefox, Proxy Configuration](/images/FirefoxProxy.png)    

5. On Firefox go to our vulnerable website at <http://sbr2020.herokuapp.com/login> and enter the following email and password combination: 
      - Email: fake@email.com
      - Password: bogusPass
      - Click the Sign In button  (Do not worry if nothing happens, the next steps are completed on Burp Suite)    

      ![Firefox, sbr2020 Web App](/images/loginPage.JPG)   

6. Open Burp Suite and go to PROXY > INTERCEPT and you should have something that looks similar to this.    

      **- Click on ACTIONS and SEND TO INTRUDER and stop the listener by clicking on INTERCEPT IS ON button. You should also at this time go to Firefox and turn your Network settings back to their original status.**    

      ![Burp, Intercept](/images/BurpProxyIntercept.JPG)   

      **- Go to INTRUDER > POSITIONS and change the values between the § signs to 'username' and 'passwords, and then change the attack type to CLUSTER BOMB ' as shown below**    

      ![Burp, Intruder Position](/images/BurpIntruderPosition.JPG)  

      **- Go to INTRUDER > PAYLOADS and make sure PAYLOAD SETS is set to 1 and on PAYLOAD OPTIONS click LOAD. Find the username.txt file you created and load. Then click select on PAYLOAD SETS set 2 and on PAYLOAD OPTIONS click LOAD and find the passwords.txt file you created and load.**   

      ![Burp, Intruder Payloads](/images/BurpPayload1.JPG)  

      **- Go to INTRUDER > OPTIONS > GREP MACTH and select clear.**

      ![Burp, Intruder Options](/images/BurpIntruderOptions1.JPG)  

      **- Go to INTRUDER > OPTIONS > REDIRECTIONS and select ON-SITE ONLY.**   

      ![Burp, Intruder Options](/images/BurpIntruderOptions2.JPG)  

      **- You are ready to start the attack. Go to the top of the OPTIONS page and on the right top hand you will see a START ATTACK button, click on it. A new window will open showing every combination of usernames and passwords being tried on the webpage**

      ![Burp, Attack Page](/images/BurpAttack.JPG)  

7. Analyze results from attack page and you will see on the length column only one is different from the others. Let's test these credentials on our webpage. **Go to  <http://sbr2020.herokuapp.com/login> and enter:**    

      - Email: janedoe@yahoo.com
      - Password: monkey   

      ![Firefox, sbr2020 Web App](/images/loginPage2.JPG)    

      **- You are now logged in using someone else's credentials obtained through a broken authentication attack!**

      ![Firefox, sbr2020 Web App](/images/Lists.JPG)   
      
**[Video of Broken Authentication on Vulnerable Web App](https://media.oregonstate.edu/media/t/0_ltwydozv/151221042)**

### Analysis
This broken authentication attack can be prevented by adding an account lockout procedure, a process of automatically disabling a user account based on too many failed logon attempts. To do this we added a column to the users table in the databse to keep track of failed logon attempts. After three attempts the account is locked and the brute force attack fails. 

```
db.client.query({"text":"SELECT * FROM users WHERE email=$1 AND password=$2 AND attempts < 3", "values":[email, password]}, (err, result, rows)=>{
```

**AND attempts < 3** being the relevant code. 

**[Video of Broken Authentication on Secure Web App](https://media.oregonstate.edu/media/t/0_g4wp9t8k/151221042)**
