
---
author: karpathy on Twitter
title: Tweets From Andrej Karpathy
category: #tweets
url: https://twitter.com/karpathy

---

![rw-book-cover](https://pbs.twimg.com/profile_images/1296667294148382721/9Pr6XrPB.jpg)

## Metadata
- Author: [[@karpathy on Twitter]]
- Full Title: Tweets From Andrej Karpathy
- Category: #tweets
- URL: https://twitter.com/karpathy

## Highlights
- New (2h13m 😅) lecture: "Let's build the GPT Tokenizer"
  Tokenizers are a completely separate stage of the LLM pipeline: they have their own training set, training algorithm (Byte Pair Encoding), and after training implement two functions: encode() from strings to tokens, and decode() back from tokens to strings. In this lecture we build from scratch the Tokenizer used in the GPT series from OpenAI.<img src='https://pbs.twimg.com/media/GGzDVPMasAAtLs0.jpg'/> ([View Tweet](https://twitter.com/karpathy/status/1759996549109776702))
## New highlights added September 19, 2024 at 12:55 PM
- I'm playing around with generative AI tools and stitching them together into visual stories. Here I took the first few sentences of Pride and Prejudice and made it into a video.
  The gen stack used for this one:
  - [AnthropicAI](https://twitter.com/AnthropicAI) Claude took the first chapter, generated the scenes and the individual prompts to to the image generator.
  - [ideogram_ai](https://twitter.com/ideogram_ai) took the prompts and generate the images
  - [LumaLabsAI](https://twitter.com/LumaLabsAI) took the images and animated them
  - [elevenlabsio](https://twitter.com/elevenlabsio) for narration
  - [veedstudio](https://twitter.com/veedstudio) to stitch it together
  (Many of these choices are just what I happened to use for this one while exploring a bunch of things). Anyway honestly it was pretty messy and there is a ton of copy pasting between all of the tools, and even this little video with 3 scenes took me about an hour.
  There is a huge storytelling opportunity here for whoever can make this convenient. Who is building the first 100% AI-native movie maker?<video controls><source src="https://video.twimg.com/ext_tw_video/1808686080688074753/pu/pl/h-q0eO0qlKUcRunS.m3u8?tag=12" type="application/x-mpegURL"><source src="https://video.twimg.com/ext_tw_video/1808686080688074753/pu/vid/avc1/480x270/Qgq4lMeuK6DcC0Me.mp4?tag=12" type="video/mp4"><source src="https://video.twimg.com/ext_tw_video/1808686080688074753/pu/vid/avc1/640x360/CWAhG3T5NcydEbiR.mp4?tag=12" type="video/mp4"><source src="https://video.twimg.com/ext_tw_video/1808686080688074753/pu/vid/avc1/1280x720/alTMuEv5czUPgXFa.mp4?tag=12" type="video/mp4">Your browser does not support the video tag.</video> ([View Tweet](https://twitter.com/karpathy/status/1808686307331428852))
- # RLHF is just barely RL
  Reinforcement Learning from Human Feedback (RLHF) is the third (and last) major stage of training an LLM, after pretraining and supervised finetuning (SFT). My rant on RLHF is that it is just barely RL, in a way that I think is not too widely appreciated. RL is powerful. RLHF is not. Let's take a look at the example of AlphaGo. AlphaGo was trained with actual RL. The computer played games of Go and trained on rollouts that maximized the reward function (winning the game), eventually surpassing the best human players at Go. AlphaGo was not trained with RLHF. If it were, it would not have worked nearly as well. 
  What would it look like to train AlphaGo with RLHF? Well first, you'd give human labelers two board states from Go, and ask them which one they like better:
  Then you'd collect say 100,000 comparisons like this, and you'd train a "Reward Model" (RM) neural network to imitate this human "vibe check" of the board state. You'd train it to agree with the human judgement on average. Once we have a Reward Model vibe check, you run RL with respect to it, learning to play the moves that lead to good vibes. Clearly, this would not have led anywhere too interesting in Go. There are two fundamental, separate reasons for this:
  1. The vibes could be misleading - this is not the actual reward (winning the game). This is a crappy proxy objective. But much worse,
  2. You'd find that your RL optimization goes off rails as it quickly discovers board states that are adversarial examples to the Reward Model. Remember the RM is a massive neural net with billions of parameters imitating the vibe. There are board states are "out of distribution" to its training data, which are not actually good states, yet by chance they get a very high reward from the RM.
  For the exact same reasons, sometimes I'm a bit surprised RLHF works for LLMs at all. The RM we train for LLMs is just a vibe check in the exact same way. It gives high scores to the kinds of assistant responses that human raters statistically seem to like. It's not the "actual" objective of correctly solving problems, it's a proxy objective of what looks good to humans. Second, you can't even run RLHF for too long because your model quickly learns to respond in ways that game the reward model. These predictions can look really weird, e.g. you'll see that your LLM Assistant starts to respond with something non-sensical like "The the the the the the" to many prompts. Which looks ridiculous to you but then you look at the RM vibe check and see that for some reason the RM thinks these look excellent. Your LLM found an adversarial example. It's out of domain w.r.t. the RM's training data, in an undefined territory. Yes you can mitigate this by repeatedly adding these specific examples into the training set, but you'll find other adversarial examples next time around. For this reason, you can't even run RLHF for too many steps of optimization. You do a few hundred/thousand steps and then you have to call it because your optimization will start to game the RM. This is not RL like AlphaGo was.
  And yet, RLHF is a net helpful step of building an LLM Assistant. I think there's a few subtle reasons but my favorite one to point to is that through it, the LLM Assistant benefits from the generator-discriminator gap. That is, for many problem types, it is a significantly easier task for a human labeler to select the best of few candidate answers, instead of writing the ideal answer from scratch. A good example is a prompt like "Generate a poem about paperclips" or something like that. An average human labeler will struggle to write a good poem from scratch as an SFT example, but they could select a good looking poem given a few candidates. So RLHF is a kind of way to benefit from this gap of "easiness" of human supervision. There's a few other reasons, e.g. RLHF is also helpful in mitigating hallucinations because if the RM is a strong enough model to catch the LLM making stuff up during training, it can learn to penalize this with a low reward, teaching the model an aversion to risking factual knowledge when it's not sure. But a satisfying treatment of hallucinations and their mitigations is a whole different post so I digress. All to say that RLHF *is* net useful, but it's not RL.
  No production-grade *actual* RL on an LLM has so far been convincingly achieved and demonstrated in an open domain, at scale. And intuitively, this is because getting actual rewards (i.e. the equivalent of win the game) is really difficult in the open-ended problem solving tasks. It's all fun and games in a closed, game-like environment like Go where the dynamics are constrained and the reward function is cheap to evaluate and impossible to game. But how do you give an objective reward for summarizing an article? Or answering a slightly ambiguous question about some pip install issue? Or telling a joke? Or re-writing some Java code to Python? Going towards this is not in principle impossible but it's also not trivial and it requires some creative thinking. But whoever convincingly cracks this problem will be able to run actual RL. The kind of RL that led to AlphaGo beating humans in Go. Except this LLM would have a real shot of beating humans in open-domain problem solving.
  ![](https://pbs.twimg.com/media/GUZ4siVa8AA74Rp.jpg) ([View Tweet](https://twitter.com/karpathy/status/1821277264996352246))