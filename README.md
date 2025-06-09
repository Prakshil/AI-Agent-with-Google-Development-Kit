# AI Agent Project with Google Development Kit

This project demonstrates the implementation of various AI agents using the Google Development Kit (ADK). Each agent is designed to perform specific tasks, such as answering questions, fetching stock prices, maintaining state, and more.

## Project Structure
- `agent/agent.py`: Contains the implementation of all agents.
- `agent/__init__.py`: Initialization file for the module.

## Agents Overview

### 1. Basic Agent
**Description:**
A simple agent that answers questions concisely.

**Code Snippet:**
```python
basic_agent = Agent(
    name="basic_agent",
    model="gemini-1.5-flash-001",
    description="A simple agent that answers questions",
    instruction="""
    You are a helpful stock market assistant. Be concise.
    If you don't know something, just say so.
    """,
)
```

**Usage:**
Run the agent to answer general questions.

---

### 2. Basic Agent with Tool
**Description:**
An agent that fetches stock prices using the `get_stock_price` tool.

**Code Snippet:**
```python
def get_stock_price(ticker: str):
    stock = yf.Ticker(ticker)
    price = stock.info.get("currentPrice", "Price not available")
    return {"price": price, "ticker": ticker}

tool_agent = Agent(
    name="tool_agent",
    model="gemini-1.5-flash-001",
    description="A simple agent that gets stock prices",
    instruction="""
    You are a stock price assistant. Always use the get_stock_price tool.
    Include the ticker symbol in your response.
    """,
    tools=[get_stock_price],
)
```

**Usage:**
Provide a stock ticker symbol to fetch its current price.

---

### 3. Agent with State
**Description:**
An agent that remembers recent stock searches.

**Code Snippet:**
```python
def get_stock_price(ticker: str, tool_context: ToolContext):
    stock = yf.Ticker(ticker)
    price = stock.info.get("currentPrice", "Price not available")
    if "recent_searches" not in tool_context.state:
        tool_context.state["recent_searches"] = []
    recent_searches = tool_context.state["recent_searches"]
    if ticker not in recent_searches:
        recent_searches.append(ticker)
        tool_context.state["recent_searches"] = recent_searches
    return {"price": price, "ticker": ticker}

stateful_agent = Agent(
    name="stateful_agent",
    model="gemini-1.5-flash-001",
    description="An agent that remembers recent searches",
    instruction="""
    You are a stock price assistant. Use the get_stock_price tool.
    I'll remember your previous searches and can tell you about them if you ask.
    """,
    tools=[get_stock_price],
)
```

**Usage:**
Ask for stock prices and query recent searches.

---

### 4. Multi-Tool Agent
**Description:**
An agent with multiple tools for stock information.

**Code Snippet:**
```python
def get_stock_info(ticker: str):
    stock = yf.Ticker(ticker)
    company_name = stock.info.get("shortName", "Name not available")
    sector = stock.info.get("sector", "Sector not available")
    return {
        "ticker": ticker,
        "company_name": company_name,
        "sector": sector
    }

multi_tool_agent = Agent(
    name="multi_tool_agent",
    model="gemini-1.5-flash-001",
    description="An agent with multiple stock information tools",
    instruction="""
    You are a stock information assistant. You have two tools:
    - get_stock_price: For prices
    - get_stock_info: For company name and sector
    """,
    tools=[get_stock_price, get_stock_info],
)
```

**Usage:**
Use tools to fetch stock prices and company information.

---

### 5. Structured Output Agent
**Description:**
An agent that provides structured output for stock recommendations.

**Code Snippet:**
```python
class StockAnalysis(BaseModel):
    ticker: str = Field(description="Stock symbol")
    recommendation: str = Field(description="Buy or Sell recommendation")

def get_stock_data_for_prompt(ticker):
    stock = yf.Ticker(ticker)
    price = stock.info.get("currentPrice", 0)
    target_price = stock.info.get("targetMeanPrice", 0)
    return price, target_price

structured_agent = LlmAgent(
    name="structured_agent",
    model="gemini-1.5-flash-001",
    description="An agent with structured output",
    instruction="""
    You are a stock advisor. Analyze the stock ticker provided by the user.
    Return Buy or Sell recommendation in JSON format.
    If target price > current price: recommend Buy
    Otherwise: recommend Sell
    """,
    output_schema=StockAnalysis,
    output_key="stock_analysis"
)
```

**Usage:**
Provide a stock ticker to get a JSON recommendation.

---

### 6. Callback Agent
**Description:**
An agent that tracks tool usage with callbacks.

**Code Snippet:**
```python
def before_tool_callback(tool: BaseTool, args: Dict[str, Any], tool_context: ToolContext) -> Optional[Dict]:
    if "tool_usage" not in tool_context.state:
        tool_context.state["tool_usage"] = {}
    tool_usage = tool_context.state["tool_usage"]
    tool_name = tool.name
    tool_usage[tool_name] = tool_usage.get(tool_name, 0) + 1
    tool_context.state["tool_usage"] = tool_usage
    print(f"[LOG] Running tool: {tool_name}")
    return None

callback_agent = Agent(
    name="callback_agent",
    model="gemini-1.5-flash-001",
    description="An agent with callbacks",
    instruction="""
    You are a stock assistant. Use get_stock_data tool to check stock prices.
    This agent keeps track of how many times tools have been used.
    """,
    tools=[get_stock_data],
    before_tool_callback=before_tool_callback,
    after_tool_callback=after_tool_callback,
)
```

**Usage:**
Track tool usage and log callback events.

---

## Steps to Use
1. Install dependencies:
   ```bash
   pip install yfinance pydantic google-adk
   ```

2. Run the script:
   ```bash
   python agent/agent.py
   ```

3. Interact with the agents based on their descriptions and capabilities.

4. Check logs for detailed information:
   ```bash
   tail -F C:\Users\Prakshil\AppData\Local\Temp\agents_log\agent.latest.log
   ```

---

## Notes
- Ensure you have the required privileges to create symbolic links if using logging features.
- Use UTF-8 encoding for logging to avoid Unicode errors.

---

Feel free to explore and extend the functionality of these agents!
