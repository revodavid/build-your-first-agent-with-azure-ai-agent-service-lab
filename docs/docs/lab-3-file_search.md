## Introduction

Using file-based information to ground a conversation is highly effective, especially for finding details about a product or service stored in a document. The Azure AI Agent Service includes a [File Search tool](https://learn.microsoft.com/en-us/azure/ai-services/agents/how-to/tools/file-search){:target="_blank"} that enables agents to retrieve information directly from uploaded files, such as user-supplied documents or product data.

In this lab, you'll learn how to enable the File Search tool and upload the Tents Data Sheet to a vector store for the agent. Once activated, the tool allows the agent to search the file and deliver relevant responses. Documents can be uploaded to the agent for all users or linked to a specific user thread. This lab will focus on uploading a document directly to the agent.

A [vector store](https://en.wikipedia.org/wiki/Vector_database){:target="_blank"} is a database optimized for storing and searching vectors (numeric representations of text data). The File Search tool uses the vector store for [semantic search](https://en.wikipedia.org/wiki/Semantic_search){:target="_blank"} to search for relevant information in the uploaded document.

## Lab Exercise

1. Open the **datasheet/contoso-tents-datasheet.pdf** file from VS Code.
2. **Review** the file’s contents to understand the information it contains, as this will be used to ground the agent’s responses.

3. Open the `main.py`.

4. **Uncomment** the following lines by removing the **"# "** characters

    ```python
    # INSTRUCTIONS_FILE = "instructions/instructions_file_search.txt"
    # vector_store = await utilities.create_vector_store(project_client, DATA_SHEET_FILE)
    # file_search_tool = FileSearchTool(vector_store_ids=[vector_store.id])
    # toolset.add(file_search_tool)
    ```

    !!! warning
        The lines to be uncommented are not adjacent. When removing the # character, ensure you also delete the space that follows it.

5. Review the code in the `main.py` file.

    After uncommenting, your code should look like this:

    ``` python
    INSTRUCTIONS_FILE = "instructions/instructions_function_calling.txt"
    INSTRUCTIONS_FILE = "instructions/instructions_code_interpreter.txt"
    INSTRUCTIONS_FILE = "instructions/instructions_file_search.txt"
    # INSTRUCTIONS_FILE = "instructions/instructions_code_bing_grounding.txt"


    async def add_agent_tools():
        """Add tools for the agent."""

        # Add the functions tool
        toolset.add(functions)

        # Add the code interpreter tool
        code_interpreter = CodeInterpreterTool()
        toolset.add(code_interpreter)

        # Add the file search tool
        vector_store = await utilities.create_vector_store(project_client, DATA_SHEET_FILE)
        file_search_tool = FileSearchTool(vector_store_ids=[vector_store.id])
        toolset.add(file_search_tool)

        # Add the Bing grounding tool
        # bing_connection = await project_client.connections.get(connection_name=BING_CONNECTION_NAME)
        # bing_grounding = BingGroundingTool(connection_id=bing_connection.id)
        # toolset.add(bing_grounding)
    ```

### Review the Instructions

The **instructions/instructions_file_search.txt** file provides guidance on how the LLM should use File Search for grounding purposes.

### Run the Agent App

1. Review the **create_vector_store** function in the **utilities.py** file. The create_vector_store function uploads the Tents Data Sheet and saves it in a vector store. To observe the vector store creation process, set a [breakpoint](https://code.visualstudio.com/Docs/editor/debugging){:target="_blank"} in the function.
2. Press <kbd>F5</kbd> to run the app.
3. In the terminal, the app starts, and the agent app will prompt you to **Enter your query**.

### Start a Conversation with the Agent

The following conversation uses data from both the Contoso sales database and the uploaded Tents Data Sheet, so the results will vary depending on the query.

1. **Show as a table the beginner tents we sell?**

    Notice that the agent responds with relevant information from the Tents Data Sheet. Review the data sheet to understand why the agent selected those specific tents.

2. **What were the sales of beginner tents in 2024 by region?**

## Stop the Agent App

When you're done, type **exit**, or press <kbd>Shift</kbd>+<kbd>F5</kbd> to stop the agent app.
