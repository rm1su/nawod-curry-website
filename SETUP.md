# NAWOD CURRY Website — Setup & Deployment Guide

## Quick Overview

This is a static website (single HTML file + config) ready to deploy. It includes:
- Bilingual JP/EN content with toggle
- Snipcart e-commerce (merch shop with cart + checkout)
- Instagram feed integration (API or widget)
- Google Maps embed
- Responsive mobile design

**Estimated setup time: 1-2 hours**

---

## 1. Domain

**nawodcurry.com** is registered but appears unused (shows error page). Check with Nawod's owner if they own it. If not, alternatives:

| Domain | Status | Cost |
|--------|--------|------|
| nawodcurry.com | Taken (may be parked) | Make offer or ~$10-50 |
| nawodcurry.jp | Available | ~$40/year |
| nawodcurry.tokyo | Likely available | ~$15/year |
| nawod.curry | Not a TLD | N/A |

**Recommended:** Try to acquire `nawodcurry.com` first (contact current owner via Namecheap). If not available, `nawodcurry.jp` is the next best for a Tokyo restaurant.

**Where to buy:** Namecheap (https://namecheap.com) or Google Domains

---

## 2. Hosting (Netlify — Free)

### Initial Deploy

1. Create a GitHub repo:
   ```bash
   cd nawod-curry-website
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/YOUR_USERNAME/nawod-curry-website.git
   git push -u origin main
   ```

2. Go to https://app.netlify.com
3. Sign up / log in with GitHub
4. Click "Add new site" → "Import an existing project"
5. Select your GitHub repo
6. Deploy settings are auto-detected from `netlify.toml`
7. Click "Deploy site"

Your site will be live at `https://random-name.netlify.app` within 60 seconds.

### Connect Custom Domain

1. In Netlify dashboard → Site settings → Domain management
2. Click "Add custom domain"
3. Enter your domain (e.g., `nawodcurry.com`)
4. Follow DNS instructions:
   - **Option A (Recommended):** Point your domain's nameservers to Netlify
   - **Option B:** Add a CNAME record pointing to your Netlify URL
5. SSL certificate is automatically provisioned (free, via Let's Encrypt)

### Continuous Deployment

Every `git push` to `main` automatically redeploys the site. No manual steps needed.

---

## 3. Snipcart E-Commerce Setup

Snipcart handles the full checkout flow (cart, payment, shipping, taxes) with no backend needed.

### Create Account

1. Go to https://snipcart.com
2. Sign up (free to test, 2% transaction fee on live)
3. Go to Account → API Keys
4. Copy your **Public API Key**

### Configure

1. Open `index.html`
2. Find this line near the bottom:
   ```html
   data-api-key="YOUR_SNIPCART_PUBLIC_API_KEY"
   ```
3. Replace `YOUR_SNIPCART_PUBLIC_API_KEY` with your actual key

### Payment Processing

1. In Snipcart dashboard → Payment Gateway
2. Connect Stripe (recommended for Japan):
   - Create Stripe account at https://stripe.com
   - Stripe supports JPY and Japanese payment methods
   - Connect Stripe to Snipcart
3. Alternative: Square, PayPal

### Configure Shipping

1. Snipcart dashboard → Shipping
2. Add shipping rates for Japan domestic:
   - Standard: ¥500-800 (adjust based on actual shipping costs)
   - Express: ¥1,200+
3. For international: add separate rate or use weight-based

### Configure Taxes

1. Snipcart dashboard → Taxes
2. For Japan: consumption tax is 10% (included in price by convention)
3. Set tax to "included in price" for JPY products

### Adding New Products

To add a new product, copy this template into the shop section of `index.html`:

```html
<button class="snipcart-add-item buy-btn"
  data-item-id="unique-product-id"
  data-item-price="PRICE_IN_YEN"
  data-item-url="/index.html"
  data-item-name="Product Name"
  data-item-description="Short description"
  data-item-currency="jpy"
  data-item-weight="WEIGHT_IN_GRAMS"
  data-item-quantity="1"
  data-item-max-quantity="10">
  ADD TO CART
</button>
```

For products with size variants, add:
```html
data-item-custom1-name="Size"
data-item-custom1-options="S|M|L|XL|XXL"
```

### Snipcart Costs
- **Test mode:** Free (unlimited testing)
- **Live mode:** 2% per transaction (no monthly fee)
- **Stripe fees:** ~3.6% + ¥40 per transaction (Japan)
- **Total per sale:** ~5.6% + ¥40

---

## 4. Instagram Feed Integration

You have two options. Choose based on complexity preference:

### Option A: Behold.so Widget (Easiest — 5 minutes)

1. Go to https://behold.so
2. Create a free account
3. Add @nawodcurry as a feed source
4. Customize the widget appearance (grid, dark theme)
5. Copy the embed code
6. Replace the `<div id="insta-feed">` section in `index.html` with the embed code

**Pros:** No coding, auto-updates, free tier available
**Cons:** Behold branding on free tier, limited customization

### Option B: Instagram Basic Display API (More Control)

Requires a Meta Developer account and Facebook App:

1. **Create Meta Developer Account**
   - Go to https://developers.facebook.com
   - Create a new app → Consumer type
   - Add "Instagram Basic Display" product

2. **Configure Instagram Basic Display**
   - Go to Instagram Basic Display settings
   - Add Instagram Test User: use the @nawodcurry account
   - Set Valid OAuth Redirect URI: `https://your-domain.com/auth/`
   - Set Deauthorize Callback URL: `https://your-domain.com/deauth/`

3. **Get Access Token**
   - In the Instagram Basic Display → Basic Display tab
   - Under "User Token Generator", click "Generate Token" for the test user
   - Authorize the app via Instagram login
   - Copy the short-lived token
   - Exchange for a long-lived token (60 days):
     ```
     GET https://graph.instagram.com/access_token
       ?grant_type=ig_exchange_token
       &client_secret=YOUR_APP_SECRET
       &access_token=SHORT_LIVED_TOKEN
     ```

4. **Add Token to Website**
   - Open `index.html`
   - Find: `const INSTAGRAM_TOKEN = 'YOUR_INSTAGRAM_ACCESS_TOKEN';`
   - Replace with your long-lived token

5. **Token Refresh (Important!)**
   Long-lived tokens expire after 60 days. Set up auto-refresh:
   - Use Netlify Functions or a cron job to refresh every 50 days:
     ```
     GET https://graph.instagram.com/refresh_access_token
       ?grant_type=ig_refresh_token
       &access_token=LONG_LIVED_TOKEN
     ```

**Recommendation:** Start with **Behold.so** (Option A) to get live quickly, then switch to the API (Option B) later if you want full control.

---

## 5. Google Maps

The current embed uses a generic Shimokitazawa location. To get the exact pin:

1. Go to https://maps.google.com
2. Search: `東京都世田谷区北沢2-14-8 春日ビル`
3. Click "Share" → "Embed a map"
4. Copy the iframe `src` URL
5. Replace the `src` in the `<iframe>` tag in `index.html`

The CSS filter `filter: invert(90%) hue-rotate(180deg)...` makes the map match the dark theme. Adjust or remove if you prefer the default look.

---

## 6. Images

The site currently has placeholder boxes where photos should go. To add real images:

### Recommended Image Sources
- Instagram posts (download high-res versions)
- Have a photographer shoot the restaurant (the site mentions @hiinstn)
- Food photography from the owner's phone

### Image Locations in index.html

| Section | What to Replace | Recommended Size |
|---------|----------------|-----------------|
| Hero background | Add background-image to `.hero` | 1920x1080px |
| Story section | Replace `.story-image` div with `<img>` | 800x1200px (portrait) |
| Menu cards | Add images above card content | 600x400px |
| Shop products | Replace `.shop-card-image` divs with `<img>` | 600x600px (square) |
| Instagram feed | Auto-populated via API/widget | N/A |

### Image Optimization
- Use WebP format for best performance
- Compress with https://squoosh.app
- Keep hero image under 200KB, others under 100KB
- Add `loading="lazy"` to images below the fold

---

## 7. Content Updates

### Updating Menu Items
Search for `<!-- Menu -->` in index.html. Each `.menu-card` has both JP and EN text. Edit both `data-lang="ja"` and `data-lang="en"` content.

### Adding Events
Search for `<!-- Events -->`. Copy an existing `.event-card` block and modify the date, title, and description.

### Updating Hours
Search for `<!-- Info / Access -->`. Business hours are in the `.info-item-value` blocks under the "Hours" label.

---

## 8. Ongoing Costs Summary

| Service | Cost | Notes |
|---------|------|-------|
| Netlify hosting | Free | Includes SSL, CDN, auto-deploy |
| Domain | ~$12-40/year | Depends on TLD |
| Snipcart | 2% per transaction | No monthly fee |
| Stripe | ~3.6% + ¥40/transaction | Standard Japan rates |
| Behold.so (Instagram) | Free or $6/mo | Free tier has branding |
| Google Maps embed | Free | Standard embed, no API key needed |
| **Total fixed costs** | **~$12-88/year** | **Before any sales** |

---

## 9. Deployment Checklist

- [ ] Register or acquire domain
- [ ] Push code to GitHub
- [ ] Deploy to Netlify
- [ ] Connect custom domain + verify SSL
- [ ] Create Snipcart account + add API key
- [ ] Connect Stripe to Snipcart
- [ ] Set up shipping rates in Snipcart
- [ ] Set up Instagram feed (Behold or API)
- [ ] Update Google Maps embed with exact location
- [ ] Replace placeholder images with real photos
- [ ] Test checkout flow in Snipcart test mode
- [ ] Switch Snipcart to live mode when ready
- [ ] Test on mobile devices
- [ ] Share URL with Nawod's owner for review

---

## Quick Reference: File Structure

```
nawod-curry-website/
├── index.html        # The entire website (HTML + CSS + JS)
├── netlify.toml      # Netlify deployment config
├── SETUP.md          # This file
└── images/           # Create this folder for photos
    ├── hero.webp
    ├── story.webp
    ├── menu-rice-curry.webp
    ├── menu-kottu.webp
    ├── shop-tee-2026.webp
    └── shop-spice-set.webp
```
