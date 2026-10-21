# Episode 2: Mario Negrello, neuroscience × AI × education (2026-10-21)

**With:** Gabe, Stan, Jimmy (Stan's bot) and guest **Mario Negrello**, a computational neuroscientist at Erasmus MC Rotterdam.
**Recorded:** a Google Meet, not live. Streaming still needs OBS (see [docs/streaming-setup.md](../docs/streaming-setup.md)).

## Topics

### The cortical column as a Transformer
Mario and Peter König (Osnabrück) wrote *The Neuroscience of Transformers*. Their reading:
- Top-down "apical" input arrives at layer 1, and bottom-up thalamic input arrives at layer 4.
- The two meet in pyramidal cells, and a Ca²⁺ spike marks it when they coincide.
- Their correlation acts like an attention matrix, which routes information multiplicatively.

AI took its metaphors from neuroscience; now the arrow points back. The pyramidal-neuron idea goes back to König in the 90s and was later taken up by Bengio.

### Equilibrium propagation
Scellier & Bengio's local learning rule:
1. Let the network settle with the input clamped.
2. Nudge the outputs toward the target and let it settle again.
3. Compare the two equilibria.

As the nudge gets small, this approximates backprop without a separate backward pass. Gabe brought up the genre of "X is just Y" papers, such as backprop as linear attention.

### Learning Gradients and "NeuroAI: The Computing Brain"
- [Learning Gradients](https://learninggradients.substack.com/): Mario's Substack on education when the LLM "knows everything". The first skill is **selection**: choose your path, and own the knowledge you choose.
- **The platform:** "Google Maps for knowledge", used in Erasmus's NeuroAI minor.
  - It started as Mario's mind map, turned by GPT into a JSON ontology and then into learning goals.
  - A recommender (local Gemma) takes a destination plus your baseline and suggests a path.
  - The skill map is modelled on the Path of Exile passive tree. Students self-mark skills, and there's a group progress view.
  - **Card game:** about 70 cards, each a verb + a phenomenon + a model, which together make a "destination".
- **Explainers:** [NeuroAI explainers](https://mrio.github.io/NeuroAI_explainers/), for example the *Shishi-odoshi* leaky integrate-and-fire neuron.
  - They're ported from Jupyter notebooks and linked from the skill map.
  - They're versioned: student feedback goes on a backlog, an agent improves them, and humans set the quality bar.

## Bloopers / C-plot
- A 2FA login loop mid-demo: "truly a vibe-coding moment".
- Osnabrück's website turned out to be offline over a security vulnerability.
- Two people both drove the shared sim's sliders at once.
- Mario had to leave at the hour. Gabe called it "the Mario fire hose", so we'll have him back.

## Next
- Set up OBS for real live streaming ([checklist](../docs/streaming-setup.md)).
- Better audio: the live mix plus per-person local tracks (Riverside-style).
- Mario part 2.
