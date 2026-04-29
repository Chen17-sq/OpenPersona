# Extension Protocol

OpenPersona must stay sharp. The **only** mechanism for adding new domains is namespaced fact keys.

## The rule

> **A new "domain" is a new key prefix in `facts.key`. Never a new table.**

## How to add a domain (the whole process)

1. Pick a prefix: `health:` / `read:` / `work:` / `<your_namespace>:`
2. Start writing facts with the new prefix:
   ```
   facts(person_id='me', key='health:weight_goal', value='-5kg')
   facts(person_id='me', key='health:run_freq',  value='3/week')
   ```
3. The UI **automatically** groups facts by prefix into a section labeled `Capitalize(prefix)`.
4. If the new domain needs reminders (e.g. health goals → check-in reminders), use existing `reminders` table with one of the 5 `trigger_kind` values.

That's it. **Zero code changes** for the data layer.

## When you would NOT add a domain

If the proposed feature is one of:

- A new entity that isn't a person, fact, promise, event, reminder, or source → **stop**. Probably scope creep.
- A specialized UI with its own bespoke data shape (e.g., a Pomodoro timer with intervals) → **stop**. That's a different product.
- Generic todos that don't sediment into a Persona ("buy milk") → **stop**. Not OpenPersona's scope.

## Sharpness invariant

Before adding any domain, ask:

> Is the unit of meaning here `(person × time × fact_or_action)`, where `person` may be `me`, AND does it change the Persona over time?

If yes → new key prefix. If no → it doesn't belong here.

## Examples

| Proposed | Verdict | How |
|----------|---------|-----|
| Track my weight over time | ✅ in | `health:weight` facts; UI auto-groups |
| Track current book | ✅ in | `read:current` fact |
| Tag people by relationship | ✅ in | `tag:colleague` |
| Buy milk reminder | ❌ out | not Persona-shaped; use Reminders.app |
| Weekly journal entries | ❌ out (v0/v1) | needs free-text design; revisit v2 |
| Pomodoro timer with sessions | ❌ out | bespoke entity, not fact-shaped |
| Birthday reminder | ✅ already in | `date:birthday` fact auto-creates recurring_yearly reminder |
