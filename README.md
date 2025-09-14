# CrewAI-Agent
## Installation Guide
1) **Install uv**
- **On macOS/Linux:** Use `curl` to download the script and execute it with `sh`:
   ```
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```
If your system doesn’t have `curl`, you can use `wget`:
   ```
   wget -qO- https://astral.sh/uv/install.sh | sh
   ```
- **On Windows:** Use irm to download the script and iex to execute it:
   ```
   powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
   ```

2) Install CrewAI 🚀
   Run the following command to install `crewai` CLI:
   ```
   uv tool install crewai
   ```
- To verify that `crewai` is installed, run:
  ```
  uv tool list
  ```

---

## Creating the Project
**1) Create your crew**
Create a new crew project by running the following command in your terminal. This will create a new directory called `crew-agent` with the basic structure for your crew.
```
crewai create crew crew-agent
```
Upon running this command, you'll get the following output:
```
Creating folder crew_agent...
Select a provider to set up:
1. openai
2. anthropic
3. gemini
4. nvidia_nim
5. groq
6. huggingface
7. ollama
8. watson
9. bedrock
10. azure
11. cerebras
12. sambanova
13. other
q. Quit
Enter the number of your choice or 'q' to quit: <select-the-gemini-provider>

```
After selecting the option with gemini:
```
Select a model to use for Gemini:
1. gemini/gemini-1.5-flash
2. gemini/gemini-1.5-pro
3. gemini/gemini-2.0-flash-lite-001
4. gemini/gemini-2.0-flash-001
5. gemini/gemini-2.0-flash-thinking-exp-01-21
6. gemini/gemini-2.5-flash-preview-04-17
7. gemini/gemini-2.5-pro-exp-03-25
8. gemini/gemini-gemma-2-9b-it
9. gemini/gemini-gemma-2-27b-it
10. gemini/gemma-3-1b-it
11. gemini/gemma-3-4b-it
12. gemini/gemma-3-12b-it
13. gemini/gemma-3-27b-it
q. Quit
Enter the number of your choice or 'q' to quit: 1
Enter your GEMINI API key from https://ai.dev/apikey (press Enter to skip):
```

You can generate an API key for the agent from [Gemini's API Dashboard](https://aistudio.google.com/ "Gemini-API-Dashboard")

**2) Navigate to your new crew project**
```
cd latest-ai-development
```

**3) Modify your `agents.yaml` file**
```
# src/latest_ai_development/config/agents.yaml
researcher:
  role: >
    {topic} Senior Data Researcher
  goal: >
    Uncover cutting-edge developments in {topic}
  backstory: >
    You're a seasoned researcher with a knack for uncovering the latest
    developments in {topic}. Known for your ability to find the most relevant
    information and present it in a clear and concise manner.

reporting_analyst:
  role: >
    {topic} Reporting Analyst
  goal: >
    Create detailed reports based on {topic} data analysis and research findings
  backstory: >
    You're a meticulous analyst with a keen eye for detail. You're known for
    your ability to turn complex data into clear and concise reports, making
    it easy for others to understand and act on the information you provide.
```
**4) Modify your `tasks.yaml` file**
```
# src/latest_ai_development/config/tasks.yaml
research_task:
  description: >
    Conduct a thorough research about {topic}
    Make sure you find any interesting and relevant information given
    the current year is 2025.
  expected_output: >
    A list with 10 bullet points of the most relevant information about {topic}
  agent: researcher

reporting_task:
  description: >
    Review the context you got and expand each topic into a full section for a report.
    Make sure the report is detailed and contains any and all relevant information.
  expected_output: >
    A fully fledge reports with the mains topics, each with a full section of information.
    Formatted as markdown without '```'
  agent: reporting_analyst
  output_file: report.md
```
**5) Modify your `crew.py` file**
```
# src/latest_ai_development/crew.py
from crewai import Agent, Crew, Process, Task
from crewai.project import CrewBase, agent, crew, task
from crewai_tools import SerperDevTool
from crewai.agents.agent_builder.base_agent import BaseAgent
from typing import List

@CrewBase
class LatestAiDevelopmentCrew():
  """LatestAiDevelopment crew"""

  agents: List[BaseAgent]
  tasks: List[Task]

  @agent
  def researcher(self) -> Agent:
    return Agent(
      config=self.agents_config['researcher'], # type: ignore[index]
      verbose=True,
      tools=[SerperDevTool()]
    )

  @agent
  def reporting_analyst(self) -> Agent:
    return Agent(
      config=self.agents_config['reporting_analyst'], # type: ignore[index]
      verbose=True
    )

  @task
  def research_task(self) -> Task:
    return Task(
      config=self.tasks_config['research_task'], # type: ignore[index]
    )

  @task
  def reporting_task(self) -> Task:
    return Task(
      config=self.tasks_config['reporting_task'], # type: ignore[index]
      output_file='output/report.md' # This is the file that will be contain the final report.
    )

  @crew
  def crew(self) -> Crew:
    """Creates the LatestAiDevelopment crew"""
    return Crew(
      agents=self.agents, # Automatically created by the @agent decorator
      tasks=self.tasks, # Automatically created by the @task decorator
      process=Process.sequential,
      verbose=True,
    )
```
**6) [Optional] Add before and after crew functions**
```
# src/latest_ai_development/crew.py
from crewai import Agent, Crew, Process, Task
from crewai.project import CrewBase, agent, crew, task, before_kickoff, after_kickoff
from crewai_tools import SerperDevTool

@CrewBase
class LatestAiDevelopmentCrew():
  """LatestAiDevelopment crew"""

  @before_kickoff
  def before_kickoff_function(self, inputs):
    print(f"Before kickoff function with inputs: {inputs}")
    return inputs # You can return the inputs or modify them as needed

  @after_kickoff
  def after_kickoff_function(self, result):
    print(f"After kickoff function with result: {result}")
    return result # You can return the result or modify it as needed

  # ... remaining code
```
**7) Set your environment variables**
Before running your crew, make sure you have the following keys set as environment variables in your .env file:
- A [Serper.dev](https://serper.dev/ "Serper.dev") API key: SERPER_API_KEY=YOUR_KEY_HERE
- The configuration for your choice of model, such as an API key. See the LLM setup guide to learn how to configure models from any provider.

**8) Lock and install the dependencies**

- Lock the dependencies and install them by using the CLI command:
  ```
  crewai install
  ```
**9) Run your crew**
- To run your crew, execute the following command in the root of your project:
  ```
  crewai run
  ```
