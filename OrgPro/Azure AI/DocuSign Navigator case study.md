
![[Pasted image 20260102085038.png]]

- building high quality annotated evaluation dataset

- creating representative training datasets (unlabeled documents)

- prompt-engineering on o4-mini: composing separate 'expert' prompts for each extraction:
	- Capturing extensive legal guidelines
	- Addressing common error patterns
	
- using o4-mini as "teacher" model to generate training labels for all extractions

- composing small set of prompts for GPT-4o-mini
	- joint instruction covering several extractions in a single prompt
	- Rudimentary task specification
 
 - fine tuning gpt-4o-mini on the training dataset to incorporate domain specific learning from o4-mini