------------------------------------------------------
Build a CloudFront + S3 Static Website Hosting Setup
------------------------------------------------------

### Objective

Create a production-ready static website hosting solution using Amazon S3 for storage and CloudFront as a Content Delivery Network (CDN). This setup provides

- Fast global content delivery with edge caching
- HTTPS support with custom domain (optional)
- Cost-effective static asset hosting
- High availability and scalability

Components included S3 bucket configuration, static website files, CloudFront distribution, Origin Access Control (OAC), bucket policies, cache behaviors, custom error pages, and testing.

---

## Prerequisites

- AWS Account with a userrole having 'AmazonS3FullAccess', 'CloudFrontFullAccess' (or equivalent).
- Basic understanding of HTMLCSS (we'll provide simple examples).
- Optional A custom domain name and Route 53 hosted zone (for custom domain setup).
- Optional AWS Certificate Manager (ACM) certificate for HTTPS on custom domain.

---

# Overview

1. Create an S3 bucket to store static website files (HTML, CSS, JS, images).
2. Upload a simple static website with multiple pages and assets.
3. Configure bucket policy to allow CloudFront access via Origin Access Control (OAC).
4. Create a CloudFront distribution pointing to the S3 bucket.
5. Configure cache behaviors, default root object, and custom error pages.
6. Test the distribution and verify caching behavior.
7. Optional Configure custom domain with HTTPS.

---

# Steps

---

## 1) Create an S3 Bucket for Website Hosting

Console S3 -> Buckets -> Create bucket

### Bucket Configuration

- Bucket name 'my-static-website-demo-2025' (must be globally unique)
- AWS Region Choose your preferred region (e.g., us-east-1)
- Object Ownership ACLs disabled (recommended)
- Block Public Access settings Keep all 4 checkboxes ENABLED
  - We will NOT make the bucket public
  - CloudFront will access via Origin Access Control (secure method)
- Bucket Versioning Enable (optional, but recommended for production)
- Encryption Server-side encryption with Amazon S3 managed keys (SSE-S3) - enabled by default
- Create bucket

Important Do NOT enable Static website hosting on the S3 bucket itself. We're using CloudFront as the front-end, not S3's website hosting feature.

---

## 2) Create Static Website Files Locally

Create a simple website structure on your local machine

### Project Structure

```
my-website
├── index.html
├── about.html
├── error.html
├── css
│   └── style.css
├── js
│   └── app.js
└── images
    └── logo.png
```

### index.html

```html
!DOCTYPE html
html lang=en
head
    meta charset=UTF-8
    meta name=viewport content=width=device-width, initial-scale=1.0
    titleMy Static Website - Hometitle
    link rel=stylesheet href=cssstyle.css
head
body
    header
        h1Welcome to My Static Websiteh1
        nav
            a href=index.htmlHomea
            a href=about.htmlAbouta
        nav
    header
    main
        h2CloudFront + S3 Demoh2
        pThis website is hosted on Amazon S3 and delivered via CloudFront CDN.p
        pCurrent time span id=timespanp
        img src=imageslogo.png alt=Logo style=max-width 200px;
    main
    footer
        p&copy; 2025 My Static Websitep
    footer
    script src=jsapp.jsscript
body
html
```

### about.html

```html
!DOCTYPE html
html lang=en
head
    meta charset=UTF-8
    meta name=viewport content=width=device-width, initial-scale=1.0
    titleAbout - My Static Websitetitle
    link rel=stylesheet href=cssstyle.css
head
body
    header
        h1About This Projecth1
        nav
            a href=index.htmlHomea
            a href=about.htmlAbouta
        nav
    header
    main
        h2Technology Stackh2
        ul
            liAmazon S3 - Object storage for static filesli
            liAmazon CloudFront - Global CDN for fast deliveryli
            liOrigin Access Control - Secure S3 accessli
        ul
        pa href=index.htmlBack to Homeap
    main
    footer
        p&copy; 2025 My Static Websitep
    footer
body
html
```

### error.html

```html
!DOCTYPE html
html lang=en
head
    meta charset=UTF-8
    meta name=viewport content=width=device-width, initial-scale=1.0
    title404 - Page Not Foundtitle
    link rel=stylesheet href=cssstyle.css
head
body
    header
        h1404 - Page Not Foundh1
    header
    main
        pSorry, the page you're looking for doesn't exist.p
        pa href=index.htmlReturn to Homeap
    main
    footer
        p&copy; 2025 My Static Websitep
    footer
body
html
```

### cssstyle.css

```css
 {
    margin 0;
    padding 0;
    box-sizing border-box;
}

body {
    font-family 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    line-height 1.6;
    color #333;
    background-color #f4f4f4;
}

header {
    background linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color white;
    padding 2rem;
    text-align center;
}

nav {
    margin-top 1rem;
}

nav a {
    color white;
    text-decoration none;
    margin 0 1rem;
    padding 0.5rem 1rem;
    background rgba(255, 255, 255, 0.2);
    border-radius 5px;
    transition background 0.3s;
}

nav ahover {
    background rgba(255, 255, 255, 0.3);
}

main {
    max-width 800px;
    margin 2rem auto;
    padding 2rem;
    background white;
    border-radius 8px;
    box-shadow 0 2px 10px rgba(0,0,0,0.1);
}

h2 {
    color #667eea;
    margin-bottom 1rem;
}

footer {
    text-align center;
    padding 1rem;
    color #666;
    margin-top 2rem;
}

ul {
    margin-left 2rem;
    margin-top 1rem;
}

li {
    margin 0.5rem 0;
}
```

### jsapp.js

```javascript
 Simple script to display current time
function updateTime() {
    const timeElement = document.getElementById('time');
    if (timeElement) {
        const now = new Date();
        timeElement.textContent = now.toLocaleTimeString();
    }
}

 Update time every second
setInterval(updateTime, 1000);
updateTime();

console.log('Website loaded successfully via CloudFront!');
```

### imageslogo.png

- Create or download a simple logo image (PNG format)
- Alternatively, use a placeholder from httpsvia.placeholder.com200x200.png

---

## 3) Upload Website Files to S3

Console S3 - Buckets - Select 'my-static-website-demo-2025'

### Upload Files

- Click Upload
- Add files
  - Drag all files and folders (index.html, about.html, error.html, css, js, images)
- Permissions Keep default (no public access)
- Properties Keep default
- Click Upload

### Verify Upload

- After upload completes, navigate through the bucket to verify folder structure
  - index.html
  - about.html
  - error.html
  - cssstyle.css
  - jsapp.js
  - imageslogo.png

---

## 4) Create CloudFront Distribution

Console CloudFront -> Distributions -> Create distribution

### Origin Settings

- Origin domain Select your S3 bucket from dropdown ('my-static-website-demo-2025.s3.amazonaws.com')
- Origin path Leave blank (serves from bucket root)
- Name Auto-populated (keep default or customize)
- Origin access Origin access control settings (recommended)
  - Click Create control setting
  - Name 'my-website-oac'
  - Signing behavior Sign requests (recommended)
  - Origin type S3
  - Click Create
  - Note You'll need to update the S3 bucket policy after creating the distribution

### Default Cache Behavior Settings

- Compress objects automatically Yes (enables gzipbrotli compression)
- Viewer protocol policy Redirect HTTP to HTTPS
- Allowed HTTP methods GET, HEAD (static websites don't need POSTPUTDELETE)
- Restrict viewer access No (for public website)
- Cache key and origin requests
  - Cache policy CachingOptimized (recommended)
  - Origin request policy None

### Function Associations (Optional)

- Leave blank for now (advanced use cases)

### Settings

- Price class Use all edge locations (best performance) or choose based on your needs
- AWS WAF web ACL None (or add if you have one)
- Alternate domain name (CNAME) Leave blank for now (we'll cover custom domains later)
- Custom SSL certificate Default CloudFront certificate
- Supported HTTP versions HTTP2, HTTP3
- Default root object index.html (important!)
- Standard logging Off (or enable to S3 bucket for access logs)
- IPv6 Enabled

### Custom Error Pages (Configure after creation)

- We'll add this in the next step

- Click Create distribution

### Copy the Bucket Policy

After creating the distribution, you'll see a banner saying

The S3 bucket policy needs to be updated

- Click Copy policy
- This copies a JSON policy to your clipboard

---

## 5) Update S3 Bucket Policy for OAC Access

Console S3 - Buckets - 'my-static-website-demo-2025' - Permissions - Bucket policy

- Click Edit
- Paste the policy copied from CloudFront (it should look like this)

```json
{
    Version 2012-10-17,
    Statement [
        {
            Sid AllowCloudFrontServicePrincipal,
            Effect Allow,
            Principal {
                Service cloudfront.amazonaws.com
            },
            Action s3GetObject,
            Resource arnawss3my-static-website-demo-2025,
            Condition {
                StringEquals {
                    AWSSourceArn arnawscloudfrontYOUR-ACCOUNT-IDdistributionYOUR-DISTRIBUTION-ID
                }
            }
        }
    ]
}
```

- Click Save changes

This policy allows ONLY your specific CloudFront distribution to access objects in the bucket. Much more secure than making the bucket public!

---

## 6) Configure Custom Error Pages

Console CloudFront - Distributions - Select your distribution - Error pages

### Add 404 Error Response

- Click Create custom error response
- HTTP error code 404 Not Found
- Customize error response Yes
- Response page path error.html
- HTTP response code 404
- Click Create custom error response

### Add 403 Error Response (Optional)

- Repeat for 403 Forbidden - error.html

Why When CloudFront can't find a file in S3, it returns 403 (Access Denied) or 404. Custom error pages provide better user experience.

---

## 7) Wait for Distribution Deployment

Console CloudFront - Distributions - Your distribution

- Status will show Deploying initially
- Wait 5-15 minutes for status to change to Enabled and Last modified date updates
- You can proceed once the deployment is complete

During this time, CloudFront is
- Provisioning edge locations globally
- Configuring cache behaviors
- Setting up OAC authentication

---

## 8) Test the CloudFront Distribution

### Get Distribution Domain Name

Console CloudFront - Distributions - Your distribution

- Copy the Distribution domain name (e.g., d1234abcd5678.cloudfront.net)

### Test in Browser

Open the following URLs

1. Home page httpsd1234abcd5678.cloudfront.net
   - Should display index.html with styling and JavaScript working

2. About page httpsd1234abcd5678.cloudfront.netabout.html
   - Should display about page with navigation working

3. Static assets 
   - Check browser DevTools (F12) - Network tab
   - Verify CSS, JS, and images load correctly
   - Look for x-cache Hit from cloudfront header on subsequent reloads

4. 404 Error httpsd1234abcd5678.cloudfront.netnonexistent.html
   - Should display custom error.html page

5. HTTP to HTTPS redirect httpd1234abcd5678.cloudfront.net
   - Should automatically redirect to HTTPS

### Test with curl

```bash
# Test home page
curl -I httpsd1234abcd5678.cloudfront.net

# Look for CloudFront headers
# x-cache Hit from cloudfront (cached) or Miss from cloudfront (first request)
# x-amz-cf-pop Edge location code (e.g., IAD89-C1)

# Test caching - run twice
curl httpsd1234abcd5678.cloudfront.netindex.html
# First request x-cache Miss from cloudfront
curl httpsd1234abcd5678.cloudfront.netindex.html
# Second request x-cache Hit from cloudfront
```

---

### Invalidate Cache (When you update files)

If you update files in S3, CloudFront will still serve old cached versions until TTL expires.

To force immediate update

Console CloudFront - Distributions - Your distribution - Invalidations

- Click Create invalidation
- Object paths 
  - `` (invalidate everything)
  - or `index.html` (specific file)
  - or `css` (all CSS files)
- Click Create invalidation
- Wait 1-2 minutes for invalidation to complete

Cost First 1000 invalidation paths per month are free, then $0.005 per path.

Better approach for production Use versioned file names (e.g., style.v2.css, app-20250207.js) so you never need to invalidate.

---

# Traffic Flow Explanation

1. User in Australia requests httpsd1234abcd5678.cloudfront.netindex.html
2. DNS resolves to nearest CloudFront edge location (e.g., Sydney)
3. CloudFront edge checks local cache
   - If cached (HIT) Returns file immediately (low latency ~10-50ms)
   - If not cached (MISS) Edge fetches from S3 origin in your chosen region
4. CloudFront authenticates with S3 using OAC (Origin Access Control)
5. S3 returns the file to CloudFront edge
6. CloudFront caches the file and returns to user
7. Next user in Australia gets cached version (HIT) - much faster!

---