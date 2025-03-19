---
theme: dracula
colorSchema: 'dark'
highlighter: shiki
lineNumbers: true
canvasWidth: 1080
src: pages/loading-demo.md
---

# What This Presentation Was About

This was a presentation about the project I'm working on

Based on the git history of the project

Using Slidev

>Slidev (slide + dev, /slaɪdɪv/) is a web-based slides maker and presenter. It's designed for developers to focus on writing content in Markdown while also having the power of HTML and Vue components to deliver pixel-perfect layouts and designs with embedded interactive demos in your presentations.

---
layout: timeline
currentDate: '2023-06-27'
---

<CommitInfo 
  title="Make MVP API run locally (#534)"
  author="Magnus Kvittingen Fresvik"
  :message="'🎉✨Added an MVP feature for gudrun and corresponding entities'"
  changes="37 files changed, 2310 insertions(+), 4 deletions(-)"
  :files="['.pre-commit-config.yaml','api/README-API.md','api/src/app.py','api/src/authentication/access_control.py','api/src/authentication/authentication.py','api/src/authentication/mock_token_generator.py','api/src/authentication/models.py','api/src/common/exception_handlers.py','api/src/common/exceptions.py','api/src/common/logger.py','api/src/common/middleware.py','api/src/common/responses.py','api/src/config.py','api/src/entities/ConverterItem.py','api/src/entities/Converters.py','api/src/entities/TotalMolarStreamItem.py','api/src/entities/WellDataInputItem.py','api/src/entities/WellItem.py','api/src/features/gudrun/gudrun_case_data/fluid_lib_cme_fluid_5fluids.txt','api/src/features/gudrun/gudrun_feature.py','api/src/features/gudrun/use_cases/get_converter_by_id.py','api/src/features/gudrun/use_cases/get_converters.py','api/src/features/gudrun/use_cases/post_convert_rate_to_moles.py','api/src/features/health_check/health_check_feature.py','api/src/features/whoami/whoami_feature.py','api/src/init.sh','api/src/pytest.ini','api/src/tests/__init__.py','api/src/tests/integration/__init__.py','api/src/tests/integration/features/health_check/health_check_feature_test.nopy','api/src/tests/integration/features/whoami/whoami_feature_test.nopy','api/src/tests/unit/__init__.py','api/temp_test_folder/temp_gudrun_example_data/api_test_data_5fluids.csv','api/temp_test_folder/temp_gudrun_example_data/gudrun_dev.magic','api/temp_test_folder/temp_gudrun_example_data/gudrun_original.magic','poetry.lock','pyproject.toml']"
  date="2023-06-27"
/>


---
layout: timeline
currentDate: '2025-03-17'
---
## A small thing

 Wanted statuses of files changed

\> Would you kindly update CommitInfo component to support status
   of files in this format:
```ts
  :files="[{'path': 'fluidmagic/eos/eos.py', 'status': 'Modified'},
  {'path': 'fluidmagic/eos/eos_tools.py', 'status': 'Deleted'},
  {'path': 'fluidmagic/eos/eos_utils.py', 'status': 'Added'}]"
```
  But it should also be backwards compatible with the simple 
  lists:
```ts
:files="['.bumpversion.cfg','README.md']
```

---
layout: timeline
currentDate: '2023-10-27'
---

<CommitInfo 
  title="feat: 🐵✨✅♻️👷New endpoints and internal model based on yaml and blob files(#880)"
  author="Magnus Kvittingen Fresvik"
  :message="'* Dependency injection CaseRunner\n\n* Added facilities model and endpoint, in startup config\n\n* update env template\n\n* change radixconfig to promotion-style\n\n* add functionality to run with process\n\n* Optional support for modelling yaml\n\n---------\n\nCo-authored-by: Boathy Murugendran [murugend@gmail.com]'"
  changes="33 files changed, 1129 insertions(+), 551 deletions(-)"
  :files="[{'path': '.github/workflows/on-push-api.yml', 'status': 'Modified'},{'path': 'api/.env-template', 'status': 'Modified'},{'path': 'api/api/app.py', 'status': 'Modified'},{'path': 'api/api/config.py', 'status': 'Modified'},{'path': 'api/api/entities/ConverterItem.py', 'status': 'Modified'},{'path': 'api/api/entities/Converters.py', 'status': 'Modified'},{'path': 'api/api/entities/FacilitiesItem.py', 'status': 'Added'},{'path': 'api/api/entities/WellDataInputItem.py', 'status': 'Modified'},{'path': 'api/api/entities/WellDataItem.py', 'status': 'Modified'},{'path': 'api/api/features/facilities/facilities_feature.py', 'status': 'Added'},{'path': 'api/api/features/facilities/use_cases/get_facilities.py', 'status': 'Added'},{'path': 'api/api/features/gudrun/case_data/dummy_local/dummy_eos.e300', 'status': 'Deleted'},{'path': 'api/api/features/gudrun/case_data/dummy_local/dummy_fluid_lib.txt', 'status': 'Deleted'},{'path': 'api/api/features/gudrun/gudrun_feature.py', 'status': 'Deleted'},{'path': 'api/api/features/gudrun/use_cases/get_converter_by_id.py', 'status': 'Deleted'},{'path': 'api/api/features/gudrun/use_cases/get_converters.py', 'status': 'Deleted'},{'path': 'api/api/features/gudrun/use_cases/post_convert_rate_to_moles.py', 'status': 'Deleted'},{'path': 'api/api/features/rate_to_moles/case_data/dum/dum_libfile.txt', 'status': 'Added'},{'path': 'api/api/features/rate_to_moles/case_data/dum/eos/wack.e300', 'status': 'Added'},{'path': 'api/api/features/rate_to_moles/case_data/dum/processes/dum_wack_process.magic', 'status': 'Added'},{'path': 'api/api/features/rate_to_moles/case_data/magic/dum_eos_collection.magic', 'status': 'Added'},{'path': 'api/api/features/rate_to_moles/case_data/magic/dum_process_collection.magic', 'status': 'Added'},{'path': 'api/api/features/rate_to_moles/rate_to_moles_feature.py', 'status': 'Added'},{'path': 'api/api/features/rate_to_moles/use_cases/post_convert_rate_to_moles.py', 'status': 'Added'},{'path': 'api/api/features/whoami/whoami_feature.py', 'status': 'Modified'},{'path': 'api/api/fm_config.py', 'status': 'Added'},{'path': 'api/poetry.lock', 'status': 'Modified'},{'path': 'api/pyproject.toml', 'status': 'Modified'},{'path': 'api/radixconfig.yaml', 'status': 'Modified'},{'path': 'api/tests/conftest.py', 'status': 'Modified'},{'path': 'api/tests/integration/features/facilities/facilities_feature_test.py', 'status': 'Added'},{'path': 'api/tests/integration/features/gudrun/gudrun_feature_test.py', 'status': 'Deleted'},{'path': 'api/tests/integration/features/rate_to_moles/rate_to_moles_feature_test.py', 'status': 'Added'},{'path': 'api/tests/startup_data_test.magic', 'status': 'Added'}]"
  date="2023-10-27"
/>

---
layout: timeline
currentDate: '2025-03-17'
---
## It just worked 
```
● I've updated the CommitInfo component to support both file
  formats and include file status indicators:

  1. Added support for both file formats:
    - Simple string arrays: `['.bumpversion.cfg','README.md']`
    - Detailed objects: `[{'path': 'fluidmagic/eos/eos.py', 'status': 'Modified'}]`
  2. Added status indicators:
    - 'Added' - Green plus icon
    - 'Deleted' - Red minus icon with strikethrough text
    - 'Modified' - Blue edit icon
    - 'Changed' - Yellow update icon
  3. Enhanced the file display:
    - Shows only the filename with optional path context
    - Visual styling for each status type (color-coded icons)
    - Strikethrough for deleted files
    - Maintains file extension syntax highlighting
  4. Made the component backward compatible:
    - Simple string arrays default to 'Modified' status
    - File paths are properly parsed to display just the filename when needed
  5. Changed the heading from "Modified files" to "Changed files" to better reflect the variety of statuses now supported.
  
  The component now provides a more GitHub-like experience with
  visual status indicators while maintaining compatibility with
  both file list formats.
```

---
layout: center
currentDate: '2025-03-17'
---

# And then unfortunately this became an AI presentation

 It was at this point that I realized this is way more interesting than whatever I'm trying to do here.

 >62 slides and I haven't even made it to v0.2

 
---
layout: default
---

## Testing it further
 *Think hard* and creatively for this next task.

  I want you to create a new component that metaphorically visualizes the operations of FluidMagic.
   
  ```python
  """FluidMagic™ is a software tool for consistent conversion of fluid streams, it includes:

  - Conversion from volumes to molar compositions.
  - Conversions from molar compositions to volumes.
  - Conversions between different EOS descriptions (moles-to-moles).
  - Conversion from moles to mass and from mass to volumes.
  """
  ```

  So something dynamic that changes states and it should support a simple user interaction.

  I then want you to make a file in /pages that implements this new component


---
layout: default
---

```
✻ Thinking…
  I'll create a new Vue component that visually represents FluidMagic's operations. This would be a
   dynamic visualization that shows the conversion between different fluid states (volumes, molar 
  compositions, mass, etc.) and includes some user interaction.

  Let's break down what I need to do:

  1. Create a new component in the components directory
  2. Design a visualization that shows conversion between different states
  3. Add user interaction to allow users to see different conversions
  4. Create a page in /pages that implements this component

  Before I start, let me explore the project structure to understand the existing Vue components 
  and their style.

  - /home/magnus/slidev/fluidmagic-retro/
    - components/
      - CommitInfo.vue
      - Counter.vue
      - ProjectTimeline.vue
    - layouts/
      - timeline.vue
    - pages/
      - imported-slides.md
    - slides.md

  **Let me check the existing components to understand the styling and component structure.**
```

---
src: /pages/fluid-magic-demo.md
---


---
layout: default
---
**It'll commit and push the changes for you too:**

```
> commit --no-gpg
● Call(Prepare commit)…
  ⎿  Bash(git diff --staged)…
  ⎿  Bash(git log -3 --oneline)…
  ⎿  Done (2 tool uses · 0.1s)

● Bash(git commit -m "Add FluidMagic visualizer component and demo slide" --no-gpg)…
  ⎿  [master 15df9c4] Add FluidMagic visualizer component and demo slide
      3 files changed, 300 insertions(+), 2 deletions(-)
      create mode 100644 components/FluidMagicVisualizer.vue
      create mode 100644 pages/fluid-magic-demo.md

● I've committed the changes with the FluidMagic visualizer component and demo slide. The commit
  includes:

  1. A new FluidMagicVisualizer.vue component that:
    - Creates an interactive fluid visualization with state transitions
    - Shows dynamic conversions between volume, molar, and mass states
    - Features animated bubbles and color changes for visual interest
  2. A new /pages/fluid-magic-demo.md slide that:
    - Introduces the FluidMagic concept
    - Incorporates the visualization component
    - Provides context and instructions for interaction
```

---
theme: default
background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: shiki
drawings:
  persist: false
transition: slide-left
title: Tools available to Claude
---

# Sonnet 3.7 + Tools

<style>
.tools-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1em;
  font-size: 0.9em;
}
.tools-table th {
  background-color:rgb(253, 163, 67);
  text-align: left;
  padding: 12px;
  font-weight: bold;
}
.tools-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
  background-color: transparent;
}
.tools-table tr:hover {
  background-color: rgba(0,0,0,0.05);
}
</style>

<table class="tools-table">
  <thead>
    <tr>
      <th>Tool</th>
      <th>Description</th>
      <th>Permission Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>AgentTool</strong></td>
      <td>Runs a sub-agent to handle complex, multi-step tasks</td>
      <td>No</td>
    </tr>
    <tr>
      <td><strong>BashTool</strong></td>
      <td>Executes shell commands in your environment</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td><strong>GlobTool</strong></td>
      <td>Finds files based on pattern matching</td>
      <td>No</td>
    </tr>
    <tr>
      <td><strong>GrepTool</strong></td>
      <td>Searches for patterns in file contents</td>
      <td>No</td>
    </tr>
    <tr>
      <td><strong>LSTool</strong></td>
      <td>Lists files and directories</td>
      <td>No</td>
    </tr>
    <tr>
      <td><strong>FileReadTool</strong></td>
      <td>Reads the contents of files</td>
      <td>No</td>
    </tr>
    <tr>
      <td><strong>FileEditTool</strong></td>
      <td>Makes targeted edits to specific files</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td><strong>FileWriteTool</strong></td>
      <td>Creates or overwrites files</td>
      <td>No</td>
    </tr>
    <tr>
      <td><strong>NotebookReadTool</strong></td>
      <td>Reads and displays Jupyter notebook contents</td>
      <td>No</td>
    </tr>
    <tr>
      <td><strong>NotebookEditTool</strong></td>
      <td>Modifies Jupyter notebook cells</td>
      <td>Yes</td>
    </tr>
  </tbody>
</table>

---
layout: center
---
### **Adjective**

*agentic*

> That behaves like an agent: able to express or expressing agency or control on one's own behalf or on the behalf of another. 

---
layout: center
---
## Developerish


```
  Before I start, let me explore the project structure to understand the existing Vue components 
  and their style.
```

```
  Let me check the existing components to understand the styling and component structure.
```
The first time I saw it make a mistake, it did not feel like an LLM hallucinating,
but a developer working

Considers what it knows it does not know.

> Liberal use of the word know, I know



---
layout: center
---

# Falling for the hype again?

It's copilot with wings, no big deal

But making copilot get the right context is painful

You've already made a mental map of the connections

Why does the stupid bot that can parse the entire codebase in a wink, not get it?

---
layout: center
---

## Using context to get it

`init`

```
## Code Style Guidelines
- Vue components use Composition API with `<script setup lang="ts">` syntax
- Component structure: script → template → style
- Use TypeScript for type safety (`lang="ts"` in script tags)
- PascalCase for component filenames (e.g., `Counter.vue`, `ProjectTimeline.vue`)
- camelCase for variables, functions, methods, and props
- Utility-first CSS with custom classes where needed
- Define props with type validation and required status
- Use `computed()` for derived state
- Prefer defensive coding with optional chaining when appropriate

## Error Handling
- Use Math.max/min for value clamping
- Handle null/undefined values with optional chaining
- Use computed properties for derived data to prevent runtime errors
```

---
layout: default
---

## Finding context
```
● Let me check the end of the slides file to see how to
  incorporate the new slide:

● Bash(tail -n 30                                    …
      /home/magnus/slidev/fluidmagic-retro/slides.md)
  ⎿  💸💸💸
     ![alt text](/image-1.png)

     I know Vue at about the same level as before

    🟠🟠🟠
     ---
     layout: timeline
     currentDate: '2025-03-19'
     ---

     ## Not yet
     `/mcp`
     Connections to MCP servers.

     `/review`
     Claude reviews a pull request

     [Docs](https://docs.anthropic.com/en/docs/agents-and-tools/c
     laude-code/overview)
```

---
layout: center
---
## Finding its limits

Wanted to test the grasp of what it was working on


\>**Think deeply** about this one, because I want you to be creative \
and come up with a new component that would be a nice addition \
to this slidev presentation. Make a new vue component and add \
it a new slide in /pages


> Claude will think more based on the words you use. For example, “think hard” will trigger more extended thinking than saying “think” alone.

---
layout: default
src: ./pages/code-metrics.md
---

---
layout: center
---

## Impressive, but not quite 100%

It manged to parse the components and this 1700 line long markdown file. And figured out the presentation had to do with **Code Evolution**

It added the new slide to the presentation (unprompted), it missed the breakpoint by one slide, where I went off the rails and switched to writing about Claude instead.

Initally it tried to do something much more fancy than the graph it ended up with. The inital idea was good, but it was one of those 80% AI solutions.

And if you look closely at it, you can see the extra finger.

But still impressive

---
layout: timeline
currentDate: '2025-03-17'
---
## Cons
```
● Read(file_path: "slides.md")…
  ⎿  File content (26011 tokens) exceeds maximum allowed tokens (20000). Please use offset and limit
      parameters to read specific portions of the file, or use the GrepTool to search for specific 
     content.
```

slides.md is currently 1600 lines long. But can parse specific parts.\
For example the final slide was added to line 1800


`Can you make the events more readable on hover`

Struggles with css and such. Which makes some sense. Can also forget about tailwind etc

It tried and failed to fix the literal edge-cases at the end of the timeline. But works surprisingly well using these relativly niche tools

💸💸💸
![alt text](/image-1.png)


---
layout: timeline
currentDate: '2025-03-17'
---
## Cons cont
No windows support at the moment 

I wouldn't have bothered to learn Vue, just to make a slideshow \
But this is how you learn new things, by making stuff \
I know Vue at about the same level as before


It still does not read thoughts

---
layout: center
---

## Look into
`/mcp`
Connections to MCP servers

`/review` 
Claude reviews a pull request

[Docs](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview)


---
src: pages/claude-code-final.md
---
