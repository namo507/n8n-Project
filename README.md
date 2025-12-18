# Weather Guardian - n8n Daily Weather Automation

![n8n](https://img.shields.io/badge/n8n-Workflow%20Automation-brightgreen)
![OpenWeatherMap](https://img.shields.io/badge/OpenWeatherMap-API-orange)
![Supabase](https://img.shields.io/badge/Supabase-Database-green)

## What I Built

I created an intelligent weather automation system using n8n that runs every morning at 8:00 AM. The workflow fetches weather data from OpenWeatherMap, analyzes it for different alert conditions (like precipitation, heat waves, or frost), stores everything in a Supabase database, and sends me a nicely formatted HTML email report.

This project was built as a take-home assessment for ServiceAgent, and I wanted to showcase not just the technical implementation but also how automation can make daily tasks more convenient.

## Features

Here's what the workflow does:

- **Daily Automated Weather Checks** - Runs automatically every day at 8:00 AM
- **Intelligent Alert Detection** - Detects precipitation, heat, and frost conditions
- **Beautiful HTML Email Reports** - Sends emoji-rich weather summaries
- **Database Logging** - Stores all weather data in Supabase PostgreSQL
- **Comprehensive Metrics** - Tracks temperature, humidity, wind speed, and conditions
- **Creative Formatting** - Includes professional weather summaries with contextual tips

## Workflow Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   WEATHER GUARDIAN WORKFLOW                     │
└─────────────────────────────────────────────────────────────┘

          ┌─────────────────────────┐
          │   [1] SCHEDULE TRIGGER  │
          │     Daily at 8:00 AM    │
          └───────────┬─────────────┘
                   │
                   ↓
          ┌───────────┴─────────────┐
          │  [2] WEATHER API CALL  │
          │  Fetch OpenWeatherMap   │
          └───────────┬─────────────┘
                   │
                   ↓
          ┌───────────┴─────────────┐
          │   [3] WEATHER LOGIC    │
          │   Analyze & Alerts      │
          └───────────┬─────────────┘
                   │
                   ↓
          ┌───────────┴─────────────┐
          │  [4] SUPABASE INSERT   │
          │  Store weather_logs     │
          └───────────┬─────────────┘
                   │
                   ↓
          ┌───────────┴─────────────┐
          │    [5] EMAIL SENDER     │
          │   HTML Report Send      │
          └─────────────────────────┘
```

The workflow is a simple linear flow where each step passes data to the next:

1. **Schedule Trigger** - I configured this to run daily at 8:00 AM using a cron expression
2. **Weather API Call** - Fetches current weather data from OpenWeatherMap for London
3. **Weather Logic** - JavaScript code that analyzes the data and determines if there are any weather alerts
4. **Supabase Insert** - Stores all the weather information in my PostgreSQL database
5. **Email Sender** - Sends me a formatted HTML email with the weather summary

## How to Set This Up

### What You'll Need

Before you start, make sure you have:
- An n8n account (you can use [app.n8n.cloud](https://app.n8n.cloud) or self-host it)
- An OpenWeatherMap API key
- A Supabase account and project
- An email service configured (I used Gmail with an App Password)

### Step 1: Get Your OpenWeatherMap API Key

First, you'll need to sign up for a free OpenWeatherMap account:

1. Go to [openweathermap.org](https://openweathermap.org/api) and create an account
2. Navigate to the **API Keys** section in your profile
3. Generate a new API key (note: it takes 1-2 hours to activate)
4. Save this key - you'll need it for the workflow

The workflow uses this API endpoint:
```
https://api.openweathermap.org/data/2.5/weather?q={city}&appid={API_KEY}&units=metric
```

### Step 2: Set Up Your Supabase Database

I chose Supabase because it's easy to set up and has a great free tier:

1. Create a new project at [supabase.com](https://supabase.com)
2. Go to the **SQL Editor** in your project dashboard
3. Run this SQL script to create the weather_logs table:

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

4. Go to Settings -> API and copy your **Project URL** and **Anon Key**

### Step 3: Configure Your Email

For the email node, you'll need to set up credentials. I used Gmail:

- **For Gmail:** Enable 2-factor authentication and generate an App Password
- **For SMTP:** Get your server details from your email provider
- **For SendGrid/Mailgun:** Use your API credentials

### Step 4: Import the Workflow

Now you can import my workflow into your n8n instance:

1. Download the `weather-guardian-workflow.json` file from this repository
2. In n8n, go to **Workflows** -> **Import from File**
3. Upload the JSON file
4. Configure the credentials for each node:
   - **OpenWeatherMap HTTP Request:** Add your API key to the URL
   - **Supabase HTTP Request:** Add your Project URL and Anon Key in the headers
   - **Email Node:** Configure with your email service credentials

### Step 5: Test and Activate

Before setting it live, I recommend testing it:

1. Click **Execute Workflow** to run it manually
2. Check that:
   - Weather data fetches successfully
   - Alert logic works correctly
   - Data appears in your Supabase `weather_logs` table
   - You receive the email with proper formatting
3. Once everything works, toggle the workflow to **Active**

## Database Schema

Here's the structure of the weather_logs table:

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key (auto-generated) |
| `run_at` | TIMESTAMP | When the workflow ran |
| `city` | TEXT | City name |
| `temperature` | FLOAT | Temperature value |
| `temperature_unit` | TEXT | "C" or "F" |
| `condition` | TEXT | Weather condition like "Clear" or "Rain" |
| `humidity` | INT | Humidity percentage |
| `wind_speed` | FLOAT | Wind speed in m/s or mph |
| `alert_type` | TEXT | "precipitation", "heat", "frost", or "none" |
| `raw_response` | JSONB | Full API response for future reference |

## The Alert Logic

I wrote some JavaScript code to detect different weather alerts:

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

It checks for keywords in the weather condition, then checks temperature thresholds for heat waves and frost warnings.

## What the Email Looks Like

The email I receive every morning looks like this:

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

### "Invalid API Key" Error

If you see this error, your OpenWeatherMap API key probably hasn't activated yet. It can take 1-2 hours after generation. Just wait a bit and try again.

### Supabase Insert Fails

I ran into this a few times during development. Check that:
- Your table schema matches exactly what I have above
- Your Project URL and Anon Key are correct
- Row Level Security (RLS) policies aren't blocking the insert - you can disable RLS for testing

### Email Not Sending

For Gmail users, make sure you're using an App Password, not your regular password. Also:
- Check you're using the right SMTP port (587 for TLS, 465 for SSL)
- Make sure the "From" email matches your authenticated account

### Workflow Doesn't Trigger at 8 AM

Make sure:
- The workflow is toggled to **Active** (not just saved)
- Check your n8n instance timezone settings
- Verify the cron expression is `0 8 * * *`

## Repository Structure

```
n8n-Project/
├── README.md                             # This file
└── weather-guardian-workflow.json         # The complete n8n workflow
```

## What Makes This Project Unique

I tried to add some personal touches:

- **Contextual Tips:** The email includes activity suggestions based on the weather
- **Smart Formatting:** Used HTML and emojis to make the reports visually appealing
- **Raw Response Storage:** I store the complete API response in JSONB so I can analyze historical data later
- **Alert Categorization:** The logic handles multiple alert types with clear thresholds
- **Production-Ready:** Includes error handling and comprehensive logging

## What I Learned

This was my first time working with n8n, and I learned a lot about:
- Setting up automated workflows with cron triggers
- Working with REST APIs and handling JSON data
- Database design with PostgreSQL/Supabase
- Creating conditional logic in JavaScript
- Formatting HTML emails that look good

## Assessment Deliverables

As requested in the take-home assessment, this repository includes:

- Complete n8n workflow exported as JSON (`weather-guardian-workflow.json`)
- Comprehensive README with setup instructions (this file)
- Visual workflow diagram
- API configuration documentation
- Database schema and setup instructions
- Troubleshooting guide

## Get In Touch

If you have any questions about this project, feel free to reach out:

- **Email:** namit507@terpmail.umd.edu
- **GitHub:** [@namo507](https://github.com/namo507)
- **n8n Instance:** https://namit507.app.n8n.cloud

## License

This project was created for educational and assessment purposes.

---

**Built for ServiceAgent using n8n, OpenWeatherMap, and Supabase**

*Last Updated: December 17, 2025*
