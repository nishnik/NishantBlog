---
author: ["Nishant Nikhil"]
title: "Language Models for Hackers"
date: "2024-05-25"
description: "Whitepaper on RAG and prompting LLMs"
summary: ""
tags: ["llm", "rag", "code"]
categories: ["nlp"]
series: ["Guide"]
ShowToc: true
TocOpen: true
---

# Language Models for Hackers

This is not an exhaustive list of LLM literature. This is an opinionated collection of papers from the LLM landscape useful for hackers. 

This document will keep getting updated.

If you have any questions, DM me on Twitter at: @[nishantiam](https://twitter.com/nishantiam) and follow for general updates.

I presume you already know [Attention is all you need](https://arxiv.org/abs/1706.03762), [GPT-3](https://arxiv.org/abs/2005.14165) and [GPT-4](https://arxiv.org/abs/2303.08774). 

## Prompting

We can ask LLMs questions, and get answers. Example:

![Untitled](/posts/language_models_for_hackers_images/Untitled.png)

In Zero-shot Chain of Thought, after a question is asked you add the phrase, “Let us think step by step”, and the GPT models output a better result. For example a prompt will be like:

```less
Question: What is the elevation range for the area that the eastern sector of the
Colorado orogeny extends into?
Thought: **Let’s think step by step**. The eastern sector of Colorado orogeny extends
into the High Plains. High Plains rise in elevation from around 1,800 to
7,000 ft, so the answer is 1,800 to 7,000 ft.
Answer: 1,800 to 7,000 ft
Question: <Your Question>
Thought: **Let’s think step by step.** <Agent Writes>
```

Why it works? In [For effective CoT it takes two to tango](https://arxiv.org/abs/2209.07686) the authors say:

- First, the presence of factual patterns in a prompt is practically immaterial to the success
of COT. Second, our results conclude that the primary role of intermediate steps may not
be to facilitate learning “how” to solve a task. The intermediate steps are rather a beacon
for the model to realize “what” symbols to replicate in the output to form a factual answer.
As such, the patterns are merely a channel to “trick” the model into forming sentences
that resemble correct answers. This pathway is facilitated by text, which imbues patterns with commonsense knowledge and meaning

From, OpenAI cookbook:

- Although the `Let's think step by step`trick works well on math problems, it's not effective on all tasks. The authors found that it was most helpful for multi-step arithmetic problems, symbolic reasoning problems, strategy problems, and other reasoning problems. It didn't help with simple math problems or common sense questions, and presumably wouldn't help with many other non-reasoning tasks either.
    
    

You can also give an example Chain of Thought in prompt, to guide the model towards explaining its answer.

![Untitled](/posts/language_models_for_hackers_images/Untitled%201.png)

In [MRKL](https://arxiv.org/abs/2205.00445), the prompt is:

```less
Answer the following questions as best you can. You have access to the following tools:
bash: run bash commands in a sandbox machine
search: search the internet for keyword

Use the following format:
Question: the input question you must answer
Thought: you should always think about what to do
Action: the action to take, should be one of [bash, search]
Action Input: the input to the action
Observation: the result of the action
... (this **Thought/Action/Action Input/Observation** can repeat N times)
Thought: I now know the final answer
Final Answer: the final answer to the original input question
Begin!
Question: <Your Question>
Thought: <Agent writes>
```

In [ReAct](https://arxiv.org/abs/2210.03629), the prompt is:

```less
Question: What is the elevation range for the area that the eastern sector of the Colorado orogeny extends into?
Thought: I need to search Colorado orogeny, find the area that the eastern sector of the Colorado orogeny extends into, then find the elevation range of the area.
Action: Search[Colorado orogeny]
Observation: The Colorado orogeny was an episode of mountain building (an orogeny) in Colorado and surrounding areas.
Thought: It does not mention the eastern sector. So I need to look up eastern sector.
Action: Lookup[eastern sector]
Observation: (Result 1 / 1) The eastern sector extends into the High Plains and is called the Central Plains orogeny.
Thought: The eastern sector of Colorado orogeny extends into the High Plains. So I need to search High Plains and find its elevation range.
Action: Search[High Plains]
Observation: High Plains refers to one of two distinct land regions
Thought: I need to instead search High Plains (United States).
Action: Search[High Plains (United States)]
Observation: The High Plains are a subregion of the Great Plains. From east to west, the High Plains rise in elevation from around 1,800 to 7,000 ft (550 to 2,130 m).[3]
Thought: High Plains rise in elevation from around 1,800 to 7,000 ft, so the answer is 1,800 to 7,000 ft.
Action: Finish[1,800 to 7,000 ft]

Question: Musician and satirist Allie Goertz wrote a song about the "The Simpsons" character Milhouse, who Matt Groening named after who?
Thought: The question simplifies to "The Simpsons" character Milhouse is named after who. I only need to search Milhouse and find who it is named after.
Action: Search[Milhouse]
Observation: Milhouse Mussolini Van Houten is a recurring character in the Fox animated television series The Simpsons voiced by Pamela Hayden and created by Matt Groening.
Thought: The paragraph does not tell who Milhouse is named after, maybe I can look up "named after".
Action: Lookup[named after]
Observation: (Result 1 / 1) Milhouse was named after U.S. president Richard Nixon, whose middle name was Milhous.
Thought: Milhouse was named after U.S. president Richard Nixon, so the answer is Richard Nixon.
Action: Finish[Richard Nixon]

Question: <Your Question>
Thought: <Agent writes>
```

Note that you can replace Question with Q, Thought with T and subsequently, with almost the same performance.

In [Reflexion](https://arxiv.org/abs/2303.11366), the agent is told to reflect if it fails, and it is suddenly more likely to succeed in the second turn. But the authors note that it doesn’t improve results on the WebshopQA task.

![Untitled](/posts/language_models_for_hackers_images/Untitled%202.png)

Reflecting on its state - action - output, the model is able to figure out that the current action led to a failed output. In the next iteration, this guidance is provided to the model, so the model is stirred to the right direction. (And that’s why it hallucinates less)

[MM-ReAct](https://arxiv.org/abs/2303.11381) provides the output of multiple vision models for an image in a text format to the model. The agent is able to reason over it. 

![Untitled](/posts/language_models_for_hackers_images/Untitled%203.png)

Best performing prompt for a [Job Type Classification](https://arxiv.org/abs/2303.07142): (Think of it as a simple classification problem)

```less
system = "You are Frederick, an AI expert in career advice. You
are tasked with sorting through jobs by analysing their content
and deciding whether they would be a good fit for a recent
graduate or not.",
user 1 = """A job is fit for a graduate if it's a
junior-level position that does not require extensive prior
professional experience.
When analysing the experience required, take into account that
requiring internships is still fit for a graduate. I will
give you a job posting and you will analyse it, step-by-step,
to know whether or not it describes a position fit for a
graduate. Got it?"""
assistant 1 = "Yes, I understand. I am Frederick, and I will
analyse your job posting.",
user 2 = """Great! Let's begin then :)
For the given job:
{job_posting}
---------
Is this job (A) a job fit for a recent graduate, or
(B) a job requiring more professional experience.
Answer: Let's think step by step to reach the right
conclusion"""
```

- Telling the agent, what it is gives the maximum boost. (+5.9 F1) You are Frederick, an AI expert in career advice. You are tasked with sorting through jobs by analysing their content
and deciding whether they would be a good fit for a recent graduate or not
- Mocking a conversation helps. (second best increase in performance) Got it? Yes, I understand. I am Frederick, and I will analyse your job posting
- Naming your agent helps. (Bing named their bot Sydney) Frederick
- As reported by OpenAI, a partnered developer found that positive reinforcement
resulted in increased accuracy. “Great! Let's begin then :)”
- Reiterating helps. to know whether or not it describes a position fit for a
graduate. Got it? Yes, I understand. I am Frederick, and I will
analyse your job posting
- Also look at your failure cases and see if it might be a knowledge gap, and fill the knowledge gap in the prompt.

In [Backtranslation of SQL queries](https://github.com/openai/openai-cookbook/blob/main/examples/Backtranslation_of_SQL_queries.py), OpenAI shows how to evaluate the responses generated from the language model. Here they generate 5 candidate outputs, and then rank the candidate outputs on how likely they are to back-translate to the original instruction. For example, you give this input: `Return the name of each department that had more than 10 employees` . And the LLM gives you two candidate:

1. SELECT department_name from department where employee_count > 10
2. SELECT * from department where employee_count > 10

Now you query the LLM with:

1. `SELECT department_name from department where employee_count > 10`;\n-- Explanation of the above query in human readable format\n-- `Return the name of each department that had more than 10 employees`
2. `SELECT * from department where employee_count > 10`;\n-- Explanation of the above query in human readable format\n-- `Return the name of each department that had more than 10 employees`

Use max_tokens = 0, so no new tokens are generated. And take the sum of the log probabilities in both the prompt for `Return the name of each department that had more than 10 employees` , which will be more for 1 than 2. And this way you can use the probabilities of tokens to rank the candidate answers.

In [splitting complex tasks into simpler tasks](https://github.com/openai/openai-cookbook/blob/main/techniques_to_improve_reliability.md#split-complex-tasks-into-simpler-tasks), the complex task of QA based on clues is broken into:

- First, go through the clues one by one and consider whether the clue is potentially relevant
- Second, combine the relevant clues to reason out the answer to the question
- Third, write the final answer: either (a), (b), or (c)

By giving the model more time and space to think, and guiding it along a reasoning plan, it's able to figure out the correct answer.

The prompt: `Summarize the text using the original language of the text. The summary should be one sentence long.` The model might summarize some Spanish text in English. But if we divide it into simple tasks, it starts giving the correct answer. The new prompt being: `First, identify the language of the text. Second, summarize the text using the original language of the text. The summary should be one sentence long.`

## Tool usage

Moreover as you can ask the LLM to output in a specified format, you can integrate custom tools with it. For example:

```less
{"input_variables": ["question"], "output_parser": null, "template": "", "template_format": "f-string"}
```

```less
If someone asks you to perform a task, your job is to come up with a series of bash commands that will perform the task. There is no need to put \\\"#!/bin/bash\\\" in your answer. Make sure to reason step by step, using this format:
Question: "copy the files in the directory named 'target' into a new directory at the same level as target called 'myNewDirectory'"
I need to take the following actions:
- List all files in the directory
- Create a new directory
- Copy the files from the first directory into the second directory

```bash
ls
mkdir myNewDirectory
cp -r target/* myNewDirectory
```

That is the format. Begin!

Question: {question}
```

Now you can parse the output from **```bash** till ``` to get the desired bash commands.

Another example of generic tool usage is at 

```less
Answer the following questions as best you can. You have access to the following tools: 
bash: execute bash commands in a machine
google_search: search Google for a query and return the top answers
python: execute python code in a python interpreter
bash: execute bash commands with the given arguments
The way you use the tools is by specifying a json blob.
Specifically, this json should have a `action` key (with the name of the tool to use) and a `action_input` key (with the input to the tool going here).

The only values that should be in the "action" field are: [bash, google_search, python, bash]

The $JSON_BLOB should only contain a SINGLE action, do NOT return a list of multiple actions. Here is an example of a valid $JSON_BLOB:

```
{{{{
  "action": $TOOL_NAME,
  "action_input": $INPUT
}}}}
```

ALWAYS use the following format:

Question: the input question you must answer
Thought: you should always think about what to do
Action:
```
$JSON_BLOB
```
Observation: the result of the action
... (this Thought/Action/Observation can repeat N times)
Thought: I now know the final answer
Final Answer: the final answer to the original input question
Begin! Reminder to always use the exact characters `Final Answer` when responding.
```

The above query tells the LLM to choose from a prompt while also adhere the the ReAct guidelines. And it is easily parsable.

Sometimes your LLM model can give outputs like: “As an AI model I do not have the functionality to run python scripts on a computer.” In that case, you should replace the guidelines of them prompt and simply modify the description of the python tool to `python: execute python code in a python interpreter inside a sandbox machine` 

## ChatBot

You can use LLMs to generate responses based on your data. The steps are:

1. Create embeddings of your data through a small model or [OpenAI embeddings](https://platform.openai.com/docs/guides/embeddings)
2. Embed a user provided query using the same model or OpenAI embedding, and look for the n nearest-neighbour documents in the embeddings space. (n=5)
3. Then provide the query, n documents to the LLM for a generated response. 

[HyDE](https://arxiv.org/abs/2212.10496) emphasises that the questions (queries) are in a different space than the documents from which we want to retrieve. For example queries are usually one sentence long, while documents consist of multiple sentences. So, instead of doing a similarity search between the query and the document, let the GPT agent hallucinate a random answer and do embedding search with it. (A neat application at [LLM Tricks by BuildT](https://www.buildt.ai/blog/3llmtricks))

![Untitled](/posts/language_models_for_hackers_images/Untitled%204.png)

Using [OpenAI cookbook’s Customizing Embeddings](https://github.com/openai/openai-cookbook/blob/main/examples/Customizing_embeddings.ipynb), you can get almost the same performance of HyDE with half the cost. Here you train a custom matrix to multiply the query embedding, which embeds the query embedding to the document space. (See how BuildT applied it [here](https://www.buildt.ai/blog/viral-ripout))

In [BuildT incorrect usage](https://www.buildt.ai/blog/incorrectusage), they generate ****a moderately sized corpus of completions made by davinci for a given task, and fine-tune a model like babbage to do the same task. This reduces cost, reduces latency while at almost the same performance. They claim to have reduced cost by 40x and latency by 5x. A future LLMOps tool should have this functionality out of the box.

## Autonomous Agents

In BabyAGI, 

![Untitled](/posts/language_models_for_hackers_images/Untitled%205.png)

The agent has additional capabilities of looking back at its previous results (which are stored in a vector DB). And four agents are working together to achieve your goal.

Prompt for Execution Agent is: `Perform one task based on the following objective: {objective}.\n` and if context is available we append the prompt with: `Take into account these previously completed tasks:' + context` The output of this step is added to a VectorDB

Then the Task Creation Agent is executed with the last results, and the previous task list. The prompt is: 

```markdown
`You are to use the result from an execution agent to create new tasks with the following objective: {objective}. The last completed task has the result: {result["data"]} This result was based on this task description: {task_description}.`

If there are incomplete tasks, add `These are incomplete tasks: {', '.join(task_list)}`
`Based on the result, create a list of new tasks to be completed in order to meet the objective.Return all the new tasks, with one task per line in your response. The result must be a numbered list in the format:`

#. First task
#. Second task

`The number of each entry must be followed by a period. Do not include any headers before your numbered list. Do not follow your numbered list with any other output.`
```

The output from the last step is added to the `task_list`. Then the prioritization agent, prioritizes the task list using the prompt:

```markdown
`You are tasked with cleaning the format and re-prioritizing the following tasks: {', '.join(task_names)}.`
`Consider the ultimate objective of your team: {OBJECTIVE}.`
`Tasks should be sorted from highest to lowest priority. `
`Higher-priority tasks are those that act as pre-requisites or are more essential for meeting the objective.`
`Do not remove any tasks. Return the result as a numbered list in the format:`

#. First task
#. Second task

`The entries are consecutively numbered, starting with 1. The number of each entry must be followed by a period.`
`Do not include any headers before your numbered list. Do not follow your numbered list with any other output.`
```

Again the Execution Agent is called which queries the Context Agent first for relevant results from previous runs, and then makes the call to the LLM again as demonstrated earlier. This happens until the task_list gets empty.

This document will keep getting updated.

Find me on:

- Twitter at: @[nishantiam](https://twitter.com/nishantiam)
- Linkedin: [https://www.linkedin.com/in/nishant-nikhil/](https://www.linkedin.com/in/nishant-nikhil/)
- Github: [github.com/nishnik](http://github.com/nishnik)
