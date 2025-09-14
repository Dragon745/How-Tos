# Firebase Hosting Setup Instructions

## Overview

This document provides step-by-step instructions for setting up Google Cloud Platform (GCP) and Firebase for hosting your website.

**Why You Need to Create Your Own Accounts:**

Instead of providing hosting services through my account (which would involve monthly hosting charges), I'm asking you to create and own the hosting accounts yourself. This approach offers several benefits:

- **Full Control**: You own and control your hosting infrastructure
- **No Third-Party Charges**: You pay Google directly, avoiding any markup or monthly fees through me
- **Transparency**: You have complete visibility into costs and usage
- **Independence**: You're not dependent on my hosting account for your website
- **Cost-Effective**: Google's free tier covers most small business needs, and you only pay for what you use

By following these instructions, you'll create and own the hosting accounts, giving you full control while avoiding third-party hosting charges.

## Prerequisites

- A Google account (Gmail)
- A valid credit/debit card for billing setup
- Approximately 15-20 minutes to complete the setup

## Step 1: Create Google Cloud Platform Account

### 1.1 Sign Up for Google Cloud

1. Navigate to [Google Cloud Platform](https://console.cloud.google.com/)
2. Click **"Get started for free"**
3. Sign in with your Google account
4. Complete the account setup process

### 1.2 Set Up Billing

1. Go to [Google Cloud Billing](https://console.cloud.google.com/billing)
2. Click **"Create billing account"** (if you don't have one) or **"Link a billing account"**
3. Add your payment method (credit/debit card)
4. **Important**: Google provides $300 in free credits for the first 90 days
5. You will only be charged if usage exceeds the free tier limits
6. **Note**: You must complete billing setup before creating any projects

### 1.3 Grant Access to Developer

1. Navigate to [IAM & Admin](https://console.cloud.google.com/iam-admin/iam)
2. Click **"Grant Access"**
3. Add my email address: **contact@syedqutubuddin.in**
4. Assign the **Owner** role

**Note**: The Owner role gives me full access to everything - APIs, billing, Firebase, reCAPTCHA, and all project management. This is much simpler than assigning multiple specific roles, and I'll be able to set up all required APIs (Google Maps, Places, Geocoding, reCAPTCHA) and create API keys myself.

## Step 2: Create Firebase Project

### 2.1 Create New Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click **"Add project"**
3. Enter project name: `your-project-name` (or your preferred name)
4. Choose whether to enable Google Analytics (recommended: **Yes**)
5. Select your Google Cloud billing account (created in Step 1.2)
6. Click **"Create project"**
7. Wait for the project to be created (usually takes 1-2 minutes)

### 2.2 Enable Billing for Firebase

1. In the Firebase Console, go to **Project Settings** → **Usage and billing**
2. Click **"Upgrade project"**
3. Select the **Blaze (Pay as you go)** plan
4. Link your Google Cloud billing account (created in Step 1.2)
5. **Note**: This is required for Firebase Functions, but you'll stay within free tier limits for most usage
6. Confirm the upgrade (no immediate charges will occur)

### 2.3 Grant Developer Access

1. In Firebase Console, go to **Project Settings** → **Users and permissions**
2. Click **"Add member"**
3. Add my email address: **contact@syedqutubuddin.in**
4. Assign the **Owner** role
5. Click **"Add member"**

**Additional Access Needed for reCAPTCHA:**

6. Go to [reCAPTCHA Admin Console](https://www.google.com/recaptcha/admin)
7. Click **"Add member"** or **"Invite user"**
8. Add my email address: **contact@syedqutubuddin.in**
9. Assign **Admin** or **Owner** role for reCAPTCHA management

## What Happens Next

Once you complete these steps, I will:

- ✅ Enable all required Google Cloud APIs (Maps, Places, Geocoding, reCAPTCHA)
- ✅ Create and configure API keys with proper security restrictions
- ✅ Set up reCAPTCHA site and configure security settings
- ✅ Configure Firebase Hosting for your website
- ✅ Set up Firebase Functions for backend logic
- ✅ Configure Google Maps integration
- ✅ Integrate reCAPTCHA for form protection
- ✅ Deploy the website to your Firebase project
- ✅ Set up automated deployments
- ✅ Configure monitoring and analytics
- ✅ Handle custom domain setup (if needed)

**Important**: While you own the accounts and billing, I handle all the technical setup and ongoing maintenance. You won't need to manage any technical aspects - I'll take care of everything once you provide me with the necessary access.

## Cost Information

### Free Tier Limits (per month)

- **Firebase Hosting**: 10GB storage, 10GB transfer
- **Firebase Functions**: 2M invocations, 400K GB-seconds
- **Google Maps API**: $200 in free credits
- **Google Cloud**: $300 free credits for first 90 days

### Typical Monthly Costs

For a small business website, you'll likely stay within free tier limits. If you exceed them, costs are typically:

- Hosting: $0.026 per GB
- Functions: $0.40 per million invocations
- Maps API: $2-7 per 1000 requests (depending on API type)

### Getting Help

If you encounter any issues during setup:

1. Check the [Firebase Documentation](https://firebase.google.com/docs)
2. Review [Google Cloud Documentation](https://cloud.google.com/docs)
3. Contact me with specific error messages

## Security Best Practices

- ✅ Restrict API keys to specific domains/IPs
- ✅ Use Firebase Security Rules
- ✅ Enable HTTPS only
- ✅ Regularly review access permissions
- ✅ Monitor usage and costs

## Next Steps

After completing this setup:

1. Share the project details with me at **contact@syedqutubuddin.in**
2. Provide the Firebase project ID
3. Share any custom domain information
4. Confirm API keys are properly configured

---

**Note**: Keep your billing information and project access secure. Only share access with trusted individuals.
