# CogniFi Tester Onboarding Guide & FAQ

Welcome to the **CogniFi Beta Cohort**! 🚀

CogniFi is a privacy-first, self-hosted personal finance application designed to help you track income, expenses, and savings with powerful automation and custom pay-cycle alignment. This guide walks you through setting up your environment, importing transactions, and configuring your budget.

---

## 📋 Table of Contents
1. [Day 1: Initial Deployment & Setup](#1-day-1-initial-deployment--setup)
2. [Setting Up Your Pay Cycle (Budget Months)](#2-setting-up-your-pay-cycle-budget-months)
3. [Configuring Recurring Income & Expenses](#3-configuring-recurring-income--expenses)
4. [Bank Statement Import & The Rules Engine](#4-bank-statement-import--the-rules-engine)
5. [Granular Weekly Tracking (Groceries & Fuel)](#5-granular-weekly-tracking-groceries--fuel)
6. [Troubleshooting & FAQ](#6-troubleshooting--faq)

---

## 1. Day 1: Initial Deployment & Setup

CogniFi is distributed as a single lightweight Docker container. It bundles both the backend .NET 10 API and the React frontend static assets, persisting all data to a local SQLite database file.

### Step A: docker-compose.yml Setup
Create a directory for your application files and add the following `docker-compose.yml` configuration:

```yaml
services:
  cognifi:
    image: ghcr.io/flexitcorp/cognifi-community:latest
    container_name: cognifi-app
    restart: unless-stopped
    ports:
      - "8080:5000"
    volumes:
      # Maps your local SQLite data directory
      - ./data:/app/data
      # Maps application logs
      - ./logs:/app/logs
    environment:
      - DATABASE_PATH=/app/data/cognifi.db
      - Jwt__Key=YOUR_SECURE_JWT_SECRET_KEY_HERE_MIN_32_CHARS
      - Jwt__Issuer=CogniFi
      - Jwt__Audience=CogniFi
      - License__PublicKey=YOUR_BETA_LICENSE_PUBLIC_KEY
      - Licensing__StartupEntitlement__LicenseFilePath=/app/data/CogniFi-License.json
      - TZ=Australia/Sydney # Change to your preferred timezone
```

Create a `.env` file in the same directory:
```env
# Change this secret key to a secure 32-character string
JWT_SECRET_KEY=some_super_secret_signing_key_32_characters_long
LICENSE_PUBLIC_KEY=your_issued_beta_license_verification_key
```

### Step B: Start the Container
Run the following command to download and boot the application:
```bash
docker compose up -d
```
The application will listen on port `8080` (or whichever port you mapped to host port `5000`).

### Step C: Upload Your Beta License File
1. Download your beta license file (`CogniFi-License.json`).
2. Open your web browser and navigate to `http://localhost:8080/setup/startup-license`.
3. Upload the license JSON file. This registers the cryptographic update entitlement for your container.
4. Once verified, click **Continue to Registration** to create your admin account.

---

## 2. Setting Up Your Pay Cycle (Budget Months)

Traditional budgeting apps force you to budget from the 1st to the 30th/31st of each calendar month. If you are paid weekly, fortnightly, or on a custom monthly day, this mismatch makes tracking cash flow difficult.

### Creating Month Definitions
During the initial Setup Wizard (or inside the **Settings** panel), you can define custom month boundaries:
* **Standard Calendar Months**: Boundaries align with default month start/end dates.
* **Custom Month Boundaries**: Define custom starting days (e.g., the 15th of each month). If you define the 15th, your budget month for January will run from **January 15th to February 14th**.
* When you import transactions, CogniFi automatically analyzes their dates and assigns them to the correct budget month according to your custom boundaries.

---

## 3. Configuring Recurring Income & Expenses

To automate your budget planning, you should set up templates for predictable cash flows.

### Recurring Income
For each source of income (e.g., your salary), configure the following:
* **Pay Cycle Frequency**: Weekly, fortnightly, monthly, or irregular.
* **Amounts**: Track gross, net, and tax deductions.
* **Pay Period Variations**: For complex pay frequencies, you can define specific pay calendars (e.g., defining how the 3rd paycheck in a 5-week month behaves).
* **Expected Pay Date**: Enter your regular pay day (e.g., every second Thursday) so the system can project upcoming cash flows on your dashboard.

### Recurring Expenses
Add recurring expense templates for repeating bills:
* Set name, category (e.g., Housing, Subscriptions), amount, and frequency.
* Specify whether the amount is fixed (like rent) or variable (like power).
* The application automatically populates these items into your new monthly budget sheets as they are generated.

---

## 4. Bank Statement Import & The Rules Engine

Importing bank statements manually preserves 100% of your financial privacy (no scraping login details or bank servers).

### How to Import Transactions
1. Export a statement CSV or QIF file from your bank's portal.
2. Inside CogniFi, navigate to **Transactions** > **Import**.
3. Select your bank format or use the default mapping, and upload the file.
4. CogniFi checks for duplicate entries using transaction IDs, timestamps, and amounts, ensuring no transaction is logged twice.

### Creating Matching Rules
When you import a statement, many transactions will be "Unmatched."
1. Click on an unmatched transaction and select the appropriate budget item (e.g., *Petrol* or *Electricity*).
2. The UI will prompt you to **Create a Vendor Matching Rule** based on the transaction description.
3. For example, manual categorization of a transaction from `WOOLWORTHS SUPERMARKETS SYDNEY` will create a rule matching the pattern `WOOLWORTHS`.
4. On all subsequent uploads, the matching engine will automatically categorize any description containing `WOOLWORTHS` into the *Groceries* budget item.

---

## 5. Granular Weekly Tracking (Groceries & Fuel)

Some categories are best managed on a shorter feedback loop than a whole month. 

* Under **Category Settings**, you can toggle **Weekly Tracking** on variable expense items like Groceries or Dining Out.
* This splits your monthly planned budget for that item into weekly milestones.
* The budget screen will show a week-by-week progress bar so you can see if you are overspending early in the month.

---

## 6. Troubleshooting & FAQ

### Q: Why am I seeing an "Update-Entitlement Locked" screen?
This screen appears if the container image you pulled has a release date **newer** than the update entitlement window in your license key.
* **Fix**: Either roll back your container image tag to a version built within your active license period, or upload a renewed license JSON key at `/setup/startup-license`.

### Q: How do I back up my database?
Since CogniFi uses SQLite, all your data is stored in a single database file.
* **Fix**: Simply back up the `cognifi.db` file located in the local `./data` directory mapped in your Docker Compose file. You can automate this using standard backup cron jobs.

### Q: What happens to my data when the 90-day beta license expires?
Your application will **never** stop running or lock you out of your historical data. After license expiration:
* You can continue using the application indefinitely on the current version.
* You will only be restricted from updating to newer container builds published after the expiry date.

### Q: How do I add another user (partner/family member)?
CogniFi supports multi-user isolation, but the number of active user accounts is gated by your license tier:
* **Personal**: Allows 1 user account.
* **Family**: Allows up to 5 user accounts.
* **Professional**: Unlimited user accounts.
To add more accounts, you must upgrade your license tier. Go to **Settings** > **License Info** to check your limits.
