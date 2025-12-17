# Weather Guardian - n8n Daily Weather Automation

![n8n](https://img.shields.io/badge/n8n-Workflow%20Automation-brightgreen)
![OpenWeatherMap](https://img.shields.io/badge/OpenWeatherMap-API-orange)
![Supabase](https://img.shields.io/badge/Supabase-Database-green)

## Project Overview

An intelligent daily weather automation system built with n8n that fetches weather data, analyzes conditions, detects weather alerts, stores data in Supabase, and sends beautiful HTML email reports.

### Built for ServiceAgent Take-Home Assessment

## Features

- **Daily Automated Weather Checks** - Runs every day at 8:00 AM
- **Intelligent Alert Detection** - Detects precipitation, heat, and frost conditions
- **Beautiful HTML Email Reports** - Emoji-rich weather summaries
- **Database Logging** - Stores all weather data in Supabase PostgreSQL
- **Comprehensive Metrics** - Temperature, humidity, wind speed, conditions
- **Creative Formatting** - Professional weather summaries with contextual tips

## Workflow Architecture

![Workflow Diagram](workflow-screenshot.png)

The workflow consists of 5 nodes:

1. **Schedule Trigger** - Daily execution at 8:00 AM
2. **Weather API Call** - Fetch OpenWeatherMap data
3. **Weather Logic** - Analyze and detect alerts
4. **Supabase Insert** - Store in weather_logs table
5. **Email Sender** - Send HTML report to user

## Setup Instructions

### Prerequisites
- n8n account ([app.n8n.cloud](https://app.n8n.cloud) or self-hosted)
- OpenWeatherMap API key
- Supabase account with project
- Email service (Gmail, SMTP, etc.)

### 1. OpenWeatherMap API Setup

1. Sign up at [openweathermap.org](https://openweathermap.org/api)
2. Navigate to **API Keys** section
3. Generate a new API key (activation takes 1-2 hours)
4. Copy your API key

**API Endpoint Used:**
```
https://api.openweathermap.org/data/2.5/weather?q={city}&appid={API_KEY}&units=metric
```

### 2. Supabase Database Setup

1. Create a new project at [supabase.com](https://supabase.com)
2. Navigate to **SQL Editor**
3. Run this table creation script:

```sql
CREATE TABLE weather_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_at TIMESTAMP DEFAULT NOW(),
  city TEXT NOT NULL,
  temperature FLOAT NOT NULL,
  temperature_unit TEXT NOT NULL,
  condition TEXT NOT NULL,
  humidity INT NOT NULL,
  wind_speed FLOAT NOT NULL,
  alert_type TEXT,
  raw_response JSONB
);
```

4. Get your **Project URL** and **Anon Key** from Settings -> API

### 3. Email Configuration

Configure email credentials in n8n:
- **Gmail:** Enable 2FA + generate App Password
- **SMTP:** Get server details from your provider
- **SendGrid/Mailgun:** Use API credentials

### 4. Import Workflow to n8n

1. Download `weather-guardian-workflow.json`
2. In n8n, go to **Workflows** -> **Import from File**
3. Upload the JSON file
4. Configure credentials:
   - **OpenWeatherMap HTTP Request:** Add API key to URL parameter
   - **Supabase HTTP Request:** Add Project URL + Anon Key as headers
   - **Email Node:** Configure your email service

### 5. Test and Activate

1. Click **Execute Workflow** to test manually
2. Verify:
   - Weather data fetches successfully
   - Alert logic detects conditions correctly
   - Data appears in Supabase `weather_logs` table
   - Email arrives with correct formatting
3. Toggle **Active** to enable daily 8:00 AM runs

## Database Schema

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key (auto-generated) |
| `run_at` | TIMESTAMP | Execution timestamp |
| `city` | TEXT | City name |
| `temperature` | FLOAT | Temperature value |
| `temperature_unit` | TEXT | "C" or "F" |
| `condition` | TEXT | Weather condition (e.g., "Clear", "Rain") |
| `humidity` | INT | Humidity percentage |
| `wind_speed` | FLOAT | Wind speed (m/s or mph) |
| `alert_type` | TEXT | "precipitation", "heat", "frost", or "none" |
| `raw_response` | JSONB | Full API response |

## Alert Detection Logic

```javascript
if (condition.match(/rain|snow|drizzle|storm|thunder/i)) {
  alertType = "precipitation";
} else if (tempC > 32 || tempF > 90) {
  alertType = "heat";
} else if (tempC < 0 || tempF < 32) {
  alertType = "frost";
} else {
  alertType = "none";
}
```

## Email Format

**Subject:** `Daily Weather for London - January 15, 2025`

**Body:**
```
Good morning!

Here's your weather summary for London:

Temperature: 18°C (64°F)
Condition: Partly Cloudy
Humidity: 65%
Wind Speed: 3.5 m/s

ALERT: None - Perfect day ahead!

Tip: Great weather for outdoor activities!
```

## Troubleshooting

### Issue: "Invalid API Key" Error
**Solution:** Wait 1-2 hours after OpenWeatherMap key generation for activation.

### Issue: Supabase Insert Fails
**Solution:** 
- Verify table schema matches exactly
- Check Project URL and Anon Key are correct
- Ensure RLS policies allow inserts (disable for testing)

### Issue: Email Not Sending
**Solution:**
- Gmail: Use App Password, not regular password
- Check SMTP port (587 for TLS, 465 for SSL)
- Verify "From" email matches authenticated account

### Issue: Workflow Doesn't Trigger at 8 AM
**Solution:**
- Ensure workflow is toggled **Active**
- Check n8n timezone settings
- Verify cron expression: `0 8 * * *`

## Repository Structure

```
n8n-Project/
├── README.md                             # This file
├── weather-guardian-workflow.json         # Exportable n8n workflow
└── workflow-screenshot.png                # Visual workflow diagram
```

## Unique Features

- **Contextual Tips:** Weather-based activity suggestions
- **Emoji-Rich Reports:** Beautiful visual email formatting
- **Raw Response Storage:** Full API data preserved in JSONB
- **Alert Categorization:** Smart weather condition detection
- **Production-Ready:** Error handling + comprehensive logging

## Assessment Deliverables

- Exported n8n workflow (`weather-guardian-workflow.json`)
- Comprehensive README (this file)
- Workflow screenshot (`workflow-screenshot.png`)
- API configuration documentation
- Database schema + setup instructions
- Setup instructions + troubleshooting

## Contact

- **Email:** namit507@terpmail.umd.edu
- **GitHub:** [@namo507](https://github.com/namo507)
- **n8n Instance:** https://namit507.app.n8n.cloud

## License

This project is created for educational and assessment purposes.

---

**Built with care for ServiceAgent using n8n, OpenWeatherMap, and Supabase**

*Last Updated: December 17, 2025*
