## Abstract 

We explore other things. 

______

## Introduction

<div style="text-align: left;">
<img class="b-lazy" src=data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== data-src="assets/world_model_comic.jpeg" style="display: block; margin: auto; width: 100%;"/>
<figcaption>A World Model, from Scott McCloud's <i>Understanding Comics</i>. <dt-cite key="understandingcomics,understandingcomics_blog"></dt-cite></figcaption>
</div>

Humans develop a mental model of the world based on what they are able to perceive with their limited senses. The decisions and actions we make are based on this internal model. Jay Wright Forrester, the father of system dynamics, described a mental model as:

&ldquo;*The image of the world around us, which we carry in our head, is just a model. Nobody in his head imagines all the world, government or country. He has only selected concepts, and relationships between them, and uses those to represent the real system.*&rdquo; <dt-cite key="forrester"></dt-cite>

To handle the vast amount of information that flows through our daily lives, our brain learns an abstract representation of both spatial and temporal aspects of this information. We are able to observe a scene and remember an abstract description thereof <dt-cite key="facial_identity_primate_brain,single_neuron_viz"></dt-cite>. Evidence also suggests that what we perceive at any given moment is governed by our brain’s prediction of the future based on our internal model <dt-cite key="primary_viz_cortex_past_present,mt_motion"></dt-cite>.

<div style="text-align: left;">
<img class="b-lazy" src=data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== data-src="assets/kitaoka.jpeg" style="display: block; margin: auto; width: 100%;"/>
<figcaption>What we see is based on our brain's prediction of the future. <dt-cite key="kitaoka,pdi,Watanabe2018"></dt-cite></figcaption>
</div>

One way of understanding the predictive model inside of our brains is that it might not be about just predicting the future in general, but predicting future sensory data given our current motor actions <dt-cite key="Keller2012,Leinweber2017"></dt-cite>. We are able to instinctively act on this predictive model and perform fast reflexive behaviours when we face danger <dt-cite key="survival_optimization"></dt-cite>, without the need to consciously plan out a course of action.

Take baseball for example. A baseball batter has milliseconds to decide how they should swing the bat -- shorter than the time it takes for visual signals from our eyes to reach our brain. The reason we are able to hit a 100mph fastball is due to our ability to instinctively predict when and where the ball will go. For professional players, this all happens subconsciously. Their muscles reflexively swing the bat at the right time and location in line with their internal models' predictions <dt-cite key="mt_motion"></dt-cite>. They can quickly act on their predictions of the future without the need to consciously roll out possible future scenarios to form a plan <dt-cite key="mt_motion_article"></dt-cite>.

<div style="text-align: center;">
<img class="b-lazy" src=data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== data-src="assets/mccloud_baseball.jpeg" style="display: block; margin: auto; width: 100%;"/>
<figcaption>We learn to perceive time <i>spatially</i> when we read comics. According to cartoonist and comics theorist Scott McCloud, &ldquo;<i>in the world of comics, time and space are one and the same.</i>&rdquo; Art © Scott McCloud. <dt-cite key="understandingcomics"></dt-cite></figcaption>
</div>

In many reinforcement learning (RL) problems <dt-cite key="Kaelbling:96,sutton_barto,wiering2012"></dt-cite>, an artificial agent also benefits from having a good representation of past and present states, and a good predictive model of the future <dt-cite key="Werbos87specifications,dyna_slides"></dt-cite>, preferably a powerful predictive model implemented on a general purpose computer such as a recurrent neural network (RNN) <dt-cite key="s05_making_the_world_differentiable,s05a_cm,s05b_rl"></dt-cite>. 

Large RNNs are highly expressive models that can learn rich spatial and temporal representations of data. However, many *model-free* RL methods in the literature often only use small neural networks with few parameters. The RL algorithm is often bottlenecked by the *credit assignment problem*<dt-fn>In many RL problems, the feedback (positive or negative reward) is given at end of a sequence of steps. The credit assignment problem tackles the problem of figuring out which steps caused the resulting feedback--which steps should receive credit or blame for the final result?</dt-fn>, which makes it hard for traditional RL algorithms to learn millions of weights of a large model, hence in practice, smaller networks are used as they iterate faster to a good policy during training.

Ideally, we would like to be able to efficiently train large RNN-based agents. The backpropagation algorithm <dt-cite key="Linnainmaa:1970,Kelley:1960,werbos1982sensitivity"></dt-cite> can be used to train large neural networks efficiently. In this work we look at training a large neural network<dt-fn>Typical model-free RL models have in the order of $10^3$ to $10^6$ model parameters. We look at training models in the order of $10^7$ parameters, which is still rather small compared to state-of-the-art deep learning models with $10^8$ to even $10^{9}$ parameters. In principle, the procedure described in this article can take advantage of these larger networks if we wanted to use them.</dt-fn> to tackle RL tasks, by dividing the agent into a large world model and a small controller model. We first train a large neural network to learn a model of the agent's world in an unsupervised manner, and then train the smaller controller model to learn to perform a task using this world model. A small controller lets the training algorithm focus on the credit assignment problem on a small search space, while not sacrificing capacity and expressiveness via the larger world model. By training the agent through the lens of its world model, we show that it can learn a highly compact policy to perform its task.

In this article, we combine several key concepts from a series of papers from 1990--2015 on RNN-based world models and controllers <dt-cite key="s05_making_the_world_differentiable,s05a_cm,s05b_rl,s05c_boredom,learning_to_think"></dt-cite> with more recent tools from probabilistic modelling, and present a simplified approach to test some of those key concepts in modern RL environments <dt-cite key="openai_gym"></dt-cite>. Experiments show that our approach can be used to solve a challenging race car navigation from pixels task that previously has not been solved using more traditional methods.

Most existing *model-based* RL<dt-cite key="rl_survey,s03_overview"></dt-cite> approaches learn a model of the RL environment, but still train on the actual environment. Here, we also explore fully replacing an actual RL environment with a generated one, training our agent's controller only inside of the environment generated by its own internal world model, and transfer this policy back into the actual environment.

To overcome the problem of an agent exploiting imperfections of the generated environments, we adjust a *temperature* parameter of internal world model to control the amount of uncertainty of the generated environments. We train an agent's controller inside of a noisier and more uncertain version of its generated environment, and demonstrate that this approach helps prevent our agent from taking advantage of the imperfections of its internal world model. We will also discuss other related works in the model-based RL literature that share similar ideas of learning a dynamics model and training an agent using this model.

______

## Agent Model

We present a simple model inspired by our own cognitive system. In this model, our agent has a visual sensory component that compresses what it sees into a small representative code. It also has a memory component that makes predictions about future codes based on historical information. Finally, our agent has a decision-making component that decides what actions to take based only on the representations created by its vision and memory components.

<div id = "overview_diagram_div" style="text-align: center;">
<img id="overview_diagram" src="assets/world_model_overview.svg" style="display: block; margin: auto; width: 720px;"/>
</div>

Our agent consists of three components that work closely together: Vision (V), Memory (M), and Controller (C).

### VAE (V) Model

The environment provides our agent with a high dimensional input observation at each time step. This input is usually a 2D image frame that is part of a video sequence. The role of the V model is to learn an abstract, compressed representation of each observed input frame.

<div style="text-align: left;">
<img class="b-lazy" src=data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== data-src="assets/vae.svg" style="display: block; margin: auto; width: 100%;"/>
<br/>
<figcaption>Flow diagram of a Variational Autoencoder. <dt-cite key="vae,vae_dm"></dt-cite></figcaption>
</div>

We use a Variational Autoencoder (VAE) <dt-cite key="vae,vae_dm"></dt-cite> as the V model in our experiments. In the following demo, we show how the V model compresses each frame it receives at time step $t$ into a low dimensional *latent vector* $z_t$. This compressed representation can be used to reconstruct the original image.

<div style="text-align: left;">
<div id="doomvae_sketch" class="unselectable"></div>
<figcaption style="color:#FF6C00;">Interactive Demo</figcaption>
<figcaption>A VAE trained on screenshots obtained from a VizDoom <dt-cite key="vizdoom,takecover"></dt-cite> environment. You can load randomly chosen screenshots to be encoded into a small latent vector <span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>z</mi></mrow><annotation encoding="application/x-tex">z</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.43056em;vertical-align:0em;"></span><span class="base textstyle uncramped"><span class="mord mathit" style="margin-right:0.04398em;">z</span></span></span></span>, which is used to reconstruct the original screenshot. You can also experiment with adjusting the values of the <span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>z</mi></mrow><annotation encoding="application/x-tex">z</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.43056em;vertical-align:0em;"></span><span class="base textstyle uncramped"><span class="mord mathit" style="margin-right:0.04398em;">z</span></span></span></span> vector using the slider bars to see how it affects the reconstruction, or randomize <span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>z</mi></mrow><annotation encoding="application/x-tex">z</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.43056em;vertical-align:0em;"></span><span class="base textstyle uncramped"><span class="mord mathit" style="margin-right:0.04398em;">z</span></span></span></span> to observe the space of possible screenshots learned by our VAE.</figcaption>
</div>

### MDN-RNN (M) Model

While it is the role of the V model to compress what the agent sees at each time frame, we also want to compress what happens over time. For this purpose, the role of the M model is to predict the future. The M model serves as a predictive model of the future $z$ vectors that V is expected to produce. Because many complex environments are stochastic in nature, we train our RNN to output a probability density function $p(z)$ instead of a deterministic prediction of $z$.

<div style="text-align: center;">
<img class="b-lazy" src=data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== data-src="assets/mdn_rnn_new.svg" style="display: block; margin: auto; width: 80%;"/>
<figcaption>RNN with a Mixture Density Network output layer. The MDN outputs the parameters of a mixture of Gaussian distribution used to sample a prediction of the next latent vector <span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>z</mi></mrow><annotation encoding="application/x-tex">z</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.43056em;vertical-align:0em;"></span><span class="base textstyle uncramped"><span class="mord mathit" style="margin-right:0.04398em;">z</span></span></span></span>.</figcaption>
</div>

In our approach, we approximate $p(z)$ as a mixture of Gaussian distribution, and train the RNN to output the probability distribution of the next latent vector $z_{t+1}$ given the current and past information made available to it.

More specifically, the RNN will model $P(z_{t+1} \; | \; a_t, z_t, h_t)$, where $a_t$ is the action taken at time $t$ and $h_t$ is the *hidden state* of the RNN at time $t$. During sampling, we can adjust a *temperature* parameter $\tau$ to control model uncertainty, as done in <dt-cite key="sketchrnn"></dt-cite> -- we will find adjusting $\tau$ to be useful for training our controller later on.

<div style="text-align: center;">
<video class="b-lazy" data-src="assets/mp4/sketch_rnn_insect.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 100%;" ></video>
<figcaption>SketchRNN <dt-cite key="sketchrnndemo"></dt-cite> is an example of a MDN-RNN used to predict the next pen strokes of a sketch drawing. We use a similar model to predict the next latent vector <span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>z</mi></mrow><annotation encoding="application/x-tex">z</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.43056em;vertical-align:0em;"></span><span class="base textstyle uncramped"><span class="mord mathit" style="margin-right:0.04398em;">z</span></span></span></span>.</figcaption>
</div>

This approach is known as a Mixture Density Network <dt-cite key="bishop,mdntf"></dt-cite> combined with a RNN (MDN-RNN) <dt-cite key="graves_rnn,mdnrnn_tutorial"></dt-cite>, and has been used successfully in the past for sequence generation problems such as generating handwriting <dt-cite key="graves_rnn,carter2016experiments"></dt-cite> and sketches <dt-cite key="sketchrnn"></dt-cite>.

### Controller (C) Model

The Controller (C) model is responsible for determining the course of actions to take in order to maximize the expected cumulative reward of the agent during a rollout of the environment. <!--To test our hypothesis that the representations inside our world model (V and M) contain most of the information required to solve a problem, we deliberately make C as simple and small as possible.--> In our experiments, we deliberately make C as simple and small as possible, and trained separately from V and M, so that most of our agent's complexity resides in the world model (V and M).

C is a simple single layer linear model that maps $z_t$ and $h_t$ directly to action $a_t$ at each time step:

$a_t = W_c \; [z_t \; h_t]\; + \; b_c$

In this linear model, $W_c$ and $b_c$ are the weight matrix and bias vector that maps the concatenated input vector $[z_t \; h_t]$ to the output action vector $a_t$.<dt-fn>To be clear, the prediction of $z_{t+1}$ is not fed into the controller C directly -- just the hidden state $h_t$ and $z_t$. This is because $h_t$ has all the information needed to generate the parameters of a mixture of Gaussian distribution, if we want to sample $z_{t+1}$ to make a prediction.</dt-fn>

### Putting Everything Together

The following flow diagram illustrates how V, M, and C interacts with the environment:
<!--During a rollout, it is important to note that C does not directly see the actual observations given to the agent, in this case, a 2D grid of pixels. The actual observation is first processed by V at each time step $t$ to produce $z_t$. The inputs into C is this latent vector $z_t$ concatenated with M's hidden state $h_t$ at each time step. C will then output an action vector $a_t$ for motor control. M will then take the current $z_t$ and action $a_t$ as an input to update its own hidden state to produce $h_{t+1}$ to be used at time $t+1$. -->

<div id = "overview_diagram_div" style="text-align: center;">
<img class="b-lazy" src=data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== data-src="assets/world_model_schematic.svg" style="display: block; margin: auto; width: 65%;"/>
<figcaption>Flow diagram of our Agent model. The raw observation is first processed by V at each time step <span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>t</mi></mrow><annotation encoding="application/x-tex">t</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.61508em;"></span><span class="strut bottom" style="height:0.61508em;vertical-align:0em;"></span><span class="base textstyle uncramped"><span class="mord mathit">t</span></span></span></span> to produce <span class="katex"><span class="katex-mathml"><math><semantics><mrow><msub><mi>z</mi><mi>t</mi></msub></mrow><annotation encoding="application/x-tex">z_t</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.58056em;vertical-align:-0.15em;"></span><span class="base textstyle uncramped"><span class="mord"><span class="mord mathit" style="margin-right:0.04398em;">z</span><span class="vlist"><span style="top:0.15em;margin-right:0.05em;margin-left:-0.04398em;"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span><span class="reset-textstyle scriptstyle cramped"><span class="mord mathit">t</span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span>​</span></span></span></span></span></span>. The input into C is this latent vector <span class="katex"><span class="katex-mathml"><math><semantics><mrow><msub><mi>z</mi><mi>t</mi></msub></mrow><annotation encoding="application/x-tex">z_t</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.58056em;vertical-align:-0.15em;"></span><span class="base textstyle uncramped"><span class="mord"><span class="mord mathit" style="margin-right:0.04398em;">z</span><span class="vlist"><span style="top:0.15em;margin-right:0.05em;margin-left:-0.04398em;"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span><span class="reset-textstyle scriptstyle cramped"><span class="mord mathit">t</span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span>​</span></span></span></span></span></span> concatenated with M's hidden state <span class="katex"><span class="katex-mathml"><math><semantics><mrow><msub><mi>h</mi><mi>t</mi></msub></mrow><annotation encoding="application/x-tex">h_t</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.69444em;"></span><span class="strut bottom" style="height:0.84444em;vertical-align:-0.15em;"></span><span class="base textstyle uncramped"><span class="mord"><span class="mord mathit">h</span><span class="vlist"><span style="top:0.15em;margin-right:0.05em;margin-left:0em;"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span><span class="reset-textstyle scriptstyle cramped"><span class="mord mathit">t</span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span>​</span></span></span></span></span></span> at each time step. C will then output an action vector <span class="katex"><span class="katex-mathml"><math><semantics><mrow><msub><mi>a</mi><mi>t</mi></msub></mrow><annotation encoding="application/x-tex">a_t</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.58056em;vertical-align:-0.15em;"></span><span class="base textstyle uncramped"><span class="mord"><span class="mord mathit">a</span><span class="vlist"><span style="top:0.15em;margin-right:0.05em;margin-left:0em;"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span><span class="reset-textstyle scriptstyle cramped"><span class="mord mathit">t</span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span>​</span></span></span></span></span></span> for motor control. M will then take the current <span class="katex"><span class="katex-mathml"><math><semantics><mrow><msub><mi>z</mi><mi>t</mi></msub></mrow><annotation encoding="application/x-tex">z_t</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.58056em;vertical-align:-0.15em;"></span><span class="base textstyle uncramped"><span class="mord"><span class="mord mathit" style="margin-right:0.04398em;">z</span><span class="vlist"><span style="top:0.15em;margin-right:0.05em;margin-left:-0.04398em;"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span><span class="reset-textstyle scriptstyle cramped"><span class="mord mathit">t</span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span>​</span></span></span></span></span></span> and action <span class="katex"><span class="katex-mathml"><math><semantics><mrow><msub><mi>a</mi><mi>t</mi></msub></mrow><annotation encoding="application/x-tex">a_t</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.58056em;vertical-align:-0.15em;"></span><span class="base textstyle uncramped"><span class="mord"><span class="mord mathit">a</span><span class="vlist"><span style="top:0.15em;margin-right:0.05em;margin-left:0em;"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span><span class="reset-textstyle scriptstyle cramped"><span class="mord mathit">t</span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span>​</span></span></span></span></span></span> as an input to update its own hidden state to produce <span class="katex"><span class="katex-mathml"><math><semantics><mrow><msub><mi>h</mi><mrow><mi>t</mi><mo>+</mo><mn>1</mn></mrow></msub></mrow><annotation encoding="application/x-tex">h_{t+1}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.69444em;"></span><span class="strut bottom" style="height:0.902771em;vertical-align:-0.208331em;"></span><span class="base textstyle uncramped"><span class="mord"><span class="mord mathit">h</span><span class="vlist"><span style="top:0.15em;margin-right:0.05em;margin-left:0em;"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span><span class="reset-textstyle scriptstyle cramped"><span class="mord scriptstyle cramped"><span class="mord mathit">t</span><span class="mbin">+</span><span class="mord mathrm">1</span></span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span>​</span></span></span></span></span></span> to be used at time <span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>t</mi><mo>+</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">t+1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.64444em;"></span><span class="strut bottom" style="height:0.72777em;vertical-align:-0.08333em;"></span><span class="base textstyle uncramped"><span class="mord mathit">t</span><span class="mbin">+</span><span class="mord mathrm">1</span></span></span></span>.</figcaption>
</div>

Below is the pseudocode for how our agent model is used in the OpenAI Gym <dt-cite key="openai_gym"></dt-cite> environment. Running this function on a given `controller` C will return the cumulative reward during a rollout of the environment.

<dt-code block language="python">
def rollout(controller):
  ''' env, rnn, vae are '''
  ''' global variables  '''
  obs = env.reset()
  h = rnn.initial_state()
  done = False
  cumulative_reward = 0
  while not done:
    z = vae.encode(obs)
    a = controller.action([z, h])
    obs, reward, done = env.step(a)
    cumulative_reward += reward
    h = rnn.forward([a, z, h])
  return cumulative_reward
</dt-code>

This minimal design for C also offers important practical benefits. Advances in deep learning provided us with the tools to train large, sophisticated models efficiently, provided we can define a well-behaved, differentiable loss function. Our V and M models are designed to be trained efficiently with the backpropagation algorithm using modern GPU accelerators, so we would like most of the model's complexity, and model parameters to reside in V and M. The number of parameters of C, a linear model, is minimal in comparison. This choice allows us to explore more unconventional ways to train C -- for example, even using evolution strategies (ES) <dt-cite key="Rechenberg1973,Schwefel1977,visuales"></dt-cite> to tackle more challenging RL tasks where the credit assignment problem is difficult.

To optimize the parameters of C, we chose the Covariance-Matrix Adaptation Evolution Strategy (CMA-ES) <dt-cite key="cmaes,cmaes_original,visuales"></dt-cite> as our optimization algorithm since it is known to work well for solution spaces of up to a few thousand parameters. We evolve parameters of C on a single machine with multiple CPU cores running multiple rollouts of the environment in parallel.

For more specific information about the models, training procedures, and environments used in our experiments, please refer to the <a href="#appendix">Appendix</a>.

______

## Car Racing Experiment: World Model for Feature Extraction

A predictive world model can help us extract useful representations of space and time. By using these features as inputs of a controller, we can train a compact and minimal controller to perform a continuous control task, such as learning to drive from pixel inputs for a top-down car racing environment <dt-cite key="carracing_v0"></dt-cite>. In this section, we describe how we can train the Agent model described earlier to solve a car racing task. To our knowledge, our agent is the first known solution to achieve the score required to solve this task.<dt-fn>We find this task interesting because although it is not difficult to train an agent to wobble around randomly generated tracks and obtain a mediocre score, CarRacing-v0 defines "solving" as getting average reward of 900 over 100 consecutive trials, which means the agent can only afford very few driving mistakes.</dt-fn>

<div style="text-align: center;">
<video class="b-lazy" data-src="assets/mp4/carracing_mistake_short.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 100%;" ></video>
<figcaption>Our agent learning to navigate a top-down racing environment. <dt-cite key="carracing_v0"></dt-cite></figcaption>
</div>

In this environment, the tracks are randomly generated for each trial, and our agent is rewarded for visiting as many tiles as possible in the least amount of time. The agent controls three continuous actions: steering left/right, acceleration, and brake.

To train our V model, we first collect a dataset of 10,000 random rollouts of the environment. We have first an agent acting randomly to explore the environment multiple times, and record the random actions $a_t$ taken and the resulting observations from the environment.<dt-fn>We will discuss an iterative training procedure later on for more complicated environments where a random policy is not sufficient.</dt-fn> We use this dataset to train V to learn a latent space of each frame observed. We train our VAE to encode each frame into low dimensional latent vector $z$ by minimizing the difference between a given frame and the reconstructed version of the frame produced by the decoder from $z$. The following demo shows the results of our VAE after training:

<div style="text-align: left;">
<div id="carvae_sketch" class="unselectable"></div>
<figcaption style="color:#FF6C00;">Interactive Demo</figcaption>
<figcaption>Our VAE trained on observations from CarRacing-v0 <dt-cite key="carracing_v0"></dt-cite>. Despite losing details during this lossy compression process, latent vector <span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>z</mi></mrow><annotation encoding="application/x-tex">z</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.43056em;vertical-align:0em;"></span><span class="base textstyle uncramped"><span class="mord mathit" style="margin-right:0.04398em;">z</span></span></span></span> captures the essence of each 64x64px image frame.</figcaption>
</div>

We can now use our trained V model to pre-process each frame at time $t$ into $z_t$ to train our M model. Using this pre-processed data, along with the recorded random actions $a_t$ taken, our MDN-RNN can now be trained to model $P(z_{t+1} \; | \; a_t, z_t, h_t)$ as a mixture of Gaussians.<dt-fn>Although in principle, we can train V and M together in an end-to-end manner, we found that training each separately is more practical, achieves satisfactory results, and does not require exhaustive hyperparameter tuning. As images are not required to train M on its own, we can even train on large batches of long sequences of latent vectors encoding the entire 1000 frames of an episode to capture longer term dependencies, on a single GPU.</dt-fn>

In this experiment, the world model (V and M) has no knowledge about the actual reward signals from the environment. Its task is simply to compress and predict the sequence of image frames observed. Only the Controller (C) Model has access to the reward information from the environment. Since there are a mere 867 parameters inside the linear controller model, evolutionary algorithms such as CMA-ES are well suited for this optimization task.

The figure below compares actual the observation given to the agent and the observation captured by the world model. We can use the VAE to reconstruct each frame using $z_t$ at each time step to visualize the quality of the information the agent actually sees during a rollout:

<div>
<video class="b-lazy" data-src="assets/mp4/carracing_vae_compare.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 100%;" ></video>
<table style="text-align:center;width:100%;border:none">
  <tr>
    <td style="width:50%;border:none"><figcaption>Actual observations from the environment.</figcaption></td>
    <td style="width:50%;border:none"><figcaption>What gets encoded into <span class="katex"><span class="katex-mathml"><math><semantics><mrow><msub><mi>z</mi><mi>t</mi></msub></mrow><annotation encoding="application/x-tex">z_t</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.58056em;vertical-align:-0.15em;"></span><span class="base textstyle uncramped"><span class="mord"><span class="mord mathit" style="margin-right:0.04398em;">z</span><span class="vlist"><span style="top:0.15em;margin-right:0.05em;margin-left:-0.04398em;"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span><span class="reset-textstyle scriptstyle cramped"><span class="mord mathit">t</span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span>​</span></span></span></span></span></span>.</figcaption></td>
  </tr>
</table>
</div>

______

## Procedure

To summarize the Car Racing experiment, below are the steps taken:

1. Collect 10,000 rollouts from a random policy.
2. Train VAE (V) to encode each frame into a latent vector $z \in \mathcal{R}^{32}$.
3. Train MDN-RNN (M) to model $P(z_{t+1} \; | \; a_t, z_t, h_t)$.
4. Evolve Controller (C) to maximize the expected cumulative reward of a rollout.

<table style="text-align:left;width:320px;">
  <tr>
    <th>Model</th>
    <th>Parameter Count</th>
  </tr>
  <tr>
    <td>VAE</td>
    <td>4,348,547</td>
  </tr>
  <tr>
    <td>MDN-RNN</td>
    <td>422,368</td>
  </tr>
  <tr>
    <td>Controller</td>
    <td>867</td>
  </tr>
</table>

______

## Car Racing Experiment Results

### V Model Only

Training an agent to drive is not a difficult task if we have a good representation of the observation. Previous works <dt-cite key="browser_car,mar_io_kart,keras_car"></dt-cite> have shown that with a good set of hand-engineered information about the observation, such as LIDAR information, angles, positions and velocities, one can easily train a small feed-forward network to take this hand-engineered input and output a satisfactory navigation policy. For this reason, we first want to test our agent by handicapping C to only have access to V but not M, so we define our controller as $a_t = W_c \; z_t \;+ \; b_c$.

<div>
<video class="b-lazy" data-src="assets/mp4/carracing_z_only.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 100%;" ></video>
<figcaption>Limiting our controller to see only <span class="katex"><span class="katex-mathml"><math><semantics><mrow><msub><mi>z</mi><mi>t</mi></msub></mrow><annotation encoding="application/x-tex">z_t</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.58056em;vertical-align:-0.15em;"></span><span class="base textstyle uncramped"><span class="mord"><span class="mord mathit" style="margin-right:0.04398em;">z</span><span class="vlist"><span style="top:0.15em;margin-right:0.05em;margin-left:-0.04398em;"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span><span class="reset-textstyle scriptstyle cramped"><span class="mord mathit">t</span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span>​</span></span></span></span></span></span>, but not <span class="katex"><span class="katex-mathml"><math><semantics><mrow><msub><mi>h</mi><mi>t</mi></msub></mrow><annotation encoding="application/x-tex">h_t</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.69444em;"></span><span class="strut bottom" style="height:0.84444em;vertical-align:-0.15em;"></span><span class="base textstyle uncramped"><span class="mord"><span class="mord mathit">h</span><span class="vlist"><span style="top:0.15em;margin-right:0.05em;margin-left:0em;"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span><span class="reset-textstyle scriptstyle cramped"><span class="mord mathit">t</span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span>​</span></span></span></span></span></span> results in wobbly and unstable driving behaviours. </figcaption>
</div>

Although the agent is still able to navigate the race track in this setting, we notice it wobbles around and misses the tracks on sharper corners. This handicapped agent achieved an average score of 632 $\pm$ 251 over 100 random trials, in line with the performance of other agents on OpenAI Gym's leaderboard <dt-cite key="carracing_v0"></dt-cite> and traditional Deep RL methods such as A3C <dt-cite key="carracing_cs221,carracing_cs234"></dt-cite>. Adding a hidden layer to C's policy network helps to improve the results to 788 $\pm$ 141, but not quite enough to solve this environment.

### Full World Model (V and M)

The representation $z_t$ provided by our V model only captures a representation at a moment in time and doesn't have much predictive power. In contrast, M is trained to do one thing, and to do it really well, which is to predict $z_{t+1}$. Since M's prediction of $z_{t+1}$ is produced from the RNN's hidden state $h_t$ at time $t$, this vector is a good candidate for the set of learned features we can give to our agent. Combining $z_t$ with $h_t$ gives our controller C a good representation of both the current observation, and what to expect in the future.

<!--The agent only sees both $z_t$ and $h_t$.-->
<div>
<video class="b-lazy" data-src="assets/mp4/carracing_z_and_h.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 100%;" ></video>
<figcaption>Driving is more stable if we give our controller access to both <span class="katex"><span class="katex-mathml"><math><semantics><mrow><msub><mi>z</mi><mi>t</mi></msub></mrow><annotation encoding="application/x-tex">z_t</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.58056em;vertical-align:-0.15em;"></span><span class="base textstyle uncramped"><span class="mord"><span class="mord mathit" style="margin-right:0.04398em;">z</span><span class="vlist"><span style="top:0.15em;margin-right:0.05em;margin-left:-0.04398em;"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span><span class="reset-textstyle scriptstyle cramped"><span class="mord mathit">t</span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span>​</span></span></span></span></span></span> and <span class="katex"><span class="katex-mathml"><math><semantics><mrow><msub><mi>h</mi><mi>t</mi></msub></mrow><annotation encoding="application/x-tex">h_t</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.69444em;"></span><span class="strut bottom" style="height:0.84444em;vertical-align:-0.15em;"></span><span class="base textstyle uncramped"><span class="mord"><span class="mord mathit">h</span><span class="vlist"><span style="top:0.15em;margin-right:0.05em;margin-left:0em;"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span><span class="reset-textstyle scriptstyle cramped"><span class="mord mathit">t</span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span style="font-size:0em;">​</span></span>​</span></span></span></span></span></span>.</figcaption>
</div>

Indeed, we see that allowing the agent to access the both $z_t$ and $h_t$ greatly improves its driving capability. The driving is more stable, and the agent is able to seemingly attack the sharp corners effectively. Furthermore, we see that in making these fast reflexive driving decisions during a car race, the agent does not need to *plan ahead* and roll out hypothetical scenarios of the future. Since $h_t$ contain information about the probability distribution of the future, the agent can just query the RNN instinctively to guide its action decisions. Like a seasoned Formula One driver or the baseball player discussed earlier, the agent can instinctively predict when and where to navigate in the heat of the moment.

| Method | $\;\;$ Average Score over 100 Random Tracks $\;\;$ |
|---|---|
| DQN <dt-cite key="dqn_racecar"></dt-cite> | ->343 $\pm$ 18<- |
| A3C (continuous) <dt-cite key="carracing_cs234"></dt-cite> | ->591 $\pm$ 45<- |
| A3C (discrete) <dt-cite key="carracing_cs221"></dt-cite> | ->652 $\pm$ 10<- |
| ceobillionaire's algorithm (unpublished) <dt-cite key="carracing_v0"></dt-cite>  | ->838 $\pm$ 11<-  |
| V model only, $z$ input  | ->632 $\pm$ 251<-  |
| V model only, $z$ input with a hidden layer | ->788 $\pm$ 141<-  |
| **Full World Model**, $z$ and $h$ | ->**906 $\pm$ 21**<-  |

Our agent was able to achieve a score of 906 $\pm$ 21 over 100 random trials, effectively solving the task and obtaining new state of the art results. Previous attempts <dt-cite key="carracing_cs221,carracing_cs234"></dt-cite> using traditional Deep RL methods obtained average scores of 591--652 range, and the best reported solution on the leaderboard <dt-cite key="carracing_v0"></dt-cite> obtained an average score of 838 $\pm$ 11 over 100 random consecutive trials. Traditional Deep RL methods often require pre-processing of each frame, such as employing edge-detection <dt-cite key="carracing_cs234"></dt-cite>, in addition to stacking a few recent frames <dt-cite key="carracing_cs221,carracing_cs234"></dt-cite> into the input. In contrast, our world model takes in a stream of raw RGB pixel images and directly learns a spatial-temporal representation. To our knowledge, our method is the first reported solution to solve this task.

______

## Car Racing Dreams

Since our world model is able to model the future, we are also able to have it come up with hypothetical car racing scenarios on its own. We can ask it to produce the probability distribution of $z_{t+1}$ given the current states, *sample* a $z_{t+1}$ and use this sample as the real observation. We can put our trained C back into this dream environment generated by M. The following demo shows how our world model can be used to generate the car racing environment:

<div style="text-align: left;">
<div id="carrnn_sketch" class="unselectable"></div>
<figcaption style="color:#FF6C00;">Interactive Demo</figcaption>
<figcaption>Our agent driving inside of its own dream world. Here, we deploy our trained policy into a fake environment generated by the MDN-RNN, and rendered using the VAE's decoder. You can override the agent's actions by tapping on the left or right side of the screen, or by hitting arrow keys (left/right to steer, up/down to accelerate or brake). The uncertainty level of the environment can be adjusted by changing <span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>τ</mi></mrow><annotation encoding="application/x-tex">\tau</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.43056em;vertical-align:0em;"></span><span class="base textstyle uncramped"><span class="mord mathit" style="margin-right:0.1132em;">τ</span></span></span></span> using the slider on the bottom right.</figcaption>
</div>

We have just seen that a policy learned inside of the real environment appears to somewhat function inside of the dream environment. This begs the question -- can we train our agent to learn inside of its own dream, and transfer this policy back to the actual environment?

______

## VizDoom Experiment: Learning Inside of a Dream

If our world model is sufficiently accurate for its purpose, and complete enough for the problem at hand, we should be able to substitute the actual environment with this world model. After all, our agent does not directly observe the reality, but only sees what the world model lets it see. In this experiment, we train an agent inside the dream environment generated by its world model trained to mimic a VizDoom <dt-cite key="vizdoom"></dt-cite> environment.

<div style="text-align: center;">
<video class="b-lazy" data-src="assets/mp4/doom_lazy_small.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 100%;" ></video>
<figcaption>Our final agent solving the <i>VizDoom: Take Cover</i> environment. <dt-cite key="vizdoom,takecover"></dt-cite></figcaption>
</div>

The agent must learn to avoid fireballs shot by monsters from the other side of the room with the sole intent of killing the agent. There are no explicit rewards in this environment, so to mimic natural selection, the cumulative reward can be defined to be the number of time steps the agent manages to stay alive during a rollout. Each rollout in the environment runs for a maximum of 2100 time steps ($\sim$ 60 seconds), and the task is considered solved if the average survival time over 100 consecutive rollouts is greater than 750 time steps ($\sim$ 20 seconds) <dt-cite key="takecover"></dt-cite>.

______

## Procedure

The setup of our VizDoom experiment is largely the same as the Car Racing task, except for a few key differences. In the Car Racing task, M is only trained to model the next $z_{t}$. Since we want to build a world model we can train our agent in, our M model here will also predict whether the agent dies in the next frame (as a binary event $done_t$, or $d_t$ for short), in addition to the next frame $z_t$.

Since the M model can predict the $done$ state in addition to the next observation, we now have all of the ingredients needed to make a full RL environment. We first build an OpenAI Gym environment interface by wrapping a `gym.Env` <dt-cite key="openai_gym"></dt-cite> interface over our M if it were a real Gym environment, and then train our agent inside of this *virtual* environment instead of using the actual environment.

In this simulation, we don't need the V model to encode any real pixel frames during the hallucination process, so our agent will therefore only train entirely in a latent space environment. This has many advantages that will be discussed later on.

This virtual environment has an identical interface to the real environment, so after the agent learns a satisfactory policy in the virtual environment, we can easily deploy this policy back into the actual environment to see how well the policy transfers over.

To summarize the *Take Cover* experiment, below are the steps taken:

1. Collect 10,000 rollouts from a random policy.
2. Train VAE (V) to encode each frame into a latent vector $z \in \mathcal{R}^{64}$, and use V to convert the images collected from (1) into the latent space representation.
3. Train MDN-RNN (M) to model $P(z_{t+1}, d_{t+1} \; | \; a_t, z_t, h_t)$.
4. Evolve Controller (C) to maximize the expected survival time inside the virtual environment.
5. Use learned policy from (4) on actual Gym environment.

<table style="text-align:left;width:320px;">
  <tr>
    <th>Model</th>
    <th>Parameter Count</th>
  </tr>
  <tr>
    <td>VAE</td>
    <td>4,446,915</td>
  </tr>
  <tr>
    <td>MDN-RNN</td>
    <td>1,678,785</td>
  </tr>
  <tr>
    <td>Controller</td>
    <td>1,088</td>
  </tr>
</table>

______

## Training Inside of the Dream

After some training, our controller learns to navigate around the dream environment and escape from deadly fireballs launched by monsters generated by the M model.<!--a policy where it can avoid fireballs
by moving from one side of the room to the other side of the room. Since the monsters shoot at where the player is located, rather than where the player will be in the future, this is a reasonable policy, and also a strategy that human players often discover as well.<dt-fn>This is also the policy that the author discovered while playing the game himself.</dt-fn>--> Our agent achieved a *score* in this virtual environment of $\sim$ 900 time steps.

The following demo shows how our agent navigates inside its own dream. The M model learns to generate monsters that shoot fireballs at the direction of the agent, while the C model discovers a policy to avoid these generated fireballs. Here, the V model is only used to decode the latent vectors $z_t$ produced by M into a sequence of pixel images we can see:

<div style="text-align: left;">
<div id="doomrnn_sketch" class="unselectable"></div>
<figcaption style="color:#FF6C00;">Interactive Demo</figcaption>
<figcaption>Our agent discovers a policy to avoid generated fireballs. In this demo, you can override the agent's action by using the left/right keys on your keyboard, or by tapping on either side of the screen. You can also control the uncertainty level of the environment by adjusting the temperature parameter using slider on the bottom right.</dt-fn></figcaption>
</div>

Here, our RNN-based world model is trained to mimic a complete game environment designed by human programmers. By learning only from raw image data collected from random episodes, it learns how to simulate the essential aspects of the game -- such as the game logic, enemy behaviour, physics, and also the 3D graphics rendering.

For instance, if the agent selects the left action, the M model learns to move the agent to the left and adjust its internal representation of the game states accordingly. It also learns to block the agent from moving beyond the walls on both sides of the level if the agent attempts to move too far in either direction. Occasionally, the M model needs to keep track of multiple fireballs being shot from several different monsters and coherently move them along in their intended directions. It must also detect whether the agent has been killed by one of these fireballs.

Unlike the actual game environment, however, we note that it is possible to add extra uncertainty into the virtual environment, thus making the game more challenging in the dream environment. We can do this by increasing the temperature $\tau$ parameter during the sampling process of $z_{t+1}$, as done in <dt-cite key="sketchrnn"></dt-cite>. By increasing the uncertainty, our dream environment becomes more difficult compared to the actual environment. The fireballs may move more randomly in a less predictable path compared to the actual game. Sometimes the agent may even die due to sheer misfortune, without explanation.

We find agents that perform well in higher temperature settings generally perform better in the normal setting. In fact, increasing $\tau$ helps prevent our controller from taking advantage of the imperfections of our world model -- we will discuss this in more depth later on.

______

## Transfer Policy to Actual Environment

<div>
<video class="b-lazy" data-src="assets/mp4/doom_real_deploy.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 100%;" ></video>
<figcaption>Deploying our policy learned inside of the dream RNN environment back into the actual VizDoom environment.</figcaption>
</div>

We took the agent trained inside of the virtual environment and tested its performance on the original VizDoom scenario. The score over 100 random consecutive trials is $\sim$ 1100 time steps, far beyond the required score of 750 time steps, and also much higher than the score obtained inside the more difficult virtual environment.<dt-fn>We will discuss how this score compares to other models later on.</dt-fn>

<div>
<video class="b-lazy" data-src="assets/mp4/doom_real_vae.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 100%;" ></video>
<table style="text-align:center;width:100%;border:none">
  <tr>
    <td style="width:50%;border:none"><figcaption>Cropped 64x64px frame of environment.</figcaption></td>
    <td style="width:50%;border:none"><figcaption>Reconstruction from latent vector.</figcaption></td>
  </tr>
</table>
</div>

We see that even though the V model is not able to capture all of the details of each frame correctly, for instance, getting the number of monsters correct, the agent is still able to use the learned policy to navigate in the real environment. As the virtual environment cannot even keep track of the exact number of monsters in the first place, an agent that is able to survive the noisier and uncertain virtual nightmare environment will thrive in the original, cleaner environment.

______

## Cheating the World Model

In our childhood, we may have encountered ways to exploit video games in ways that were not intended by the original game designer <dt-cite key="video_game_exploits"></dt-cite>. Players discover ways to collect unlimited lives or health, and by taking advantage of these exploits, they can easily complete an otherwise difficult game. However, in the process of doing so, they may have forfeited the opportunity to learn the skill required to master the game as intended by the game designer. In our initial experiments, we noticed that our agent discovered an *adversarial* policy to move around in such a way so that the monsters in this virtual environment governed by M never shoots a single fireball during some rollouts. Even when there are signs of a fireball forming, the agent moves in a way to *extinguish* the fireballs.

<div>
<video class="b-lazy" data-src="assets/mp4/doom_adversarial.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 100%;" ></video>
<figcaption>Agent discovers an adversarial policy that fools the monsters inside the world model into never launching any fireballs during some rollouts.</figcaption>
</div>

Because M is only an approximate probabilistic model of the environment, it will occasionally generate trajectories that do not follow the laws governing the actual environment. As we previously pointed out, even the number of monsters on the other side of the room in the actual environment is not exactly reproduced by M. For this reason, our world model will be exploitable by C, even if such exploits do not exist in the actual environment.

As a result of using M to generate a virtual environment for our agent, we are also giving the controller access to all of the hidden states of M. This is essentially granting our agent access to all of the internal states and memory of the game engine, rather than only the game observations that the player gets to see. Therefore our agent can efficiently explore ways to directly manipulate the hidden states of the game engine in its quest to maximize its expected cumulative reward. The weakness of this approach of learning a policy inside of a learned dynamics model is that our agent can easily find an adversarial policy that can fool our dynamics model -- it will find a policy that looks good under our dynamics model, but will fail in the actual environment, usually because it visits states where the model is wrong because they are away from the training distribution.

This weakness could be the reason that many previous works that learn dynamics models of RL environments do not actually use those models to fully replace the actual environments <dt-cite key="action_conditional_video_prediction,recurrent_env_sim"></dt-cite>. Like in the M model proposed in <dt-cite key="s05_making_the_world_differentiable,s05a_cm,s05b_rl"></dt-cite>, the dynamics model is deterministic, making it easily exploitable by the agent if it is not perfect. Using Bayesian models, as in PILCO <dt-cite key="pilco"></dt-cite>, helps to address this issue with the uncertainty estimates to some extent, however, they do not fully solve the problem. Recent work <dt-cite key="Nagabandi2017"></dt-cite> combines the model-based approach with traditional model-free RL training by first initializing the policy network with the learned policy, but must subsequently rely on model-free methods to fine-tune this policy in the actual environment.

To make it more difficult for our C to exploit deficiencies of M, we chose to use the MDN-RNN as the dynamics model of the *distribution* of possible outcomes in the actual environment, rather than merely predicting a deterministic future. Even if the actual environment is deterministic, the MDN-RNN would in effect approximate it as a stochastic environment. This has the advantage of allowing us to train C inside a more stochastic version of any environment -- we can simply adjust the temperature parameter $\tau$ to control the amount of randomness in M, hence controlling the tradeoff between realism and exploitability.

Using a mixture of Gaussian model may seem excessive given that the latent space encoded with the VAE model is just a single diagonal Gaussian distribution. However, the discrete modes in a mixture density model are useful for environments with random discrete events, such as whether a monster decides to shoot a fireball or stay put. While a single diagonal Gaussian might be sufficient to encode individual frames, an RNN with a mixture density output layer makes it easier to model the logic behind a more complicated environment with discrete random states.

For instance, if we set the temperature parameter to a very low value of $\tau=0.1$, effectively training our C  with an M that is almost identical to a deterministic LSTM, the monsters inside this generated environment fail to shoot fireballs, no matter what the agent does, due to mode collapse. M is not able to transition to another mode in the mixture of Gaussian model where fireballs are formed and shot. Whatever policy learned inside of this generated environment will achieve a perfect score of 2100 most of the time, but will obviously fail when unleashed into the harsh reality of the actual world, underperforming even a random policy.

In the following demo, we show that even low values of $\tau \sim 0.5$ make it difficult for the MDN-RNN to generate fireballs:

<div style="text-align: center;">
<div id="doomrnn_cheating_sketch" class="unselectable"></div>
<figcaption style="color:#FF6C00;">Interactive Demo</figcaption>
<figcaption>For low <span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>τ</mi></mrow><annotation encoding="application/x-tex">\tau</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.43056em;vertical-align:0em;"></span><span class="base textstyle uncramped"><span class="mord mathit" style="margin-right:0.1132em;">τ</span></span></span></span> settings, monsters in the M model rarely shoot fireballs. Even when you try to increase <span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>τ</mi></mrow><annotation encoding="application/x-tex">\tau</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height:0.43056em;"></span><span class="strut bottom" style="height:0.43056em;vertical-align:0em;"></span><span class="base textstyle uncramped"><span class="mord mathit" style="margin-right:0.1132em;">τ</span></span></span></span> to 1.0 using the slider bar, the agent will occasionally extinguish fireballs still being formed, by fooling M.</figcaption>
</div>

By making the temperature $\tau$ an adjustable parameter of M, we can see the effect of training C inside of virtual environments with different levels of uncertainty, and see how well they transfer over to the actual environment. We experiment with varying $\tau$ of the virtual environment, training an agent inside of this virtual environment, and observing its performance when inside the actual environment.

| ->$\;\;$Temperature$\;\;$<- | ->$\;\;$ Score in Virtual Environment<- | ->$\;\;$Score in Actual Environment$\;\;$<- |
|---|---|---|
| ->0.10<- | ->2086 $\pm$ 140<- | ->193 $\pm$ 58<-  |
| ->0.50<- | ->2060 $\pm$ 277<- | ->196 $\pm$ 50<-  |
| ->1.00<- | ->1145 $\pm$ 690<- | ->868 $\pm$ 511<-  |
| ->1.15<- | ->918 $\pm$ 546<- | ->1092 $\pm$ 556<-  |
| ->1.30<- | ->732 $\pm$ 269<- | ->753 $\pm$ 139<-  |
| ->Random Policy Baseline<- | ->N/A<- | ->210 $\pm$ 108<- |
| ->Gym Leaderboard <dt-cite key="takecover"></dt-cite><- | ->N/A<- | ->820 $\pm$ 58<- |

In the table above, while we see that increasing $\tau$ of M makes it more difficult for C to find adversarial policies, increasing it too much will make the virtual environment too difficult for the agent to learn anything, hence in practice it is a hyperparameter we can tune. The temperature also affects the types of strategies the agent discovers. For example, although the best score obtained is 1092 $\pm$ 556 with $\tau=1.15$, increasing $\tau$ a notch to 1.30 results in a lower score but at the same time a less risky strategy with a lower variance of returns. For comparison, the best reported score <dt-cite key="takecover"></dt-cite> is 820 $\pm$ 58.

______

## Iterative Training Procedure

In our experiments, the tasks are relatively simple, so a reasonable world model can be trained using a dataset collected from a random policy. But what if our environments become more sophisticated? In any difficult environment, only parts of the world are made available to the agent only after it learns how to strategically navigate through its world.

For more complicated tasks, an iterative training procedure is required. We need our agent to be able to explore its world, and constantly collect new observations so that its world model can be improved and refined over time. An iterative training procedure, adapted from *Learning To Think* <dt-cite key="learning_to_think"></dt-cite> is as follows:

1. Initialize M, C with random model parameters.
2. Rollout to actual environment $N$ times. Agent may learn during rollouts. Save all actions $a_t$ and observations $x_t$ during rollouts to storage device.
3. Train M to model $P(x_{t+1}, r_{t+1}, a_{t+1}, d_{t+1} \; | \; x_t, a_t, h_t)$ and train C to optimize expected rewards inside of M.
4. Go back to (2) if task has not been completed.

We have shown that one iteration of this training loop was enough to solve simple tasks. For more difficult tasks, we need our controller in Step 2 to actively explore parts of the environment that is beneficial to improve its world model. An exciting research direction is to look at ways to incorporate artificial curiosity and intrinsic motivation <dt-cite key="schmidhuber_creativity,s07_intrinsic,s08_curiousity,pathak2017,intrinsic_motivation"></dt-cite> and information seeking <dt-cite key="SchmidhuberStorck:94,Gottlieb2013"></dt-cite> abilities in an agent to encourage novel exploration <dt-cite key="Lehman2011"></dt-cite>. In particular, we can augment the reward function based on improvement in compression quality <dt-cite key="schmidhuber_creativity,s07_intrinsic,s08_curiousity,learning_to_think"></dt-cite>.

<div style="text-align: center;">
<video class="b-lazy" data-src="assets/mp4/pendulum01.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 100%;" ></video>
<figcaption>Swing-up Pendulum from Pixels: Generated rollout after the first iteration. M has difficulty predicting states of a swung up pole since the data collected from the initial random policy is near the steady state in the bottom half. Despite this, C still learns to swing the pole upwards when deployed inside of M. </figcaption>
</div>

<div style="text-align: center;">
<video class="b-lazy" data-src="assets/mp4/pendulum20.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 100%;" ></video>
<figcaption>Swing-up Pendulum from Pixels: Generated rollout after 20 iterations. Deploying policies that swing the pole upwards in the actual environment gathered more data that recorded the pole being in the top half, allowing M to model the environment more accurately, and C to learn a better policy inside of M.</figcaption>
</div>

In the present approach, since M is a MDN-RNN that models a probability distribution for the next frame, if it does a poor job, then it means the agent has encountered parts of the world that it is not familiar with. Therefore we can adapt and reuse M's training loss function to encourage curiosity. By flipping the sign of M's loss function in the actual environment, the agent will be encouraged to explore parts of the world that it is not familiar with. The new data it collects may improve the world model.

The iterative training procedure requires the M model to not only predict the next observation $x$ and $done$, but also predict the action and reward for the next time step. This may be required for more difficult tasks. For instance, if our agent needs to learn complex motor skills to walk around its environment, the world model will learn to imitate its own C model that has already learned to walk. After difficult motor skills, such as walking, is absorbed into a large world model with lots of capacity, the smaller C model can rely on the motor skills already absorbed by the world model and focus on learning more higher level skills to navigate itself using the motor skills it had already learned.<dt-fn>Another related connection is to muscle memory. For instance, as you learn to do something like play the piano, you no longer have to spend working memory capacity on translating individual notes to finger motions -- this all becomes encoded at a subconscious level.</dt-fn>

<div style="text-align: center;">
<img class="b-lazy" src=data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== data-src="assets/memory_consolidation.svg" style="display: block; margin: auto; width: 86%;"/>
<br/>
<figcaption style="text-align: center;">How information becomes memory. <dt-cite key="memory_consolidation"></dt-cite></figcaption>
</div>

An interesting connection to the neuroscience literature is the work on hippocampal replay that examines how the brain replays recent experiences when an animal rests or sleeps. Replaying recent experiences plays an important role in memory consolidation <dt-cite key="Foster2017"></dt-cite> -- where hippocampus-dependent memories become independent of the hippocampus over a period of time <dt-cite key="memory_consolidation"></dt-cite>. As Foster <dt-cite key="Foster2017"></dt-cite> puts it, replay is "less like dreaming and more like thought". We invite readers to read *Replay Comes of Age* <dt-cite key="Foster2017"></dt-cite> for a detailed overview of replay from a neuroscience perspective with connections to theoretical reinforcement learning.

Iterative training could allow the C--M model to develop a natural hierarchical way to learn. Recent works about self-play in RL <dt-cite key="asymmetric_self_play,competitive_self_play,continuous_adaptation_via_meta_learning"></dt-cite> and PowerPlay <dt-cite key="s10_powerplay,s11_powerplay"></dt-cite> also explores methods that lead to a natural curriculum learning <dt-cite key="s09_optimal_order"></dt-cite>, and we feel this is one of the more exciting research areas of reinforcement learning.

______

## Related Work

There is extensive literature on learning a dynamics model, and using this model to train a policy. Many concepts first explored in the 1980s for feed-forward neural networks (FNNs) <dt-cite key="Werbos87specifications,Munro87,RobinsonFallside89,Werbos89identification,NguyenWidrow89"></dt-cite> and in the 1990s for RNNs <dt-cite key="s05_making_the_world_differentiable,s05a_cm,s05b_rl,s05c_boredom"></dt-cite> laid some of the groundwork for *Learning to Think* <dt-cite key="learning_to_think"></dt-cite>. The more recent PILCO <dt-cite key="pilco,pilco_tutorial,McAllister2017"></dt-cite> is a probabilistic model-based search policy method designed to solve difficult control problems. Using data collected from the environment, PILCO uses a Gaussian process (GP) model to learn the system dynamics, and then uses this model to sample many trajectories in order to train a controller to perform a desired task, such as swinging up a pendulum, or riding a unicycle.

While Gaussian processes work well with a small set of low dimensional data, their computational complexity makes them difficult to scale up to model a large history of high dimensional observations. Other recent works <dt-cite key="deep_pilco,Depeweg2017"></dt-cite> use Bayesian neural networks instead of GPs to learn a dynamics model. These methods have demonstrated promising results on challenging control tasks <dt-cite key="Hein2017"></dt-cite>, where the states are known and well defined, and the observation is relatively low dimensional. Here we are interested in modelling dynamics observed from high dimensional visual data where our input is a sequence of raw pixel frames.

In robotic control applications, the ability to learn the dynamics of a system from observing only camera-based video inputs is a challenging but important problem. Early work on RL for active vision trained an FNN to take the current image frame of a video sequence to predict the next frame <dt-cite key="s04_trajectories"></dt-cite>, and use this predictive model to train a fovea-shifting control network trying to find targets in a visual scene. To get around the difficulty of training a dynamical model to learn directly from high-dimensional pixel images, researchers explored using neural networks to first learn a compressed representation of the video frames.  Recent work along these lines <dt-cite key="learning_deep_dynamical_models_from_image_pixels,from_pixels_to_torques"></dt-cite> was able to train controllers using the bottleneck hidden layer of an autoencoder as low-dimensional feature vectors to control a pendulum from pixel inputs. Learning a model of the dynamics from a compressed latent space enable RL algorithms to be much more data-efficient <dt-cite key="deep_spacial_autoencoders,embed_to_control,finn_lecture"></dt-cite>. We invite readers to watch Finn's lecture on Model-Based RL <dt-cite key="finn_lecture"></dt-cite> to learn more.

Video game environments are also popular in model-based RL research as a testbed for new ideas. Guzdial et al. <dt-cite key="game_engine_learning"></dt-cite> used a feed-forward convolutional neural network (CNN) to learn a forward simulation model of a video game. Learning to predict how different actions affect future states in the environment is useful for game-play agents, since if our agent can predict what happens in the future given its current state and action, it can simply select the best action that suits its goal. This has been demonstrated not only in early work <dt-cite key="NguyenWidrow89,s04_trajectories"></dt-cite> (when compute was a million times more expensive than today) but also in recent studies <dt-cite key="learn_to_act_by_predicting_future"></dt-cite> on several competitive VizDoom <dt-cite key="vizdoom"></dt-cite> environments.

The works mentioned above use FNNs to predict the next video frame. We may want to use models that can capture longer term time dependencies. RNNs are powerful models suitable for sequence modelling <dt-cite key="graves_rnn"></dt-cite>. In a lecture called *Hallucination with RNNs* <dt-cite key="graves_lecture"></dt-cite>, Graves demonstrated the ability of RNNs to learn a probabilistic model of Atari game environments. He trained RNNs to learn the structure of such a game and then showed that they can hallucinate similar game levels on its own.

<div style="text-align: center;">
<img class="b-lazy" src=data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== data-src="assets/world_models_1990.jpeg" style="display: block; margin: auto; width: 80%;"/>
<figcaption style="text-align: center;">A controller with internal RNN model of the world. <dt-cite key="s05_making_the_world_differentiable"></dt-cite></figcaption>
</div>

Using RNNs to develop internal models to reason about the future has been explored as early as 1990 in a paper called *Making the World Differentiable* <dt-cite key="s05_making_the_world_differentiable"></dt-cite>, and then further explored in <dt-cite key="s05a_cm,s05b_rl,s05c_boredom"></dt-cite>. A more recent paper called *Learning to Think* <dt-cite key="learning_to_think"></dt-cite> presented a unifying framework for building a RNN-based general problem solver that can learn a world model of its environment and also learn to reason about the future using this model. Subsequent works have used RNN-based models to generate many frames into the future <dt-cite key="recurrent_env_sim,action_conditional_video_prediction,Denton2017"></dt-cite>, and also as an internal model to reason about the future <dt-cite key="Silver2016,imagination_agent,Watters2017"></dt-cite>.

In this work, we used evolution strategies (ES) to train our controller, as it offers many benefits. For instance, we only need to provide the optimizer with the final cumulative reward, rather than the entire history. ES is also easy to parallelize -- we can launch many instances of `rollout` with different solutions to many workers and quickly compute a set of cumulative rewards in parallel. Recent works <dt-cite key="pathnet,openai,stablees,stanley2017"></dt-cite> have confirmed that ES is a viable alternative to traditional Deep RL methods on many strong baseline tasks.

Before the popularity of Deep RL methods <dt-cite key="dqn"></dt-cite>, evolution-based algorithms have been shown to be effective at finding solutions for RL tasks <dt-cite key="neat,gom5_ne_accelerated,gom2_coevolve,hyperneat,pepg,evolving_neural_networks"></dt-cite>. Evolution-based algorithms have even been able to solve difficult RL tasks from high dimensional pixel inputs <dt-cite key="kou1_torcs,hausknecht,parker2012"></dt-cite>. More recent works <dt-cite key="vae_evolution"></dt-cite> also combine VAE and ES, which is similar to our approach.

______

## Discussion

We have demonstrated the possibility of training an agent to perform tasks entirely inside of its simulated latent space world. This approach offers many practical benefits. For instance, video game engines typically require heavy compute resources for rendering the game states into image frames, or calculating physics not immediately relevant to the game. We may not want to waste cycles training an agent in the actual environment, but instead train the agent as many times as we want inside its simulated environment. Agents that are trained incrementally to simulate reality may prove to be useful for transferring policies back to the real world. Our approach may complement *sim2real* approaches outlined in previous work <dt-cite key="Bousmalis2017,Higgins2017"></dt-cite>.

Furthermore, we can take advantage of deep learning frameworks to accelerate our world model simulations using GPUs in a distributed environment. The benefit of implementing the world model as a fully differentiable recurrent computation graph also means that we may be able to train our agents in the dream directly using the backpropagation algorithm to fine-tune its policy to maximize an objective function <dt-cite key="s05_making_the_world_differentiable,s05a_cm,s05b_rl"></dt-cite>.

The choice of implementing V as a VAE and training it as a standalone model also has its limitations, since it may encode parts of the observations that are not relevant to a task. After all, unsupervised learning cannot, by definition, know what will be useful for the task at hand. For instance, our VAE reproduced unimportant detailed brick tile patterns on the side walls in the Doom environment, but failed to reproduce task-relevant tiles on the road in the Car Racing environment. By training together with an M that predicts rewards, the VAE may learn to focus on task-relevant areas of the image, but the tradeoff here is that we may not be able to reuse the VAE effectively for new tasks without retraining. Learning task-relevant features has connections to neuroscience as well. Primary sensory neurons are released from inhibition when rewards are received, which suggests that they generally learn task-relevant features, rather than just any features, at least in adulthood <dt-cite key="Pi2013"></dt-cite>.

Another concern is the limited capacity of our world model. While modern storage devices can store large amounts of historical data generated using an iterative training procedure, our LSTM-based <dt-cite key="lstm,s12_lstm_forget"></dt-cite> world model may not be able to store all of the recorded information inside of its weight connections. While the human brain can hold decades and even centuries of memories to some resolution <dt-cite key="brain_capacity"></dt-cite>, our neural networks trained with backpropagation have more limited capacity and suffer from issues such as catastrophic forgetting <dt-cite key="Ratcliff1990,French1994,Kirkpatrick2016"></dt-cite>. Future work will explore replacing the VAE and MDN-RNN with higher capacity models <dt-cite key="outrageously_large_neural_nets,hypernetworks,suarez2017,wavenet,attention"></dt-cite>, or incorporating an external memory module <dt-cite key="Gemici2017"></dt-cite>, if we want our agent to learn to explore more complicated worlds.

<div style="text-align: center;">
<img class="b-lazy" src=data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== data-src="assets/world_models_1990_feedback.jpeg" style="display: block; margin: auto; width: 65%;"/>
<figcaption style="text-align: center;">Ancient drawing (1990) of a RNN-based controller interacting with an environment. <dt-cite key="s05_making_the_world_differentiable"></dt-cite></figcaption>
</div>

Like early RNN-based C--M systems <dt-cite key="s05_making_the_world_differentiable,s05a_cm,s05b_rl,s05c_boredom"></dt-cite>, ours simulates possible futures time step by time step, without profiting from human-like hierarchical planning or abstract reasoning, which often ignores irrelevant spatial-temporal details. However, the more general *Learning To Think* <dt-cite key="learning_to_think"></dt-cite> approach is not limited to this rather naive approach. Instead it allows a recurrent C to learn to address "subroutines" of the recurrent M, and reuse them for problem solving in arbitrary computable ways, e.g., through hierarchical planning or other kinds of exploiting parts of M's program-like weight matrix. A recent *One Big Net* <dt-cite key="onebignet2018"></dt-cite> extension of the C--M approach
collapses C and M into a single network, and uses PowerPlay-like <dt-cite key="s10_powerplay,s11_powerplay"></dt-cite> behavioural replay (where the behaviour of a teacher net is compressed into a student net <dt-cite key="chunker91and92"></dt-cite>) to avoid forgetting old prediction and control skills when learning new ones. Experiments with those more general approaches are left for future work.

*If you would like to discuss any issues or give feedback, please visit the [GitHub](https://github.com/worldmodels/worldmodels.github.io/issues) repository of this page for more information.*
