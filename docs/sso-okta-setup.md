# Okta SSO Setup Guide for Meridian Voice Dashboard

This guide walks you through configuring Okta Single Sign-On (SSO) with SAML 2.0 for the Meridian Voice Dashboard.

## Prerequisites

Before you begin, ensure you have:

- **Okta admin access** - You need administrator privileges in your Okta organization to create and configure applications
- **Meridian Enterprise tier** - SSO is only available on the Enterprise tier subscription

## Step 1: Create a SAML 2.0 App in Okta

1. Log in to your Okta Admin Console
2. Navigate to **Applications** > **Applications** in the left sidebar
3. Click **Create App Integration**
4. Select **SAML 2.0** as the sign-in method
5. Click **Next**

### General Settings

- **App name**: Meridian Voice Dashboard
- **App logo** (optional): Upload your company logo or the Meridian logo
- Click **Next**

### Configure SAML

Enter the following SAML settings:

- **Single sign-on URL** (ACS URL): `https://auth.meridianvoice.com/saml/callback`
  - Check "Use this for Recipient URL and Destination URL"
- **Audience URI (SP Entity ID)**: `https://auth.meridianvoice.com/saml`
- **Name ID format**: EmailAddress
- **Application username**: Email

### Attribute Statements

Configure the following attribute mappings:

| Name | Name format | Value |
|------|-------------|-------|
| email | Unspecified | user.email |
| firstName | Unspecified | user.firstName |
| lastName | Unspecified | user.lastName |
| role | Unspecified | user.role |

**Note**: The `role` attribute should map to an Okta user attribute that contains one of: `admin`, `manager`, `agent`, or `viewer`

### Feedback

- Select **I'm an Okta customer adding an internal app**
- Click **Finish**

## Step 2: Get IdP Metadata URL

1. After creating the app, go to the **Sign On** tab
2. In the **SAML Setup** section, right-click on **Identity Provider metadata** link
3. Copy the metadata URL (it will look like: `https://your-org.okta.com/app/your-app-id/sso/saml/metadata`)
4. Save this URL - you'll need it in the next step

## Step 3: Assign Users to the Application

1. Go to the **Assignments** tab in your Okta app
2. Click **Assign** > **Assign to People** or **Assign to Groups**
3. Assign the users or groups who should have access to Meridian Voice Dashboard
4. Click **Done**

## Step 4: Enable SSO in Meridian Dashboard

1. Log in to Meridian Voice Dashboard as an administrator
2. Navigate to **Settings** > **SSO**
3. Select **SAML 2.0** as the SSO protocol
4. Paste the **IdP metadata URL** from Step 2 into the metadata URL field
5. Configure the following settings:
   - **Default Role**: Choose the default role for new SSO users (typically `viewer`)
   - **Allowed Domains**: Enter your company email domains (e.g., `yourcompany.com`)
   - **Auto-provision users**: Enable this to automatically create user accounts on first SSO login
6. Click **Save Configuration**

## Step 5: Test SSO

**Important**: Always test with a non-admin user first to avoid locking yourself out.

1. Open an incognito/private browser window
2. Navigate to your Meridian Voice Dashboard URL
3. Click **Sign in with SSO**
4. Enter your company email domain
5. You should be redirected to Okta for authentication
6. After successful authentication, you should be redirected back to Meridian Dashboard

### Troubleshooting

If SSO login fails:

- **Invalid SAML Response**: Verify the Entity ID and ACS URL are correct in both Okta and Meridian
- **User not provisioned**: Ensure the user is assigned to the Okta application
- **Attribute mapping errors**: Check that all required attributes (email, firstName, lastName) are being sent from Okta
- **Domain not allowed**: Verify the user's email domain is in the Allowed Domains list in Meridian

## Step 6: Enable SSO Enforcement (Optional)

Once SSO is tested and working:

1. In Meridian Dashboard, go to **Settings** > **SSO**
2. Enable **Require SSO for all users**
3. This will disable password-based login for all users in allowed domains

**Warning**: Keep at least one admin account with a different email domain that can still use password login as a backup.

## Additional Resources

- For OIDC setup instead of SAML, see the [OIDC configuration guide](./sso-oidc-setup.md)
- For advanced attribute mapping and role customization, contact Meridian support
- For multi-tenant SSO setup, see the Enterprise documentation

## Support

If you encounter issues during setup, contact Meridian support at support@meridianvoice.com with:
- Your Okta organization ID
- The IdP metadata URL
- Screenshots of any error messages
- SAML trace logs (if available)
