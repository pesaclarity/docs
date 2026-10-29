# 0010. Dates in the user's timezone

Date: 2026-10-29 · Status: Accepted

## Context

The app worked out every calendar day in Nairobi time (UTC+3), which suited Kenya only. With a base currency for any country (0009), people elsewhere saw transactions near midnight on the wrong day, and month totals shifted. Profile gained a timezone setting, but day starts followed it while dates stayed in Nairobi time, so the two disagreed outside UTC+3.

## Decision

Every date, day start, month and export timestamp uses one timezone: the one chosen in Profile › Timezone, or the phone's own when none is chosen. The offset is fixed while it applies; the app does not apply daylight-saving rules. Tests pin East Africa Time so results do not depend on the machine running them.

## Consequences

- Kenyan users see no change; others get their own midnights.
- Changing the timezone re-files transactions near midnight onto the other day; totals move with them.
- Zones with daylight saving (such as CET) are an hour off for part of the year until the app reads the device's rules per date.
