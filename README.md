## Project Overview - Agent Shellton

NOTE: This is a **sample submssion** for the [Kaggle Agents Intensive Capstone project](https://www.kaggle.com/competitions/agents-intensive-course-capstone-2025/). Use this as a point of reference for structuring your submission. Avoid simply copying and reusing logic and or concepts.

NOTE: This sample submssion was inspired and lifted from the official [ADK-Samples](https://github.com/google/adk-samples/tree/main/python/agents/blog-writer) repository. Special thanks to Pier Paolo Ippolito for his contributions. 

This project contains the core logic for Agent Shellton, a multi-agent system designed to assist users in creating various types of blog posts. The agent is built using Google Agent Development Kit (ADK) and follows a modular architecture.

![Architecture](./thumbnail.png "Optional Title")

### Problem Statement

Writing blogs manually is laborious because it requires significant time investment in research, drafting, editing, and formatting each piece of content from scratch. The repetitive nature of structuring posts and maintaining consistent tone across multiple articles can quickly become mentally exhausting and drain creative energy. Manual blog writing also struggles to scale when content demands increase, forcing writers to choose between quality and quantity or invest in hiring additional staff. Automation can streamline research gathering, generate initial drafts, handle formatting consistency, and maintain publishing schedules, allowing human writers to focus their expertise on strategic direction, creative refinement, and adding unique insights that truly require human judgment.

### Solution Statement

Agents can automatically research topics by gathering information from multiple sources, synthesizing key insights, and identifying trending themes relevant to your target audience. They can generate initial draft outlines or full articles based on specific parameters like tone, length, significantly reducing the time spent on the blank page problem. Additionally, agents can manage the entire publishing workflow by scheduling posts, distributing content across multiple platforms, monitoring performance metrics, and even suggesting improvements based on engagement data—transforming blog management from a manual chore into a streamlined, data-driven process.

### Architecture
Core to Agent Shellton is the `blogger_agent` -- a prime example of a multi-agent system. It's not a monolithic application but an ecosystem of specialized agents, each contributing to a different stage of the blog creation process. This modular approach, facilitated by Google's Agent Development Kit, allows for a sophisticated and robust workflow. The central orchestrator of this system is the `interactive_blogger_agent`.

![Architecture](./flow_adk_web.png "Optional Title")

The `interactive_blogger_agent` is constructed using the `Agent` class from the Google ADK. Its definition highlights several key parameters: the `name`, the `model` it uses for its reasoning capabilities, and a detailed `description` and `instruction` set that governs its behavior. Crucially, it also defines the `sub_agents` it can delegate tasks to and the `tools` it has at its disposal.

The real power of the `blogger_agent` lies in its team of specialized sub-agents, each an expert in its domain.

**Content Strategist: `robust_blog_planner`**

This agent is responsible for creating a well-structured and comprehensive outline for the blog post. If a codebase is provided, it will intelligently incorporate sections for code snippets and technical deep dives. To ensure high-quality output, it's implemented as a `LoopAgent`, a pattern that allows for retries and validation. The `OutlineValidationChecker` ensures that the generated outline meets predefined quality standards.

**Technical Writer: `robust_blog_writer`**

Once the outline is approved, the `robust_blog_writer` takes over. This agent is an expert technical writer, capable of crafting in-depth and engaging articles for a sophisticated audience. It uses the approved outline and codebase summary to generate the blog post, with a strong emphasis on detailed explanations and illustrative code snippets. Like the planner, it's a `LoopAgent` that uses a `BlogPostValidationChecker` to ensure the quality of the written content.

**Editor: `blog_editor`**

The `blog_editor` is a professional technical editor that revises the blog post based on user feedback. This allows for an iterative and collaborative writing process, ensuring the final article meets the user's expectations.

**Social Media Marketer: `social_media_writer`**

To maximize the reach of the created content, the `social_media_writer` generates promotional posts for platforms like Twitter and LinkedIn. This agent is an expert in social media marketing, crafting engaging and platform-appropriate content to drive traffic to the blog post.

### Essential Tools and Utilities

The `blogger_agent` and its sub-agents are equipped with a variety of tools to perform their tasks effectively.

**File Saving (`save_blog_post_to_file`)**

A simple yet essential tool that allows the `interactive_blogger_agent` to export the final blog post to a Markdown file.

**Codebase Analysis (`analyze_codebase`)**

This tool is crucial for generating technically accurate and relevant content. It ingests a directory, traverses its files using `glob` and `os`, and creates a consolidated `codebase_context`. It even handles potential `UnicodeDecodeError` exceptions by attempting to read files with a different encoding, ensuring robustness.

**Validation Checkers (`OutlineValidationChecker`, `BlogPostValidationChecker`)**

These custom `BaseAgent` implementations are a key part of the system's robustness. They check for the presence and validity of the blog outline and post, respectively. If the validation fails, they do nothing, causing the `LoopAgent` to retry. If the validation succeeds, they escalate with `EventActions(escalate=True)`, which signals to the `LoopAgent` that it can proceed. This is a powerful mechanism for ensuring quality and controlling the flow of execution in a multi-agent system.

### Conclusion

The beauty of the `blogger_agent` lies in its iterative and collaborative workflow. The `interactive_blogger_agent` acts as a project manager, coordinating the efforts of its specialized team. It delegates tasks, gathers user feedback, and ensures that each stage of the content creation process is completed successfully. This multi-agent coordination, powered by the Google ADK, results in a system that is modular, reusable, and scalable.

The `blogger_agent` is a compelling demonstration of how multi-agent systems, built with powerful frameworks like Google's Agent Development Kit, can tackle complex, real-world problems. By breaking down the process of technical content creation into a series of manageable tasks and assigning them to specialized agents, it creates a workflow that is both efficient and robust.

### Value Statement

Agent Shellton reduced my blog development time by 6-8 hours per week, enabling me to produce more content at higher quality. I have also been producing blogs across new domains - as the agent drives research that I'd otherwise not be able to do given time constraints and subject matter expertise.

If I had more time I would add an additional agent to scan various sites for trending topics and use that research to inform my blog topics. This would require integrating applicable MCP servers or building custom tools. 

## Installation

This project was built against Python 3.11.3.

It is suggested you create a vitrual environment using your preferred tooling e.g. uv.

Install dependenies e.g. pip install -r requirements.txt

### Running the Agent in ADK Web mode

From the command line of the working directory execute the following command. 

```bash
adk web
```

**Run the integration test:**

```bash
python -m tests.test_agent
```

## Project Structure

The project is organized as follows:

*   `blogger_agent/`: The main Python package for the agent.
    *   `agent.py`: Defines the main `interactive_blogger_agent` and orchestrates the sub-agents.
    *   `sub_agents/`: Contains the individual sub-agents, each responsible for a specific task.
        *   `blog_planner.py`: Generates the blog post outline.
        *   `blog_writer.py`: Writes the blog post.
        *   `blog_editor.py`: Edits the blog post based on user feedback.
        *   `social_media_writer.py`: Generates social media posts.
    *   `tools.py`: Defines the custom tools used by the agents.
    *   `config.py`: Contains the configuration for the agents, such as the models to use.
*   `eval/`: Contains the evaluation framework for the agent.
*   `tests/`: Contains integration tests for the agent.



## Workflow

The `interactive_blogger_agent` follows this workflow:

1.  **Analyze Codebase (Optional):** If the user provides a directory, the agent analyzes the codebase to understand its structure and content.
2.  **Plan:** The agent delegates the task of generating a blog post outline to the `robust_blog_planner`.
3.  **Refine:** The user can provide feedback to refine the outline. The agent continues to refine the outline until it is approved by the user.
4.  **Visuals:** The agent asks the user to choose their preferred method for including visual content.
5.  **Write:** Once the user approves the outline, the agent delegates the task of writing the blog post to the `robust_blog_writer`.
6.  **Edit:** After the first draft is written, the agent presents it to the user and asks for feedback. The `blog_editor` revises the blog post based on the feedback. This process is repeated until the user is satisfied with the result.
7.  **Social Media:** After the user approves the blog post, the agent asks if they want to generate social media posts. If the user agrees, the `social_media_writer` is used.
8.  **Export:** When the user approves the final version, the agent asks for a filename and saves the blog post as a markdown file using the `save_blog_post_to_file` tool.

