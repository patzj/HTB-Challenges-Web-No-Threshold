# HTB-Challenges-Web-No-Threshold
Prepare for the finest magic products out there. However, please be aware that we've implemented a specialized protective spell within our web application to guard against any black magic aimed at our web shop.🔮🎩

![Untitled](https://github.com/patzj/HTB-Challenges-Web-No-Threshold/assets/10325457/b9640524-f641-4c1b-bd48-8bff0fc8a127)

## Reconnaissance

The application has two easily identifiable features: *Add to Cart* and *Login*. The former necessitates user authentication, whereas the latter is currently inaccessible. Based on this, it can be assumed that the challenge revolves around finding a way to bypass the authentication process.

![Untitled (1)](https://github.com/patzj/HTB-Challenges-Web-No-Threshold/assets/10325457/2cb308aa-3ee8-475d-93ca-2eb80bab7262)

The HTTP response also lacks sufficient information to pinpoint the cause of this issue.

## Scanning

This type of protection usually involves pattern matching of URL paths. Let's attempt to URL-encode the login path by encoding one letter from `/auth/login`, transforming it into `/auth/logi%6e`.

![Untitled (2)](https://github.com/patzj/HTB-Challenges-Web-No-Threshold/assets/10325457/905419bc-0ca2-4b0f-8609-240ae644c9d6)

Now, let's test if the login form is vulnerable to SQL injection. Initially, we'll try using `' OR '1'='1` in both the username and password fields to determine if it successfully goes through. Additionally, we should modify the form action to `/auth/logi%6e` using the browser's developer tools to prevent our POST request from being blocked.

![Untitled (3)](https://github.com/patzj/HTB-Challenges-Web-No-Threshold/assets/10325457/7b35d094-17e8-447c-aed4-45ed1cb15157)

Alright, we've successfully bypassed the login form, and now we're facing the 2FA code verification.

![Untitled (4)](https://github.com/patzj/HTB-Challenges-Web-No-Threshold/assets/10325457/270355a8-9f8a-4b70-8064-a6b657db0e41)

Upon closer examination, we observe that the 2FA code consists of 4 characters. However, it remains unclear whether it is alphanumeric or solely numeric.

## Exploitation 1 - Brute-Force 2FA Code via Burp Intruder *(Failed)*

At this point, we'll make the assumption that the 2FA codes are exclusively numeric. Our approach is to employ a brute-force method, utilizing a pre-defined list of 4-digit codes from _SecList_.

However, after a number of attempts, it becomes evident that there is some form of rate-limiting mechanism in place.

![Untitled (5)](https://github.com/patzj/HTB-Challenges-Web-No-Threshold/assets/10325457/4ba64de4-dd10-4bcf-9dbc-e43376f1c466)

## Exploitation 2 - Brute-Force 2FA via Custom Script *(Success)*

We observe that the application has not yet issued a cookie. Therefore, we can infer that the number of attempts is being tracked based on the IP address. To circumvent this, we must deceive the server into believing that each brute-force attempt originates from a distinct IP address. This can be achieved by setting the `X-Forwarded-For` header.

We need to write a script that will enable us to log in using SQL injection, send 2FA verification requests that appear to originate from different IP addresses, and print the contents of page upon successful verification.

Upon running the custom script, it is apparent that it has successfully circumvented all implemented security measures. The flag has been captured successfully!

![Untitled (6)](https://github.com/patzj/HTB-Challenges-Web-No-Threshold/assets/10325457/fca1e943-132c-401b-94f5-ffea240d0b63)

_Note: The script is available in the references._

## Post-Exploitation

The security of the login page could have been enhanced by validating against multiple encoded patterns of the path. The application should utilize parameterized query statements to prevent SQL injection vulnerabilities. Additionally, for a more robust security posture, the 2FA code verification scheme should be strengthened—consider making the 2FA code more complex and implementing measures such as invalidating the code after a few unsuccessful attempts.

## References
- https://portswigger.net/burp/documentation/desktop/tools/intruder
- https://github.com/danielmiessler/SecLists
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For
- https://gist.github.com/patzj/f9554934f1ac7680aad5f18cd4bb6d94
