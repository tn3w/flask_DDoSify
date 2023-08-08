<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://github.com/tn3w/flask_DDoSify/releases/download/v0.3/blocked-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="https://github.com/tn3w/flask_DDoSify/releases/download/v0.3/blocked-light.png">
  <img alt="Picture from Block Page" src="https://github.com/tn3w/flask_DDoSify/releases/download/v0.3/blocked-dark.png">
</picture>
  
# flask_DDoSify
A DDoS defense system for flask applications, first sends users to a captcha page without a javascript script and creates a confirmation cookie/url arg after the captcha.

> [!WARNING]
> The syntax was changed with the latest version [0.7](https://github.com/tn3w/flask_DDoSify/releases/tag/v0.7). See [Personalization](#personalization) and [Changelog](https://github.com/tn3w/flask_DDoSify/compare/v0.6...v0.7)

## How does flask_DDoSify work?
Downloads public IP block lists[^1] and compares this data with the user, for more security the API of [Stop Forum Spam](https://www.stopforumspam.com/) is also used. If needed, a captcha is displayed to the user (or the robot) based on the strength set.[^2] Javascript is not needed for this, as the content is rendered on the server.[^3]

An example script could look like this:[^4]
```python
from flask import Flask
from ddosify import DDoSify

app = Flask(__name__)
ddosify = DDoSify(app, default_hardness=2)

@app.route("/")
def index():
    return 'Hello Human!'

app.run(host = "localhost", port = 8080)
```

## Application purposes
A few application purposes:
  - Protect against DDoS attacks [^5]
  - Your website contains content that should not be read by a robot
  - A login website
  - A dark web page that simply needs to be secured a bit

### Why should you use DDoSify if you host a Flask server?
A quick and easy implementation allows even small websites or a small team of developers to quickly get robot protection. It also doesn't use third-party providers, which limits data collection from Google, Facebook and the creepy data brokers.[^6] Everything is open source, meaning you can rewrite the code yourself and perhaps make it more private.

# Instructions

## Installation guide
1. Make sure you have the latest version of Python and Pip installed, you also need git installed.
2. Clone the repository to your computer with `git clone https://github.com/tn3w/flask_DDoSify` or download the ZIP file and unzip it.
3. Make sure the folder that was created is called `flask_DDoSify`.
4. Now use the following command to install all required packages: `python -m pip install -r flask_DDoSify/requirements.txt`.
5. Now create your flask script in the folder where the subfolder `flask_DDoSify` was installed.
6. Make sure that after:
   ```python
   app = Flask(__name__)
   ```
   You add the line:
   ```python
    ddosify = DDoSify(app)
   ```
   And at the beginning of the file add the import:
   ```python
   from flask_DDoSify.ddosify import DDoSify
   ```
For more information, see the sample code above.

## Personalization

1. `actions` Arg

   To change the response in the case of certain routes / endpoints, you can use the actions parameter.
   
   Example of a website that allows all bots on the main page, enforces captchas on the login page, and blocks all robots on the registration page:
   ```python
   ddosify = DDoSify(app, actions={"/": "let", "/login": "fight", "/register": "block"})
   ```

   When using "*" before or after the urlpath / endpoint you can address multiple urls.

   Example of a website where all urls with /api/ are allowed through, all urls starting with "/dogs/" show everyone a captcha and all urls ending with "/cats/" block bots:
   ```python
   ddosify = DDoSify(app, actions={"*/api/*": "let", "/dogs/*": "fight", "*/cats/": "block"})
   ```
   
   All actions:
   | Name of action | Executing Action                                                     |
   | -------------- | -------------------------------------------------------------------- |
   | let            | Allows all traffic through, regardless of whether the IP is blocked. |
   | block          | Blocks all traffic if it is on a block list, without captcha.        |
   | fight          | Displays a captcha to all traffic, whether suspicious or not.        |
   | captcha        | Default value, shows only suspicious traffic captchas.               |
   <br>

2. `hardness` Arg
   
   To change the hardness of a captcha for specific routes or endpoints use hardness.

   Example of a website that sets the hardness of the main page to 1 (= easy), on the login page to 2 (= normal) and on the register page to 3 (= hard):
   ```python
   ddosify = DDoSify(app, hardness={"/": 1, "/login": 2, "/register": 3})
   ```

   When using "*" before or after the urlpath / endpoint you can address multiple urls, like actions.

   All hardness levels:
   | Hardness Level | Captcha modification                                                                                                               |
   | -------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
   | 1              | The captcha is easy. Only a text captcha with 6 - 8 characters is displayed                                                        |
   | 2              | The captcha is normal. Only a text captcha with 9 - 11 characters is displayed                                                     |
   | 3              | The hardness of the captcha is hard, a 9 - 14 number audio captcha is displayed in addition to the 10 - 12 character text captcha. |
   <br>

3. `template_dirs` Arg

   To change the template directory of a particular route use the template_dirs arg.

   Example of a website that has a specific template directory on /api/:
   ```python
   ddosify = DDoSify(app, template_dirs={"/api/*": "/path/to/special/template/dir"})
   ```

   In a template directory must look like this:
   ```
   templatedir\
              \captcha.html
              \block.html
              \changelanguage.html
   ```
   > FIXME: If template file does not exist 404 is displayed
   <br>

4. `default_action` Arg

   To specify the default action of all routes or endpoints use the default_action arg.

   Example of a very paranoid website that has set its action to "fight" for all routes:
   ```python
   ddosify = DDoSify(app, default_action="figth")
   ```
   <br>

5. `default_hardness` Arg

   To specify the default hardness of all routes or endpoints use the default_hardness arg.

   Example of a very paranoid website that has set its hardness to 3 (= hard) for all routes:
   ```python
   ddosify = DDoSify(app, default_hardness=3)
   ```
   <br>

6. `default_template_dir` Arg

   To specify the default template_dir of all routes or endpoints use the default_template_dir arg.

   Example of a web page with custom template_dir:
   ```python
   ddosify = DDoSify(app, default_template_dir="/path/to/my/custom/template/dir")
   ```
   <br>

7. `verificationage` Arg

   Indicates the time in seconds how long a solved captcha is valid (Default: 3600 = 1 hour)

   Website with 3 hours verificationage:
   ```python
   ddosify = DDoSify(app, verificationage=10800)
   ```
   <br>

8. `withoutcookies` Arg

   If True, no cookies are created, and verification is proven via URL args (Default: False)

   Website with withoutcookies enabled:
   ```python
   ddosify = DDoSify(app, withoutcookies=True)
   ```
   <br>

9. `block_crawler` Arg

   If True, crawlers like Googlebot, further are estimated via their user agent as suspicious and not the website, good for websites that should not be crawled (Default: False)

   Web page with block_crawler enabled:
   ```python
   ddosify = DDoSify(app, block_crawler=True)
   ```

<br>

[^1]: The block lists of [FireHol](https://firehol.org/), [Ipdeny](https://www.ipdeny.com), [Emerging Threats](https://rules.emergingthreats.net), [MyIp.ms](https://myip.ms/) and a list of [Tor exit nodes](https://www.torproject.org/) are used. These lists, the last excluded, only offer protection against data centres or known attackers.
[^2]: Text and, if the set strength is above 2, audio captchas can already be partially solved by robots, this is a solution for small websites or, e.g. dark web sites that cannot use third party solutions. However, it should still provide sufficient protection.
[^3]: For a captcha to work, however, the user's IP and user agent must normally be stored. The website may also collect data such as language to translate the website. Cookies can also be used, this is decided by the server administrator.
[^4]: Attention, the script may not see the correct IP when trying a request to localhost:8080, for testing you can use cloudflared tunnels for example.
[^5]: Only if you have a large server that is supposed to protect a small server from DDoS attacks.
[^6]: Only if you do not use other services such as Google Analytics/Meta Pixel on your website.
