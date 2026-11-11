# 0016. Spaces are kept apart, and people control them

Date: 2026-11-11 · Status: Accepted

## Context

Spaces (Personal, a business, a household and others) each have their own encrypted database. People need to change a space's settings, fix transactions logged in the wrong space, and clear or delete a space's data, with a way back if they change their mind.

## Decision

Data is never shared between spaces: nothing is read or copied from one space into another. Transactions are never edited; they can be recategorised or moved to another space, which takes them out of the first one. Each space has its own settings, and any space but Personal can change kind after a warning of what comes in and goes away; a business can switch between products, services or both. Clear data removes a space's records but keeps its name, accounts, categories and settings; Delete space removes the space. Both can keep a locked copy on the phone for 30 days, at the person's choice, and can also delete that space's online backups only when the person ticks it. Personal cannot be deleted on its own: Delete all my data removes every space and everything the app keeps, after the app lock, at once.

## Consequences

- A restore of a kept copy goes back into its own space only.
- The vault and Google Drive locations gain a delete call; backups in a phone folder are left for the person to manage.
- Deleting everything also deletes the database key, so nothing left behind could be opened.
