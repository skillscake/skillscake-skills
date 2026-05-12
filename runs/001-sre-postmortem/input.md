We get about 6 major incidents a year. Make a skill that pulls the incident channel `#inc-YYYYMMDD-shortname` via the Slack API and publishes a postmortem doc to Confluence at `SRE/Postmortems/{YYYYMMDD-shortname}`. env vars `SLACK_BOT_TOKEN`, `CONFLUENCE_DOMAIN`, `CONFLUENCE_API_TOKEN`

Sections in order: summary, customer impact (services + $), timeline (PT), root cause, contributing factors, 5 whys, what went well, what went wrong, action items. Use sre best practices. Action items go to Linear under project INC. use `LINEAR_API_KEY`

Blameless. Roles (primary, secondary, IC), not names

Link `SRE/Runbooks/{service}`

Handle missing severity, partial timelines, and channel chitchat. Don't reveal personal things that appear in chat

The agent needs to reach all the tools right every time, understand, and post