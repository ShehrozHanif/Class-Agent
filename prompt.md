# Second Parameter Of AGENT CLASS

## [Google Colab](https://github.com/ShehrozHanif/Class-Agent/blob/main/readme.md "Agent Class Practical Implementation")


##  prompt: Prompt | DynamicPromptFunction | None = None

        prompt: Prompt | DynamicPromptFunction | None = None
        """A prompt object (or a function that returns a Prompt). Prompts allow you to dynamically
        configure the instructions, tools and other config for an agent outside of your code. Only
        usable with OpenAI models, using the Responses API.
        """


### 🧾 What Is a Prompt?
In the world of AI agents (like OpenAI Agents SDK), a prompt is a special instruction or message that tells the AI how to behave before it gives a response.

#### 🔤 Simple Definition:
> A **prompt** is the system message that sets the AI's role, tone, and context before it answers.

For example:

    You are a travel agent. Help the user find flights.

So the AI will act like a travel agent in all its replies.

### 🎛️ Types of Prompts in Agents SDK
You can pass the prompt parameter in three ways:

    prompt: Prompt | DynamicPromptFunction | None = None

Let’s understand each:

#### 1. 📝 Prompt
A **fixed/static** instruction.

* Use this when your agent's role never changes.

* Example:

    prompt = Prompt("You are a math tutor. Help with solving math problems.")

#### 2. ⚙️ DynamicPromptFunction
A **dynamic prompt function** that builds the prompt based on **the situation,** like user message, uploaded files, or chat history.

* Use this when you want the AI to adapt to different inputs.

* Example:

    @agent.dynamic_prompt
    def generate_prompt(data):
        if "pdf" in data.raw_input.lower():
            return Prompt("You are a document analyst. Summarize the PDF.")
        return Prompt("You are a general assistant.")
`
#### 3. 🚫 None
No special prompt is given — the AI uses a default instruction or none at all.

* **Use this when** you don’t need any custom instructions, or you’ll add them later.


### 🕐 When to Use Each?

| Type                    | When to Use                                                                                    |
| ----------------------- | ---------------------------------------------------------------------------------------------- |
| `Prompt`                | When the AI has a single, simple role (e.g., travel agent, code assistant).                    |
| `DynamicPromptFunction` | When the AI should adapt based on the user's message, uploaded files, or conversation history. |
| `None`                  | When you're experimenting, or using default agent behavior.                                    |


### ✅ Summary
> prompt: Prompt | DynamicPromptFunction | None = None means:

> You can pass either a **fixed prompt**, a **dynamic function**, or no **prompt** — depending on how flexible and smart you want your AI to be.



## Question:
### so if we set none than what happen ?

    prompt: None

> then no custom instruction is given to the AI when the agent starts. Here's what happens in simple terms:

### 🧠 What Happens If prompt=None?
* The AI agent uses a default behavior based on the task or tools you give it.

* It does not receive any special guidance like “you are a helpful assistant” or “you are a travel agent.”

* It behaves like a general-purpose assistant, trying to help based on your query and available tools.

### 📌 Example:
If you do:

    agent = Agent(
        tools=[search_tool, calculator],
        prompt=None,
    )

And the user says:

> "What's the capital of Japan?"

The AI will:

* Look at the tools (e.g. search_tool)

* See no prompt instructions

* Respond with general reasoning and use the tools if necessary


### ⚠️ When is None useful?


| ✅ Good For                            | ❌ Not Good For                                         |
| ------------------------------------- | ------------------------------------------------------ |
| Fast prototyping                      | Complex roleplay agents                                |
| Default assistant behavior            | Multi-turn memory-based agents                         |
| When prompt comes from another source | When you want strong control over tone and personality |


### 🧠 Tip:
Even if prompt=None, if you define a goal, tools, or files, the agent still works — it just won’t be “guided” in how to behave unless you provide a prompt.


## Question 2:
### 🔍 What Happens If You Set prompt=None and Don’t Provide Any Tools?


If you write:

    agent = Agent(
        tools=[],       # No tools
        prompt=None,    # No instructions
    )

Then here’s what happens:

### 🧠 Behavior of the Agent:
* The agent behaves like a plain LLM chat interface.

* It uses just the model (like GPT or Gemini) without any extra power or logic.

* It will still respond to queries like:

> “Tell me a joke”
  
>  “What is the capital of France?”
  
>  “Explain quantum physics simply”

…but it won’t be able to call any tools, run functions, access APIs, or do anything outside basic LLM abilities.

### 🔧 Summary Table

| Scenario                            | Result                                               |
| ----------------------------------- | ---------------------------------------------------- |
| `tools=[]`, `prompt=None`           | Plain LLM, no personality, no tools                  |
| `tools=[]`, `prompt=Prompt(...)`    | Guided LLM (personality or role), but still no tools |
| `tools=[...]`, `prompt=None`        | Tool-using agent, but no special personality         |
| `tools=[...]`, `prompt=Prompt(...)` | Full agent (tools + instruction) ✅                   |





<br><br><br>

# prompt: DynamicPromptFunction

👉 You must use an **OpenAI API key**, because:

* These dynamic prompt features are part of the OpenAI Agents SDK, and they only work with OpenAI models (like gpt-4o, gpt-4-turbo, etc.).

* These tools rely on OpenAI's internal orchestration, which doesn't support Gemini for GenerateDynamicPromptData.

> To use the @dynamic_prompt function or GenerateDynamicPromptData, you need an OpenAI API key — Gemini models do not support these features.


<br>

## 🧾 What Is a Dynamic Prompt Function?


A dynamic prompt function lets you customize what the AI agent thinks before it answers — depending on what the user says, sends, or does.

📘 Simple Definition
> A dynamic prompt function is a special function that creates the system message (or “instruction”) for the AI based on the    
  situation — like the current message, uploaded files, or past chat history.

Instead of writing one fixed instruction, like:

    "You are a travel agent. Help the user find flights."

You can tell the AI:

> "If the user mentions flights, act like a travel expert.

> If they ask about hotels, act like a hotel advisor.

> If they upload a document, read and use it first."


### 🧠 Why Use Dynamic Prompts?
* 🧩 Context-aware: Adjusts instructions based on the current message, files, or history.

* 🧠 Smarter behavior: Helps the AI focus only on what's relevant right now.

* 🔄 Reusable agents: One agent can act like many specialists — based on the dynamic prompt.

* 📂 File-aware: You can tell the AI to read and understand uploaded files before replying.


### 🧪 Example (Human-Friendly)
Imagine you're building an AI that helps with:

* Travel

* Resumes

* PDF reports

Instead of 3 different agents, you write one dynamic_prompt like:


    @agent.dynamic_prompt
    def generate_prompt(data):
        if "flight" in data.raw_input.lower():
            return Prompt("You are a travel agent. Help with flights.")
        elif data.files:
            return Prompt("You are a file analyst. Read the file and answer based on it.")
        else:
            return Prompt("You're a helpful assistant. Answer politely.")


Now the **same agent** behaves differently:

* If the user asks about "tickets", it gives flight info.

* If they upload a resume, it gives tips.

* If they send a PDF report, it summarizes it.


| Feature              | Description                                                             |
| -------------------- | ----------------------------------------------------------------------- |
| 🎯 **Contextual**    | Generates instructions based on user input, files, thread history, etc. |
| 🧠 **Flexible**      | Changes the agent's behavior without restarting it.                     |
| 🛠 **Tool-aware**    | Can guide the agent on which tools to prefer or avoid.                  |
| ⚡ **Efficient**      | One smart agent can replace many simple ones.                           |
| 🧵 **History-aware** | Can include past conversation in the decision.                          |



### 🧵 History-aware – What Does It Mean?
> A history-aware dynamic prompt means your AI agent can see and use past conversation messages to decide what kind of instruction it should follow right now.

### 💡 Real-World Example:
Imagine this chat:

1. **User:** "I want help writing a resume."

2. **AI:** "Sure! What job are you applying for?"

3. **User:** "Software Engineer."

Now in step 4, the AI could:

* **Check the history** (it knows you’re writing a resume for Software Engineering),

* And dynamically build this prompt:

    You are a resume expert. The user is applying for a Software Engineer job. Help improve the resume.

Instead of starting fresh every time, the AI remembers the context using run_context.thread_messages.

### 📌 How It Works in Code

    @agent.dynamic_prompt
    def generate_prompt(data):
        messages = data.thread_messages
        if "resume" in messages[-1].content:
            return Prompt("You're a resume expert. Help the user improve it.")

✅ With this, your AI behaves like it’s in a continuous conversation — just like a human!



| Feature             | **Static Instruction**     | **Dynamic Prompt (`generate_dynamic_prompt`)** |
| ------------------- | -------------------------- | ---------------------------------------------- |
| 📦 Prompt Content   | Hardcoded once             | Dynamically generated every run                |
| 🧠 React to User    | Not adaptive               | Can adjust for different users or intents      |
| 🧰 Tool Awareness   | Must be manually specified | Tools list injected in `run_context.tools`     |
| 🔁 Thread Awareness | Not easy                   | Access to `run_context.thread.messages`        |
| 📁 File Awareness   | Not available              | Can access uploaded files                      |
| ⚙️ Scalable Logic   | Hard to manage             | Central place for smart logic                  |



### 🔍 Example Use Case
#### Say: You want your agent to behave differently if:
* The user is a "student" vs "teacher"

* The message history includes a topic about "math"

* A file is uploaded containing a CSV

In a static prompt, you'd have to:

    "If the user says something about math and is a student and uploaded a CSV, then..."



That's complex, hard to maintain, and not scalable.

But in a dynamic prompt:

    def dynamic_prompt(data: GenerateDynamicPromptData):
        messages = data.run_context.thread.messages
        tools = data.run_context.tools
        files = data.run_context.thread.files

        prompt = "You are a helpful assistant.\n"

        if "student" in messages[-1].content.lower():
            prompt += "The user is a student. Be educational.\n"

        if any("math" in msg.content.lower() for msg in messages):
            prompt += "They are discussing math. Provide equations.\n"

        if files:
            prompt += "A file is uploaded. Use tools to process it.\n"

        return Prompt(content=prompt)

This gives you **true flexibility** — you're adjusting instructions **based on live context.**

### 🤔 What About the Tools Being Outside?
Yes, tools must be declared outside. But here's the trick:

When you define tools:

    @tool
    def my_tool(input: str) -> str:
        ...

That tool is passed into run_context.tools, so inside your dynamic prompt, you can do:

    tool_names = [tool.name for tool in data.run_context.tools]

And adjust the instructions accordingly:

    if "summarize_file" in tool_names:
        prompt += "You have a tool to summarize files. Use it if file is uploaded.\n"

So the **dynamic prompt becomes smart** about what tools are available at run-time.


### ✅ Summary: When to Use generate_dynamic_prompt

Use dynamic prompt when:

* Your agent's behavior depends on user input, thread history, uploaded files, or available tools

* You want to scale logic without making your instruction messy

* You want to personalize the agent per use-case or user role



### 🧠 Your Core Question:
> In a dynamic prompt function, tools are registered outside the prompt function. So what special role does GenerateDynamicPrompt 
  play, if we can already specify tools in normal agent instructions?

### ✅ What Is Special About GenerateDynamicPrompt?
The GenerateDynamicPrompt feature isn't about adding tools — it's about dynamically changing the prompt based on:

* User input

* Thread history

* Files uploaded

* Context like time, user ID, prior actions, etc.

**The tools are registered outside** because tools are part of the agent setup, not the prompt generation logic.

So what does GenerateDynamicPrompt enable?

### 🔍 Imagine a Use Case Without GenerateDynamicPrompt
If you use a static instruction, like this:

    agent = assistant_agent(
        instructions="You are a helpful travel assistant. Always use the 'flight_search' tool when asked about flights.",
        tools=[flight_search]
    )

Then:

* You always get the same system prompt.

* You can’t conditionally change the prompt if the user starts asking about hotels, uploading files, etc.

* You can’t scale to context-specific instructions.

### 💡 What GenerateDynamicPrompt Enables
Now imagine this:

    @agent.dynamic_prompt
    def generate_dynamic_prompt(data: GenerateDynamicPromptData) -> Prompt:
        if "flight" in data.raw_input.lower():
            return Prompt("You're a flight expert. Prioritize using 'flight_search'.")
        elif "hotel" in data.raw_input.lower():
            return Prompt("You're a hotel expert. Prioritize using 'hotel_lookup'.")
        elif data.files:
            return Prompt("The user uploaded a file. Read it before answering.")
        else:
            return Prompt("Be a general assistant.")

With this:
✅ You **tailor** the system instructions just-in-time
✅ You make the assistant **adapt** to what’s happening
✅ You can build **smarter agents** without retraining or rerunning tools manually

### 📌 Tools Stay Outside for Good Reason
Because:

* You don’t want to dynamically re-register tools every message (it’s expensive and confusing)

* Tools are fixed capabilities of the agent

* The dynamic prompt decides how and when to use them, not which ones exist

### 🔄 Real World Analogy
Think of tools as **skills** the assistant has:

* Cooking

* Searching

* Summarizing

The dynamic prompt is like telling the assistant:

> “Right now, pretend you're a chef because the user wants recipes.”

You're not changing the skills, you're just telling the assistant which mindset to operate in.

