## Introduction

### What is Function Calling

Function calling enables Large Language Models (LLMs) to interact with external systems, execute tasks, and integrate with APIs. The LLM determines when to invoke a function based on user prompts and returns structured data for application use. Developers then implement the function logic within the application.

In this workshop, the function logic is used to execute the LLM dynamically generated SQL queries against the SQLite database.

### Enabling Function Calling

If you’re familiar with [Azure OpenAI Function Calling](https://learn.microsoft.com/azure/ai-services/openai/how-to/function-calling){:target="_blank"}, it requires defining a function schema for the LLM. Azure AI Agent Service supports this approach and also offers a more flexible option.

With the Azure AI Agent Service and its Python SDK, you can define the function schema directly within the Python function’s docstring. This approach keeps the definition and implementation together, simplifying maintenance and enhancing readability.

For example, in the **sales_data.py** file, the **async_fetch_sales_data_using_sqlite_query** function uses a docstring to specify its signature, inputs, and outputs. The SDK parses this docstring to generate the callable function for the LLM:

``` python

async def async_fetch_sales_data_using_sqlite_query(self: "SalesData", sqlite_query: str) -> str:
    """
    This function is used to answer user questions about Contoso sales data by executing SQLite queries against the database.

    :param sqlite_query: The input should be a well-formed SQLite query to extract information based on the user's question. The query result will be returned as a JSON object.
    :return: Return data in JSON serializable format.
    :rtype: str
    """
```

### Dynamic SQL Generation

When the app starts, it incorporates the database schema and key data into the instructions for the Azure AI Agent Service. Using this input, the LLM generates SQLite-compatible SQL queries to respond to user requests expressed in natural language.

## Lab Exercise

In this lab, you'll enable the function logic to execute dynamic SQL queries against the SQLite database. The function will be called by the LLM to answer user questions about Contoso sales data.

1. Open the `main.py`.

1. **Uncomment** the following lines by removing the **"# "** characters

    ```python
    # INSTRUCTIONS_FILE = "instructions/instructions_function_calling.txt"
    # toolset.add(functions)
    ```

    !!! warning
        The lines to be uncommented are not adjacent. When removing the # character, ensure you also delete the space that follows it.

1. Review the Code in main.py.

    After uncommenting, your code should look like this:

    ``` python
    INSTRUCTIONS_FILE = "instructions/instructions_function_calling.txt"
    # INSTRUCTIONS_FILE = "instructions/instructions_code_interpreter.txt"
    # INSTRUCTIONS_FILE = "instructions/instructions_file_search.txt"
    # INSTRUCTIONS_FILE = "instructions/instructions_code_bing_grounding.txt"


    async def add_agent_tools():
        """Add tools for the agent."""

        # Add the functions tool
        toolset.add(functions)

        # Add the code interpreter tool
        # code_interpreter = CodeInterpreterTool()
        # toolset.add(code_interpreter)

        # Add the file search tool
        # vector_store = await utilities.create_vector_store(project_client, DATA_SHEET_FILE)
        # file_search_tool = FileSearchTool(vector_store_ids=[vector_store.id])
        # toolset.add(file_search_tool)

        # Add the Bing grounding tool
        # bing_connection = await project_client.connections.get(connection_name=BING_CONNECTION_NAME)
        # bing_grounding = BingGroundingTool(connection_id=bing_connection.id)
        # toolset.add(bing_grounding)
    ```

### Review the Instructions

Open the **instructions/instructions_function_calling.txt** file and review the **Tools** section for details on the function-calling instructions.

!!! note
    The {database_schema_string} placeholder is replaced in the instructions with the actual database schema when the application initializes.

    ```python
    # Replace the placeholder with the database schema string
    instructions = instructions.replace("{database_schema_string}", database_schema_string)
    ```

### Review the Function Logic

Examine the **async_fetch_sales_data_using_sqlite_query** function in the **sales_data.py** file.

!!! note annotate "The SDK parses the function’s docstring to generate the function definition passed to the LLM. This function is invoked by the LLM to execute dynamic SQL queries."

```python
async def async_fetch_sales_data_using_sqlite_query(self: "SalesData", sqlite_query: str) -> str:
        """
        This function is used to answer user questions about Contoso sales data by executing SQLite queries against the database.

        :param sqlite_query: The input should be a well-formed SQLite query to extract information based on the user's question. The query result will be returned as a JSON object.
        :return: Return data in JSON serializable format.
        :rtype: str
        """
```

### Run the Agent Application

1. Press <kbd>F5</kbd> to run the application.
2. In the terminal, you'll see the application start, and the agent app will prompt you to enter your query.

    ![Agent Application](./media/run-the-agent.png){:width="500"}

### Start a Conversation with the Agent

Start asking questions about Contoso sales data. For example:

1. **Help**

    Here is an example of the LLM response to the `help` query:

    ```text
    I’m here to help with your sales data inquiries at Contoso. Could you please provide more details about what you need assistance with? Here are some example queries you might consider:

    - "What were the sales by region?"
    - "What was last quarter's revenue?"
    - "Which products sell best in Europe?"
    - "Total shipping costs by region?"

    Feel free to ask any specific questions related to Contoso sales data!
    ```

    !!! info
        The LLM will provide a list of starter questions that were defined in the instructions file.
        Try asking help in your language, for example `help in Hindi` or `help in Italian`.

2. **What are the sales by region?**

    Here is an example of the LLM response to the `sales by region` query:

    ```text
    Here are the total sales by region for Contoso:

    | Region         | Total Revenue  |
    |----------------|----------------|
    | AFRICA         | $5,227,467     |
    | ASIA-PACIFIC   | $5,363,718     |
    | CHINA          | $10,540,412    |
    | EUROPE         | $9,990,708     |
    | LATIN AMERICA  | $5,386,552     |
    | MIDDLE EAST    | $5,312,519     |
    | NORTH AMERICA  | $15,986,462    |
    ```

    !!! info
        The LLM calls the `async_fetch_sales_data_using_sqlite_query` function to execute a dynamic SQL query on the SQLite database. You can see the LLM generated SQL query in the terminal output. 

        ```text
        Function Call Tools: async_fetch_sales_data_using_sqlite_query
        Executing query: SELECT region, SUM(revenue) AS total_revenue FROM sales_data GROUP BY region;
        ```

        The retrieved data is returned to the LLM, formatted as `Markdown` according to the specifications in the instruction file, and returned to the user.

### Debug the Application

Set a [breakpoint](https://code.visualstudio.com/Docs/editor/debugging){:target="_blank"} in the `async_fetch_sales_data_using_sqlite_query` function to see the LLM requesting data.

![Breakpoint](./media/breakpoint.png){:width="500"}

### Ask More Questions

Now that you’ve set a breakpoint, ask additional questions about Contoso sales data to observe the function logic in action. Step through the function to execute the database query and return the results to the LLM.

Try these questions:

1. **What countries have the highest sales?**
2. **What were the sales of tents in the United States in April 2022?**

## Stop the Agent App

When you're done, press <kbd>Shift</kbd>+<kbd>F5</kbd> or click the **Stop** button to stop the debugger.

![Stop the debugger](./media/stop-debugger.png){:width="500"}

### Disable the Breakpoint

Remember to disable the breakpoint before running the application again.