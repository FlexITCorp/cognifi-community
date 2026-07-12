# CogniFi - Self-Hosted Personal Budgeting Application

CogniFi is a privacy-first, self-hosted personal finance and budgeting application designed to help individuals and families track income, expenses, and savings with powerful automation, intelligent matching features, and custom pay-cycle alignment.

---

## 🚀 Key Features

* **Smart Pay-Cycle Budgeting**: Align budget periods with your actual pay schedule (e.g., fortnightly or custom calendar definitions) rather than strict calendar months.
* **Intelligent Transaction Matching**: Import bank statements (CSV/QIF) and let the rules engine automatically categorize transactions based on vendor patterns.
* **Recurring Income & Expenses**: Automatically generate monthly budget items from recurring templates.
* **Multi-User Isolation & Collaboration**: Deploy a single container and support multiple users with secure, isolated data environments.
* **Privacy & Control**: Keep 100% ownership of your financial data on your own infrastructure.

---

## 📦 Deployment Guide

CogniFi is distributed as a single lightweight Docker container containing both the React frontend and the .NET Web API.

### 1. Prerequisites
* **Docker & Docker Compose** installed on your host system.
* **A custom domain and reverse proxy** (e.g., Caddy, Nginx, or Traefik) configured with SSL.

### 2. Create the Configuration
Create a directory on your host (e.g., `cognifi`) and add the following two files:

#### `docker-compose.yml`
```yaml
services:
  cognifi:
    build:
      context: ..
      dockerfile: Dockerfile
    image: ghcr.io/flexitcorp/cognifi-community:latest
    container_name: cognifi-app
    restart: unless-stopped
    
    ports:
      # Map container port 5000 to host port 8080
      - "8080:5000"
    
    volumes:
      # Persist SQLite database on host filesystem
      - ../data:/app/data
      # Persist application logs
      - ../logs:/app/logs
    
    environment:
      # Database configuration
      - DATABASE_PATH=/app/data/cognifi.db
      
      # JWT Authentication
      - Jwt__Key=${JWT_SECRET_KEY}
      - Jwt__Issuer=${JWT_ISSUER:-CogniFi}
      - Jwt__Audience=${JWT_AUDIENCE:-CogniFi}
      
      # Startup entitlement file paths
      - Licensing__StartupEntitlement__LicenseFilePath=/app/data/CogniFi-License.json
      - Licensing__StartupEntitlement__ReleaseManifestPath=/app/data/CogniFi-ReleaseManifest.json
      
      # ASP.NET Core settings
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=http://+:5000
      
      # Enable legacy registration for test execution
      - Features__AllowLegacyRegistration=true
      
      # Disable rate limits during local test runs
      - RateLimiting__AuthLogin__PermitLimit=9999
      - RateLimiting__RegisterWithLicense__PermitLimit=9999
      - RateLimiting__StartupUpload__PermitLimit=9999
      - RateLimiting__AuthRefresh__PermitLimit=9999
      - RateLimiting__Entitlement__PermitLimit=9999
      - RateLimiting__LicenseApi__PermitLimit=9999
      - RateLimiting__ApiGeneral__PermitLimit=9999
      
      # Optional: Customize logging level
      - Logging__LogLevel__Default=Information
      - Logging__LogLevel__Microsoft.AspNetCore=Warning
      
      # Optional: Set timezone (default is UTC)
      - TZ=Australia/Sydney
    
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:5000/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 10s
    
    # Resource limits (optional - adjust based on your needs)
    deploy:
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M
```

#### `.env`
Create a `.env` file in the same directory to define your secure secrets:
```env
# Secure JWT signing key (minimum 32 characters)
JWT_SECRET_KEY=your_super_secret_jwt_key_here_change_in_production
```

### 3. Start the Container
Run the following command to download and start the application in the background:
```bash
docker compose up -d
```
The application will listen on port `8080` inside the container network. Point your reverse proxy to forward traffic to `http://localhost:8080`.

---

## 🔑 Licensing & Registration

CogniFi uses an update-entitlement model. Active testers can download and run all releases published during their active license term. After expiration, the application continues to run, but upgrading to newer releases requires an active license.

### Obtain a Free Beta License
During the beta cohort phase, we are issuing **90-day time-limited Personal Tier licenses** to all active testers. 
- You can download the shared standard public beta key directly: **[Download CogniFi-License.json](https://raw.githubusercontent.com/FlexITCorp/cognifi-community/main/CogniFi-License.json)**.
- Alternatively, request your own unique key directly from the [🔑 How to Get Your Free 90-Day Beta License](https://github.com/FlexITCorp/cognifi-community/discussions/3) discussion thread.
- Save this file as `CogniFi-License.json` in your local directory (next to `docker-compose.yml`).

### First-Time Onboarding
1. When starting the application for the first time, navigate to `/setup/startup-license` in your browser.
2. Upload your signed license JSON file (`CogniFi-License.json`). This registers the update entitlement on your local instance.
3. Proceed to the Registration screen to create your admin account. The signup flow will automatically detect and associate the uploaded license.

For a comprehensive walkthrough of the onboarding wizard, configuration instructions, and features (like smart rules matching or weekly tracking), see the [Onboarding & FAQ Guide](USER_ONBOARDING_GUIDE.md).

---


## 🩺 Diagnostics & Health Checks

You can query the following endpoints to verify the status of your deployment:
* **Container Health**: `GET /health` (returns standard status and entitlement flags)
* **Entitlement Diagnostics**: `GET /api/entitlement-status` (details active license metadata, build date, and entitlement status)

---

## 🔮 Future Roadmap

We are actively developing new enhancements to expand CogniFi's capabilities. Here is what is planned for upcoming releases:

* **Dynamic Shift Pay Estimator & Forecaster (High Priority)**: A proactive earnings forecasting tool where users can simulate upcoming gross pay by entering shift tags (Day/Noon/Night) and hours. Supports complex compounding multipliers, regional public holiday API integrations, and split-rate logic for shifts crossing the midnight boundary.
* **Household Budget Sharing**: Inviting members, managing roles (Owner/Admin/Member/Viewer), and collaborative budgeting under a shared workspace.
* **Advanced Reports & Insights**: Enhanced charts showing category-based spending trends, budget-to-actual variance analysis, and net worth/savings rate tracking over time.
* **Drag-and-Drop Transaction Categorization**: Drag imported bank transactions directly into category folders for fast, visual organizing.
* **Streamlined Category Management**: A simplified, low-friction workflow to quickly add and organize budget categories and items without labor-intensive forms or multiple clicks.
* **Bank API Integration**: Automated transaction imports directly from bank feeds (in addition to existing CSV and QIF manual uploads).

---

## 💬 Community & Feedback

Need help setting up your container, want to request a feature, or report a bug? Join the [CogniFi Discussions Community](https://github.com/FlexITCorp/cognifi-community/discussions)!
* **Questions & Setup Help**: Post a thread in the `Q&A` section.
* **Roadmap & Suggestions**: Share feature requests or suggest improvements under the [Roadmap & Suggestions](https://github.com/FlexITCorp/cognifi-community/discussions/2) discussion.
* **Technical Bugs**: Please report any non-critical technical bugs under the `Bug Reports` category.
