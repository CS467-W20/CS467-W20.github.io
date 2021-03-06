---
layout: post
title: Security Misconfiguration 
---

# Security Misconfiguration
Security misconfiguration is commonly a result of insecure default configurations, incomplete or ad hoc configurations, open cloud storage, misconfigured HTTP headers, and verbose error messages containing sensitive information. Not only must all operating systems, frameworks, libraries, and applications be securely configured, but they must be patched/upgraded in a timely fashion.<sub>[1]</sub>

### Exercise
For this exercise we will be downloading OWASP ZAP, an open source penetration testing tool. Download the appropriate installer from ZAP’s download location at <https://www.zaproxy.org/download/> and execute the installer.

### DISCLAIMER:    
> You should only use ZAP to attack an application you have permission to test with an active attack. Because this is a simulation that acts like a real attack, actual damage can be done to a site’s functionality, data, etc. If you are worried about using ZAP, you can prevent it from causing harm (though ZAP’s functionality will be significantly reduced) by switching to safe mode.     
> To switch ZAP to safe mode, click the arrow on the mode dropdown on the main toolbar to expand the dropdown list and select Safe Mode.<sub>[2]</sub>

1. After installing, click on the ZAP icon and select No I do not want to persist this session at this moment in time, then click Start.

   ![ZAP Start](/images/Start.JPG)

2. On the welcome screen you will see three options, click on the Automated Scan. 

   ![ZAP Welcome](/images/Welcome.JPG)
   
3. Enter the following information before clicking on attack button: 
    - URL to attack: https://sbr2020.herokuapp.com
    - Use traditional spider: check 
    - Use AJAX spider: uncheck   

ZAP will proceed to crawl the web application with its spider and passively scan each page it finds. Then ZAP will use the active    scanner to attack all of the discovered pages,functionality, and parameters.    

   ![Zap Automated Scan](/images/Scan.JPG) 
    
 4. After the scan is complete go to the Alerts tab and click on each alert to read about each vulnerability. 
 
    **X-Frame-Options Header Not Set**    
    As you can see this is a Medium risk alert and the additional information provided shows the Description of the issue explaining why this makes the web app vulnerable and to which kind of attacks, a Solution and a Reference page.    
    Possible Attack: ClickJacking
    
    ![Zap Alert 1](/images/Xframe.JPG) 
    
    **Absence of Anti-CSRF Tokens**    
    Possible Attack: Cross-Site Request Forgery 

    ![Zap Alert 2](/images/Tokens.JPG) 
    
    **Cookie without SameSite Attribute**  
    ZAP highlights the piece of code and/or part of the request that has an issue. This along with the Solution and Reference windows makes it easier to find a solution for each vulnerability.    
    Possible Attack: Cross-Site Request Forgery, Cross-Site Script Inclusion, Timing attacks 
    
    ![Zap Alert 3](/images/cookie.JPG)
    
    **Cookie without secure flag**  
    Possible Attack: Sniffing unencrypted HTTP connection
    
    ![Zap Alert 4](/images/encrypt.JPG)
    
    **Cross-Domain Javascript Source File Inclusion**  
    If you include a script from an external domain, then you are trusting that domain with the data and functionality of your application, and you are trusting the domain's own security to prevent an attacker from modifying the script to perform malicious actions within your application.
    
    ![Zap Alert 5](/images/crossDomain.JPG)
    
    **Incomplete or No Cache-control and Pragma HTTP Header Set**  
    Possible Attack: Allows browser and proxies to cache content
    
    ![Zap Alert 6](/images/no-Cache.JPG)
    
    **Server Leaks Information via X-Powered-By**  
    Possible Attack: Access to frameworks/components information may facilitate attackers identifying known vulnerabilities 
    
    ![Zap Alert 7](/images/Leak.JPG)
    
    **X-Content-Type-Options Header Missing**  
    Possible Attack: MIME-sniffing 
    
    ![Zap Alert 8](/images/Sniffing.JPG)
    
    **Sensitive Information in URL**
    Possible Attack: Sensitive Data Exposure
    
    ![Zap Alert 9](/images/Information.JPG)   

**[Video of Security Misconfiguration on Vulnerable Web App](https://media.oregonstate.edu/media/t/1_ul2ib9aj/151221042)**

### Analysis

This week's exercise shows the importance of penetration testing and how a simple misconfiguration can lead to a major vulnerabilities on our site. 

**X-Frame-Options Header Not Set**     
> ClickJacking is an attack that occurs when an attacker uses a transparent iframe in a window to trick a user into clicking on a CTA, such as a button or link, to another server in which they have an identical looking window. The attacker in a sense hijacks the clicks meant for the original server and sends them to the other server. This is an attack on both the visitor themselves and on the server.<sub>[3]</sub>   

**Server Leaks Information via X-Powered-By**    
> Hackers can exploit known vulnerabilities in Express and Node if they know you’re using it. Express (and other web technologies like PHP) set an X-Powered-By header with every request, indicating what technology powers the server. Express, for example, sets this, which is a dead giveaway that your server is powered by Express.<sub>[4]</sub>     

**Note**: Showing you how to perform a ClickJacking attack and finding known vulnerabilities for Express is out of scope for this exercise.     

To enable X-Frame-Options we simply run the npm helmet package (Helmet helps secure Express apps by setting various HTTP headers)    
``` npm install helmet --save ```   
and added the following line of code in our app.js file.    
``` app.use(helmet.frameguard()) ```

To disable the X-Powered-By header to stop it from advertising we are using Express we add the following to the app.js file.   
``` app.disable('x-powered-by') ```

After enabling XFO and disabling x-powered-by we run the Automated Scan on our secure website and notice no X-Frame-Options or Server Leaks information alerts this time.

   ![Zap Secure](/images/Hardened.JPG)   

**[Video of Security Misconfiguration on Secure Web App](https://media.oregonstate.edu/media/t/1_eyxu9z55)**

### References
1. <https://owasp.org/www-project-top-ten/>  
2. <https://www.zaproxy.org/pdf/ZAPGettingStartedGuide-2.9.pdf>
3. <https://www.keycdn.com/blog/x-frame-options>
4. <https://helmetjs.github.io/docs/hide-powered-by/>
