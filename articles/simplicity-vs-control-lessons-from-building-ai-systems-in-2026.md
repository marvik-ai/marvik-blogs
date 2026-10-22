# Simplicity vs Control: Lessons from Building AI Systems in 2026

- Date: March 17th, 2026
- Category: Technical
- Authors: Victoria López; Nicolás Alcoba; Gonzalo Cerisola; Tomás Spoturno
- URL: https://www.marvik.ai/blog/simplicity-vs-control-lessons-from-building-ai-systems-in-2026

---

## Microsoft Co-Innovation Lab: Rapid Prototyping at the SotA of AI

Over the past year, the breakneck speed of AI advancement has forced us to accept a somewhat uncomfortable reality: the technical solution we consider optimal today will likely be deprecated within six months. At Marvik, we have been working alongside Microsoft at the Co-Innovation Lab in Uruguay since 2023, developing rapid prototypes for projects across a wide range of industries, from Agritech to Healthcare. Over time, we have moved from designing complex modular architectures to seeing how new integrated tools, models and platforms simplify solutions in the blink of an eye. This dynamic demands not only technical agility but a fundamental shift in mindset .

At the Microsoft Co-Innovation Lab, our dynamic is built on constant experimentation. Our goal is to accelerate AI product development, focusing on the critical early stages of solution modeling. This position grants us a privileged, and sometimes dizzying, perspective on how technological breakthroughs can completely change how we approach a problem.

In this blog, we analyze three use cases that we tackled at different stages over the past year. We will explore how solutions that initially required complex architectures were redesigned months later using state-of-the-art tools, models or platforms. Beyond the technical shift, we share our assessment of the balance between efficiency, control, and cost.

## Why is this important?

Documenting this evolution is not just an exercise in technical nostalgia; it is a strategic necessity for three reasons:

- Expectation Management: Understanding that the newest solution is not always the most robust for a production environment.

- Architectures with an Expiration Date: Accepting that the design we implement today, no matter how optimized, could become obsolete in months. This forces a shift in our approach: designing flexible solutions that can be replaced by more efficient tools without breaking the entire ecosystem. ‍

- Redefining Roles: Adapting our team's capabilities to a world where solution design carries more weight than writing code.

## Case Study 1: Automated Document Contrast

A pharmaceutical company needed to compare two documents for every drug product they released: a PDF with the printed label design, including layout and formatting instructions, and a Word file with the official medical information that should appear on it. Any discrepancy between the two was a regulatory risk. The process was done entirely by hand.

The challenge was not just the comparison itself. The PDF was a visual design file, not plain text, which meant the content could appear in a different order than the Word document. Before any comparison could happen, both files needed to be extracted, normalized, and made compatible.

We built a modular pipeline. Azure Document Intelligence handled the extraction of both files, converting the PDF into structured content that could be compared against the normalized Word text. We then chunked both documents semantically and indexed them with Azure AI Search, which let us retrieve only the most relevant sections before sending them to a language model for comparison. This reduced the risk of chained errors from automated chunking or PDF misreads .

The output was a structured list of discrepancies, each mapped back to its exact location in the original PDF with bounding box coordinates, so the regulatory team could see flagged sections highlighted directly on the label design. The pipeline worked well. It also required orchestrating several services and a meaningful amount of custom code.

Azure Content Understanding arrived promising to collapse much of that into a single integrated service. We tested it. It handled some cases well, but introduced errors in others that the modular pipeline caught reliably. For a regulatory context, where a missed discrepancy is not just a bug but a compliance failure, that was not a trade-off we could accept .

This case did not end with us switching to the simpler tool. It ended with us staying on the complex one, and that is the point. The promise of integrated solutions is real, but not every problem can afford to trade accuracy for convenience. When the cost of a missed error is high enough, the visibility that a modular pipeline gives you, the ability to inspect extracted text, retrieved chunks, and model outputs as separate artifacts, is not overhead. It is the product.

Content Understanding is still in preview. We will revisit it.

## Case Study 2: Voice Conversational Agents

Without a doubt, chat agents exploded in popularity during the second half of 2025. Naturally, the next step was the conversational voice agent. At the Lab, we tackled our first conversational agent case in August 2025. The client’s goal was to build an agent capable of interviewing a user over a phone call, gathering data to assess whether they were a potential credit candidate.

## The 2025 context

When we approached this case, we decided to test two different methods: the first and most innovative used Azure's STS (Speech-to-Speech) solution via the Voice Live API; the second and more classic approach used a modular pipeline of STT → Reasoning LLM → TTS using Azure Speech Services.

The first approach promised a simpler solution and faster deployment, as it only requires a few lines of code to set up a session and make a call to the API. However, as we moved forward with the Proof of Concept (PoC), we quickly realized that high latency and the agent's robotic voice led to a poor user experience, to the point where we couldn't maintain a conversation without awkward silences between interlocutors or even during the agent's speech. From a development standpoint, the black-box design of the API made it extremely difficult to track where the bottleneck was, as we did not have access to the underlying process. In our opinion, the product was not yet in a state where it could scale or be deemed reliable for production.

Because of this, we decided to return to the more traditional design, breaking the task down into its fundamental blocks, which are shown in the following diagram.

This gave us more control over latency, performance details, and overall solution costs. We were finally able to pinpoint that the high latency in the conversation wasn't caused by the transcription model which, in fact, was very accurate, but was instead rooted in the synthesis model.

Another advantage of this design is that costs are transparent and easy to project. For example, the transcription model costs $1 USD per hour of audio, while the speech synthesis model is priced at $15 USD per million synthesized characters. In a standard one-hour conversation between two people, we can calculate the costs of these specific blocks directly:

- Audio Input (Transcription): 1 hour = $1 USD

- Audio Output (Synthesis): 1 hour ≈ $0.15 USD

## The leap to January 2026

Fast forward to January 2026, we encountered another client with a similar use case: a conversational agent designed to interview customers over the phone and collect their data to later fill out a sworn declaration. This time, we faced an extra challenge: the agent needed a tool to dynamically update a client’s record as the conversation progressed. This required additional processing time between interactions, which naturally threatened to increase latency. We decided to give Azure’s real-time Voice Agents another chance and, to our surprise, the improvement in user experience was remarkable.

The agent's response speed ensured a fluid interaction, even when the tool was busy updating the data records. Additionally, the synthesis model offered a much more natural tone, with no interruptions in the agent’s dialogue.

In just a few months, Azure's Voice-Live API had managed to significantly improve the results we could achieve while maintaining its hallmark simplicity of implementation. As we can see in the following diagram, the complete solution remained a single building block.

However, this simplicity comes at the cost of visibility . This makes debugging errors or optimizing specific parts of the pipeline cumbersome if not impossible. For instance, the Voice Live API offers a limited selection of transcription models, which becomes a significant hurdle when dealing with uncommon accents or local idioms.

Another major disadvantage is that usage costs are far less transparent, making it difficult to project the cost of scaling to production. Unlike standalone transcription or synthesis models, the Voice Live API is priced per million tokens, but it does not specify the breakdown between input, output, or the model’s reasoning tokens. In essence, calculating the actual cost of the product remains a "gray area" until we reach the stage of testing with a small group of users. As a reference, the total cost for 4 hours of testing was under $5 USD. While this provides a useful baseline for the cost order, it’s important to remember that PoC pricing rarely translates linearly to production, where scaling and edge cases can quickly shift the numbers.

This case perfectly illustrates a recurring tension we face when designing a solution: the choice between the quick development of an integrated 'black box' and the granular control of a custom pipeline. While the January 2026 update proves that managed services eventually catch up in quality, it also serves as a reminder that moving faster often means sacrificing the ability to peek under the hood. As we move forward, our challenge is no longer just building the technology, but strategically deciding how much control we are willing to trade for the sake of simplicity.

## Case Study 3: Image Classification

#### February 2025 context

Image classification is a classic problem that we have faced on numerous occasions at the Lab, and we have traditionally approached it by fine-tuning vision models. One of our success stories involved a Uruguayan company dedicated to overseeing and promoting the country's meat industry. The ultimate goal was to develop a solution to classify meat cuts as either “good quality” or “bad quality”. The company provided a dataset of approximately 2,500 images of matambre (rose meat) cuts with their corresponding labels. These images were very consistent in terms of lighting, background, and quality but a significant detail was the substantial class imbalance, as “good quality” cuts represented only about 25% of the total images.

With this in mind, we decided to fine-tune a YOLO11 model, which naturally required a focused effort on image preprocessing to handle the low variability of the dataset and the class imbalance. It also involved a cost associated with the compute power needed to train the model, which in this case was an Azure Virtual Machine equipped with a GPU. We present the diagram for this solution in the following figure.

Overall, the process from the initial data work to deploying a trained model took one week and we achieved very satisfying results, reaching 95% accuracy.

#### The leap to October 2025

Several months later, we encountered a case from another Uruguayan company, this time specializing in the maintenance of fire suppression systems for professional kitchens. A key part of their maintenance process involved technicians taking photos of three specific components: the level of a green liquid agent inside a tank, a gauge position, and the tank's weight measured on an analog scale. These photos served as the official record to determine whether a kitchen was cleared for operation.

For the first component, the liquid level, the criteria was simple: the system was "fit" if the green liquid was visible through the tank’s inspection window. For the second, the gauge's needle position indicated whether the fire suppression system had been activated (rendering it "unfit"). Finally, for the third component, the weight shown on the analog scale had to fall within a specific numerical range.

The client approached the Lab looking to develop a classification model for each of these three components. They provided a dataset of over two thousand images per case, but there was a catch: they were completely unlabeled . Additionally, since the photos were taken by technicians using mobile phones inside various kitchens, the images suffered from high variability in lighting, object positioning, and overall quality.

These conditions led us to the question: Is training a vision model the best option for this case? Instead, we decided to test inference using one of the multimodal models offered by Azure. We started with the then-brand-new GPT 5.1 to classify the liquid level, assuming it would be a straightforward task. We manually labeled a small test set of 10 diverse images and, as expected, it was no challenge for the model, as we achieved 100% accuracy. We repeated the process with 10 photos of the gauges and, with a well-crafted prompt, again reached 100% accuracy. And the best part, this PoC was developed in one two-hour session. Here’s a diagram of the final solution.

The story was quite different with the scale . Even though a human could easily read the measurements with fine precision, the model was unable to provide the accuracy the problem required. This is a well-documented limitation of multimodal models, primarily due to spatial resolution constraints caused by image tokenization. No matter how refined the prompt was, this specific problem could not be solved with the same simplicity as the previous two.

This case led us to view a classic and generally "solved" problem through a new lens. While image classification using a multimodal model carries a per-inference cost that might exceed that of a custom fine-tuned model, it significantly reduces the time and effort spent on data preparation, labeling, and training and solves the generalization problem we tend to have when training vision models with low variation in samples.

However, multimodal models are not a "silver bullet" for every use case, at least not yet. The trade-off between the speed of deployment and the precision of specialized models remains a critical decision point. In scenarios with high spatial accuracy, the traditional approach of building a dedicated vision pipeline still holds its ground.

## The Opacity Trade-off

A pattern runs through all three cases. Every time we moved toward an integrated, end-to-end solution, we gained speed and lost visibility. That trade-off is worth examining on its own terms, because it affects not just how we build but how we debug, audit, and maintain.

In a modular pipeline, failure is isolable . When the voice agent built on STT → LLM → TTS produced high latency, we could trace the bottleneck to a specific block. When the document contrast pipeline missed a discrepancy, we could inspect the extracted text, the retrieved chunks, and the model's comparison output as separate artifacts. The architecture itself makes diagnosis possible.

Integrated solutions collapse that structure . With the Voice Live API, a degraded response could come from transcription quality, reasoning, synthesis, or network conditions, and the API surface gives you no way to distinguish between them. With Content Understanding applied to document contrast, a missed discrepancy leaves the same unresolvable question open: was it extraction, retrieval, or the model's comparison step?

This is not an argument against integrated solutions. The January 2026 voice agent case proved that managed services can reach production quality with genuine simplicity. The document contrast case with Content Understanding was built in hours instead of days, even in preview. As these tools continue to improve, the decision will rarely be about capability.It will be about whether the opacity that comes bundled with simplicity is acceptable for that specific problem.

## Our Take

The biggest shift in how we work is not in the tools. It is in the questions we ask before picking one.

A new model or integration is not inherently better because it is the latest release. The right architecture depends on the problem: how much a failure costs, how much visibility production requires, how quickly the solution needs to ship, and whether the simplified version has matured enough to be trusted.

What these three cases share is that none of them had a permanent answer. The pharmaceutical document pipeline stayed modular because accuracy was non-negotiable. The voice agent moved to an integrated solution because the quality had genuinely caught up. The image classifier ended up splitting the problem in two, with different approaches for different components.

Our role has shifted as a result. We are no longer primarily developers of analytical solutions. We spend more time on solution modeling and critical thinking than on writing code. The core question is no longer "how do we build this?" but "what does this problem actually need, and what is available today that can answer it?"

That shift in thinking also changes how we design. In an environment where a relevant new model or integration can appear every few months, the goal is not to build the most sophisticated solution, but the most replaceable one . That means defining clear boundaries between components, keeping business logic decoupled from specific models or providers, and treating prompts and configurations as versioned artifacts rather than embedded strings. Not because we expect to rebuild constantly, but because the cost of change should be a deliberate choice, not an accident of how something was wired together .

The decision between modular and integrated is not made once at the start of a project . It gets revisited as tools mature, as quality catches up, and as the cost of opacity becomes clearer. There is no permanent answer. Design for the solution you need now, with enough clarity in its structure that you can replace any part of it when something better arrives. It will.

‍

If you want to explore the tools and approaches we discussed, these are good starting points:

- Voice Live API for real-time voice agents

- Image classification on Azure

- Create Document Intelligence Studio custom projects
