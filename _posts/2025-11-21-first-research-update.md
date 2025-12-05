---
layout: post
title: "Is your Vibecoding Agent Lying To You?"
# title: "I built a transparency dashboard to catch silent failures in vibecoded systems, and here's what I found."
date: 2025-11-21
category: research
---


Over the last few months, I've been trying to recreate HCI systems from their research papers by vibecoding. 

It has been a challenge, but not in the ways that I expected. 

I thought it would be easy to vibecode these HCI systems, because their research papers are basically very detailed specs, so I always had a clear target to build toward. 

Most of my vibecoded scaffolds also ran without throwing too many syntax errors. (After trying a bunch of models and agents, I found Claude Sonnet 4.5 with the Cline Agent to be the most reliable for me...)

The real challenge was trust. 

I kept wondering whether my vibecoded implementations were *actually* doing what I intended them to do. Every time I vibecoded a system, I'd wonder: 

* *“did it actually query GPT or did it just hardcode responses?”*
* *“Is it using the exact prompts I've specified in the backend or did it hallucinate and rewrite those prompts?”*
* *“Is it actually updating the database or just updating local storage?”*

I didn't want to vibecode something that just *looked* right from the frontend. I wanted to know what was actually being vibecoded under the hood. 

This pushed me to more deliberately start asking:

1. Which backend signals (prompts, workflow stages, file changes, etc.) are actually useful for helping developers detect silent failures in vibecoded systems?

2. If I surface those signals, does it actually change how people debug (diagnosing problems, intervening, and steering the system back toward their intended behavior)?

**most agent code generation is spec-driven**
---
I usually write a detailed natural language instruction, like a spec, when I want an agent to build a system for me. 

Spec-driven vibecoding is common in tools like Lovable, Replit, and Bubble.io where users describe what they want to build in natural language, and then the agent builds it, and the user oversees what the agent generates.

But "oversight" only works if I can actually see what the agent is doing.

Right now, the hardest vibecoded parts for me to trust have been exactly in the places that matter the most for these HCI systems:
* what prompts are being sent,
* how are workflows and stages being wired togehter,
* and how are the underlying data structures and database entries changing over time

Just looking at the raw logs or traces wasn't particularly helpful for me. My goal was to surface the internal signals that help me answer whether the vibecoded system was really doing what I asked, and *where did it drift from my intent?*. 

If you're into this rabbit hole, there's some nice related work:
https://arxiv.org/abs/2306.01941
https://arxiv.org/pdf/2509.10652

So I built a system around that idea.

**rasoi**
---

I built Rasoi, a spec-driven system that reconstructs HCI systems directly from their research papers and exposes the coding agent's backend decisions at each step. 

I use HCI systems papers as a testbed because they're a nice mix of complex, multi-step workflows that are hard for agents to reproduce, and clear textual descriptions of the intended behavior. That combination is perfect for verification (did I build it right?) and validation (did I build the right thing?).

Here's what Rasoi does:

1. Reads an HCI systems paper
2. Extracts a user workflow (the core inputs, outputs, components, and stages of the system) into a simple "spec"
3. Uses that spec to vibecode a running scaffold
4. Logs key internal signals in a transparency dashboard:
* Exact prompts sent to the LLM
* Workflow transitions & API calls
* Changes to core data structures (like task trees)

The idea is that you can monitor both frontend behavior and backend reasoning at the same time, without needing to dig through code.

After each generation cycle, Rasoi verifies the scaffold against the spec with automated checks. You can then compare mismatches against the paper's stated goalls and iterate:

spec → scaffold → agent reflection → agent correction → regeneration

My labmate Billy calls this process "vibe refinement", a term that I will now adopt! The loop pushes the system closer and closer to the behavior described in the paper, while keeping you in control of how it evolves.

**a very brief system walkthrough**
---
Jumpstarter is an HCI system that takes in a user's goal (like *"I want to apply for a PhD this year"*), elicits context from a user (*"what which field of study? which locations?"*), and then takes that context and breaks down the goal into actionable tasks (*"shortlist advisors in Human-AI alignment", "complete a draft of your SOP"*, etc.). Once it identifies an actionable task, it drafts responses for the user, like an SOP draft, or email drafts to send to potential advisors.

<div class="walkthrough-section">

<!-- [REPLACE "SHREY" WITH "I"; briefly describe Jumpstarter. like it breaks down all the steps for something, etc. ]
[CAN REMOVE THE UPLOAD AND EXTRACT PART -- JUST SIMPLIFY IT ; SHOW THE PROMPTS WORKFLOW FIRST AND THEN SHOW THE OCR EXTRACTION IMAGE] -->

<p>To reconstruct Jumpstarter, I first upload the PDF into Rasoi.</p>

<!-- <div class="walkthrough-step">
<h3>1. </h3>

<img src="/img/research-update-1/upload_paper.png" alt="Uploading Jumpstarter PDF to Rasoi" class="walkthrough-image">
</div> -->

<div class="walkthrough-step">
<h3>1. Rasoi reads the PDF and automatically processes it</h3>

<p>by extracting all text content and running OCR on any images containing prompts or system instructions.</p>

<!-- <img src="/img/research-update-1/process_pdf.png" alt="Processing PDF" class="walkthrough-image">
<p class="image-caption">Rasoi extracts text and identifies images with prompts</p> -->

<p>The OCR extraction finds 3 images in the paper's appendix containing prompts used to build the original Jumpstarter system. I can verify whether the extracted text matches the images.</p>

<img src="/img/research-update-1/OCR.png" alt="OCR extraction of prompts" class="walkthrough-image">
<p class="image-caption">OCR results showing extracted prompts from the paper's appendix</p>

<p>After I review each extracted prompt and approve them, Rasoi saves the approved prompts to reference them later.</p>
</div>

<div class="walkthrough-step">
<h3>2. Next, Rasoi extracts the system's core workflow</h3>

<p>into a "simple spec" that captures the complete user workflow, all LLM prompts and their exact text, visual/UI requirements for each stage, and technical requirements like APIs and libraries.</p>

<!-- <img src="/img/research-update-1/extracted_spec_1.png" alt="Generated spec - part 1" class="walkthrough-image">
<p class="image-caption">The extracted specification showing workflow stages and prompts</p> -->

<img src="/img/research-update-1/extracted_spec_2.png" alt="Generated spec - part 2" class="walkthrough-image">
<p class="image-caption">The extracted specification showing workflow stages</p>
</div>

<div class="walkthrough-step">
<h3>3. I read through the generated spec to validate whether it correctly represents how Jumpstarter operates.</h3>

<p>Since it looks accurate, I click "Generate system".</p>
<!-- 
<img src="/img/research-update-1/generate_system.png" alt="Generate system button" class="walkthrough-image">
<p class="image-caption">Ready to generate the system from the validated spec</p> -->
</div>

<!-- <div class="walkthrough-step">
<h3>5. Rasoi generates the Jumpstarter system using a multi-stage deployment approach:</h3>
["MULTI-STAGE DEPLOYMENT APPROACH" IS TOO WORDY AND DOESN'T GET THE MESSAGE ACROSS]
<ul>
<li><strong>Stage 1:</strong> Static UI components with placeholder content</li>
<li><strong>Stage 2:</strong> User interaction handlers and state management</li>
<li><strong>Stage 3:</strong> API routes with mock data</li>
<li><strong>Stage 4:</strong> Real LLM calls replacing mock data</li>
</ul>
[CAN PUT THIS IN A FOOTNOTE BUT I'D JUST COLLAPSE THIS**]
<p>Each stage builds incrementally on the previous one. After all 4 stages complete, Shrey clicks "Launch system".</p>
</div> -->

<div class="walkthrough-step">
<h3>4. Rasoi builds and automatically launches both systems</h3>

<p>Rasoi installs all dependencies, starts the backend server (port 8001), starts the frontend server (port 3001), opens the system in my browser, and <strong>opens the Transparency Dashboard in a separate window (port 3002)</strong>.</p>
<!-- <img src="/img/research-update-1/jumpstarter_landing_page.png" alt="Jumpstarter system" class="walkthrough-image"> -->
<!-- <div class="image-pair">
  <img src="/img/research-update-1/jumpstarter_landing_page.png" alt="Jumpstarter system">
  <img src="/img/research-update-1/transparency_db_unconnected.png" alt="Transparency dashboard">
</div> -->
<!-- <p class="image-caption">The generated Jumpstarter system (above) and Transparency Dashboard open simultaneously</p> -->
<p><strong>The Transparency Dashboard connects to the running Jumpstarter system and monitors all LLM interactions in real time.</strong></p>
</div>

<div class="walkthrough-step">
<h3>5. The generated Jumpstarter system and Transparency Dashboard open simultaneously. I enter my goal into Jumpstarter and click "Start Planning"</h3>
<!-- [PUT THE TASK TREE DATA STRUCTURE FIRST, BEFORE THE BACKEND PROMPT PART] -->
<img src="/img/research-update-1/jumpstarter_input_goal.png" alt="Entering goal in Jumpstarter" class="walkthrough-image">
<p class="image-caption">I enter my goal: "I want to host Thanksgiving dinner this year"</p>

<p>The system generates context questions, but instead of the 3 focused questions from the paper, it outputs a long laundry list of weak questions.</p>

<!-- <img src="/img/research-update-1/jumpstarter_laundry_list_contextQs.png" alt="Laundry list of questions" class="walkthrough-image">
<p class="image-caption">The system generates too many unfocused questions instead of 3 targeted ones</p> -->

<p><strong>I check the Transparency Dashboard.</strong> The dashboard shows exactly which prompt the backend is sending to the LLM. I click on the <code>{task_input}</code> variable to verify if my goal is being passed correctly.</p>

<p><strong>The transparency dashboard reveals the issue: the backend is using a short, hallucinated prompt</strong> instead of the sophisticated few-shot prompt from the paper's appendix.</p>


<img src="/img/research-update-1/transparency_wrong_prompt.png" alt="Transparency dashboard showing workflow" class="walkthrough-image">
<p class="image-caption">The transparency dashboard reveals which prompt is being used and allows inspection of variables</p>

<div class="image-pair">
  <img src="/img/research-update-1/backend_wrong_prompt.png" alt="Transparency dashboard showing the wrong prompt">
  <img src="/img/research-update-1/jumpstarter_laundry_list_contextQs.png" alt="Jumpstarter showing the laundry list of questions">
</div>
<p class="image-caption">Hallucinated prompt shown in generated backend (left) and the questions displayed in frontend (right)</p>

<p><strong>I open the original paper to compare.</strong> The paper's appendix shows a detailed few-shot prompt with examples that should generate exactly 3 focused questions. The backend is using a completely different, much simpler prompt!</p>

<div class="image-pair">
  <img src="/img/research-update-1/paper_appendix_prompt.png" alt="Original paper prompt">
  <img src="/img/research-update-1/transparency_paper_source.png" alt="Dashboard showing paper source">
</div>
<p class="image-caption">The original paper's few-shot prompt (left) vs. what the dashboard shows (right) - revealing the mismatch</p>

<p>By seeing the actual backend prompt in the dashboard, I immediately spot the mismatch and fix it.</p>
</div>
</div>

<div class="walkthrough-step">
<h3>6. I fix the prompt and test again</h3>

<p>The system now generates the proper 3 focused questions. I answer them: "Roughly 5 guests", "1 vegetarian person", "I'm a complete beginner", and clicks "Create My Plan".</p>

<img src="/img/research-update-1/jumpstarter_valid_3_questions.png" alt="Fixed context questions" class="walkthrough-image">
<p class="image-caption">The 3 corrected questions with my answers</p>

<p><strong>But the system generates tasks that completely ignore his answers</strong>—suggesting "Venue Rental and Setup for 100+ guests" and "Professional Catering Coordination" instead of a simple home dinner for 5 people.</p>

<img src="/img/research-update-1/wrong_task_breakdown.png" alt="Wrong task breakdown" class="walkthrough-image">
<p class="image-caption">Generated tasks mention 100+ guests, banquet halls, and commercial kitchens—completely ignoring my "5 guests, beginner" context</p>

<p><strong>I open the Task Tree Browser in the Transparency Dashboard to diagnose the problem.</strong></p>

<img src="/img/research-update-1/task_tree_BUG_singlepanel.png" alt="Task tree showing lost context bug" class="walkthrough-image" style="max-width: 50%;">
<p class="image-caption">The Task Tree Browser reveals the "lost context" bug</p>

<p><strong>The tree visualization makes the bug immediately obvious:</strong></p>

<ul>
<li><strong>Root level:</strong> Context stored correctly ("Roughly 5", "1 vegetarian", "Beginner") ✓</li>
<li><strong>Child tasks:</strong> Completely ignore this context ("100+ guests", "banquet hall", "commercial kitchen") ✗</li>
</ul>

<p>The context is there, but the task generation stage isn't using it!</p>
</div>

<div class="walkthrough-step">
<h3>7. I fix the context flow and regenerates</h3>

<p><strong>I ask the coding agent to fix the context elicitation.</strong> He points out that the task generation stage needs to actually use the stored context from the root node.</p>

<p>After the agent regenerates the task decomposition logic, I test again with the same inputs ("5 guests", "1 vegetarian", "Beginner").</p>

<img src="/img/research-update-1/fixed_task_breakdown.png" alt="Fixed task breakdown" class="walkthrough-image">
<p class="image-caption">The corrected task breakdown now properly reflects my context: small home dinner, beginner-friendly recipes, accommodating 1 vegetarian</p>

<p><strong>The tasks now make sense:</strong> "Plan a Simple Menu", "Prepare a Guest List", "Shop for Ingredients"—all appropriate for a beginner hosting 5 people at home.</p>

<p>By using the Task Tree Browser to visualize where context was getting lost, I could precisely identify and fix the bug, turning an invisible backend problem into a visible, solvable issue.</p>
</div>




**results**
---

I tested the helpfulness of Rasoi's transparency dashboard by using Rasoi to regenerate Jumpstarter (https://arxiv.org/abs/2410.03882), which is a human–AI planning system for task-structured context curation.

<!-- I fed the Jumpstarter paper into Rasoi, let it extract a workflow spec, and had the agent generate a scaffold that I iterated on. -->
 Using the transparency dashboard, I could see not just *what* the regenerated system did, but *how* it did it, so that as I vibecoded, I could validate and verify that it actually matched my intent.

Here's what Rasoi's transparency dashboard helped me with:

**1. catching hallucinated prompts**

In the first iteration of my regenerated Jumpstarter, the frontend looked fine. The UI behaved roughly as expected. But the outputs were wordy, almost irrelevant, and really just didn't match the examples in the paper.

The transparency dashboard helped me discover that the root cause was that the regenerated Jumpstarter wasn't using the original prompts from the paper, even though I had explicitly included them in the spec. whenever the system called GPT, the coding agent had hallucinated new prompts instead of reusing the ones from the spec.

Once I saw that, it was easy to fix. I corrected the prompts to match the original paper, regenerated that part of the system, and immediately saw the quality of the outputs improve.

---

**2. catching missing workflow steps while rebuilding Jumpstarter**

Jumpstarter's core idea is storing user context at each node in the task tree and using it to generate relevant subtasks.

The Task Tree Browser in the transparency dashboard made a critical bug immediately visible: __the context was being stored but not used__.

When I answered "5 guests, 1 vegetarian, beginner cook," the tree showed:

- __Root node__: Context stored correctly ("5 guests", "1 vegetarian", "beginner")
- __Child tasks__: Generated for "100+ guests", "banquet hall", "commercial kitchen"

Without the tree visualization, this would have been nearly impossible to diagnose. The frontend just showed tasks—there was no indication that context existed but wasn't being passed down. The tree made the data flow (or lack thereof) explicit.

Once I saw this, I could point the agent to the exact problem: "The task generation stage needs to actually use the stored context from the root node." After regeneration, the tree showed context properly flowing to child nodes, and the tasks made sense.

This likely would have been painful to debug just by clicking around in the UI... but mostly, I'm not even sure I would have recognized this bug in the first place since it was happening silently in the backend and there were no direct frontend signals in the generated Jumpstarter showing me that context elicitation had not been implemented correctly at all.

---

**3. rediscovering a real bug from the original system**

At one point, I clicked on a task repeatedly and noticed that no context-eliciting questions appeared. My first assumption was that this was yet another bug in the regenerated Jumpstarter.

But when I checked the dashboard, I saw that the system was actually calling the correct prompt from the original Jumpstarter design.

In other words:
* Rasoi had faithfully reproduced the original behavior
* It had also reproduced a small edge-case bug that existed in the original system

Being able to step through the regenerated system's prompts and data structures made it much easier to understand and reproduce that original bug, rather than assuming that my vibecoded implementation was incorrect, and then going down a fruitless rabbit hole trying to figure out why.

---

**4. author validation and "vibe-refinement"**

Finally, I asked one of the original Jumpstarter authors to review the regenerated system.

They confirmed that the regenerated Jumpstarter matched the original system's behavior and workflow. Interestingly, they also pointed out that some of the regenerated UI elements were actually better than what they had originally built!

Their reaction was basically: I'd like to use Rasoi to "vibe-refine" my existing system. Keep the core logic, but borrow some of the regenerated UI and structure to upgrade my original system.

It was cool to think about how Rasoi could be used for more than just validating and verifying the construction of a system and made me wonder, *could this also be a tool for improving our original system?*

**my takeaway**
---

After this Jumpstarter regeneration, I'm convinced that a transparency dashboard is useful for validating and verifying a vibecoded application. Transparency interfaces can help people build trust in vibecoded applications and steer them more deliberately.


In the end, I was able to vibecode an end-to-end HCI system from scratch by:
* Grounding the agent in a paper-derived spec,
* Exposing its backend decisions through a transparency dashboard, and
* Iteratively correcting the system whenever it drifted from the intended workflow.

In the future, I'm interested in building a tool that makes it easier for other developers to create their own transparency dashboards as they vibecode. 

---

*Have questions or thoughts? Feel free to email me!*
