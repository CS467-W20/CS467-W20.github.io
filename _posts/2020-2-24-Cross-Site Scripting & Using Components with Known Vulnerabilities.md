---
layout: post
title: Cross-Site Scripting & Using Components with Known Vulnerabilities
---
# Cross-Site Scripting
Cross-Site scripting is a method of exploitation that injects code whether that be reflected or stored on to a webpage. Stored cross-site scripting attacks grant an adversary the ability to upload malicious code onto a database accessed by the targeted web-application. Once this code is successfully uploaded to the web application, the adversary waits for an unsuspecting user to access the targeted web application and run the malicious code. This code can allow an adversary to do a variety of attacks that range from stealing user cookies, to cause users to be redirected to a malicious web application.

## Video Containing Exercise
https://media.oregonstate.edu/media/t/0_d5zn643a

## Exercise
For this exercise we will be conducting a stored cross-site scripting attack on https://sbr2020.herokuapp.com. The attack will cause other unsuspecting users to be redirected to a seperate webpage.

1. Access https://sbr2020.herokuapp.com and login with the following credentials username: user123@gmail.com password: 123user.

 ![Normal login](/images/xss/one.JPG)

2. Take advantage of the vulnerable webpage by modifying the UTL: https://sbr2020.herokuapp.com/lists to https://sbr2020.herokuapp.com/lists/1. This allows you to access a specific list owned by another unknown user within the web application.

 ![Modify_URL](/images/xss/two.JPG)

3. Type the following script: into the input area and save the script into the web application. This will upload the script into the database used to interact with the web application.

 ![Script](/images/xss/three.JPG)

4. Wait for unsuspecting user to access the modified list. 

5. Acting as a potential victim, access https://sbr2020.herokuapp.com and login with the following credentials: username cocho@gmail.com and password: password. 
 
 ![NormalUser](/images/xss/four.JPG)
 
6. Select Groceries list by selecting the update button. This will access the list from the database and run the script previously uploaded to the web application and redirect the user to a potentially malicious web application. 

 ![NormalLsit](/images/xss/five.JPG)
 
7. You are now redirected to for this example, however, this can easily be a malicious web application owned by an adversary.

 ![Redirect](/images/xss/six.JPG)


# Using Components with Known Vulnerabilities
Using components with known vulnerabilities is one way users and organizations as a whole fall victim to exploitation. Adversaries routinely keep themselves updated with the most up to date vulnerabilities with tools such as CVE (Common Vulnerabilities and Exposures) lists which publish the most recent known vulnerabilities for computing technology. Once a vulnerable technology within a webpage is identified, an adversary can use it to find an avenue to more sensitive or critical information such as user credentials.

[**CVE-2017-5941**](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5941) An issue was discovered in the node-serialize package 0.0.4 for Node.js. Untrusted data passed into the unserialize() function can be exploited to achieve arbitrary code execution by passing a JavaScript Object with an Immediately Invoked Function Expression (IIFE).      

**Note:** Since this specific vulnerability has to do with serialization we will tackle it in our next week in the Insecure Deserialization Exercise.  

### Analysis 
Prevent a known vulnerabilities attacks by:     
- Continuously monitor sources like [CVE](https://cve.mitre.org/) (Common Vulnerabilities and Exposure) and [NVD](https://nvd.nist.gov/) (National Vulnerabilities Database). Subscribe to security bulletins related to the components you use.    
- Inventory both client side and server side components and their dependencies. 
- Remove any unused dependencies, features, components, files and documentation. 
- Only obtain components from official sources over secure links. Prefer signed packages  to reduce the chance of including malicious  or modified code. 

### References
1. https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A9-Using_Components_with_Known_Vulnerabilities
2. https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5941
