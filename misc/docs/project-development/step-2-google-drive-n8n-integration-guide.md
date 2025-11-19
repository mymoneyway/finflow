# Google Drive n8n Integration Guide (FinFlow-Compliant)

This guide shows how to connect **Google Drive** to **n8n** using **Google Cloud Console** with proper OAuth2 credentials (no hardcoded secrets, FinFlow-safe).

<!-- mdformat-toc start --slug=github --maxlevel=6 --minlevel=2 -->

- [Create a Google Cloud Project](#create-a-google-cloud-project)
- [Enable Google APIs](#enable-google-apis)
  - [Google Drive API](#google-drive-api)
  - [Google Sheets API](#google-sheets-api)
- [Configure OAuth overview](#configure-oauth-overview)
- [Create OAuth2 Credential for n8n](#create-oauth2-credential-for-n8n)
- [Add Tester User](#add-tester-user)
- [Create Credential in n8n](#create-credential-in-n8n)

<!-- mdformat-toc end -->

______________________________________________________________________

## Create a Google Cloud Project<a name="create-a-google-cloud-project"></a>

1. Open: https://console.cloud.google.com
1. Top-left -> **Project dropdown** -> **New Project**
1. Name it: **FinFlow**
1. Click **Create**

______________________________________________________________________

## Enable Google APIs<a name="enable-google-apis"></a>

### Google Drive API<a name="google-drive-api"></a>

1. Left menu -> **APIs & Services -> Library**
1. Search: **Google Drive API**
1. Click **Enable**

### Google Sheets API<a name="google-sheets-api"></a>

1. Left menu -> **APIs & Services -> Library**
1. Search: **Google Sheets API**
1. Click **Enable**

______________________________________________________________________

## Configure OAuth overview<a name="configure-oauth-overview"></a>

1. Left menu -> **APIs & Services -> OAuth consent screen**
1. User type: **External**
1. App name: **FinFlow ETL**
1. App email: **[email]**
1. Save

______________________________________________________________________

## Create OAuth2 Credential for n8n<a name="create-oauth2-credential-for-n8n"></a>

1. Left menu -> **APIs & Services -> Credentials**
1. Click **Create Credentials -> OAuth Client ID**
1. Application type: **Web application**
1. Name: `n8n-finflow-drive-localhost-oauth`
1. Add **Authorized redirect URI**: `http://localhost:5678/rest/oauth2-credential/callback`
1. Save -> Copy **Client ID** + **Client Secret**

______________________________________________________________________

## Add Tester User<a name="add-tester-user"></a>

Google blocks OAuth in Testing mode unless you add your Gmail account.

1. Open Google Cloud Console -> *APIs & Services -> OAuth consent screen*.
1. Scroll to **Test users** -> click **Add users**.
1. Add your login: `[email]` -> Save.
1. Retry the OAuth connection in n8n.

______________________________________________________________________

## Create Credential in n8n<a name="create-credential-in-n8n"></a>

1. Open **n8n**
1. Left panel -> **Credentials**
1. Click **New Credential** -> search **Google Drive OAuth2**
1. Fill in:
   - **Client ID** -> paste
   - **Client Secret** -> paste
   - **Callback URL** must match this exactly: `http://localhost:5678/rest/oauth2-credential/callback`
1. Click **Connect**
1. Google login window appears -> choose your Drive account
1. Approve access
