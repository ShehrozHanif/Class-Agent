[Google Colab](https://github.com/ShehrozHanif/Class-Agent/blob/main/readme.md "Agent Class Practical Implementation")

<br>

# 🔷 1. Why is Agent written as Agent[TContext]?
## 🔍 What is TContext?
In Python typing, TContext is a type variable — a placeholder for whatever context you want to pass to your agent and tools.

> 🔁 This is called generics, and it allows the SDK to work with any context structure you define.

### 💡 Analogy:
Think of TContext like a briefcase of information that your agent carries while doing its job.

### ✅ Why is it useful?
You might want to store user session info, logs, counters, or even conversation history.

That “context” is shared across:

* the agent

* the tools

* the guardrails

* and other logic.





<br><br><br>

# Dynamic Instructions

## ✅ Code Example: Personalized Answers Based on Context
* We’ll write a small agent that:

* Knows Shehroz is a VIP

* But doesn’t respond the same way to random users

### 🔹 Step 1: Setup

    !pip install -U openai-agents

### 🔹 Step 2: Imports and Context Class

    from dataclasses import dataclass
    from agents import Agent, Runner


    # Define context with username and VIP flag
    @dataclass
    class MyContext:
        user_name: str
        is_vip: bool

### 🔹 Step 3: Create Dynamic Instruction Function

    # This instruction checks the user's status
    def vip_instructions(ctx, agent):
        if ctx.context.is_vip:
            return f"{ctx.context.user_name} is a VIP user. Answer politely and offer special help."
        else:
            return f"{ctx.context.user_name} is a regular user. Answer normally."


### 🔹 Step 4: Create Agent

    agent = Agent[MyContext](
        name="VIPAssistant",
        instructions=vip_instructions,
        model="gpt-4o"
    )


### 🔹 Step 5: Test Cases

* ✅ Case A: User is Shehroz (VIP)

        context_a = MyContext(user_name="Shehroz", is_vip=True)

        result_a = await Runner.run(
            starting_agent=agent,
            input="What is my VIP status?",
            context=context_a
        )

        print("✅ Case A: VIP user (Shehroz)")
        print(result_a.output)

* ❌ Case B: User is Afnan (Not VIP)

        context_b = MyContext(user_name="Afnan", is_vip=False)

        result_b = await Runner.run(
            starting_agent=agent,
            input="What is my VIP status?",
            context=context_b
        )

        print("\n❌ Case B: Normal user (Afnan)")
        print(result_b.output)

* ❌ Case C: No Context at All

        # Using default instruction since we can’t use context in dynamic function without it
        agent_no_context = Agent(
            name="NoContextBot",
            instructions="Answer the user's question based only on what they say. Do not assume VIP status.",
            model="gpt-4o"
        )

        result_c = await Runner.run(
            starting_agent=agent_no_context,
            input="What is my VIP status?"
        )

        print("\n❌ Case C: Unknown user (No context)")
        print(result_c.output)


## 🧾 Example Outputs (Expected)


| Case | What we pass            | What agent replies                                               |
| ---- | ----------------------- | ---------------------------------------------------------------- |
| ✅ A  | Context: Shehroz, VIP   | "Yes Shehroz, you are a VIP. How can I help you today?"          |
| ❌ B  | Context: Afnan, not VIP | "Sorry Afnan, you are not marked as VIP."                        |
| ❌ C  | No context at all       | "I'm not sure who you are. Can you please verify your identity?" |


<br><br>


# Now another example

### We will now build:

| #   | Feature                  | Description                                            |
| --- | ------------------------ | ------------------------------------------------------ |
| ✅ 1 | VIP-Only Tool            | A tool that only VIPs can use (e.g. "premium support") |
| ✅ 2 | Coupon Generator         | Gives special coupon if VIP                            |
| ✅ 3 | Handoff to another agent | VIPs go to a VIP agent; others stay with default       |


    from dataclasses import dataclass
    from agents import Agent, Runner, function_tool


    @dataclass
    class MyContext:
        user_name: str
        is_vip: bool


### ✅ Feature 1: VIP-Only Tool — get_premium_support()
    @function_tool
    def get_premium_support(ctx, agent) -> str:
        if ctx.context.is_vip:
            return "✅ You have access to premium support. A human agent will contact you shortly."
        else:
            return "❌ Sorry, premium support is available only for VIPs."


### ✅ Feature 2: VIP Coupon Generator Tool — get_vip_coupon()

    @function_tool
    def get_vip_coupon(ctx, agent) -> str:
        if ctx.context.is_vip:
            return "🎁 Your VIP coupon code is: VIP-30-SHEHROZ"
        else:
            return "⚠️ You are not eligible for a VIP coupon."


### ✅ Feature 3: Handoff — VIP Agent
This agent handles only VIPs after handoff.

    vip_agent = Agent(
        name="VIPSupportAgent",
        instructions="You are a VIP support agent. Always be extra polite and thank the user for being a VIP.",
        model="gpt-4o"
    )


## ✅ Main Agent With All Features

    def master_instruction(ctx, agent):
        return (
            f"Welcome {ctx.context.user_name}!\n"
            f"Check if the user is VIP. "
            f"If they ask for support, coupon, or anything VIP-related, call tools or delegate to VIP agent.\n"
            f"Use `get_premium_support` or `get_vip_coupon`. Handoff to VIPSupportAgent if needed."
        )


    master_agent = Agent[MyContext](
        name="MasterAgent",
        instructions=master_instruction,
        model="gpt-4o",
        tools=[get_premium_support, get_vip_coupon],
        handoffs=[vip_agent]
    )



### 🧪 Test Cases
* ✅ A. VIP User – Shehroz

        context_vip = MyContext(user_name="Shehroz", is_vip=True)

        result_vip_support = await Runner.run(
            starting_agent=master_agent,
            input="Can I get premium support?",
            context=context_vip
        )

        result_vip_coupon = await Runner.run(
            starting_agent=master_agent,
            input="Give me my VIP coupon",
            context=context_vip
        )

        print("🔹 VIP – Support:\n", result_vip_support.output)
        print("🔹 VIP – Coupon:\n", result_vip_coupon.output)


* ❌ B. Normal User – Afnan

        context_non_vip = MyContext(user_name="Afnan", is_vip=False)

        result_afnan_support = await Runner.run(
            starting_agent=master_agent,
            input="I need premium support",
            context=context_non_vip
        )

        result_afnan_coupon = await Runner.run(
            starting_agent=master_agent,
            input="Do I have a VIP coupon?",
            context=context_non_vip
        )

        print("🔸 Afnan – Support:\n", result_afnan_support.output)
        print("🔸 Afnan – Coupon:\n", result_afnan_coupon.output)



### 🧾 Example Outputs

| User              | Input                        | Output                                    |
| ----------------- | ---------------------------- | ----------------------------------------- |
| ✅ Shehroz (VIP)   | "Can I get premium support?" | ✅ You have access to premium support...   |
| ✅ Shehroz         | "Give me my VIP coupon"      | 🎁 Your VIP coupon code is...             |
| ❌ Afnan (Not VIP) | "I need premium support"     | ❌ Sorry, premium support is only for VIPs |
| ❌ Afnan           | "VIP coupon?"                | ⚠️ Not eligible for VIP coupon            |

## 🧠 Final Summary


| Feature    | What we built                             |
| ---------- | ----------------------------------------- |
| 🛠️ Tool 1 | `get_premium_support()` — VIP only        |
| 🎟️ Tool 2 | `get_vip_coupon()` — coupon if VIP        |
| 🤝 Handoff | To `VIPSupportAgent` for special cases    |
| 🧠 Context | Decides user role and tool behavior       |
| 🗣️ Input  | Triggers the action (question or request) |


# Questions:


## ✅ Q1.
> In the dynamic instruction function, why don't we use @function_tool?
  Is it not required?

## ✅ Q2.
> In the dynamic function (like my_instructions(ctx, agent)), 
  We already set this function inside the Agent,
  So why do we still need to pass the agent again as a parameter?


### ✅ ANSWER 1: Why We Don’t Use @function_tool on instructions
Very simple answer:
Because @function_tool is only for tools, not instructions.

Let me explain:

🛠️ What is @function_tool?
It's a decorator to register your tool functions.

🧪 Example:


    from agents import function_tool

    @function_tool
    def get_weather(city: str) -> str:
        return f"The weather in {city} is sunny"
✅ This function is a tool — the agent can call it like "Hey, use this tool!"

❌ But instructions are not tools.
They are not “called” by the agent. They are the "brain instructions" that tell the agent how to behave.

So this is correct without any decorator:

    def my_instructions(ctx, agent):
        return f"Welcome, {ctx.context.user_name}"

✅ Don’t put @function_tool on instructions.

<br>

### ✅ ANSWER 2: Why Pass the agent Parameter into the Instruction Function?

Imagine this setup:

    agent = Agent[MyContext](
    instructions=my_instructions,
    ...
    )

So you’re telling the agent:

> "Hey, this is your system prompt. Use the my_instructions function to generate it."

Now you ask:

> "Why does that function also receive the agent itself again?"

🔍 Answer:
✅ Yes — it's the same agent that is calling the function.

> The Agent passes itself into the function as a parameter.

### 💡 Why? Because sometimes, the function might want to read agent settings dynamically.
For example:

    def my_instructions(ctx, agent):
        return f"Hello {ctx.context.user_name}, you are talking to {agent.name}"

✅ Now the instruction uses the agent's name dynamically.

Even though you already set the function in the agent,
you still pass the agent back into the function
so that the function can access the agent’s properties.


### 🧠 Analogy:
It’s like this:

* You program a robot (Agent) with a function (my_instructions)

* That robot says:

> “Hey, I need to generate my behavior — I’ll run the function you gave me.”

* But when running it, it passes itself as input — so the function can look at its own settings.

✅ That’s why agent is passed — just in case the function wants to use it.




## ✅ Q3
> Why We Write Agent[MyContext]

### Let’s break this line:

    agent = Agent[MyContext](...)


This means:

> “Hey Python, I’m telling you this agent will use the context MyContext.”

So that Python can:

* Understand the type of data in the context (user_name, etc)

* Give you smart IntelliSense help

* Prevent errors

### 🔍 Why [MyContext] in square brackets?
Because this Agent class is a generic class.

> Generics = allow code to work with any type, but stay safe.

This line:

    class Agent(Generic[TContext])
...means:

> “I can work with any context you give me — just tell me which one.”

So when you write:

    Agent[MyContext]

It means:

> “This agent is using the context MyContext, which has a field called user_name.”

✅ So now, inside your function:

    def my_instructions(ctx, agent):
        return ctx.context.user_name

Python knows ctx.context is a MyContext — so .user_name exists.



# ✅ Q4: Is Every Function a Callable?
Short Answer:
✅ Yes — every normal function you write in Python is a callable.

    def hello():
        return "hi"

    print(callable(hello))  # True

### ❌ What is not a callable?
Some examples that are not callable:

    x = 5
    print(callable(x))  # False

    y = "hello"
    print(callable(y))  # False

✅ But if you write a function using def or lambda, then it's callable.

Even classes can be made callable using __call__, but that’s advanced — you don’t need that yet.






<br><br><br>


# 🔍 What is MaybeAwaitable[str]?
Let’s break it into simple words:

✅ “MaybeAwaitable[str]” means:
> This value might be a normal string (str) or might be something you need to await (an async string).

In Python, this is just a way of saying:

    def get_instructions(...) -> str         # ✅ Sync version
    async def get_instructions(...) -> str   # ✅ Async version

Both are OK!

🧠 WHY does this matter?
Because OpenAI’s Agent SDK is flexible.

> It doesn’t force you to always write async. 
  But if you need to wait for something, like fetching from a database or API, you can use async.

✅ Real-World Analogy
🎯 Sync (str)
You're asking your assistant:

> "What's your job?"

She instantly replies:

> "I'm a shopping expert."
(She already knows → no delay)

🎯 Async (awaitable)
You're asking:

> "What should I do in Lahore?"

She first says:

> "Wait a second... let me look up the weather and events."

Then she replies:

> "There's a festival and it's sunny — perfect day for Liberty Market."

(She needed to fetch/check something before answering.)

🔧 In Code Terms:
✅ Sync Version (no async, no await)

    def instructions(ctx, agent) -> str:
        return "You are a shopping bot for Pakistan."
    This is simple. No external data needed.
    The SDK just reads the string directly.

✅ Async Version (with async and maybe await)

    async def instructions(ctx, agent) -> str:
        # maybe someday you want to query a database here
        city = ctx.context.get("city", "Pakistan")
        return f"You are a shopping bot for {city}."
    This lets you use await inside if needed:


    user = await get_user_from_db(ctx.context["user_id"])
