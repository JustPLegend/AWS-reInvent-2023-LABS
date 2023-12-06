# AWS reInvent 2023 lab 
# Filtering and blocking web incursions with AWS WAF

## Lab overview

Welcome to the AWS Web Application Firewall Challenge Lab! AWS WAF is a web application firewall that helps protect your web applications or APIs against common web exploits and bots that may affect availability, compromise security, or consume excessive resources. AWS WAF gives you control over how traffic reaches your applications by enabling you to create security rules that control bot traffic and block common attack patterns, such as SQL injection (SQLi) and cross-site scripting (XSS). You can also customize rules that filter out specific traffic patterns.

In this lab, you are a security engineer working for a company that just launched an online juice store. You have been asked to look for and mitigate SQLi and XSS vulnerabilities in their web application.

Unlike traditional labs which included detailed, step-by-step instructions, this Challenge Lab provides you with objectives and only minimal direction. You will need to apply your knowledge of security and common web exploits to complete the tasks.

In the first set of tasks, you are presented with an insecure web application and prompted to use SQL injection and cross-site scripting attacks to exploit it. If you get stuck, expand the Hint and Solution menus for for assistance with your attacks.

In the second set of tasks, you deploy and configure AWS WAF with a series of managed and custom rules designed to mitigate the attacks you previously launched.

## OBJECTIVES
By the end of this lab, you will be able to do the following:

- Deploy simple SQLi and XSS attacks to compromise a web application.
- Create a web access control list (web ACL) in AWS WAF and associate it with an Application Load Balancer.
- Apply AWS managed rule groups to a web ACL.
- Apply custom rules to a web ACL.

## TECHNICAL KNOWLEDGE PREREQUISITES

Advanced knowledge of SQL, as well as a solid understanding of network security, common web protocols, and the RESTful framework is required for this lab. You should also be comfortable working in a Windows Server environment using Remote Desktop.

## Architecture

![architecture-waf](images/Filtering-and-blocking-web-incursions-with-AWS%20WAF-architecture.png)
## TASK 1 Configure OWSAP Zap and Firefox


In this task, you use Fleet Manager connect to a Windows Server EC2 instance. Once connected to the instance, you configure OWASP Zed Attack Proxy (ZAP) and Firefox. OWASP ZAP Proxy is an open source penetration testing tool maintained by the Open Worldwide Application Security Project (OWASP). As a “man-in-the-middle” proxy, ZAP intercepts all communications between a tester’s browser and a target web app. This allows pen testers to inspect, modify, and manipulate requests and responses to uncover vulnerabilities.

In a panel to the left of these instructions, you will find a series of URLs. Copy the URL labeled PenetrationTestingHost and paste it into a new browser tab.
You are brought to the Fleet Manager - Remote Desktop connection page.

**WARNING:** Please make sure that you are using the Chrome as the Internet browser because for Fleet Manager RDP, only Chrome browser supports bidirectional copying and pasting between RDP sessions and your local machine.

**Note:** If you are unable to use RDP with Fleet Manager, you can also connect to your windows instance using a Remote Desktop client.

- For Authentication type choose Key pair.

- From the panel to the left of these instructions, choose  Download PEM.

- Save the file to a directory of your choice.

- For Key pair content, choose Browse your local machine to select the key pair file.

- Select the Browse button to find and upload the PEM key from your local machine.

- Select the Connect button to launch an RDP session with the PenTestingHost.

**Caution**: Fleet Manager RDP connections have a maximum session duration of 60 minutes. When that duration is reached, Fleet Manager ends the session. If you run into any issues while interacting with your instance via Fleet Manager, open the Actions  drop-down menu, and then select Renew session to restart your session.

- Choose the icon with four arrows  to expand the window containing your RDP session.

**Note**: You will be prompted with a Networks pop-up window asking: Do you want to allow your PC to be discoverable by other PCs and devices on this network? Choose No.

----

- Choose the desktop shortcut labeled OWASP ZAP 2.12.0 to open the ZAP application.
The ZAP logging terminal appears, followed by the application splash screen. Wait for the configuration process to complete and do not close any windows that appear on the screen.

**Learn more: Refer to [OWASP Zed Attack Proxy](https://www.zaproxy.org/) (ZAP) for more information about OWASP ZAP.**

A popup window appears asking if you want to persist the ZAP Session. **Select Yes**, I want to persist this session with name based on the current timestamp and then choose the Start button.

- Choose the maximize button at the top of the screen to expand the ZAP Console window.

- Look for and choose the dark green circle on the toolbar. This is the Disable the ZAP HUD button.

- Choose the Manual Explore button in the center of the screen.

- You are brought to the Manual Explore page.

**Configure the following options:**
From the panel to the left of these instructions, copy the URL labeled JuiceShopURL and paste it into the URL to explore field in ZAP.
Leave the Enable HUD checkbox unselected.
Open the dropdown menu next to Explore your application and select Firefox.
Choose the Launch Browser button.

 Note: ZAP automatically configures Firefox to proxy through ZAP. This eliminates the need to worry about about certificate validation warnings for sites using HTTPS.

- Firefox opens and connects to the JuiceShop application.

## Finished task1 on [link](https://www.youtube.com/watch?v=20FckRSh8Lw&list=PLttknGhVwq_Jl6QbbVZ9VuTxHqX7S6EFk)

----

## TASK2 Reconnaissance

In this task, you conduct reconnaissance in the hope that it will yield useful information that will aid in your attack. Reconnaissance testing can help you determine which form fields are vulnerable to SQL injection and XSS exploits. You can also find error messages that may reveal details about the database schema or software version being used.

- TASK 2.1: GENERATE HTTP REQUESTS

In the following steps, you interact with the Juice Shop application. The HTTP requests generated by your interactions are intercepted and logged by the ZAP Console. In subsequent steps, you will modify and replay these requests to exploit the application.

Complete the following actions in the web interface:

- Use the search bar to search for apple juice

- Navigate to the registration page (https://[JuiceShopDomain].cloudfront.net/#/register) and create a new user.
- Navigate to the login page (https://[JuiceShopDomain].cloudfront.net/#/login) and log in as your new user.

![search-juice](Security-Identity-Compliance/images/juice-shop-search.png)

TASK 2.2: SQL INJECTION

The API requests that were sent to the application when you searched for apple juice, created a new user, and then logged in as that user were intercepted by ZAP. In the following steps, you use the ZAP Console to find these requests, modify them, and see if you can exploit the application.

- In the taskbar at the bottom of the Windows interface, choose the blue OWASP ZAP icon to return to the ZAP Console.

Now let’s use the ZAP Console to see if any of the input fields are vulnerable to SQL injection.

- Choose the Search button in the panel at the bottom of the ZAP Console.

In the following steps, you attempt to force the Juice Shop application to display an error message that provides you with information about the application and database.

 Note: Error-based injections take advantage of information contained in error messages to learn about the types of information stored in a database and its structure.

 Command: Enter apple juice in the Search field then press ENTER.

Among the search results, look for and select a GET request with the following URL: 

    https://[JuiceShopDomain].cloudfront.net/rest/products/search?q=. 
    
Open the context menu (right-click) and select Open/Resend with Request Editor.

The Manual Request Editor opens. It has two tabs - Request and Response, each of which is divided into two panels. Headers are displayed in the upper panel and the message body is displayed in the bottom panel. This window enables you to modify and replay HTTP requests. Start by testing to see if you can exploit the search field by passing a boolean attack in the query string.

 Note: HTTP query strings are the part of the URL that follows a question mark (?) and are used to send additional data to a web server as part of an HTTP request. When you submitted 

    apple juice

in the search bar, apple%20juice (the URL-encoded equivalent of apple juice) was sent to the API as a query string.

With the Request tab selected, place your cursor in the text area and update the URL in the first line of the header so that it ends with /search?q=' or 1=1 -- and then choose the Send button.

Note: An SQL boolean attacks is a type of SQL injection that manipulates queries using true/false conditions. In this case, the string ' or 1=1 --   alters the query so that it always return true, due to the  1=1 condition. The double hyphen -- comments out the rest of the query, preventing any further conditions from being applied.

The Response tab opens, displaying an unhandled exception. The message body is displayed in the lower panel. 

The response has provided you with exactly the information you were looking for and highlights the importance of proper error handling. Based on the response, you now know that the application uses an SQLite database.

 Note: Knowing the database type (MySQL, SQLite, Oracle, etc.) is crucial for crafting an effective SQL injection payload. Different database systems have different vulnerabilities and methods for gaining access. Reconnaissance helps ensure you develop an injection that is tailored to the specific configuration of the target database.

## Finished task2 on [link](https://www.youtube.com/watch?v=s1gBcL1C0kk&list=PLttknGhVwq_Jl6QbbVZ9VuTxHqX7S6EFk&index=2)

----
## TASK3 Persisted XSS attack
Now that we’ve discovered that the site is using SQLite and vulnerable to SQL injection, let’s see if it’s also susceptible to cross-site scripting attacks. In the following steps, you use a persisted XSS attack to embed a script in the list of registered users.

 Note: A persisted XSS attack, also known as a stored XSS attack, occurs when an attacker injects malicious scripts into a web application’s database or other permanent storage. This injected code is then served to users when they access the affected web pages, causing the malicious script to execute within their browsers. The attacker can use this script to steal sensitive information, manipulate web content, or perform other unauthorized actions on behalf of the user.

Close the Manual Request Editor window and return to the search bar.

 Command: In the Search bar enter  api/Users and press ENTER.

From the search results, choose the POST request whose URL ends with /api/Users, open the Context menu (right-click) and select Open/Resend with Request Editor.

The request opens in the Manual Request Editor.

 Note: This is the request that was sent to the server when you created a new user in the application interface. It is a POST request sent to the Users endpoint and includes a variety of standard headers including a User-Agent, the Content-Type, and a cookie. The message body should look similar to the following:

    {"email":"myuser@example.com","password":"123456","passwordRepeat":"123456","securityQuestion":{"id":1,"question":"Your eldest siblings middle name?","createdAt":"2023-04-19T15:48:55.057Z","updatedAt":"2023-04-19T15:48:55.057Z"},"securityAnswer":"123456"}

 It’s possible that not all of the data included in this message body is required to create a new user. Let’s try to create a new user using only the email, password, and passwordRepeat keys and insert the XSS payload into the email key-value pair.

Command: Place your cursor inside of the message body panel and replace the existing message with the following text:

    {"email":"<iframe src=\"javascript:alert(`xss`)\">","password":"123456","passwordRepeat":"123456"}

----
The following list explains the components of the script:

**iframe:** Creates an iframe element

**src=:** Sets the iframe’s source (src) attribute to a JavaScript code snippet
javascript:alert(

**xss**
): Triggers an alert with the message “xss”


----
Choose the Send button to replay the request with your modified payload.


Great work! The Response tab opens and displays a message stating that your user was created. Your malicious script has now been written to the database. In the future, if another someone views this user’s profile, the script will launch inside of their browser.

 Note: The JSON response includes additional fields such as deluxeToken, and role that you did not include in your request. This suggests that the Users table in the database includes attributes that are not visible in the website UI. These may come in handy in subsequent tasks.

## Finished task3 on [link](https://www.youtube.com/watch?v=1M-ByiKqUMY&list=PLttknGhVwq_Jl6QbbVZ9VuTxHqX7S6EFk&index=3)

----

## CHALLENGE - 

## TASK 4  Bypass the login page

This is the first challenge task in this lab. Use what you have learned about ZAP, SQL injection attacks, and the Juice Shop database to complete the task. If you need assistance, expand the Hint menus to reveal additional information. If you are unable to complete the task, expand the Solution menu and follow the instructions inside it to achieve your objective. Do not proceed to the next task without first completing this challenge.

OBJECTIVE
Your objective in this challenge is to bypass the Juice Shop login page.

CHALLENGE SUB-TASKS
Navigate to https://[JuiceShopDomain].cloudfront.net/#/login.
Find an exploit that enables you to log into the website as an authenticated user.


HInt1: Try completing this challenge using only Firefox. You should be able to exploit the login page without replaying any requests in ZAP.

SOLUTION: 

Choose the Account button at the top of the screen and then select Login from the dropdown menu that appears.

 Command: In the Email field, enter 

' or 1=1 --

 Command: In the Password field, enter 

123456

Choose the Log in button.

You are brought back to the main application window, but this time you are logged in as an authenticated user.

You bypassed the login authentication by manipulating the SQL query. The application was supposed to find records from the Users table matching the email and password you provided (note that 123456 was converted into an MD5 hash) and verify that the account had not been deleted (deletedAt IS NULL).

However, because you inserted 'or 1=1 – in the email field, the query’s logic was changed. The query starts by looking at the email column in the Users table for ‘’. Normally this would not return any results, because ‘’ is interpreted as an empty string. However, because you added the or logical operator followed by a condition that is always true (1=1), the query matches all rows in the Users table. Furthermore, since – comments out the rest of the query, the the AND password and AND deletedAt IS NULL conditions are ignored. Essentially, this query returns all users in the database and then logs you in as the user with id=1.

## TASK5 CHALLENGE ~~

 Challenge - Create a new admin user
This is the second challenge task in this lab. Just like the previous challenge, if you need assistance, you can expand the Hint and Solution menus. Do not proceed to the next task without first completing this challenge.

OBJECTIVE
Your objective in this challenge is to exploit the user registration page and create a new user with administrative privileges.

CHALLENGE SUB-TASKS
Use the ZAP Console to find an exploit that enables you to create a user with administrative privileges.

HINT 1: 
Search for an API request sent to 

/api/Users
 that created a new user. Examine the Request and Response. Can you identify any differences between them?

HINT 2:

Recall that you were able to create a new user without including the securityQuestion key-value pair. Perhaps there are other key-value pairs that could be added to a request?

**SOLUTION:**

Return to the ZAP Console.

 Command: Use the Search bar to look for requests sent to the api/Users endpoint. Choose one of them, open the Context menu (right-click) and select Open/Resend with Request Editor.

Compare the message bodies in the Request and Response tabs. Note that the Response payload contains additional key-value pairs and that one of them is “role”:“customer”.

It seems possible that if there is a customer role, then there are likely other roles as well. 

    admin

is a common role name for administrators. In the following steps, you modify the API request to pass “role”:“admin” in the message body.

- Choose the Request tab and scroll down to the payload.

 Command: Place your cursor inside of the Message body panel and replace the payload with the following text:

    {"email":"admin@example.com","password":"123456","passwordRepeat":"123456","role":"admin"}

- Choose the Send button at the top of the window.

    ******************************
    **** This is OUTPUT ONLY. ****
    ******************************

    {"status":"success","data":{"username":"","deluxeToken":"","lastLoginIp":"0.0.0.0","profileImage":"/assets/public/images/uploads/defaultAdmin.png","isActive":true,"id":23,"email":"admin@example.com","role":"admin","updatedAt":"2023-04-20T02:09:42.966Z","createdAt":"2023-04-20T02:09:42.966Z","deletedAt":null}}



--------

## TASK 6 Creating a Web ACL in AWS WAF

Now that you’ve confirmed that the application is vulnerable to SQLi and XSS attacks, it’s time to defend it. In this task, you create a web access control list (web ACL) in AWS WAF. A web ACL is a web application firewall that lets you monitor the HTTP and HTTPS requests to AWS resources.

In the top-right corner of the screen, choose the icon with four arrows  to minimize the window containing your RDP session.

At the top of the AWS Management Console, in the search bar, search for and choose  WAF & Shield.

- Choose the Create web ACL button.

Before entering any configuration details, under Resource type, select  CloudFront distributions.

The page should refresh.

In the panel at the top of the screen, enter the following configuration:
    Name: JuiceShopACL
    Description: Managed Rule sets and custom rules for JuiceShop
    CloudWatch metric name: JuiceShopACL
    Resource type: Amazon CloudFront distributions
    Scroll to the bottom of the screen and in the Associated AWS resources panel, choose Add AWS resources.
    The Add AWS resources popup appears.

- Select the checkbox next to your Cloudfront distribution and then choose Add.

- Choose the Next button.

- You are brought to the Add rules and rule groups page.

In the Rules panel, open the Add rules  dropdown menu and select Add managed rule groups.
Start by adding managed rule groups to your ACL. Managed rule groups are collections of predefined, ready-to-use rules created and maintained by AWS and AWS Marketplace sellers.

At the top of the list of managed rule groups, select the  AWS managed rule groups dropdown menu.

- Scroll down the page to the Core rule set and choose the Add to web ACL toggle button next to it.

 Note: The Core rule set contains rules that are generally applicable to web applications. This group provides protection against exploitation of a wide range of vulnerabilities, including those described in OWASP publications.

- Continue down the list and choose the toggle button next to SQL database.
 
 Note: The SQL database rule group contains rules to block request patterns associated with exploitation of SQL databases, including SQL injection attacks.

- Scroll to the bottom of the page and select Add rules.
- The managed rule groups you added are displayed at the top of the screen.

Note: Examine the panel immediately below the rules. Your ACL is currently consuming 900 out of a possible 5000 web ACL capacity units (WCUs). Applying ACLs requires compute resources, which are measured in WCUs. AWS WAF calculates capacity differently for each rule type, to reflect each rule’s relative cost. Simple rules that cost little to run use fewer WCUs than more complex rules that use more processing power. For example, a size constraint rule statement uses fewer WCUs than a statement that inspects requests using a regex pattern set.

- Scroll to the bottom of the page and choose the Next button.
- You are brought to the Set rule priority page. This page allows you to change the order in which rules are applied.

- Leave the default priority unchanged and choose the Next button.

- Do not change any of the metrics settings, scroll to the bottom of the page, and choose the Next button.

- Take a moment to review your ACL and then choose the Cerate web ACL button at the bottom of the page.

## TASK 6 finished on the [link](https://www.youtube.com/watch?v=ywRA-O7nR1k&list=PLttknGhVwq_Jl6QbbVZ9VuTxHqX7S6EFk&index=4)

----

## TASK7  Testing your web ACL

In this task, you replay the attacks you launched at the earlier in this lab, to verify that your newly created web ACL mitigates them.

From the panel to the left of these instructions, copy the URL labeled PenetrationTestingHost and paste it into a new browser tab to return to your Fleet Manager session.

If your existing session has ended, choose the Try again button and use the PEM key you previously downloaded to reconnect to the PenetrationTestingHost.

- Choose the icon with four arrows  to expand the window containing your RDP session.

Let’s start by checking to see if the ACL will prevent users from exploiting the search bar with a boolean attack.

- Open the ZAP Console and select the Search tab.

 Command: Enter products/search in the Search bar and then select the Search button.

- Locate and choose the **GET** request in the search results whose URL ends with  /rest/products/search?q=

- Open the context menu (right-click) and select Open/Resend with Request Editor.

The Manual Request Editor opens.

 Command: Update the end of the URL in the first line of the request to /rest/products/search?q=' or 1=1 --

and then choose the Send button.

Excellent! So far, you ACL is working as expected. Now let’s see if your ACL prevents attackers from exploiting the login page.

- Close the Manual Request Editor.

 Command: Return to the Searches tab and enter rest/User/login in the Search bar.

- Select any of the matching results, open the context menu (right-click), and choose Open/Resend with Request Editor.

 Command: Choose the Request tab and replace the payload with the following text:

    {"email":"' or 1=1 --","password":"123"}

- Choose the Send button.

Once again, your managed rule groups have successfully defended the Juice Shop application. Now let’s see how they handle an XSS attack.

 Command: Return to the Search tab and enter  **api/Users** in the Search bar.

Locate the POST request that created the xss@eample.com user. Select it, open the context menu (right-click), and choose Open/Resend with Request Editor.

 Command: Choose the Request tab and replace the payload with the following text:

    {"email":"<iframe src=\"javascript:alert(`xss`)\">","password":"123456","passwordRepeat":"123456"}

- Choose the Send button.

Uh-oh! A new user with “role”:“admin” was created. This is not surprising, as the message body did not include anything obviously malicious. One can easily imagine many legitimate situations in which an API request could contain a string like “role”:“admin”. In this specific case, however, you have identified this as a critical vulnerability and will need to create a custom WAF rule to block such requests.

## TASK7 Finished on the [link](https://www.youtube.com/watch?v=RNm3wJDD1ms&list=PLttknGhVwq_Jl6QbbVZ9VuTxHqX7S6EFk&index=5)

----

## TASK 8 Working with the AWS WAF rule builder

The managed rule groups you applied in the previous task blocked most of your attacks, but one of them got through. In this task, you use the AWS WAF rule builder to create a custom WAF rule that designed to mitigate the attacks not prevented by the managed rule groups.

In the top-right corner of the screen, choose the icon with four arrows  to minimize the window containing your RDP session.

- At the top of the AWS Management Console, in the search bar, search for and choose WAF & Shield.

- Choose the JuiceShopACL link.

The JuiceShopACL page appears, displaying a chart of requests inspected by the web ACL.

- Open the Rules tab.

- Choose the Add rules button and then select Add my own rules and rule groups from the dropdown menu.

 Note: Custom rules enable you to build your own logic for processing requests and provide a higher degree of flexibility and control. Rules can inspect many aspects of a request, including its headers, query strings, URI path, and body. Then, actions are applied to the request according to the conditions specified in the rule.

In the the following steps, you configure a custom rule that prevents the  api/Users endpoint from accepting requests containing “role”:“admin” in the message body.

 Security: Give careful thought to the statements included in your rules. Rules that are too broad may result in false positives. On the other hand, granular rules - particularly those that involve JSON parsing - consume significantly more WCUs.

- In the Rule type panel, select Rule builder.

- Scroll down the page to the Name field and enter  CustomSQLiRuleForJuiceShop.

- Open the If a request dropdown menu and select matches all the statements (AND).

 Note: Multiple statements can be chained together using AND, OR, and NOT operators.


In the **Statement 1** panel, enter the following configuration:

    Negate statement results: Leave checkbox unselected
    Inspect: URI path
    Match type: Contains string
    String to match:  api/Users
    Text transformation: None

Note: This statement should prevent the search bar from being exploited. It blocks all requests that include a query string that start with an apostrophe (').

In the **Statement 2** panel, enter the following configuration:

    Negate statement results: Leave checkbox unselected
    Inspect: Body
    Content type: Plain text
    Match type: Contains string
    String to match: "role":"admin"
    Text transformation: None
    Oversize handling: Continue - Inspect the contents that are within the size limitations according to the rule inspection criteria

- In the Action panel, select Block.

 Note: This statement should prevent the user registration page from being exploited. It blocks all requests whose URI includes the string  api/Users and whose message body includes “role”:“admin”. Note that this rule is only applied to requests passing through Cloudfront. Site administrators will still be able to create new admin users by connecting directly with the Juice Shop server.

- Now that your rule has been configured, take a moment to confirm that it is valid.

- Scroll to the top of the page and choose the Validate button above the Name field.
- A banner appears confirming that the rule is valid.

- At the bottom of the page, choose the Add rule button.



## Finished task8 on the [link](https://www.youtube.com/watch?v=OLOMw33XvLM&list=PLttknGhVwq_Jl6QbbVZ9VuTxHqX7S6EFk&index=6)

----

## Task 9: Test the custom rule


In this task, you return to the OWASP ZAP Console and again attempt to exploit the search and user registration functionality in the Juice Shop application.

Retrieve the URL labeled PenetrationTestingHost from the panel to the left of the instructions and paste it into a new browser tab.

If your previous RDP session has ended, choose the Try again button and use the PEM key to reconnect to the PenetrationTestingHost.

- Choose the icon with four arrows  to expand the window containing your RDP session.

 Command: Return to the Search tab and enter  api/Users in the Search bar.

- Select any of the matching POST requests, open the context menu (right-click), and choose Open/Resend with Request Editor.

- Choose the Request tab and replace the payload with the following text:

    {"email":"newadmin2@example.com","password":"123456","passwordRepeat":"123456","role":"admin"}

Choose the Send button to send the malicious payload again.

Output:

******************************
**** This is OUTPUT ONLY. ****
******************************

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<HTML><HEAD><META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=iso-8859-1">
<TITLE>ERROR: The request could not be satisfied</TITLE>
</HEAD><BODY>
<H1>403 ERROR</H1>
<H2>The request could not be satisfied.</H2>
<HR noshade size="1px">
Request blocked.
We can't connect to the server for this app or website at this time. There might be too much traffic or a configuration error. Try again later, or contact the app or website owner.
<BR clear="all">
If you provide content to customers through CloudFront, you can find steps to troubleshoot and help prevent this error by reviewing the CloudFront documentation.
<BR clear="all">
<HR noshade size="1px">
<PRE>
Generated by cloudfront (CloudFront)
Request ID: BWqQgCAH_8UUbEnDXWl2lwVq6zJ8ejzwBgouZ-79SNepchHkntX-Xw==
</PRE>
<ADDRESS>
</ADDRESS>
</BODY></HTML>


Your custom rule blocked the attack. Of course, it’s possible that the application remains vulnerable to other attacks and it would be wise to enable logging so that you can gather additional information about traffic passing through your web ACL. These logs can be sent to an Amazon CloudWatch Logs log group, an Amazon Simple Storage Service (Amazon S3) bucket, or an Amazon Kinesis Data Firehose.

Conclusion

- You have successfully done the following:
- Deployed simple SQLi and XSS attacks to compromise a web application.
- Created a web access control list (web ACL) in AWS WAF and associated it with an Application Load Balancer.
- Applied AWS managed rule groups to a web ACL.
- Applied custom rules to a web ACL.


## Finished task9 on the [link](https://www.youtube.com/watch?v=s2vkQkhSdv4&list=PLttknGhVwq_Jl6QbbVZ9VuTxHqX7S6EFk&index=7)
