# Meridian Voice Dashboard

Welcome to the Meridian Voice Dashboard repository.

## Features

- Real-time voice analytics and monitoring
- Call management and routing
- Team collaboration tools
- Comprehensive reporting and insights
- PCI DSS-compliant transcript redaction

## PCI Compliance and Transcript Redaction

Meridian Voice Dashboard includes built-in PCI DSS-compliant transcript redaction to automatically detect and redact sensitive payment information from call transcripts.

### Supported Payment Data Redaction

- **Credit card numbers** (Visa, Mastercard, Amex, Discover, JCB, Diners Club)
- **CVV codes** (3 or 4 digits)
- **Expiration dates**

### Redaction Features

- **Two redaction modes**: Mask (partial) or Remove (complete)
- **Applies to**: Transcripts, webhooks, exports, and Dashboard views
- **Audio recording controls**: Optional pause during payment collection
- **Real-time redaction**: Applied as transcripts are generated
- **Enterprise tier feature**: Available on Enterprise plans only

### Setup Guide

For detailed setup instructions, see:

- **[PCI Redaction Setup Guide](docs/pci-redaction-setup.md)** - Complete guide for enabling and configuring PCI transcript redaction
- **Example Configuration** - See `config/pci-redaction.example.json` for a sample PCI redaction configuration

### Quick Configuration

Enable PCI redaction via API:

```javascript
await client.agents.update(agentId, {
  pciRedaction: {
    enabled: true,
    mode: "mask",
    pauseRecordingOnPayment: true
  }
});
```

Or via Dashboard: **Settings** > **Compliance** > **PCI Redaction**

## SSO Configuration

Meridian Voice Dashboard supports Single Sign-On (SSO) for Enterprise tier customers. We support both **SAML 2.0** and **OIDC** protocols.

### Supported SSO Providers

- Okta
- Azure AD
- Google Workspace
- OneLogin
- Auth0
- Custom SAML 2.0 providers

### Setup Guides

For detailed setup instructions, see:

- **[Okta SSO Setup Guide](docs/sso-okta-setup.md)** - Step-by-step guide for configuring Okta with SAML 2.0
- **Example Configuration** - See `config/sso-config.example.json` for a sample SSO configuration

### SSO Requirements

- **Meridian Enterprise tier subscription** - SSO is only available on Enterprise plans
- **Admin access** to your identity provider
- **Email domain verification** - Required for domain-based SSO enforcement

### Configuration

SSO can be configured through:

1. **Meridian Dashboard UI** - Navigate to Settings > SSO > SAML 2.0 or OIDC
2. **Configuration file** - Use `config/sso-config.example.json` as a template

For questions or support with SSO setup, contact support@meridianvoice.com.

## Getting Started

(Add your getting started instructions here)

## Documentation

- [PCI Redaction Setup Guide](docs/pci-redaction-setup.md)
- [SSO Setup Guide](docs/sso-okta-setup.md)
- API Documentation (coming soon)
- User Guide (coming soon)

## Support

For technical support, contact support@meridianvoice.com.

For compliance-related questions, contact compliance@meridianvoice.com.

## License

(Add your license information here)
