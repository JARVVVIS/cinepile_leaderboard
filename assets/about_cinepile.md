## CinePile Data Construction

In May 2024, we launched CinePile, a long video QA dataset with about 300,000 training samples and 5,000 test samples. It's designed for question diversity, covering areas like temporal understanding, plot analysis, character dynamics, setting, and themes, pushing models to tackle diverse aspects of video comprehension. CinePile also focuses on question difficulty—humans outperform the best commercial vision models by 25% and open-source ones by 65%. Check out a [sample scene](https://www.youtube.com/watch?v=Z4DDrBjEBHE) and Q&A pairs from CinePile in the figure below.

<a name="teaser"></a> ![Sample Scene](imgs/teaser_figure.png)

CinePile stands out from earlier datasets by overcoming issues like small scale or focusing too much on simple perceptual questions (e.g., "What color is the car?"). Instead, it uses raw video from YouTube clips and detailed audio descriptions from platforms like AudioVault, which are designed for visually impaired audiences. These descriptions offer rich context beyond basic visuals, helping us create more complex questions. CinePile includes about 9,400 movie clips from various genres and eras, with both dialogue and visual descriptions (from audio transcriptions), which we call "scene-text-annotation.

<a name="og_pipeline"></a> ![Advesarial Refinement Pipeline](imgs/og_pipeline.svg)

The large size of CinePile is enabled by our novel pipeline (visualized in [Figure 2](#og_pipeline)) for automated question generation and verification using large language models. To automate question creation, we first built question templates by leveraging datasets like MovieQA and TVQA. We clustered the questions in the embedding space of a textual similarity model ([WhereIsAI/UAE-Large-V1](https://huggingface.co/WhereIsAI/UAE-Large-V1)) and then prompted GPT-4 with 10 random examples from each cluster to generate a question template and a prototypical question for each.
Since templates aren’t always relevant to every movie clip, we use Gemini to pick the best ones for each scene. Next, we feed a language model the scene’s text, selected template names (e.g., "Physical Possession"), sample questions, and a system prompt to create scene-specific questions. A well-designed prompt helps the model focus on the whole scene, generating deeper questions and avoiding superficial ones. We found that providing prototypical examples and including timestamps for the different dialogues and visual descriptions prevents GPT-4 from hallucinating and leads to more plausible multiple-choice question (MCQ) distractors. Additionally, asking the model to provide a rationale for its answers improves the quality of the questions. Using this approach, we generate about 32 questions per video.


While our process typically generates well-formed, answerable questions, some turn out to be trivial or based on basic concepts that don’t require watching the clip. To fix this, we used several large language models (LLMs) to filter or flag these issues.

1. Degeneracy: A question is "degenerate" if its answer is obvious from the question itself, like "What is the color of the pink house?" These made up a small portion of our dataset. Since manual review wasn’t feasible at our scale, we used three LLMs—Gemini, GPT-3.5, and Phi-1.5—to automate this. If all three got it right without context, the question was likely degenerate and excluded from the evaluation set.

2. Vision Reliance: Some MCQs could be answered just from dialogue, without relying on any visual information. Using the Gemini model, we checked whether it could answer using only dialogue. If it did, we scored the question as 0 for visual reliance, otherwise 1. 

3. Hardness: To measure hardness, we checked if a model could answer the question even with full context (i.e., both visual descriptions and subtitles).