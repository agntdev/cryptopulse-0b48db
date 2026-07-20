# CryptoPulse — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Personal Telegram bot for private cryptocurrency price tracking with customizable price threshold and percent change alerts. Users maintain private watchlists and notification settings while the owner receives anonymized usage analytics.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- private cryptocurrency traders
- crypto investors
- Telegram bot users seeking price alerts

## Success criteria

- Users receive accurate price alerts based on configured thresholds/percent changes
- Owner accesses aggregated analytics without PII exposure
- Persistent user-specific watchlists survive bot restarts

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Initialize user profile and launch onboarding flow
- **/price** (command, actor: user, command: /price) — Show current price of specified ticker or full watchlist summary
- **Add to Watchlist** (button, actor: user, callback: watchlist:add) — Display quick-add buttons for popular coins and custom ticker input
- **Manage Alerts** (button, actor: user, callback: alerts:manage) — Configure price threshold and percent change alerts for watchlist items

## Flows

### Onboarding
_Trigger:_ /start

1. Request time zone and morning digest preference
2. Set default quiet hours (22:00-08:00)
3. Display initial watchlist management options

_Data touched:_ user profile

### Price Alert Creation
_Trigger:_ watchlist:add confirmation

1. Select alert type (threshold/percent change)
2. Configure parameters (price level, percent, time window)
3. Set cooldown period and hysteresis

_Data touched:_ watchlist entry, price alert types

### Morning Digest
_Trigger:_ scheduled local time

1. Check quiet hours status
2. Compile overnight price changes
3. Send summary if outside quiet hours

_Data touched:_ user profile, notification records

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User Profile** _(retention: persistent)_ — User preferences and settings
  - fields: Telegram ID, locale, time zone, quiet hours, digest time, subscription limits
- **Watchlist Entry** _(retention: persistent)_ — Tracked cryptocurrency with alert settings
  - fields: ticker, display name, alert types, last notified, cooldown status
- **Notification Record** _(retention: persistent)_ — Sent alert history for analytics
  - fields: timestamp, user ID (private), ticker, alert type, price data
- **Owner Analytics** _(retention: persistent)_ — Aggregated usage statistics
  - fields: total users, active users, alert trigger counts

## Integrations

- **Telegram** (required) — Bot API messaging and notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- /stats command for aggregated metrics
- Configure default fiat currency (USD)
- Adjust per-user limits (30 tickers default)

## Notifications

- Direct price alerts with price change details
- Morning digest of overnight movements
- Owner aggregate analytics reports

## Permissions & privacy

- All user data is private by design
- Price data stored with anonymized user identifiers
- Notification logs retained for analytics (90-day window)

## Edge cases

- Unknown/invalid ticker handling with suggestions
- Price source failures with retry logic
- Quiet hours overlapping with alert triggers

## Required tests

- End-to-end alert triggering with cooldown/hysteresis validation
- Privacy compliance verification
- Aggregate analytics accuracy test

## Assumptions

- Default fiat is USD
- 1-hour cooldown for alerts
- Morning digest is optional and quiet-hours aware
