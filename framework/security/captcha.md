---
description: "Use Authencode image challenges and short-lived confirmation tokens to understand two-phase verification, caching and replay boundaries."
icon: fingerprint
---

# CAPTCHAs and Confirmation Tokens

Authencode uses image challenges to reduce automation abuse. It verifies the answer and then issues a short-lived confirmation token for consumption by subsequent business processes. Therefore, displaying pictures, submitting answers and confirming consumption are three steps, and the challenge token should not be directly regarded as a verified result.

## Prerequisites

Deploy Zongsoft.Security.Captcha and the main Security plugin. HTTP scenarios also require Security.Web. The host should provide the framework's distributed cache contract; multiple instances must use a shared cache, otherwise different nodes may not find the same challenge.

{% code title="Captcha.deploy (append fragment)" %}
```ini
[plugins zongsoft security captcha]
nuget:Zongsoft.Security.Captcha
```
{% endcode %}

## Client Process

1. Request the Authencode scheme through the deployed verification code endpoint.
2. Display the returned PNG and retain the challenge token from the `X-Captcha` response header.
3. Submit the user answer and challenge token composed of `token:code`, and the provider also accepts `token=code`.
4. Save the confirmation token returned after successful verification and hand it to the business process that requires the verification code.
5. The business finally verifies the confirmation token, consumes the corresponding confirmation cache item and deletes the original challenge.

When using the browser across domains, you must also confirm that the response header can be read by the front end; the reverse proxy cannot discard `X-Captcha`. Check target environment font and image dependencies when image rendering fails.

## Validity Period and One-time Semantics

The currently implemented challenge is valid for 10 minutes and the confirmation token is valid for 5 minutes. The final verification is to consume the confirmation item through cached removal and value operations, and the same confirmation token cannot be used successfully again.

{% hint style="warning" %}
🚨 Successful answer verification itself will not delete the challenge immediately. The same challenge can generate multiple confirmation tokens before final consumption, so the entire challenge process cannot be described as strictly single use. The business also needs its own request rate limiting and operation idempotent rules.
{% endhint %}

## What to Verify When Integrating

Test wrong answers, challenge expiration, confirmation expiration, duplicate confirmation and cross-node confirmation respectively. It only calls the image generator to get a PNG, and does not verify the cache, token and business confirmation processes.

The verification code is used to prevent abuse and does not prove the user's identity. Provide an alternative process for users who are unable to complete the picture challenge, and do not log answers or tokens.

Implementation basis: [AuthencodeCaptcha](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/captcha/AuthencodeCaptcha.cs), [Captcha controller](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/api/Controllers/CaptchaController.cs).
