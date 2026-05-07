1/6
In his Week 11 blog, my colleague Ephrata made a crucial observation: prompt engineering alone couldn't capture the Tenacious sales tone. Fine-tuning was required. But what actually changes at the weight level that makes training succeed where a prompt fails? 🧵

2/6
When you write a system prompt, you do not change the model's weights. You change its transient activation states. The attention mechanism acts as a spotlight, dynamically routing information from your instruction tokens to the generated output. It's short-term memory.

3/6
Because a prompt is a spotlight, it can only illuminate what's already there. If you prompt "be a professional sales rep," the model attends to the strongest pre-trained association for that concept: generic, clichéd corporate speak ("I hope this email finds you well").

4/6
Fine-tuning, however, permanently updates the projection matrices ($W_Q, W_K, W_V, W_O$) and MLP layers. It rewrites the base probability distribution of the vocabulary. You aren't fighting the model's natural gravitational pull with a prompt spotlight; you are moving the center of gravity.

5/6
This explains "The Walrus Effect" (over-indexing). Prompts apply stylistic constraints bluntly, resulting in caricatures (acting like a pirate = saying "Arrr!" constantly). Tone is a distributed statistical property requiring millions of micro-decisions that only weight shifts can handle smoothly.

6/6
A prompt gives the model new instructions; fine-tuning changes its structural reflexes. For deep style transfer, you cannot out-prompt a fundamental weight distribution. 
Read my full explainer on the weight-level mechanics here: [Link to Blog]
