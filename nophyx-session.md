# Codex conversation

## User

I am going to describe the project that i am trying to build and i want you to just ask me anything and everything. Rightnow we are thinking more about technology and what to use(i would prefer to use react with vite for the frontend and go for the backend tho). What the main idea is that in Bulgaria there is something called "Parliament control" in which every parliament meeting(usually 3 a week) is recorded in video and also transcribed fully and what i want to do is make something like a search engine for it(it kind of is both simple and also complex at the same time and it can also stand as a strong foundation for a lot of functionality that can make the whole thing a lot better). I am going to give you an psuedo-example of what a user may want to do and how they would achieve it before giving you what this project is supposed to do. Lets say that i want to check if a political party has done anything towards their promises(they are usually posted on their sites), what i can do in nophyx is that can search for that party and using specific tags and keywords to be able to see what they have done. For now its supposed to be more of a tool that helps you instead of the full framework of how you search and find information, but it could be built upon.The key steps as I see (but i can entirely be wrong) are 1. Downloading constantly the new transcripts of the meetings; 2. Somehow indexing and adding them to the search engine; 3. Being able to search through them . That is the main pillar of the project(of course with adding more functionality it would become more accessible), and also some things that are the first priority after the search engine: 4. Being able to give a "prompt"(just saying what to search) and having a llm being able to set the tags and keywords for the search(of course not searching right away but giving you to see them before submitting). 5. When we go to the result page to be able to see something like an excert that has caught the filters and then to be able to click it to go the the full thing; 6. For searching for like a policy to be able to see how each party has voted 7. Other things like 6 which would be incredibelly useful for searching.; What this project is mainly for is to be able to easily and acurately gather information about politics because a lot of young people get the same generic answer of "just read each party's statements, follow the news" when asking how to get more insight about politics which at face-value seems like a good answer, but also an astronomical task, especially in the everchaning political climate that just shifts around almost monthly. I know that i have probably forgotten a lot of big details, but just ask. To be honest the search engine part is the most merky thing about the whole project; the only ready to use solution that i know of is elastic search, but maybe that wont cut it. Also not sure how we need to tag and categorize each and everything. Another thing is that for features like 6. or 7. they need to be thought out before actually categorizing because it would be a headache to do after that.

## Assistant

I’ll check what’s already in the project and how Parliament publishes transcripts and voting records. Then we can work through the choices that shape the search engine, starting with what a useful result should look like.

## Activity

    $ git status --short && ls -la
    total 16
    drwxr-xr-x  6 jorobg users 4096 Oct 22 02:29 .
    drwxr-xr-x 17 jorobg users 4096 Oct 22 01:42 ..
    dr-xr-xr-x  2 jorobg users   40 Oct 22 02:29 .agents
    dr-xr-xr-x  2 jorobg users   40 Oct 22 02:29 .aws
    dr-xr-xr-x  2 jorobg users   40 Oct 22 02:29 .codex
    drwxr-xr-x  7 jorobg users 4096 Oct 22 01:42 .git
    -rw-r--r--  1 jorobg users  572 Oct 22 01:42 .gitignore
    ✓ • 0ms

## Activity

    $ pwd && rg --files -g 'AGENTS.md' -g '!node_modules' -g '!vendor' -g '!dist' -g '!build' -g '!package-lock.json' -g '!pnpm-lock.yaml' -g '!yarn.lock' -g '!go.sum' -g '!Cargo.lock' -g '!bun.lock*' -g '!\.git' -g '*' .
    /home/jorobg/Coding/nophyx
    ./.gitignore
    ./.git/packed-refs
    ./.git/config
    ./.git/logs/refs/heads/main
    ./.git/logs/refs/remotes/origin/HEAD
    ./.git/logs/HEAD
    ./.git/description
    ./.git/refs/heads/main
    ./.git/refs/remotes/origin/HEAD
    ./.git/index
    ./.git/objects/pack/pack-38868bb8ff5a58e7cbd24f48187c1628d742ae98.pack
    ./.git/objects/pack/pack-38868bb8ff5a58e7cbd24f48187c1628d742ae98.idx
    ./.git/objects/pack/pack-38868bb8ff5a58e7cbd24f48187c1628d742ae98.rev
    ./.git/hooks/pre-applypatch.sample
    ./.git/hooks/fsmonitor-watchman.sample
    ./.git/hooks/commit-msg.sample
    ./.git/hooks/pre-receive.sample
    ./.git/hooks/applypatch-msg.sample
    ./.git/hooks/sendemail-validate.sample
    ./.git/hooks/update.sample
    ./.git/hooks/prepare-commit-msg.sample
    ./.git/hooks/pre-commit.sample
    ./.git/hooks/push-to-checkout.sample
    ./.git/hooks/pre-push.sample
    ./.git/hooks/pre-merge-commit.sample
    ./.git/hooks/pre-rebase.sample
    ./.git/hooks/post-update.sample
    ./.git/info/exclude
    ./.git/HEAD
    ✓ • 0ms

## Activity

    Searched the web for site.parliament.bg пленарни заседания стенограми гласуване парламентарен контрол

## Activity

    $ for p in /AGENTS.md /home/AGENTS.md /home/jorobg/AGENTS.md /home/jorobg/Coding/AGENTS.md /home/jorobg/Coding/nophyx/AGENTS.md; do if [ -f "$p" ]; then printf '%s\n' "$p"; cat "$p"; fi; done
    ✓ • 0ms

## Activity

    Opened https://www.parliament.bg/bg/plenaryst/ns/61/ID/11050

## Assistant

I’ll treat **React with Vite and Go** as your working preferences. The repository is essentially empty, so there’s no existing architecture to work around.

One useful distinction: **парламентарен контрол** is the questions-and-answers part of Parliament’s work; **пленарни заседания** is the broader set of sittings, including legislative debates and votes. Your examples seem to need that broader coverage. Parliament’s published transcript pages also list downloads for voting by parliamentary group and by individual MP, which gives us a separate source to investigate for the voting feature. [Example sitting](https://www.parliament.bg/bg/plenaryst/ns/61/ID/11050)

Your concern about future categorization is worth addressing early, but we don’t need to invent every topic tag now. I’d distinguish:

- **Source facts:** sitting, date, speaker, affiliation at the time, agenda item, bill, and vote.
- **Derived labels:** topics such as healthcare or education, synonyms, and machine-generated classifications.

My initial design principle would be to preserve the original documents and their relationships so we can revise classifications and rebuild the search index later. Choosing Elasticsearch—or another engine—comes after deciding what searches must succeed.

Also, “mentioned a policy,” “proposed a measure,” and “voted for a measure” should be distinguishable evidence. That distinction matters directly to your example of investigating a party’s promises.

Let’s work through this in small rounds. First, three decisions that determine how much data and search capability the foundation needs.

## Activity

    Searched the web for site.parliament.bg парламентарни комисии постоянни временни подпомагат дейността законопроекти, site.parliament.bg комисия стенограми заседания 2025

## Assistant

**Plenary sittings** are meetings of Parliament as a whole—the main chamber where MPs debate legislation, hold votes, and conduct parliamentary control. **Committees** are smaller groups of MPs responsible for particular subjects, such as healthcare or finance. They examine proposals in detail and prepare reports and recommendations for Parliament. Some committees are temporary and investigate a particular issue. [Parliament’s explanation of committees](https://www.parliament.bg/bg/parliamentarycommittees)

For Nophyx, the practical difference is:

| Source | What someone could discover |
|---|---|
| Plenary debate | What an MP argued publicly, responses from other parties, and the surrounding chamber debate. |
| Committee proceedings | Detailed objections, proposed changes, technical explanations, and contributions from invited participants. |

For example, researching a housing proposal could involve finding its detailed discussion in a committee and then following it into a plenary debate. Committee participants can include ministers, experts, and civil society representatives, so we need to distinguish their roles from those of MPs. [Example committee participation rules](https://parliament.bg/bg/parliamentarycommittees/2336)

Including both fits your research goal. It adds more sources to collect and interpret, and we still need to check how consistently their records are published. I’ll keep both in scope based on your answer.

I’ve also recorded your search preference as **keyword search plus related-concept retrieval**, with keyword matches prioritized and conceptual matches appearing lower. The LLM’s proposed keywords and filters would remain editable before submission, as you originally described.

The next choices affect the technology much more than the frontend framework: how far back we search, what it can cost to operate, and how much infrastructure you want to maintain. The budget options below are spending preferences, not estimates of what the project will cost.

## Activity

    Searched the web for site.elastic.co/docs/reference/text-analysis/analysis-lang-analyzer bulgarian analyzer, site.docs.opensearch.org bulgarian analyzer hybrid search, site.ai.google.dev/gemini-api/docs embeddings gemini embedding multilingual

## Assistant

Your history priorities are clear: **use the current term to develop and test, then backfill to 2021, with earlier records a lower priority**. Users should be able to set a date range within whatever coverage is available.

I haven’t established that older records use incompatible formats. That’s something to investigate, not assume. Historical coverage also means handling MPs changing affiliations, parliamentary groups forming or dissolving, and identifying the same person across terms.

Free hosting and Gemini access change the cost discussion. I’ll prioritize quality and reasonable resource use, with a self-hosted deployment that can contain several Docker services. We can leave the exact Gemini model open until you know what eduHack provides; access to generation models doesn’t necessarily establish access to embedding models, which are used for meaning-based retrieval.

On your original Elasticsearch concern: **it is a serious candidate for this project**. It already has a Bulgarian language analyzer, including stemming to help match different forms of words. OpenSearch also supports Bulgarian. That makes both worth evaluating against actual Bulgarian parliamentary queries before selecting one. [Elasticsearch language support](https://www.elastic.co/docs/reference/text-analysis/analysis-lang-analyzer), [OpenSearch language support](https://docs.opensearch.org/latest/analyzers/language-analyzers/index/)

Two product choices need particular care: whether a party filter means *speaking* or *being discussed*, and whether a subject tag is reliable enough to exclude everything outside it. I also need your development timeframe to keep the technology discussion realistic.

## Assistant

When is the eduHack demo or submission deadline, and are you building alone or with a team? Include roughly how much development time you have; if the deadline is undecided, say so.

When someone selects a party, which evidence should the initial transcript search return?
- Provide separate “spoken by” and “mentions” modes, defaulting to “spoken by” (Recommended).
- Only speeches by that party’s MPs initially; mentions can be searched as ordinary text.
- Include both together in the same results.

For subject tags such as healthcare or education, which approach fits your vision? This is separate from factual filters such as speaker, date, or committee.
- Start with a small, curated topic list and automatically suggested labels that are clearly identified as inferred (Recommended).
- Start without subject tags; use keywords, related-concept search, and factual filters until we understand the corpus.
- Make a rich, automatically generated topic hierarchy a core feature from the start.

## Activity

    Searched the web for site.docs.opensearch.org latest hybrid search keyword semantic filter, site.elastic.co/docs solutions search hybrid search lexical semantic, site.postgresql.org/docs/current text search dictionaries snowball languages bulgarian

## Activity

    Searched the web for site.parliament.bg/bg/parliamentarycommittees "Стенограма" "2025", site.parliament.bg "Отворени данни" "Гласувания", site.parliament.bg "Гласуване по парламентарни групи" "csv"

## Activity

    Searched the web for site.strazha.bg стенограми търсене, site.github.com strazha parliament bulgaria

## Assistant

Your historical rollout is clear: **current term for testing → backfill to 2021 → older records later**, with date-range filters throughout.

One useful reference I found is [Стража](https://www.strazha.bg/), which already presents parliamentary speeches, votes, and historical membership. We can use it to help identify what Nophyx should make easier.

The next questions are:

1. **What’s your deadline and team size?** Roughly how much development time do you have before the eduHack submission?
2. **What should selecting a party mean?** Speeches by its MPs, mentions of that party by anyone, or two separate modes? I recommend separate modes, defaulting to speeches by its MPs.
3. **How central are subject tags?** Would you prefer a small curated topic list with automatically suggested labels, postpone topic tags until search works, or make a detailed topic hierarchy essential from the start? My starting recommendation is the small curated list; factual filters such as date and speaker would remain separate.

## User

H

> When is the eduHack demo or submission deadline, and are you building alone or with a team? Include roughly how much development time you have; if the deadline is undecided, say so.

well it is up to nov 5th and today is oct 22, but i think that we will manage. Also after that i plan to continue to develeop it. Also it seems that also for this eduhack only 25percent is the actual project in the flesh and a for some reason more is given to the idea, presentation and such

## Assistant

That gives you about two weeks, with development continuing afterward. Given the scoring you described, I’d aim for one convincing, reliable research workflow for the demo, while designing the data model for the larger product. We don’t need every planned source or feature working by November 5 to demonstrate that foundation.

I’ll keep the hackathon milestone separate from the longer-term scope. The main unresolved product choices are still what selecting a party means and how subject tags should work.

## Assistant

That makes November 5 a **demonstration milestone**, with the fuller product developing afterward. Given the scoring you described, I’d prioritize demonstrating a trustworthy flow: describe a topic → review suggested search criteria → find relevant excerpts → inspect the original context.

That also gives the presentation something concrete to explain: what works today, how you verify its accuracy, and how voting records and broader coverage extend it.

One missing practical detail: **who will be building it?**

- How many people will actively write code, and how will you divide the work?
- What experience do you already have with React, Go, databases, and Docker?
- Is Go something you already use, or something you want to learn through this project?

Your answers will help me distinguish useful architecture from infrastructure that would consume your development time.

## User

The thing is that i plan to for this to be coded by me. What i plan is to use llms to aid the process a lot, and so i need a really detailed plan for everything, when adding something critical to the system and just in general i need somehting like claude.md but for codex, so that my llms always have detailed context of the project. I have a lot of experience with react,go and databases, but not so much with docker.I know go and use it when needed.

## Assistant

Then we can plan around you building the system in Go and React, with LLMs assisting throughout. Your experience makes those choices straightforward; Docker mainly needs a clear development setup and operating instructions.

I’ll check Codex’s project-instruction conventions using the **openai-docs** skill. We should separate persistent instructions for coding agents from the detailed architecture and implementation plan, so both stay useful as the project grows.

## Activity

    $ cat /home/jorobg/.codex/skills/.system/openai-docs/SKILL.md
    ---
    name: "openai-docs"
    description: "Use for Codex models/pricing, scheduled tasks, skills, settings, setup, troubleshooting, customization, automations, and self-knowledge—including 'you,' 'your,' 'this app,' or 'this coding agent' when they refer to Codex—and for OpenAI APIs/products and ChatGPT Work. Also use for model choice/migration, prompting, SDKs, Responses, Realtime, agents, evals, and Chat/Work/Codex comparisons. Do not use for generic app/software tasks that merely mention Codex."
    metadata:
      short-description: "Codex models/pricing, scheduled tasks, skills, settings, setup, troubleshooting, and self-knowledge; OpenAI APIs and ChatGPT Work. 'You'/'this app' means Codex only."
    ---
    
    # OpenAI Docs
    
    Provide current, cited OpenAI product, API, model, and Codex guidance. Read zero or one primary reference.
    
    **First substantive action:** Search the user's exact requested official OpenAI documentation topic and any explicitly named model using a concise, topic-specific query of 2-6 essential terms. When an already-available direct official documentation search and page-retrieval capability is present, use it first: search, then fetch or open the matching official page before general web search. Otherwise, immediately use official-domain web search, then actually open or fetch the relevant official page. Complete this source order before reading a reference, inspecting local or repository files, running a Codex manual or model resolver, drafting a plan, or answering from memory. Use the actual fetched page, not a search snippet or an unopened link. If one official search or page does not establish the answer, search another appropriate official domain and actually open or fetch the result. Preserve the exact requested model; never substitute a newer model.
    
    **Only exception:** An explicitly requested, genuinely broad, cross-topic Codex setup, orientation, or system-map synthesis may use the manual first when shell execution and an allowed temporary cache are available. A specific Codex feature, setting, command, error, model, or requested citation remains docs-first. Mixed Chat/Work/Codex comparisons are official documentation questions, not manual-first Codex requests.
    
    For generic software tasks, answer the software task directly. OpenAI implementation, debugging, SDK, API, prompting, agent, and eval requests are not generic.
    
    For a straightforward factual or citation-only request, follow the source order and do not read a route reference. This includes straightforward API facts, ChatGPT Work or mixed Chat/Work/Codex comparisons, model tiers, aliases, Pro mode, reasoning settings, factual migration baselines, and narrow Codex facts. Prioritize `learn.chatgpt.com` for ChatGPT Work.
    
    ## Choose one primary route
    
    Use the first matching route, and read its reference only when the requested task needs that specialized workflow:
    
    - **Explicitly requested local documentation integration:** Read [integration guidance](references/mcp-diagnostics.md) only when the user explicitly requests that local integration.
    - **Model migration, upgrades, or model-specific prompting:** Read [model-migration.md](references/model-migration.md) for actual migration planning, implementation, dynamic target resolution, or prompt changes. Preserve an explicitly requested target.
    - **Model selection and comparisons:** Read [model-selection.md](references/model-selection.md) only when nuanced current, latest, default, cost, latency, quality, or modality tradeoffs need more guidance. Do not run a migration resolver for selection alone.
    - **Product, API, ChatGPT Work, and mixed Chat/Work/Codex documentation:** Read [official-docs.md](references/official-docs.md) only when fetched official pages leave source selection, API schemas, or the requested implementation unresolved. This route is not manual-first.
    - **Explicitly broad Codex setup, orientation, or cross-topic synthesis:** Read [codex-self-knowledge.md](references/codex-self-knowledge.md) when the eligible Codex manual or deeper Codex procedures are needed.
    
    Read at most one primary reference. Do not open every route, bundled model guide, or helper script. Read a supporting reference or run a helper only when the chosen workflow demonstrably needs it.
    
    ## Source and execution boundaries
    
    - Search, open, fetch, and cite only `developers.openai.com`, `platform.openai.com`, and `learn.chatgpt.com`. Cite the page that supports the claim. State uncertainty when official sources do not establish pricing, availability, account access, limits, or behavior.
    - Preserve an explicitly requested model for selection, migration, and prompting. Resolve an unspecified latest or current migration target only after searching and fetching current official guidance.
    - Use `references/latest-model.md` only as a disclosed fallback after current official model guidance does not answer the question. Read `references/upgrading-to-gpt-6-astra.md` only for an actual, requested GPT-6 migration; read `references/prompting-guide.md` only for requested prompting work.
    - Before building, running, editing, debugging, or testing an API-backed app or tool, use `openai-platform-api-key` first when available. Documentation, conceptual examples, model selection, and read-only guidance do not require an API key.
    - Say "OpenAI Docs" or "official OpenAI documentation" in user-facing answers. Keep exact official citations and examples concise.
    ✓ • 0ms

## Activity

    $ rg --files --hidden -g '!\.git/**' -g '!node_modules/**' -g '!vendor/**' -g 'AGENTS.md' -g '*README*' -g '*PLAN*' -g '*ARCHITECTURE*' -g '*claude*' -g '*CLAUDE*' -g '*codex*' . .agents .codex
    ✗ (1) • 0ms

## Activity

    Searched the web for Codex AGENTS.md project instructions

## Activity

    Opened https://developers.openai.com/codex/guides/agents-md

## Activity

    Searched for 'Codex reads'

## Assistant

**The Codex equivalent is `AGENTS.md`.** Codex reads it as repository guidance when starting work, and it can point agents to more detailed project documents. [Official OpenAI documentation](https://learn.chatgpt.com/docs/agent-configuration/agents-md)

For Nophyx, I’d organize that context like this:

| Document | Purpose |
|---|---|
| `AGENTS.md` | Project summary, development commands, coding conventions, required checks, and instructions for finding and maintaining the relevant documentation. |
| `docs/product.md` | Intended users, research workflows, feature priorities, and what each filter or result means. |
| `docs/architecture.md` | Components, data model, ingestion pipeline, search behavior, API contracts, and deployment. |
| `docs/decisions/` | Significant decisions and their reasoning—for example, why we chose a search engine or how historical affiliations work. |
| `docs/plans/` | Detailed implementation milestones with dependencies, acceptance criteria, verification steps, and current progress. |

The `docs/` structure is my recommendation; those filenames aren’t special Codex features. `AGENTS.md` would tell an agent which documents to read for each kind of task. OpenAI also documents maintaining implementation plans for work that spans multiple sessions. [Planning guidance](https://developers.openai.com/cookbook/articles/codex_exec_plans)

I’d make **documentation updates part of completing a change**. If we change indexing, for example, the same work must update the search specification, explain any reindexing requirement, and record the verification results. The documents should clearly distinguish implemented behavior, agreed plans, and unresolved questions.

For this project, the eventual plan needs particular detail around source preservation, speaker identification, historical affiliations, transcript segmentation, ranking, and recovery from failed imports. Those are places where an apparently small implementation shortcut could undermine later features.

Your experience also means we can choose the backend architecture on its merits. For Docker, the plan can include exact startup, migration, backup, and recovery commands.

Before documenting the data model, let’s settle the two outstanding filter questions and one related distinction: **a parliamentary group and an individual political party aren’t necessarily the same entity**, particularly for coalitions.

## Assistant

We’ve settled two useful rules: **“spoken by” and “mentions” are separate search modes**, and affiliation filtering initially uses **the parliamentary group at the time of the speech**.

On topics, there are actually two independent choices:

- **Who defines the vocabulary:** you curate it, or an LLM invents and changes it.
- **How it is organized:** a flat list, or a hierarchy of topics and subtopics.

A curated vocabulary can have a rich hierarchy. Automatic classification can assign passages to that vocabulary. We don’t have to choose between manual tagging and an entirely AI-generated system.

| Approach | Advantages | Costs and weaknesses |
|---|---|---|
| Small curated vocabulary, automatic classification | Consistent filters; easier to explain and evaluate; manageable for one developer. | Broad categories can be unhelpful for narrow investigations; classification can still be wrong. |
| Detailed curated hierarchy, automatic classification | Supports browsing from broad subjects into specific issues; predictable category names. | More work to define boundaries, handle overlapping subjects, and maintain the vocabulary. |
| Automatically generated hierarchy | Can discover unexpected themes and suggest specific topics without defining everything beforehand. | Can produce duplicate categories, inconsistent granularity, and unstable names or groupings; harder to compare results over time. |

For example, a discussion about mortgage subsidies could belong to **housing**, **finance**, and **social policy**. A strict tree that forces every passage into one branch would be awkward. We should allow multiple topic assignments.

**My recommendation is a small curated vocabulary with room for reviewed expansion.** An LLM assigns the initial topics and can suggest subtopics. You approve additions to the public vocabulary when they prove useful. Specific names, bill identifiers, and search phrases can remain searchable without each becoming a permanent category.

Two implementation choices would preserve that flexibility:

1. **Store topic assignments separately from the source text**, including whether they are machine-generated or reviewed and which classification version produced them. Revising the taxonomy may require reclassification and reindexing, but it shouldn’t require rebuilding the document collection.
2. **Classify passages or speeches at an appropriate granularity.** Labeling an entire sitting “healthcare” shouldn’t automatically make every unrelated speech in it a healthcare result.

Also, topic tags should help users narrow a search when they choose to. Ordinary keyword and meaning-based retrieval should still search untagged passages. Otherwise, a classification mistake could hide useful evidence before the search engine even considers it.

That gives us a path toward a rich hierarchy while keeping the first vocabulary understandable and testable.

## Assistant

I’ll include **`docs/backlog.md`** in the planned documentation for feature ideas and later improvements. Each entry can record the user benefit, proposed behavior, priority, dependencies, and status. `AGENTS.md` should make clear that backlog entries are ideas to assess, rather than instructions to implement automatically.

For the reader, I’d start with a simple Nophyx page that opens at the matching speech, shows surrounding context, and links to the official source. Features such as synchronized video, annotations, or comparing passages can go into the backlog without complicating that first version. Bulgarian will be the initial interface and search language.

There’s an important distinction in the tagging workflow: **reviewing the vocabulary does not mean reviewing every tag assignment**.

For example, once “healthcare” is an approved topic, the system can automatically assign it to thousands of passages. You wouldn’t approve those individually. Review would apply when the system proposes a *new category*, such as “access to specialist care.” Those proposals could be consolidated into occasional batches.

That was my intended recommendation, but your comment could also mean you want new categories published automatically. Those lead to different maintenance and search behavior, so let’s pin that down.

## Assistant

That settles tagging: **assign approved topics automatically, and review only changes to the vocabulary**.

The next useful input is a few concrete searches you personally want to make. Those will give us acceptance cases for ranking and help judge whether the search is actually useful. I’ll also inspect the published source formats more closely so the ingestion plan is grounded in the data we can obtain.

## Assistant

Give me two or three searches you would actually want Nophyx to handle, preferably with the Bulgarian wording you might type, and briefly describe what a useful result would show. You don’t need to know the exact transcript—for example, these could concern a party’s activity on a policy, a remembered statement, or how discussion of an issue changed over time.

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/bg/plenaryst/ns/61/ID/11050
    curl: (6) Could not resolve host: www.parliament.bg
    ✗ (6) • 0ms

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/bg/plenaryst/ns/61/ID/11050
    <!doctype html>
    <html lang="bg">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport"
              content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <meta name="robots" content="index, follow" />
        <meta name="googlebot" content="index, follow" />
        <meta name="revisit-after" content="10 days" />
        <meta name="revisit" content="10 days" />
        <meta name="author" content="parliament.bg" />
        <meta property="og:image" content="https://www.parliament.bg/images/logo.png" />
    
    
        <title>Parliament.bg</title>
         <link href="/assets/css/uikit.min.css" type="text/css" rel="stylesheet"/>
         <link href="/assets/css/uikit-rtl.min.css" type="text/css" rel="stylesheet"/>
    
    
    
    
        <link href="/css/app.css?id=a243f516ec5440f8c2a1" type="text/css" rel="stylesheet"/>
        <link href="/css/all.min.css" type="text/css" rel="stylesheet"/>
    
        <link rel="alternate" type="application/rss+xml" title="Новини от Народно събрание"  href="https://www.parliament.bg/feed/news" />
        <link rel="alternate" type="application/rss+xml" title="Последни законопроекти от Народно събрание"  href="https://www.parliament.bg/feed/bills" />
        <link rel="alternate" type="application/rss+xml" title="Последни закони от Народно събрание"  href="https://www.parliament.bg/feed/laws" />
        <link rel="alternate" type="application/rss+xml" title="Последни проекти на решения от Народно събрание"  href="https://www.parliament.bg/feed/ns_acts" />
        <link rel="alternate" type="application/rss+xml" title="Последни решения от Народно събрание"  href="https://www.parliament.bg/feed/desision" />
        <link rel="alternate" type="application/rss+xml" title="Програма за работата на Народното събрание"  href="https://www.parliament.bg/feed/plenary" />
    
    </head>
    <body>
    <div id="app">
        <v-header-top></v-header-top>
        <v-header></v-header>
        <main style="">
            <div class="">
                <keep-alive>
    
                 <router-view :key="$route.fullPath">
                </router-view>
                </keep-alive>
    
            </div>
        </main>
         <v-footer></v-footer>
    </div>
    <script src="/assets/js/uikit.min.js"></script>
    <script src="/assets/js/uikit-icons.min.js"></script>
    
    
    <script src="/js/manifest.js?id=3c768977c2574a34506e" defer></script>
    <script src="/js/vendor.js?id=30747c22e0c342961509" defer></script>
    <script src="/js/app.js?id=dd817c677b4b20596aae" defer></script>
    </body>
    </html>
    <script id="f5_cspm">(function(){var f5_cspm={f5_p:'DKHAIFICBPONDAJBBMPOEIMNFCCMJCFABGMNPHJBOCLPNNOEDOCHLDCCIEADOOGAMGBBLGCOAADOFMPFNGCAOMAIAAGJCNJGJJKGOHMHNJGKGFCGAOEBCMMAENEDMLDO',setCharAt:function(str,index,chr){if(index>str.length-1)return str;return str.substr(0,index)+chr+str.substr(index+1);},get_byte:function(str,i){var s=(i/16)|0;i=(i&15);s=s*32;return((str.charCodeAt(i+16+s)-65)<<4)|(str.charCodeAt(i+s)-65);},set_byte:function(str,i,b){var s=(i/16)|0;i=(i&15);s=s*32;str=f5_cspm.setCharAt(str,(i+16+s),String.fromCharCode((b>>4)+65));str=f5_cspm.setCharAt(str,(i+s),String.fromCharCode((b&15)+65));return str;},set_latency:function(str,latency){latency=latency&0xffff;str=f5_cspm.set_byte(str,40,(latency>>8));str=f5_cspm.set_byte(str,41,(latency&0xff));str=f5_cspm.set_byte(str,35,2);return str;},wait_perf_data:function(){try{var wp=window.performance.timing;if(wp.loadEventEnd>0){var res=wp.loadEventEnd-wp.navigationStart;if(res<60001){var cookie_val=f5_cspm.set_latency(f5_cspm.f5_p,res);window.document.cookie='f5avr0875602566aaaaaaaaaaaaaaaa_cspm_='+encodeURIComponent(cookie_val)+';path=/;'+'';}
    return;}}
    catch(err){return;}
    setTimeout(f5_cspm.wait_perf_data,100);return;},go:function(){var chunk=window.document.cookie.split(/\s*;\s*/);for(var i=0;i<chunk.length;++i){var pair=chunk[i].split(/\s*=\s*/);if(pair[0]=='f5_cspm'&&pair[1]=='1234')
    {var d=new Date();d.setTime(d.getTime()-1000);window.document.cookie='f5_cspm=;expires='+d.toUTCString()+';path=/;'+';';setTimeout(f5_cspm.wait_perf_data,100);}}}}
    f5_cspm.go();}());</script>
    ✓ • 0ms

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/js/app.js | rg -o '.{0,100}(plenaryst|pl-sten|committee-sten|api/v1).{0,180}' | head -n 45
    ,e.Pl_Sten_id>0?t._t("default",[a("em",[a("router-link",{attrs:{to:{path:"/"+t.$route.params.lang+"/plenaryst/ns/55/ID/"+e.Pl_Sten_id},title:"Пленарно заседание"}},[t._v("\n                                            Пленарно заседание\n                                        ")])],1)]):t
    on(e){return a("li",{key:e.L_Act_A_id},[a("router-link",{attrs:{to:{path:"/"+t.$route.params.lang+"/plenaryst/ns/55/ID/"+e.Pl_Sten_id},title:" Пленарна зала"}},[t._v("\n                                    "+t._s(t._f("moment")(e.L_Act_St_date,"DD/MM/YYYY"))+"\n
    nted:function(){this.getSteno(this.stId)},methods:{getSteno:function(t){var e=this;t>0&&axios.get("/pl-sten/".concat(t)).then((function(t){e.plSten=t.data,setTimeout((function(){e.loading=!1}),500)}))}}},Xt=Object(c.a)(Kt,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return
    rchive",{attrs:{model:"Pl_StenV",index:"0",index2:"0",title:"Стенограма от пленарно заседание",url:"plenaryst/ns/55/ID/"}})],1)],1)}),[],!1,null,"649ab6f4",null).exports,Jt={components:{Spin:o.a,Bc:v.a},props:["colT","url"],data:function(){return{loading:!0,meta:"",app_title:"2",filterFn:
    t:j,props:{url:"gdpr"}},{path:"gdpr/:id",name:"GDPR profile",component:j,props:{url:"gdpr"}},{path:"plenaryst",name:"Steno",component:Wt,props:{url:"plenaryst"}},{path:"doc",name:"PlDoc",component:Oe,props:{url:"doc"}},{path:"doc/ID/:id",name:"PlDoc profile",component:Oe,props:{url:"doc"}
    },{path:"plenaryst/ns/:ns/ID/:id",name:"Steno profile",component:Wt,props:{url:"plenaryst"}},{path:"parliamentarycontrol",name:"PC control",component:vt,props:{url:"parliamentarycontrol"}},{path:"parliamentarycontrol/ID/:id",name:"PC control profile",component:vt,props
    ((function(t,e,a){var s=t.params.lang;s="en"==s?"en":"bg",La.locale=s,a()})),E.a.defaults.baseURL="/api/v1";new Vue({el:"#app",i18n:La,router:oa})},buo3:function(t,e,a){(t.exports=a("I1BE")(!1)).push([t.i,".ps-reg[data-v-bd1b6940]{height:320px}",""])},c68J:function(t,e,a){"use strict";
    ✓ • 0ms

## Activity

    Opened https://www.parliament.bg/api/v1/pl-sten/11050

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/js/app.js | rg -o '.{0,140}(getSteno|Pl_StenV|Co_Sten|/com-|/pl-st|committee.*sten).{0,250}' | head -n 32
    e1_2":"Parliamentary groups;","adr_line1_3":"MPs\' offices;","Address2":"Sofia 1169, 1 Knyaz Alexander I Sq. ","adr_line2_0":"Parliamentary committees;","adr_line2_1":"Reception for citizens;","adr_line2_2":"Providing access to official information;","adr_line2_3":"Procurement;","contacts":"Contacts","phone-main":"+ 359 2 939 39","esf":"European Social Fund","opdu":"Operational programme Good Governance"},"misc":{"m-order":"Order by","m-to-date":"To date","m-to":"to","member_mp":"of Parliament","with":"with","menu":"Menu","to-mandate":"during the entire term","search-criteria":"Please, enter search criteria","pg-group":"Ceased to exist on 05.07.2024 (ROPNA, art. 12, paragraph 3)","pg-group_3577":"Existed until 20.03.2025 (ROPNA, art. 12, paragraph 3)"},"index":{"archiveNews":"Archive news","archiveVideo":"Archive video","live":"LIVE","nolive":"There is currently no live broadcast","plenary":"Plenary sitting","plenaryS":"Plenary sittings","news":"News","actual":"Upcoming events","plenaryHall":"Plenary hall","currentNSid":"59","currentNS":"49th National Assembly","legislation":"Legislation","bills":"Bills","laws":"Laws","doss":"Laws related to EU law","nato1":"Spring session of the NATO parliamentary assembly 2024","nato2":"Spring session of the NATO parliamentary assembly 24-27 May 2024","ceipd1":"Bulgarian Presidency of the CEI Parliamentary Dimension","ceipd2":"CEI PD Parliamentary Committee meeting, Sofia, 16-17 May 2022 ","ceipd3":"CEI PD Parliamentary Assembly, Sofia, 14-15 November 2022"},"col2":"Parliamentary Groups","col3":"Standing Parliamentary Committees","col31":"Parliamentary Committees","col4":"Ad Hoc Committees","col6":"Parliamentary Delegations","col9":"Parliamentary Subcommittees","col11":"Friendship groups","col14":"Public Council","col_adm":"Administration","col_adr":"Address","SiteTitleLabel":"National Assembly of the Republic of Bulgaria","SiteTitleLabelHeader":"National Assembly of the Republic of Bulgaria","HomeLabel":"National Assembly of the Republic of Bulgaria - Home","SearchLabel":"search","Home":"Home","UpcomingEvents":"Upcoming events","UpcomingSit":"Upcoming sittings","UpcomingSitWeek":"All of the current week","QuickLinks":"Quick Links","ChairmanHome":" President of National Assembly","videoArchivePl":"Plenary sittings archive","videoArchiveCm":"Committees sittings archive","VideoLabel":"Videos","PlSittingsLabel":"Plenary sittings","PlProgramLabel":"Plenary sitting","PlProgramLabelLong":"Plenary sitting","CmSittingsLabel":"Committees sitting","CmSittingsLabelTitle":"Committees sitting","CmSittingsLabel1":"Sitting of ","CmSittingsLabel2":"Sitting of committee","CmSittingsDocuments":"Documents","CmSittingsHall":"Hall","CmSittingsSteno":"Стенограма от заседания на комисия","CmSittingsReport":"Report","CmSittingsBills":"Bills","CmSittingsMPsAt":"Attended MPs","PGroupBills":"Bills by parliamentary group","ControlLabel":"Parliamentary control","IncomingLabel":"Upcoming events","LatestNewsLabel":"Latest news","LatestPlProgramLabel":"Latest sitiings","MPs":"Members of Parliament","ToDate":"to date","ToDate2":"present","MPInfo":"Profile","PGinfo":"The Members of the National Assembly may form Parliamentary Groups by party or political affiliation. The minimum number of Members required to form a Parliamentary Group is ten. No Member of the National Assembly may be a member of more than one Parliamentary Group. The terms of group membership, the commencement and termination thereof, are established by the Parliamentary Group concerned and in accordance with the provisions of the Rules of Organisation and Procedure of the National Assembly, Chapter Four: Parliamentary Groups. .","DLinfo":"","CMinfo":"The National Assembly elects from among its members Standing and Ad hoc Committees.The Standing Committees assist the activity of the National Assembly and exercise on its behalf parliamentary control.The Ad hoc Committees are elected on a particular occasion to investigate specific matters and to conduct inquiries. The Committees consider bills, draft resolutions, declarations and addresses, and prepare reports, make recommendations and give opinions thereon. Committee meetings allow for participation representatives of state, industrial or public organisations, scientists and experts besides the movers of the bills. The National Assembly may alter the type, number and composition of the Standing Committees. The composition of the Standing Committees is based on the principle of proportional representation of the Parliamentary Groups. ","CM1":"Standing Committees","CM2":"Subcommittees","CM3":"Ad Hoc Committees","CM_created":"Created on","CM_des_start":"","CM_des_end":"","MPPB":"Presiding body","MP1":"Date of birth ","MP2":"Profession","MP3":"Languages","MP4":"Marital status","MP5":"Научни звания","MP6":"Political force","MP7":"Member of the previos NA","MP8":"Constituency","MP9":"Parliamentary activity","MP10":"Presented Bills","MP101":"","MP102":"","MP103":"","MP104":"","MP11":"Speeches in plenary","MP12":"Committees","MP13":"Delegations","MP14":"Friendship Groups ","MP_CV":"Curriculum vitae","MP_ab1":"Unjustified absences of parliamentary members from the plenary sittings","MP_ab2":"Unjustified absences of members of the standing committees","MP_ab4":"Member of parliament","MP_ab5":"Type","MP_ab6":"Date","MP_ab7":"No results found","MP_ab8":"Found","MP_ab9":"result(s)","MP_ab10":"Search form","MP_ab11":"From","MP_ab12":"To","MP_ab13":"Member of parliament","MP_ab14":"Constituency","MP_ab15":"Parliamentary group","MP_ab16":"Parliamentary committee","MP_ab":"Отсъствия","MainAddress":"1169 Sofia,","SearchTitle":"Search form","SearchKey":"Keyword","SearchDate1":"From","SearchDate2":"To","SearchAction":"Search","ResetAction":"Reset","SearchRes1":"Found","SearchRes2":"result(s)","SearchRes3":"No serch results","Font1":"Decrease font size","Font3":"Increase font size","ExportXML":"Export in XML","ExportXLS":"Export in XLS","ExportCSV":"Export in CSV","NewsMore":"More...","NewsTtitle":"Latest News","LatestBills":"Latest Bills","FrontLinks":"Links","MP15":"First name ","MP16":"Middle name","MP17":"Last name /Family name/","MP18":"Constituency","MP19":"Majoritarian candidate","MP20":"Parliamentary group","MP201":"Coalition","MP21":"Internet site","MP22":"Parliamentary control","MP23":"Overview","MP24":"Question to","MP25":"about","Com1":"Sittings","Com2":"News","Com3":"Information","Com4":"Documents","Com5":"Bills","Com6":"Reports","Com7":"Transcript","Com8":"Members","ComPhone":"Phone:","ComFax":"Fax:","ComStream1":"Video stream of the sitting of","ar_Name":"Name","ar_count1":"Мembers at the beginning","ar_count2":"Мembers at the end","ar_count3":"Total number","RSS_News":"News","RSS_Inc":"Upcoming events","RSS_Bills":"Bills","RSS_Decl":"Declarations","RSS_Des":"Decisions","RSS_Absent":"Unjustified absences","RSS_ComMeet":"Committees Sittings","RSS_plenary":"Plenary sittings","RSS_control":"Parliamentary control","RSS_live":"Live Parliament.bg","Vit1":"Please enter name!","Vit2":"Please enter surname!","Vit3":"Please enter last name!","Vit4":"Please enter city!","Vit5":"Please enter the symbols of the image!","Vit5a":"Please enter address!","Vit5b":"Please enter message!","Vit5c":"Please enter e-mail!","Vit6":"Name","Vit7":"Surname","Vit8":"Last name","Vit9":"City","Vit10":"Address","Vit11":"Phone","Vit12":"Mobile","Vit13":"Prefered answer","Vit14":"e-mail","Vit15":"post address","Vit16":"Message","Vit17":"Attachment","Vit18":"Please enter the symbols of the image","Vit19":"Send","Vit20":"Signal","Vit21":"Proposal","Vit21a":"Question","Vit22":"Document №","Vit23":"Access code","Vit24":"Check","Vit25":"Successful registration","Vit26":"Your registration number is","Vit27":"Your password is","STH1":"Distribution of the Members of Parliament of the  National Assembly by gender:","ST":"Members of Parliament","ST11":"Male","ST12":"Female","STH2":"Distribution of the Members of Parliament by age:","ST21":"over 60","STH3":"Distribution of the Members of Parliament by education:","ST31":"University","ST32":"High-school","STH4":"Distribution of the Members of Parliament by profession:","STH5":"Distribution of imported bills by parliamentary groups:","STH6":"Distribution of imported bills by members of Parliament:","STH7":"Plenary sittings:","STH8":"Parliamentary control:","STH9":"Committees Sittings:","STH10":"Adopted bills:","STH11":"Desisions:","STH12":"Declarations:","STH13":"Laws:","STH14":"Distribution of the Members by parliamentary experience:","GS1":"","GS3":"","Gal_overview":"All pictures","Gal_pause":"Pause","Gal_prevoius":"Previous","Gal_next":"Next","Gal_forward":"Forward","Gal_back":"Back","TV_Video":"Video","TV_Player":"Video player","TV_ExtPlayer":"External player","TV_live":"live","TV_Low":"low","TV_High":"high","TV_Part":"Part","TV_program_stream":"Real time streaming","TV_program":"Program","TV_steno":"","TV_archive":"Video archive","TV_present":"","TV_info":"Information","TV_plenary":"Plenary sitting","TV_plenary1":"Plenary sittings","TV_information":"Here you can watch the plenary sittings of the National Assembly in real time. Sittings are held on Wednesdays,Thursdays and Fridays from 9am - 2pm. The duration of the sittings could be extended by decision of the National Assembly. ","CM_program":"Заседания от комисии","CM_extplayer":"Гледайте с външен плеър","VSS_Name":"Name","VSS_Fnumber":"Filing №","VSS_FDate":"Filing date","VSS_Fimporter":"Importer","VSS_Documents":"Documents","VSS_Steno":"Shorthand protocol","VSS_Stan":"Report"}')},Z923:function(t,e,a){"use strict";var s=a("azlx");a.n(s).a},Zgzk:function(t,e,a){(t.exports=a("I1BE")(!1)).push([t.i,".ps-reg[data-v-27ab6573]{height:320px}",""])},aIuE:function(t,e,a){"use strict";var s=a("1ntf");a.n(s).a},aLFB:function(t,e,a){(t.exports=a("I1BE")(!1)).push([t.i,".ps-reg[data-v-6c83d37d]{height:320px}",""])},aTTm:function(t,e,a){var s=a("JA6M");"string"==typeof s&&(s=[[t.i,s,""]]);var n={hmr:!0,transform:void 0,insertInto:void 0};a("aET+")(s,n);s.locals&&(t.exports=s.locals)},azlx:function(t,e,a){var s=a("Vj8G");"string"==typeof s&&(s=[[t.i,s,""]]);var n={hmr:!0,transform:void 0,insertInto:void 0};a("aET+")(s,n);s.locals&&(t.exports=s.locals)},b3iz:function(t,e,a){(t.exports=a("I1BE")(!1)).push([t.i,".ps-leg[data-v-726e2b8a]{height:220px}",""])},bOpA:function(t,e,a){var s=a("MpEZ");"string"==typeof s&&(s=[[t.i,s,""]]);var n={hmr:!0,transform:void 0,insertInto:void 0};a("aET+")(s,n);s.locals&&(t.exports=s.locals)},bS7c:function(t,e,a){var s=a("fPvC");"string"==typeof s&&(s=[[t.i,s,""]]);var n={hmr:!0,transform:void 0,insertInto:void 0};a("aET+")(s,n);s.locals&&(t.exports=s.locals)},bSTZ:function(t,e,a){(t.exports=a("I1BE")(!1)).push([t.i,"@media only screen and (max-device-width:992px){.ps-nav[data-v-6fe4363a]{max-height:450px}}",""])},bUC5:function(t,e,a){"use strict";a.r(e);var s=a("jE9Z"),n=a("XuX8"),i=a.n(n),l=(a("/aIJ"),a("1tPa")),o=a("pkBR"),r={components:{Spin:o.a},props:["progId"],data:function(){return{loading:!0,plenaryProgram:{points:{}}}},watch:{progId:function(){this.getProgram(this.$route.params.lang,this.progId)},"$route.params.lang":function(){this.getProgram(this.$route.params.lang,this.progId)}},mounted:function(){this.getProgram(this.$route.params.lang,this.progId)},methods:{getProgram:function(t,e){var a=this;e=e>0?e:0,axios.get("/plenaryprogram/".concat(t,"/").concat(e)).then((function(t){a.plenaryProgram=t.data,setTimeout((function(){a.loading=!1}),500)}))},trimTime:function(t){return t.slice(0,-3)}}},c=a("KHd+"),_=Object(c.a)(r,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",[a("div",{staticClass:"mark"},[t._v("\n        "+t._s(t.$t("PlProgramLabelLong"))+"\n        "+t._s(t._f("moment")(t.plenaryProgram.Pl_Prog_date1,"DD/MM/YYYY"))+"\n        "),t.plenaryProgram.Pl_Prog_date1!=t.plenaryProgram.Pl_Prog_date2?a("span",[t._v("\n            -\n            "+t._s(t._f("moment")(t.plenaryProgram.Pl_Prog_date2,"DD/MM/YYYY"))+"\n        ")]):t._e(),t._v(" "),a("br"),t._v(" "),1!=t.plenaryProgram.CNoTime?t._t("default",[a("b-badge",{attrs:{variant:"success"}},[t._v("\n                "+t._s(t.trimTime(t.plenaryProgram.CTimeFrom))+"\n\n                "),t._v(" "),"bg"==t.$route.params.lang?t._t("default",[t._v(" ч.")]):t._e()],2)]):t._e(),t._v(" "),2==t.plenaryProgram.Pl_ProgT_id?a("div",{staticClass:"mt-3"},[a("b-badge",{attrs:{variant:"danger"}},[t._v(t._s(t.plenaryProgram.Pl_ProgTL_value)+"\n                ")])],1):t._e(),t._v(" "),2==t.plenaryProgram.Pl_ProgS_id?a("div",{staticClass:"mt-3"},[a("span",{staticClass:"text-muted text-xs font-italic"},[t._v(t._s(t.plenaryProgram.Pl_ProgSL_value)+"\n            ")])]):t._e(),t._v(" "),t.plenaryProgram.Pl_Prog_hd?a("div",[a("span",{staticClass:"text-xs font-italic"},[t._v(t._s(t.plenaryProgram.Pl_Prog_hd)+"\n            ")])]):t._e()],2),t._v(" "),t.plenaryProgram.points.length?t._e():a("div",{domProps:{innerHTML:t._s(t.plenaryProgram.Pl_ProgL_value)}}),t._v(" "),a("ol",t._l(t.plenaryProgram.points,(function(e){return a("li",{key:e.Pl_ProgP_id},[1==e.Pl_ProgP_active?a("i",{staticClass:"cis-pencil text-primary"}):t._e(),t._v(" "),e.A_ns_C_id?a("router-link",{attrs:{to:{path:"/"+t.$i18n.locale+"/parliamentarygroups/"+e.A_ns_C_id}}},[t._v("\n                Предложение по чл. 47, ал. 8 от ПОДНС от\n                "+t._s(e.A_ns_CL_value)+"\n            ")]):t._e(),t._v(" "),e.Pl_Control_id?a("router-link",{attrs:{to:{path:"/"+t.$i18n.locale+"/parliamentarycontrol/ID/"+e.Pl_Control_id},title:e.Name}},[t._v("\n                "+t._s(e.Name)+"\n            ")]):a("span",{domProps:{innerHTML:t._s(e.Name)}}),t._v(" "),a("ul",[t._l(e.acts,(function(e){return a("li",{key:e.L_Act_id},[1===e.L_Act_T_id?a("router-link",{attrs:{to:{name:"Bills profile",params:{lang:t.$route.params.lang,id:e.L_Act_id}},title:e.L_ActL_title}},[t._v("\n                        "+t._s(e.L_ActL_title)+" "+t._s(e.L_Act_sign)+"\n                    ")]):a("router-link",{attrs:{to:{name:"Acts profile",params:{lang:t.$route.params.lang,id:e.L_Act_id}},title:e.L_ActL_title}},[t._v("\n                        "+t._s(e.L_ActL_title)+" "+t._s(e.L_Act_sign)+"\n                    ")])],1)})),t._v(" "),t._l(e.documents,(function(e){return a("li",{key:e.L_ActD_id},[a("a",{attrs:{href:"/pub/plenary_documents/"+e.L_ActD_file,download:"",title:e.L_ActD_name}},[t._v("\n                        "+t._s(e.L_ActD_T_name)+" - "+t._s(e.L_ActD_name)+"\n                        "+t._s(e.L_ActD_sign))]),e.L_ActD_importer?a("span",[t._v(", "+t._s(e.L_ActD_importer)+"\n                    ")]):t._e()])}))],2)],1)})),0)])}),[],!1,null,"542b7c6e",null).exports;function m(t,e,a){return e in t?Object.defineProperty(t,e,{value:a,enumerable:!0,configurable:!0,writable:!0}):t[e]=a,t}var d=a("oVFp");window.Vue=a("XuX8"),Vue.use(d);var u={name:"Index",components:{videoPlayer:l.videoPlayer,plenaryProgram:_},data:function(){return{loading:!0,meta:"",playerOptions:{},imgPath:"",listNews:[],listLeg:[],calendar:[],listCalendar:[],liveCalendar:{plenary:[],com:[]},M_NewsMG_file:"",mainNews:[{M_NewsMG_file:"",media:{M_NewsMG_file:""}}],vaList:[],tsPreview:[],munList:[],pgList:[],coalList:[],collFilter:{},mpFilter:{},filterRules:{}}},metaInfo:function(){return{title:"".concat(this.$t("title")),meta:[{name:"description",content:"".concat(this.$t("description"))},{name:"keywords",content:"".concat(this.$t("keywords"))},{property:"og:title",content:"".concat(this.$t("title"))},{property:"og:site_name",content:"".concat(this.$t("title"))},{property:"og:url",content:"https://www.parliament.bg/"}]}},mounted:function(){var t=this;"bg"!=this.$route.params.lang&&"en"!=this.$route.params.lang||(this.getCalendar(this.$route.params.lang),this.interval=setInterval((function(){return t.getCalendar(t.$route.params.lang)}),6e4),this.getData(this.$route.params.lang),this.onPlayVideo(),this.getFilters(this.$route.params.lang))},watch:{"collFilter.vaId":function(){this.collFilter.vaId?this.mpFilter.vaId=this.collFilter.vaId.id:this.mpFilter.vaId=""},"collFilter.munId":function(){this.collFilter.munId?this.mpFilter.munId=this.collFilter.munId.id:this.mpFilter.munId=""},"collFilter.coalId":function(){this.collFilter.coalId?this.mpFilter.coalId=this.collFilter.coalId.id:this.mpFilter.coalId=""},"collFilter.pgId":function(){this.collFilter.pgId?this.mpFilter.pgId=this.collFilter.pgId.id:this.mpFilter.pgId=""}},computed:{},methods:{getCalendar:function(t){var e=this;axios.get("/calendar/".concat(t)).then((function(t){e.calendar=t.data,setTimeout((function(){e.loading=!1}),500)}))},getData:function(t){var e=this;axios.get("/front-news/".concat(t,"/1")).then((function(t){e.mainNews=t.data,setTimeout((function(){e.loading=!1}),500)})),axios.get("/ts-preview/".concat(t)).then((function(t){e.tsPreview=t.data,setTimeout((function(){e.loading=!1}),500)})),axios.get("/front-news/".concat(t,"/5")).then((function(t){e.listNews=t.data,setTimeout((function(){e.loading=!1}),500)})),axios.get("/front-act-list").then((function(t){e.listLeg=t.data,setTimeout((function(){e.loading=!1}),500)})),axios.get("/calendar-list/".concat(t)).then((function(t){e.listCalendar=t.data,setTimeout((function(){e.loading=!1}),500)})),axios.get("/calendar-live/".concat(t)).then((function(t){e.liveCalendar=t.data,setTimeout((function(){e.loading=!1}),500)}))},onPlayerPlay:function(t){},onPlayerPause:function(t){},onPlay1:function(){this.playerOptions.poster=""},trimTime:function(t){return t.slice(0,-3)},onPlayVideo:function(){this.playerOptions={width:"450px",height:"300px",muted:!0,language:"en",playbackRates:[.7,1,1.5,2],sources:[{type:"application/x-mpegurl",src:"https://e205-ts.cdn.bg/parliament/fls/zplenarna.stream/playlist.m3u8"}],poster:"/media/bg/player_cover.jpg"},this.player},playerStateChanged:function(t){},playerReadied:function(t){},reset:function(){this.collFilter.munId="",this.collFilter.vaId="",this.collFilter.pgId="",this.collFilter.coalId=""},getFilters:function(t){var e=this;axios.get("/va-list").then((function(t){e.vaList=t.data,setTimeout((function(){e.loading=!1}),500)})),axios.get("/mun-list/0").then((function(t){e.munList=t.data,setTimeout((function(){e.loading=!1}),500)})),axios.get("/coll-list/".concat(t,"/2")).then((function(t){e.pgList=t.data,setTimeout((function(){e.loading=!1}),500)})),axios.get("/coal-list/".concat(t)).then((function(t){e.coalList=t.data,setTimeout((function(){e.loading=!1}),500)}))},listMps:function(){var t;this.$router.push({name:"MP list",params:{},query:(t={pgId:this.mpFilter.pgId},m(t,"pgId",this.mpFilter.pgId),m(t,"munId",this.mpFilter.munId),m(t,"vaId",this.mpFilter.vaId),m(t,"coalId",this.mpFilter.coalId),m(t,"name1",this.mpFilter.A_ns_MPL_Name1),m(t,"name2",this.mpFilter.A_ns_MPL_Name2),m(t,"name3",this.mpFilter.A_ns_MPL_Name3),t)})}}},p=(a("4bwt"),Object(c.a)(u,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",{staticClass:"p-container"},["bg"==t.$route.params.lang||"en"==t.$route.params.lang?a("div",[a("b-row",{staticClass:"p-container-front mt-5"},[a("b-col",{staticClass:"p-block news col-sm-12  col-12 col-lg-6 col-md-6",attrs:{cols:"12"}},[t.mainNews.media?a("div",{staticClass:"content layer",style:{background:"url(\n                            "+t.imgPath+t.mainNews.media.M_NewsMG_file+"\n                        )"}},[t._v("\n                     \n                    "),a("div",{staticClass:"p-news-title"},[a("div",{staticClass:"text"},[a("router-link",{attrs:{title:t.mainNews.M_NewsL_title,to:{name:"News profile",params:{lang:t.$route.params.lang,id:t.mainNews.M_News_id}}}},[t._v("\n                                "+t._s(t.mainNews.M_NewsL_title)+"\n                            ")])],1)])]):t._e(),t._v(" "),a("div",{staticClass:"content-footer"},[a("router-link",{attrs:{title:t.$t("index.archiveNews"),to:{name:"News",params:{lang:t.$route.params.lang}}}},[t._v("\n                        "+t._s(t.$t("index.archiveNews"))+"\n                        "),a("i",{staticClass:"cis-newspaper"})])],1)]),t._v(" "),a("b-col",{staticClass:"p-block video  col-sm-12  col-12 col-lg-6 col-md-6",attrs:{cols:"12"}},[a("div",{staticClass:"content"},[a("div",{staticClass:"layer"},[a("div",{staticClass:"live float-right"},[a("i",{staticClass:"cis-circle text-danger"}),t._v("\n                            "+t._s(t.$t("index.live"))+"\n                        ")]),t._v(" "),a("div",{staticClass:"clearfix"}),t._v(" "),a("perfect-scrollbar",{staticClass:"ps-live"},t._l(t.calendar,(function(e){return a("div",{key:e.Cid},[e.CIndex>0&&e.details?t._t("default",[e.details.VS_ID?a("div",{staticClass:"live-item"},[a("span",{staticClass:"dt"},[t._v("\n                                            "+t._s(t._f("moment")(e.CDate,"DD/MM/YYYY"))+"\n                                        ")]),t._v(" "),a("i",{staticClass:"cil-calendar-today"}),t._v(" "),a("span",{staticClass:"dt"},[1!=e.CNoTime?t._t("default",[t._v("\n                                                "+t._s(t.trimTime(e.CTimeFrom))+"\n                                                "),a("i",{staticClass:"cil-clock",class:{"text-danger":1==e.active}})]):t._e()],2),t._v(" "),"pl"==e.CType?a("div",[e.active?a("router-link",{attrs:{title:t.$t("index.plenary"),to:{name:"Video",params:{lang:t.$route.params.lang}}}},[t._v("\n                                                "+t._s(t.$t("index.plenary"))+"\n                                            ")]):a("div",[t._v("\n                                                "+t._s(t.$t("index.plenary"))+"\n                                            ")])],1):a("div",[e.active?a("router-link",{attrs:{title:e.details.title,to:{name:"Video",params:{lang:t.$route.params.lang}}}},[t._v("\n                                                "+t._s(e.details.title)+"\n                                            ")]):a("div",[t._v("\n                                                "+t._s(e.details.title)+"\n                                            ")])],1)]):t._e()]):t._e()],2)})),0)],1)]),t._v(" "),a("div",{staticClass:"content-footer"},[a("router-link",{attrs:{title:t.$t("index.archiveVideo"),to:{name:"Video",params:{lang:t.$route.params.lang}}}},[t._v("\n                        "+t._s(t.$t("index.archiveVideo"))+"\n                        "),a("i",{staticClass:"cis-movie"})])],1)])],1),t._v(" "),a("b-row",{staticClass:"p-container-front"},[a("b-col",{staticClass:"p-panel-title"},[t._v("\n                "+t._s(t.$t("index.news"))+"\n            ")])],1),t._v(" "),a("b-row",{staticClass:"p-container-front mt-0"},t._l(t.listNews,(function(e){return a("b-col",{key:e.M_News_id,staticClass:"p-panel p-panel-pl"},[a("router-link",{attrs:{title:e.M_NewsL_title,"aria-label":e.M_NewsL_title,to:{name:"News profile",params:{lang:t.$route.params.lang,id:e.M_News_id}}}},[a("div",{staticClass:"p-image",style:{background:"url(\n                            "+t.imgPath+e.media.M_NewsMG_file+"\n                        )"}})]),t._v(" "),a("div",{staticClass:"p-title"},[a("router-link",{attrs:{to:{name:"News profile",params:{lang:t.$route.params.lang,id:e.M_News_id}},title:e.M_NewsL_title}},[t._v("\n                        "+t._s(e.M_NewsL_title)+"\n                    ")])],1)],1)})),1),t._v(" "),a("b-row",{staticClass:"p-container-front"},[a("b-col",{staticClass:"p-panel-title"},[t._v("\n                "+t._s(t.$t("index.actual"))+"\n            ")])],1),t._v(" "),a("b-row",{staticClass:"p-container-front mt-0"},[a("b-col",{staticClass:"p-panel event  p-panel-pl"},[a("div",{staticClass:"p-header act1"},[a("br"),a("br"),a("br"),t._v(" "),a("div",{staticClass:"p-event-title"},[t._v("\n                        "+t._s(t.$t("index.plenaryHall"))+"\n                    ")])]),t._v(" "),a("div",{staticClass:"p-event-content"},[a("perfect-scrollbar",{staticClass:"ps-cal"},[a("ul",{staticClass:"p-common-list"},t._l(t.listCalendar.plenary,(function(e){return a("li",{key:e.cid},[a("router-link",{attrs:{title:t.$t("index.plenary"),to:{path:"/"+t.$route.params.lang+"/plenaryprogram/ID/"+e.id}}},[t._v("\n                                    "+t._s(t._f("moment")(e.CDate,"DD/MM/YYYY"))+"\n                                    "),1!=e.CNoTime?t._t("default",[t._v("\n                                        "+t._s(t.trimTime(e.CTimeFrom))+"\n                                    ")]):t._e(),t._v(" "),a("br"),t._v("\n                                    "+t._s(t.$t("index.plenary"))+"\n                                ")],2)],1)})),0)])],1)]),t._v(" "),a("b-col",{staticClass:"p-panel event p-panel-pl"},[a("div",{staticClass:"p-header act2"},[a("br"),a("br"),a("br"),t._v(" "),a("div",{staticClass:"p-event-title"},[t._v("\n                        "+t._s(t.$t("CmSittingsLabel"))+"\n                    ")])]),t._v(" "),a("div",{staticClass:"p-event-content"},[a("perfect-scrollbar",{staticClass:"ps-cal"},[a("ul",{staticClass:"p-common-list"},t._l(t.listCalendar.com,(function(e){return a("li",{key:e.cid},[a("router-link",{attrs:{title:e.title,to:{path:"/"+t.$route.params.lang+"/committees-sitting"}}},[t._v("\n                                    "+t._s(t._f("moment")(e.CDate,"DD/MM/YYYY"))+"\n                                    "+t._s(t.trimTime(e.CTimeFrom))+"\n\n                                    "),e.VS_ID?a("i",{staticClass:"cis-video"}):t._e(),t._v(" "),a("br"),t._v("\n                                    "+t._s(e.title)+"\n                                ")])],1)})),0)])],1)]),t._v(" "),a("b-col",{staticClass:"p-panel event  p-panel-pl"},[a("div",{staticClass:"p-header act3"},[a("br"),a("br"),a("br"),t._v(" "),a("div",{staticClass:"p-event-title"},[t._v("\n                        "+t._s(t.$t("IncomingLabel"))+"\n                    ")])]),t._v(" "),a("div",{staticClass:"p-event-content"},[a("perfect-scrollbar",{staticClass:"ps-cal"},[a("ul",{staticClass:"p-common-list"},t._l(t.listCalendar.incoming,(function(e){return a("li",{key:e.cid},[a("router-link",{attrs:{title:e.title,to:{path:"/"+t.$route.params.lang+"/calendar"}}},[t._v("\n                                    "+t._s(t._f("moment")(e.CDate,"DD/MM/YYYY"))+"\n                                    "+t._s(t.trimTime(e.CTimeFrom))+"\n                                    "),e.VS_ID?a("i",{staticClass:"cis-video"}):t._e(),t._v(" "),a("br"),t._v("\n                                    "+t._s(e.title)+"\n                                ")])],1)})),0)])],1)])],1),t._v(" "),t.liveCalendar.plenary.length&&"bg"==t.$route.params.lang?a("b-row",{staticClass:"p-container-front i18n-l"},[a("b-col",{staticClass:"p-panel-title"},[t._v("\n                "+t._s(t.$t("index.plenaryS"))+"\n            ")])],1):t._e(),t._v(" "),t.liveCalendar.plenary.length&&"bg"==t.$route.params.lang?a("b-row",{staticClass:"p-container-front mt-0"},[a("b-col",{staticClass:"p-panel plenary"},[a("div",{staticClass:"p-header-plenary"},[a("div",{staticClass:"live float-left"},[t._v("\n                        Пленарно заседание\n                    ")])]),t._v(" "),a("b-row",[a("b-col",{staticClass:"stream col-sm-12  col-12 col-lg-6 col-md-6 p-panel-pl"},[a("video-player",{ref:"videoPlayer",staticClass:"video-player-box",attrs:{"aria-label":"Пленарно заседание",title:"Пленарно заседание",options:t.playerOptions,playsinline:!0,customEventName:"customstatechangedeventname"},on:{play:function(e){return t.onPlayerPlay(e)},pause:function(e){return t.onPlayerPause(e)},statechanged:function(e){return t.playerStateChanged(e)},ready:t.playerReadied}})],1),t._v(" "),a("b-col",{staticClass:"program col-sm-12  col-12 col-lg-6 col-md-6 p-panel-pl"},[a("perfect-scrollbar",{staticClass:"ps-program"},[a("plenary-program",{attrs:{progId:t.liveCalendar.plenary[0].id}})],1)],1)],1)],1)],1):t._e(),t._v(" "),"bg"==t.$route.params.lang?a("b-row",{staticClass:"p-container-front"},[a("b-col",{staticClass:"p-panel-title"},[t._v("\n                "+t._s(t.$t("index.legislation"))+"\n            ")])],1):t._e(),t._v(" "),"bg"==t.$route.params.lang?a("b-row",{staticClass:"p-container-front mt-0"},[a("b-col",{staticClass:"p-panel legislation l-1  p-panel-pl"},[a("div",{staticClass:"p-body leg-1"},[a("br"),t._v(" "),a("div",{staticClass:"p-leg-title"},[a("span",{staticClass:"pl-3"},[a("router-link",{attrs:{to:{name:"Bills",params:{lang:t.$route.params.lang}},title:t.$t("index.bills")}},[t._v("\n                                "+t._s(t.$t("index.bills"))+"\n                            ")])],1),t._v(" "),a("br"),a("br")]),t._v(" "),a("div",{staticClass:"content"},[a("perfect-scrollbar",{staticClass:"ps-leg"},[a("ul",t._l(t.listLeg.bills,(function(e){return a("li",{key:e.L_Act_id},[a("router-link",{attrs:{to:{name:"Bills profile",params:{lang:t.$route.params.lang,id:e.L_Act_id}},title:e.L_ActL_title}},[t._v("\n                                        "+t._s(e.L_ActL_title)+"\n                                    ")])],1)})),0)])],1)])]),t._v(" "),a("b-col",{staticClass:"p-panel legislation l-2  p-panel-pl"},[a("div",{staticClass:"p-body leg-2"},[a("br"),t._v(" "),a("div",{staticClass:"p-leg-title"},[a("span",{staticClass:"pl-3"},[a("router-link",{attrs:{to:{name:"Law",params:{lang:t.$route.params.lang}},title:t.$t("index.laws")}},[t._v("\n                                "+t._s(t.$t("index.laws"))+"\n                            ")])],1),t._v(" "),a("br"),a("br")]),t._v(" "),a("div",{staticClass:"content"},[a("perfect-scrollbar",{staticClass:"ps-leg"},[a("ul",t._l(t.listLeg.laws,(function(e){return a("li",{key:e.L_Act_id},[a("router-link",{attrs:{to:{name:"Law profile",params:{lang:t.$route.params.lang,id:e.L_Act_id}},title:e.L_ActL_title}},[t._v("\n                                        "+t._s(e.L_ActL_title)+"\n                                    ")])],1)})),0)])],1)])]),t._v(" "),a("b-col",{staticClass:"p-panel legislation l-3  p-panel-pl"},[a("div",{staticClass:"p-body leg-3"},[a("br"),t._v(" "),a("div",{staticClass:"p-leg-title"},[a("span",{staticClass:"pl-3"},[a("router-link",{attrs:{to:{name:"euDossier",params:{lang:t.$route.params.lang}},title:t.$t("index.doss")}},[t._v("\n                                "+t._s(t.$t("index.doss"))+"\n                            ")])],1),t._v(" "),a("br"),a("br")]),t._v(" "),a("div",{staticClass:"content"},[a("perfect-scrollbar",{staticClass:"ps-leg"},[a("ul",t._l(t.listLeg.trans,(function(e){return a("li",{key:e.ID},[a("router-link",{attrs:{to:{name:"euDossier profile",params:{lang:t.$route.params.lang,id:e.ID}},title:e.dName}},[t._v("\n                                        "+t._s(e.dName)+"\n                                    ")])],1)})),0)])],1)])])],1):t._e(),t._v(" "),"bg"==t.$route.params.lang&&t.tsPreview.length>0?t._t("default",[a("b-row",{staticClass:"p-container-front"},[a("b-col",{staticClass:"p-panel-title"},[t._v("\n                    Изборни процедури\n                ")])],1),t._v(" "),a("b-row",{staticClass:"mt-0"},[a("b-col",{staticClass:"p-th th-1  p-panel-pl"},[a("ol",{staticClass:" card-columns card-list"},t._l(t.tsPreview,(function(e){return a("li",{key:e.Cts_id},[a("router-link",{attrs:{to:{path:"/bg/theme-site/ID/"+e.Cts_id},title:e.CtsL_value}},[t._v("\n                                "+t._s(e.CtsL_value)+"\n                            ")])],1)})),0)])],1)]):t._e(),t._v(" "),"bg"==t.$route.params.lang?a("b-row",{staticClass:"p-container-front"},[a("b-col",{staticClass:"p-panel-title"},[t._v("\n                Регистри\n            ")])],1):t._e(),t._v(" "),"bg"==t.$route.params.lang?a("b-row",{staticClass:"mt-0"},[a("b-col",{staticClass:"p-registers reg-1  p-panel-pl"},[a("router-link",{attrs:{to:{name:"Referendum",params:{lang:t.$route.params.lang}},title:"Регистър по Закона за пряко участие на\n                            гражданите в държавната власт и местното\n                            самоуправление"}},[a("div",{staticClass:"title"},[a("div",{staticClass:"pl-3"},[t._v("\n                            Регистър по Закона за пряко участие на\n                            гражданите в държавната власт и местното\n                            самоуправление\n                        ")])])])],1),t._v(" "),a("b-col",{staticClass:"p-registers reg-2  p-panel-pl"},[a("router-link",{attrs:{to:{name:"Register conflict",params:{lang:t.$route.params.lang}},title:"Регистър по чл. 82, ал. 1 от Закона за Сметната палата"}},[a("div",{staticClass:"title"},[a("div",{staticClass:"pl-3"},[t._v("\n                            Регистър по чл. 82, ал. 1 от Закона за
    ... command output truncated for persistence ...
    t._v("Код 94 - Откриване на акредитив")])]),t._v(" "),a("li",[a("span",[t._v("\n                            Код 95 - Преводи чрез директен дебит и по\n                            инициатива на БНБ\n                        ")])]),t._v(" "),a("li",[a("span",[t._v("\n                            Код 96 - Преводи по нареждане на титуляра с\n                            други платежни документи\n                        ")])]),t._v(" "),a("li",[a("span",[t._v("\n                            Код 97 - Операции по откриване/закриване на\n                            депозити\n                        ")])]),t._v(" "),a("li",[a("span",[t._v("От Код 98 до Код 99 - Служебни операции в\n                            БНБ")])])])])],1),t._v(" "),a("div",{staticClass:"clearfix"}),t._v(" "),a("div",{attrs:{id:"doc-list"}})],1),t._v(" "),a("b-row",{staticClass:"p-archive-period"},[a("b-col",[a("div",{ref:"period-title",staticClass:"mt-4 mb-3"},[t.setYear?a("span",{staticClass:"p-archive-title"},[t._v("\n                    "+t._s(t.setMonth)+" "+t._s(t.setYear)+"\n                ")]):t._e()]),t._v(" "),a("ul",{staticClass:"p-common-list"},t._l(t.periodBulletin,(function(e,s){return a("li",{key:s},[a("a",{attrs:{href:e.Sb_file,download:"",title:t.Изтегли}},[t._v("\n                        Файл от дата\n                        "+t._s(t._f("moment")(e.Sb_date,"DD/MM/YYYY"))+"\n                    ")])])})),0)])],1),t._v(" "),a("b-row",{staticClass:"p-archive-list"},t._l(t.archiveYear,(function(e){return a("b-col",{key:e.t_year,staticClass:"col-12 col-sm-6 col-lg-2 col-md-2"},[a("div",{staticClass:"archive-head"},[t._v("\n                "+t._s(e.t_year)+"\n            ")]),t._v(" "),a("ul",t._l(e.t_month_list,(function(s){return a("li",{key:s.t_month},[a("span",{directives:[{name:"scroll-to",rawName:"v-scroll-to",value:"#doc-list",expression:"'#doc-list'"}],attrs:{title:s.C_18n_value,"aria-label":s.C_18n_value},on:{click:function(a){t.periodList(e.t_year,s.t_month),t.setYear=e.t_year,t.setMonth=s.C_18n_value}}},[t._v("\n                        "+t._s(s.C_18n_value)+"\n                    ")])])})),0)])})),1)],1)}),[],!1,null,"7e5899aa",null).exports,qe={components:{Spin:o.a,Bc:v.a},props:["url"],data:function(){return{loading:!0,meta:"",app_title:"2",filterFn:"",fnSession:[],currentPage:1,perPage:30,totalRows:0,sortBy:null,sortDesc:!1,fnFields:[{key:"index",label:"№",sortable:!0},{key:"A_ns_Rg_num",label:"Рег. №",sortable:!0},{key:"A_ns_Rg_date",label:"Дата",sortable:!0},{key:"A_ns_Rg_info",label:"Вид на документа",sortable:!0}],fnResult:[]}},metaInfo:function(){return{title:"".concat(this.meta.C_NavL_value," - ").concat(this.$t("title")),meta:[{name:"description",content:"".concat(this.$t("description"))},{name:"keywords",content:"".concat(this.$t("keywords"))},{property:"og:title",content:"".concat(this.meta.C_NavL_value," - ").concat(this.$t("title"))},{property:"og:site_name",content:"".concat(this.$t("title"))},{property:"og:url",content:"https://www.parliament.bg/".concat(this.meta.C_Lang_string,"/").concat(this.url)}]}},mounted:function(){var t=this;this.getData(),this.$services.meta.getMeta(this.$route.params.lang,this.url,(function(e){t.meta=e}))},methods:{getData:function(){var t=this;axios.get("/reg35").then((function(e){t.fnResult=e.data,setTimeout((function(){t.loading=!1}),500)}))}}},Ge=Object(c.a)(qe,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",[a("bc",{attrs:{path:t.meta}}),t._v(" "),a("h1",{staticClass:"p-container-title"},[t._v(t._s(t.meta.C_NavL_value))]),t._v(" "),a("div",[a("b-form-group",{staticClass:"mt-3",attrs:{label:"Търсене","label-for":"filter-input","label-cols-sm":"3","label-align-sm":"right","label-size":"sm"}},[a("b-form-input",{attrs:{id:"filter-input",title:"Търсене","aria-label":"Търсене",type:"search",placeholder:""},model:{value:t.filterFn,callback:function(e){t.filterFn=e},expression:"filterFn"}})],1),t._v(" "),a("b-table",{attrs:{striped:"",hover:"",borderless:"",outlined:"","head-variant":"unset",responsive:"sm","current-page":t.currentPage,"per-page":t.perPage,filter:t.filterFn,items:t.fnResult,fields:t.fnFields,"sort-by":t.sortBy,"sort-desc":t.sortDesc},on:{"update:sortBy":function(e){t.sortBy=e},"update:sort-by":function(e){t.sortBy=e},"update:sortDesc":function(e){t.sortDesc=e},"update:sort-desc":function(e){t.sortDesc=e}},scopedSlots:t._u([{key:"cell(index)",fn:function(e){return[t._v("\n                "+t._s(e.index+1)+"\n            ")]}},{key:"cell(A_ns_Rg_date)",fn:function(e){return[t._v("\n                "+t._s(t._f("moment")(e.item.A_ns_Rg_date,"DD/MM/YYYY"))+"\n            ")]}}])}),t._v(" "),a("b-pagination",{staticClass:"my-0",attrs:{"total-rows":t.fnResult.length,"per-page":t.perPage,limit:30,size:"sm"},model:{value:t.currentPage,callback:function(e){t.currentPage=e},expression:"currentPage"}})],1)],1)}),[],!1,null,"318988c8",null).exports,He={components:{Spin:o.a,Archive:b.a,LightBox:G.a,Bc:v.a},props:["colT","url"],data:function(){return{loading:!0,meta:"",app_title:"2",imgPath:"",galEvent:{gallery:[]},archiveNews:[]}},metaInfo:function(){return{title:"".concat(this.meta.C_NavL_value," - ").concat(this.$t("title")),meta:[{name:"description",content:"".concat(this.$t("description"))},{name:"keywords",content:"".concat(this.$t("keywords"))},{property:"og:title",content:"".concat(this.meta.C_NavL_value," - ").concat(this.$t("title"))},{property:"og:site_name",content:"".concat(this.$t("title"))},{property:"og:url",content:"https://www.parliament.bg/".concat(this.meta.C_Lang_string,"/").concat(this.url)}]}},watch:{"$route.params.lang":function(){var t=this;this.getProfile(this.$route.params.lang,this.$route.params.id,this.colT),this.$services.meta.getMeta(this.$route.params.lang,this.url,(function(e){t.meta=e}))}},mounted:function(){var t=this;this.$services.meta.getMeta(this.$route.params.lang,this.url,(function(e){t.meta=e})),this.$route.params.id&&this.getData(this.$route.params.lang,this.$route.params.id)},methods:{getData:function(t,e){var a=this;axios.get("/gallery/".concat(t,"/").concat(e)).then((function(t){a.galEvent=t.data,setTimeout((function(){a.loading=!1}),500)}))},openGallery:function(t){this.$refs.lightbox.showImage(t)}}},Ue=Object(c.a)(He,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",[a("bc",{attrs:{path:t.meta}}),t._v(" "),a("h1",{staticClass:"p-container-title"},[t._v(t._s(t.meta.C_NavL_value))]),t._v(" "),a("div",[t.$route.params.id?a("b-row",[a("b-col",[a("b-row",[a("b-col",[a("div",{staticClass:"p-media-title"},[t._v("\n                            "+t._s(t.galEvent.G_Gal_EL_value)+"\n                        ")]),t._v(" "),a("div",{staticClass:"p-note"},[t._v("\n                            "+t._s(t._f("moment")(t.galEvent.G_Gal_E_date,"DD/MM/YYYY"))+"\n                        ")])])],1),t._v(" "),a("b-row",t._l(t.galEvent.gallery,(function(e,s){return a("b-col",{key:s,staticClass:"mb-5",attrs:{cols:"2"}},[a("img",{directives:[{name:"lazy",rawName:"v-lazy",value:e.src,expression:"att.src"}],staticClass:"p-media-img",attrs:{src:e.src,alt:t.galEvent.G_Gal_EL_value,title:t.galEvent.G_Gal_EL_value,"aria-label":t.galEvent.G_Gal_EL_value},on:{click:function(e){return t.openGallery(s)}}})])})),1),t._v(" "),a("LightBox",{ref:"lightbox",attrs:{media:t.galEvent.gallery,showLightBox:!1,"show-caption":!0}})],1)],1):t._e(),t._v(" "),a("archive",{attrs:{model:"G_Gallery_Events",index:"0",index2:"0",title:"",url:"gallery/ID/"}})],1)],1)}),[],!1,null,"2a260c61",null).exports,ze={components:{Spin:o.a,Bc:v.a},props:["colT","url"],data:function(){return{loading:!0,meta:"",app_title:"2",imgPath:"",listCalendar:[]}},metaInfo:function(){return{title:"".concat(this.meta.C_NavL_value," - ").concat(this.$t("title")),meta:[{name:"description",content:"".concat(this.$t("description"))},{name:"keywords",content:"".concat(this.$t("keywords"))},{property:"og:title",content:"".concat(this.meta.C_NavL_value," - ").concat(this.$t("title"))},{property:"og:ince_name",content:"".concat(this.$t("title"))},{property:"og:url",content:"https://www.parliament.bg/".concat(this.meta.C_Lang_string,"/").concat(this.url)}]}},watch:{"$route.params.lang":function(){var t=this;this.getData(this.$route.params.lang),this.$services.meta.getMeta(this.$route.params.lang,this.url,(function(e){t.meta=e}))}},mounted:function(){var t=this;this.$services.meta.getMeta(this.$route.params.lang,this.url,(function(e){t.meta=e})),this.getData(this.$route.params.lang)},methods:{getData:function(t){var e=this;axios.get("/calendar-list/".concat(t)).then((function(t){e.listCalendar=t.data,setTimeout((function(){e.loading=!1}),500)}))},trimTime:function(t){return t.slice(0,-3)}}},Ke=Object(c.a)(ze,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",[a("bc",{attrs:{path:t.meta}}),t._v(" "),a("h1",{staticClass:"p-container-title"},[t._v(t._s(t.meta.C_NavL_value))]),t._v(" "),a("div",t._l(t.listCalendar.incoming,(function(e){return a("div",{key:e.Cid,staticClass:"p-obj mt-5"},[a("div",{staticClass:"p-obj-info mark"},[a("i",{staticClass:"cil-calendar-check text-success"}),t._v("\n                "+t._s(t._f("moment")(e.CDate,"DD/MM/YYYY"))+"\n                "+t._s(t.trimTime(e.CTimeFrom))+" "),a("br"),t._v(" "),a("i",{staticClass:"cis-location-pin text-primary"}),t._v("\n                "+t._s(e.title)+"\n            ")]),t._v(" "),a("div",{staticClass:"mt-2",domProps:{innerHTML:t._s(e.body)}})])})),0)],1)}),[],!1,null,"145f4846",null).exports,Xe={components:{Spin:o.a,Archive:b.a,Bc:v.a},props:["colT","url"],data:function(){return{loading:!0,meta:"",app_title:"2",imgPath:"",links:[],archiveNews:[]}},metaInfo:function(){return{title:"".concat(this.meta.C_NavL_value," - ").concat(this.$t("title")),meta:[{name:"description",content:"".concat(this.$t("description"))},{name:"keywords",content:"".concat(this.$t("keywords"))},{property:"og:title",content:"".concat(this.meta.C_NavL_value," - ").concat(this.$t("title"))},{property:"og:site_name",content:"".concat(this.$t("title"))},{property:"og:url",content:"https://www.parliament.bg/".concat(this.meta.C_Lang_string,"/").concat(this.url)}]}},watch:{"$route.params.id":function(){this.getData(this.$route.params.lang)},"$route.params.lang":function(){var t=this;this.getData(this.$route.params.lang),this.$services.meta.getMeta(this.$route.params.lang,this.url,(function(e){t.meta=e}))}},mounted:function(){var t=this;this.$services.meta.getMeta(this.$route.params.lang,this.url,(function(e){t.meta=e})),this.getData(this.$route.params.lang)},methods:{getData:function(t){var e=this;axios.get("/links/".concat(t)).then((function(t){e.links=t.data,setTimeout((function(){e.loading=!1}),500)}))}}},Ze=Object(c.a)(Xe,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",[a("bc",{attrs:{path:t.meta}}),t._v(" "),a("h1",{staticClass:"p-container-title"},[t._v(t._s(t.meta.C_NavL_value))]),t._v(" "),a("div",[a("div",{staticClass:"accordion",attrs:{role:"tablist"}},t._l(t.links,(function(e){return a("b-card",{key:e.C_LinkT_id,staticClass:"mb-1",attrs:{"no-body":""}},[a("b-card-header",{staticClass:"p-1",attrs:{"header-tag":"header",role:"tab"}},[a("b-button",{directives:[{name:"b-toggle",rawName:"v-b-toggle",value:"accordion-"+e.C_LinkT_id,expression:"'accordion-' + link.C_LinkT_id"}],staticClass:"p-acc-tab",attrs:{block:"",variant:"primary",title:e.C_LinkTL_value,"aria-label":e.C_LinkTL_value}},[t._v("\n                        "+t._s(e.C_LinkTL_value)+"\n                        "),a("i",{staticClass:"cis-chevron-bottom p-icon-t float-right"})])],1),t._v(" "),a("b-collapse",{attrs:{id:"accordion-"+e.C_LinkT_id,accordion:"my-accordion",role:"tabpanel",visible:3===e.C_LinkT_id}},[a("b-card-body",[a("b-card-text",[a("ul",{staticClass:"p-common-list"},t._l(e.tLinks,(function(s){return a("li",{key:s.C_Link_id},[a("a",{attrs:{href:s.C_Link_URL,target:"_blank",title:s.C_LinkL_value}},[7==e.C_LinkT_id?t._t("default",[t._v("\n                                            "+t._s(s.C_CountryL_value)+" -\n                                        ")]):t._e(),t._v("\n                                        "+t._s(s.C_LinkL_value)+"\n                                    ")],2)])})),0)])],1)],1)],1)})),1)])],1)}),[],!1,null,"63adc3a3",null).exports,We={components:{Spin:o.a,Bc:v.a},props:["url"],data:function(){return{loading:!0,meta:"",app_title:"2",filterFn:"",fnFields:[{key:"index",label:"№",sortable:!0},{key:"DATA_OT",label:"Дата от",sortable:!0},{key:"DATA_DO",label:"Дата до",sortable:!0},{key:"IMENA",label:"Изпълнител",sortable:!0},{key:"VAZLOJITEL_TEXT",label:"Възложител",sortable:!0},{key:"TEMA",label:"Тема",sortable:!0},{key:"SUMA",label:"Сума",sortable:!0}],currentPage:1,perPage:30,totalRows:0,sortBy:null,sortDesc:!1,fnResult:[]}},metaInfo:function(){return{title:"".concat(this.meta.C_NavL_value," - ").concat(this.$t("title")),meta:[{name:"description",content:"".concat(this.$t("description"))},{name:"keywords",content:"".concat(this.$t("keywords"))},{property:"og:title",content:"".concat(this.meta.C_NavL_value," - ").concat(this.$t("title"))},{property:"og:site_name",content:"".concat(this.$t("title"))},{property:"og:url",content:"https://www.parliament.bg/".concat(this.meta.C_Lang_string,"/").concat(this.url)}]}},mounted:function(){var t=this;this.getData(),this.$services.meta.getMeta(this.$route.params.lang,this.url,(function(e){t.meta=e}))},methods:{getData:function(){var t=this;axios.get("/civil-contracts").then((function(e){t.fnResult=e.data,t.totalRows=t.fnResult.length,setTimeout((function(){t.loading=!1}),500)}))},onFiltered:function(t){this.totalRows=t.length,this.fnResult.length=t.length,this.currentPage=1,this.fnResult.count=t.length},resPerPage:function(t){return t+(this.currentPage-1)*this.perPage}}},Je=Object(c.a)(We,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",[a("bc",{attrs:{path:t.meta}}),t._v(" "),a("h1",{staticClass:"p-container-title"},[t._v(t._s(t.meta.C_NavL_value))]),t._v(" "),a("div",[t.loading?a("spin"):t._e(),t._v(" "),a("b-form-group",{staticClass:"mt-3",attrs:{label:"Търсене","label-for":"filter-input","label-cols-sm":"3","label-align-sm":"right","label-size":"sm"}},[a("b-form-input",{attrs:{id:"filter-input",type:"search",title:"Търсене","aria-label":"Търсене",placeholder:""},model:{value:t.filterFn,callback:function(e){t.filterFn=e},expression:"filterFn"}})],1),t._v("\n        Намерени "+t._s(t.fnResult.length)+" резултата\n        "),a("b-table",{attrs:{striped:"",hover:"",borderless:"",outlined:"","head-variant":"unset",responsive:"sm","sort-icon-left":"",filter:t.filterFn,items:t.fnResult,fields:t.fnFields,"sort-by":t.sortBy,"sort-desc":t.sortDesc,"current-page":t.currentPage,"per-page":t.perPage},on:{"update:sortBy":function(e){t.sortBy=e},"update:sort-by":function(e){t.sortBy=e},"update:sortDesc":function(e){t.sortDesc=e},"update:sort-desc":function(e){t.sortDesc=e},filtered:t.onFiltered},scopedSlots:t._u([{key:"cell(index)",fn:function(e){return[t._v("\n                "+t._s(t.resPerPage(e.index+1))+"\n            ")]}},{key:"cell(DATA_OT)",fn:function(e){return[t._v("\n                "+t._s(t._f("moment")(e.item.DATA_OT,"DD/MM/YYYY"))+"\n            ")]}},{key:"cell(DATA_DO)",fn:function(e){return[t._v("\n                "+t._s(t._f("moment")(e.item.DATA_DO,"DD/MM/YYYY"))+"\n            ")]}},{key:"cell(VAZLOJITEL_TEXT)",fn:function(e){return[e.item.A_ns_MP_id?t._t("default",[a("router-link",{attrs:{to:{path:"/bg/MP/"+e.item.A_ns_MP_id},title:e.item.A_ns_MPL_Name1+" "+e.item.A_ns_MPL_Name2+" "+e.item.A_ns_MPL_Name3}},[t._v("\n                        "+t._s(e.item.A_ns_MPL_Name1)+"\n                        "),2==e.item.A_ns_MP_FM?t._t("default",[a("strong",[t._v(t._s(e.item.A_ns_MPL_Name2))]),t._v("\n                            "+t._s(e.item.A_ns_MPL_Name3)+"\n                        ")]):t._t("default",[t._v("\n                            "+t._s(e.item.A_ns_MPL_Name2)+"\n                            "),a("strong",[t._v(" "+t._s(e.item.A_ns_MPL_Name3))])])],2)]):e.item.A_ns_C_id?t._t("default",[a("router-link",{attrs:{to:{path:"/bg/parliamentarycommittees/"+e.item.A_ns_C_id},title:e.item.A_ns_CL_value_short}},[t._v("\n                        "+t._s(e.item.A_ns_CL_value_short)+"\n                    ")])]):t._t("default",[t._v("\n                    "+t._s(e.item.VAZLOJITEL_TEXT)+"\n                ")]),t._v(" "),a("router-link",{attrs:{to:{path:"/bg/theme-site/ID/"+e.item.Cts_id},title:e.item.CtsL_value}},[t._v("\n                    "+t._s(e.item.CtsL_value)+"\n                ")])]}}],null,!0)}),t._v(" "),a("b-pagination",{staticClass:"my-0",attrs:{"total-rows":t.totalRows,"per-page":t.perPage,limit:30,size:"sm"},model:{value:t.currentPage,callback:function(e){t.currentPage=e},expression:"currentPage"}})],1)],1)}),[],!1,null,"4a425f0a",null).exports;var Qe=a("oVFp");window.Vue=a("XuX8"),Vue.use(Qe);var ta={components:{videoPlayer:l.videoPlayer},props:["stream","type","source","count","playlist"],data:function(){return{playerOptions:{},pinValue:88168,pin:"",url:"",action:0}},watch:{},mounted:function(){jwplayer("jwplayer").setup({file:"https://e118-ts.cdn.bg/dbg/test/chunklist_w12.m3u8",aspectratio:"16:9",autostart:!0,mute:!0,stretching:"uniform",liveSyncDuration:3,playbackRateControls:!0})},methods:function(t,e,a){return e in t?Object.defineProperty(t,e,{value:a,enumerable:!0,configurable:!0,writable:!0}):t[e]=a,t}({onPlayerPlay:function(t){},onPlayerPause:function(t){},onPlay1:function(){this.playerOptions.poster=""},trimTime:function(t){return t.slice(0,-3)},onPlayVideo1:function(t,e){this.playerOptions={width:"600px",height:"350px",muted:!0,language:"en",playbackRates:[.7,1,1.5,2],sources:[{type:e,src:t}],poster:"/media/bg/player_cover.jpg"},this.player},playerStateChanged:function(t){},playerReadied:function(t){}},"trimTime",(function(t){return t.slice(0,-3)}))},ea=Object(c.a)(ta,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",[a("div",{staticClass:"mt-5 mb-5 ml-5"},[t._t("default",[a("b-row",[a("b-col",{staticClass:"col-12 col-sm-12 col-lg-3 col-md-3"},[a("b-form-input",{attrs:{placeholder:"pin",title:"","aria-label":""},model:{value:t.pin,callback:function(e){t.pin=e},expression:"pin"}})],1),t._v(" "),a("b-col",{staticClass:"col-12 col-sm-12 col-lg-3 col-md-3"},[a("b-form-input",{attrs:{placeholder:"url",title:"","aria-label":""},model:{value:t.url,callback:function(e){t.url=e},expression:"url"}})],1),t._v(" "),a("b-col",{staticClass:"col-12 col-sm-12 col-lg-3 col-md-3"},[a("b-btn",{staticClass:"p-btn",on:{click:function(e){t.onPlayVideo1(t.url,"application/x-mpegurl"),t.action=1}}},[t._v("\n                        Play\n                    ")])],1)],1)]),t._v(" "),t.pin==t.pinValue&&1==t.action?t._t("default",[a("br")]):t._e(),t._v(" "),t.pin!=t.pinValue&&1==t.action?t._t("default",[a("br"),t._v(" "),a("span",{staticClass:"text-danger"},[t._v("Wrong pin")])]):t._e(),t._v(" "),t.pin==t.pinValue&&1===t.action?t._t("default",[a("video-player",{ref:"videoPlayer",staticClass:"ml-4 video-player-box",attrs:{options:t.playerOptions,playsinline:!0,customEventName:"customstatechangedeventname"},on:{play:function(e){return t.onPlayerPlay(e)},pause:function(e){return t.onPlayerPause(e)},statechanged:function(e){return t.playerStateChanged(e)},ready:t.playerReadied}})]):t._e(),t._v(" "),a("b-col",{staticClass:"col-6"},[a("div",{staticClass:"col-2",attrs:{id:"jwplayer"}})])],2)])}),[],!1,null,"0c1d9a2a",null).exports,aa={components:{Spin:o.a,Bc:v.a},props:["url"],data:function(){return{loading:!0,meta:"",app_title:"2",imgPath:"",navigation:[]}},metaInfo:function(){return{title:"".concat(this.meta.C_NavL_value," - ").concat(this.$t("title")),meta:[{name:"description",content:"".concat(this.$t("description"))},{name:"keywords",content:"".concat(this.$t("keywords"))},{property:"og:title",content:"".concat(this.meta.C_NavL_value," - ").concat(this.$t("title"))},{property:"og:site_name",content:"".concat(this.$t("title"))},{property:"og:url",content:"https://www.parliament.bg/".concat(this.meta.C_Lang_string,"/").concat(this.url)}]}},watch:{},mounted:function(){var t=this;this.$services.meta.getMeta(this.$route.params.lang,this.url,(function(e){t.meta=e})),this.loadNav(this.$i18n.locale,1)},methods:{setLocale:function(t){this.$i18n.locale=t,this.$router.push({params:{lang:t}})},loadNav:function(t,e){var a=this;axios.get("/nav-list/".concat(t,"/").concat(e)).then((function(t){a.navigation=t.data,setTimeout((function(){a.loading=!1}),500)}))}}},sa=Object(c.a)(aa,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",[a("bc",{attrs:{path:t.meta}}),t._v(" "),a("h1",{staticClass:"p-container-title"},[t._v(t._s(t.meta.C_NavL_value))]),t._v(" "),a("div",[a("ul",{staticClass:"pl-5 ml-5"},t._l(t.navigation,(function(e){return a("li",{key:e.C_Nav_id,staticClass:"nav-item "},[e.C_NavL_URL?a("a",{staticClass:"nav-link",attrs:{href:e.C_NavL_URL,title:e.C_NavL_value,"aria-label":e.C_NavL_value,"aria-haspopup":"true","aria-expanded":"false"}},[t._v(t._s(e.C_NavL_value))]):a("a",{attrs:{href:"",title:e.C_NavL_value,"aria-label":e.C_NavL_value,"data-toggle":"dropdown","aria-haspopup":"true","aria-expanded":"false"}},[t._v(t._s(e.C_NavL_value))]),t._v(" "),t._l(e.sub_nav,(function(e){return a("ul",{key:e.C_Nav_id},[a("li",{staticClass:"sub-level"},[e.C_NavL_URL?a("a",{attrs:{title:e.C_NavL_value,"aria-label":e.C_NavL_value,href:e.C_NavL_URL}},[t._v("\n                            "+t._s(e.C_NavL_value)+"\n                        ")]):a("router-link",{attrs:{title:e.C_NavL_value,"aria-label":e.C_NavL_value,to:{path:"/"+e.C_Lang_string+"/"+e.C_Nav_string,params:{lang:e.C_Lang_string}}}},[t._v("\n                            "+t._s(e.C_NavL_value)+"\n                        ")])],1),t._v(" "),a("ul",{},t._l(e.sub_nav,(function(e){return a("li",{key:e.C_Nav_id,staticClass:"nav-item"},[e.C_NavL_URL?a("a",{attrs:{title:e.C_NavL_value,"aria-label":e.C_NavL_value,href:e.C_NavL_URL}},[t._v("\n                                "+t._s(e.C_NavL_value)+"\n                            ")]):a("router-link",{attrs:{title:e.C_NavL_value,"aria-label":e.C_NavL_value,to:{path:"/"+e.C_Lang_string+"/"+e.C_Nav_string,params:{lang:e.C_Lang_string}}}},[t._v("\n                                "+t._s(e.C_NavL_value)+"\n                            ")])],1)})),0)])}))],2)})),0)])],1)}),[],!1,null,"7e505890",null).exports,na={components:{Spin:o.a,Bc:v.a},props:["colT","url"],data:function(){return{loading:!0,meta:"",app_title:"2",imgPath:"",news:[],archiveNews:[],gse:{}}},metaInfo:function(){return{title:"".concat(this.meta.C_NavL_value," - ").concat(this.$t("title")),meta:[{name:"description",content:"".concat(this.$t("description"))},{name:"keywords",content:"".concat(this.$t("keywords"))},{property:"og:title",content:"".concat(this.meta.C_NavL_value," - ").concat(this.$t("title"))},{property:"og:site_name",content:"".concat(this.$t("title"))},{property:"og:url",content:"https://www.parliament.bg/".concat(this.meta.C_Lang_string,"/").concat(this.url)}]}},watch:{"$route.query.string":function(){this.getData(this.$route.params.lang,this.$route.query.string,1)}},mounted:function(){var t=this;this.$services.meta.getMeta(this.$route.params.lang,this.url,(function(e){t.meta=e})),this.getData(this.$route.params.lang,this.$route.query.string,1)},methods:{getData:function(t,e,a){var s=this;axios.get("https://www.googleapis.com/customsearch/v1?key=AIzaSyA6E0kml4Faxu87dvTXM1GD20oE66D8c74&cx=010475036129657804310:vls7otvi6rs&q=".concat(e,"&nextPage=true&lr=lang_").concat(t,"&start=").concat(a)).then((function(t){s.gse=t.data,setTimeout((function(){s.loading=!1}),500)}))},scrollTo:function(){window.scrollTo(0,0)}}},ia=Object(c.a)(na,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",[a("bc",{attrs:{path:t.meta}}),t._v(" "),a("h1",{staticClass:"p-container-title"},[t._v(t._s(t.meta.C_NavL_value))]),t._v(" "),a("div",{ref:"search-list"}),t._v(" "),t.$route.query.string?t._e():a("div",[a("em",[t._v(t._s(t.$t("misc.search-criteria")))])]),t._v(" "),t.loading?a("spin"):a("div",[a("div",{staticClass:"mb-3"},[t._v("\n            "+t._s(t.$t("MP_ab8"))+" "+t._s(t.gse.queries.request[0].totalResults)+"\n            "+t._s(t.$t("MP_ab9"))+"\n            "+t._s(t.$t("misc.with"))+"\n\n            "),a("em",[t._v('"'+t._s(t.gse.queries.request[0].searchTerms)+'"')])]),t._v(" "),t._l(t.gse.items,(function(e,s){return a("div",{key:s.t_id,staticClass:"mark mb-4"},[a("div",{domProps:{innerHTML:t._s(e.htmlSnippet)}}),t._v(" "),a("div",[a("strong",[t._v(t._s(e.title))]),t._v(" "),a("a",{attrs:{href:e.link,title:e.link}},[t._v(t._s(e.link))])])])})),t._v(" "),a("b-row",[a("b-col",[a("div",{staticClass:"float-left"},[t.gse.queries.previousPage?a("b-button",{staticClass:"p-btn",attrs:{variant:"info",title:t.$t("Gal_back"),"aria-label":t.$t("Gal_back")},on:{click:function(e){return t.getData(t.$route.params.lang,t.$route.query.string,t.gse.queries.previousPage[0].startIndex)}}},[a("i",{staticClass:"cis-chevron-left"}),t._v(" "),t._v("\n                        "+t._s(t.$t("Gal_back"))+"\n                    ")]):t._e()],1),t._v(" "),a("div",{staticClass:"float-right"},[t.gse.queries.nextPage?a("b-button",{staticClass:"p-btn",attrs:{variant:"info",title:t.$t("Gal_forward"),"aria-label":t.$t("Gal_forward")},on:{click:function(e){t.getData(t.$route.params.lang,t.$route.query.string,t.gse.queries.nextPage[0].startIndex),t.scrollTo()}}},[t._v("\n                        "+t._s(t.$t("Gal_forward"))+"\n                        "),a("i",{staticClass:"cis-chevron-right"})]):t._e()],1)])],1)],2)],1)}),[],!1,null,"86041b88",null).exports;i.a.use(s.a);var la=[{path:"/",redirect:"/bg"},{path:"/tv",redirect:"/bg/video"},{path:"/nciom",redirect:"/bg/ncpi"},{path:"/ns140",redirect:"/bg"},{path:"/referendum",redirect:"/bg/register-referendum"},{path:"/register",redirect:"/bg/register-conflict"},{path:"/:lang",component:{render:function(t){return t("router-view")}},children:[{path:"/",component:p},{path:"news",name:"News",component:U,props:{url:"news"}},{path:"contacts",name:"Contacts",component:de,props:{url:"contacts"}},{path:"news/ID/:id",name:"News profile",component:U,props:{url:"news"}},{path:"gallery/ID/:id",name:"Gallery profile",component:Ue,props:{url:"gallery"}},{path:"gallery",name:"Gallery",component:Ue,props:{url:"gallery"}},{path:"calendar",name:"calendar",component:Ke,props:{url:"calendar"}},{path:"video",name:"Video",component:Q,props:{url:"video"}},{path:"stream-tst",name:"streamTest",component:ea,props:{url:"stream-tst"}},{path:"video/ID/:id",name:"Video profile",component:Q,props:{url:"video"}},{path:"video/cID/:cid",name:"Video profile C",component:Q,props:{url:"video"}},{path:"plenaryprogram",name:"Plenary",component:K,props:{url:"plenaryprogram"}},{path:"plenaryprogram/ID/:id",name:"Plenary profile",component:K,props:{url:"plenaryprogram"}},{path:"gdpr",name:"GDPR",component:j,props:{url:"gdpr"}},{path:"gdpr/:id",name:"GDPR profile",component:j,props:{url:"gdpr"}},{path:"plenaryst",name:"Steno",component:Wt,props:{url:"plenaryst"}},{path:"doc",name:"PlDoc",component:Oe,props:{url:"doc"}},{path:"doc/ID/:id",name:"PlDoc profile",component:Oe,props:{url:"doc"}},{path:"plenaryst/ns/:ns/ID/:id",name:"Steno profile",component:Wt,props:{url:"plenaryst"}},{path:"parliamentarycontrol",name:"PC control",component:vt,props:{url:"parliamentarycontrol"}},{path:"parliamentarycontrol/ID/:id",name:"PC control profile",component:vt,props:{url:"parliamentarycontrol"}},{path:"voteofnoconfidence",name:"VoteNoc",component:ht,props:{url:"voteofnoconfidence"}},{path:"voteofnoconfidence/ID/:id",name:"VoteNoc profile",component:gt,props:{url:"voteofnoconfidence"}},{path:"voteofconfidence",name:"Vote",component:yt,props:{url:"voteofconfidence"}},{path:"voteofconfidence/ID/:id",name:"Vote profile",component:Pt,props:{url:"voteofconfidence"}},{path:"debate",name:"Debate",component:$t,props:{url:"debate"}},{path:"debate/ID/:id",name:"Debate profile",component:kt,props:{url:"debate"}},{path:"hearing",name:"Hearing",component:Tt,props:{url:"hearing"}},{path:"hearing/ID/:id",name:"Hearing profile",component:It,props:{url:"hearing"}},{path:"pl_blic",name:"pcBlic",component:Rt,props:{url:"pl_blic"}},{path:"pl_blic/ID/:id",name:"Blic profile",component:Vt,props:{url:"pl_blic"}},{path:"enquiry",name:"Enquiry",component:wt,props:{url:"enquiry",type:62}},{path:"enquiry/ID/:id",name:"Enquiry profile",component:Nt,props:{url:"enquiry",type:62}},{path:"topical_nature",name:"Topical",component:wt,props:{url:"topical_nature",type:61}},{path:"topical_nature/ID/:id",name:"Topical profile",component:Nt,props:{url:"topical_nature",type:61}},{path:"eudocs",name:"euDocs",component:se,props:{url:"eudocs"}},{path:"ceudocs",name:"ceuDocs",component:se,props:{url:"ceudocs"}},{path:"euparldocs",name:"euParlDocs",component:se,props:{url:"euparldocs"}},{path:"eudocs/ID/:id",name:"euDocs Profile",component:ie,props:{url:"eudocs"}},{path:"ceudocs/ID/:id",name:"ceuDocs Profile",component:ie,props:{url:"ceudocs"}},{path:"euparldocs/ID/:id",name:"parlDocs Profile",component:ie,props:{url:"euparldocs"}},{path:"eu-dossier",name:"euDossier",component:Qt,props:{url:"eu-dossier"}},{path:"eu-dossier/ID/:id",name:"euDossier profile",component:ee,props:{url:"eu-dossier"}},{path:"publicprocurement",name:"PPr",component:Se,props:{url:"publicprocurement"}},{path:"publicprocurement/ID/:id",name:"PPr profile",component:Se,props:{url:"publicprocurement"}},{path:"ns_acts",name:"Acts",component:Bt,props:{url:"ns_acts"}},{path:"ns_acts/ID/:id",name:"Acts profile",component:qt,props:{url:"ns_acts"}},{path:"laws",name:"Law",component:Ht,props:{url:"laws",actType:1,actName:"Закон"}},{path:"laws/ID/:id",name:"Law profile",component:zt,props:{url:"laws",actType:1,actName:"Закон"}},{path:"desision",name:"Desision",component:Ht,props:{url:"desision",actType:2,actName:"Решение"}},{path:"desision/ID/:id",name:"Desision profile",component:zt,props:{url:"desision",actType:2,actName:"Решение"}},{path:"declaration",name:"Declarion",component:Ht,props:{url:"declaration",actType:3,actName:"Декларация"}},{path:"declaration/ID/:id",name:"Declarion profile",component:zt,props:{url:"declaration",actType:3,actName:"Декларация"}},{path:"bills",name:"Bills",component:et,props:{url:"bills"}},{path:"bills/ID/:id",name:"Bills profile",component:st,props:{url:"bills"}},{path:"petitions",name:"Petitions",component:it,props:{url:"petitions",ptType:1}},{path:"dv-tax",name:"DvPetitions",component:it,props:{url:"dv-tax",ptType:2}},{path:"public-consultations",name:"billsConsult",component:ot,props:{url:"public-consultations"}},{path:"public-consultations/ID/:id",name:"PB Consult profile",component:ct,props:{url:"public-consultations"}},{path:"parliamentarygroups",name:"PG list",component:h,props:{colT:"2",url:"parliamentarygroups"}},{path:"parliamentarygroups/:id",name:"PG profile",component:M,props:{colT:"2",url:"parliamentarygroups"}},{path:"parliamentarycommittees",name:"PC list",component:h,props:{colT:"3",url:"parliamentarycommittees"}},{path:"parliamentarycommittees/:id",alias:"parliamentarycommittees/members/:id",name:"PC profile",component:M,props:{colT:"3",url:"parliamentarycommittees"}},{path:"parliamentarycommittees/:id/sitting/:sitId",alias:"parliamentarycommittees/members/:id/sitting/:sitId",name:"PC profile Sitting",component:M,props:{colT:"3",url:"parliamentarycommittees"}},{path:"parliamentarycommittees/:id/news/:newsId",alias:"parliamentarycommittees/members/:id/news/:newsId",name:"PC profile News",component:M,props:{colT:"3",url:"parliamentarycommittees"}},{path:"parliamentarycommittees/:id/reports/:stanId",alias:"parliamentarycommittees/members/:id/reports/ID/:stanId",name:"PC profile Reports",component:M,props:{colT:"3",url:"parliamentarycommittees"}},{path:"parliamentarycommittees/:id/standpoint/:stpId",alias:"parliamentarycommittees/members/:id/standpoint/ID/:stpId",name:"PC profile Stanpoints",component:M,props:{colT:"3",url:"parliamentarycommittees"}},{path:"parliamentarycommittees/:id/steno/:stenoId",alias:"parliamentarycommittees/members/:id/steno/:stenoId",name:"PC profile Steno",component:M,props:{colT:"3",url:"parliamentarycommittees"}},{path:"committees-sitting",name:"Incoming com sitting ",component:mt,props:{url:"committees-sitting"}},{path:"presidingbody",name:"PB list",component:D,props:{co
    ✓ • 0ms

## Assistant

The Parliament site loads its content through an underlying data API. I found the endpoint used for individual plenary transcripts; I’m checking its fields and the committee equivalent. This may let us collect structured records directly, though we’ll still need to handle changes to that interface.

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/api/v1/pl-sten/11050 | python3 -c 'import sys,json; d=json.load(sys.stdin); print(json.dumps(d,ensure_ascii=False)[:16000])'
    {"Pl_Sten_id": 11050, "Pl_Sten_date": "2025-09-19", "Pl_Sten_sub": "ПЕТДЕСЕТ И ПЪРВО НАРОДНО СЪБРАНИЕ, ДЕВЕТДЕСЕТ И ОСМО ЗАСЕДАНИЕ, София, петък, 19 септември 2025 г. Открито в 9,19 ч.", "Pl_Sten_body": "Председателствали: председателят Наталия Киселова и заместник-председателите Драгомир Стойнев и Юлиана Матеева<br />\r\n<br />\r\nСекретари: Габриел Вълков и Иван Иванов<br />\r\n<br />\r\n9,02 ч.<br />\r\n<br />\r\nПРЕДСЕДАТЕЛ НАТАЛИЯ КИСЕЛОВА: Добър ден, уважаеми колеги! Моля за регистрация. <br />\r\nРегистрирани са 112 народни представители. <br />\r\nСлед 15 минути ще направим нова регистрация.<br />\r\n<br />\r\nОткрито в 9,19 ч.<br />\r\n<br />\r\nПРЕДСЕДАТЕЛ НАТАЛИЯ КИСЕЛОВА: Уважаеми колеги, моля за регистрация.<br />\r\nИмаме кворум.<br />\r\nОткривам днешното пленарно заседанието. (Звъни.)<br />\r\nТОШКО ЙОРДАНОВ (ИТН, от място): Процедура!<br />\r\nПРЕДСЕДАТЕЛ НАТАЛИЯ КИСЕЛОВА: Заповядайте, господин Йорданов.<br />\r\nТОШКО ЙОРДАНОВ (ИТН): Благодаря, госпожо Председател.<br />\r\nПредлагам да включим камерите на БНТ и преди това – проверка на кворума.<br />\r\nПРЕДСЕДАТЕЛ НАТАЛИЯ КИСЕЛОВА: Бяха направени две процедурни предложения от господин Йорданов – за включване на камерите на Българската национална телевизия и проверка на кворума.<br />\r\nПърво поставям на гласуване предложението за проверка на кворума. <br />\r\nБлагодаря Ви.<br />\r\nПоставям на гласуване предложението на господин Йорданов за включване на камерите на Българската национална телевизия.<br />\r\nГласували 169 народни представители: за 50, против 6, въздържали се 113.<br />\r\nПредложението не е прието.<br />\r\nРЕПЛИКА ОТ ВЪЗРАЖДАНЕ: Прегласуване!<br />\r\nПРЕДСЕДАТЕЛ НАТАЛИЯ КИСЕЛОВА: Господин Йончев иска процедура.<br />\r\nНякой иска ли прегласуване? (Реплика.)<br />\r\nКой от ВЪЗРАЖДАНЕ, тъй като са много малко? Не видях. Кой? Господин Йордан Тодоров поиска.<br />\r\nПрегласуване за това дали да бъдат включени камерите на Българската национална телевизия.<br />\r\nГласували 173 народни представители: за 55, против 9, въздържали се 109.<br />\r\nПредложението не е прието.<br />\r\nЗаповядайте, господин Йончев, за процедура.<br />\r\nИЛИЯН ЙОНЧЕВ (БСП – Обединена левица): Предлагам да бъдат включени микрофоните на БНР и преди това – проверка на кворума.<br />\r\nПРЕДСЕДАТЕЛ НАТАЛИЯ КИСЕЛОВА: Уважаеми колеги, бяха направени две процедурни предложения: проверка на кворума, след това ще подложа на гласуване предложението на господин Йончев.<br />\r\nРегистрирани са 143 народни представители.<br />\r\nПодлагам на гласуване предложението на господин Йончев за включване на микрофоните на Българското национално радио.<br />\r\nГласували 175 народни представители: за 54, против 10, въздържали се 111.<br />\r\nПредложението не е прието.<br />\r\nГосподин Василев поиска процедура. <br />\r\nПЕТЪР ПЕТРОВ (ВЪЗРАЖДАНЕ, от място): Прегласуване!<br />\r\nПРЕДСЕДАТЕЛ НАТАЛИЯ КИСЕЛОВА: Извинявайте, господин Василев – може и оттук да гласувате, тъй като господин Петър Петров поиска прегласуване за включване на микрофоните.<br />\r\nМоля, режим на прегласуване. <br />\r\nГласували 173 народни представители: за 55, против 11, въздържали се 107.<br />\r\nПредложението не е прието.<br />\r\nЗаповядайте, господин Василев.<br />\r\nАСЕН ВАСИЛЕВ (ПП-ДБ): Благодаря, госпожо Киселова.<br />\r\nПреди две седмици по време на блицконтрола премиера го нямаше, аз имах въпрос към него. Съгласно Правилника премиерът трябва да се яви на изслушване и да отговори на въпроса следващата седмица. Писмено изпратих към Вас процедура това да се случи – не виждаме премиера.<br />\r\nИскам половин час почивка, за да може да бъде включено изслушването в дневния ред днес и да дойде премиерът да отговори на въпроса. (Оживление.)<br />\r\nПоловин час почивка от името на група, за да дойде премиерът.<br />\r\nПРЕДСЕДАТЕЛ НАТАЛИЯ КИСЕЛОВА: Почивката ще бъде 15 минути. <br />\r\nДържа да кажа, че съм уведомила министър-председателя, днес не е в София. (Реплики.)<br />\r\nКой принизява парламента, е спорен въпрос.<br />\r\n15 минути почивка по искане на парламентарна група.<br />\r\n<br />\r\n(След почивката.)<br />\r\nПРЕДСЕДАТЕЛ НАТАЛИЯ КИСЕЛОВА: Уважаеми колеги, продължаваме нашата работа. <br />\r\nНа основание чл. 47, ал. 5 от Правилника за организацията и дейността на Народното събрание във връзка с постъпило предложение за включване на точка в дневния ред на пленарното заседание от председателя… (Шум и реплики от ВЕЛИЧИЕ.)<br />\r\nЗапочнах процедурата. Гледам Ви, слушам Ви. Какво желаете? Процедура ли? След като приключим с включването в дневния ред, ще Ви дам думата.<br />\r\nОт председателя на Комисията по енергетика е постъпило предложение и след проведен Председателски съвет предлагам в програмата за работата на Народното събрание за днешното пленарно заседание да се включи следната допълнителна точка: Второ гласуване на Законопроекта за изменение и допълнение на Закона за ограничаване изменението на климата. Законопроектът е приет на първо гласуване на 12 септември 2025 г. като точка първа за днес, 19 септември. <br />\r\nМоля, колеги, режим на гласуване.<br />\r\nГласували 154 народни представители: за 125, против 17, въздържали се 12. <br />\r\nПредложението е прието, а точка 1 – фиксираната за днес, ще бъде точка 2. <br />\r\nЗаповядайте, господин Гаджев, а след това е госпожа Лариса Савова.<br />\r\nХРИСТО ГАДЖЕВ (ГЕРБ-СДС): Уважаема госпожо Председател, уважаеми дами и господа народни представители! Правя процедура за удължаване на законодателната част на днешното заседание до изчерпването на първите две точки, а именно фиксираната точка първа и новата точка, която сега вкарахме, тоест Закона за статистиката и Закона за изменение на климата, след което 3 часа парламентарен контрол и във връзка с това – проверка на кворума.<br />\r\nПРЕДСЕДАТЕЛ НАТАЛИЯ КИСЕЛОВА: Уважаеми колеги, господин Гаджев направи две предложения: първото е за удължаване на законодателната ни дейност до приемането на двете точки – тази, която току-що включихме, и тази, която е от седмичната ни програма, и проверка на кворума. <br />\r\nГосподин Гаджев, парламентарният контрол да бъде 3 часа, нали така? (Шум и реплики: „Той го каза!“)<br />\r\nПърво, правим проверка на кворума. Благодаря Ви.<br />\r\nМоля, по групи.<br />\r\nВторото предложение беше за удължаване на законодателната част от днешното пленарно заседание с приемането на двете точки и след това – парламентарен контрол 3 часа. <br />\r\nМоля, режим на гласуване.<br />\r\nГласували 148 народни представители за 123, против 4, въздържали се 21.<br />\r\nПредложението е прието. <br />\r\nЗаседанието ще продължи до изчерпване на програмата.<br />\r\nЗаповядайте, госпожо Савова, за процедура.<br />\r\nЛАРИСА САВОВА (ВЕЛИЧИЕ): Госпожо Председател, 30 минути почивка от името на ВЕЛИЧИЕ. (Шум и реплики от ГЕРБ-СДС: „Четете ли Правилника? Лъжеш! Не може да искаш почивка, не си в ръководството на групата!“)<br />\r\nПРЕДСЕДАТЕЛ НАТАЛИЯ КИСЕЛОВА: Уважаема колега, в ръководството ли сте на парламентарната група, защото... Ще излезе ли председател или заместник-председател на парламентарната група? <br />\r\nЗаповядайте, госпожо Катинчарова.<br />\r\nКРАСИМИРА НИНОВА-КАТИНЧАРОВА (ВЕЛИЧИЕ): Потвърждавам това, което парламентарният секретар на парламентарната група Лариса Савова каза, и използвам случая да направя процедура по начина на водене.<br />\r\nГоспожо Киселова, Вие просто нямате спирка! (Шум и реплики.) Да ни виждате отгоре, а за пореден път да извивате Правилника така, както на Вас Ви е удобно, мерите кворум за 18 –20 секунди! И не знам байрак ли, какво трябва да веем тук, но давате абсолютно поводи да Ви искаме оставката. Това е... <br />\r\nРазбирате ли, не може просто да не спазвате Правилника! Вие орязвате права – за пореден път не ми давате на мен и на моите колеги, поради факта, че не ни виждате! Според мен това не е спонтанно. Не можем да се изкажем, не можем да вземем процедура, започвате следващата и ние губим тази възможност. Приемете това като поредно обвинение за това, че не спазвате правилата. Вие самата, която сте първа сред равни, трябва да давате пример за това. Благодаря.<br />\r\nПРЕДСЕДАТЕЛ НАТАЛИЯ КИСЕЛОВА: Всички български граждани и народните представители видяха по колко секунди е проверката. С Вашата група ще разговаряме, за да можем да избягваме занапред този проблем. <br />\r\nБеше поискана почивка – 15 минути почивка, сега е 9,55 ч. – до 10,10 ч. (Шум и реплики.) <br />\r\nСлед почивката има заявени три декларации, държа да кажа.<br />\r\n<br />\r\n(След почивката.)<br />\r\nПРЕДСЕДАТЕЛ ДРАГОМИР СТОЙНЕВ: Уважаеми народни представители! Часът е 10,10 и продължаваме нашата работа.<br />\r\nПреди да дам думата за декларация, да прочета едно съобщение. <br />\r\nМинистърът на отбраната в изпълнение на Закона за преминаването през и пребиваването на територията на Република България на съюзнически и на чужди въоръжени сили е уведомил Народното събрание за издадени от него заповеди, с които е разрешил преминаването през и пребиваването на територията на Република България на съюзнически и чужди въоръжени сили и средства. <br />\r\nУведомленията са с входящи номера № 51-503-09-116 и № 51-503-09-117 и са предоставени на Комисията по отбрана.<br />\r\nЗа декларация от парламентарна група – заповядайте, господин Николов, имате думата.<br />\r\nДИМИТЪР НИКОЛОВ (ГЕРБ-СДС): Уважаеми господин Председател, колеги народни представители! На 22 септември ще отбележим поредното знаково събитие в своята история – през 1908 г. България става независима държава, за да заеме полагащото ѝ се достойно място сред останалите европейски нации. Дата, която несъмнено заслужава по-дълбоко осмисляне и уважение от българското общество. <br />\r\nНа 22 септември 1908 г. във Велико Търново тържествено е обявено за независимо Царство България, а княз Фердинанд приема титлата „цар на българите“. Изобщо не е случаен фактът, че в старопрестолната историческа и духовна столица е кулминацията на празничните прояви. С манифест княз Фердинанд обявява независимостта на България. С този акт на практика се отхвърлят последните васални връзки с Османската империя. Изборът на място не е случайно. Тук, във Велико Търново, Фердинанд иска да придаде легитимност на Третото българско царство, като прави връзка с Първото и Второто българско царство.<br />\r\nВ църквата „Св. 40 мъченици“ е погребан цар Калоян. Предполага се, че в тази църква е погребан и цар Иван Асен. Княжество България става независима държава начело с коронования цар Фердинанд. Високата порта, а след това и Великите сили признават официално българската независимост. Чрез обявяването на независимостта, постигната най-вече чрез умелата дипломатичност и международния авторитет на княз Фердинанд, българският народ показва на света, че едва 30 години след освобождението си вече е духовно, културно и материално укрепнал. Дълго време значимостта на това събитие от новата история се подценява и неглижира, но на тази дата през 1908 г. България – една от най-старите държави в Европа, отново се позиционира на световната политическа и икономическа карта. <br />\r\nУсловията за обявяване на независимостта на младата българска държава в началото на 20 век са изключително благоприятни. През лятото на 1908 г. Младотурската революция в Османската империя завършва с успех за реформистите. Австро-Унгария – една от великите сили, наложили Берлинския договор, се е готвила да анексира две от провинциите на империята – Босна и Херцеговина, респективно да го наруши. Затова и българският княз Фердинанд се е обърнал директно към император Франц Йосиф да съгласуват своите действия. Австро-Унгария обявява анексията на Босна и Херцеговина на 20 септември. Два дни по-късно в църквата „Св. 40 мъченици“ княз Фердинанд прочита прословутия Манифест за обявяването на независимостта и се отслужва молебен за благоденствието на българската държава. След това министър-председателят Малинов прочита отново Манифеста на историческия хълм Царевец пред събралото се хилядно множество. Провъзгласяването на независимостта не е само голям успех за българската дипломация, но и след васалното княжество започва да се нарича Царство България, а българският владетел вече се титулува цар. <br />\r\nДеликатната ситуация, която настъпва след нарушаването на Берлинския договор, е успешно разрешена. Първо, била е спечелена Британската империя, която поставя условието да започнат преговори между българското правителство и Високата порта. Преговорите започват веднага и българската делегация е водена от Андрей Ляпчев. По време на тях Високата порта иска България да плаща голям данък. В качеството си на министър-председател Малинов заявява, че независимост не се откупува. Дори се стига до частична мобилизация на Българската армия. Русия не желае военен конфликт на Балканите и се заема да посредничи в преговорите. Тя се съгласява да опрости дълга на Османската империя, останал още от Руско-турската война, в замяна на което Високата порта се отказва да иска обезщетение от България и признава нейната независимост. Официално това става на 6 април 1909 г., след което в продължение на 10 дни европейските сили признават България за царство и за независима държава. (Реплики от народния представител Цончо Ганев.)<br />\r\nЗа сравнително кратък период от време от дълбока провинция страната ни придобива внушителен европейски облик, печели сериозно уважение със своя бърз икономически и културен растеж. С Решение на Народното събрание от 10 септември 1998 г. – предполагам, че тук в залата има няколко народни представители, които са гласували това решение, и Указ на президента Петър Стоянов 22 септември е обявен за национален празник.<br />\r\nНезависимостта на България е резултат от изящна дипломация и проява на силна политическа воля. Национално постижение, повод за самочувствие и гордост, както и висока отговорност и пред нас самите, които сме тук в тази зала. Можем да опазим и да съхраним това наследство, само работейки за духовното и материалното израстване на държавата, издигайки се над политическите страсти и тесните егоистични прояви. Да живее и пребъде независима България! (Ръкопляскания от ГЕРБ-СДС, възглас от народния представител Цончо Ганев: „Браво!“)<br />\r\nПРЕДСЕДАТЕЛ ДРАГОМИР СТОЙНЕВ: Благодаря Ви, господин Николов.<br />\r\nДекларация от името на парламентарната група на „БСП – Обединена левица“ – госпожо Бояджиева, заповядайте.<br />\r\nМАРИАНА БОЯДЖИЕВА (БСП – Обединена левица): Уважаеми господин Председател, уважаеми госпожи и господа народни представители! На 22 септември ще отбележим 117-годишнина от Обявяването на независимостта на България. <br />\r\nПрез 1908 г. на тази дата в старата българска столица Велико Търново се е състояло тържественото провъзгласяване на независимостта на Северна и Южна България в независимо царство, обявено в Манифеста към българския народ с думите: „Да живее свободна и независима България! Да живее българският народ!“. В славния царев град Търнов е написана нова и величествена страница в хилядолетната българска история. <br />\r\nОт църквата „Св. 40 мъченици“ българите смело, гордо и решително заявяват, че няма да позволят повече някой друг да решава съдбата им, да ги разделят и да определят бъдещето им. С този акт България не само става независима държава, но официално се завършва процесът на съединението от 1885 г. на Княжество България с Източна Румелия.<br />\r\nПрокламирането на независимостта има огромно историческо значение и превръща държавата ни в самостоятелен международноправен субект. Това е венецът в десетилетните усилия на българите да имат своя държава и самоуправление. Провъзгласяването на независимостта е политически акт, извършен от правителството на Александър Малинов. 22 септември 1908 г. се превръща в поредния пример след Съединението в българската история, когато в резултат на общите национални усилия се постигат идеалите на Българското възраждане. Използва се изключително благоприятната обстановка в Европа, за да бъде провъзгласена българската независимост. Така българск
    ✓ • 104ms

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/js/app.js | rg -o '.{0,140}(getSteno|Pl_StenV|Co_Sten|/com-|/pl-st|steno/).{0,250}' | head -n 40
    rams.stanId),this.$route.params.stpId>0&&this.getStan2(this.$route.params.lang,this.$route.params.stpId),this.$route.params.stenoId>0&&this.getSteno(this.$route.params.lang,this.$route.params.stenoId)},methods:{getProfile:function(t,e,a,s,n){var i=this;this.mpFilter.date?(s=s,this.rqDate=s):(s="",this.rqDate=new Date);var l="";this.$route.query.date&&(this.rqDate=this.$route.query.date,l=this.$r
    ams:o}).then((function(t){i.collListMP=t.data,setTimeout((function(){i.loading=!1}),500)}))},getPastSit:function(t,e){var a=this;axios.get("/com-past-sit/".concat(t,"/").concat(e)).then((function(t){a.colPastSit=t.data,setTimeout((function(){a.loading=!1}),500)}))},getMeeting:function(t,e){var a=this;axios.get("/com-meeting/".concat(t,"/").concat(e)).then((function(t){a.colMeeting=t.data,setT
    imeout((function(){a.loading=!1}),500)}))},getPastNews:function(t,e){var a=this;axios.get("/com-past-news/".concat(t,"/").concat(e)).then((function(t){a.colPastNews=t.data,setTimeout((function(){a.loading=!1}),500)}))},getNews:function(t,e){var a=this;axios.get("/com-news/".concat(t,"/").concat(e)).then((function(t){a.colNews=t.data,setTimeout(
    (function(){a.loading=!1}),500)}))},getDocs:function(t,e){var a=this;axios.get("/com-docs/".concat(t,"/").concat(e)).then((function(t){a.comDocs=t.data,setTimeout((function(){a.loading=!1}),500)}))},getActs:function(t,e,a){var s=this;axios.get("/com-acts/".concat(t,"/").concat(e,"/").concat(a)).then((function(t){s.comActs=t.data,setT
    imeout((function(){s.loading=!1}),500)}))},getBills:function(t,e,a){var s=this;axios.get("/com-acts/".concat(t,"/").concat(e,"/").concat(a)).then((function(t){s.comBills=t.data,setTimeout((function(){s.loading=!1}),500)}))},getStan1:function(t,e){var a=this;axios.get("/com-stan/".concat(t,"/").concat(e)).then((function(t){a.comStan1=t.data,set
    Timeout((function(){a.loading=!1}),500)}))},getStan2:function(t,e){var a=this;axios.get("/com-stan/".concat(t,"/").concat(e)).then((function(t){a.comStan2=t.data,setTimeout((function(){a.loading=!1}),500)}))},getSteno:function(t,e){var a=this;axios.get("/com-steno/".concat(t,"/").concat(e)).then((function(t){a.comSteno=t.data,setTimeout((func
     "),a("archive",{attrs:{model:"A_Cm_Steno",index:t.$route.params.id,index2:"0",title:"",url:"parliamentarycommittees/"+t.$route.params.id+"/steno/",query:""}})],1)],2):t._e(),t._v(" "),3==t.colT&&"bg"==t.$route.params.lang?a("b-tab",{attrs:{active:t.$route.params.stpId>0},on:{click:function(e){t.collTab=9}}},[a("template",{slot:"title"},[t._v("\n                        "+t._s(t.$t("Com9"))+"\n
         ")])],1)]):t._t("default",[a("em",[a("router-link",{attrs:{to:{path:"/"+t.$route.params.lang+"/parliamentarycommittees/"+e.A_ns_C_id+"/steno/"+e.A_Cm_Stid},title:e.A_ns_CL_value}},[t._v("\n                                            "+t._s(e.A_ns_CL_value)+"\n                                        ")])],1)])],2)})),0)])],1),t._v(" "),a("b-col",{staticClass:"col-9 col-sm-9 col-lg-3 col-md
    on(e){return a("li",{key:e.L_Act_A_id},[a("router-link",{attrs:{to:{path:"/"+t.$route.params.lang+"/parliamentarycommittees/"+e.A_ns_C_id+"/steno/"+e.A_Cm_Stid},title:e.A_ns_CL_value}},[t._v("\n                                    "+t._s(t._f("moment")(e.L_Act_St_date,"DD/MM/YYYY"))+"\n                                    - "+t._s(e.A_ns_CL_value)+" -\n                                    "+t._s(
    59f3f77d",null).exports),Kt={components:{stream:L},props:["stId"],data:function(){return{loading:!0,plSten:[]}},watch:{stId:function(){this.getSteno(this.stId)},"$route.params.lang":function(){this.getSteno(this.stId)}},mounted:function(){this.getSteno(this.stId)},methods:{getSteno:function(t){var e=this;t>0&&axios.get("/pl-sten/".concat(t)).then((function(t){e.plSten=t.data,setTimeout((function
    ,t._v(" "),a("div",[a("div",{staticClass:"mb-2"},[a("pl-st-int",{attrs:{stId:t.$route.params.id}})],1),t._v(" "),a("archive",{attrs:{model:"Pl_StenV",index:"0",index2:"0",title:"Стенограма от пленарно заседание",url:"plenaryst/ns/55/ID/"}})],1)],1)}),[],!1,null,"649ab6f4",null).exports,Jt={components:{Spin:o.a,Bc:v.a},props:["colT","url"],data:function(){return{loading:!0,meta:"",app_title:"2",f
    oint/ID/:stpId",name:"PC profile Stanpoints",component:M,props:{colT:"3",url:"parliamentarycommittees"}},{path:"parliamentarycommittees/:id/steno/:stenoId",alias:"parliamentarycommittees/members/:id/steno/:stenoId",name:"PC profile Steno",component:M,props:{colT:"3",url:"parliamentarycommittees"}},{path:"committees-sitting",name:"Incoming com sitting ",component:mt,props:{url:"committees-sitti
    ✓ • 0ms

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/api/v1/pl-sten/11050 | python3 -c 'import sys,json; d=json.load(sys.stdin); print(json.dumps({k: ({"length":len(v),"sample":v[:200]} if isinstance(v,str) and len(v)>800 else v) for k,v in d.items()},ensure_ascii=False,indent=2))'
    {
      "Pl_Sten_id": 11050,
      "Pl_Sten_date": "2025-09-19",
      "Pl_Sten_sub": "ПЕТДЕСЕТ И ПЪРВО НАРОДНО СЪБРАНИЕ, ДЕВЕТДЕСЕТ И ОСМО ЗАСЕДАНИЕ, София, петък, 19 септември 2025 г. Открито в 9,19 ч.",
      "Pl_Sten_body": {
        "length": 358886,
        "sample": "Председателствали: председателят Наталия Киселова и заместник-председателите Драгомир Стойнев и Юлиана Матеева<br />\r\n<br />\r\nСекретари: Габриел Вълков и Иван Иванов<br />\r\n<br />\r\n9,02 ч.<br />\r\n<br "
      },
      "files": [
        {
          "Pl_StenDid": 18057,
          "Pl_StenDname": "Гласуване по парламентарни групи",
          "Pl_StenDfile": "/pub/StenD/20250923114750_gv190925.pdf",
          "Pl_StenDtype": "pdf"
        },
        {
          "Pl_StenDid": 18058,
          "Pl_StenDname": "Гласуване по парламентарни групи",
          "Pl_StenDfile": "/pub/StenD/20250923114803_gv190925.xlsx",
          "Pl_StenDtype": "xls"
        },
        {
          "Pl_StenDid": 18059,
          "Pl_StenDname": "Гласуване по парламентарни групи",
          "Pl_StenDfile": "/pub/StenD/20250923114812_gv190925.csv",
          "Pl_StenDtype": "xls"
        },
        {
          "Pl_StenDid": 18060,
          "Pl_StenDname": "Поименно гласуване",
          "Pl_StenDfile": "/pub/StenD/20250923114825_iv190925.pdf",
          "Pl_StenDtype": "pdf"
        },
        {
          "Pl_StenDid": 18061,
          "Pl_StenDname": "Поименно гласуване",
          "Pl_StenDfile": "/pub/StenD/20250923114832_iv190925.xlsx",
          "Pl_StenDtype": "xls"
        },
        {
          "Pl_StenDid": 18062,
          "Pl_StenDname": "Поименно гласуване",
          "Pl_StenDfile": "/pub/StenD/20250923114839_iv190925.csv",
          "Pl_StenDtype": "xls"
        }
      ],
      "video": {
        "Vidate": "2025-09-19",
        "Vicount": 2,
        "default": "https://parliament.bg/Gallery/video/archive-2025_09_19_1.mp4",
        "playlist": [
          {
            "item": 1,
            "file": "https://parliament.bg/Gallery/video/archive-2025_09_19_1.mp4"
          },
          {
            "item": 2,
            "file": "https://parliament.bg/Gallery/video/archive-2025_09_19_2.mp4"
          }
        ]
      },
      "videoArchive": 1
    }
    ✓ • 549ms

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/js/app.js | rg -o '.{0,120}(/archive|archive-list|archive-date|/ns"|/ns/|/coll-list|/coll-profile|/mp-profile).{0,350}' | head -n 30
    [t._v(t._s(t.title))]):t._t("default",[t._v(t._s(e.t_label))])],2))],1)})),0)])],1),t._v(" "),a("b-row",{staticClass:"p-archive-list"},t._l(t.archiveYear,(function(e){return a("b-col",{key:e.t_year,staticClass:"col-12 col-sm-6 col-lg-2 col-md-2"},[a("div",{staticClass:"archive-head"},[t._v("\n                "+t._s(e.t_year)+"\n            ")]),t._v(" "),a("ul",t._l(e.t_month_list,(function(s){return a("li",{key:s.t_month},[a("span",{directives:[{name:"scroll-to",rawName:"v-scr
    )),axios.get("/mun-list/0").then((function(t){e.munList=t.data,setTimeout((function(){e.loading=!1}),500)})),axios.get("/coll-list/".concat(t,"/2")).then((function(t){e.pgList=t.data,setTimeout((function(){e.loading=!1}),500)})),axios.get("/coal-list/".concat(t)).then((function(t){e.coalList=t.data,setTimeout((function(){e.loading=!1}),500)}))},listMps:function(){var t;this.$router.push({name:"MP list",params:{},query:(t={pgId:this.mpFilter.pgId},m(t,"pgId",this.mpFilter.pgId
    ams.id,this.colT):this.getData(this.$route.params.lang,this.colT)},methods:{getData:function(t,e){var a=this;axios.get("/coll-list/".concat(t,"/").concat(e)).then((function(t){a.collList=t.data,setTimeout((function(){a.loading=!1}),500)})),3==e&&(axios.get("/coll-list/".concat(t,"/4")).then((function(t){a.collList4=t.data,setTimeout((function(){a.loading=!1}),500)})),axios.get("/coll-list/".concat(t,"/9")).then((function(t){a.collList9=t.data,setTimeout((function(){a.loading=
    !1}),500)})))},getProfile:function(t,e,a){var s=this;axios.get("/coll-list-mp/".concat(t,"/").concat(e,"/").concat(a)).then((function(t){s.collListMP=t.data,setTimeout((function(){s.loading=!1}),500)}))}}},h=(a("7KHh"),Object(c.a)(f,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",[a("bc",{attrs:{path:t.meta}}),t._v(" "),a("h1",{staticClass:"p-container-title"},[t._v(t._s(t.meta.C_NavL_value))]),t
    .$route.query.date&&(this.rqDate=this.$route.query.date,l=this.$route.query.date),1==n&&(l=s);var o={date:l};axios.get("/coll-list-mp/".concat(t,"/").concat(e,"/").concat(a),{params:o}).then((function(t){i.collListMP=t.data,setTimeout((function(){i.loading=!1}),500)}))},getPastSit:function(t,e){var a=this;axios.get("/com-past-sit/".concat(t,"/").concat(e)).then((function(t){a.colPastSit=t.data,setTimeout((function(){a.loading=!1}),500)}))},getMeeting:function(t,e){var a=this;
    )),axios.get("/mun-list/0").then((function(t){e.munList=t.data,setTimeout((function(){e.loading=!1}),500)})),axios.get("/coll-list/".concat(t,"/2")).then((function(t){e.pgList=t.data,setTimeout((function(){e.loading=!1}),500)})),axios.get("/coal-list/".concat(t)).then((function(t){e.coalList=t.data,setTimeout((function(){e.loading=!1}),500)}))}}},D=(a("FVlf"),Object(c.a)(k,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",[a("bc",{attrs:{path:t.meta}}),
    is.getProfile(this.$route.params.lang,this.$route.params.mpId)},methods:{getProfile:function(t,e){var a=this;axios.get("/mp-profile/".concat(t,"/").concat(e)).then((function(t){a.mp=t.data,setTimeout((function(){a.loading=!1}),500)}))}}},x=Object(c.a)(w,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",[a("bc",{attrs:{path:t.meta}}),t._v(" "),a("h1",{staticClass:"p-container-title"},[t._v("\n        "+t._s(t.meta.C_NavL_value)+"\n    ")]),t._v(" "),t.loa
             "),e.Pl_Sten_id>0?t._t("default",[a("em",[a("router-link",{attrs:{to:{path:"/"+t.$route.params.lang+"/plenaryst/ns/55/ID/"+e.Pl_Sten_id},title:"Пленарно заседание"}},[t._v("\n                                            Пленарно заседание\n                                        ")])],1)]):t._t("default",[a("em",[a("router-link",{attrs:{to:{path:"/"+t.$route.params.lang+"/parliamentarycommittees/"+e.A_ns_C_id+"/steno/"+e.A_Cm_Stid},title:e.A_ns_CL_value}},[t._v(
    all,(function(e){return a("li",{key:e.L_Act_A_id},[a("router-link",{attrs:{to:{path:"/"+t.$route.params.lang+"/plenaryst/ns/55/ID/"+e.Pl_Sten_id},title:" Пленарна зала"}},[t._v("\n                                    "+t._s(t._f("moment")(e.L_Act_St_date,"DD/MM/YYYY"))+"\n                                    Пленарна зала -\n                                    "+t._s(e.L_Act_St_name)+"\n                                    ("+t._s(e.L_Act_St2_name)+")\n
    v(" "),a("archive",{attrs:{model:"Pl_StenV",index:"0",index2:"0",title:"Стенограма от пленарно заседание",url:"plenaryst/ns/55/ID/"}})],1)],1)}),[],!1,null,"649ab6f4",null).exports,Jt={components:{Spin:o.a,Bc:v.a},props:["colT","url"],data:function(){return{loading:!0,meta:"",app_title:"2",filterFn:"",currentPage:1,perPage:30,totalRows:0,sortBy:null,sortDesc:!1,fn:{},dossT:[],dossTh:[],fnFields:[{key:"RN_FULL",label:"Номер на акт",sortable:!0},{key:"ActT_name",label:"Ви
    function(e){t.meta=e})),this.getProfile(this.$route.params.lang)},methods:{getProfile:function(t){var e=this;axios.get("/archive/".concat(t)).then((function(t){e.listNs=t.data,setTimeout((function(){e.loading=!1}),500)}))}}},Ye=Object(c.a)(Te,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",[a("bc",{attrs:{path:t.meta}}),t._v(" "),a("h1",{staticClass:"p-container-title"},[t._v(t._s(t.meta.C_NavL_value))]),t._v(" "),a("div",[a("b-card",{staticClass:"p
    "ar_count2"):"A_ns_C_total_count"==t?this.$t("ar_count3"):this.$t("ar_Name")},getNS:function(t,e){var a=this;axios.get("/archive/".concat(t,"/").concat(e)).then((function(t){a.nsData=t.data,setTimeout((function(){a.loading=!1}),500)}))},getProfile:function(t,e,a){var s=this;a?(a=a,this.rqDate=a):(a="",this.rqDate=new Date),axios.get("/coll-list-ns/".concat(t,"?nsId=").concat(e,"&date=").concat(a)).then((function(t){s.collListMP=t.data,setTimeout((function(){s.loading=!1}),5
    00)})),axios.get("/coll-list-pr/".concat(t,"?nsId=").concat(e,"&date=").concat(a)).then((function(t){s.collListMPr=t.data,setTimeout((function(){s.loading=!1}),500)}))}}},Fe=(a("B2Cn"),Object(c.a)(Ie,(function(){var t=this,e=t.$createElement,a=t._self._c||e;return a("div",[a("bc",{attrs:{path:t.meta}}),t._v(" "),a("h1",{staticClass:"p-container-title"},[t._v("\n        "+t._s
    "moment")(e.L_ActD_date_ds,"DD/MM/YYYY"))+"\n                    ")])])})),0)])],1),t._v(" "),a("b-row",{staticClass:"p-archive-list"},t._l(t.archiveYear,(function(e){return a("b-col",{key:e.t_year,staticClass:"col-12 col-sm-6 col-lg-2 col-md-2"},[a("div",{staticClass:"archive-head"},[t._v("\n                "+t._s(e.t_year)+"\n            ")]),t._v(" "),a("ul",t._l(e.t_month_list,(function(s){return a("li",{key:s.t_month},[a("span",{directives:[{name:"scroll-to",rawName:"v-scr
    s(t._f("moment")(e.Sb_date,"DD/MM/YYYY"))+"\n                    ")])])})),0)])],1),t._v(" "),a("b-row",{staticClass:"p-archive-list"},t._l(t.archiveYear,(function(e){return a("b-col",{key:e.t_year,staticClass:"col-12 col-sm-6 col-lg-2 col-md-2"},[a("div",{staticClass:"archive-head"},[t._v("\n                "+t._s(e.t_year)+"\n            ")]),t._v(" "),a("ul",t._l(e.t_month_list,(function(s){return a("li",{key:s.t_month},[a("span",{directives:[{name:"scroll-to",rawName:"v-scr
    component:Oe,props:{url:"doc"}},{path:"doc/ID/:id",name:"PlDoc profile",component:Oe,props:{url:"doc"}},{path:"plenaryst/ns/:ns/ID/:id",name:"Steno profile",component:Wt,props:{url:"plenaryst"}},{path:"parliamentarycontrol",name:"PC control",component:vt,props:{url:"parliamentarycontrol"}},{path:"parliamentarycontrol/ID/:id",name:"PC control profile",component:vt,props:{url:"parliamentarycontrol"}},{path:"voteofnoconfidence",name:"VoteNoc",component:ht,props:{url:"voteo
    /"),ua=a.n(da),pa=a("yvkt"),va=a("pKFz"),fa=a.n(va),ha=(a("bfyA"),a("9rmn"),{archiveList:function(t,e,a,s,n){axios.get("archive-list/".concat(t,"/").concat(e,"/").concat(a,"/").concat(s)).then((function(t){n(t.data)})).catch((function(t){handleError(t)}))},periodList:function(t,e,a,s,n,i,l){axios.get("archive-period/".concat(t,"/").concat(e,"/").concat(a,"/").concat(s,"/").concat(n,"/").concat(i)).then((function(t){l(t.data)})).catch((function(t){handleError(t)}))}}),ba={getMet
    lX:!0},toggledNav:!1,langs:["bg","en"],links:[{title:"Home",href:"/"},{title:"BB",href:"/bg/blog"},{title:"ns",href:"/bg/ns"},{title:"New",href:"/bg/create"}],navigation:[]}},mounted:function(){this.loadNav(this.$i18n.locale,1)},watch:{"$i18n.locale":function(){this.loadNav(this.$i18n.locale,1)},$route:function(){$("#navbarContent").collapse("hide")}},methods:{setLocale:function(t){this.$i18n.locale=t,this.$router.push({params:{lang:t}})},loadNav:function(t,e){var a=thi
    ✓ • 0ms

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/js/app.js | rg -o '.{0,160}(archiveList\(|periodList\(|t.comSteno).{0,280}' | head -n 30
    0,meta:"",setYear:"",setMonth:"",archiveYear:[],archivePeriod:[]}},watch:{"$route.params.id":function(){}},mounted:function(){var t=this;this.$services.archive.archiveList(this.$route.params.lang,this.model,this.index,this.index2,(function(e){t.archiveYear=e}))},methods:{periodList:function(t,e,a,s){var n=this;this.$services.archive.periodList(this.$route.params.lang,this.model,t,e,this.index,this.index2,(function(t){n.archivePeriod=t}))}}},n=a("KH
    l-to",rawName:"v-scroll-to",value:"#period-title",expression:"'#period-title'"}],attrs:{"aria-label":s.C_18n_value,title:s.C_18n_value},on:{click:function(a){t.periodList(e.t_year,s.t_month,0,0),t.setYear=e.t_year,t.setMonth=s.C_18n_value}}},[t._v("\n                        "+t._s(s.C_18n_value)+"\n                    ")])])})),0)])})),1)],1)}),[],!1,null,"64b176c4",null);e.a=i.exports},Lh3N:function(t,e,a){var s=a("9hjc");"string"==typeof s&&(s=[
    ",[t.$route.params.stenoId>0?a("b-row",{staticClass:"p-obj"},[a("b-col",[a("h2",{staticClass:"p-obj-title"},[t._v("\n                                    "+t._s(t.comSteno.A_Cm_St_sub)+"\n                                ")]),t._v(" "),a("div",{staticClass:"p-sub"},[t._v("\n                                    "+t._s(t._f("moment")(t.comSteno.A_Cm_St_date,"DD/MM/YYYY"))+"\n                                ")]),t._v(" "),a("ul",t._l(t.comSteno.acts,(f
               "+t._s(e.L_Act_sign)+",\n                                        ")])],1)})),0),t._v(" "),a("div",{staticClass:"p-obj-body",domProps:{innerHTML:t._s(t.comSteno.A_Cm_St_text)}})])],1):t._e(),t._v(" "),a("archive",{attrs:{model:"A_Cm_Steno",index:t.$route.params.id,index2:"0",title:"",url:"parliamentarycommittees/"+t.$route.params.id+"/steno/",query:""}})],1)],2):t._e(),t._v(" "),3==t.colT&&"bg"==t.$route.params.lang?a("b-tab",{attrs:{ac
    e,this.fn.search=1}},mounted:function(){var t=this;this.$services.meta.getMeta(this.$route.params.lang,this.url,(function(e){t.meta=e})),this.$services.archive.archiveList(this.$route.params.lang,this.model,0,0,(function(e){t.archiveYear=e})),this.$route.query.date&&(this.fn.date2=this.$route.query.date,this.fn.search=1),this.setFunnel(),this.getData()},methods:{setFunnel:function(){var t=this;axios.get("/pl-doc",{params:this.fn}).then((function(e)
    e:"scroll-to",rawName:"v-scroll-to",value:"#doc-list",expression:"'#doc-list'"}],attrs:{title:s.C_18n_value,"aria-label":s.C_18n_value},on:{click:function(a){t.periodList(e.t_year,s.t_month),t.setYear=e.t_year,t.setMonth=s.C_18n_value}}},[t._v("\n                        "+t._s(s.C_18n_value)+"\n                    ")])])})),0)])})),1)],1)}),[],!1,null,"fc574c04",null).exports,Be={components:{Spin:o.a,Archive:b.a,Bc:v.a},props:["url"],data:function
    is.url)}]}},watch:{},mounted:function(){var t=this;this.$services.meta.getMeta(this.$route.params.lang,this.url,(function(e){t.meta=e})),this.$services.archive.archiveList(this.$route.params.lang,this.model,0,0,(function(e){t.archiveYear=e}))},methods:{periodList:function(t,e){var a=this;axios.get("/period-sebra/".concat(t,"/").concat(e)).then((function(t){a.periodBulletin=t.data,setTimeout((function(){a.loading=!1}),500)}))}}},je=Object(c.a)(Be,(f
    e:"scroll-to",rawName:"v-scroll-to",value:"#doc-list",expression:"'#doc-list'"}],attrs:{title:s.C_18n_value,"aria-label":s.C_18n_value},on:{click:function(a){t.periodList(e.t_year,s.t_month),t.setYear=e.t_year,t.setMonth=s.C_18n_value}}},[t._v("\n                        "+t._s(s.C_18n_value)+"\n                    ")])])})),0)])})),1)],1)}),[],!1,null,"7e5899aa",null).exports,qe={components:{Spin:o.a,Bc:v.a},props:["url"],data:function(){return{lo
    ✓ • 0ms

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/api/v1/archive-list/bg/A_Cm_Steno/3797/0
    [{"t_year":2026,"t_month_list":[{"t_month":6,"C_18n_value":"\u044e\u043d\u0438"},{"t_month":7,"C_18n_value":"\u044e\u043b\u0438"},{"t_month":9,"C_18n_value":"\u0441\u0435\u043f\u0442\u0435\u043c\u0432\u0440\u0438"},{"t_month":10,"C_18n_value":"\u043e\u043a\u0442\u043e\u043c\u0432\u0440\u0438"}]}]
    ✓ • 670ms

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/api/v1/archive-period/bg/Pl_StenV/2026/10/0/0
    [{"t_id":11190,"t_label":11190,"t_date":"2026-10-21"},{"t_id":11189,"t_label":11189,"t_date":"2026-10-16"},{"t_id":11188,"t_label":11188,"t_date":"2026-10-15"},{"t_id":11186,"t_label":11186,"t_date":"2026-10-14"},{"t_id":11187,"t_label":11187,"t_date":"2026-10-13"},{"t_id":11185,"t_label":11185,"t_date":"2026-10-09"},{"t_id":11184,"t_label":11184,"t_date":"2026-10-08"},{"t_id":11183,"t_label":11183,"t_date":"2026-10-07"},{"t_id":11181,"t_label":11181,"t_date":"2026-10-02"},{"t_id":11180,"t_label":11180,"t_date":"2026-10-01"}]
    ✓ • 0ms

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/api/v1/archive-period/bg/A_Cm_Steno/2026/10/3797/0
    [{"t_id":9934,"t_label":"\u0421\u044a\u043a\u0440\u0430\u0442\u0435\u043d \u043f\u0440\u043e\u0442\u043e\u043a\u043e\u043b \u043e\u0442 \u0437\u0430\u0441\u0435\u0434\u0430\u043d\u0438\u0435 \u2116 7","t_date":"2026-10-07"}]
    ✓ • 523ms

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/api/v1/pl-sten/11190 | python3 -c 'import sys,json; d=json.load(sys.stdin); print(json.dumps({k: ({"length":len(v),"sample":v[:250]} if isinstance(v,str) and len(v)>800 else v) for k,v in d.items()},ensure_ascii=False,indent=2))'
    {
      "Pl_Sten_id": 11190,
      "Pl_Sten_date": "2026-10-21",
      "Pl_Sten_sub": "ПЕТДЕСЕТ И ВТОРО НАРОДНО СЪБРАНИЕ, ВТОРО ИЗВЪНРЕДНО ЗАСЕДАНИЕ, София, сряда, 21 октомври 2026 г.",
      "Pl_Sten_body": "Председателствал: председателят Михаела Доцова <br />\r\n<br />\r\nСекретари: Тансер Бейти и Любен Иванов<br />\r\n<br />\r\n9,02 ч.<br />\r\nПРЕДСЕДАТЕЛ МИХАЕЛА ДОЦОВА: Добро утро, уважаеми госпожи и господа народни представители!<br />\r\nМоля за режим на регистрация.<br />\r\nНяма кворум.<br />\r\nСледваща регистрация след 30 минути.<br />\r\n<br />\r\n9,31 ч.<br />\r\nПРЕДСЕДАТЕЛ МИХАЕЛА ДОЦОВА: Уважаеми госпожи и господа народни представители, моля за режим на регистрация.<br />\r\nУважаеми колеги, няма кворум, поради което заседание не може да бъде проведено.<br />\r\n<br />\r\n9,33 ч.<br />\r\n<br />\r\nПредседател:<br />\r\nМихаела Доцова<br />\r\n<br />\r\nСекретари: <br />\r\nТансер Бейти<br />\r\nЛюбен Иванов",
      "files": [
        {
          "Pl_StenDid": 18899,
          "Pl_StenDname": "Гласуване по парламентарни групи",
          "Pl_StenDfile": "/pub/StenD/20261021135824_gv211026-provaleno.pdf",
          "Pl_StenDtype": "pdf"
        },
        {
          "Pl_StenDid": 18900,
          "Pl_StenDname": "Гласуване по парламентарни групи",
          "Pl_StenDfile": "/pub/StenD/20261021135831_gv211026-provaleno.xlsx",
          "Pl_StenDtype": "xls"
        },
        {
          "Pl_StenDid": 18901,
          "Pl_StenDname": "Гласуване по парламентарни групи",
          "Pl_StenDfile": "/pub/StenD/20261021135838_gv211026-provaleno.csv",
          "Pl_StenDtype": "xls"
        },
        {
          "Pl_StenDid": 18902,
          "Pl_StenDname": "Поименно гласуване",
          "Pl_StenDfile": "/pub/StenD/20261021135853_iv211026-provaleno.pdf",
          "Pl_StenDtype": "pdf"
        },
        {
          "Pl_StenDid": 18903,
          "Pl_StenDname": "Поименно гласуване",
          "Pl_StenDfile": "/pub/StenD/20261021135859_iv211026-provaleno.xlsx",
          "Pl_StenDtype": "xls"
        },
        {
          "Pl_StenDid": 18904,
          "Pl_StenDname": "Поименно гласуване",
          "Pl_StenDfile": "/pub/StenD/20261021135906_iv211026-provaleno.csv",
          "Pl_StenDtype": "xls"
        }
      ],
      "video": [],
      "videoArchive": false
    }
    ✓ • 0ms

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/api/v1/com-steno/bg/9934 | python3 -c 'import sys,json; d=json.load(sys.stdin); print(json.dumps({k: ({"length":len(v),"sample":v[:900]} if isinstance(v,str) and len(v)>1200 else v) for k,v in d.items()},ensure_ascii=False,indent=2))'
    {
      "A_Cm_St_date": "2026-10-07",
      "A_Cm_St_sub": "Съкратен протокол от заседание № 7",
      "A_Cm_St_text": {
        "length": 1630,
        "sample": "Съкратен протокол от заседание № 7<br />\r\nНа 7 октомври 2026 г. се проведе редовно заседание на Комисията по външна политика при следния<br />\r\nДНЕВЕН РЕД:<br />\r\n1.\tЗаконопроект за ратифициране на Споразумението между Международната банка за икономическо сътрудничество (МБИС) и правителството на Република България за окончателно уреждане на взаимните претенции и задължения във връзка с оттеглянето на Република България от Споразумението за организацията и дейността на МБИС и от членството в МБИС, №52-602-02-19, внесен от Министерския съвет на 28.09.2026 г. <br />\r\n2.\tЗаконопроект за ратифициране на Конвенцията за Бюрото на Европейската конференция за гражданска авиация, №52-602-02-20, внесен от Министерския съвет на 28.09.2026 г.<br />\r\n3.\tАктуални въпроси към министъра на външните работи в съответствие с чл. 28 от Правилника за организацията и дейността на Народното събрание. (точката "
      },
      "A_Cm_Stid": 9934,
      "acts": []
    }
    ✓ • 0ms

## User

continue

## Assistant

The source inspection uncovered a distinction we need to handle: **some committee records are abbreviated minutes rather than full transcripts**. The committee sample I checked is explicitly titled “Съкратен протокол.” We should preserve that distinction in search results and avoid presenting summaries as verbatim speeches. [Sample committee record](https://www.parliament.bg/bg/parliamentarycommittees/3797/steno/9934)

I also verified that a plenary record exposes its transcript text, date, voting-file links, and video links through the site’s data API. The transcript itself is a large HTML text field, so identifying individual speeches will still require parsing. [Sample plenary record](https://www.parliament.bg/api/v1/pl-sten/11050)

I’ll continue from those findings. Your original party-and-policy example gives us a starting research workflow; you can add the more specific search examples whenever you have them.

## Assistant

That gives us a clear search contract: include committee minutes with explicit labels, and show **direct matches** and **related-by-meaning matches** as separate result groups.

We now have enough product direction to compare the backend options properly. I’m checking the search capabilities and deployment requirements before recommending the stack; the main remaining infrastructure unknown is what “free hosting” provides in terms of memory and CPU.

## Assistant

What hosting will you have available: a server or VPS with Docker access, a university or hackathon platform, or something else? If you know the RAM, CPU, and disk limits, include them; “not decided yet” is a useful answer too. This determines whether a dedicated search engine fits comfortably.

## Activity

    Searched the web for site.opensearch.org about Apache 2.0 open source search, site.docs.opensearch.org latest install-and-configure install-opensearch docker memory 4 GB, site.docs.opensearch.org latest field types knn-vector filter highlight bulgarian, site.cloud.google.com vertex-ai generative-ai docs embeddings get-text-embeddings Bulgarian gemini-embedding-001

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/api/v1/archive/bg | python3 -c 'import sys,json; d=json.load(sys.stdin); print(json.dumps(d[:3] if isinstance(d,list) else d,ensure_ascii=False,indent=2)[:5000])'
    [
      {
        "A_ns_id": 61,
        "A_ns_start": "2024-11-11",
        "A_ns_end": "2026-04-30",
        "A_nsL_value": "ПЕТДЕСЕТ И ПЪРВО НАРОДНО СЪБРАНИЕ",
        "pg": 11,
        "cm": 28,
        "cm_v": 6,
        "dl": 10,
        "cm_u": 0,
        "fg": 96,
        "pc": 0
      },
      {
        "A_ns_id": 60,
        "A_ns_start": "2024-06-19",
        "A_ns_end": "2024-11-11",
        "A_nsL_value": "ПЕТДЕСЕТО НАРОДНО СЪБРАНИЕ",
        "pg": 8,
        "cm": 25,
        "cm_v": 4,
        "dl": 10,
        "cm_u": 0,
        "fg": 95,
        "pc": 0
      },
      {
        "A_ns_id": 59,
        "A_ns_start": "2023-04-12",
        "A_ns_end": "2024-06-19",
        "A_nsL_value": "ЧЕТИРИДЕСЕТ И ДЕВЕТО НАРОДНО СЪБРАНИЕ",
        "pg": 7,
        "cm": 26,
        "cm_v": 11,
        "dl": 10,
        "cm_u": 4,
        "fg": 95,
        "pc": 0
      }
    ]
    ✓ • 245ms

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/api/v1/coll-list/bg/2 | python3 -c 'import sys,json; d=json.load(sys.stdin); print(json.dumps(d[:3] if isinstance(d,list) else d,ensure_ascii=False,indent=2)[:5000])'
    [
      {
        "A_ns_C_id": 3763,
        "A_ns_CT_id": 2,
        "A_ns_CL_value": "Парламентарна група \"Прогресивна България\"",
        "A_ns_C_active_count": 131,
        "A_ns_C_date_F": "2026-04-30",
        "A_ns_C_date_T": "9999-12-31"
      },
      {
        "A_ns_C_id": 3764,
        "A_ns_CT_id": 2,
        "A_ns_CL_value": "Парламентарна група на \"ГЕРБ – СДС\"",
        "A_ns_C_active_count": 39,
        "A_ns_C_date_F": "2026-04-30",
        "A_ns_C_date_T": "9999-12-31"
      },
      {
        "A_ns_C_id": 3766,
        "A_ns_CT_id": 2,
        "A_ns_CL_value": "Парламентарна група \"Движение за права и свободи – ДПС\"",
        "A_ns_C_active_count": 21,
        "A_ns_C_date_F": "2026-04-30",
        "A_ns_C_date_T": "9999-12-31"
      }
    ]
    ✓ • 0ms

## Assistant

My current recommendation is **PostgreSQL for the authoritative records, plus OpenSearch for retrieval**, with the Go API and ingestion worker sharing one codebase.

OpenSearch has Bulgarian language analysis and vector-search support, and its Apache 2.0 licensing makes it a straightforward self-hosted candidate. Elasticsearch remains a capable alternative. [Language support](https://docs.opensearch.org/latest/analyzers/language-analyzers/index/), [Vector search](https://docs.opensearch.org/latest/mappings/supported-field-types/knn-vector/), [OpenSearch licensing](https://opensearch.org/About/)

The separation has a practical purpose: if we change topic assignments, passage boundaries, or ranking, we can rebuild the search index from preserved records. The deployment would need enough memory for both database and search services; I’m leaving that recommendation conditional on the hosting details.

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/api/v1/coll-list-mp/bg/3763/2 | python3 -c 'import sys,json; d=json.load(sys.stdin); s=lambda x: ({k: (list(v[0].keys()) if v and isinstance(v[0],dict) else len(v)) if isinstance(v,list) else list(v.keys()) if isinstance(v,dict) else str(v)[:180] for k,v in x.items()} if isinstance(x,dict) else {"count":len(x),"first_keys":list(x[0].keys()) if x and isinstance(x[0],dict) else []}); print(json.dumps(s(d),ensure_ascii=False,indent=2))'
    {
      "A_ns_C_id": "3763",
      "A_ns_CT_id": "2",
      "A_ns_C_date_F": "2026-04-30",
      "A_ns_C_parent_id": "0",
      "A_ns_CL_value": "Парламентарна група \"Прогресивна България\"",
      "A_ns_CL_desc": "None",
      "A_ns_C_active_count": "131",
      "A_ns_CDid": "549",
      "A_ns_CDemail": "52_PB@parliament.bg",
      "A_ns_CDroom": "None",
      "A_ns_CDphone": "None",
      "A_ns_CDrules": "",
      "A_ns_CDmembers": "None",
      "A_ns_CDend": "None",
      "L_Act_id_start": "None",
      "L_Act_id_end": "None",
      "L_Act_date2_start": "None",
      "L_Act_date2_end": "None",
      "L_ActL_final_start": "None",
      "L_ActL_final_end": "None",
      "pc": "None",
      "colListMP": [
        "A_ns_C_id",
        "A_ns_CL_value",
        "A_ns_CL_value_short",
        "A_ns_MSP_date_F",
        "A_ns_MSP_date_T",
        "A_ns_MP_FM",
        "A_ns_MP_id",
        "A_ns_MP_leg_count",
        "A_ns_MP_com_count",
        "A_ns_MP_del_count",
        "A_ns_MP_frd_count",
        "A_ns_MPL_Name1",
        "A_ns_MPL_Name2",
        "A_ns_MPL_Name3",
        "A_ns_MP_PosL_value",
        "A_ns_Va_name",
        "A_ns_MP_img"
      ]
    }
    ✓ • 505ms

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/api/v1/archive/bg/62 | python3 -c 'import sys,json; d=json.load(sys.stdin); print(json.dumps(d,ensure_ascii=False,indent=2)[:3500])'
    curl: (22) The requested URL returned error: 500
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
        import sys,json; d=json.load(sys.stdin); print(json.dumps(d,ensure_ascii=False,indent=2)[:3500])
                           ~~~~~~~~~^^^^^^^^^^^
      File "/usr/lib/python3.14/json/__init__.py", line 298, in load
        return loads(fp.read(),
            cls=cls, object_hook=object_hook,
            parse_float=parse_float, parse_int=parse_int,
            parse_constant=parse_constant, object_pairs_hook=object_pairs_hook, **kw)
      File "/usr/lib/python3.14/json/__init__.py", line 352, in loads
        return _default_decoder.decode(s)
               ~~~~~~~~~~~~~~~~~~~~~~~^^^
      File "/usr/lib/python3.14/json/decoder.py", line 345, in decode
        obj, end = self.raw_decode(s, idx=_w(s, 0).end())
                   ~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^
      File "/usr/lib/python3.14/json/decoder.py", line 363, in raw_decode
        raise JSONDecodeError("Expecting value", s, err.value) from None
    json.decoder.JSONDecodeError: Expecting value: line 1 column 1 (char 0)
    ✗ (1) • 0ms

## Assistant

The implementation plan can now cover the data model, ingestion, search, reader, and agent documentation in detail. One scheduling decision remains: the **minimum acceptable November 5 demo**. This sets what gets protected if integration or source parsing takes longer than expected; the longer-term scope stays as we’ve discussed.

## Activity

    $ git status --short
    ✓ • 0ms

## Activity

    $ go version
    go version go1.27.2-X:nodwarf5 linux/amd64
    ✓ • 0ms

## Activity

    $ node --version
    v26.12.0
    ✓ • 0ms

## Activity

    $ docker compose version
    Docker Compose version v2.29.7-desktop.1
    ✓ • 48ms

## Activity

    $ curl --fail --silent --show-error --location --max-time 25 https://www.parliament.bg/js/app.js | rg -o 'axios.get\("/[^\"]*"' | sort -u
    axios.get("/"
    axios.get("/act-pr/"
    axios.get("/act/"
    axios.get("/archive/"
    axios.get("/bill/"
    axios.get("/calendar-list/"
    axios.get("/calendar-live/"
    axios.get("/calendar/"
    axios.get("/civil-contracts"
    axios.get("/coal-list/"
    axios.get("/coll-list-mp/"
    axios.get("/coll-list-ns/"
    axios.get("/coll-list-pr/"
    axios.get("/coll-list/"
    axios.get("/com-acts/"
    axios.get("/com-docs/"
    axios.get("/com-meeting/"
    axios.get("/com-news/"
    axios.get("/com-past-news/"
    axios.get("/com-past-sit/"
    axios.get("/com-stan/"
    axios.get("/com-steno/"
    axios.get("/conflict-decl-list/1"
    axios.get("/conflict-decl-list/2"
    axios.get("/conflict-register"
    axios.get("/contacts/"
    axios.get("/eu-docs/"
    axios.get("/eu-dossier-theme"
    axios.get("/eu-dossier-type"
    axios.get("/eu-dossier/"
    axios.get("/eu-options"
    axios.get("/front-act-list"
    axios.get("/front-news/"
    axios.get("/gallery/"
    axios.get("/links/"
    axios.get("/mp-profile/"
    axios.get("/mun-list/0"
    axios.get("/nav-list/"
    axios.get("/ncpi"
    axios.get("/news/"
    axios.get("/parl-register"
    axios.get("/period-sebra/"
    axios.get("/petitions-list/"
    axios.get("/pl-blic/"
    axios.get("/pl-control/"
    axios.get("/pl-debate/"
    axios.get("/pl-doc"
    axios.get("/pl-doc-period/"
    axios.get("/pl-doc-type"
    axios.get("/pl-doc/"
    axios.get("/pl-enquiry/"
    axios.get("/pl-hearing/"
    axios.get("/pl-sten/"
    axios.get("/plenaryprogram/"
    axios.get("/proc-status"
    axios.get("/proc-type"
    axios.get("/proc/"
    axios.get("/public-consult/"
    axios.get("/referendum"
    axios.get("/referendum-info"
    axios.get("/referendum/"
    axios.get("/reg35"
    axios.get("/students-rs"
    axios.get("/students-rs/"
    axios.get("/ts-list/bg"
    axios.get("/ts-preview/"
    axios.get("/ts-profile/bg/"
    axios.get("/upcoming-sittings/"
    axios.get("/va-list"
    axios.get("/video-archive/"
    axios.get("/video-archive2/"
    axios.get("/vote-list"
    axios.get("/vote-noc-list"
    axios.get("/vote-noc/"
    axios.get("/vote/"
    ✓ • 1.33s

## Assistant

The **November 5 minimum** is now clear: real parliamentary data, Bulgarian prompt → editable search criteria → relevant excerpts → the Nophyx transcript reader. Broader coverage, semantic results, and richer topic features can follow if needed.

My recommended architecture is:

| Component | Role |
|---|---|
| React + Vite + TypeScript | Search interface, criteria editor, results, and transcript reader. |
| Go API and worker | Serve searches and documents; collect, parse, classify, and index records. Both share one codebase. |
| PostgreSQL | Authoritative documents, source versions, speakers, historical affiliations, topics, and job state. |
| OpenSearch | Bulgarian keyword search, filters, excerpts, and the separate semantic-results group. |
| Gemini | Suggest search criteria and classify passages; embedding access still needs verification. |
| Docker Compose | Run the services together locally and on a suitable server. |

OpenSearch supports both [Bulgarian analysis](https://docs.opensearch.org/latest/analyzers/language-analyzers/index/) and [vector search](https://docs.opensearch.org/latest/mappings/supported-field-types/knn-vector/). Elasticsearch is also capable; my preference for OpenSearch here is its straightforward [Apache 2.0 licensing](https://opensearch.org/About/).

The key architectural rule is that **the search index is rebuildable**. Original records and their relationships remain preserved when we change classification or ranking. Searching the text also doesn’t require us to anticipate every possible topic tag.

For your LLM-assisted development, the documentation will include:

- `AGENTS.md`: persistent coding-agent instructions and a map to relevant documents.
- Product and architecture specifications, including data contracts and failure handling.
- Detailed milestone plans with acceptance checks and recorded progress.
- Decision records explaining important choices.
- `docs/backlog.md` for feature ideas, dependencies, and priorities.

Codex recognizes [`AGENTS.md` as project guidance](https://learn.chatgpt.com/docs/agent-configuration/agents-md); the other documents will be explicitly referenced from it.

One deployment detail remains before I can firmly size this setup: **what does your available hosting look like?** A Linux server with persistent storage and around 8 GB RAM would be a reasonable initial planning target, subject to measurement. If hosting is undecided, say so—we can document that assumption and develop locally first. Docker Compose is already installed in this workspace.

## User

> What hosting will you have available: a server or VPS with Docker access, a university or hackathon platform, or something else? If you know the RAM, CPU, and disk limits, include them; “not decided yet” is a useful answer too. This determines whether a dedicated search engine fits comfortably.
i have a 3060ti, intel i5 11th gen and 16gb of ram. i may be able to get more compute tho

## Assistant

That gives us a reasonable starting machine for development and the hackathon demo. I’ll check the resources visible to this workspace and the search engine’s memory guidance before setting the Docker limits. The GPU won’t be needed for keyword search or calls to Gemini; it could help later if you choose to run embedding models locally.

## Activity

    $ cat /etc/os-release /sys/fs/cgroup/memory.max
    NAME="Arch Linux"
    PRETTY_NAME="Arch Linux"
    ID=arch
    BUILD_ID=rolling
    ANSI_COLOR="38;2;23;147;209"
    HOME_URL="https://archlinux.org/"
    DOCUMENTATION_URL="https://wiki.archlinux.org/"
    SUPPORT_URL="https://bbs.archlinux.org/"
    BUG_REPORT_URL="https://gitlab.archlinux.org/groups/archlinux/-/issues"
    PRIVACY_POLICY_URL="https://terms.archlinux.org/docs/privacy-policy/"
    LOGO=archlinux-logo
    cat: /sys/fs/cgroup/memory.max: No such file or directory
    ✗ (1) • 0ms

## Activity

    $ free -h
                   total        used        free      shared  buff/cache   available
    Mem:            15Gi       7.1Gi       2.6Gi       108Mi       6.3Gi       8.4Gi
    Swap:           13Gi          0B        13Gi
    ✓ • 0ms

## Activity

    $ lscpu
    Architecture:                            x86_64
    CPU op-mode(s):                          32-bit, 64-bit
    Address sizes:                           39 bits physical, 48 bits virtual
    Byte Order:                              Little Endian
    CPU(s):                                  12
    On-line CPU(s) list:                     0-11
    Vendor ID:                               GenuineIntel
    Model name:                              12th Gen Intel(R) Core(TM) i5-12400F
    CPU family:                              6
    Model:                                   151
    Thread(s) per core:                      2
    Core(s) per socket:                      6
    Socket(s):                               1
    Stepping:                                2
    Microcode version:                       0x3e
    CPU(s) scaling MHz:                      87%
    CPU max MHz:                             4400.0000
    CPU min MHz:                             800.0000
    BogoMIPS:                                4992.00
    Flags:                                   fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault cat_l2 cdp_l2 ssbd ibrs ibpb stibp ibrs_enhanced tpr_shadow flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid rdt_a rdseed adx smap clflushopt clwb intel_pt sha_ni xsaveopt xsavec xgetbv1 xsaves split_lock_detect user_shstk avx_vnni dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp hwp_pkg_req hfi vnmi umip pku ospke waitpkg gfni vaes vpclmulqdq rdpid movdiri movdir64b fsrm md_clear serialize arch_lbr ibt flush_l1d arch_capabilities
    Virtualization:                          VT-x
    L1d cache:                               288 KiB (6 instances)
    L1i cache:                               192 KiB (6 instances)
    L2 cache:                                7.5 MiB (6 instances)
    L3 cache:                                18 MiB (1 instance)
    NUMA node(s):                            1
    NUMA node0 CPU(s):                       0-11
    Vulnerability Gather data sampling:      Not affected
    Vulnerability Ghostwrite:                Not affected
    Vulnerability Indirect target selection: Not affected
    Vulnerability Itlb multihit:             Not affected
    Vulnerability L1tf:                      Not affected
    Vulnerability Mds:                       Not affected
    Vulnerability Meltdown:                  Not affected
    Vulnerability Mmio stale data:           Not affected
    Vulnerability Old microcode:             Not affected
    Vulnerability Reg file data sampling:    Not affected
    Vulnerability Retbleed:                  Not affected
    Vulnerability Spec rstack overflow:      Not affected
    Vulnerability Spec store bypass:         Mitigation; Speculative Store Bypass disabled via prctl
    Vulnerability Spectre v1:                Mitigation; usercopy/swapgs barriers and __user pointer sanitization
    Vulnerability Spectre v2:                Mitigation; Enhanced / Automatic IBRS; IBPB conditional; PBRSB-eIBRS SW sequence; BHI BHI_DIS_S
    Vulnerability Srbds:                     Not affected
    Vulnerability Tsa:                       Not affected
    Vulnerability Tsx async abort:           Not affected
    Vulnerability Vmscape:                   Mitigation; IBPB before exit to userspace
    ✓ • 0ms

## Activity

    Opened https://docs.opensearch.org/latest/install-and-configure/install-opensearch/index/

## Assistant

This workspace reports an **i5-12400F, about 16 GB RAM, and roughly 8 GB currently available**. That supports keeping PostgreSQL and a single OpenSearch node in the initial architecture.

I’d start with a **4 GB memory limit for OpenSearch, including a 2 GB Java heap**, and roughly another 2 GB across PostgreSQL, the Go processes, and frontend development tools. These are starting budgets to test, not measured requirements. OpenSearch needs memory outside its Java heap as well. [Memory configuration guidance](https://docs.opensearch.org/latest/install-and-configure/install-opensearch/index/#important-settings)

We can begin with this machine and measure indexing time, search latency, and peak memory before arranging more compute. The remaining distinction is whether it will also serve the public website.

## Activity

    • Context compacted

## Assistant

Your PC is a reasonable starting point for development and a small demo. For public access, I’m checking how to give it an HTTPS link while keeping PostgreSQL and the search engine private. The main constraint will be keeping the PC awake and connected whenever people need to use it.

## Activity

    $ git status --short
    ✓ • 0ms

## Activity

    Opened https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/

## Activity

    Opened https://developers.cloudflare.com/tunnel/get-started/

## Assistant

I’d use Cloudflare Tunnel to route the public site to your Go server, which would serve both the built React frontend and the API. It connects outward from your PC, so it doesn’t require a public IP address or router port forwarding. [Cloudflare’s documentation](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/)

For the demo, a temporary address needs no domain or account, but changes whenever the tunnel restarts. A stable address requires a domain configured with Cloudflare. That choice affects what link you can put in the submission. [Temporary links](https://developers.cloudflare.com/tunnel/get-started/quick-tunnels/), [stable-address setup](https://developers.cloudflare.com/tunnel/get-started/)

## User

if i close this terminal and shutown my pc does this get saved

## Assistant

I’ll check how Codex saves and resumes terminal conversations so I can tell you exactly what will survive a shutdown.

## Activity

    $ cat /home/jorobg/.codex/skills/.system/openai-docs/SKILL.md
    ---
    name: "openai-docs"
    description: "Use for Codex models/pricing, scheduled tasks, skills, settings, setup, troubleshooting, customization, automations, and self-knowledge—including 'you,' 'your,' 'this app,' or 'this coding agent' when they refer to Codex—and for OpenAI APIs/products and ChatGPT Work. Also use for model choice/migration, prompting, SDKs, Responses, Realtime, agents, evals, and Chat/Work/Codex comparisons. Do not use for generic app/software tasks that merely mention Codex."
    metadata:
      short-description: "Codex models/pricing, scheduled tasks, skills, settings, setup, troubleshooting, and self-knowledge; OpenAI APIs and ChatGPT Work. 'You'/'this app' means Codex only."
    ---
    
    # OpenAI Docs
    
    Provide current, cited OpenAI product, API, model, and Codex guidance. Read zero or one primary reference.
    
    **First substantive action:** Search the user's exact requested official OpenAI documentation topic and any explicitly named model using a concise, topic-specific query of 2-6 essential terms. When an already-available direct official documentation search and page-retrieval capability is present, use it first: search, then fetch or open the matching official page before general web search. Otherwise, immediately use official-domain web search, then actually open or fetch the relevant official page. Complete this source order before reading a reference, inspecting local or repository files, running a Codex manual or model resolver, drafting a plan, or answering from memory. Use the actual fetched page, not a search snippet or an unopened link. If one official search or page does not establish the answer, search another appropriate official domain and actually open or fetch the result. Preserve the exact requested model; never substitute a newer model.
    
    **Only exception:** An explicitly requested, genuinely broad, cross-topic Codex setup, orientation, or system-map synthesis may use the manual first when shell execution and an allowed temporary cache are available. A specific Codex feature, setting, command, error, model, or requested citation remains docs-first. Mixed Chat/Work/Codex comparisons are official documentation questions, not manual-first Codex requests.
    
    For generic software tasks, answer the software task directly. OpenAI implementation, debugging, SDK, API, prompting, agent, and eval requests are not generic.
    
    For a straightforward factual or citation-only request, follow the source order and do not read a route reference. This includes straightforward API facts, ChatGPT Work or mixed Chat/Work/Codex comparisons, model tiers, aliases, Pro mode, reasoning settings, factual migration baselines, and narrow Codex facts. Prioritize `learn.chatgpt.com` for ChatGPT Work.
    
    ## Choose one primary route
    
    Use the first matching route, and read its reference only when the requested task needs that specialized workflow:
    
    - **Explicitly requested local documentation integration:** Read [integration guidance](references/mcp-diagnostics.md) only when the user explicitly requests that local integration.
    - **Model migration, upgrades, or model-specific prompting:** Read [model-migration.md](references/model-migration.md) for actual migration planning, implementation, dynamic target resolution, or prompt changes. Preserve an explicitly requested target.
    - **Model selection and comparisons:** Read [model-selection.md](references/model-selection.md) only when nuanced current, latest, default, cost, latency, quality, or modality tradeoffs need more guidance. Do not run a migration resolver for selection alone.
    - **Product, API, ChatGPT Work, and mixed Chat/Work/Codex documentation:** Read [official-docs.md](references/official-docs.md) only when fetched official pages leave source selection, API schemas, or the requested implementation unresolved. This route is not manual-first.
    - **Explicitly broad Codex setup, orientation, or cross-topic synthesis:** Read [codex-self-knowledge.md](references/codex-self-knowledge.md) when the eligible Codex manual or deeper Codex procedures are needed.
    
    Read at most one primary reference. Do not open every route, bundled model guide, or helper script. Read a supporting reference or run a helper only when the chosen workflow demonstrably needs it.
    
    ## Source and execution boundaries
    
    - Search, open, fetch, and cite only `developers.openai.com`, `platform.openai.com`, and `learn.chatgpt.com`. Cite the page that supports the claim. State uncertainty when official sources do not establish pricing, availability, account access, limits, or behavior.
    - Preserve an explicitly requested model for selection, migration, and prompting. Resolve an unspecified latest or current migration target only after searching and fetching current official guidance.
    - Use `references/latest-model.md` only as a disclosed fallback after current official model guidance does not answer the question. Read `references/upgrading-to-gpt-6-astra.md` only for an actual, requested GPT-6 migration; read `references/prompting-guide.md` only for requested prompting work.
    - Before building, running, editing, debugging, or testing an API-backed app or tool, use `openai-platform-api-key` first when available. Documentation, conceptual examples, model selection, and read-only guidance do not require an API key.
    - Say "OpenAI Docs" or "official OpenAI documentation" in user-facing answers. Keep exact official citations and examples concise.
    ✓ • 0ms

## Activity

    $ codex --help
    WARNING: proceeding, even though we could not create PATH aliases: Read-only file system (os error 30)
    Codex CLI
    
    If no subcommand is specified, options will be forwarded to the interactive CLI.
    
    Usage: codex [OPTIONS] [PROMPT]
           codex [OPTIONS] <COMMAND> [ARGS]
    
    Commands:
      agents            Browse all agent sessions on the shared local app-server daemon
      exec              Run Codex non-interactively [aliases: e]
      review            Run a code review non-interactively
      login             Manage login
      logout            Remove stored authentication credentials
      mcp               Manage external MCP servers for Codex
      plugin            Manage Codex plugins
      app-server        [experimental] Run the app server or related tooling
      remote-control    [experimental] Manage the app-server daemon with remote control enabled
      completion        Generate shell completion scripts
      update            Update Codex to the latest version
      doctor            Diagnose local Codex installation, config, auth, and runtime health
      sandbox           Run commands within a Codex-provided sandbox
      debug             Debugging tools
      apply             Apply the latest diff produced by Codex agent as a `git apply` to your local
                        working tree [aliases: a]
      resume            Resume a previous interactive session (picker by default; use --last to continue
                        the most recent)
      queue             Queue a message for an existing session
      archive           Archive a saved session by id or session name
      delete            Permanently delete a saved session by id or session name
      migrate-rollouts  Inspect or migrate legacy local sessions to paginated thread history
      unarchive         Unarchive a saved session by id or session name
      fork              Fork a previous interactive session (picker by default; use --last to fork the
                        most recent)
      cloud             [EXPERIMENTAL] Browse tasks from Codex Cloud and apply changes locally
      exec-server       [EXPERIMENTAL] Run the standalone exec-server service
      features          Inspect feature flags
      help              Print this message or the help of the given subcommand(s)
    
    Arguments:
      [PROMPT]
              Optional user prompt to start the session
    
    Options:
      -c, --config <key=value>
              Override a configuration value that would otherwise be loaded from `~/.codex/config.toml`.
              Use a dotted path (`foo.bar.baz`) to override nested values. The `value` portion is parsed
              as TOML. If it fails to parse as TOML, the raw string is used as a literal.
    
              Examples: - `-c model="o3"` - `-c 'sandbox_permissions=["disk-full-read-access"]'` - `-c
              shell_environment_policy.inherit=all`
    
          --enable <FEATURE>
              Enable a feature (repeatable). Equivalent to `-c features.<name>=true`
    
          --disable <FEATURE>
              Disable a feature (repeatable). Equivalent to `-c features.<name>=false`
    
          --remote <ADDR>
              Connect the TUI to a remote app server endpoint.
    
              Accepted forms: `ws://host:port`, `wss://host:port`, `unix://`, or `unix://PATH`.
    
          --remote-auth-token-env <ENV_VAR>
              Name of the environment variable containing the bearer token to send to a remote app
              server websocket
    
          --strict-config
              Error out when config.toml contains fields that are not recognized by this version of
              Codex
    
      -i, --image <FILE>...
              Optional image(s) to attach to the initial prompt
    
      -m, --model <MODEL>
              Model the agent should use
    
          --oss
              Use open-source provider
    
          --local-provider <OSS_PROVIDER>
              Specify which local provider to use (lmstudio or ollama). If not specified with --oss,
              will use config default or show selection
    
      -p, --profile <CONFIG_PROFILE_V2>
              Layer $CODEX_HOME/<name>.config.toml on top of the base user config
    
      -s, --sandbox <SANDBOX_MODE>
              Select the sandbox policy to use when executing model-generated shell commands
    
              [possible values: read-only, workspace-write, danger-full-access]
    
          --approve-for-me
              Route approval requests through automatic review using the workspace-write sandbox
    
          --dangerously-bypass-approvals-and-sandbox
              Skip all confirmation prompts and execute commands without sandboxing. EXTREMELY
              DANGEROUS. Intended solely for running in environments that are externally sandboxed
    
          --dangerously-bypass-hook-trust
              Run enabled hooks without requiring persisted hook trust for this invocation. DANGEROUS.
              Intended only for automation that already vets hook sources
    
      -C, --cd <DIR>
              Tell the agent to use the specified directory as its working root
    
          --worktree
              Run the session in a new managed Git worktree
    
          --add-dir <DIR>
              Additional directories that should be writable alongside the primary workspace
    
      -a, --ask-for-approval <APPROVAL_POLICY>
              Configure when the model requires human approval before executing a command
    
              Possible values:
              - on-request: The model decides when to ask the user for approval
              - never:      Never ask for user approval Execution failures are immediately returned to
                the model
    
          --search
              Enable live web search. When enabled, the native Responses `web_search` tool is available
              to the model (no per‑call approval)
    
          --no-alt-screen
              Disable alternate screen mode
    
              Runs the TUI in inline mode, preserving terminal scrollback history.
    
          --no-daemon
              Run without the shared background server, even if it is already running
    
      -h, --help
              Print help (see a summary with '-h')
    
      -V, --version
              Print version
    ✓ • 0ms

## Activity

    Searched the web for Codex CLI resume saved sessions

## Activity

    Opened https://learn.chatgpt.com/docs/cli/reference

## Activity

    Searched for '### `codex resume`' in https://learn.chatgpt.com/docs/cli/reference

## Assistant

**Yes—Codex saves conversations locally, so you can resume after shutting down your PC.** [Official OpenAI documentation](https://learn.chatgpt.com/docs/reference/troubleshooting)

When you return, open a terminal and run:

```bash
cd /home/jorobg/Coding/nophyx
codex resume --last
```

Alternatively, run `codex resume` to choose this conversation from the saved sessions. [Resume instructions](https://learn.chatgpt.com/docs/cli/reference#codex-resume)

Our decisions are currently in this conversation; **we haven’t created `AGENTS.md` or the project’s planning documents yet**. Resuming this session lets us continue where we left off.

## User

is there a way to transfer this session on another pc?

## Assistant

I’ll check the supported way to move a saved Codex session to another PC, including what you need to copy.

## Activity

    $ codex resume --help
    WARNING: proceeding, even though we could not create PATH aliases: Read-only file system (os error 30)
    Resume a previous interactive session (picker by default; use --last to continue the most recent)
    
    Usage: codex resume [OPTIONS] [SESSION_ID] [PROMPT]
    
    Arguments:
      [SESSION_ID]
              Session id (UUID) or session name. UUIDs take precedence if it parses. If omitted, use
              --last to pick the most recent recorded session
    
      [PROMPT]
              Optional user prompt to start the session
    
    Options:
      -c, --config <key=value>
              Override a configuration value that would otherwise be loaded from `~/.codex/config.toml`.
              Use a dotted path (`foo.bar.baz`) to override nested values. The `value` portion is parsed
              as TOML. If it fails to parse as TOML, the raw string is used as a literal.
    
              Examples: - `-c model="o3"` - `-c 'sandbox_permissions=["disk-full-read-access"]'` - `-c
              shell_environment_policy.inherit=all`
    
          --last
              Continue the most recent session without showing the picker
    
          --all
              Show all sessions (disables cwd filtering and shows CWD column)
    
          --enable <FEATURE>
              Enable a feature (repeatable). Equivalent to `-c features.<name>=true`
    
          --disable <FEATURE>
              Disable a feature (repeatable). Equivalent to `-c features.<name>=false`
    
          --include-non-interactive
              Include non-interactive sessions in the resume picker and --last selection
    
          --remote <ADDR>
              Connect the TUI to a remote app server endpoint.
    
              Accepted forms: `ws://host:port`, `wss://host:port`, `unix://`, or `unix://PATH`.
    
          --remote-auth-token-env <ENV_VAR>
              Name of the environment variable containing the bearer token to send to a remote app
              server websocket
    
          --strict-config
              Error out when config.toml contains fields that are not recognized by this version of
              Codex
    
      -i, --image <FILE>...
              Optional image(s) to attach to the initial prompt
    
      -m, --model <MODEL>
              Model the agent should use
    
          --oss
              Use open-source provider
    
          --local-provider <OSS_PROVIDER>
              Specify which local provider to use (lmstudio or ollama). If not specified with --oss,
              will use config default or show selection
    
      -p, --profile <CONFIG_PROFILE_V2>
              Layer $CODEX_HOME/<name>.config.toml on top of the base user config
    
      -s, --sandbox <SANDBOX_MODE>
              Select the sandbox policy to use when executing model-generated shell commands
    
              [possible values: read-only, workspace-write, danger-full-access]
    
          --approve-for-me
              Route approval requests through automatic review using the workspace-write sandbox
    
          --dangerously-bypass-approvals-and-sandbox
              Skip all confirmation prompts and execute commands without sandboxing. EXTREMELY
              DANGEROUS. Intended solely for running in environments that are externally sandboxed
    
          --dangerously-bypass-hook-trust
              Run enabled hooks without requiring persisted hook trust for this invocation. DANGEROUS.
              Intended only for automation that already vets hook sources
    
      -C, --cd <DIR>
              Tell the agent to use the specified directory as its working root
    
          --worktree
              Run the session in a new managed Git worktree
    
          --add-dir <DIR>
              Additional directories that should be writable alongside the primary workspace
    
      -a, --ask-for-approval <APPROVAL_POLICY>
              Configure when the model requires human approval before executing a command
    
              Possible values:
              - on-request: The model decides when to ask the user for approval
              - never:      Never ask for user approval Execution failures are immediately returned to
                the model
    
          --search
              Enable live web search. When enabled, the native Responses `web_search` tool is available
              to the model (no per‑call approval)
    
          --no-alt-screen
              Disable alternate screen mode
    
              Runs the TUI in inline mode, preserving terminal scrollback history.
    
          --no-daemon
              Run without the shared background server, even if it is already running
    
      -h, --help
              Print help (see a summary with '-h')
    
      -V, --version
              Print version
    ✓ • 0ms

## Activity

    $ rg --files --hidden /home/jorobg/.codex -g '*manual*' -g '*session*' -g '*migration*' -g '*rollout*' -g '!sessions/**' -g '!archived_sessions/**' -g '!plugins/**' -g '!skills/**' -g '!node_modules/**' -g '!logs/**' -g '!log/**' -g '!shell_snapshots/**' -g '!auth*' -g '!history*' -g '!cache/**'
    /home/jorobg/.codex/.sandbox_migration
    /home/jorobg/.codex/sessions/2026/10/22/rollout-2026-10-22T01-42-49-01a160a1-a87a-7912-ab4e-00f4c79ebc7f.jsonl
    /home/jorobg/.codex/sessions/2026/10/23/rollout-2026-10-23T11-50-20-01a167f4-3725-7642-8fc5-2d19c63cd2fd.jsonl
    /home/jorobg/.codex/plugins/cache/openai-curated-remote/work-pets/0.1.6/references/upload-session.md
    /home/jorobg/.codex/session_index.jsonl
    /home/jorobg/.codex/skills/.system/openai-docs/references/model-migration.md
    /home/jorobg/.codex/skills/.system/openai-docs/scripts/fetch-codex-manual.mjs
    /home/jorobg/.codex/plugins/cache/openai-curated-remote/sites/1.1.1/skills/sites/templates/vinext-starter/scripts/connector-preview/connector-preview-session.mjs
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/zoom-apps-sdk/troubleshooting/migration.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/cobrowse-sdk/references/session-events.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/cobrowse-sdk/concepts/session-lifecycle.md
    /home/jorobg/.codex/.tmp/plugins/plugins/temporal/skills/temporal-ops/references/ops/cloud-migration.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/cobrowse-sdk/examples/session-events.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/video-sdk/references/session-lifecycle.md
    /home/jorobg/.codex/.tmp/plugins/plugins/nvidia/skills/omniverse-realtime-viewer/references/cpp-native-viewer/renderer-session.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/video-sdk/windows/examples/session-join-pattern.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/video-sdk/android/examples/session-join-pattern.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/video-sdk/flutter/examples/session-join-pattern.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/video-sdk/macos/examples/session-join-pattern.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/video-sdk/linux/examples/session-join-pattern.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/phone/references/deprecations-and-migrations.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/video-sdk/unity/examples/session-join-pattern.md
    /home/jorobg/.codex/.tmp/plugins/plugins/notion/skills/notion-spec-to-implementation/examples/database-migration.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/video-sdk/web/examples/session-join-pattern.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/rtms/examples/manual-websocket.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/video-sdk/ios/examples/session-join-pattern.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/general/use-cases/flutter-video-sessions.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/general/use-cases/react-native-video-sessions.md
    /home/jorobg/.codex/.tmp/plugins/plugins/zoom/skills/video-sdk/react-native/examples/session-join-pattern.md
    ✓ • 0ms

## Activity

    Searched the web for Codex transfer session another computer, Codex sessions migrate rollouts paginated history

## Activity

    Opened https://learn.chatgpt.com/docs/remote-connections.md

## Activity

    Opened https://learn.chatgpt.com/docs/remote-connections

## Activity

    $ python - <<'PY'
        import json, os
        from pathlib import Path
        print('Current thread:', os.environ.get('CODEX_THREAD_ID', '(not exposed)'))
        root = Path('/home/jorobg/.codex/sessions')
        for path in sorted(root.rglob('*.jsonl')):
            with path.open() as f:
                first = json.loads(f.readline())
            meta = first.get('payload', {})
            print(json.dumps({'file': str(path), 'bytes': path.stat().st_size, 'record_type': first.get('type'), 'id': meta.get('id'), 'cwd': meta.get('cwd'), 'cli_version': meta.get('cli_version'), 'history_mode': meta.get('history_mode')}, ensure_ascii=False))
        PY
    Current thread: 01a160a1-a87a-7912-ab4e-00f4c79ebc7f
    {"file": "/home/jorobg/.codex/sessions/2026/10/22/rollout-2026-10-22T01-42-49-01a160a1-a87a-7912-ab4e-00f4c79ebc7f.jsonl", "bytes": 2205830, "record_type": "session_meta", "id": "01a160a1-a87a-7912-ab4e-00f4c79ebc7f", "cwd": "/home/jorobg/Coding/nophyx", "cli_version": "0.163.0", "history_mode": "paginated"}
    {"file": "/home/jorobg/.codex/sessions/2026/10/23/rollout-2026-10-23T11-50-20-01a167f4-3725-7642-8fc5-2d19c63cd2fd.jsonl", "bytes": 94048, "record_type": "session_meta", "id": "01a167f4-3725-7642-8fc5-2d19c63cd2fd", "cwd": "/home/jorobg/Coding/nophyx", "cli_version": "0.163.0", "history_mode": "paginated"}
    ✓ • 0ms

## Assistant

Yes. The official OpenAI documentation describes a portable Markdown export and a desktop-app feature that transfers the chat itself. Your installed CLI uses the newer paginated history format, so I’m checking before recommending that you copy individual session files.

## Activity

    Searched for 'thread/import' in https://learn.chatgpt.com/docs/app-server

## Assistant

Yes. **The simplest way to carry our discussion to another PC is to export it:**

1. In this Codex terminal, after this response finishes, enter:
   ```text
   /export nophyx-session.md
   ```
2. Copy that file and your `nophyx` project folder to the other PC.
3. Install Codex, sign in, and start it in the project folder.
4. Give it this instruction:
   > Read nophyx-session.md and continue planning Nophyx. Preserve the decisions we already made, identify the remaining questions, and stay in Plan Mode.

This starts a **new chat with our saved conversation as context**. The export command is documented in the [official OpenAI documentation](https://learn.chatgpt.com/docs/developer-commands#export-a-conversation-with-export).

To transfer **the actual saved chat**, the desktop app also supports **Hand off** between connected computers with matching repository projects. That requires more setup than exporting. [Chat handoff instructions](https://learn.chatgpt.com/docs/remote-connections#hand-off-a-chat-between-hosts)
