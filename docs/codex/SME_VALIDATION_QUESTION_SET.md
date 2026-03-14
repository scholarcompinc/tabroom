# SME Validation Question Set

## Purpose

This document is the first-pass subject-matter-expert validation set derived from the discovery corpus.

Its purpose is to identify the questions that source analysis alone is unlikely to answer confidently enough for architecture and release planning.

This is not an interview script for one meeting. It is a working validation backlog.

## How To Use This Document

These questions are best answered by a mix of:

- experienced tournament directors
- tab room operators
- coaches
- judges
- affiliation/program stakeholders

The goal is not to get theoretical answers. The goal is to learn:

- what is used,
- what breaks most often,
- what users will refuse to lose,
- what can be simplified safely.

## Section 1: Tournament Director / Tab Room Operator Questions

## Core Setup and Operations

1. Which setup areas are essential before a tournament is runnable, and which ones are often left at defaults?
2. Which settings do you actively change almost every tournament?
3. Which settings are confusing but still operationally necessary?
4. Which setup tasks consume the most staff time before the tournament starts?

## Pairing and Assignment

5. Which pairing modes are used most often in practice for debate, speech, and congress?
6. Which automatic pairing behaviors do you trust, and which ones do you routinely override?
7. What are the most common reasons you manually adjust a paired round?
8. Which judge assignment problems happen most frequently?
9. Which room assignment problems happen most frequently?

## Recovery and Disaster Handling

10. Which “disaster” situations actually occur during live tournaments most often?
11. When something goes wrong mid-round, which repair actions do you need immediately?
12. Which operator screens do you keep open constantly during live operations?
13. Which recovery workflows are must-have in the first release for you to trust the platform?

## Results and Breaks

14. Which tiebreak/result configurations are common defaults versus rare exceptions?
15. How often do you need to regenerate or correct results after initial computation?
16. Which break-generation edge cases cause the most anxiety or manual intervention?

## Reporting and Print

17. Which printed artifacts do you still rely on in real tournaments?
18. If print support were limited in the first release, which outputs are absolutely non-negotiable?
19. Which reports matter during the tournament versus after it ends?

## Section 2: Coach Questions

## Registration and Change Management

20. Which registration workflows matter most for coach usability?
21. Which registration or late-change scenarios are most painful today?
22. Which visibility features do coaches rely on most after registration closes?

## Judge Preferences and Strikes

23. How important are paradigms, prefs, and strikes to your willingness to adopt a new system?
24. Which preference workflows are truly necessary versus “nice to have”?
25. Are there parts of current prefs/strikes behavior that are more complex than coaches actually need?

## Public and Self-Service Views

26. Which public or coach-facing pages are used most heavily during a tournament day?
27. How much does familiarity of terminology and workflow order matter versus exact page layout?

## Section 3: Judge Questions

## Ballot Experience

28. What makes ballot entry reliable or unreliable in real venues?
29. Which ballot actions must work smoothly on a phone under bad connectivity?
30. Which kinds of ballot confusion happen most often today?

## Paradigms and Identity

31. How important are judge paradigms and judge profile visibility?
32. Which judge self-service actions are important enough for first release?

## Section 4: Specialized Stakeholder Questions

## District / NSDA / National Support

33. Which district or national workflows are truly essential versus “annual pain but rare”?
34. Which district/national documents or exports are non-negotiable?
35. Which qualification or alternate rules are most likely to cause real damage if wrong?

## NCFL / NAUDL / Other Program Modes

36. Which specialized organization workflows would block adoption in your target market?
37. Which external exports or syncs are contractual or mission-critical?

## Section 5: Product Strategy Questions

38. Is congress required in the first operating release, or can debate + speech go first?
39. Are paradigms required for initial market trust?
40. Are financial workflows required for launch, or can billing be simplified initially?
41. Is historical import from tabroom.com expected at launch?
42. Which affiliation-specific modes actually matter for the first commercial targets?

## Section 6: Validation Questions About Simplification

43. Which parts of the current system feel over-configured?
44. Which options or settings could be safely replaced with strong defaults?
45. Which reports or exports could disappear without operational harm?
46. Which workflows should be redesigned rather than copied?

## Section 7: Validation Questions About Trust

47. What are the top three things that would make you trust a rebuilt system to run a live tournament?
48. What are the top three failures that would make you reject it immediately?
49. Which operator-audit features are required for trust?
50. How much manual override power must the operator have?

## Most Important Questions To Answer Before ScholarComp Mapping

If validation time is limited, these questions appear most important:

1. Which event formats must be in Release 1?
2. Which print/report artifacts are mandatory for live use?
3. Which recovery workflows are must-have on day one?
4. Which settings are truly used often?
5. Which preference/strike/paradigm features are essential for trust?
6. Which specialized affiliation workflows are commercially necessary early?
7. How much degraded-network tolerance is required for ballots?

## Suggested Next Use Of This Document

Use this question set to produce:

- a validation interview plan,
- a resolved-scope appendix,
- a simplification-safe list,
- and a set of explicit “must preserve” workflows before architecture work starts in ScholarComp.
