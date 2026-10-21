---
description: Research the fundamental, first-principles ideas behind a topic and the best sources that explain them
argument-hint: <topic>
---

Find the core ideas behind this topic, and the best writing that explains each one:

$ARGUMENTS

The goal is understanding, not coverage. A good result is a short list of concepts that, once understood, make the rest of the topic obvious and let the user act faster. Each one is paired with the source that explains it most clearly. Only include ideas that are true and still hold today. An idea that was right for older tools or a different scale will hold the user back if they learn it as fact.

## Think like a physicist

Approach the topic the way a physicist approaches nature. Look for the smallest set of principles from which everything else can be derived, and hold each one to a high standard of rigor.

The target is a **spanning set** for the topic: a few independent ideas that together cover the whole space, so anything in the topic can be described as a combination of them. With enough research this set tends to emerge on its own, because the same few ideas keep showing up under different names. Pete Hodgson's feature toggles article is the model. Once you see that toggles vary along two independent axes, how long they live and how dynamic they are, the four kinds of toggle fall out, and every real toggle sits somewhere in that grid.

- **Find the principle under the instances.** Many things that look different are the same idea. Client and server is the shape behind a web server, interprocess communication, a database connection, and a language server: one side waits for requests, the other sends them, and they agree on a protocol. Learn the principle once and every instance becomes a variation. Prefer the principle over any one instance of it.
- **Look for the axes.** Ask which independent dimensions things in the topic vary along. A good set of categories comes from crossing those dimensions, so it covers every case and the categories don't overlap. If two items always change together, they're one dimension, not two.
- **Test generality.** A real core principle explains things in more than one place. If an idea only explains the topic it came from, it's probably a detail, not a principle.
- **Derive, don't list.** For each principle, show what follows from it. If a common practice in the topic can be derived from a principle already on the list, it doesn't need its own entry.
- **Know where it stops.** Every principle rests on assumptions, the way Newton's laws hold until speeds get close to light's. Say what each idea assumes and the conditions where it breaks.
- **Keep definitions, principles, and consequences separate.** A definition names something. A principle is a claim about how things behave. A consequence follows from principles. Don't present a consequence as if it were fundamental.
- **Prefer what can be checked.** Favor ideas stated precisely enough to be tested or measured over ones that sound wise but predict nothing.

## What makes a source worth saving

Check these while skimming a page:
1. **The author did the thing.** They designed, built, named, or maintain it, or read its source themselves. Look for "we built", "our v1", "I read the code" in place of "teams should". A well-sourced reference that many people check, like a good Wikipedia article, also qualifies.
2. **It explains how it works, not just what it is.** There's a mechanism (a hash in the path, a token in a kernel table), not a feature list or usage steps sold as "how it works".
3. **It names and defines its ideas precisely.** You come away with terms you can search for and reason with, often ones the author introduced.
4. **Its examples are real and checkable.** Code as text, actual schemas, numbers, dates, named incidents and systems, not "for example, Netflix uses this".
5. **It takes a side.** "I recommend X", "Y is preferable", with the reason. Weak pages end with "it depends".
6. **It states costs and limits.** What breaks, what isn't solved, what the approach costs, and what the author got wrong.
7. **It shows the path, not just the answer.** Earlier versions, failed attempts, rejected alternatives, changes of mind.
8. **It goes deep on a narrow scope.** It says what it leaves out. Short is fine: a one-page site can be the best source if it's where the idea started and every paragraph adds something.
9. **Its details hold up.** Links point where they claim, the page doesn't contradict itself, and good writing elsewhere cites it as the origin.

Being a company blog is fine. Cursor's and Flightcontrol's posts qualify because the engineers explain their real system in depth. What matters is whether the post explains or sells.

Signs of a page to skip:
- no author or date, a team byline, an "AI-assisted" note, or a product pitch inside the advice
- a "Top N best practices" page in one repeated template
- praise or filler words ("powerful", "robust", "seamless", "essential") where an explanation should be
- big company names with no detail, or claims with no mechanism, numbers, or sources
- advice that fits any topic ("write tests", "stay informed"), or "learning curve" as its only caveat
- a definition that blurs the idea into a nearby, more familiar one
- a conclusion that repeats the intro, or a restatement of the original source with nothing added

The user picked these as excellent. They show the standard, not topics or formats to copy:
- https://martinfowler.com/articles/feature-toggles.html
- https://www.hyrumslaw.com/
- https://research.swtch.com/vgo-import
- https://nesbitt.io/2026/01/02/how-dependabot-actually-works.html
- https://eelcovisser.org/publications/2004/DolstraVJ04.pdf
- https://cursor.com/blog/git-at-any-scale

## Process

1. **Break the topic down.** Before searching, ask what the topic reduces to: the few principles everything else in it follows from, and which broader principles it's an instance of (the way IPC is an instance of client and server). Group the candidates into 2–5 sub-areas.
2. **Search in parallel.** Spawn one `general-purpose` subagent per sub-area, all in one message. Brief each one with its sub-area and the source checks above. Tell it to find the concepts that matter most and the single best source for each. It must fetch every page to confirm the page says what it claims, and check that the idea still holds today. Ask it to look at niche personal blogs, original papers, and talks, not just the first page of search results. It should return at most ~8 items as `concept — URL — author/venue/date — the core idea in one sentence — what it assumes and where it stops holding — other places the same principle shows up — why this source is the best one`.
3. **Check that the set spans the topic.** Take a few concrete cases from the topic, including odd ones, and describe each in terms of the ideas found so far. Something that can't be described means an idea is missing, so search for it. When new sources stop adding new ideas and only rename ones you have, the set is complete.
4. **Curate.** Merge the results and drop duplicates. Cut anything peripheral, secondhand, outdated, or below the source bar. Cut anything that is just a consequence of another item on the list, and fold it into that item instead. When two sources cover the same idea, keep the clearer one. Fewer, deeper items beat a long list.
5. **Check your picks.** Every link was fetched and works. Every one-line idea is the actual idea from that source, not a paraphrase of its title.

## Output

Group by area, nesting deeper or related items under their parent:

```
<Area>
- <Concept> (<url>) — <the core idea in one sentence>
    - <Concept that follows from it> (<url>) — <core idea>
```

Order the list from most fundamental to most derived. If the topic has clear independent axes, name them first and show the categories they produce, the way the toggles article does. After the list, add a few lines covering:
- the first principle the whole topic rests on, if there is one
- where else that principle shows up outside this topic
- the main assumptions the list relies on, and when they'd stop holding
