# PCI Transcript Redaction Setup Guide for Meridian Voice

This guide covers how to enable and configure PCI DSS-compliant transcript redaction in Meridian Voice to automatically detect and redact sensitive payment information from call transcripts.

## Prerequisites

Before enabling PCI transcript redaction, ensure you have:

- **Meridian Enterprise tier subscription** - PCI redaction features are only available on Enterprise plans
- **PCI DSS compliance enabled** - Navigate to **Settings** > **Security** and ensure PCI DSS compliance mode is enabled for your account

## What Gets Redacted

Meridian Voice automatically detects and redacts the following sensitive payment information:

- **Credit card numbers** - Supports Visa, Mastercard, American Express, Discover, JCB, and Diners Club patterns
- **CVV codes** - 3 or 4 digit security codes
- **Expiration dates** - In formats like 12/25, 12/2025, etc.

## Redaction Modes

Choose between two redaction modes based on your compliance requirements:

### Mask Mode (Recommended)

Replaces sensitive data while preserving partial information for customer service purposes.

**Example:**
- Credit card: `****-****-****-1234`
- CVV: `***`
- Expiration: `**/**`

### Remove Mode

Completely removes sensitive data and replaces it with a redaction marker.

**Example:**
- Credit card: `[REDACTED]`
- CVV: `[REDACTED]`
- Expiration: `[REDACTED]`

## Where Redaction Applies

When enabled, PCI redaction is applied across:

- ✅ **Stored transcripts** - All transcript text stored in the database
- ✅ **Webhook payloads** - Real-time webhook events containing transcript data
- ✅ **Analytics exports** - CSV/JSON exports from the Dashboard
- ✅ **Call logs in Dashboard** - Transcript views in the web interface
- ✅ **API responses** - Any API endpoints returning transcript data

## Important: Audio Recording Considerations

**⚠️ Audio recordings are NOT automatically redacted.** The transcript redaction feature only applies to text transcripts.

To maintain PCI compliance with audio recordings, you have two options:

### Option 1: Disable Call Recording

Set `recordCalls: false` in your agent configuration to prevent audio recording entirely.

### Option 2: Pause Recording During Payment

Configure recording to automatically pause when payment information is being collected. This requires:

1. Enabling `pauseRecordingOnPayment` in your PCI redaction config
2. Setting up the `tool.invoked` webhook to detect when payment collection tools are used
3. The recording will automatically pause when payment tools are detected and resume after

## Enable PCI Redaction in Meridian Dashboard

1. Log in to Meridian Voice Dashboard as an administrator
2. Navigate to **Settings** > **Compliance**
3. Locate the **PCI Redaction** section
4. Toggle **Enable PCI Redaction** to ON
5. Select your preferred **Redaction Mode**:
   - **Mask** - Partial masking (recommended)
   - **Remove** - Complete removal
6. Enable **Pause Recording on Payment** if you want audio recordings to pause during payment collection
7. Click **Save Changes**

## Enable PCI Redaction via API

You can also enable and configure PCI redaction programmatically using the Meridian API:

```javascript
await client.agents.update(agentId, {
  pciRedaction: {
    enabled: true,
    mode: "mask", // or "remove"
    pauseRecordingOnPayment: true
  }
});
```

### API Configuration Options

```javascript
{
  pciRedaction: {
    enabled: true,              // Enable/disable redaction
    mode: "mask",               // "mask" or "remove"
    pauseRecordingOnPayment: true  // Pause audio recording during payment
  }
}
```

## Testing Redaction

To verify that PCI redaction is working correctly, use Meridian's test card numbers in sandbox mode:

### Test Credit Card Numbers

| Card Type | Number | Expected Result |
|-----------|--------|-----------------|
| Visa | 4532015112830366 | `****-****-****-0366` (mask mode) |
| Mastercard | 5425233430109903 | `****-****-****-9903` (mask mode) |
| Amex | 374245455400126 | `****-******-*0126` (mask mode) |
| Discover | 6011000991300009 | `****-****-****-0009` (mask mode) |

### Test Procedure

1. Create a test agent in sandbox mode with PCI redaction enabled
2. Make a test call to the agent
3. During the call, speak one of the test credit card numbers above
4. Also mention: "The CVV is 123 and the expiration date is 12/25"
5. End the call
6. View the transcript in the Dashboard or via API
7. Verify that:
   - Credit card numbers are redacted correctly
   - CVV is redacted (shows as `***` or `[REDACTED]`)
   - Expiration date is redacted (shows as `**/**` or `[REDACTED]`)
8. Check webhook payloads to ensure redaction is applied there as well

### Testing Audio Recording Pause

If you enabled `pauseRecordingOnPayment`:

1. Configure a payment collection tool/function for your agent
2. During a test call, trigger the payment tool
3. Verify that the recording status shows "paused" during payment collection
4. After payment collection completes, verify recording resumes
5. Check the audio file to confirm no audio was recorded during the payment collection phase

## Compliance Notes

### PCI DSS Requirements

PCI redaction helps you meet the following PCI DSS requirements:

- **Requirement 3.3** - Mask PAN (Primary Account Number) when displayed
- **Requirement 3.4** - Render PAN unreadable wherever it is stored
- **Requirement 3.2.1** - Do not store sensitive authentication data after authorization (CVV)

### Data Retention

- Redaction is applied in real-time as transcripts are generated
- Historical transcripts created before enabling redaction are NOT automatically redacted
- To redact historical data, contact Meridian support for a one-time migration

### Audit Logging

All changes to PCI redaction settings are logged in the audit log:

- Navigate to **Settings** > **Security** > **Audit Log**
- Look for events: `pci_redaction.enabled`, `pci_redaction.disabled`, `pci_redaction.mode_changed`

## Webhook Configuration

If you're using webhooks to receive transcript data, ensure your webhook handlers respect redacted data:

```javascript
// Example webhook handler
app.post('/webhooks/meridian', (req, res) => {
  const { event, data } = req.body;
  
  if (event === 'transcript.completed') {
    // Transcript data will already be redacted
    const transcript = data.transcript;
    console.log('Redacted transcript:', transcript);
    
    // Do NOT attempt to store or log unredacted versions
    // All PCI data has been redacted by Meridian
  }
  
  res.sendStatus(200);
});
```

## Troubleshooting

### Redaction Not Working

**Issue**: Credit card numbers still visible in transcripts

**Solutions**:
1. Verify PCI redaction is enabled in **Settings** > **Compliance** > **PCI Redaction**
2. Check that the credit card number matches a supported pattern (Visa, Mastercard, Amex, etc.)
3. Ensure you're viewing a transcript created AFTER enabling redaction
4. Clear your browser cache and reload the Dashboard

### Partial Redaction

**Issue**: Some digits are visible that shouldn't be

**Solutions**:
1. This may be expected behavior in "mask" mode, which preserves the last 4 digits
2. Switch to "remove" mode if you need complete redaction
3. Contact support if numbers other than the last 4 digits are visible

### Recording Still Contains Payment Info

**Issue**: Audio recordings contain payment information

**Solutions**:
1. Verify `pauseRecordingOnPayment` is enabled in your PCI redaction config
2. Ensure your payment collection tool is properly configured to trigger the recording pause
3. Check the `tool.invoked` webhook is being fired when payment tools are used
4. Consider disabling call recording entirely with `recordCalls: false`

### API Returns Unredacted Data

**Issue**: API responses contain unredacted payment information

**Solutions**:
1. Ensure you're using the latest version of the Meridian API client
2. Verify PCI redaction is enabled at the account level, not just per-agent
3. Check your API key has the correct permissions
4. Contact support if the issue persists

## Additional Resources

- [PCI DSS Compliance Overview](https://www.pcisecuritystandards.org/)
- [Meridian Voice Security Documentation](./security-overview.md)
- [Webhook Reference](./webhook-reference.md)
- [API Documentation](https://docs.meridianvoice.com/api)

## Support

For assistance with PCI redaction setup or compliance questions:

- **Email**: compliance@meridianvoice.com
- **Support Portal**: support.meridianvoice.com
- **Enterprise Support**: Available 24/7 for Enterprise customers

For PCI DSS compliance audits or certifications, request a copy of Meridian's SOC 2 Type II report and PCI DSS Attestation of Compliance (AOC) from your account manager.
