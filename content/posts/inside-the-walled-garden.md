---
title: "Inside the Walled Garden"
date: 2026-06-10
summary: "I had a personal AI vault that genuinely worked. The one thing it couldn't touch was the work that mattered most. So I moved it inside the walls - and the governance wall turned out to be the enabler."
tags: ["AI", "Governance", "Zettelkasten", "Copilot", "VS Code"]
draft: false
---

*My second brain finally gets to think about all of my actual job.*

I built a second brain that genuinely works. A vault of plain Markdown notes, an AI thinking partner that reads them, and a way of working I've written about more than once here. It made me faster, and more importantly it made me more insightful. But it had a glass ceiling, one that I spent months looking straight up through at the work I most wanted help with.

The block had nothing to do with the tools. Plenty of work thinking was fine in my personal vault - just never the genuinely interesting, high-impact kind. That work is soaked in company context - customers, accounts, strategy, the real substance of the day job. None of that could go anywhere near a personal vault wired to a personal AI account, and quite rightly so. The single most valuable category of my thinking was the exact category my second brain wasn't allowed to touch.

I'd built myself a beautiful kitchen. My own pantry, stocked with everything I could keep at home. But the ingredients I actually wanted to cook with - the good stuff - were all locked away at work, and I couldn't carry them out the door. So I did the only thing that made sense. I moved the whole kitchen into the office.

## Approved doesn't have to mean downgraded

Here's the worry everyone has about "corporately approved" tools: that approved is a polite word for worse. The sanctioned option is the beige one, three versions behind, the one that makes you quietly mourn the thing you actually liked.

It didn't play out that way. I rebuilt the same workflow on tools my company already blesses - with Copilot and our enterprise Azure OpenAI doing the AI heavy lifting - and I lost nothing. I even kept my sous-chef: Copilot lets me choose my model, so I'm working alongside the latest Opus, the same calibre of help I'd had at home. I'm still the cook. Same hands, same recipes - except now the kitchen's in the office, and for the first time the pantry is fully stocked.

## An ode to VS Code

Now for the part I did not see coming. The tool that became my second brain is VS Code - an IDE, already sitting on my corporate VDI. On the surface it looks far more suited to a software engineer than a typical knowledge worker, but it has something most note apps would envy: a vibrant community of extensions that can completely transform and customise it. Whatever capability you can imagine probably already exists, and if it doesn't, a weekend of homebrew code will get you there. I pointed all of that at notes instead of code, and an editor built for engineers turned into the best vault I've run.

And here's the bit the rest of the knowledge-work world hasn't quite clocked yet. Engineers have spent the last couple of years with an AI partner living *inside* the document - GitHub Copilot sitting right there in the editor, reading what's open, aware of the whole project, ready to draft, question, or rework in place. We gave that to the people writing code and somehow decided the people writing strategy could make do with a chat window in a different tab, copying context back and forth by hand. I'd already got past that the hard way: Obsidian in one window, Claude Code in another, both pointed at the same vault of files. The AI was in the room - it could see everything I could - it was just split across two panes I had to keep wrangling. VS Code closes that seam. Same AI in the room, now in one window, looking at the same notes I am with nothing to wrangle. For knowledge work, that's the difference between an assistant you have to manage and one that's already up to speed.

If you want to try this, here's what makes VS Code a credible Obsidian replacement:

- **Still just plain `.md`** - the format is mine and the method is mine, the AI-augmented way of thinking I've been building and writing about, my own chain of cognition. Nothing locks me to a single vendor; if any one of these tools vanished tomorrow, the way I work survives it. (The company data, of course, stays exactly where governance wants it. Portable format is not the same as portable data, and I'm not walking anything out the door.)
- **Git underneath it all** - the whole vault is a Git repository: full version history, every change reversible, backed up and synced without a proprietary cloud in the middle. A second brain with an undo button that goes back years, on infrastructure my company already trusts.
- **GitHub Copilot** - the AI partner embedded in the editor itself, reading the notes you have open and the wider vault around them. The piece no standalone notes app can match.
- **An operating doc and a memory** - a `COPILOT.md` at the vault root tells the AI how this place works: the conventions, the house style, the things it must never do. Every conversation starts from the same brief. A paired memory layer keeps notes on how I work and what we've learned, so the partnership accrues rather than resets. The instructions are the standing orders; the memory is the experience.
- **Extensions** that turn an editor into a vault:
    - **Foam** gives you the wikilinks, backlinks, and graph view - the connective tissue of a second brain.
    - **Front Matter CMS** turns your note metadata into database-style views. Its graph is genuinely better than the one I left behind.
    - **Mark Sharp** renders your Markdown into a clean, editable page, so you're not staring at raw syntax all day.
    - **Markdown All in One** handles the authoring ergonomics.

And when no extension quite fits, you build the missing piece by describing it - which is exactly how I handled the part I'd most feared losing: search. In my old personal setup I'd built proper retrieval, not just keyword but four ways into a vault: keyword for what I said, metadata for what I filed, the graph for what I connected, and semantic search for what I actually meant. I described what I wanted, had Copilot help me refactor my own homebrew tools, and the whole thing came across - including the trick I'd miss most, surfacing notes that *should* be linked and aren't. The same AI that helps me think wrote the tools that help me find. And because it all lives in one window, retrieval doesn't just answer me - it feeds the partner in the editor directly. A second brain pays you back through retrieval, and here it pays back twice.

## The payoff compounds

When the ceiling lifted, the difference was not subtle.

For the first time, my whole job lives in the vault - not just the low-stakes corner of it. A few weeks ago I got blindsided by a late invite to an account review, the kind where you're expected to walk in with a point of view and I didn't have much time to pull mine together. Thankfully, everything I needed was already in the vault - meeting transcripts, various presentations, the half-formed notes I'd jotted to myself over months since I was handed the account. I had Copilot pull those threads together, and we shaped a single coherent summary of the current strategy at short notice. The depth of what came back changed how I work. **I'm not going back.** I genuinely don't know how I'd return to thinking about my work without it.

And this is where it compounds. Every piece of real work I do now lands back in the vault - the transcript, the strategy, the rough note - and becomes context for the next piece of work. More real context in, sharper thinking out, which becomes more context in. That loop was always there, but it used to run on the low-stakes scraps. Now it runs on the work that actually matters, and the more I feed it, the more it gives back. That's where the curve stops being linear.

And the loop doesn't have to stop at the edge of my vault. Right now I feed it by hand - I decide what lands there. The next step is letting it reach into the enterprise context I'm already entitled to see: Work IQ and the organisational graph, MCP connectors into the systems I'm cleared to consume. The vault stops being a place I deposit knowledge and becomes a lens onto everything around it that I'm permitted to use. The thinking gets sharper still, because it's no longer limited to what I remembered to write down.

## The wall is the point

We tend to talk about the walled garden as a cage - the place innovation goes to be governed to death. I'd flip it. The wall is exactly what made the valuable thing possible. Outside it, I could only ever think about the low-stakes slice of my work. Inside it, I can finally point real AI leverage at real company knowledge, appropriately, with the classification and control that makes it allowed in the first place.

Governance didn't block the powerful use case. Governance is what unlocked it.

There's a flip side for the enterprise, though. The moment a tool like this can reach into company systems, the value of those systems depends entirely on how accessible they are. Data locked in a silo nobody can connect to is data that doesn't get thought with. The organisations that pull ahead in this AI-powered way of working will be the ones that treat their data sources as a product - deliberately made accessible, secure, and transparent for exactly this kind of consumption. The wall isn't just what keeps the wrong things out; it's what makes it safe to plug the right things in.

The tools matter less than the lesson under them. Own your notes in a format nobody can take from you. Use the AI surface your company already trusts. Keep your right to choose the model. Then point the whole thing at the work that actually matters, and let it compound.

That last part was always the real engine. The vault, the extensions, the model are all just infrastructure. The thinking is still mine - it just finally gets to work with the whole picture of what I'm actually doing.
