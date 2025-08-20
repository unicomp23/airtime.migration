# Third-Party Integration Requirements

## Overview
This document identifies all third-party services that need to be updated during the airtime.com to cantina.com migration.

---

## 🔗 Third-Party Services Inventory

### Critical External Services

#### 1. **Payment Processors**
- **Stripe**
  - Webhook endpoints: `https://api.airtime.com/webhooks/stripe`
  - Return URLs: `https://platform.airtime.com/payment/complete`
  - Action: Update in Stripe Dashboard
  
- **PayPal**
  - IPN URL: `https://api.airtime.com/webhooks/paypal`
  - Return URL: `https://platform.airtime.com/payment/success`
  - Cancel URL: `https://platform.airtime.com/payment/cancel`
  - Action: Update in PayPal Developer Portal

#### 2. **Authentication Providers**
- **OAuth2 Providers**
  - Google: Redirect URIs in Google Cloud Console
  - Facebook: Valid OAuth Redirect URIs
  - Apple: Return URLs in Apple Developer Portal
  - GitHub: Authorization callback URL
  
- **SAML Providers**
  - Assertion Consumer Service URL
  - Entity ID updates
  - Metadata URL changes

#### 3. **Communication Services**
- **Twilio**
  - Webhook URLs for SMS callbacks
  - Voice callback URLs
  - Status callback endpoints
  - Action: Update via Twilio Console or API

- **SendGrid**
  - Event webhook: `https://api.airtime.com/webhooks/sendgrid`
  - Unsubscribe links in templates
  - Click tracking domain
  - Action: Update in SendGrid Settings

#### 4. **CDN & Media Services**
- **CloudFront**
  - Origin domains
  - CNAME aliases
  - SSL certificates
  
- **Cloudflare**
  - DNS records
  - Page rules
  - Worker scripts
  - SSL certificates

- **S3 Buckets**
  - CORS policies allowing airtime.com
  - Bucket policies with domain restrictions
  - CloudFront OAI permissions

#### 5. **Monitoring & Analytics**
- **Datadog**
  - APM service names
  - Log pipeline filters
  - Dashboard URLs
  - Alert notification channels

- **Sumo Logic**
  - Collectors configuration
  - Source categories
  - Dashboard queries
  - Alert webhooks

- **Google Analytics**
  - Property settings
  - View filters
  - Goals with destination URLs
  - E-commerce tracking

- **Segment**
  - Source configurations
  - Destination webhooks
  - Tracking plan URLs

#### 6. **Mobile Push Notifications**
- **Firebase Cloud Messaging**
  - Server key restrictions
  - iOS app bundle IDs
  - Android package names
  - Dynamic links domain

- **Apple Push Notification Service**
  - Certificate common names
  - Push endpoints

#### 7. **Customer Support**
- **Zendesk**
  - Email forwarding addresses
  - Web widget allowed domains
  - API token restrictions
  - Webhook endpoints

- **Intercom**
  - App ID configuration
  - Webhook URLs
  - Email templates
  - Custom domain settings

#### 8. **Development Tools**
- **GitHub**
  - Webhook URLs for CI/CD
  - OAuth App settings
  - GitHub Pages custom domain
  - Status checks URLs

- **GitLab**
  - CI/CD webhook URLs
  - Integration settings
  - Mirror repository URLs

- **Jira**
  - Webhook URLs
  - Application links
  - OAuth consumers

- **Slack**
  - Webhook URLs
  - OAuth redirect URLs
  - Event subscriptions URL
  - Slash commands

#### 9. **Security Services**
- **SSL Certificate Authorities**
  - Let's Encrypt ACME challenges
  - DigiCert domain validation
  - Certificate renewal notifications

- **Web Application Firewall**
  - Cloudflare WAF rules
  - AWS WAF rules
  - Domain whitelist rules

---

## 📋 Integration Update Checklist

### Payment Processors
```bash
# Stripe webhook update via CLI
stripe webhooks create \
  --url https://api.cantina.com/webhooks/stripe \
  --enabled-events payment_intent.succeeded,payment_intent.failed

# List current webhooks to verify
stripe webhooks list
```

### OAuth Providers
```javascript
// Google OAuth update example
{
  "redirect_uris": [
    "https://platform.cantina.com/auth/google/callback",
    "https://platform.airtime.com/auth/google/callback" // Keep during transition
  ]
}
```

### Communication Services
```python
# Twilio webhook update via API
from twilio.rest import Client

client = Client(account_sid, auth_token)

# Update SMS webhook
messaging_service = client.messaging.services('MGXXXXXXXX')
messaging_service.update(
    inbound_request_url='https://api.cantina.com/webhooks/twilio/sms',
    fallback_url='https://api.cantina.com/webhooks/twilio/fallback'
)

# Update voice webhook
phone_number = client.incoming_phone_numbers('PNXXXXXXXX')
phone_number.update(
    voice_url='https://api.cantina.com/webhooks/twilio/voice',
    sms_url='https://api.cantina.com/webhooks/twilio/sms'
)
```

### Monitoring Services
```yaml
# Datadog agent configuration update
init_config:

instances:
  - url: https://platform.cantina.com
    name: platform_cantina
    tags:
      - env:production
      - domain:cantina.com
```

---

## 🔄 Migration Strategy

### Phase 1: Inventory & Access (Day 1-2)
1. List all third-party services
2. Gather admin credentials
3. Document current configurations
4. Identify services with API access vs manual updates

### Phase 2: Parallel Configuration (Day 3-4)
1. Add cantina.com URLs alongside airtime.com
2. Do NOT remove airtime.com yet
3. Test new endpoints
4. Verify dual configuration works

### Phase 3: Testing (Day 5)
1. Send test webhooks to new endpoints
2. Verify OAuth flows with new domains
3. Test payment callbacks
4. Check monitoring data flow

### Phase 4: Cutover (Day 6)
1. Update DNS to point to cantina.com
2. Monitor webhook deliveries
3. Check error rates
4. Verify all integrations working

### Phase 5: Cleanup (Day 7+)
1. Remove airtime.com URLs after 30 days
2. Update documentation
3. Archive old configurations

---

## 🚨 High-Risk Integrations

### Services Requiring Downtime
1. **Payment processors** - May need momentary pause
2. **OAuth providers** - Session invalidation possible
3. **SAML SSO** - Metadata update may log users out

### Services with Long Propagation
1. **Apple Push Notifications** - Certificate updates take time
2. **Google OAuth** - Can take hours to propagate
3. **DNS-based services** - TTL dependent

### Services Requiring Customer Action
1. **Webhook consumers** - Customers using our webhooks
2. **API integrations** - Custom integrations by partners
3. **Mobile apps** - May need app update

---

## 📊 Update Priority Matrix

| Service | Priority | Update Method | Rollback Time |
|---------|----------|--------------|---------------|
| Payment Processors | CRITICAL | Manual/API | Immediate |
| OAuth Providers | CRITICAL | Manual | 1-4 hours |
| Webhooks (Twilio, etc) | HIGH | API | Immediate |
| CDN/CloudFront | HIGH | Console/API | 5-10 mins |
| Monitoring | MEDIUM | Config files | Immediate |
| Analytics | LOW | Manual | N/A |
| Support Tools | LOW | Manual | Immediate |

---

## 🔐 Security Considerations

### API Key Restrictions
Update domain restrictions for:
- Google Maps API
- AWS API keys
- Third-party API keys

### CORS Policies
Services that need CORS updates:
- S3 buckets
- API gateways
- CDN configurations

### Certificate Pinning
Check for:
- Mobile apps with pinned certificates
- IoT devices
- Desktop applications

---

## 📝 Notification Templates

### Customer Notification
```
Subject: Important: Webhook URL Update Required

Dear [Customer Name],

As part of our infrastructure upgrade, we're migrating from airtime.com to cantina.com.

Action Required:
Please update your webhook URLs from:
https://api.airtime.com/webhooks/[your-endpoint]
To:
https://api.cantina.com/webhooks/[your-endpoint]

Timeline:
- Now: Both URLs are active
- [Date + 30 days]: airtime.com URLs will be deprecated
- [Date + 60 days]: airtime.com URLs will be disabled

Please update at your earliest convenience.
```

### Partner Notification
```
Subject: API Endpoint Migration Notice

We're updating our API endpoints from airtime.com to cantina.com.

Current endpoint: https://api.airtime.com/v1/
New endpoint: https://api.cantina.com/v1/

Both endpoints will work in parallel for 60 days.
Please update your integrations by [date].
```

---

## 🛠️ Verification Scripts

### Webhook Tester
```python
#!/usr/bin/env python3
import requests
import json

def test_webhook(old_url, new_url, payload):
    """Test webhook delivery to both URLs"""
    
    # Test old URL
    old_response = requests.post(old_url, json=payload)
    print(f"Old URL ({old_url}): {old_response.status_code}")
    
    # Test new URL
    new_response = requests.post(new_url, json=payload)
    print(f"New URL ({new_url}): {new_response.status_code}")
    
    return old_response.status_code == new_response.status_code

# Test each webhook
webhooks = [
    ("https://api.airtime.com/webhooks/stripe", 
     "https://api.cantina.com/webhooks/stripe"),
    ("https://api.airtime.com/webhooks/twilio",
     "https://api.cantina.com/webhooks/twilio"),
]

test_payload = {"test": True, "timestamp": "2025-08-20"}

for old, new in webhooks:
    test_webhook(old, new, test_payload)
```

### OAuth Flow Tester
```javascript
// Test OAuth redirect flow
const testOAuthFlow = async (provider) => {
  const providers = {
    google: {
      old: 'https://platform.airtime.com/auth/google',
      new: 'https://platform.cantina.com/auth/google'
    },
    facebook: {
      old: 'https://platform.airtime.com/auth/facebook',
      new: 'https://platform.cantina.com/auth/facebook'
    }
  };
  
  // Initiate OAuth flow with new URL
  window.location.href = providers[provider].new;
};
```

---

## 📋 Post-Migration Verification

### Checklist
- [ ] All payment webhooks receiving events
- [ ] OAuth logins working for all providers
- [ ] Email delivery functioning
- [ ] SMS delivery functioning
- [ ] Push notifications delivering
- [ ] Monitoring data flowing
- [ ] Analytics tracking working
- [ ] Support tickets creating
- [ ] API rate limits applied
- [ ] SSL certificates valid

### Monitoring Dashboard
Create dashboard to track:
- Webhook delivery success rate
- OAuth login success rate
- API call patterns
- Error rates by integration
- Third-party service health

---

## 🚨 Emergency Contacts

### Service Contacts
- **Stripe Support**: [+1-888-888-8888]
- **Twilio Support**: [support ticket URL]
- **AWS Support**: [Console URL]
- **Cloudflare Support**: [Dashboard URL]

### Internal Contacts
- **Integration Team Lead**: [Contact]
- **Security Team**: [Contact]
- **Customer Success**: [Contact]

---

## 📝 Notes

1. **Keep both domains active** during transition period
2. **Test thoroughly** before removing old URLs
3. **Monitor continuously** during and after migration
4. **Document everything** for future reference
5. **Communicate proactively** with customers and partners

**Total Third-Party Services**: ~30-40  
**Estimated Update Time**: 2-3 days  
**Risk Level**: HIGH (external dependencies)