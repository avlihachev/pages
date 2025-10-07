---
title: "Garden Weather Bot"
description: "Intelligent Telegram bot for gardeners with automated weather notifications"
date: 2025-10-07T10:00:00+03:00
tags: ["telegram", "bot", "weather", "nodejs", "typescript", "api"]
categories: ["projects", "automation"]
featured: true
draft: false
ShowToc: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
---

# Garden Weather Bot

**Smart Telegram bot for gardeners** — automated weather notifications that help make informed decisions about plant care.

## Live Project & Real-Time Status

- **Try the Bot**: [@jordbrukbot](https://t.me/jordbrukbot) on Telegram
- **Web Interface**: [garden-weather-bot.fly.dev](https://garden-weather-bot.fly.dev/)
- **Source Code**: [GitHub Repository](https://github.com/avlihachev/garden_bot)

### Live Status Monitor

<div id="garden-bot-status-main" style="padding: 2rem; border: 1px solid #e1e5e9; border-radius: 12px; background: #ffffff; margin: 2rem 0;">
  <div style="text-align: center;">
    <h3>Garden Bot Live Status</h3>
    <p>Loading real-time status...</p>
    <p><strong>Status:</strong> 🟢 Online</p>
    <p><strong>Users:</strong> 40+ active gardeners</p>
    <p><strong>Uptime:</strong> 99.8%</p>
  </div>
</div>

<script src="https://garden-weather-bot.fly.dev/widget.js"></script>
<script>
document.addEventListener("DOMContentLoaded", function () {
  if (typeof GardenBotStatus !== 'undefined') {
    const widget = new GardenBotStatus({
      container: "#garden-bot-status-main",
      apiUrl: "https://garden-weather-bot.fly.dev/api/status",
      theme: "light"
    });
    widget.init();
  }
});
</script>

The status widget above shows real-time monitoring including uptime, active users, system health checks, and 24-hour activity metrics.

## Problem

Gardeners and farmers spend significant time checking weather forecasts to:

- Determine if plants need watering
- Find optimal timing for garden work
- Prepare for adverse weather conditions

## Solution

Garden Bot automatically sends personalized notifications based on:

- **User location** — precise data for specific areas
- **Custom thresholds** — configurable conditions for alerts
- **Real-time data** — integration with OpenWeatherMap API

## Key Features

### Geolocation

- GPS coordinates support
- City name search
- Automatic timezone detection

### Notification Settings

- Temperature and humidity thresholds
- Customizable notification timing
- Enable/disable toggle

### Weather Data

- Current weather and 5-day forecast
- Temperature, humidity, wind speed
- Precipitation probability

### Automation

- Daily weather checks
- Smart notifications only when needed
- Reliable task scheduler

## Tech Stack

### Backend

- **Node.js + TypeScript** — type safety and modern JavaScript
- **Telegraf.js** — powerful Telegram Bot API library
- **Express.js** — web server for status and API
- **node-cron** — task scheduler

### External APIs

- **OpenWeatherMap API** — reliable weather data
- **Telegram Bot API** — user interaction
- **Google reCAPTCHA** — bot protection

### Infrastructure

- **Fly.io** — cloud hosting with auto-scaling
- **Docker** — application containerization
- **GitHub Actions** — CI/CD pipeline

## Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Telegram      │    │   Garden Bot    │    │  OpenWeather    │
│     Users       │◄──►│     Server      │◄──►│      API        │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              ▼
                       ┌─────────────────┐
                       │   File System   │
                       │   User Data     │
                       └─────────────────┘
```

## Bot Commands

| Command      | Description                  |
| ------------ | ---------------------------- |
| `/start`     | Initialize bot interaction   |
| `/location`  | Set user location            |
| `/threshold` | Configure threshold values   |
| `/status`    | View current settings        |
| `/toggle`    | Enable/disable notifications |
| `/help`      | Command reference            |

## Implementation Highlights

### Security

- Comprehensive user input validation
- reCAPTCHA spam protection
- Secure token storage in environment variables

### Monitoring

- Detailed action logging
- Performance metrics
- Web interface for status monitoring

### UX/UI

- Intuitive command structure
- Friendly Russian language messages
- Quick setup in just a few steps

### Performance

- Asynchronous request processing
- Weather data caching
- Optimized API calls

## Source Code & Project Structure

Open source project on GitHub: [github.com/avlihachev/garden_bot](https://github.com/avlihachev/garden_bot)

### Project Structure

```
garden_bot/
├── src/
│   ├── bot/              # Telegram bot logic
│   ├── services/         # Business logic services
│   ├── config.ts         # Configuration
│   └── index.ts          # Application entry point
├── public/               # Static files
├── Dockerfile           # Containerization
├── fly.toml            # Fly.io configuration
└── package.json        # Node.js dependencies
```

## Results & Metrics

### Experience Gained

- **TypeScript** — strict typing for reliability
- **Telegram Bot API** — messenger platform integration
- **Cloud technologies** — Fly.io deployment
- **API integrations** — external service integration
- **Monitoring** — dashboard creation for tracking

## Technical Achievements

### Reliability

- Automatic error recovery
- Graceful shutdown handling
- Data validation at all levels

### Scalability

- Modular architecture
- Horizontal scaling readiness
- Efficient resource utilization

### Code Quality

- 100% TypeScript coverage
- Linting and formatting
- Well-documented codebase

## Technical Skills Showcased:

- **Full-stack development** with modern JavaScript/TypeScript
- **API integration** and real-time data processing
- **Bot development** and conversational UI design
- **Cloud deployment** and DevOps practices
- **Database design** and user data management

---

**Garden Weather Bot** demonstrates a modern approach to solving everyday problems through automation and intelligent notifications. The project showcases full-stack development skills from concept to production deployment.

## Get Custom Solutions

Interested in building your own Telegram bot or automation system? I create custom solutions for businesses and individuals.

---

_Garden Bot is actively maintained and continuously improved based on user feedback. The complete source code is available on GitHub, and I'm always happy to discuss technical implementation details._
