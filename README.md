# TikTok OAuth Redirect Page

This repository contains a **public-safe HTML page** for handling both TikTok Business OAuth and TikTok Shop OAuth using the same redirect URL.

The page detects which authorization flow was used and sends a simple payload to the same **Power Automate flow**.

---

## Supported TikTok OAuth Flows

### TikTok Business / Ads

Example callback:

```text
https://awi-systems.github.io/tiktok-oauth-redirect/tiktok_redirect.html?auth_code=example&code=example&state=example
```

The redirect page identifies this as:

```text
type = business
```

### TikTok Shop

Example callback:

```text
https://awi-systems.github.io/tiktok-oauth-redirect/tiktok_redirect.html?app_key=6la75j8ggs28c&code=example&locale=en&shop_region=PH&state=example
```

The redirect page identifies this as:

```text
type = shop
```

The detection is based on the presence of the `app_key` parameter in the TikTok Shop callback.

---

## Features

- **Single Redirect URL** for TikTok Business and TikTok Shop.
- **Automatic Authorization Type Detection**.
- **Power Automate Integration** using the same HTTP trigger.
- Captures the authorization `code`.
- Captures the `state` parameter.
- No TikTok access tokens, refresh tokens, or app secrets are included in the HTML.
- Simple request body for Power Automate.

---

## Power Automate Request Body

The redirect page sends only three values:

```json
{
  "type": "shop",
  "code": "AUTHORIZATION_CODE",
  "state": "STATE_VALUE"
}
```

For TikTok Business:

```json
{
  "type": "business",
  "code": "AUTHORIZATION_CODE",
  "state": "STATE_VALUE"
}
```

For TikTok Shop:

```json
{
  "type": "shop",
  "code": "AUTHORIZATION_CODE",
  "state": "STATE_VALUE"
}
```

---

## How Authorization Type Is Detected

The redirect page checks for the TikTok Shop `app_key` parameter:

```javascript
const appKey = params.get("app_key");
const type = appKey ? "shop" : "business";
```

Therefore:

### Business callback

```text
?auth_code=example&code=example&state=example
```

Results in:

```text
type = business
```

### Shop callback

```text
?app_key=6la75j8ggs28c&code=example&locale=en&shop_region=PH&state=example
```

Results in:

```text
type = shop
```

The `locale` and `shop_region` values are not sent to Power Automate because the current requirement is to send only `type`, `code`, and `state`.

---

## Authorization Code Handling

The page supports both callback formats by checking:

```javascript
const authCode = params.get("auth_code") || params.get("code");
```

This keeps the existing TikTok Business behavior while also supporting the TikTok Shop callback.

The value is then sent to Power Automate as:

```json
"code": "AUTHORIZATION_CODE"
```

---

## How to Use

1. Push `tiktok_redirect.html` to the existing GitHub repository.
2. Keep the same GitHub Pages URL:

```text
https://awi-systems.github.io/tiktok-oauth-redirect/tiktok_redirect.html
```

3. Use the same redirect URL for the TikTok Business and TikTok Shop authorization flows where the respective TikTok configuration accepts the shared redirect URL.
4. In Power Automate, configure the HTTP trigger to accept:

```json
{
  "type": "string",
  "code": "string",
  "state": "string"
}
```

5. Add a Condition in Power Automate using `type`:

```text
type is equal to shop
```

6. If **Yes**, continue with TikTok Shop processing.
7. If **No**, continue with TikTok Business / Ads processing.

---

## Recommended Power Automate Flow

```text
TikTok Authorization
        |
        +-----------------------------+
        |                             |
 TikTok Business                 TikTok Shop
        |                             |
        +-------------+---------------+
                      |
                      v
             GitHub Redirect Page
                      |
                      v
               Power Automate
                      |
                      v
                 Check type
                  /       \
               shop      business
                |            |
                v            v
          Shop Processing  Business Processing
```

---

## Power Automate Condition

Use the `type` property received from the redirect page:

```text
type is equal to shop
```

### If Yes

Process the authorization as **TikTok Shop**.

### If No

Process the authorization as **TikTok Business / Ads**.

This keeps both integrations in one Power Automate trigger while allowing each authorization flow to have its own processing logic.

---

## Security Notes

- **No secrets in HTML**: The page does not contain TikTok app secrets, access tokens, or refresh tokens.
- **HTTPS required**: GitHub Pages provides HTTPS by default.
- **State validation**: Validate the `state` value inside Power Automate/backend when it is used to protect the authorization flow.
- **Access token handling**: Token exchange and token storage should happen inside Power Automate/backend, not in the public HTML page.
- **Power Automate URL**: The HTTP trigger URL is embedded in the public HTML. Treat it as a sensitive endpoint and rotate/revoke it if it becomes exposed or compromised.

---

## Example Flow

```text
Email / System
      |
      v
TikTok Authorization URL
      |
      v
TikTok Login / Authorization
      |
      v
GitHub Redirect Page
      |
      +---- app_key exists? ----+
      |                         |
     YES                       NO
      |                         |
 type = shop              type = business
      |                         |
      +-----------+-------------+
                  |
                  v
          Power Automate
                  |
                  v
             Process OAuth
                  |
                  v
        Exchange Code / Token
                  |
                  v
          Continue API Access
```

---

## License

This project is licensed under the MIT License.
