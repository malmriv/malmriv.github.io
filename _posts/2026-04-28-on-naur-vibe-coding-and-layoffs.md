---
title: "On Naur, vibe coding, and the people you can't replace with a context window"
date: 2026-04-28
permalink: /posts/2026/04/on-naur-vibe-coding-and-layoffs/
tags:
  - software engineering
  - AI
  - consulting
  - programming
  - personal
---

I read Peter Naur's *Programming as Theory Building* this week. It is from 1985, and it argues something that I find more relevant now than it has been in decades: that what programmers build is not really code, but a *theory* of the problem they are solving. The code is a side effect. The theory lives in the heads of the people who built the system, and when those people leave, the theory leaves with them. Documentation does not save you. The code does not save you. You can read every line of a system written by someone else and still not have what they had.

If you have not read the paper, [it is not long and worth your time](https://blog.almag.ro/files/naur1985programming.pdf).

This might seem like a weird thing to fixate on in 2026, but it's actually more important now than at any other point in the last 40 years. The reason I want to write about it now is that we seem to be running a real-world experiment on Naur's claim, and the results are not going to be... palatable.

## The layoffs are not what they look like

[Microsoft, Meta](https://www.theguardian.com/technology/2026/apr/23/meta-microsoft-tech-ai-layoffs), [Capgemini](https://oecd.ai/en/incidents/2026-04-10-67cc), [Salesforce, Oracle](https://www.newsweek.com/all-tech-giants-announcing-sweeping-layoffs-2026-11872935). The reasoning, when there is any, is some version of: AI is making us more productive, so we need fewer people. Sometimes it is more honest and the reasoning is "we want our share price to go up." Either way the result is the same: the people who built the systems are leaving, and the assumption is that what they leave behind, the code and the documentation and the Confluence pages and the Jira tickets, is enough for someone else (or some*thing* else) to pick up where they left off.

Naur would have laughed at this. He spent the paper explaining that this is exactly the thing you cannot do. You can hand someone a complete codebase and complete documentation, and they will still not be able to maintain it the way the original team could. They will misjudge which changes are safe and which are dangerous, and they will probably reinvent things the original team already tried and discarded. Feature requests that break invariants nobody ever documented will get accepted because, to the people who built the system, those invariants were too obvious to write down.

This was true in 1985. It is true now. The only thing that has changed is that there is a new actor in the room (the LLM) and a new belief about what that actor can do. In the following sections, I will try to elaborate on why LLMs will not save us from bad decisions. 

## What an LLM actually loads

Here is the optimistic version of the story, the one I think most executives have in their heads even if they have never said it out loud: the original team built a system and wrote some documentation. They are gone now, but a sufficiently capable LLM can read the entire codebase and the entire documentation set in a few seconds and act as a stand-in for the lost theory. The remaining engineers can ask the LLM questions and the LLM will answer them. Productivity is preserved. Cost is reduced. Everyone who wears a tie wins.

This is wrong, and the way it is wrong matters.

What an LLM loads when you point it at a codebase is the artifact, the code itself. The artifact is the residue of the theory. It is what is left after the theory has done its work. Reading the artifact gives you a model of the artifact, which is useful for some things, but it is not the same as having the model that *produced* the artifact. The producing model is the one that knows which feature requests are reasonable and which ones are landmines. It is the one that knows why a particular function exists in a particular place, even though it could have lived somewhere else, because three years ago a customer reported a bug that nobody else remembers. The LLM does not have access to this. Nobody does, except the people who left.

There is a famous example of this gap, and it predates LLMs by twenty years. When the source code for Quake III Arena was released in 2005, programmers found a function called `Q_rsqrt` that computed an inverse square root using a sequence of operations that looked like nonsense. Inverse square root are used everywhere in ray-tracing models because that's what you use to normalise vectors, compute angles and countless other operations in vector algebra. The relevant lines of the function were these:

```c
i  = * ( long * ) &y; // evil floating point bit level hacking
i  = 0x5f3759df - ( i >> 1 );  // what the fuck?
y  = * ( float * ) &i;
```

The comments are from the original source, and they are the entire documentation. The code reinterprets a float's bit pattern as an integer, subtracts it from a *magic constant* (yes, that's what it was called) and reinterprets the result as a float again, which happens to be an extremely accurate approximation of `1/√{input}`. It works because of a clever exploitation of how [IEEE 754 floating-point numbers](https://en.wikipedia.org/wiki/IEEE_754) are laid out in memory, but knowing *that* tells you nothing about *why* the constant is `0x5f3759df` specifically, or who came up with it, or how. The code was complete, commented and in full use, but the theory (the mental model in the programmer's head) was still missing.

People spent years figuring this out. There was a small academic literature about it. Chris Lomont [wrote a paper](https://www.lomont.org/papers/2003/InvSqrt.pdf) deriving an optimal constant from first principles and found a slightly different value, then went looking for whoever had originally chosen `0x5f3759df` and could not find them. The provenance of the constant traces, probably, back through a chain of "I learned it from a guy who learned it from a guy" that ends at [William Kahan](https://es.wikipedia.org/wiki/William_Kahan), but this was reconstructed historically. The code did not contain it.

(Anyone who has done any scientific-oriented programming knows this. You spend a few hours studying a particularly useful numerical method only to find out that the implementation is actually a `while` loop, a matrix product and a table of constants a French guy computed in the 18th century. The mental model is where the substance is at, the code is just a residue of that model).

This is what Naur was talking about. The artifact is right there, fully readable and commented. The theory that produced the artifact is missing, and no amount of staring at the artifact gets you the theory back. A modern LLM pointed at the Quake III source can give you a fluent explanation of what the function does, especialy because the LLM itself was probably trained on the documentation generated during the interest in that function. It cannot tell you why the constant is what it is, because that information is not in the source. It was never in the source. It was in the head of whoever picked the constant, and that person did not write it down and is now 92 years old and uninterested in online debates about Quake III.

Now scale this up. A working enterprise system has thousands of `Q_rsqrt`-equivalents, **small choices made for reasons that were obvious at the time and never documented**. Most of them are less interesting than the magic number, but they add up to the theory of the system, and that theory is what tells you whether a proposed change is going to break something three months from now. The LLM cannot give you that, because it is not in the input. 

So what you get, in practice, is a reduced version of the documentation, passed through a model that smooths it into plausible-sounding text, and then handed to a human maintainer who has to do their actual work using this reduction. There are now **two layers of compression between the original theory and the person trying to maintain the system**. The first layer is whatever made it into the docs (lossy). Whatever the LLM produces when asked about the docs (also lossy, in a different way) forms a second layer. The maintainer is downstream of both.

Naur did not anticipate LLMs but he anticipated this exact problem. He just thought the second layer was textbooks and training courses, but the mechanism is the same.

## The legibility problem

Here is the part I keep coming back to, and the part that I think most people writing about this are not mentioning: it does not matter whether AI-augmented teams produce worse software. They certainly do, for the reasons above. What matters is whether the degradation is *visible to the people approving budgets*.

For most enterprise software, quality is nearly invisible until something catastrophic happens. The system runs, and some numbers go up. A dashboard is green. Customers file tickets at roughly the same rate they always did, and most of those tickets get closed, and the ones that do not get closed get folded into a backlog that everyone agrees is "tech debt" and nobody is going to look at for the moment. None of this changes when the team that built the system is replaced by a smaller team plus an LLM. It cannot change, because the things that have actually degraded (judgment, taste, the ability to recognize that a proposed change is going to cause a problem in six months) do not show up on dashboards. They show up later, when the catastrophe arrives, and by that point the causal chain back to the staffing decision is too long for anyone to trace.

This is the [bear case](https://en.wikipedia.org/wiki/Market_sentiment) for software quality over the next five years. Not that companies will run their software into the ground in some visible way. They will run it into the ground in an invisible way. The bridge will not collapse,but it will just get gradually less safe to walk on, and at some point you will notice that you have been walking on it for a while without quite trusting your steps.

We have a recent example of what this looks like, and it is X (formerly Twitter). When [Musk fired most of the engineering and trust-and-safety teams](https://www.nytimes.com/2022/11/04/technology/elon-musk-twitter-layoffs.html) in late 2022, the prediction from the loudest critics was that the site would go down within weeks. It did not. The site kept working. The tweets kept loading. From the outside, in the first few months, the doomers looked wrong and the people saying "see, you didn't need all those engineers" looked right.

What actually happened took longer to become visible. [Bots swarmed the platform](https://viterbischool.usc.edu/news/2025/02/a-platform-problem-hate-speech-and-bots-still-thriving-on-x/), with no one to curb the onslaught of fake content. The recommendation algorithm started promoting blue-ckeck (paid) accounts in a way that made the timeline feel like a different product. Grok, the platform's own AI, was caught [generating sexualized images of children](https://www.bbc.com/news/articles/cvg1mzlryxeo) and, separately, [parroting neo-Nazi talking points](https://www.bbc.com/news/articles/c4g8r34nxeno) after a botched system prompt change. Moderation degraded to the point that the type of person who stayed on the platform shifted, which in turn shifted what the platform *was*, which in turn drove away more of the people whose presence had made it valuable in the first place. Even if Mastodon and Bluesky have not entered the mainstream yet, the [Twitter exodus](https://www.nbcnews.com/tech/tech-news/x-sees-largest-user-exodus-musk-takeover-rcna179793) (*X-odus?*) is undeniably a fact. None of these individual facts are "the site went down". All of them are the system getting less safe to walk on.

The X case is also instructive because of what it tells you about legibility. From a pure uptime-and-revenue dashboard, you could probably make a case that the cuts were fine. The site stayed up, costs went down. Some metric somewhere went up. In the end, the degradation was real but it was the kind of thing that does not show up in a quarterly review until it has been compounding for two or three years, by which point the people who made the decision are either gone or have built a narrative in which the decline is somebody else's fault. This is roughly what I expect to happen at every large company that decides its engineering org is mostly redundant now that the LLMs are deemed "good enough".

## What was always true, and what is new

I do not want to overstate the originality of what is happening. Naur's claim, taken seriously, means that companies have *always* been running on borrowed time with respect to their software. Layoffs and attrition have always destroyed theory. People retire, change jobs, get hit by buses, win the lottery. The team that wrote the original version of any system that has been around for more than a few years is mostly gone, and the maintenance is being done by people who inherited the artifact and built up some imperfect approximation of the theory through *years* of contact with it.

What is truly new is both the speed of the destruction and the belief that the destruction is recoverable.

Previous generations of middle managers knew, in some inarticulate way, that losing the team that built X meant X was now a black box. You could see it in how they handled critical systems. They did not lay off the team that maintained the mainframe, nor did they let the COBOL guy retire without a long handover. They did not, for the most part, assume that you could swap out the people without consequences for the artifact. I am sure they were wrong about a lot of things, but not about this.

For the most part, the current generation seem to be acting under the assumption that an LLM can re-derive the theory from the artifacts. This is a very specific delusion and it deserves to be named: the belief that the relationship between a codebase and the theory of that codebase is *recoverable* in the same way that the relationship between a sentence and its meaning is recoverable. It is not. The theory is what produced the code. You cannot run that arrow backwards by reading the code very carefully, and you cannot run it backwards by feeding the code to a model that reads it very fast. 

## The juniors are the next problem

There is a longer-term consequence of all this that I think will not be visible for a few more years. Right now, juniors are the obvious losers in the consulting market. Many of the things they used to do, routine troubleshooting, monitoring, documentation of existing solutions, can be handed to a mid-level person plus an LLM and the math works out. Companies are doing this math. Junior hiring is collapsing in a lot of places.

The argument I want to make is that this is not simply bad for current juniors, as an isolated problem. If the current trend continues, this might be catastrophic for everyone in 2035.

Juniors have always been valuable because they are the next generation of theory-holders being grown. The reason a senior consultant has good judgment is not that they were born with it, but that they spent some years being confused, making mistakes, watching senior people make decisions and slowly understanding why those decisions were the right ones. That process is the apprenticeship. The output of the process is the senior. There is no shortcut.

If you skip that step for a decade, because mid-level consultants + LLMs can do the work that juniors used to do, you get no seniors in 2035, because nobody went through the formative confusion that builds the kind of judgment Naur was talking about. The theory that lives in senior heads in 2035 has to come from somewhere, and that somewhere is the years of contact with real systems and real teams that those seniors had as juniors and mid-level engineers in 2025-2030.

Companies are not going to discover this until the seniors who currently exist start retiring and the shortage becomes painfully obvious. By that point the pipeline cannot be repaired in any reasonable timeframe. You cannot go from zero juniors to a senior with ten years of experience in less than ten years. Maybe the people who should be worried about this already know this, but in an age of AI euphoria, it's probably easier to think that the future looks so bright nothing can go that wrong. 

## The displacement, and what I think actually happens to the rest of us

For mid-level consultants, the picture is messier. The work that used to be routinely junior is going to land on them. This is not all bad. There is real productivity to be had in handing repetitive things to a model and reviewing the output. I have done it. It works.

But the job changes. You spend more time validating LLM output and less time doing the work yourself. You absorb the cognitive load of two or three roles, with a tool that helps with the volume but not with the responsibility. The mental models you need to do this well are exactly the ones that take years to develop, which is not a problem for current mid-level consultants but is going to be a problem for whoever is supposed to replace them (the aforementioned juniors).

I also do not think this gets cheaper for companies in the long run. The headline savings (fewer salaries) are real but the transition costs are not free, and as everyone adopts the same tools, the productivity gain stops being a differentiator and becomes the new baseline. We will end up doing more work for the same money, with smaller teams, more concentrated risk, and a worse pipeline of people behind us. The companies that figured this out first will look smart for a few quarters. The ones that come after will just be matching the new baseline.

## Accountability



There is a line I keep thinking about, [from a 1979 IBM training manual](https://simonwillison.net/2025/Feb/3/a-computer-can-never-be-held-accountable/):

![A computer can never be held accountable. Therefore a computer must never make a management decision.](https://github.com/malmriv/malmriv.github.io/blob/master/_posts/images/a-computer-can-never-be-held-accountable.jpg?raw=true)

It is one of the cleanest statements I can think of about why humans need to be in the loop. You can build any system you want, but at the end of the chain there has to be a person or an organization that owns the consequences. That ownership is what makes the system trustworthy. It is what makes contracts enforceable, what makes regulators able to do their job, what gives customers someone to call when something goes wrong.

I used to think this was the strongest argument for why headcount could not collapse all the way. Someone has to be responsible, the LLM cannot do that. So at minimum you need a person at every level where a decision gets made.

I am less sure now. The thing about **corporations** is that they **have gotten extremely good at distributing responsibility so that no individual is accountable for anything**. Catastrophic failures happen and the post-mortem concludes that it was a "process failure," or that "lessons have been learned," or that the relevant person retired six months earlier and is now very invested in raising pidgeons. Just yesterday, [The Register](https://www.theregister.com/2026/04/27/cursoropus_agent_snuffs_out_pocketos/) reported on the Claude-powered loss of a startup database, and the results were just what you could expect. So, adding an LLM to the daily workings of a corporation is just one more layer in a system that was already designed to make sure nobody can be blamed for anything too bad.

The IBM rule still describes what *should* be true. A computer cannot be held accountable, therefore a computer must never make a management decision. But the rule does not enforce itself. What enforces it is regulators, lawsuits, public pressure, and a working culture inside the company that takes ownership seriously. All of those things are weaker than they used to be. So the rule is going to get tested, and I do not think the test goes well.

## What I think we should be doing

I am wary of ending a post like this with a tidy list of recommendations because I do not really have any that I am confident about. The forces here are large and the people who could change them have other priorities. But if I had to say what I would tell my own team:

Document the theory instead of the code, which documents itself. What does not document itself is *why* the code looks the way it does, what was tried and rejected, what invariants the team takes for granted, what the failure modes look like in practice. Write that down. Write it down even if it feels stupid. Write it down for the version of you that will exist in three years and has forgotten everything, because you will most likely forget the details. I know I do.

Treat senior people as carriers of the theory and act accordingly. If a senior is leaving, the cost behind that is theory walking out the door. Pay for proper handovers. Pay for overlap. If management does not want to pay for these things, that is a signal about how seriously they take their systems being maintained.

**Hire juniors anyway**. I know the math does not work in the short term. I know the LLM plus a mid can do what a junior used to do. Hire juniors anyway, because the alternative is a pipeline that is dry in ten years, and ten years is not that long.

Be skeptical when someone tells you the LLM has read the codebase and understands it. **The LLM has read the artifact**. The theory was never in the artifact in the first place.

That last one is the thing Naur was telling us forty-one years ago. We did not listen then either, but the cost was lower because we did not think we had a way out. Now we think we have a way out, and we are probably about to find out that we do not.
