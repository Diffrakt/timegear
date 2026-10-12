[Skip to main content](#main-content)
Y'all this is a sexy paper; context language models : r/LocalLLaMA

Open menuOpen navigation
Go to Reddit Home

[Sign Up](https://www.reddit.com/register/)Sign up for Reddit
  [Log In](https://www.reddit.com/login/)Log in to Reddit

Expand user menuOpen settings menu




[![](https://styles.redditmedia.com/t5_81eyvm/styles/communityIcon_cumnsvx9kzma1.png?width=96&height=96&frame=1&auto=webp&s=a5f00ebcb6dcd381b6e081d0764399f7e70108f0)

Go to LocalLLaMA](https://www.reddit.com/r/LocalLLaMA/)

[r/LocalLLaMA](https://www.reddit.com/r/LocalLLaMA/)
6d ago

[u/Combinatorilliance](https://www.reddit.com/user/Combinatorilliance/)

Y'all this is a sexy paper; context language models
===================================================

[News](/r/LocalLLaMA/?f=flair_name%3A%22News%22)

[![](https://external-preview.redd.it/yall-this-is-a-sexy-paper-context-language-models-v0-q3evP6JeDpAC2MdSQHWYxnCYTqbJkElIQsLFqVSdkss.png?width=640&crop=smart&auto=webp&s=283ab0678bac942406ba4b6781cef4107386a163)](https://arxiv.org/abs/2609.37725)[![r/LocalLLaMA - Y'all this is a sexy paper; context language models](https://external-preview.redd.it/yall-this-is-a-sexy-paper-context-language-models-v0-q3evP6JeDpAC2MdSQHWYxnCYTqbJkElIQsLFqVSdkss.png?width=640&crop=smart&auto=webp&s=283ab0678bac942406ba4b6781cef4107386a163)](https://arxiv.org/abs/2609.37725)

arxiv.org

Abstract page for arXiv paper 2609.37725: Context Language Models

[Open](https://arxiv.org/abs/2609.37725)

[Paper linky - Context Language Models](https://arxiv.org/abs/2609.37725)

The central idea of the paper is incredibly simple. Give a model the ability to edit its context on-the-go like a file has major benefits on task performance, context management (memory) and even computational efficiency (both wall clock and total flops). Their paper shows mostly benefits and relatively small downsides.

You can [try it out](https://github.com/lolipopshock/pi-clm) as a plugin for pi!

In short, pros and cons
-----------------------

Pros:

1. Improves outcomes on long running tasks

   * Coding and deep research tasks
   * Open discovery problems (long horizon research tasks, /goal loops etc)
2. Inference can become more compute-efficient and wall-clock efficient

   * Note, this depends on a caching optimization in the inference engine
3. Much less context bloat, meaning it's more (V)RAM efficient
4. No more slow and unreliable compacts

Cons:

1. The cache optimization only exists for SGLang
2. Prompt injections (including hallucinated instructions) are much less likely to be forgotten, increasing risks
3. Requires harness customizations (authors supply a pi plugin)

Some more context
-----------------

The approach works by modifying the harness to allow access to the context as a file. A model is allowed to edit the context as it would any other file.

They've tested the approach on models as small as qwen3.6 9b, as well as on qwen3.8 27b and claude sonnet 4.6.

Out-of-the-box, meaning just a small addition to the system prompt and tools to edit the context as a file, task performance, context management and efficiency measures remain approximately the same or improve by a little bit. The smaller qwen3.6 9b model in particular lost a little bit of efficiency, suggesting it works better on larger (smarter) models.

Performance can be massively improved with RL training, which the authors also did.

Wanna try it out?
-----------------

You can try it out right now if you use pi

1. Install the plugin <https://github.com/lolipopshock/pi-clm>, this comes from the authors directly
2. After installation, adjust settings with `/clm settings`:

   * Set steering to `house-brief.md` (modifies the system prompt, I suppose this should be left disabled for RL'd models only, of which there are none right now)
   * Enable "One tool per turn"; this one is important for performance
   * Enable "Size trailer"; this one appends context usage after every tool result. Without it, models are much less inclined to modify context on-the-go for large tool calls

Fin
---

Let me know how it goes!

Last, I also consulted this video by "Prompt Engineering" on YouTube in addition to the paper: <https://www.youtube.com/watch?v=Bgtr1Ue40Jo>

Share

[![u/OReilly_Learning avatar](https://styles.redditmedia.com/t5_3yzd7l/styles/profileIcon_xi8a7bsatlwg1.png?width=48&height=48&frame=1&auto=webp&crop=48%3A48%2Csmart&s=037ec3ee091cbe9a758fada41a0358fbdf0fb1aa)
OReilly\_Learning](/user/OReilly_Learning/)•
[Ad](/user/OReilly_Learning/)

O’Reilly for Business gives your VPs and engineers the verified expertise they need to make tech decisions that actually scale. Trusted by the Fortune 500. No hallucinations, just high-fidelity learning.

Contact Us

oreilly.com

![Thumbnail image: O’Reilly for Business gives your VPs and engineers the verified expertise they need to make tech decisions that actually scale. Trusted by the Fortune 500. No hallucinations, just high-fidelity learning.](https://preview.redd.it/mwhexbyzscwg1.png?width=320&height=180&auto=webp&s=f4923cd48ed65b6a129bf090b6b9bb2ef1346bbd)

Best

Open comment sort options

- Best
- Top
- New
- Controversial
- Old
- Q&A

Comments Section
================

[b183729](https://www.reddit.com/user/b183729/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe2d74g/)

I will read the paper, but before I dive in, wouldn't this force the model to re-infer the whole context from the point of the change onwards?

[Combinatorilliance](https://www.reddit.com/user/Combinatorilliance/)


[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe2eofv/)

It does, but not completely, first of all they introduced a caching optimization for the inference engine that addresses this (I'm not sure about the specifics).

Second, if you use compaction at all, the slowdown would happen anyway (although only once, not n times for each edit early in the context). The paper shows that on-the-go compaction improves task performance on certain tasks too, I assume this is only true for longer tasks and that this works by reducing noise in the context that is irrelevant to the task at hand.

Last, the paper also says you can steer the model to prefer editing only the last message which apparently steers the model quite reliably not to do this unless it's absolutely necessary.

[![u/wishstudio avatar](https://www.redditstatic.com/avatars/defaults/v2/avatar_default_3.png)](https://www.reddit.com/user/wishstudio/)

[wishstudio](https://www.reddit.com/user/wishstudio/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe2t81c/)

Its “optimization” is basically only recomputing changed part and splicing the resulting partial kv cache to the original one. So anything after the changed part is kind of using stale values. The paper claims this won’t affect accuracy but who knows. Even this crude version involves re-rotating keys because the lengths of changed part may be different. I suspect it will be much more difficult to implement for complex attention mechanisms like deepseek flash’s CSA and HCA.

2 more replies

[2 more replies](/r/LocalLLaMA/comments/1wyf63m/comment/pe2t81c/?force-legacy-sct=1)

[More replies](/r/LocalLLaMA/comments/1wyf63m/comment/pe2eofv/?force-legacy-sct=1)

[![u/kaisurniwurer avatar](https://www.redditstatic.com/avatars/defaults/v2/avatar_default_0.png)](https://www.reddit.com/user/kaisurniwurer/)

[kaisurniwurer](https://www.reddit.com/user/kaisurniwurer/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe6hkxw/)

Even a change in the topmost ~2k tokens (depending on the SWA step) will force reprocessing of the full context.

This is why those ugly "preserve thinking" flags came to be.

1 more reply

[1 more reply](/r/LocalLLaMA/comments/1wyf63m/comment/pe2d74g/?force-legacy-sct=1)

[![u/FullOf_Bad_Ideas avatar](https://www.redditstatic.com/avatars/defaults/v2/avatar_default_1.png)](https://www.reddit.com/user/FullOf_Bad_Ideas/)

[FullOf\_Bad\_Ideas](https://www.reddit.com/user/FullOf_Bad_Ideas/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe3vdlu/)

![Profile Badge for the Achievement Top 1% Commenter](https://i.redd.it/qwkcwa2zi3yd1.png)Top 1% Commenter

What is this "Summary" that is trailing close by in terms of the performance lol?

> Method Acc. (%) ↑ PFLOPs / Q ↓ Summary 34.7 → 42.1 4.01 → 2.19 CLM 28.8 → 42.5 1.52 → 1.34

it's ridiculous if summary works better by default and works basically as good after training.

Tested only on a few small Qwen models, might as well fall apart on bigger ones.

Meta Superintelligence lab has been putting out a lot of shitty papers recently, I like FAIR papers much better!

[Combinatorilliance](https://www.reddit.com/user/Combinatorilliance/)


[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe686bh/)

Edited 6d ago

You mean table 2? I'm not sure I agree it performs worse than summary? Pflops lower is better, and accuracy higher is better.

Out of the box summary has better accuracy than the clm method, that makes a lot of sense to me, this is for a 9b model, I can see it botch edits prior to RL. They note that larger models perform better out of the box.

After RL on the 9b model, summary and CLM perform about the same on accuracy, which supports my "botches edits" hypothesis.

However, both out of the box and after RL, CLM is significantly more efficient than the summary strategy, which is a very meaningful finding unless I'm missing something important you saw?

They also explicitly note that smaller models are worse at being a CLM out of the box, so yes you do need to be wary with anything smaller than like qwen 27b-range models:

> Before training, CLM with Qwen3.5-9B underperforms the summary harness by six points due to the smaller model’s limited context-management capabilities

In other comparisons, their method straight up outperforms summary on accuracy and efficiency, see figure 5, figure 6 for instance.

[![Comment Image](https://preview.redd.it/yall-this-is-a-sexy-paper-context-language-models-v0-rcujjaunnsth1.jpeg?width=2340&format=pjpg&auto=webp&s=c1995bdb1ac121acec38701ff4491f0fdd3c7b82)](https://preview.redd.it/yall-this-is-a-sexy-paper-context-language-models-v0-rcujjaunnsth1.jpeg?width=2340&format=pjpg&auto=webp&s=c1995bdb1ac121acec38701ff4491f0fdd3c7b82)

[![u/FullOf_Bad_Ideas avatar](https://www.redditstatic.com/avatars/defaults/v2/avatar_default_1.png)](https://www.reddit.com/user/FullOf_Bad_Ideas/)

[FullOf\_Bad\_Ideas](https://www.reddit.com/user/FullOf_Bad_Ideas/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe69bce/)

![Profile Badge for the Achievement Top 1% Commenter](https://i.redd.it/qwkcwa2zi3yd1.png)Top 1% Commenter

> However, both out of the box and after RL, CLM is significantly more efficient than the summary strategy, which is a very meaningful finding unless I'm missing something important you saw?

Unless I'm mistaken, they use SGLang suffix caching to get the high efficiency numbers. It's a dirty method of changing KV cache.

Numbers in papers are usually benchmaxxed to look good and most of the time fall apart when rigorous study checks for their accuracy. So if something shows that it's barely beating some other method, in a larger sample size even that gain is probably smaller. Meta superintelligence lab was doing it already in the past.

3 more replies

[3 more replies](/r/LocalLLaMA/comments/1wyf63m/comment/pe69bce/?force-legacy-sct=1)

[More replies](/r/LocalLLaMA/comments/1wyf63m/comment/pe686bh/?force-legacy-sct=1)

[More replies](/r/LocalLLaMA/comments/1wyf63m/comment/pe3vdlu/?force-legacy-sct=1)

[![u/BringTea_666 avatar](https://www.redditstatic.com/avatars/defaults/v2/avatar_default_7.png)](https://www.reddit.com/user/BringTea_666/)

[BringTea\_666](https://www.reddit.com/user/BringTea_666/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe356ff/)

![Profile Badge for the Achievement Top 1% Commenter](https://i.redd.it/qwkcwa2zi3yd1.png)Top 1% Commenter

So it's basically silly tavern but in paper form ? Context shennegans are probably the oldest form of context management as early models had like 512-1000-4000 context size.

[![u/antwon_dev avatar](https://www.redditstatic.com/avatars/defaults/v2/avatar_default_2.png)](https://www.reddit.com/user/antwon_dev/)

[antwon\_dev](https://www.reddit.com/user/antwon_dev/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe4kdep/)

I can’t believe I was dealing with 4K context on GPT 3.5 in late 2022 lmfao we have come so far in 4 short years

[![u/wayneworkman avatar](https://www.redditstatic.com/avatars/defaults/v2/avatar_default_5.png)](https://www.reddit.com/user/wayneworkman/)

[wayneworkman](https://www.reddit.com/user/wayneworkman/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe5d017/)

my from scratch model has 32k. yeah we've come a long way.

[More replies](/r/LocalLLaMA/comments/1wyf63m/comment/pe4kdep/?force-legacy-sct=1)

[More replies](/r/LocalLLaMA/comments/1wyf63m/comment/pe356ff/?force-legacy-sct=1)

[![u/TelnyxLLC avatar](https://styles.redditmedia.com/t5_6dyd4t/styles/profileIcon_vaschbkw0bkf1.png?width=48&height=48&frame=1&auto=webp&crop=48%3A48%2Csmart&s=8035e1ce101a774f0e1215f44bff9cc1d987c80f)
u/TelnyxLLC](/user/TelnyxLLC/)•
[Ad](/user/TelnyxLLC/)

Every tenth token is on us. Open models on Telnyx GPUs.

telnyx.com

![Clickable image which will reveal the video player: Every tenth token is on us. Open models on Telnyx GPUs.](https://external-preview.redd.it/bzV2OWM5bHF1aHBoMZAnATjpMF-NqYX2J8MOMFLU8Cdg2Hr9EFTY0wBxqVCO.jpeg?format=pjpg&auto=webp&s=4d2488b42bf72c7794632d55fe12ba82d318fe31)

Collapse video player

[![u/Magos avatar](https://www.redditstatic.com/avatars/defaults/v2/avatar_default_1.png)](https://www.reddit.com/user/Magos/)

[Magos](https://www.reddit.com/user/Magos/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe38xbc/)

It's pretty interesting, but after debating with 6.1 Sol for 15 minutes on this paper, I'm not sure the gains here are better than using a traditional Ralph Loop system with handoff?

If the point of rewrite is during compaction, that is basically similar to a Ralph Loop iteration.

I guess theoretically you can trim context mid-way through an iteration using this method, but I'm pretty sure you have to recalculate the context that way (no easy way for me to test right now)m which defeats the purpose of making it faster due to prompt caching being the standard nowadays

[Combinatorilliance](https://www.reddit.com/user/Combinatorilliance/)


[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe65ckh/)

Well, I can't say. I don't have any measurements on the ralph loop with handoff, this is also why they introduced context bench because this kinda thing wasn't really being benchmarked anyway.

I'm sure the paper's approach isn't going to prove some kind of global optimum, but it does open the discussion further

[More replies](/r/LocalLLaMA/comments/1wyf63m/comment/pe38xbc/?force-legacy-sct=1)

[![u/VoiceApprehensive893 avatar](https://styles.redditmedia.com/t5_5tio0h/styles/profileIcon_3u0fnyok1qkf1.jpeg?width=64&height=64&frame=1&auto=webp&crop=64%3A64%2Csmart&s=d4a0e7f66169bb06c3291d31eb3434bfd44ce916)](https://www.reddit.com/user/VoiceApprehensive893/)

[VoiceApprehensive893](https://www.reddit.com/user/VoiceApprehensive893/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe6f7xb/)

recompute all kv cache 🗣️ 🔥 🔥

[![u/Imaginary-Unit-3267 avatar](https://www.redditstatic.com/avatars/defaults/v2/avatar_default_5.png)](https://www.reddit.com/user/Imaginary-Unit-3267/)

[Imaginary-Unit-3267](https://www.reddit.com/user/Imaginary-Unit-3267/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe51nrc/)

This sounds fascinating, and resembles things I've played with, but what I worry about is whether it would just invalidate the cache over and over. You say they made some kind of "optimizations" in SGLang, but I use llama.cpp, so that doesn't apply here - do you think the quality improvement is worth the massive increase in latency from prompt reprocessing over and over, or is it not worth the bother for me?

[Combinatorilliance](https://www.reddit.com/user/Combinatorilliance/)


[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe664yf/)

I think it's not production ready in that regard. This is a super promising research direction, but the speed tradeoff is very real (tested it yesterday on my machine with llama.cpp) and my qwen3.8 27b got confused on the editing initially because the plugin acted a bit weird.

I'd wait until "a few more papers down the line", once people have started tinkering with it a bit more and some issues have been smoothed out then it sounds more realistic.

Some people in this thread also pointed out that the caching trick isn't ideal either and has some drawbacks that need to be taken into consideration.

[![u/Past-Town-9807 avatar](https://www.redditstatic.com/avatars/defaults/v2/avatar_default_6.png)](https://www.reddit.com/user/Past-Town-9807/)

[Past-Town-9807](https://www.reddit.com/user/Past-Town-9807/)

[2d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pey6x3r/)

I've done all sorts of experiments with context engineering and stopped every single one of them because it was just so punishingly expensive. At my most extreme I was doing I think 2 or 3 messages tops in a session, one of them acting as a state machine summarizing the previous turns and the others for new tool results or new user prompt. I assume others have tried all this stuff too. The problem is that the punishment ($$, or wall time) is so sharp for busting the cache that it's just not worth it. Also it worked better for older models that weren't trained to actually solve everything you tell them to solve, and made it slightly worth the extra costs. Newer models \*are\* so trained, so they don't need to have their context so heavily controlled anymore. At least IME anyway. So now for me cache is king.

[More replies](/r/LocalLLaMA/comments/1wyf63m/comment/pe51nrc/?force-legacy-sct=1)

[![u/__Maximum__ avatar](https://www.redditstatic.com/avatars/defaults/v2/avatar_default_3.png)](https://www.reddit.com/user/__Maximum__/)

[\_\_Maximum\_\_](https://www.reddit.com/user/__Maximum__/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe6ky5s/)

I think prime-agent is already doing it?

[Combinatorilliance](https://www.reddit.com/user/Combinatorilliance/)


[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe6o4q0/)

No clue, never heard of prime agent before! Even so, sounds great to do more research on it!

[More replies](/r/LocalLLaMA/comments/1wyf63m/comment/pe6ky5s/?force-legacy-sct=1)

[stoppableDissolution](https://www.reddit.com/user/stoppableDissolution/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe28tf1/)

![Profile Badge for the Achievement Top 1% Commenter](https://i.redd.it/qwkcwa2zi3yd1.png)Top 1% Commenter

Isnt it basically cache compaction, that ends up driving the models insane after few iterations?

[Combinatorilliance](https://www.reddit.com/user/Combinatorilliance/)


[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe2b39g/)

No, two key differences:

1. The model itself decides what to do and how, cache compaction assumes a specific strategy and a specific output template, in fact, you can even steer the process as you go by telling the model what is and what isn't important according to you
2. Cache compaction relies on the model making many decisions on what is and what is not important in one go, which is known to introduce faults

> that ends up driving the models insane after few iterations

This works better than cache compaction does for tasks that take over 20 hours in total for qwen3.6 26b, so at least on the long horizon benchmarks the models don't go insane at all

[More replies](/r/LocalLLaMA/comments/1wyf63m/comment/pe28tf1/?force-legacy-sct=1)

[![u/EnvironmentalLeg8506 avatar](https://www.redditstatic.com/avatars/defaults/v2/avatar_default_5.png)](https://www.reddit.com/user/EnvironmentalLeg8506/)

[EnvironmentalLeg8506](https://www.reddit.com/user/EnvironmentalLeg8506/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe4s4mc/)

You mentioned in the comments that you can steer it as you go by telling the model what is and isn't important to you. Once the context is a file the model edits, that file is its working memory, and the injection risk you listed comes from the same property that makes it useful: whatever gets written in stays until something takes it out.

So the edit history matters about as much as the edits. If each context edit lands as a diff you can read, and you can revert one without restarting the run, a hallucinated instruction becomes something you can find and drop instead of something that quietly shapes the rest of the task. It also lets the steering go both ways, since you see what it decided to keep and not only what you told it to keep.

That's the first thing I'd look for in the pi plugin before trusting it with anything long.

[Combinatorilliance](https://www.reddit.com/user/Combinatorilliance/)


[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe65sob/)

Holy bot!

[More replies](/r/LocalLLaMA/comments/1wyf63m/comment/pe4s4mc/?force-legacy-sct=1)

[![u/charmander_cha avatar](https://www.redditstatic.com/avatars/defaults/v2/avatar_default_0.png)](https://www.reddit.com/user/charmander_cha/)

[charmander\_cha](https://www.reddit.com/user/charmander_cha/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe24xyb/)

É uma evolução do RLM?

[Combinatorilliance](https://www.reddit.com/user/Combinatorilliance/)


[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe26s9c/)

Yes, this work is based on RLM. I saw that the Github author of the pi-rlm plugin also worked on this project, I assume he's an author on the paper too.

[More replies](/r/LocalLLaMA/comments/1wyf63m/comment/pe24xyb/?force-legacy-sct=1)

[ladz](https://www.reddit.com/user/ladz/)

[6d ago](https://www.reddit.com/r/LocalLLaMA/comments/1wyf63m/comment/pe52g3i/)

I remain confused why inference engines themselves don't implement these sort tools for use during thinking. Why do we have to wait for the next turn?

New to Reddit?

Create your account and connect with a world of communities.

المواصلة باستخدام Googleالمواصلة باستخدام Googleيفتح الرابط في علامة تبويب جديدة

Continue with Email

Continue with Phone Number

By continuing, you agree to our
[User Agreement](https://www.redditinc.com/policies/user-agreement)
and acknowledge that you understand the
[Privacy Policy](https://www.redditinc.com/policies/privacy-policy).

* [What's the hype around Context Language Models](https://www.reddit.com/r/interviewstack/comments/1wvfqrk/whats_the_hype_around_context_language_models/) 

  [![](https://styles.redditmedia.com/t5_fzvtwl/styles/communityIcon_72fhwpn3up3g1.png?width=64&frame=1&auto=webp&s=ca26b1a03c899b2e6a3641c711e658b8d68750d0)

  r/interviewstack](https://www.reddit.com/r/interviewstack/)
  10d ago

  [### What's the hype around Context Language Models](/r/interviewstack/comments/1wvfqrk/whats_the_hype_around_context_language_models/)

  [![r/interviewstack - What's the hype around Context Language Models](https://external-preview.redd.it/NGFpanEwajA3eXNoMcNwgXO6guoZHWg_cMb51g7C-Md2Uvkvr0ShMQXSke9U.png?width=140&height=140&crop=1%3A1%2Csmart&format=jpg&auto=webp&s=02cbdfd9f77c6624f710cf094f46c3ac40b3bcb5)](https://www.reddit.com/r/interviewstack/comments/1wvfqrk/whats_the_hype_around_context_language_models/)

  2 upvotes

  ---
* [can anyone tell me what the heck model was used to make this and what the prompt could've been?](https://www.reddit.com/r/aivideo/comments/1u7vo6r/can_anyone_tell_me_what_the_heck_model_was_used/) 

  [![](https://styles.redditmedia.com/t5_fwb59/styles/communityIcon_zhuecbirl3pc1.png?width=64&frame=1&auto=webp&s=1939d5d13023e1046a05d437a3d91dad4ca52a5d)

  r/aivideo](https://www.reddit.com/r/aivideo/)
  4mo ago

  [### can anyone tell me what the heck model was used to make this and what the prompt could've been?](/r/aivideo/comments/1u7vo6r/can_anyone_tell_me_what_the_heck_model_was_used/)

  [![r/aivideo - can anyone tell me what the heck model was used to make this and what the prompt could've been?](https://external-preview.redd.it/eGtueDh3OXJxcTdoMU6BK8nJrVsPPImTZpt4EcRLsttGQww49DYJHjSI6Sv_.png?width=140&height=140&crop=1%3A1%2Csmart&format=jpg&auto=webp&s=68cf68933bf599eb455c1fa113ee0a75769709cd)](https://www.reddit.com/r/aivideo/comments/1u7vo6r/can_anyone_tell_me_what_the_heck_model_was_used/)

  995 upvotes
  153 comments

  ---
* ["Hot singles in your area"](https://www.reddit.com/r/nosleep/comments/uvoawr/hot_singles_in_your_area/) 

  [![](https://styles.redditmedia.com/t5_2rm4d/styles/communityIcon_q054jahxvm5b1.png?width=64&frame=1&auto=webp&s=55e7a5dbfbe318835288c2b101355bb12373be62)

  r/nosleep](https://www.reddit.com/r/nosleep/)
  4y ago

  [### "Hot singles in your area"](/r/nosleep/comments/uvoawr/hot_singles_in_your_area/)

  913 upvotes
  37 comments

  ---
* [![u/Botim_App avatar](https://styles.redditmedia.com/t5_f207eb/styles/profileIcon_4dhgryz62zff1.png?width=48&height=48&frame=1&auto=webp&crop=48%3A48%2Csmart&s=b04dea6aeddf849d3359b3834bca792186c30849)
   Botim\_App](/user/Botim_App/)•
  Ad

  Start investing with botim

  ![Thumbnail image: Start investing with botim](https://preview.redd.it/emc4qeh1i85h1.jpeg?auto=webp&s=4ad803f4675bace1d9dbb4dc4f91314d8381aa74)

  botim.me

  Learn More

  ---
* [Long-running agents, context management](https://www.reddit.com/r/PiCodingAgent/comments/1ucu75a/longrunning_agents_context_management/) 

  [![](https://styles.redditmedia.com/t5_gr7c83/styles/communityIcon_knv1mdyu2ymg1.png?width=64&frame=1&auto=webp&s=7b2b9924b6e2991c2854e761b143ac89ea4fedaa)

  r/PiCodingAgent](https://www.reddit.com/r/PiCodingAgent/)
  4mo ago

  [### Long-running agents, context management](/r/PiCodingAgent/comments/1ucu75a/longrunning_agents_context_management/)

  13 upvotes
  12 comments

  ---
* [i hate alternative terms for sex](https://www.reddit.com/r/ihatethissmug/comments/1vm5aqz/i_hate_alternative_terms_for_sex/) 

  [![](https://styles.redditmedia.com/t5_gr2jvu/styles/communityIcon_iggg72djmumg1.jpg?width=64&frame=1&auto=webp&s=91a28acefd305f11ca5fd8ce7235f2fd8551d58c)

  r/ihatethissmug](https://www.reddit.com/r/ihatethissmug/)
  2mo ago

  [### i hate alternative terms for sex](/r/ihatethissmug/comments/1vm5aqz/i_hate_alternative_terms_for_sex/)

  [![r/ihatethissmug - i hate alternative terms for sex](https://preview.redd.it/hhp9ppuksvih1.jpeg?width=140&height=91&auto=webp&s=d81f106768160659f34c2e997cfd8cbbc8e5bec9)](https://www.reddit.com/r/ihatethissmug/comments/1vm5aqz/i_hate_alternative_terms_for_sex/)

  218 upvotes
  104 comments

  ---
* [Hot Trash](https://www.reddit.com/r/aivideo/comments/1u26f47/hot_trash/) 

  [![](https://styles.redditmedia.com/t5_fwb59/styles/communityIcon_zhuecbirl3pc1.png?width=64&frame=1&auto=webp&s=1939d5d13023e1046a05d437a3d91dad4ca52a5d)

  r/aivideo](https://www.reddit.com/r/aivideo/)
  4mo ago

  [### Hot Trash](/r/aivideo/comments/1u26f47/hot_trash/)

  [![r/aivideo - Hot Trash](https://external-preview.redd.it/czhnMDJieG1haDZoMYBmR6x67vcHBwaGQQDPWYfjSn1T2FpmIEkXTQBZHs38.png?width=140&height=78&format=jpg&auto=webp&s=8ba255eef68b575a904c814224abf44f7671e0cf)](https://www.reddit.com/r/aivideo/comments/1u26f47/hot_trash/)

  601 upvotes
  82 comments

  ---
* [I hate the whole "I miss sexy concepts in kpop"](https://www.reddit.com/r/kpop_uncensored/comments/1x0eqt5/i_hate_the_whole_i_miss_sexy_concepts_in_kpop/) 

  [![](https://styles.redditmedia.com/t5_95i5nk/styles/communityIcon_virarxkalwic1.png?width=64&frame=1&auto=webp&s=c1ff978d1995fb7af0c53cc4c1d3aa2a0ef3eb50)

  r/kpop\_uncensored](https://www.reddit.com/r/kpop_uncensored/)
  4d ago

  [### I hate the whole "I miss sexy concepts in kpop"](/r/kpop_uncensored/comments/1x0eqt5/i_hate_the_whole_i_miss_sexy_concepts_in_kpop/)

  85 upvotes
  55 comments

  ---
* [Tracing the thoughts of a large language model](https://www.reddit.com/r/neoliberal/comments/1tcwsk3/tracing_the_thoughts_of_a_large_language_model/) 

  [![](https://styles.redditmedia.com/t5_2sfn3/styles/communityIcon_ayxbkqz6ds331.jpg?width=64&frame=1&auto=webp&s=c22d22b8b588a579b43cc18e78fef1c9c2cebb1d)

  r/neoliberal](https://www.reddit.com/r/neoliberal/)
  5mo ago

  [### Tracing the thoughts of a large language model](/r/neoliberal/comments/1tcwsk3/tracing_the_thoughts_of_a_large_language_model/)

  [![r/neoliberal - Tracing the thoughts of a large language model](https://external-preview.redd.it/azjdy4LiLTVyWBYTdVlmVFzCgNMSwRqI4dvnanVR_dc.jpeg?width=140&height=73&auto=webp&s=10b92ddcb3dbe2f75effd67ba536a783ba77f7a9)](https://www.anthropic.com/research/tracing-thoughts-language-model "Link from r/neoliberal - Tracing the thoughts of a large language model")

  101 upvotes
  117 comments

  ---
* [Human edit 🥱🤑✅🔞](https://www.reddit.com/r/Caldruki/comments/1ujxjxu/human_edit/) 

  [![](https://styles.redditmedia.com/t5_esq374/styles/communityIcon_dtqlv7kl2rsg1.png?width=64&frame=1&auto=webp&s=ace5cc8881f8bd2398aaf55fbbe0eca82105aa7c)

  r/Caldruki](https://www.reddit.com/r/Caldruki/)
  3mo ago

  [### Human edit 🥱🤑✅🔞](/r/Caldruki/comments/1ujxjxu/human_edit/)

  [![r/Caldruki - Human edit 🥱🤑✅🔞](https://external-preview.redd.it/MmtjNHBwdjhzZ2FoMRbaeqdDyd5GRA3ijy5fqYNFJNhDnV86cG1GYc3ysIAm.png?width=140&height=140&format=jpg&auto=webp&s=fec50a9be22fea1e6507fb632dd131a055fbba83)](https://www.reddit.com/r/Caldruki/comments/1ujxjxu/human_edit/)

  252 upvotes
  40 comments

  ---
* [This is sex appeal in it's purest form](https://www.reddit.com/r/digitalfoundry/comments/1vtca1s/this_is_sex_appeal_in_its_purest_form/) 

  [![](https://styles.redditmedia.com/t5_38ibx/styles/communityIcon_um1vokdrysd51.jpeg?width=64&frame=1&auto=webp&s=79fb9b350bc07745a4544d73d2e4a9b867bb3be3)

  r/digitalfoundry](https://www.reddit.com/r/digitalfoundry/)
  2mo ago

  [### This is sex appeal in it's purest form](/r/digitalfoundry/comments/1vtca1s/this_is_sex_appeal_in_its_purest_form/)

  [![r/digitalfoundry - This is sex appeal in it's purest form](https://preview.redd.it/x2uk3y0echkh1.png?width=140&height=140&crop=1%3A1%2Csmart&auto=webp&s=933befe188a177e8f8b6158fa46d2c8e484b1b6e)](https://www.reddit.com/r/digitalfoundry/comments/1vtca1s/this_is_sex_appeal_in_its_purest_form/)

  524 upvotes
  43 comments

  ---
* [ASSBENCH](https://www.reddit.com/r/LocalLLM/comments/1wux8j0/assbench/) 

  [![](https://styles.redditmedia.com/t5_84a9er/styles/communityIcon_7wizpqj3o0xa1.png?width=64&frame=1&auto=webp&s=fa154052cc01cffe0e6bd29b39bab163981cb618)

  r/LocalLLM](https://www.reddit.com/r/LocalLLM/)
  10d ago

  [### ASSBENCH](/r/LocalLLM/comments/1wux8j0/assbench/)

  44 upvotes
  36 comments

  ---
* [Yuh is no more sexy](https://www.reddit.com/r/Switzerland/comments/1w2dzll/yuh_is_no_more_sexy/) 

  [![](https://styles.redditmedia.com/t5_2qhp6/styles/communityIcon_klo98f4fisd41.jpg?width=64&frame=1&auto=webp&s=9de6da2742c79c5f4a9d374f3c2fa6f21d1ec089)

  r/Switzerland](https://www.reddit.com/r/Switzerland/)
  1mo ago

  [### Yuh is no more sexy](/r/Switzerland/comments/1w2dzll/yuh_is_no_more_sexy/)

  32 comments

  ---
* [MY FIRST SEXYICE FIGURE!!](https://www.reddit.com/r/sexyice2019/comments/1tj3l1y/my_first_sexyice_figure/) 

  [![](https://styles.redditmedia.com/t5_ew909w/styles/communityIcon_t87acmt6t70h1.png?width=64&frame=1&auto=webp&s=a543c23cd5187d7126653fd3fcc13336ae3f485e)

  r/sexyice2019](https://www.reddit.com/r/sexyice2019/)
  5mo ago

  [### MY FIRST SEXYICE FIGURE!!](/r/sexyice2019/comments/1tj3l1y/my_first_sexyice_figure/)

  [![r/sexyice2019 - MY FIRST SEXYICE FIGURE!!](https://preview.redd.it/kkt2bpvqsd2h1.jpg?width=140&height=140&crop=1%3A1%2Csmart&auto=webp&s=a3210bd9de3761eb5c3c2f85eeadbda3b84d36a7)](https://www.reddit.com/r/sexyice2019/comments/1tj3l1y/my_first_sexyice_figure/)

  95 upvotes
  15 comments

  ---
* [What is sexy ? 😢.](https://www.reddit.com/r/youngpeopleyoutube/comments/1qnsw5a/what_is_sexy/) 

  [![](https://styles.redditmedia.com/t5_nngwo/styles/communityIcon_t9sm8zwrd7631.png?width=64&frame=1&auto=webp&s=4e9129fca4519050d475f631bdf8556f618a2d65)

  r/youngpeopleyoutube](https://www.reddit.com/r/youngpeopleyoutube/)
  9mo ago

  [### What is sexy ? 😢.](/r/youngpeopleyoutube/comments/1qnsw5a/what_is_sexy/)

  [![r/youngpeopleyoutube - What is sexy ? 😢.](https://b.thumbs.redditmedia.com/qbD6l6ATt1ihgQ2aKgjkYw_vWULs5lwwI2B-TYSE8ug.jpg)](https://www.reddit.com/r/youngpeopleyoutube/comments/1qnsw5a/what_is_sexy/)

  37 upvotes
  11 comments

  ---
* [I hate these kinds of lazy ‘sexy’ designs](https://www.reddit.com/r/hatethissmug/comments/1vmh34u/i_hate_these_kinds_of_lazy_sexy_designs/) 

  [![](https://styles.redditmedia.com/t5_d77fuh/styles/communityIcon_5m9xlho737ae1.png?width=64&frame=1&auto=webp&s=e1f478e605b26045f6e508c51980f6fd296da789)

  r/hatethissmug](https://www.reddit.com/r/hatethissmug/)
  2mo ago

  [### I hate these kinds of lazy ‘sexy’ designs](/r/hatethissmug/comments/1vmh34u/i_hate_these_kinds_of_lazy_sexy_designs/)

  [![r/hatethissmug - I hate these kinds of lazy ‘sexy’ designs](https://preview.redd.it/uoooz5u8oyih1.jpeg?width=140&height=87&auto=webp&s=1c71e8bf29c8c9b771f566406029195e408ae1a6)](https://www.reddit.com/r/hatethissmug/comments/1vmh34u/i_hate_these_kinds_of_lazy_sexy_designs/)

  817 upvotes
  308 comments

  ---
* [Referring to large language models as "AI"](https://www.reddit.com/r/PetPeeves/comments/1qvpfla/referring_to_large_language_models_as_ai/) 

  [![](https://styles.redditmedia.com/t5_2qynj/styles/communityIcon_icc29m3rleof1.png?width=64&frame=1&auto=webp&s=8038865972e22a5e5f2e154bc544f7161dacff80)

  r/PetPeeves](https://www.reddit.com/r/PetPeeves/)
  8mo ago

  [### Referring to large language models as "AI"](/r/PetPeeves/comments/1qvpfla/referring_to_large_language_models_as_ai/)

  23 upvotes
  108 comments

  ---
* [sexy text - free AE project inside](https://www.reddit.com/r/AfterEffects/comments/1qgzf1m/sexy_text_free_ae_project_inside/) 

  [![](https://styles.redditmedia.com/t5_2rc1h/styles/communityIcon_rzgburh5iblh1.png?width=64&frame=1&auto=webp&s=36828a71612aaaa0463dacb87b79ff73ff645344)

  r/AfterEffects](https://www.reddit.com/r/AfterEffects/)
  9mo ago

  [### sexy text - free AE project inside](/r/AfterEffects/comments/1qgzf1m/sexy_text_free_ae_project_inside/)

  [![r/AfterEffects - sexy text - free AE project inside](https://external-preview.redd.it/bmQzMXpmbXB2OWVnMZQ_jP3KSbzDoM6ztvMxPDv__6Th1KgoJy3BuQH8lyXl.png?width=140&height=140&format=jpg&auto=webp&s=e0be2a65000058dd84e9d5679fec90dcafe2fadc)](https://www.reddit.com/r/AfterEffects/comments/1qgzf1m/sexy_text_free_ae_project_inside/)

  148 upvotes
  5 comments

  ---
* [So… sexy?](https://www.reddit.com/r/aislop/comments/1opx774/so_sexy/) 

  [![](https://styles.redditmedia.com/t5_blnf7b/styles/communityIcon_zy5w9c5f22dh1.png?width=64&frame=1&auto=webp&s=81086bae7cd933b6c1f5dd19f27fca3e0ea9b2d2)

  r/aislop](https://www.reddit.com/r/aislop/)
  1y ago

  [### So… sexy?](/r/aislop/comments/1opx774/so_sexy/)

  [![r/aislop - So… sexy?](https://b.thumbs.redditmedia.com/v9-0TL3tsENfiRS4CTMLs7WZsomtthldn6FevFsuapQ.jpg)](https://www.reddit.com/r/aislop/comments/1opx774/so_sexy/)

  93 upvotes
  16 comments

  ---
* [What is the best written sex scene?](https://www.reddit.com/r/writingscaling/comments/1tc7llu/what_is_the_best_written_sex_scene/) 

  [![](https://styles.redditmedia.com/t5_cck92t/styles/communityIcon_or60csaav0of1.png?width=64&frame=1&auto=webp&s=047de48cd91c48d0e42520f5fa5bbe45712bf20c)

  r/writingscaling](https://www.reddit.com/r/writingscaling/)
  5mo ago

  [### What is the best written sex scene?](/r/writingscaling/comments/1tc7llu/what_is_the_best_written_sex_scene/)

  [![r/writingscaling - What is the best written sex scene?](https://preview.redd.it/ubn6jqcg2y0h1.jpeg?width=140&height=115&auto=webp&s=051cb405ea925e9cf83bf7617b574c9b445b21f9)](https://www.reddit.com/r/writingscaling/comments/1tc7llu/what_is_the_best_written_sex_scene/)

  586 upvotes
  157 comments

  ---
* [That sexy black t shirt](https://www.reddit.com/r/trailerparkboys/comments/1wzc0tt/that_sexy_black_t_shirt/) 

  [![](https://styles.redditmedia.com/t5_2rbaa/styles/communityIcon_2c4hfold0ki21.png?width=64&frame=1&auto=webp&s=10b078baaa808e2c8ec8d9261ce30658dca39abe)

  r/trailerparkboys](https://www.reddit.com/r/trailerparkboys/)
  5d ago

  [### That sexy black t shirt](/r/trailerparkboys/comments/1wzc0tt/that_sexy_black_t_shirt/)

  [![r/trailerparkboys - That sexy black t shirt](https://preview.redd.it/aae2p0oegwth1.jpeg?width=140&height=128&auto=webp&s=b51b6461724bb028edafd5e167b2bf54d38778b7)](https://www.reddit.com/r/trailerparkboys/comments/1wzc0tt/that_sexy_black_t_shirt/)

  1.2K upvotes
  78 comments

  ---
* [Basic language model in C](https://www.reddit.com/r/C_Programming/comments/1r7wn05/basic_language_model_in_c/) 

  [r/C\_Programming](https://www.reddit.com/r/C_Programming/)
  8mo ago

  [### Basic language model in C](/r/C_Programming/comments/1r7wn05/basic_language_model_in_c/)

  [![r/C_Programming - Basic language model in C](https://external-preview.redd.it/Y3IzajVrYTVoN2tnMZP4_Zg7YIqZNzvbtM-0NW72ki5jdKm1HMEQNOp3yi9R.png?width=140&height=78&format=jpg&auto=webp&s=d4ef77317b695710ebff3ba345c3e0459713ab0e)](https://www.reddit.com/r/C_Programming/comments/1r7wn05/basic_language_model_in_c/)

  349 upvotes
  16 comments

  ---
* [sex scenes are super easy to write and if you can’t make them meaningfully connect to the plot then you are not a good writer](https://www.reddit.com/r/romantasycirclejerk/comments/1u9yit4/sex_scenes_are_super_easy_to_write_and_if_you/) 

  [![](https://styles.redditmedia.com/t5_dfcc86/styles/communityIcon_rf1aaddnnrah1.png?width=64&frame=1&auto=webp&s=c38c8a72e724c39f01df4a5e16c3c744d2b31c9a)

  r/romantasycirclejerk](https://www.reddit.com/r/romantasycirclejerk/)
  4mo ago

  [### sex scenes are super easy to write and if you can’t make them meaningfully connect to the plot then you are not a good writer](/r/romantasycirclejerk/comments/1u9yit4/sex_scenes_are_super_easy_to_write_and_if_you/)

  54 upvotes
  24 comments

  ---
* [I hate the term sexy time](https://www.reddit.com/r/ihatethissmug/comments/1v6kezl/i_hate_the_term_sexy_time/) 

  [![](https://styles.redditmedia.com/t5_gr2jvu/styles/communityIcon_iggg72djmumg1.jpg?width=64&frame=1&auto=webp&s=91a28acefd305f11ca5fd8ce7235f2fd8551d58c)

  r/ihatethissmug](https://www.reddit.com/r/ihatethissmug/)
  3mo ago

  [### I hate the term sexy time](/r/ihatethissmug/comments/1v6kezl/i_hate_the_term_sexy_time/)

  [![r/ihatethissmug - I hate the term sexy time](https://preview.redd.it/2r8qqms21gfh1.jpeg?width=140&height=140&crop=1%3A1%2Csmart&auto=webp&s=5b33db849365c28d7f4f808e3441c3de16ec152b)](https://www.reddit.com/r/ihatethissmug/comments/1v6kezl/i_hate_the_term_sexy_time/)

  1.1K upvotes
  166 comments

  ---
* [Implementing Context Language Models (CLM) in Hermes](https://www.reddit.com/r/ContextEngineering/comments/1wz40fn/implementing_context_language_models_clm_in_hermes/) 

  [![](https://styles.redditmedia.com/t5_esoxdw/styles/communityIcon_w1hwo89wbh9f1.jpeg?width=64&frame=1&auto=webp&s=5e9e109484fd48f070896bbcb67a3122c131031b)

  r/ContextEngineering](https://www.reddit.com/r/ContextEngineering/)
  5d ago

  [### Implementing Context Language Models (CLM) in Hermes](/r/ContextEngineering/comments/1wz40fn/implementing_context_language_models_clm_in_hermes/)

  1 upvote

  ---
* [AkbasCore MAM: The end of the "one big context window" era? A frozen LLM that remembers by plugging in memory cartridges instead of re-reading text (Qwen edition, open-source demo coming tomorrow)](https://www.reddit.com/r/Qwen_AI/comments/1x10yw1/akbascore_mam_the_end_of_the_one_big_context/) 

  [![](https://styles.redditmedia.com/t5_dgnkl3/styles/communityIcon_gz0zdhnxst5h1.png?width=64&frame=1&auto=webp&s=1242ccffbb378abb683127031ac732373f54ba0d)

  r/Qwen\_AI](https://www.reddit.com/r/Qwen_AI/)
  3d ago

  [### AkbasCore MAM: The end of the "one big context window" era? A frozen LLM that remembers by plugging in memory cartridges instead of re-reading text (Qwen edition, open-source demo coming tomorrow)](/r/Qwen_AI/comments/1x10yw1/akbascore_mam_the_end_of_the_one_big_context/)

  [![r/Qwen_AI - AkbasCore MAM: The end of the "one big context window" era? A frozen LLM that remembers by plugging in memory cartridges instead of re-reading text (Qwen edition, open-source demo coming tomorrow)](https://preview.redd.it/jfk7h9kitauh1.jpg?width=140&height=78&auto=webp&s=97502d2560fde58403309561db36acc5b975efe3)](https://www.reddit.com/r/Qwen_AI/comments/1x10yw1/akbascore_mam_the_end_of_the_one_big_context/)

  41 upvotes
  14 comments

  ---

![Llama](https://emoji.redditmedia.com/23w2nhjj1e9f1_t5_81eyvm/Llama)

![Llama](https://emoji.redditmedia.com/23w2nhjj1e9f1_t5_81eyvm/Llama)

<https://x.com/localllamasub>

Public

Anyone can view, post, and comment to this community

0
0

* [Reddit Rules](https://www.redditinc.com/policies/content-policy)
* [Privacy Policy](https://www.reddit.com/policies/privacy-policy)
* [User Agreement](https://www.redditinc.com/policies/user-agreement)
* [Accessibility](https://support.reddithelp.com/hc/sections/38303584022676-Accessibility)
* [Reddit, Inc. © 2026. All rights reserved.](https://redditinc.com)

Expand Navigation

Collapse Navigation

- &nbsp;
- &nbsp;
- &nbsp;
- &nbsp;

---

---

- RESOURCES

- [About Reddit](https://www.redditinc.com)
- [Advertise](https://ads.reddit.com/register?utm_source=web3x_consumer&utm_name=left_nav_cta)
- [Developer Platform](https://developers.reddit.com/?utm_source=reddit&utm_medium=left_nav_resources)
- [Reddit Pro

  BETA](/reddit-pro?utm_source=reddit&utm_medium=left_nav_resources)
- [Help](https://support.reddithelp.com/hc?utm_source=reddit&utm_medium=footer&utm_campaign=evergreen)
- [Blog](https://redditblog.com/)
- [Careers](https://www.redditinc.com/careers)
- [Press](https://www.redditinc.com/press)

---

- [Best of Reddit](https://www.reddit.com/posts/2026/global/)

---

- [Reddit Rules](https://www.redditinc.com/policies/content-policy)
- [Privacy Policy](https://www.reddit.com/policies/privacy-policy)
- [User Agreement](https://www.redditinc.com/policies/user-agreement)
- [Accessibility](https://support.reddithelp.com/hc/sections/38303584022676-Accessibility)

[Reddit, Inc. © 2026. All rights reserved.](https://redditinc.com)

![](https://id.rlcdn.com/472486.gif)
