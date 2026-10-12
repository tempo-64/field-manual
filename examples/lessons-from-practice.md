# Lessons from Practice

Real events from projects built with this method. Identifying details are removed. Each lesson names the rule it proved.

## A release blocked by its own evidence gate

A release candidate had green focused tests. The release gate compared the recorded evidence hash to the exact source revision and found the evidence came from older source. The release stopped until the evidence was regenerated.

Rule proved: bind proof to the exact revision. Focused green is never a full green release.

## Three blinded attempts, zero survivors

A product experiment built three independent approaches to the same creative problem. Reviewers judged the outputs without knowing which approach produced which result. All three failed the stated quality bar. The direction stopped instead of shipping the least bad result.

Rule proved: define the stop rule before seeing results, and let it fire.

## The review that hunted false-clear states

Before a release, independent review of a context-checking product looked specifically for ways the system could report a clean state without evidence. It found several: incomplete API results rendered as clear, malformed responses rendered as clear, and cancelled checks that disappeared. Every one became an explicit state and a regression test.

Rule proved: missing evidence is never success, and a reviewer hunting for false-clear paths finds defects the builder cannot see.

## Nine built, zero good

A generation pipeline produced nine of ten planned creative artifacts. Structural review of the outputs read as strong progress. Then the human experienced the artifacts in their real medium and judged the words unusable, and a reduced retest confirmed the gap. The architect had judged quality from text descriptions and structural properties. That judgment was wrong.

Rules proved: judge creative output in its real medium, and treat an assistant's quality assessment as one input to the human's decision.

## A review of the wrong revision

A code review ran against a stale local revision instead of the current head. The findings were internally consistent and useless. The review was rolled back and repeated against the correct revision.

Rule proved: confirm the exact revision before reviewing, auditing, or building.

## The account that flipped

A push failed with a misleading permission error because the active hosting account had switched identities. The durable fix was procedural: bind the identity check into the same invocation as the operation, and add a pre-push identity guard.

Rule proved: identity is part of release state. Check it explicitly instead of assuming it.
