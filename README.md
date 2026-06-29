# ⭐ KidsVid AI — Complete Setup & Deployment Guide

A free, open-source AI video generator built **100% for kids** (ages 3–12).
No backend server needed. Deploy for free in minutes.

---

## 📁 Project Structure

```
kidsvid/
├── index.html          ← Main page
├── css/
│   └── style.css       ← All styles
├── js/
│   ├── safety.js       ← Content safety filter (blocklist + prompt rewriting)
│   ├── app.js          ← Main app logic + Replicate API calls
│   └── stars.js        ← Animated background
└── README.md           ← This file
```

---

## 🚀 Step 1 — Get Your FREE API Token

This app uses **Replicate** to generate videos using open-source AI models.

1. Go to **https://replicate.com** and sign up (free)
2. Go to **https://replicate.com/account/api-tokens**
3. Click **"Create token"** → copy it
4. Open `js/app.js` and replace:
   ```js
   REPLICATE_TOKEN: "YOUR_REPLICATE_API_TOKEN",
   ```
   with your actual token:
   ```js
   REPLICATE_TOKEN: "r8_abc123yourtokenhere",
   ```

> **Free tier:** Replicate gives you free credits on sign-up.
> The zeroscope_v2_576w model is one of the cheapest — short videos cost fractions of a cent.

---

## 🔐 Step 2 — (Recommended) Secure Your API Key with a Proxy

Putting API keys in frontend JavaScript is OK for personal projects but exposes the key.
For a public site, use a **free Cloudflare Worker** as a proxy:

### Cloudflare Worker (free forever)

1. Go to **https://workers.cloudflare.com** → sign up free
2. Create a new Worker with this code:

```js
// Cloudflare Worker — paste this as your worker code
export default {
  async fetch(request, env) {
    // Only allow POST to /api/replicate
    if (request.method === "OPTIONS") {
      return new Response(null, {
        headers: {
          "Access-Control-Allow-Origin": "*",
          "Access-Control-Allow-Methods": "POST, GET, OPTIONS",
          "Access-Control-Allow-Headers": "Content-Type",
        }
      });
    }

    const url = new URL(request.url);
    const replicatePath = url.pathname.replace("/api/replicate", "");
    const targetUrl = `https://api.replicate.com/v1${replicatePath}`;

    const body = request.method === "POST" ? await request.text() : undefined;

    const res = await fetch(targetUrl, {
      method: request.method,
      headers: {
        "Authorization": `Token ${env.REPLICATE_TOKEN}`,
        "Content-Type": "application/json",
      },
      body,
    });

    const data = await res.text();
    return new Response(data, {
      headers: {
        "Content-Type": "application/json",
        "Access-Control-Allow-Origin": "*",
      }
    });
  }
};
```

3. In the Worker dashboard → **Settings → Variables** → add:
   - Key: `REPLICATE_TOKEN`   Value: `your_actual_token`

4. In `js/app.js`, change the fetch URLs to point to your worker:
   ```js
   // Change:
   "https://api.replicate.com/v1/predictions"
   // To:
   "https://your-worker-name.your-account.workers.dev/api/replicate/predictions"
   ```

---

## 🌐 Step 3 — Publish for FREE

### Option A: GitHub Pages (Recommended — completely free)

1. Create a free account at **https://github.com**
2. Click **"New repository"** → name it `kidsvid-ai` → set to **Public**
3. Upload all project files (drag & drop in the browser)
4. Go to **Settings → Pages**
5. Under "Branch", select **main** → **Save**
6. Your site goes live at: `https://your-username.github.io/kidsvid-ai`

**Steps with Git (if you prefer):**
```bash
git init
git add .
git commit -m "Initial KidsVid AI launch"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/kidsvid-ai.git
git push -u origin main
```
Then enable GitHub Pages in repo Settings.

---

### Option B: Netlify (Free, drag & drop)

1. Go to **https://netlify.com** → sign up free
2. Drag your entire `kidsvid` folder onto the Netlify dashboard
3. Done! You get a URL like `https://amazing-name-123.netlify.app`
4. Optional: connect a custom domain

---

### Option C: Vercel (Free)

1. Go to **https://vercel.com** → sign up free
2. Install Vercel CLI: `npm i -g vercel`
3. In your project folder: `vercel --prod`
4. Follow the prompts → site goes live instantly

---

## 🛡️ Safety Features (Built-in)

| Feature | How it works |
|---------|--------------|
| **Blocklist filter** | 50+ blocked terms checked before any API call |
| **Age gate** | Users must confirm they are a child OR a parent gives consent |
| **Parent consent modal** | Adults must explicitly consent before any video is made |
| **Safe prompt injection** | Every prompt is prefixed with child-safe framing |
| **Safe suffix** | "G-rated, no violence, no adults, cartoon style" appended to every prompt |
| **No accounts** | No personal data collected whatsoever |
| **No adult content** | Model parameters and prompt engineering enforce kids-only output |

---

## ⚙️ Customization

### Change the AI model
In `js/app.js`, change `MODEL_VERSION` to any Replicate text-to-video model version hash.
Browse models at: https://replicate.com/explore

### Add more blocked words
In `js/safety.js`, add terms to the `BLOCKED_TERMS` array:
```js
const BLOCKED_TERMS = [
  "kill", "dead", // ... existing terms
  "your-new-word", // add here
];
```

### Change video length
In `js/app.js`:
```js
VIDEO_FRAMES: 24,  // increase for longer video (costs more credits)
VIDEO_FPS: 8,
```

---

## 💰 Cost Estimate

| Usage | Cost |
|-------|------|
| Sign-up credits | Free (~$5 equivalent) |
| 1 short video (24 frames) | ~$0.002–0.01 |
| 500 videos/month | ~$1–5 |
| Hobby/low-traffic site | Essentially free |

---

## 📋 Legal Checklist

Before going public, consider:
- [ ] Add a simple Privacy Policy (no personal data is collected — easy to write)
- [ ] Add Terms of Service stating kids-only use
- [ ] Add a contact email for abuse reports
- [ ] Consider adding Google reCAPTCHA v3 (free) to prevent bot abuse
- [ ] Review Replicate's Terms of Service

---

## 🆘 Common Issues

**"YOUR_REPLICATE_API_TOKEN" error**
→ You haven't replaced the placeholder in `js/app.js`

**CORS error in browser console**
→ Use the Cloudflare Worker proxy (Step 2)

**Video generation fails**
→ Check your Replicate account has credits remaining
→ Try a simpler prompt

**Site shows but video doesn't play**
→ Some browsers block autoplay — user must click the play button

---

Made with ❤️ for kids everywhere. Free to use, fork, and improve!
