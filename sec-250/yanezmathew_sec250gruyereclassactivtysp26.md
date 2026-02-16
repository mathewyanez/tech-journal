# YanezMathew\_SEC250GruyereClassActivtySP26

![](<../.gitbook/assets/Unknown image (6)>)

**Google Gruyere Class Activity**

**Objective:**

In this activity, you will get to navigate the Google Gruyere project and carry out some exploits allowing you to do various types of injection attacks.

**Instructions:**

**Part 1: Open Gruyere**

1. In a web browser navigate to the Google Gruyere website at [https://google-gruyere.appspot.com/](https://google-gruyere.appspot.com/).

* _Click on the “Continue” button at the bottom of the page. Keep this page open, this will be where you can find challenges along with solutions_

![](<../.gitbook/assets/Unknown image (7)>)

1. In a new tab, open [https://google-gruyere.appspot.com/start](https://google-gruyere.appspot.com/start).

* _Click the start button. You should see a very plain homepage:_

![](<../.gitbook/assets/Unknown image (8)>)

1. Click on the “**sign up**” option in the top right corner of Google gruyere. Create two new accounts with basic and memorable credentials.
2. Sign into one of the created accounts and post a snippet by clicking “**New snippet**”\*\*\*\*in the top of the Google Gruyere toolbar. Post a snippet with a simple sentence about anything.

**Part 2: Gruyere Sprint!**

Now attempt to do as many of the Gruyere activities as you can **(at least 10)**! Have fun with the activities! And remember; **don’t try this on normal websites**. **You can skip the XSSI activity**. A completed lab includes the following information for each challenge:

* A screenshot of **every** step you take to complete it
  * Make sure to include a description with the screenshots so that I can understand what you did.
* A screenshot of the result of your “attack”
* Answers to the following questions:
  * Look at the [OWASP Top 10](https://owasp.org/Top10/2025/) site; what categories would the vulnerabilities you found fit into, if any?
  * What kind of impact could this vulnerability have?
  * How would you fix this vulnerability?

**These may require additional research!**

**Example of a Completed Challenge:**

**Step 1:** Created an HTML website that would send an alert to the user.

![](<../.gitbook/assets/Unknown image (9)>)

\*\*Step 2:\*\*Uploaded the html file to my Google Gruyere website.

![](<../.gitbook/assets/Unknown image (10)>)

![](<../.gitbook/assets/Unknown image (11)>)

**Step 3:** Navigated to the uploaded document.

![](<../.gitbook/assets/Unknown image (12)>)

**Challenge 1 Reflection Question Responses:**

Look at the [OWASP Top 10](https://owasp.org/Top10/2025/) site; what categories would the vulnerabilities you found fit into, if any?

**Response:**_I believe this vulnerability falls into the_[_A05:2025 - Injection_](https://owasp.org/Top10/2025/A05_2025-Injection/)\_ category. An HTML file I created is being injected into a web page. The file has a script embedded that creates a pop up alert as pictured above.\_

What kind of impact could this vulnerability have?

**Response:**_This file upload XSS vulnerability allows an attacker to execute malicious code in a victim’s browser, effectively giving them the same access as the victim within the application. An attacker could steal session cookies and take over user accounts, access and exfiltrate sensitive data, and perform actions on behalf of the user such as posting content, sending messages, changing account settings, or initiating transactions._

How would you fix this vulnerability?

**Response:**_To fix this, user-uploaded content should be kept separate from the main website. Instead of hosting it directly on the same site, it should be placed on a different web address that clearly shows its user content. This separation means that even if someone uploads something harmful, it can’t access or interfere with the main website or user accounts._

_A01:2021 - Broken Access Control (Attacks: 1, 2, 3, 7, 8, 9, 10)_

_A03:2021 - Injection (Attacks: 3, 4, 5, 6, 10)_

_A05:2021 - Security Misconfiguration (Attacks: 4, 6, 9, 10)_

_A07:2021 - Identification and Authentication Failures (Attack: 8)_

_A04:2021 - Insecure Design (Attacks: 2, 7)_

_**AttacK 1: PATH TRAVERSAL ATTACK**_

_**Attack 1:**&#x53;o basically I just typed in the path to some secret file while not even logged in, and the site just... gave it to me. No authentication, no authorization, nothing. Just straight up handed over the goods._

_This is pretty bad. An attacker can read literally any file on the server they want: passwords, config files, other users' data, source code, you name it._

_To fix it don't let users put file paths in requests, period. If you absolutely have to, use a whitelist of allowed files only. Never directly use what the user types to grab files. Also, check if they're actually logged in and have permission before serving anything. Use indirect references like file IDs instead of actual paths, and make sure everything resolves to a safe directory. It's really that simple, just don't trust user input for file access._

![](<../.gitbook/assets/Unknown image (13)>)

_Typed the path to mans file while logged out::_

![](<../.gitbook/assets/Unknown image (14)>)

_Successfully viewed the secret message:_

![](<../.gitbook/assets/Unknown image (15)>)

_**Attack 2: DOS ATTACK**_

_**I literally just added /quitserver to the end of the URL and the entire server shut down. This completely bricks the whole application for everyone. Nobody can use it, business loses money, angry customers. . If you need admin functions, put them behind actual security: multi-factor authentication, role checks not just out there.**_

![](<../.gitbook/assets/Unknown image (16)>)

_Put /quitserver at the end of the URL and shut down the server._

_**AttacK 3:**_

_**I made a username called ../resources and it completely broke the site. After learning it corrupts your user database and file system, I learned that you need to validate usernames properly only allowing normal characters.**_

![](<../.gitbook/assets/Unknown image (17)>)

_**Used a bit of path traversal making a user called ../resources.**_

![](<../.gitbook/assets/Unknown image (18)>)

_**After running it, the site appears to not be working. I am not sure why, it did nothing besides me not being able to log into the resources account anymore. I now need to reset the site.**_

_**AttacK 4:**_

_**File Upload XSS**_

My script is executed in their browser This is really bad. I can steal people's session cookies and take over their accounts. Force files to download instead of displaying them, and scan uploads for malicious files.

![](<../.gitbook/assets/Unknown image (19)>)

_Uploaded an exploit and got the site to run my script._

_**AttacK 5:**_

_**The site just took what was in the URL and dumped it straight into the page. This can steal SO much information. treat everything from the user as hostile.**_

_**Reflected XSS:**_![](<../.gitbook/assets/Unknown image (20)>)

_**URl will run script on user's browser.**_

_**AttacK 6: Reflected XSS via AJAX**_

_**Browsers labeled the GTL files as text html so they would execute them as web pages with scripts. Now any file becomes a potential attack vector. Always set the correct Content-Type header for responses.**_

![](<../.gitbook/assets/Unknown image (21)>)_**Caused a bug that caused Gruyere to return all gtl files as content type text/html.**_

_**AttacK 7: Elevation of Privledge**_

_**I just added is\_admin=True to a URL parameter when saving my profile, impact being I now have Full admin control over everything.Store user roles on the server in the session, not in requests**_

![](<../.gitbook/assets/Unknown image (22)>)

_**After making the following request:**_[_**https://google-gruyere.appspot.com/595358199394003877103001395094130363587/saveprofile?action=update\&is\_admin=True**_](https://google-gruyere.appspot.com/595358199394003877103001395094130363587/saveprofile?action=update\&is_admin=True)

_**Managed to gain an admin account which now allows me ot manage the server.**_

_**Attack 8 Cookie Manipulation**_

_**I created an account with a specific username that somehow triggered the system to give me an admin cookie. Total authentication bypass. To fix this you would have to use random session IDs that have nothing to do with usernames.**_

![](<../.gitbook/assets/Unknown image (23)>)

_**Creating account under this name issues admin ookie hihc let log as a administrator.**_ ![](<../.gitbook/assets/Unknown image (24)>)

![](<../.gitbook/assets/Unknown image (25)>)

_**AttacK 9: Information disclosure**_

[_**https://google-gruyere.appspot.com/595358199394003877103001395094130363587/dump.gtl**_](https://google-gruyere.appspot.com/595358199394003877103001395094130363587/dump.gtl)

_**I just typed /dump.gtl in the URL and the entire database dumped out for me to see. Complete data breach. Don't put database dumps or debug files anywhere near production.**_

![](<../.gitbook/assets/Unknown image (26)>)

_**Displayed the contents of the database via url.**_

_**Attack 10:Information 2**_

_**I uploaded a database dump file as a snippet, and then I could just access it through the web. The site becomes a tool for attackers to host and share stolen data. Only allow specific file types through a strict whitelist.**_

![](<../.gitbook/assets/Unknown image (27)>)

_**Uploaded the dump as a snippet so the attacker can simply upload their own copy of dump.gtl or a similar file and than access it**_
