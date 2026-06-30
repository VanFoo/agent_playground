---
name: family-vacation-planner
description: Interactively plan a vacation for a family traveling with a child or children, end to end - interview the user about the trip, research real options on the web, progressively narrow down through follow-up rounds, and deliver a final list of at least 5 specific places to stay (named hotels, resorts, or family clubs - not just cities, regions, or general areas). Use this whenever the user mentions planning a trip, vacation, holiday, or getaway that involves a kid, child, son, daughter, or family, even if they don't explicitly ask for "vacation planning" - e.g. "where should we take our 6 year old this summer" or "looking for a family-friendly resort in Spain" or "need a hotel that works for a toddler."
---

## Why this skill exists

Generic travel research tends to surface destinations or regions ("the Algarve is great for families") rather than something the user can actually book ("Pine Cliffs Resort, Albufeira"). Parents planning with a child also have constraints that change which properties are even viable - kids' club age cutoffs, pool depth, room layouts - that get lost if you skip straight to a search. This skill exists to force the slower path: understand the family's actual constraints first, then research toward specific, bookable properties, narrowing in stages rather than dumping a huge list at once.

## Step 1: Interview the family

Before searching anything, get real answers on all of the following. A natural back-and-forth is fine - you don't need to dump this as one long questionnaire - but don't move on to research until you actually have an answer for each, not an assumption:

- **Children**: how many, and each child's age. This drives which kids' club age bands, pool/water-park suitability, and activity options even make sense.
- **Budget and dates**: a rough total budget (or per-night, whichever the user finds easier) and travel dates or duration.
- **Departure point and constraints**: where they're traveling from, how far/long a flight or drive they're willing to do with a child, and anything like passport/visa status for the child if international travel comes up.
- **Interests and dealbreakers**: beach vs. mountains vs. theme parks, all-inclusive vs. self-catering, wanting a kids' club vs. wanting a quieter property, dietary or accessibility needs, etc.

Vague answers ("flexible," "anywhere," "not sure") are a fine answer - just make sure you actually asked rather than guessed.

## Step 2: Narrow in rounds, not one big search

Don't jump straight from the interview to a final list of specific hotels. Research and narrow progressively, checking in with the user between rounds:

1. **Region/style round**: search broadly based on the interview answers and present a handful of candidate *categories* - e.g. regions, trip styles, or property types ("Caribbean all-inclusive resorts," "domestic Center Parcs-style clubs," "Greek island family hotels"). Ask which direction appeals before going deeper. This avoids doing expensive, specific research in a direction the user doesn't actually want.
2. **Style/budget round**: once a direction is chosen, research further within it and present a smaller, more specific batch of options (still not final picks) - narrowing on resort style, price tier, or sub-region - and check in again.
3. **Final round**: with that narrowed-down direction confirmed, do the deep research needed to find specific, real, bookable properties.

Use real web search at each round, not assumptions from general knowledge - the goal is options that actually exist and are findable/bookable today, not plausible-sounding guesses. Specific properties, kids' club age policies, and prices change too often to rely on memory.

## Step 3: Deliver the final list

The final output must be **at least 5 specific places to stay** - named hotels, resorts, or family clubs, not cities, regions, or "areas to consider." For each one, include:

- **Name, location, and a link** to the property (official site or a booking/review page)
- **Approximate price range** for the trip as described (or per night, matching whichever the user used during the interview)
- **Kid-specific amenities** - kids' club age ranges, babysitting availability, pool/play areas, family room configurations, etc.
- **Why it fits this family** - tie the pick back to what was actually said in the interview (the kids' ages, the stated interests, the budget), not a generic description you'd give anyone. If a pick is a reasonable stretch on one criterion (e.g. slightly over budget, but unusually good for a toddler), say so rather than glossing over it.

If verified information about something (e.g. an exact price or an age cutoff) isn't available from the research, say so rather than stating it as fact.

After the per-property write-ups, always close with a **comparison table** that puts all the picks side by side - one row per property, with columns for whatever this family's actual decision criteria are (e.g. price/budget fit, the specific kid-amenity that mattered most, location or flight time, price confidence). A parent comparing options wants to scan one table, not re-read five paragraphs to reconstruct the comparison themselves. End the table with a one- or two-line bottom-line recommendation naming which pick(s) you'd lead with and why, given the table.
