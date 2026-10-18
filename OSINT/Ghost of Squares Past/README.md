# Ghost of Squares Past 

**Challenge:** Ghost of Squares Past  
**Category:** OSINT  
**Points:** 150  
**Author:** @RP-01?  
**Event:** DEADFACE CTF 2026  
**Location:** Savannah, Georgia, USA

## Challenge Description

DEADFACE has been using the historic squares of Savannah, Georgia, to conceal a coded message.

We are provided with five clues:
- Point 1: A historical riddle.
- Point 2: A photograph.
- Point 3: A historical riddle.
- Point 4: A photograph.
- Point 5: A final riddle.

The first four locations represent the four cardinal directions: North, East, South, and West. The fifth location represents their center.

The objective is to identify the five locations, retrieve the establishment years of the first four squares, and find the surname engraved on the fifth location's monument.

**Flag Format:**
`deadface{YEAR-YEAR-YEAR-YEAR-SURNAME}`

## Investigation

### Point 1 — Johnson Square (North)

The first riddle contains several historical references:

> First of them all, where the city began, A banker's green heart in Oglethorpe's plan. A Quaker from Rhode Island sleeps under stone here.

Key observations:

- "First of them all" suggests Savannah's oldest square.
- James Oglethorpe established Savannah's original town plan.
- The Quaker reference points toward Nathanael Greene, who was born in Rhode Island and is commemorated by a monument in Johnson Square.

After researching Savannah's historic squares, the location was identified as **Johnson Square**.

**Established:** 1733

### Point 2 — East (Photograph)

The second clue provides a photograph rather than a written riddle.

The task is to identify the historic square shown in the photograph and match it with its position in the compass arrangement.

Based on the proposed compass mapping, the candidate is **Oglethorpe Square**.

**Proposed establishment year:** 1742

### Point 3 — Chippewa Square (South)

The third riddle contains multiple historical and cultural references:

> Just a ways away a battle of a northern frontier Gave its name to this ground eighteen-twelve drawing near. Where a feather once drifted and a founder stands tall, Mark the year it was laid out, the newest of all.

Key observations:

- The Battle of Chippewa took place in 1814.
- The falling feather references the famous opening scene of *Forrest Gump*, filmed at Chippewa Square.
- The founder reference points to James Oglethorpe's statue.

These clues identify **Chippewa Square**.

**Established:** 1815

### Point 4 — West (Photograph)

The fourth clue is another photograph.

Using the compass arrangement, the proposed identification is **Telfair Square**, historically known as St. James Square.

**Proposed establishment year:** 1733

### Point 5 — Wright Square (Center)

The final riddle states:

> Four points now marked, good work, you did your best Their meeting point holds the key to this test. Carry their years in that order, and seek the lone name Carved in the stone where their center lays claim.

The central location was identified as **Wright Square**.

The square contains a monument dedicated to William Washington Gordon, associated with the founding of the Central of Georgia Railroad.

The surname required for the final part of the flag is:

**Gordon**

## Findings Summary

| Point | Direction | Proposed Location | Year / Name |
|---|---|---|---|
| 1 | North | Johnson Square | 1733 |
| 2 | East | Oglethorpe Square | 1742 |
| 3 | South | Chippewa Square | 1815 |
| 4 | West | Telfair Square | 1733 |
| 5 | Center | Wright Square | Gordon |

## Flag Candidate

`deadface{1733-1742-1815-1733-gordon}`

**Verification status:** The initial submission using a different Point 2 year was rejected. The revised mapping above is a working candidate, not a confirmed solution. Points 2 and 4 should be verified directly against the original photographs before claiming the challenge is solved.

## Key Takeaways

This challenge demonstrates how historical research and geographical reasoning can be combined to solve OSINT investigations.

Useful techniques included:

- Historical research into Savannah's city planning.
- Identifying monuments and historical figures.
- Connecting cultural references to real-world locations.
- Using geographical relationships to reconstruct a compass pattern.
- Verifying establishment dates through historical records.
