# Crypto Price Watcher — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Personal crypto price tracking bot with customizable alerts for price thresholds and percentage changes, private user lists, and admin analytics. Users receive notifications in private chats, with silent hours and morning summaries. Admin gets aggregated usage metrics.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- private crypto price watchers
- bot owner/administrator

## Success criteria

- users receive accurate price alerts in private chats
- admin gets daily aggregated metrics
- bot handles price feed errors without spamming users

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Initialize user account and show main menu
- **/help** (command, actor: user, command: /help) — Show quick instructions
- **/price** (command, actor: user, command: /price <ticker|all>) — Check current price of a specific coin or all tracked coins
- **/list** (command, actor: user, command: /list) — Show private tracking list with inline controls
- **Bitcoin** (button, actor: user, callback: add_ticker:BTC) — Add BTC to tracking list
- **Ethereum** (button, actor: user, callback: add_ticker:ETH) — Add ETH to tracking list
- **Toncoin** (button, actor: user, callback: add_ticker:TON) — Add TON to tracking list
- **Add other ticker** (button, actor: user, callback: add_custom_ticker) — Prompt user to enter a custom ticker symbol
- **/stats** (command, actor: admin, command: /stats) — Show aggregated user metrics to admin

## Flows

### Add coin to tracking
_Trigger:_ inline_button:add_ticker

1. Show coin confirmation
2. Add to tracking list
3. Show updated list

_Data touched:_ User, Watch

### Add custom ticker
_Trigger:_ inline_button:add_custom_ticker

1. Prompt for ticker symbol
2. Validate ticker
3. Add to tracking list
4. Show updated list

_Data touched:_ User, Watch

### Set price threshold alert
_Trigger:_ inline_button:add_threshold

1. Prompt for target price
2. Prompt for direction (above/below)
3. Create threshold alert
4. Show updated list

_Data touched:_ Watch

### Set percentage change alert
_Trigger:_ inline_button:add_percentage

1. Prompt for percentage
2. Prompt for interval
3. Prompt for direction (up/down/any)
4. Create percentage alert
5. Show updated list

_Data touched:_ Watch

### Morning summary
_Trigger:_ scheduled_daily

1. Fetch 24h price changes
2. Format summary
3. Send to users who enabled it

_Data touched:_ User, Watch, Notification

### Price alert check
_Trigger:_ scheduled_interval

1. Fetch current prices
2. Check all active alerts
3. Send notifications if triggered
4. Update alert cooldowns

_Data touched:_ Watch, Notification

### Admin stats
_Trigger:_ command:/stats

1. Aggregate user metrics
2. Format stats
3. Send to admin

_Data touched:_ User, Watch, Notification

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user account with preferences and tracking list
  - fields: telegram_id, timezone, silent_hours_start, silent_hours_end, morning_summary_time, is_admin
- **Watch** _(retention: persistent)_ — Tracked coin with alert rules
  - fields: user_id, ticker, alert_type, target_price, percentage, interval, direction, last_triggered, cooldown_until
- **Notification** _(retention: persistent)_ — Record of price alerts sent to users
  - fields: watch_id, timestamp, old_price, new_price, percentage_change, alert_type
- **PriceFeed** _(retention: persistent)_ — External price source configuration
  - fields: source_url, last_check, status
- **AdminMetric** _(retention: persistent)_ — Aggregated usage statistics
  - fields: total_users, active_alerts, top_coins, top_triggers

## Integrations

- **Telegram** (required) — Bot API messaging
- **PriceFeed** (required) — External USD price data source
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View aggregated user metrics
- Configure price feed source
- Adjust default cooldown periods
- Set admin user access

## Notifications

- Price threshold alerts
- Percentage change alerts
- Morning summaries
- Price feed error notifications
- Admin metrics updates

## Permissions & privacy

- Private tracking lists by default
- Admin sees only aggregated metrics
- Price data stored securely
- User data not shared publicly

## Edge cases

- Price feed outages
- User input typos in ticker symbols
- Overlapping alert cooldowns
- Time zone daylight saving changes
- Large number of simultaneous alerts

## Required tests

- End-to-end price alert flow with cooldown
- Morning summary delivery test
- Admin metrics aggregation test
- Error handling for price feed failures
- User privacy validation

## Assumptions

- USD is the only displayed currency
- Default cooldown is 6 hours
- Default silent hours are 23:00-08:00
- Morning summaries are optional
- Price feed has automatic retries
