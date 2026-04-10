Meta Description:  
Learn how to build parallel workflows using LangGraph and Agentic AI with practical Python examples for cricket stats & UPSC essay evaluation.

Keywords:  
parallel workflows, LangGraph, Agentic AI, Python AI examples

Title:  
Building Parallel Workflows with LangGraph and Agentic AI

---

# Building Parallel Workflows with LangGraph and Agentic AI

In the evolving landscape of artificial intelligence, building efficient workflows that handle complex tasks simultaneously is essential. This blog post explores how to create parallel workflows using LangGraph and Agentic AI. We will walk through two practical examples: a cricket statistics calculator and a UPSC essay evaluation system. Both examples are implemented in Python, demonstrating key concepts such as partial state updates, structured outputs, and reducer functions.

---

## Introduction to Parallel Workflows in AI

Parallel workflows enable multiple tasks to be executed concurrently, improving efficiency and allowing complex computations to be broken down into manageable parts. Unlike sequential workflows, where tasks depend on the completion of previous tasks, parallel workflows run independent tasks simultaneously and merge their results later.

### Recap of Previous Concepts

Previously, we covered conceptual aspects of Agentic AI and LangGraph, focusing on sequential workflows. The last session introduced a practical example of creating linear workflows using LangGraph. This post extends those learnings by demonstrating parallel workflows, including a non-LLM example and an LLM-based example for real-world applications.

---

## Example 1: Cricket Stats Parallel Workflow (Non-LLM Based)

This example builds a simple parallel workflow that calculates multiple cricket statistics based on input data points for a batsman in a single innings.

### Input Data

- Runs scored ($R$)
- Balls played ($B$)
- Number of fours ($F$)
- Number of sixes ($S$)

### Outputs to Calculate

1. **Strike Rate ($SR$):**  
   The strike rate is a measure of how quickly a batsman scores runs, calculated as:  
   $$ SR = \frac{R}{B} \times 100 $$

2. **Boundary Percentage ($BP$):**  
   The percentage of runs scored through boundaries (fours and sixes):  
   $$ BP = \frac{4F + 6S}{R} \times 100 $$

3. **Boundaries per Ball ($BPB$):**  
   The average number of balls per boundary hit:  
   $$ BPB = \frac{B}{F + S} $$

These three quantities can be calculated independently and in parallel because none depends on the output of the others.

### Workflow Structure

The workflow begins at a start node, which splits into three parallel nodes:

- Calculate Strike Rate
- Calculate Boundary Percentage
- Calculate Boundaries per Ball

After these calculations, the results merge into a summary node that compiles all outputs into a final summary and ends the workflow.

### State Definition

The workflow state maintains the following attributes:

- $R$, $B$, $F$, $S$ (input integers)
- $SR$, $BP$, $BPB$ (calculated floats)
- Summary (string summarising the results)

### Implementation Highlights

- **Partial State Updates:**  
  Each node returns only the calculated attribute(s) instead of the entire state to avoid conflicts in parallel execution. For example, the Strike Rate node returns only $\{ 'strike\_rate': SR \}$.

- **Edges and Nodes:**  
  The graph edges connect the start node to the three calculation nodes and each calculation node to the summary node, which finally connects to the end node.

- **Compilation and Execution:**  
  The graph is compiled and invoked with an initial state, for example:  
  $$ R=100, B=50, F=6, S=4 $$  
  The workflow executes all nodes in parallel and outputs the summary.

---

## Understanding Partial State Updates in Parallel Workflows

One major challenge in parallel workflows is managing state consistency. If every node returns the full state, the system cannot resolve conflicting updates to the same attributes. The solution is **partial state updates**, where each node returns only the attributes it modifies, preventing conflicts and ensuring correct merges.

---

## Example 2: UPSC Essay Evaluation Workflow (LLM Based)

This example demonstrates a more complex parallel workflow using multiple Large Language Models (LLMs) to evaluate a UPSC essay across three aspects:

- Clarity of Thought
- Depth of Analysis
- Language Quality

### Workflow Overview

- The start node receives an essay text as input.
- Three parallel evaluation nodes send the essay to different LLMs, each tasked with evaluating one aspect.
- Each LLM returns two outputs:
  - Textual feedback
  - A numeric score between 0 and 10

- Outputs from the three nodes are sent to a final evaluation node that:
  - Merges the textual feedback into a summarised overall feedback
  - Averages the numeric scores to produce a final score

### Structured Output Requirement

LLMs can produce unreliable outputs (e.g., misformatted scores). To ensure consistency, the workflow enforces **structured outputs** using a schema defined via Pydantic. This schema specifies:

- A feedback string
- A score integer between 0 and 10

The LLMs are prompted to follow this schema strictly, enabling reliable parsing and processing.

### State Definition

The state attributes include:

- Essay text (string)
- Feedback strings for each aspect (clarity, depth, language)
- Summarised overall feedback (string)
- Individual scores (list of integers)
- Average score (float)

### Reducer Function for Scores

Since the three LLMs run in parallel and produce their scores independently, their results must be merged into a single list without overwriting. A **reducer function** using Python's `operator.add` concatenates partial lists of scores into a unified list.

---

## Key Concepts and Best Practices

### 1. Managing Parallel Workflow State

- Use **partial state updates** to avoid conflicting writes.
- Return only the modified keys in node outputs.
- Merge partial outputs at converging nodes.

### 2. Structured Output from LLMs

- Define output schemas for LLM responses.
- Use schema-enforced models to ensure consistent, parsable outputs.
- This prevents issues arising from free-form text responses.

### 3. Using Reducer Functions

- Reducers aggregate outputs from parallel nodes.
- Common reducers include addition (concatenation), max, min, average.
- This is crucial when multiple nodes produce outputs destined for a single state attribute.

### 4. Graph Construction

- Define nodes clearly with their calculation or evaluation functions.
- Define edges to represent data and control flow.
- Compile the graph before execution.
- Use initial state inputs to kickstart the workflow.

---

## Practical Python Implementation Tips

- Use typed dictionaries or Pydantic models to define state structures.
- Implement node functions that accept and return dictionaries representing state fragments.
- Use LangGraph’s graph APIs to add nodes and edges.
- Handle exceptions for invalid updates, especially in parallel workflows.
- Test workflows with sample data to validate correctness.

---

## Conclusion

Building parallel workflows with LangGraph and Agentic AI unlocks powerful capabilities for AI-driven applications. Whether calculating cricket statistics or evaluating complex essays with LLMs, parallel workflows improve efficiency and modularity.

Key takeaways include the importance of partial state updates to avoid conflicts, enforcing structured outputs for reliable LLM responses, and using reducer functions to merge parallel outputs.

With these foundations, developers can design sophisticated AI workflows that scale seamlessly and handle real-world complexities elegantly.

---

## Further Learning

- Explore LangGraph documentation for advanced graph features.
- Dive deeper into Agentic AI concepts for autonomous AI agents.
- Study Pydantic for robust data validation and schema enforcement.
- Practice building sequential and parallel workflows for diverse applications.

---

If this article helped you understand parallel workflows in AI better, please share and subscribe for more deep dives into AI development with LangGraph and Agentic AI. Happy coding!