# TikTok OAuth Redirect Page

This repository contains a **public-safe HTML page** for handling TikTok OAuth authorization. It is designed to:

- Capture the `auth_code` and `state` parameters from TikTok after user authorization.
- Forward the `auth_code` securely to a **Power Automate flow**.
- Allow automated access token exchange for TikTok Ads and Campaign Insights.
- Be safe to host publicly (e.g., GitHub Pages) without exposing secrets.

---

## Features

- **Public-Safe**: No secrets or app credentials are included in the HTML.
- **State Verification**: Captures the `state` parameter sent in the authorization URL.
- **Power Automate Integration**: Sends `auth_code` and `state` to your flow for further processing.
- **Fully Automated**: Once the user clicks the TikTok auth link, the process continues without manual intervention.

---

## How to Use

1. **Clone or download** this repository.  
2. Replace the placeholder `YOUR_FLOW_HTTP_TRIGGER_URL` in `tiktok_redirect.html` with your **Power Automate HTTP Request URL**.
3. Push the file to your GitHub repository.  
4. Enable **GitHub Pages** in repository settings → select branch (`main`) and folder (`/root`).  
5. Copy the **GitHub Pages URL**, for example: https://<your-github-username>.github.io/tiktok-oauth-redirect/tiktok_redirect.html
6. Set this URL as the **Redirect URI** in your TikTok Developer App.  
7. Generate TikTok authorization URL and send it to your user:
https://business-api.tiktok.com/portal/auth
?
app_id=YOUR_APP_ID
&redirect_uri=https://<your-github-username>.github.io/tiktok-oauth-redirect/tiktok_redirect.html
&state=RANDOM_STRING
&scope=ads.management,ads.insights
8. User clicks link → TikTok login → redirects to GitHub page → page sends auth code to Power Automate → flow exchanges auth code for access token.  

---

## Security Notes

- **No secrets in HTML**: Safe to host publicly.  
- **HTTPS required**: GitHub Pages provides HTTPS by default.  
- **State parameter**: Ensure your flow validates the `state` parameter to prevent CSRF attacks.  
- **Access token handling**: All sensitive operations, including exchanging the auth code for an access token, happen inside your Power Automate flow or backend.  

---

## License

This project is licensed under the MIT License.  

---

## Example Flow Diagram
Email → TikTok Auth URL → TikTok Login → Redirect Page → Power Automate → Exchange Auth Code → Store Access Token → Fetch Ads Insights
