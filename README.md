# 🎯 DamaDam Target Profile Scraper

Automated scraper that processes specific targets from the Target sheet every 58 minutes and marks them as complete when done.

## Features

✅ **Scrapes "Pending 🚨" status** from Target sheet  
✅ **Runs every 58 minutes** via GitHub Actions  
✅ **Marks complete when done** (✅ Completed)  
✅ **Shared Google Sheet** with Online Scraper  
✅ **Append-only updates** (no row 2 insertion)  
✅ **Fixed gspread deprecation warnings**  
✅ **Modern Google Auth** (google-auth instead of oauth2client)  

## Setup

### 1. Repository Secrets

Add these secrets to your GitHub repository:

```
DAMADAM_USERNAME=your_username
DAMADAM_PASSWORD=your_password
DAMADAM_USERNAME_2=backup_username (optional)
DAMADAM_PASSWORD_2=backup_password (optional)
GOOGLE_SHEET_URL=https://docs.google.com/spreadsheets/d/your_sheet_id
GOOGLE_CREDENTIALS_JSON={"type":"service_account",...}
```

### 2. Google Sheets Setup

1. Create a Google Sheet with these tabs:
   - **Profiles** - Main data storage
   - **Target** - Target management (this scraper's focus)
   - **Tags** - Optional tag mapping
   - **Logs** - Change tracking
   - **Dashboard** - Run statistics

2. Share the sheet with your service account email

### 3. Target Sheet Format

The **Target** sheet should have these columns:
- **Column A**: Nickname (the username to scrape)
- **Column B**: Status (use "Pending" or "Pending 🚨" for targets to process)
- **Column C**: Remarks (automatically updated with results)
- **Column D**: Source (optional, defaults to "Manual")

### 4. Service Account

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing
3. Enable Google Sheets API and Google Drive API
4. Create a Service Account
5. Download the JSON key file
6. Copy the entire JSON content to `GOOGLE_CREDENTIALS_JSON` secret

## How It Works

1. **Every 58 minutes**, GitHub Actions triggers the scraper
2. Scraper scans the **Target** sheet for "Pending" status entries
3. For each pending target:
   - Updates status to "🔄 Processing"
   - Scrapes the complete profile from DamaDam
   - **Appends data** to Profiles sheet (never overwrites)
   - Marks target as "✅ Completed" with timestamp and details
4. Failed targets are marked as "❌ Failed" with error details

## Target Status Flow

```
Pending 🚨 → 🔄 Processing → ✅ Completed
                           ↘ ❌ Failed (on error)
```

## Data Structure

The scraper collects the same data as Online Scraper:
- Profile image, nickname, tags
- Last post URL and timestamp
- Friend status, city, gender, marital status
- Age, join date, followers, posts count
- Profile link, intro, verification status
- Source (Target/Manual) and scraping timestamp

## Scheduling

- **Primary**: Every 58 minutes via cron schedule
- **Manual**: Can be triggered manually with custom parameters
- **Timeout**: 57 minutes max (to avoid overlap with next run)
- **Coordination**: Runs at different times than Online Scraper (15min vs 58min)

## Adding Targets

To add new targets to scrape:

1. Open your Google Sheet
2. Go to the **Target** tab
3. Add rows with:
   - **Nickname**: The DamaDam username
   - **Status**: "Pending" or "Pending 🚨"
   - **Remarks**: Leave empty (will be auto-filled)
   - **Source**: Optional (defaults to "Manual")

## Monitoring

- Check the **Target** sheet to see completion status
- Check the **Dashboard** sheet for run statistics
- Check the **Logs** sheet for detailed change tracking
- GitHub Actions logs show real-time progress
- Failed runs upload debug artifacts

## Coordination with Online Scraper

Both scrapers use the same Google Sheet but:
- **Target Scraper**: Processes specific targets from Target sheet
- **Online Scraper**: Focuses on currently online users
- Both append data (no conflicts)
- Shared formatting and structure
- Different schedules (58min vs 15min) to avoid conflicts
