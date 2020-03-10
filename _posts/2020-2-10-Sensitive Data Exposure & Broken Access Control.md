---
layout: post
title: Sensitive Data Exposure & Broken Access Control
---
# Sensitive Data Exposure
Sensitive data exposure is the process of exploiting a lack of cryptography within a webpage to attain sensitive user information. This can be done due to maintaining a poor API design that can inadvertently cause sensitive data to be exposed to parties who should not have access. Additionally, an adversary can take advantage of a web application that stores user credentials in plaintext or using weak cryptographic algorithms. This will require an ample amount of time researching the proper use of hashing to protect passwords against offline brute-force attacks.

### Exercise (This needs to be run locally to work effectively) 

 **Setting up a local instance of the application:**     
 1. Clone repository https://github.com/ashwinsawant/sbr.git to a local directory     
 2. Download a local instance of the SBR Database:     
    a. Install heroku (sudo apt-get install heroku)     
    b. Install postgres (sudo apt-get install postgresql)     
    c. Pull local instance of database with command: heroku pg:pull DATABASE sbr --app sbr2020     
    d. Access postgres with command: sudo su postgres      
    e. Enter psql with command: psql     
    f. Make sure you are in list of foles with comand: \dt      
        f1. If user does not have role, enter: CREATE ROLE "username" WITH CREATEDB     
        f2. If user did not have a role, step c might have to be repeated     
 3. Access cloned directory and begin instance of application with "npm start"

Requirement Tools: Install a tool that allows a user to capture and view packet captures such as tcpdump or Wireshark. For this exercise we will use tcpdump to capture the packets and Wireshark to view the captured packets.

1. Run command "sudo tcpdump -i any -w capture.pcap". This command listens to any interface on a workstation using the -i option and writes the packets to a file using the -w option. Once this is running, tcpdump will display a message stating that the tool is listening on the specified interface. 
 
    ![tcpdump Command](/images/SensitiveDataExposure/tcpdump.JPG)

2. Run command "npm start" in the directory containing the SBR application. This will start the web application on the workstation's localhost with port 3000. If this command is executed correctly, the following will be seen through the terminal. 
    
    ![npm start Command](/images/SensitiveDataExposure/NPMStart.JPG)


3. Type the following entry in a web browser "localhost:3000/login". This will bring up the login page for the SBT web application. When prompted for an email address and password enter the following: email: user123@gmail.com password: 123user. If this is executed correctly, you will be redirected to the localhost:3000/lists page. 

    ![Successful Login](/images/SensitiveDataExposure/LoginLists.JPG)

4. Once this is complete enter ctrl+c on the terminals containing the tcpdump capture and the web application wince we no longer need them running. 

5. On the same window as the tcpdump capture, enter the command "Wireshark capture.pcap" to open up the captured packets in Wireshark. This will open up a Wireshark window containing the capture.pcap file.  

    ![Wireshark](/images/SensitiveDataExposure/Wireshark.JPG)

6. Enter ctrl+f and search for GET under the "String" portion of the packets. Enter find until you discover the packet containing the GET request used to authenticate with the web application. This packet will contain the username and password of the user in plaintext.

    ![Packet Capture](/images/SensitiveDataExposure/PacketCaptureDet.JPG)

**[Video of Sensitive Data Exposure on local host](https://media.oregonstate.edu/media/t/1_k9kk9jem)**

# Broken Access Control
Broken authentication is a form of vulnerability that an adversary can use to authenticate into a web page and impersonate a legitimate user. This can be done in many ways such as taking advantage of a web application that fails to lock out accounts after multiple failed attempts to brute force a weak password. Adversaries can also use information retrieved from the webpage after a failed login attempts to identify whether a username or password is correct. Improperly handling sessions by allowing users remain logged in for a long period of time despite leaving the computer unattended can enable an adversary to access the user’s session on that computer. Due to the time constraints, the web application did not maintain the functionality to take advantage of broken access control. Instead, this exercise implements XSS do display a user's cookie. With this attack, an adversary can set up a separate application to steal the cookie which can later be used to potentially hijack the user's session. 

### Exercise 

1. Access https://sbr2020.herokuapp.com and login with the following credentials username: user123@gmail.com password: 123user.

    ![Login Page](/images/cookie/login.JPG)

2. Take advantage of the vulnerable webpage by modifying the URL: https://sbr2020.herokuapp.com/lists to https://sbr2020.herokuapp.com/lists/1. This allows you to access a specific list owned my another unknown user within the web application.

    ![Vulnerable Page](/images/cookie/login.JPG)

3. Type the following script into the input area and save the script into the web application. This will upload the script into the database used to interact with the web application.

    ![Type Script](/images/cookie/scriptInput.JPG)

4. Wait for unsuspecting user to access the modified list.

5. Acting as a potential victim, access https://sbr2020.herokuapp.com and login with the following credentials: username cocho@gmail.com and password: password.

    ![Unsuspecting User](/images/cookie/unsusUser.JPG)

6. Select Groceries list by selecting the update button. 

    ![Unsuspecting User](/images/cookie/user.JPG)

7. This will cause the user's cookie to be displayed out to the screen. An aversary can potentially use a similar attack to send the user's cookie to a seperate location rather than to the screen. These cookies can later be used to hijack the user's session. 

    ![Unsuspecting User](/images/cookie/cookie.JPG)


**[Video Containing Cookie Stealing Example](https://media.oregonstate.edu/media/t/0_kbxg6swk)**

