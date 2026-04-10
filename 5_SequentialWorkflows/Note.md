Meta Description:  
Learn how to build sequential AI workflows using LangGraph with practical Python coding examples, including BMI calculation and LLM prompt chaining.

Keywords:  
LangGraph tutorial, sequential workflows, AI coding, LangChain integration

Title:  
How to Build Sequential AI Workflows with LangGraph

---

# How to Build Sequential AI Workflows with LangGraph

## Introduction to LangGraph and Sequential Workflows

In the expanding world of AI development, creating efficient workflows is crucial for managing tasks in a streamlined manner. This blog post is dedicated to walking you through building **sequential workflows** using **LangGraph**, a powerful framework for orchestrating AI processes. We will explore step-by-step practical coding examples, starting from setting up your environment to advanced prompt chaining with Large Language Models (LLMs).

### What Is LangGraph?

LangGraph is a framework designed to represent AI workflows as graphs, where each node corresponds to a task or computation, and edges define the flow between these tasks. Unlike LangChain, which specialises in LLM-related components, LangGraph allows you to build comprehensive workflows that can incorporate LangChain components for LLM interactions, making the two complementary.

### Why Sequential Workflows?

A sequential workflow is a linear chain of tasks where each step executes after the previous one completes — no branching or parallel paths. This simplicity makes sequential workflows ideal for beginners to understand core concepts of LangGraph, before moving on to more complex and branched workflows.

---

## Setting Up Your Environment for LangGraph

Before diving into coding, you need to set up a Python environment to run LangGraph with all necessary dependencies.

### Step 1: Create a Project Folder and Open VS Code

Create a folder named `langgraph_tutorials` on your desktop and open it in your preferred IDE, such as Visual Studio Code.

### Step 2: Create and Activate a Virtual Environment

Use the terminal to create a virtual environment:

$$ \texttt{python -m venv myenv} $$

Activate it using:

- On Windows:  
$$ \texttt{myenv\textbackslash Scripts\textbackslash activate} $$
- On macOS/Linux:  
$$ \texttt{source myenv/bin/activate} $$

### Step 3: Install Required Libraries

Install LangGraph, LangChain, LangChain OpenAI bindings, and `python-dotenv` for environment variable management:

$$ \texttt{pip install langgraph langchain langchain-openai python-dotenv} $$

### Step 4: Verify Installations

Create a Jupyter notebook named `0_test_installation.ipynb` and run imports to confirm successful installation:

```python
from langgraph.graph import StateGraph
from langchain.chat_models import ChatOpenAI
from dotenv import load_dotenv
```

---

## Building Your First Sequential Workflow: BMI Calculator

Now that your environment is ready, let's build a simple sequential workflow — a BMI (Body Mass Index) calculator.

### Understanding the Workflow

The workflow will take two inputs:

- Height ($h$) in metres
- Weight ($w$) in kilograms

The BMI is calculated using the formula:

$$ BMI = \frac{w}{h^2} $$

The workflow will be linear: input $\to$ calculation $\to$ output.

### Defining the State

In LangGraph, the **state** represents the data carried through the workflow. Define the state as a typed dictionary with three keys:

- $weight$: float (kilograms)
- $height$: float (metres)
- $bmi$: float (calculated BMI)

Example Python class using `TypedDict`:

```python
from typing import TypedDict

class BMIState(TypedDict):
    weight: float
    height: float
    bmi: float
```

### Creating the StateGraph

Instantiate the graph object and pass the state type:

```python
from langgraph.graph import StateGraph

graph = StateGraph(BMIState)
```

### Adding the Node and Function

A single node will perform the BMI calculation. Define a Python function named `calculate_bmi` that:

- Accepts the state as input
- Extracts weight and height
- Calculates BMI using the formula above
- Updates the state with rounded BMI (2 decimal places)
- Returns the updated state

```python
def calculate_bmi(state: BMIState) -> BMIState:
    weight = state["weight"]
    height = state["height"]
    bmi = round(weight / (height ** 2), 2)
    state["bmi"] = bmi
    return state
```

Add this node to the graph:

```python
graph.add_node("calculate_bmi", calculate_bmi)
```

### Defining Start and End Nodes

LangGraph uses dummy nodes `start` and `end` to mark workflow boundaries:

```python
from langgraph.graph import start, end

graph.add_edge(start, "calculate_bmi")
graph.add_edge("calculate_bmi", end)
```

### Compiling and Executing the Graph

Compile the graph:

```python
compiled_graph = graph.compile()
```

Invoke the graph with initial input state:

```python
initial_state = {"weight": 80, "height": 1.73, "bmi": 0.0}
final_state = compiled_graph.invoke(initial_state)
print(final_state)
```

Expected output:

```python
{'weight': 80, 'height': 1.73, 'bmi': 26.74}
```

### Visualising the Workflow

You can visually inspect the graph structure in Jupyter Notebook using additional code from LangGraph’s documentation. This helps understand the flow from `start` to `calculate_bmi` to `end`.

---

## Extending the Workflow: BMI Category Labelling

Next, add a feature to classify the calculated BMI into categories like "Underweight," "Normal," "Overweight," and "Obese."

### Modifying the State

Add a new string attribute `category` to the `BMIState`:

```python
class BMIState(TypedDict):
    weight: float
    height: float
    bmi: float
    category: str
```

### Adding a New Node for Labelling

Define a function `label_bmi` that reads the BMI from state and assigns a category:

```python
def label_bmi(state: BMIState) -> BMIState:
    bmi = state["bmi"]
    if bmi < 18.5:
        state["category"] = "Underweight"
    elif 18.5 <= bmi < 25:
        state["category"] = "Normal"
    elif 25 <= bmi < 30:
        state["category"] = "Overweight"
    else:
        state["category"] = "Obese"
    return state
```

Add this node and define edges to link it after BMI calculation:

```python
graph.add_node("label_bmi", label_bmi)
graph.add_edge("calculate_bmi", "label_bmi")
graph.add_edge("label_bmi", end)
```

Recompile and invoke the graph again to see the updated output:

```python
final_state = compiled_graph.invoke(initial_state)
print(final_state)
```

Output example:

```python
{'weight': 80, 'height': 1.73, 'bmi': 26.74, 'category': 'Overweight'}
```

---

## Building LLM-Based Workflows with LangGraph and LangChain

After mastering simple sequential workflows, the next step is integrating Large Language Models (LLMs) for more dynamic applications.

### Simple LLM QA Workflow

Create a workflow with a node that sends a question to an LLM and stores the answer in the state.

#### Defining State

```python
class LLMState(TypedDict):
    question: str
    answer: str
```

#### LLM Node Function

```python
from langchain.chat_models import ChatOpenAI

chat_model = ChatOpenAI()

def llm_qa(state: LLMState) -> LLMState:
    prompt = f"Answer the following question: {state['question']}"
    response = chat_model.invoke(prompt)
    state["answer"] = response.content
    return state
```

#### Building the Graph

```python
graph = StateGraph(LLMState)
graph.add_node("llm_qa", llm_qa)
graph.add_edge(start, "llm_qa")
graph.add_edge("llm_qa", end)
compiled_graph = graph.compile()

initial_state = {"question": "How far is the moon from the Earth?", "answer": ""}
final_state = compiled_graph.invoke(initial_state)
print(final_state["answer"])
```

### Benefits of LangGraph’s State Model

Unlike basic LangChain chains, LangGraph maintains evolving state throughout the workflow, allowing access to all intermediate outputs (e.g., question, outline, final answer) — a powerful feature for complex workflows.

---

## Advanced Workflow Example: Prompt Chaining for Blog Generation

Prompt chaining involves multiple sequential LLM calls where each node’s output feeds the next.

### Workflow Design

1. Input: Topic string.
2. Node 1: Generate a detailed blog outline based on the topic.
3. Node 2: Generate a full blog post using the topic and outline.
4. Output: Final blog content.

### Defining State

```python
class BlogState(TypedDict):
    topic: str
    outline: str
    content: str
```

### Node Functions

```python
def create_outline(state: BlogState) -> BlogState:
    prompt = f"Generate a detailed outline for a blog on the topic: {state['topic']}"
    outline = chat_model.invoke(prompt).content
    state["outline"] = outline
    return state

def create_blog(state: BlogState) -> BlogState:
    prompt = f"Write a detailed blog on '{state['topic']}' using the following outline:\n{state['outline']}"
    content = chat_model.invoke(prompt).content
    state["content"] = content
    return state
```

### Graph Construction

```python
graph = StateGraph(BlogState)
graph.add_node("create_outline", create_outline)
graph.add_node("create_blog", create_blog)
graph.add_edge(start, "create_outline")
graph.add_edge("create_outline", "create_blog")
graph.add_edge("create_blog", end)
compiled_graph = graph.compile()

initial_state = {"topic": "Rise of AI in India", "outline": "", "content": ""}
final_state = compiled_graph.invoke(initial_state)
print(final_state["outline"])
print(final_state["content"])
```

---

## Conclusion and Next Steps

In this comprehensive guide, we covered:

- Setting up LangGraph and its integration with LangChain.
- Building simple sequential workflows like a BMI calculator.
- Extending workflows with conditional nodes for BMI classification.
- Creating LLM-based workflows for question-answering.
- Implementing multi-step prompt chaining for blog generation using LangGraph.

LangGraph’s stateful graph-based approach offers a versatile and powerful way to design AI workflows that maintain context and evolve data through multiple steps. While sequential workflows demonstrate the basics, LangGraph truly shines when managing complex, non-linear workflows with branching and parallelism.

### Homework Challenge

Try extending the blog generation workflow by adding an **evaluation node** that rates the blog based on its outline and generates a score. This will enhance your understanding of managing state and chaining nodes in LangGraph.

---

If you found this guide helpful, consider subscribing to channels or blogs that delve deeper into AI workflow orchestration. Experimenting hands-on is the best way to master LangGraph and build sophisticated AI applications.

Happy coding!