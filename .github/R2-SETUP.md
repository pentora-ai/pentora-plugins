# Cloudflare R2 Setup Guide

This document explains how to configure Cloudflare R2 for hosting Pentora plugins.

## Prerequisites

- Cloudflare account with R2 enabled
- Domain managed by Cloudflare (pentora.ai)
- GitHub repository admin access

## Step 1: Create R2 Bucket

1. Go to Cloudflare Dashboard → R2
2. Click "Create bucket"
3. Bucket name: `pentora-plugins`
4. Location: Automatic (closest to users)
5. Enable Public Access (required for plugin downloads)

## Step 2: Configure Custom Domain

1. In R2 bucket settings, go to "Settings" tab
2. Under "Public Access", click "Connect Domain"
3. Enter: `plugins.pentora.ai`
4. Cloudflare will automatically create DNS records

**DNS Configuration (automatic):**
```
Type: CNAME
Name: plugins
Content: [auto-generated-r2-url].r2.cloudflarestorage.com
Proxy: Enabled (orange cloud)
TTL: Auto
```

## Step 3: Create R2 API Token

1. Go to R2 → Manage R2 API Tokens
2. Click "Create API Token"
3. Token name: `pentora-plugins-github-actions`
4. Permissions:
   - `Object Read & Write` on `pentora-plugins` bucket
5. Save the credentials:
   - **Account ID**: `abc123...`
   - **Access Key ID**: `def456...`
   - **Secret Access Key**: `ghi789...` (shown once!)

## Step 4: Configure GitHub Secrets

Go to pentora-plugins repository → Settings → Secrets and variables → Actions

Add these secrets:

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `R2_ACCOUNT_ID` | Your Cloudflare Account ID | Found in R2 overview |
| `R2_ACCESS_KEY_ID` | R2 API token access key | From Step 3 |
| `R2_SECRET_ACCESS_KEY` | R2 API token secret | From Step 3 (shown once!) |
| `CLOUDFLARE_ZONE_ID` | Zone ID for pentora.ai | Dashboard → Overview |
| `CLOUDFLARE_API_TOKEN` | API token with Cache Purge permission | API Tokens page |

### Finding Cloudflare Zone ID

1. Go to Cloudflare Dashboard
2. Select `pentora.ai` domain
3. Scroll down on Overview page
4. Copy "Zone ID" from the right sidebar

### Creating Cloudflare API Token for Cache Purge

1. Go to Profile → API Tokens
2. Click "Create Token"
3. Use "Edit zone DNS" template or create custom with:
   - Permissions: `Zone > Cache Purge > Purge`
   - Zone Resources: `Include > Specific zone > pentora.ai`
4. Create token and save it

## Step 5: Enable CORS (Optional but Recommended)

For browser-based plugin downloads, configure CORS:

1. R2 bucket → Settings → CORS policy
2. Add this policy:

```json
[
  {
    "AllowedOrigins": ["*"],
    "AllowedMethods": ["GET", "HEAD"],
    "AllowedHeaders": ["*"],
    "ExposeHeaders": ["ETag"],
    "MaxAgeSeconds": 3600
  }
]
```

## Step 6: Configure Cloudflare Cache Rules

1. Go to Cloudflare Dashboard → Caching → Cache Rules
2. Create rule for `plugins.pentora.ai`:

**Rule Name:** `Pentora Plugins Cache`

**Match:**
- Hostname equals `plugins.pentora.ai`

**Settings:**
- Browser TTL: 1 hour
- Edge TTL: 1 hour
- Cache Level: Standard
- Cache by: Query String (use default)

## Step 7: Test the Setup

### Manual Upload Test

```bash
# Install rclone
curl https://rclone.org/install.sh | sudo bash

# Configure rclone
rclone config create r2 s3 \
  provider Cloudflare \
  access_key_id YOUR_ACCESS_KEY \
  secret_access_key YOUR_SECRET_KEY \
  endpoint https://YOUR_ACCOUNT_ID.r2.cloudflarestorage.com \
  acl public-read

# Upload test file
echo "test" > test.txt
rclone copy test.txt r2:pentora-plugins/

# Verify
curl https://plugins.pentora.ai/test.txt
# Should return: test
```

### GitHub Actions Test

1. Go to pentora-plugins repository
2. Actions → "Sync Plugins to Cloudflare R2"
3. Click "Run workflow" → "Run workflow"
4. Wait for completion (~1-2 minutes)
5. Check job logs for success

### Verify Plugin Access

```bash
# Test manifest download
curl -I https://plugins.pentora.ai/manifest.yaml
# Should return: HTTP/2 200

# Test plugin download
curl https://plugins.pentora.ai/ssh/ssh-weak-cipher.yaml | head -20

# Test with pentora CLI
pentora plugin update --dry-run
# Should show: "Fetching from official..." (plugins.pentora.ai)
```

## Step 8: Monitor Usage

### R2 Dashboard

- Storage: Check bucket size
- Requests: Monitor GET requests
- Bandwidth: Track egress

### Cloudflare Analytics

- Cache Hit Rate: Should be >90% after first downloads
- Bandwidth Saved: Cached vs origin requests
- Top Paths: Most downloaded plugins

## Troubleshooting

### Issue: 403 Forbidden

**Cause:** Bucket not public or wrong permissions

**Fix:**
1. R2 bucket → Settings → Public Access
2. Enable "Allow Access"
3. Verify CORS policy

### Issue: 404 Not Found

**Cause:** File not uploaded or wrong path

**Fix:**
```bash
# List bucket contents
rclone ls r2:pentora-plugins/

# Check specific file
rclone ls r2:pentora-plugins/manifest.yaml
```

### Issue: GitHub Actions Fails

**Cause:** Missing or incorrect secrets

**Fix:**
1. Verify all 5 secrets are set
2. Check secret values (no extra spaces)
3. Regenerate API tokens if needed

### Issue: Stale Cache

**Cause:** Cloudflare cache not purged

**Fix:**
```bash
# Manual cache purge
curl -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/purge_cache" \
  -H "Authorization: Bearer $API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"hosts":["plugins.pentora.ai"]}'
```

## Cost Estimation

Cloudflare R2 Pricing (as of 2024):

- **Storage**: $0.015/GB/month
- **Class A Operations** (writes): $4.50 per million
- **Class B Operations** (reads): $0.36 per million
- **Egress**: FREE (no bandwidth charges!)

**Example for pentora-plugins:**
- Storage: ~1 MB (19 plugins) = $0.000015/month
- Writes: ~100/month (GitHub pushes) = $0.00045/month
- Reads: ~10,000/month (downloads) = $0.0036/month
- **Total: < $0.01/month** 🎉

Compare to GitHub rate limits: UNLIMITED vs 60 requests/hour

## Security Best Practices

1. **Rotate API tokens regularly** (every 90 days)
2. **Use scoped tokens** (only permissions needed)
3. **Monitor access logs** (Cloudflare Analytics)
4. **Enable Cloudflare WAF** (block malicious requests)
5. **Set up alerts** (unusual traffic patterns)

## Maintenance

### Monthly Tasks

- [ ] Review R2 usage metrics
- [ ] Check Cloudflare analytics
- [ ] Verify cache hit rate >90%
- [ ] Test plugin downloads

### Quarterly Tasks

- [ ] Rotate R2 API tokens
- [ ] Review CORS policy
- [ ] Update cache rules if needed
- [ ] Check for stale files

## Support

For issues with R2 setup:
- Cloudflare Community: https://community.cloudflare.com/
- Cloudflare Docs: https://developers.cloudflare.com/r2/

For pentora-plugins issues:
- GitHub Issues: https://github.com/pentora-ai/pentora-plugins/issues
- Documentation: https://docs.pentora.ai
