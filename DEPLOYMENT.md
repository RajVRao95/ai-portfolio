# 🚀 Deployment Guide

This guide will help you deploy your AI portfolio website and connect your Flowise agent for free.

## 📋 Prerequisites

- Your website files (index.html, styles.css, script.js, etc.)
- A running Flowise agent (local or deployed)
- A GitHub account (optional but recommended)

## 🌐 Free Hosting Options

### 1. Netlify (Recommended) ⭐

**Why Netlify?**
- Instant deployments
- Custom domains
- Form handling built-in
- HTTPS by default
- Great for static sites

**Deployment Steps:**

1. **Quick Deploy (Drag & Drop)**
   - Go to [netlify.com](https://netlify.com)
   - Sign up for a free account
   - Drag your project folder to the deployment area
   - Your site is live instantly!

2. **Git-based Deployment** (Recommended)
   ```bash
   # Create git repository
   git init
   git add .
   git commit -m "Initial portfolio commit"

   # Push to GitHub
   git remote add origin https://github.com/yourusername/ai-portfolio.git
   git push -u origin main
   ```

   - Connect your GitHub repo to Netlify
   - Enable auto-deployments
   - Every push to main branch = automatic deployment

3. **Custom Domain Setup**
   - In Netlify dashboard: Site settings → Domain management
   - Add your custom domain
   - Update DNS records as instructed

### 2. Vercel

**Perfect for Next.js but also great for static sites**

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel

# Follow the prompts
```

### 3. GitHub Pages

**Free hosting directly from your GitHub repository**

1. Create repository named `yourusername.github.io`
2. Push your files to the main branch
3. Enable GitHub Pages in repository settings
4. Your site will be available at `https://yourusername.github.io`

### 4. Firebase Hosting

**Google's free hosting with excellent performance**

```bash
# Install Firebase CLI
npm install -g firebase-tools

# Login to Firebase
firebase login

# Initialize hosting
firebase init hosting

# Deploy
firebase deploy
```

## 🤖 Connecting Your Flowise Agent

### Option 1: Deploy Flowise First

**Free Flowise Hosting Options:**

1. **Railway** (Recommended)
   ```bash
   # In your Flowise directory
   railway login
   railway init
   railway up
   ```

2. **Render**
   - Connect your Flowise GitHub repo
   - Deploy as a web service
   - Free tier available

3. **Heroku** (with limitations)
   ```bash
   # Create Heroku app
   heroku create your-flowise-app

   # Deploy
   git push heroku main
   ```

### Option 2: Connect to Deployed Flowise

Once your Flowise is deployed, update your website's `script.js`:

```javascript
// Replace the demo chat functionality
async function sendToFlowise(message) {
    const FLOWISE_URL = 'https://your-flowise-app.railway.app/api/v1/prediction/YOUR-FLOW-ID';

    try {
        const response = await fetch(FLOWISE_URL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': 'Bearer YOUR-API-KEY' // If you have API key protection
            },
            body: JSON.stringify({
                question: message,
                overrideConfig: {
                    // Any override configurations
                }
            })
        });

        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }

        const data = await response.json();
        return data.text || data.answer || data.response; // Adjust based on your API
    } catch (error) {
        console.error('Flowise API Error:', error);
        return 'Sorry, I\'m having trouble connecting right now. Please try again later.';
    }
}

// Update the sendMessage function
function sendMessage() {
    const message = chatInput.value.trim();
    if (message === '') return;

    // Add user message
    addMessage(message, true);
    chatInput.value = '';

    // Show typing indicator
    showTypingIndicator();

    // Call Flowise API
    sendToFlowise(message)
        .then(response => {
            hideTypingIndicator();
            addMessage(response);
        })
        .catch(error => {
            hideTypingIndicator();
            addMessage('Sorry, I encountered an error. Please try again.');
        });
}
```

### Option 3: Local Development with Tunneling

**For testing with local Flowise:**

1. **Using ngrok** (Recommended)
   ```bash
   # Install ngrok
   # Download from https://ngrok.com/

   # Expose your local Flowise (assuming port 3000)
   ngrok http 3000

   # Use the https URL in your website
   ```

2. **Using Cloudflare Tunnel**
   ```bash
   # Install cloudflared
   npm install -g cloudflared

   # Create tunnel
   cloudflared tunnel --url http://localhost:3000
   ```

## 🔧 Environment Variables

For security, use environment variables for API keys:

**Netlify:**
- Site settings → Environment variables
- Add `FLOWISE_API_KEY` and `FLOWISE_URL`

**Vercel:**
- Project settings → Environment Variables
- Add your variables

**GitHub Pages:**
- Use GitHub Secrets for Actions
- Not recommended for client-side API keys

## 📱 Making Your Site a PWA

Add to your `index.html`:

```html
<head>
    <!-- Existing head content -->

    <!-- PWA meta tags -->
    <meta name="theme-color" content="#6366f1">
    <link rel="manifest" href="manifest.json">
    <link rel="apple-touch-icon" href="icon-192x192.png">
</head>
```

Create `manifest.json`:

```json
{
  "name": "AI Portfolio - Your Name",
  "short_name": "AI Portfolio",
  "description": "Showcase of AI projects and intelligent agents",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#6366f1",
  "icons": [
    {
      "src": "icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

## 🔒 Security Best Practices

### 1. API Key Protection

**Never expose API keys in client-side code!**

Instead, create a serverless function:

**Netlify Functions:**
```javascript
// netlify/functions/chat.js
exports.handler = async (event, context) => {
    if (event.httpMethod !== 'POST') {
        return { statusCode: 405, body: 'Method Not Allowed' };
    }

    const { message } = JSON.parse(event.body);

    const response = await fetch(process.env.FLOWISE_URL, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${process.env.FLOWISE_API_KEY}`
        },
        body: JSON.stringify({ question: message })
    });

    const data = await response.json();

    return {
        statusCode: 200,
        body: JSON.stringify(data)
    };
};
```

### 2. CORS Configuration

If calling Flowise directly, configure CORS in your Flowise instance:

```javascript
// In your Flowise configuration
app.use(cors({
    origin: ['https://your-portfolio-site.netlify.app'],
    credentials: true
}));
```

## 📊 Analytics & Monitoring

### Google Analytics 4

Add to your `index.html`:

```html
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'GA_MEASUREMENT_ID');
</script>
```

### Performance Monitoring

Use tools like:
- Google PageSpeed Insights
- GTmetrix
- WebPageTest
- Lighthouse CI

## 🐛 Troubleshooting

### Common Issues:

1. **CORS Errors**
   ```
   Solution: Use serverless functions or configure CORS on Flowise
   ```

2. **API Key Exposure**
   ```
   Solution: Move API calls to serverless functions
   ```

3. **Slow Loading**
   ```
   Solution: Optimize images, minimize CSS/JS, use CDN
   ```

4. **Mobile Issues**
   ```
   Solution: Test on multiple devices, use responsive design tools
   ```

### Debug Tips:

1. **Check Browser Console**
   - Open Developer Tools (F12)
   - Look for JavaScript errors
   - Check Network tab for failed requests

2. **Test API Endpoints**
   ```bash
   # Test your Flowise endpoint
   curl -X POST "https://your-flowise-url/api/v1/prediction/flow-id" \
        -H "Content-Type: application/json" \
        -d '{"question": "Hello, how are you?"}'
   ```

3. **Validate HTML/CSS**
   - Use W3C validators
   - Check for syntax errors

## 🚀 Deployment Checklist

- [ ] Website files ready
- [ ] Flowise agent deployed and accessible
- [ ] API integration tested
- [ ] Environment variables configured
- [ ] Custom domain setup (optional)
- [ ] HTTPS enabled
- [ ] Analytics configured
- [ ] Performance optimized
- [ ] Mobile responsive tested
- [ ] SEO meta tags added
- [ ] Error handling implemented

## 📈 Next Steps

1. **Monitor Performance**
   - Set up Google Analytics
   - Monitor site speed
   - Track user interactions

2. **Continuous Improvement**
   - Add more AI projects
   - Update content regularly
   - Collect user feedback

3. **Scale Up**
   - Add blog section
   - Implement user authentication
   - Create project case studies

---

**Need Help?**
- Check the troubleshooting section
- Review browser console errors
- Test on multiple browsers/devices
- Reach out to the community

**Happy Deploying! 🚀**
