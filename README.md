# LLM Code generation research

This is a short research on code and unit test generation involving LLMs with the overview of the current state of things taken on Jan, 2024. The goal is to propose a concept of the product concept for software unit testing or propose research directions.

The setting is large industrial environment with big diverse codebase which can be used as knowledge base for AI to assist or automate unit test creation.

Here I want to describe what I found so far in terms of papers, articles, experiments with ChatGPT and propose the way to create valuable product.


## Table of contents
1. [Quick subjective introduction into testing](#introduction)
2. [Few examples from Code-LLM evolution](#paragraph1)
3. [GPT-4 exploration](#paragraph2)
4. [Current view on the problem from papers](#paragraph3)
5. [Conclusion: making a commercially viable product](#conclusion)
6. [References](#references)

### Quick subjective introduction into testing <a name="introduction"></a>

There are many different ways of software testing, I can propose taking [Atlassian manual article](https://www.atlassian.com/continuous-delivery/software-testing/types-of-software-testing) as a starting point. To name a few beside unit tests:

1. Integration tests.
2. Functional tests
3. End to End tests
4. Acceptance testing
5. Performance testing
6. Smoke testing
7. ...

**Integration test** can be described in the following way: For example, there is a job that builds a Docker image for deployment. For instance, a Python application - which pulls some libraries. After assembling the Docker image you run it and make a `curl` request. The essence of the test is to check whether the Docker was built correctly and whether the program was able to respond. For example, the image might pull a lot of components that need to work together, and the service might not respond because something didn't get pulled in.

**Functional test** - Say, developers send their requests to various services of the project through Postman.

**End to End testing** - or E2E testing, involves checking applications from start to finish. Its main goal is to simulate a real user scenario, for example in Selenium, or manually tested by a QA tester.

**Acceptance testing** - There is a test environment where the latest features are deployed, and there is an intermediate staging environment where the release versions of the services are deployed, on which testers run the same test collections.

**Performance testing** - self explanatory, you test the system performance under high loads of synthetic or real data.

**Smoke testing** - suppose, you have a set of commands in a Docker image's bash script that, before running the code, checks the availability of ports of the systems with which it will interact. Within the application's logic, there might be such a test that performs an initial check of the system's operability.

However, I guess **Unit testing** is the simplest and, at the same time, most complicated topic, and the one ultimately related to programming unlike other ones above. Instead of application of deployment you deal with actual functions in the code, and testing may cover either separate functions or entire pipelines - whatever you identify as a unit in your code. For example, if there are 3 functions passing data to each other, there can be 1 or 3 unit tests. If we are reading 3 different logs, then there should be 3 unit tests.

Unit testing is initiated during the project build. If something fails, the build job will fail, and the code will not be deployed. During the project build, there might be some semi-muted exceptions thrown.

From here on, I will illustrate my thoughts on a simple ETL pipeline examples like log parsing, which is pretty common for industrial applications.

Unit tests cover various problems with input and output data. How to parse invalid dates February 31st and missing commas or semicolons which make input erroneous.

A set of unit tests may include:

-   Input data, including complete garbage
-   Substituting various circumstances for a function that searches in a database or sends REST requests, etc. What if there are problems with authorization, etc.

In real-life projects, **it's impossible to have unit tests for everything**. There isn't the capability to test absolutely everything. For instance, within the scope of unit testing, it's not possible to cover all of a project's interactions with a database, etc.

For example, in the paper [CODAMOSA: Escaping Coverage Plateaus in Test Generation with Pre-trained Large Language Models](#ref1) authors introduce their solution which uses existing algorithmic software for test case discovery using, say, genetic algorithm, augmented with LLM to discover new possible test directions when automated software sees stall in coverage.

We can count that as a first example of LLM assistance in Unit testing, but such bulk of tests sending nonsense to the function might be not needed and more tailored solution might be probably more beneficial.

**In my opinion, we can create a product from code-generating LLMs if we either**:<a name="goals"></a>

1. Can rely on it almost entirely for a well defined set of tasks (**automatic solution**)
2. Can automatically spot problems with LLM discovering test cases and creating working unit test code, and involve human assistance in a way which reduces human workload compared to manual software engineering (**semi-automatic solution**)
3. Can at least involve LLM as assistant (copilot-like, side information panel or dialogue agent) which again reduces human workload compared to manual software engineering (**coding assistant solution**).

This topic is also touched in [R. Feldt, S. Kang, J. Yoon, and S. Yoo. Towards autonomous testing agents via conversational large language models](#ref2), and I will elaborate it more in the next paragraphs after my experiment with GPT4.


## Few examples from Code-LLM evolution <a name="paragraph1"></a>
Though LLM architecture and training process are definitely very important, at the moment I don't see the immediate benefit from exploration of LLM architectures and training losses. All of LLMs are not perfect at the moment in code generation, with GPT-4 reaching 94.4% precision for the first pass (no corrections) for the solution of algorithmic problems on [HumanEval benchmark](https://paperswithcode.com/sota/code-generation-on-humaneval) which consists of 164 coding problems with textual formulations. Therefore, I choose GPT-4 to test the unit test generation workflow as described in the articles and papers, and will present my results in the next section.

The LLM evolution tree is provided in [Large Language Models for Software Engineering: A Systematic Literature Review](#ref3) on page 10.
**![](https://lh7-us.googleusercontent.com/pncsLidpzPkfye-LBnZ9s-ZOias_sDvTIjACAkYRjjDDeEHRo12nH-3S8s4Zb1xJQ3tHNGG-kbs3YjR55wpqKYE33-wiXUVyYbHVRLI1xpZS_FVRugmLZJhN29zYABvbKMW9zaZgulZ12hVTWEcFH50)**

And a very brief overview of generations provided in [Cracking the Code LLMs article on Towards Data Science](https://towardsdatascience.com/cracking-the-code-llms-354505c53295). From this article:

> **Code2Vec**, 2018 - *representing code snippets into embeddings. Model tries to predict the method name from the code-snippet, by
> encoding well-named tokens and AST (Abstract Syntax Tree) paths.*
>
> **CodeBERT**, 2020 - *multimodal data pre-training, combining Natural Language and Programming Language (NL + PL) on the Transformer based
> BERT model. The model is trained on a diverse dataset comprising both
> bimodal data points pair and unimodal data points for Masked Language
> Modeling (MLM) and Replaced Token Detection (RTD) tasks.*
>
> **Codex**, 2021 - *Codex uses GPT3 architecture & tokenizer (decoder only Transformer architecture), and pre-trains on a large corpus of
> Github code. This Large Language model has 12B parameters, and was a
> state-of-art model in 2021, which showed best performance on HumanEval
> dataset by solving 28.8% of the problems at first pass. Further
> fine-tuning of the model on standalone python functions (rather than
> whole code which include configs, class implementations etc.), showed
> significant improvement, and was able to solve 37.7% of the human-eval
> dataset problem.*
>
> **CodeT5**, 2021 - *encoder-decoder model based on the T5 architecture, distinct from both CodeBERT (encoder-only) and Codex
> (decoder-only) models. It introduces a unique identifier-aware
> denoising pre-training task which helps the model distinguish and
> recover identifiers in code, enhancing its understanding of
> structure.*
>
> **PLBart**, 2021 - *denoising sequence-to-sequence modeling approach for enhanced Program and Language understanding, strategically
> combining the strengths of BERT and GPT models. This is achieved by
> combining a bidirectional encoder with an autoregressive decoder,
> allowing for a more comprehensive grasp of context and a versatile
> generation process. The model employs three denoising strategies,
> including token masking, token deletion, and token infilling.*
>
> **Code Llama**, 2023 - *scores 53% on HumanEval Dataset and 55% on MBPP dataset. -   Proposed a fine-tuning step to handle long sequences
> called Long Context Fine-Tuning, which increases context length to
> 16,384 (4x from Llama 2 context length i.e. 4096) Instruction Fine
> Tuning & Self-Instruct: One of the few models that performs
> instruction fine-tuning, which uses explicit instruction or prompts
> during the fine-tuning process. Instead of creating human feedback
> data which is expensive, authors propose a novel execution feedback
> approach to construct a self-instruction dataset.*


## GPT-4 exploration <a name="paragraph2"></a>
As long, as GPT-4 is the strongest model so far  on HumanEval benchmark, I made an experiment, loosely following the guidelines from [OpenAI cookbook](https://cookbook.openai.com/examples/unit_test_writing_using_a_multi-step_prompt_with_older_completions_api).

**The quality of code varies significantly with programming language**, so Python is taken as the most practical and simple language.

According to OpenAI, the pipeline for LLM is:

1.  *Given a Python function, we first prompt GPT-3 to explain what the function is doing.*
2.  *Second, we prompt GPT-3 to plan a set of unit tests for the function.*
    -   *If the plan is too short, we ask GPT-3 to elaborate with more ideas for unit tests.*
3.  *Finally, we prompt GPT-3 to write the unit tests.*

The benefits of the approach are:

-   *Conditional branching (e.g., only asking for elaboration if the first plan is too short)*
-   *Different models for different steps (e.g.,  `gpt-3.5-turbo-instruct`  for the text planning steps and  `gpt-4`  for the code writing step)* (**My note**: common practice for many tasks like classification is even training simple classifier or other ML model using a lot of synthetic examples from LLM and running LLM only if confidence of ML method is low).
-   *A check that re-runs the function if the output is unsatisfactory (e.g., if the output code cannot be parsed by Python's  `ast`  module)*

In my opinion, asking LLM to create summary and describe the function helps later for RAG-like mechanisms to retrieve condense information from conversations history (or summaries can be directly included into each prompt).

For testing, I took the simple ETL code written by my friend, which processes logs: https://github.com/ashd97/cucm-zoom-directions-fixer

The main function, [`parse_cdr_log`](https://github.com/ashd97/cucm-zoom-directions-fixer/blob/76da6c18203de8d1a6e1c5e521da707130c37a13/directions-fixer.py#L164), is being called in multiprocessing manner to extract records from logs which contain phone numbers from dir_of_active_phones. There core mechanism of the function is [a construction of many IFs](https://github.com/ashd97/cucm-zoom-directions-fixer/blob/76da6c18203de8d1a6e1c5e521da707130c37a13/directions-fixer.py#L210):
```
 for phone_recorder_tuple in dir_of_active_phones:
     if (fields[8] in phone_recorder_tuple or fields[30] in phone_recorder_tuple):

         # fields[8]  = "callingPartyNumber"
         # fields[29] = "originalCalledPartyNumber"
         # fields[49] = "lastRedirectDn"
         # fields[30] = "finalCalledPartyNumber"

         # In case when Genesys RP is calling (outbound), then direction will be wrong
         # finalcalledpartynumber and callingpartynumber should be swapped
         if (len(fields[8]) > 9 and len(fields[8]) < 15 and
                 fields[29] == fields[30] and
                 len(fields[30]) > 3 and len(fields[30]) < 7):
             one_cdr_record_tuple = (phone_recorder_tuple[1],
                                     fields[2],
                                     time.strftime('%Y-%m-%d %H:%M:%S',
                                                   time.localtime(int(fields[4]))),
                                     fields[55],
                                     fields[30], fields[29], fields[49], fields[8],
                                     fields[57], fields[56], fields[11], fields[33],
                                     get_releasing_party(fields[8], fields[30], fields[11], fields[33]),
                                     get_direction(fields[30], fields[8]))
             records_for_uploading.append(one_cdr_record_tuple)

         # Direct calls
         elif ((len(fields[8]) > 3 and len(fields[8]) < 7 and
                len(fields[30]) > 9 and len(fields[30]) < 15) or
               (len(fields[30]) > 3 and len(fields[30]) < 7 and
                len(fields[8]) > 9 and len(fields[8]) < 15)):
             one_cdr_record_tuple = (phone_recorder_tuple[1],
                                     fields[2],
                                     time.strftime('%Y-%m-%d %H:%M:%S',
                                                   time.localtime(int(fields[4]))),
                                     fields[55],
                                     fields[8], fields[29], fields[49], fields[30],
                                     fields[56], fields[57], fields[33], fields[11],
                                     get_releasing_party(fields[8], fields[30], fields[33], fields[11]),
                                     get_direction(fields[8], fields[30]))
             records_for_uploading.append(one_cdr_record_tuple)
```

The results can be seen there in this chat: [https://chat.openai.com/share/a90a89ab-d82e-458c-ab60-6657162882c6](https://chat.openai.com/share/a90a89ab-d82e-458c-ab60-6657162882c6).

**Network output on the prompt asking to explain the target function:**


> *The `parse_cdr_log()` function is a critical component of a larger system designed to process Call Detail Record (CDR) logs from a Cisco
> Unified Communications Manager (CUCM) system. This system aims to
> integrate call data into a separate recording system, potentially for
> compliance, quality assurance, or analytics purposes. Below is a
> detailed explanation of what `parse_cdr_log()` does, within the
> context of the entire script*:
>
> ***Function Overview***
>
> *-   **Primary Purpose**: To parse individual CDR log files, extract relevant call data based on predefined criteria, and prepare this data
> for insertion into a separate database (associated with a call
> recording system).*
> *-   ***Input Arguments***:
>     *-   `cdr_log`: The path to a CDR log file that needs to be parsed*.
>     -   *`dir_of_active_phones`: A directory (list) of phone numbers that were active on each recorder during the previous day. This list
> is used to filter the CDR log entries, ensuring that only relevant
> call records are processed.*
>     -   *`records_for_uploading`: A shared list (thanks to multiprocessing's `Manager()`) where the parsed and processed CDR
> records are stored. These records are later used for updating the
> recording system's database.*
>
> ***Key Steps in `parse_cdr_log()`***
>
> 1.  ***Opening and Reading the CDR Log**: The function starts by opening the specified CDR log file for reading.*
>
> 2.  ***Parsing Each Line**: It iterates through each line of the log file, excluding lines that match specific patterns (e.g., contain
> 'b00', 'zoom_', or 'INTEGER'), which are likely not relevant for the
> processing.*
>
> 3.  ***Extracting Data from Line**: For each relevant line, the function strips out quotation marks and splits the line into fields
> based on commas. These fields represent different aspects of a call
> record, such as calling party number, called party number, start time,
> duration, etc.*
>
> 4.  ***Filtering Based on Active Phones**: The function then filters these records based on the `dir_of_active_phones` list. It checks
> whether the calling or called party number of the record is present in
> the list of active phones.*
>
> 5.  ***Creating Record Tuples**: For records that pass the filtering step, the function constructs a tuple with selected fields from the
> record, along with additional information derived from the call data
> (like call direction and the party initiating call termination). This
> tuple is designed to represent a comprehensive view of a call for
> recording purposes.*
>
> 6.  ***Appending to `records_for_uploading`**: Each constructed tuple is appended to the `records_for_uploading` list, which is shared
> across multiple processes. This list acts as a collective pool of call
> records that have been deemed relevant for further processing or
> uploading to the recording system.*
>
>
> ***Conclusion and Context***
>
> *`parse_cdr_log()` is a part of a multi-step, multi-process workflow designed to extract, transform, and load (ETL) call data from CUCM CDR
> logs into a separate recording system. This function specifically
> handles the extraction and initial transformation of call data,
> preparing it for further processing and eventual uploading. The
> broader script includes functionalities for collecting active phone
> numbers from recorders, organizing data for uploading, and managing
> the multiprocessing workflow to efficiently process large volumes of
> CDR logs.*

**Network proposition for tests:**

> *Testing the `parse_cdr_log()` function involves crafting a series of unit tests that verify its behavior under various scenarios, focusing
> on its ability to correctly parse CDR logs, filter based on active
> phone directories, and generate accurate tuples for upload. Here's a
> plan for a comprehensive set of unit tests:*
>
> ### *1. Test Environment Setup*
>
> -   ***Mock Dependencies**: Since `parse_cdr_log()` operates on file input and interacts with global lists (like `dir_of_active_phones` and
> `records_for_uploading`), you'll need to mock these dependencies. For
> file operations, use temporary files or mock file reads. For global
> lists, create test-specific instances.*
> -   ***Sample Data Creation**: Create a set of sample CDR log lines that represent various call scenarios, including different call
> directions, parties, and outcomes. Include edge cases like calls that
> don't match active phones or have unusual characters/formatting.*
>
> ### *2. Unit Test Cases*
>
> #### *Test Case 1: Correct Parsing of Log File*
>
> -   ***Objective**: Verify that the function correctly parses a log file and extracts call data.*
> -   ***Method**: Pass a mock CDR log file with known data and verify that the function reads and parses the content correctly.*
>
> #### *Test Case 2: Filtering Based on Active Phones*
>
> -   ***Objective**: Ensure the function filters records based on the `dir_of_active_phones` list.*
> -   ***Method**: Provide a mixed list of phone numbers (some active, some not) and verify that only records with active phones are
> processed.*
>
> #### *Test Case 3: Record Tuple Creation*
>
> -   ***Objective**: Test if the function creates the correct tuple for each relevant record.*
> -   ***Method**: Check the tuples in `records_for_uploading` against expected values for each test log entry, ensuring all fields are
> correctly populated and derived values (like call direction) are
> accurate.*
>
> #### *Test Case 4: Handling of Edge Cases*
>
> -   ***Objective**: Ensure the function handles edge cases correctly, such as lines that should be ignored or records that don't match any
> active phone.*
> -   ***Method**: Include in the test log lines that match the exclusion criteria (e.g., containing 'b00', 'zoom_', 'INTEGER') and
> verify they're correctly ignored. Also, test with records not matching
> any active phone to ensure they're not processed.*
>
> #### *Test Case 5: Efficiency and Performance*
>
> -   ***Objective**: Although not traditionally part of unit testing, it's useful to assess the function's performance with large log
> files.*
> -   ***Method**: Process a large, mock CDR log file and monitor execution time and resource usage, ensuring the function scales well
> with input size.*
>
> #### *Test Case 6: Error Handling*
>
> -   ***Objective**: Verify the function's behavior in the face of errors, such as malformed log entries or IO issues.*
> -   ***Method**: Introduce errors into the mock log file (e.g., incomplete lines, inaccessible files) and assess how the function
> handles them, focusing on robustness and fault tolerance.*

However, next attempts to write test for Test case 2 were unsuccessful.

In this example, the network is:
1. Very good at summarization and general description of what function is doing
2. Describes possible test cases, sometimes acting as "captain obvious"
3. Refuses to do detailed "rubber duck" tracing of the program code
4. Generates visually good looking code using `unittest` library, but makes non-obvious and hard to debug mistakes related to mocking Python `open` function.
5. The logic with a number of IFs from the example above and positional checks for arguments like `if (fields[8] in phone_recorder_tuple or fields[30] in phone_recorder_tuple)` are too hard  for the network to generate the right mock data even with direct guidelines. **In this case, debugging GPT generated code takes the time comparable with writing such tests manually**.

In the next paragraph, I make a survey on papers and current approaches to the unit tests generation problem.

## Current view on the problem from papers <a name="paragraph3"></a>

here I want to cite a few of recent papers, creating an overview of the proposed approaches and reach conclusion based on this information and my research with GPT.

At first, in my opinion, beside the actual code generation, the task of LLM-based unit testing is closely related to

1. LLM-based Bug Reproduction problem
2. LLM debugging

Which are organically included into the testing problem.

Also, the paper [The Program Testing Ability of Large Language Models for Code](#ref8) provides some metrics which can be used, but the most important point as for me is a problem definition from peer reviewer:

> A piece of code designed to solve a certain problem may have some implementation errors. Given such code, we would like to:
> -   generate inputs that cover most of the paths in the program (high coverage)
> -   find the “true” outputs for these inputs that test the  _intended_  functionality of the code. Note that this is not the same as simply
> running each input through the program under test. The program may have errors, and the model has to “see through” the implementation to find the intended functionality. This is called the oracle problem.

> Both these problems have been extensively studied in the software engineering community. The first problem is typically solved with a combination of symbolic execution and evolutionary fuzzing. The second problem (the oracle problem) is much more challenging, and is perhaps an excellent domain to apply language models. The “intended” functionality of code is hard to infer by traditional analysis, but language models can a) use comments/documentation, and b) use semantic properties of the code itself. This paper attempts to evaluate language models to solve both of the above problems in a single pass. The language model is shown a natural language description of a programming task, and potentially buggy code solution to that task. Then the model is asked to generate test cases consisting of both inputs and oracle outputs. The generated test cases are evaluated in two ways - a) code coverage on the presented buggy code solution, b) pass rate on the ground truth oracle solution. In this paper, the buggy code to be tested is also code generated by a language model.
> The authors consider multiple settings for this:
> -   one where the code is self-generated,
>
> -   one where the code is generated by other models,
>
> -   one where the code is actually oracle code, and
>
> -   one where the code is just a placeholder stub and the model has to use only the problem description.
>      The authors analyze their results in these settings, and then use their insights to solve a related problem - code synthesis. Recent work (CodeT) has shown that generating tests along with code and using
> these tests to filter the generated code can improve generation accuracy. The authors use two insights to improve the performance of CodeT:
> -   Large models generate better tests (as measured by pass rate on the oracle) when shown their own possibly buggy generated code
> (self-generation setting), versus when not shown any code at all (placeholder setting).
>
> -   If a model generates multiple tests in sequence, then earlier tests in the sequence are more likely to be correct than those that come later.



I would like to start my overview with the paper [R. Feldt, S. Kang, J. Yoon, and S. Yoo. Towards autonomous testing agents via conversational large language models, 2023](#ref2) which provides important classification which is related to the [Goals](#goals) I identified earlier and does some experiments similar to mine. I would allow myself extensive quotes from the papers.

*This paper provides an overview of conversational and potentially autonomous testing agents by first presenting a taxonomy of such agents, describing how these agents could help developers (and increasingly so when granted with greater autonomy). A concrete example of a conversation with an LLM is provided as initial confirmation that conversational testing can be used to enhance the testing effectiveness of developers. Finally, limitations of these techniques is provided, providing context for our vision*.

This taxonomy is the part which is interesting for us in the first place. Here is the image:
**![](https://lh7-us.googleusercontent.com/bhlsrLTfNakyVVhYBSAJi7Ud-90o4YaKjxwgMp0BbmZeQE7gIqVNiOi-N7VB3BGJtIHOOPzMYE_YQ-0a1yj2NDw5MVQ2sSE7afRXaNABflHpGOm8wHgCYN8S-2-MQvw84c6KMXdHSP4cYCWo1Gp0Y_0)**
The paper starts from describing in general, that writing tests requires both high-level vision and low-level work and suggests that some sort of conversation process between high-level and low-level agents is needed to elaborate the task:

[Those things about] *test code naturally bring about a distinction between testing experts and domain experts, which existing literature on developer expertise supports
by identifying distinct types of expertise: “understanding the vision of the project” and “knowledge about tools”. Under this framework, an ideal setup would be one in which a testing expert and a domain expert collaborate to write tests for a project. The domain expert may lay out the specifications
of a project, while the testing expert may convert those specifications into concrete tests, based on the testing expert’s experience. A great strength of this process is that as a result of such a dialogue, initially unexpected, yet nuanced issues with the specification may arise, which provide opportunities to clarify the desired behavior.*

Their opinion is that LLMs are currently more capable to act in step-by-step manner under the supervision of software tester with additional information coming from middlware, e.g. program execution:

*we argue that LLMs are sufficiently well-trained with software tests to ‘fill in’ lower level details of the intention of the developer. They also exhibit some ‘knowledge’ about testing methodologies,and can adapt them to new situations. Going further, LLMs appear sufficiently capable in dialogue to converse about the results with a prospective software tester so that they could engage in a ‘Socratic’ manner: that is, they could provide counterexamples to help the developer to think their specification through, and thus uncover unexpected issues with the desired behavior, in this process clarifying what would be ideal. Equipped with appropriate ‘middleware’ which provides tools that the LLM could interact with, our eventual vision is that we can grant the LLM ‘autonomy’, in which it would set up a plan and ‘use’ the tools at its disposal to achieve the high-level objective set by the developer while abstracting away lower-level details.*

Therefore, they propose the taxonomy on the picture above. The taxonomy in Table I classifies the use of Large Language Models (LLMs) in software testing by their level of autonomy. It ranges from low autonomy, where LLMs like GitHub Copilot respond to front-end triggers, to higher autonomy in conversational testing, where LLMs interact with users to drive testing processes. At the highest autonomy level, LLMs independently execute complete testing tasks, guided only by high-level human instructions. This requires implementing middleware to integrate various testing tools and techniques for effective LLM use.

It looks like the lowest levels of autonomy act as a tutorial, provide detailed descriptions, describe possible cases and help junior engineer who is basically going through code. LLM suggestions need constant supervision and corrections after implementation. Writing test here can be quite close to rubber duck tracing with LLM infilling the the short pieces of code. Therefore, the task should be possible to decompose into such chunks.

In their paper, they perform a test with a toy function, and quickly run into LLM hallucination and "*We were not impressed by the fact that the model were now confidently explaining that the clamp function behaves in this way when it had earlier proposed this was not the case. However, the conversational mode of interaction was useful in nudging the model to give us more detailed and concrete information and in particular to provide relevant test code to exemplify its recommendations*".

In my opinion, **this conversational manner with the need to actually supervise the model creates a workload on the engineer**, though may be useful providing explanations how the code works. The middleware output (e.g. traceback) should be sent into the model together with code and objectives approved by engineer. **It can be probably implemented as a side panel which provides live explanations regarding the line of code where cursor sits, and providing immediate infilling in copilot-like manner.**. In addition, I want to mention that LLM context window is too small for efficient understanding of the code, so **probably the actual working code needs specific preprocessing which will split it into blocks, replacing them with just their interfaces and explanations for LLM**, so it can focus on the actual block being tested.

The proposition from the paper is similar:

*As such nonformalized testing techniques or guidelines are intended for humans, they could be readily adopted as part of already existing testing practices, which can improve developer acceptance of results [26], while also acting as explanations for any generated results [27]. At the greatest level of autonomy, LLMs would formulate and execute testing plans, while conversing with the developer on abstract terms. For example, in our example from the previous section, a human had to copy-and-paste the generated tests from an LLM and manually execute the tests; with the appropriate middleware, this process could be automated, and the developer would only need make higher-level decisions.*



The next two papers I want to bring here  extend the above idea to me in a way that **LLM result improvement based on middleware feedback can be iterative, and can be packed into some sort of semi-automated solution which either returns working test or cuts the process after N iterations and passes the task to human**.

The first one is basically LLM Bug reproduction pipeline based on reports: [Large Language Models are Few-shot Testers: Exploring LLM-based General Bug Reproduction (LIBRO)](#ref4).

The system in their example starts from existing bug report, but it can be a GPT suggestion of suitable test cases as well.

![](https://lh7-us.googleusercontent.com/O8bhHJE134X3tvU72EKKSMj2o2jXHxBW1dnu-BVsftPktPYUgQjGkB8ixC0ZYANiuVQ4JJkX17tB6dFAeFhjU_i11TowdGZ2IvQPNnl1hWIMEnBhME-b1RM5HgWZj5iDTPssNmacKX2OTnWf-k-wDa4)
*An overview diagram of our approach is presented in Figure 1. Given a bug report, LIBRO first constructs a prompt to query an LLM (Figure 1:(A)). Using this prompt, an initial set of test candidates are generated by querying the LLM multiple times (Figure 1:(B)). Then, LIBRO processes the tests to make them executable in the target program (Figure 1:(C)). LIBRO subsequently identifies and curates tests that are likely to be bug reproducing, and if so, ranks them to minimize developer inspection effort (Figure 1:(D)). The rest of this section explains each stage in more detail*

The second paper is [*Teaching Large Language Models to Self-Debug*](#ref5):

![](https://lh7-us.googleusercontent.com/eyaebCLYNRh80JAJfwGYOmIPnZ7RgzVNLJVObvuAQ6FwWPNVp_Ww3fK7v5QeLa03af2PrrkPps7XhXpnhMtTcpyh1fzp3k_NLyenfAOCKsf6rF-pKGBKm-49_o7yuyIrj-PHu_K3NfSqD5NBGkqFJIc)

> Figure 1 illustrates our SELF-DEBUGGING framework for iterative debugging, where we utilize a pretrained large language model without finetuning it. One turn of SELF-DEBUGGING consists of 3 steps: Generation, Explanation, and Feedback.  For the Generation step, given the problem description, the model predicts candidate programs.  During the Explanation step, the model is prompted to process the predictions in a semantically useful way, such as explaining the prediction in natural language, or creating an execution trace of the predicted code for a sample input.  Finally, for the Feedback step, a feedback message concerning the code correctness is generated. This can be determined by asking the model itself, or can be generated externally from unit tests. The debugging process terminates when the feedback message states that the prediction is correct, or a maximum allowed number of debugging turns is reached.

They have *Unit test feedback*, *Code Explanation feedback* and *Execution trace feedback* added at each iteration to the model to refine its output. While two others are pretty self-explanatory, I want to cite the description of Code Explanation Feedback:

> Code Explanation feedback (Expl). Despite recent promising progress showing that large language models can generate critiques to avoid harmful model outputs (Ganguli et al., 2023; Bai et al., 2022) and improve their performance on some natural language and reasoning tasks (Shinn et al., 2023; Kim et al., 2023; Saunders et al., 2022), prior work has yet to show the effectiveness of model-generated feedback on code generation (Chen et al., 2023a). On the other hand, large language models have been shown to be capable of describing their generated problem solutions in both text (Wei et al., 2022; Kojima et al., 2022; Zhou et al., 2023) and code (Gao et al., 2022; Chen et al., 2022) formats. Inspired by these observations, instead of teaching the large language model to predict error messages, we propose SELF-DEBUGGING via explaining the generated code. This debugging process is reminiscent of rubber duck debugging, where a programmer debugs code by explaining it line-by-line to a rubber duck (Hunt & Thomas, 2000). By describing the code implementation and comparing it to the problem description, human programmers are usually able to identify the bugs without extra guidance. Empirically, we observe that a large language model can also benefit from rubber duck debugging, especially when unit tests are not available.

Therefore, they add **rubber duck debugging** to the model to make it explain and trace every line of the code turning the program in Nature Language explanation of sort of NL pseudocode. I suppose the success may be caused by the fact that text embeddings of LLM have better quality, so LLM is more efficient in understanding and recalling it using RAG-like mechanisms.

Worth to note, that I failed to obtain rubber duck debugging from GPT-4.

The table from the Self-debug paper showing iteration of text2SQL task:
![](https://lh7-us.googleusercontent.com/A_bYlQCqr0ueyo8KI-h2XJFqKTjhmv9TtihlCVlOmVslHTt9joPtgwiNblRp9e_JnR-EN-Woh-SxIqeXffvMkaYqZYpaiTHjV2EZ-ANmbRFTc92PvvxSeglZce2U9hQ-kgpKLN9tb-94gbEvsWCLfiY)

Next paper which exploits the approach of code+feedback is [Self-Edit: Fault-Aware Code Editor for Code Generation](#ref6). The idea here is the same, their "Neural code editor" loops back to LLM with execution traceback.

Last paper to mention here is [Self-planning Code Generation with Large Language Models](#ref9). Their solution first generates a "Plan" (high-level algorithm) from initial Natural Language formulated task, and then uses the concatenation plan+initial intent to generate next iteration which they call Chain of Thought (CoT), few-shot prompting technique which enables LLMs to perform step-by-step reasoning. Every step in plan represents a single, easily implementable subtask. These sub-tasks are formulated as imperative sentences that start with verbs, focusing on the action needed in each step, because LLM cannot understand that the intent is a combination of multiple problems in direct code generation. The image from the paper illustrates their approach:

![](https://lh7-us.googleusercontent.com/Z6TqXJjZGNMzkKVhbyyvlw35Tm0h3TrFh0NrTgBmRbId8Cb2qr9GSTK-vwPm9Po_pCaUkhltr0f0hjFa86Waruedk_p5ETm6DxzIKxOKB41rOPBbs7QZgmyh66K0icnHHe6hjl54IMdV66SBFEfrAuc)

**We still operate with pass rate like 30-40-60-etc % on HumanEval and similar datasets. So our product should somehow mitigate the 30-70% of garbage generated by LLM**.

**Middleware feedback approach together performing code generation in stages which involve task decomposition, writing plans, etc, seems quite mainstream to me at the moment with the papers I checked proposing variants of those ideas. **

**The iterative approach can probably elevate us in the hierarchy of the test taxonomy, at least making some working tests automatically.  All the cases where automatically and iteratively generated test fails compilation or evaluation can be of interest to human. But it may contain just wrong understanding from LLM and consume human time on checks. Quick solution might consist of evaluation of the results with other LLMs (well-know "experts" approach), thus reducing the amount of accepted tests even more, but probably unloading human from some of the erroneous cases.**

But in general, debugging capabilities of LLMs are low, as concluded in the paper [DebugBench: Evaluating Debugging Capability of Large Language Models](#ref7):

> LLM debugging falls short of human performance. Open-source models attain a pass rate of 0 %, struggling to produce meaningful debugging responses. Closed-source LLMs significantly surpass open-source ones but still fall short of human-level performance; (2) The difficulty of fixing different types of errors differs. Multiple errors and logical errors are significantly more challenging to repair than syntax and reference errors; (3) Runtime feedback has a clear impact on LLM’s debugging performance but is not always helpful. While runtime feedback consistently boosts the debugging performance of syntax and reference bugs, the feedback information is unhelpful for logic errors.

Nevertheless, this paper provided an interesting pipeline bug implantation using GPT-4 which in my opinion can be used for training models using RL objective:

> We first collect code snippets from LeetCode (2023) community, then employ GPT-4 (OpenAI, 2023) for bug implantation and finally conduct human / LLM evaluation on the benchmark.

The next paper I want to propose, guides to some additional practical way to use LLM test generation for industry, particularly addressing coverage problem: [CODAMOSA: Escaping Coverage Plateaus in Test Generation with Pre-trained Large Language Models](#ref1)

Microsoft authors propose using LLMs as addition to algorithmic software which to assist it in discovering more cases for testing. **I think LLM can synthesize synthetic examples well tailored to the code purpose which can be starting point of algorithmic approach.**

Another idea which came into my mind while reading this, is addressing high code repeating in tests: **LLMs can be targeted not to produce new tests, but augment existing ones and reuse the code of the closest unit test function.** Perhaps it was written in some paper, but I forgot the reference.

**This direction can probably mitigate the problem of low quality code generation from LLMs: Use LLMs for augmentation of tests, test case generation to enhance algorithmic solutions, refining LLM suggestions via iteartive loop which involves rubber duck tracing and execution traceback, cutting the procedure after N iterations. In the absence of tests, the initial tests can be generated from the actual data for which the result is known.**

Previous papers used pretrained LLMs. Of course, the iterative feedback approach rises natural question about using reinforcement learning to interact dynamically with environment. Those approaches exist of course, I propose to take a look at the paper
[RLTF: Reinforcement Learning from Unit Test Feedback](#ref10).

The mainstream approach of decomposition of the problem and addressing smaller units is used in this paper. They state:


> while existing RL-based methods do employ the results of unit tests as feedback signals, their implementation is rather simple and coarse-grained. They assign the same reward to the entire episode (a complete code segment) based on the program’s submission result (true, false, runtime error, or compiler error), without considering that a part of the code may be correct while another part contains bugs. As a result, these approaches fail to capture the nuances in identifying individual code components that contribute to the overall functionality and fixing bugs specific to certain parts of the code.


> we have analyzed the distribution of error types in programs and extracted more detailed information from unit test feedback for training, such as “fine-grained feedback” and “adaptive feedback”. Fine-grained feedback categorizes errors based on their reported information and location, penalizing the specific erroneous parts of the code accordingly. Adaptive feedback provides varying rewards to programs based on the ratio of test cases they pass.

They introduce a number of different error weights for different kinds of errors. They have  Coarse-grained Feedback, Fine-grained Feedback and Adaptive Feedback in addition to supervised learning loss.

Though they use external labels which allow them to run unit tests and use supervised loss, I think those ideas can be used to enhance the aforementioned approaches with RL.

### Conclusion

I took the task of Unit testing as research objective. Other testing problems communicate with more complicated software (e.g. docker image or services as black boxes) and can rather be developed as extension of unit tests which in their minimal form can address small units of the source code (the limitation can be set manually).

To create a valuable commercial product, we need to create a solution which somehow mitigates reduces the workload of software engineer. Possible solutions elevate from different assistance tasks which might help human engineer to write and debug the code, to fully automatic solutions which may produce some tests with low degree of supervision. We need to choose the most practical level for us where we can create value.

The general result is that LLMs are good at summarization and textual description of the input data, but code generation made and especially debugging made by LLMs is still very bad. Probable reason of it is that code generation is a complex and multi-objective task, so the mainstream approaches at the moment introduce the following actions to mitigate it:

1. Splitting larger problem into detailed plans with as much as possible separation of actions.  Those plans are concatenated with initial prompt to enhance the task description for LLM.
2. Force LLM understanding of the input and output code by again providing detailed algorithmic explanations and rubber duck tracing with possible involvement of algorithmic middleware. In my opinion, those pieces of textual data are better understood by LLM and better retrieved from the knowledge base / conversation history.
3. Perform iterative generation with the usage of middleware (execution traceback and external unit tests) with some limited number of attempts. Many papers operate with benchmarks where scores for attempts 1, 5 and 10 are given.

I also think that middleware output can be not only a single error message with line number, but a full log of execution traceback of the program taken from the very beginning with the values of all the variables logged. To be able to use this data given the small LLM context window, the textual explanation and summarization proposed above can be applied with the help of another LLM which is more capable in that. Even if current out of the box models may be not trained for this specific task, a lot of information coming from the debugging process can be used for supervised and online RL fine tuning.

Reinforcement learning can be introduced to the iterative process, in theory allowing us to improve the results, especially if some ground truth data is present for more stable training (bug reports, valid use cases).

Specific problem of testing involves LLM-based Bug Reproduction problem in case test reports are available which needs from LLM code understanding and debugging capabilites.

The pipeline can be performed by multiple different models tailored to different tasks. For example, we can use specific models developed for short code generation and code embeddings like [MagicCoder](https://github.com/ise-uiuc/magicoder), and do task splitting into subtasks by powerful LLM like GPT. Also, more classical neural networks and ML models can be trained using a lot of synthetic data generated by LLM (e.g. classification task) with the ability to involve LLM only when classifier confidence is small.

**My suggestions for commercial solution, we can tackle the test taxonomy in two or three places**.

The main problem is to deal with low code and debugging quality of LLMs which create only 30-90% of working code for rather simple isolated problems like the ones from HumanEval dataset.

1. The lowest level is **code assistance**. Summarizations and explanations of each program step are already useful and we can provide it as an assistance to the software engineer, filling small code blocks in copilot-like manner, keeping human supervision. Also we can display explanation of the current line of code and suggestions in the panel nearby. This may help us to collect a lot of fine-level supervised training data for those small code blocks which may help us to improve automatic solutions while helping engineer with understanding, debugging and repetitive code tasks which are abundant in testing.
2. **Semi-automatic level**: accepting tests which are generated in the iterative LLM pipelines. It can be achieved by taking coarse data of bug reports, real enterprise data processed by the code every day, existing working unit tests, and feeding it into (possibly RL-enabled) iterative solutions. In this case, we can carefully accept the tests which work for real use-case data (even though some of them might be erroneous like `return True` ), and provide the code which failed N generation iterations for human observation of just filter it.
3. We can **add LLMs to the existing automated solutions** in such way that novel test cases proposed by LLMs do not break the procedure, but augment its logic and increase coverage which can be evaluated.
4. We can carefully apply LLM test generation involving **only small modification of existing test functions** and mock data. Or provide existing tests to the iterative generation pipeline.

LLM can also help us with auxiliary tasks like:

I. **Covering all the code in logging** - pretty simple, yet extensive task, which will in turn generate a lot of data which we can later feed into our test generation network.
II. The same for **code commenting** which may save us time later by passing those comments together with code to LLM. By the way, the code is naturally represented as Graph (e.g. AST tree) and Graph Neural Networks coupled with existing transformers (e.g. in the role of encoder or decoder) seem like a natural solution, but I don't see much publications in the topic. For example, I found 2022 article [Code comment generation based on graph neural network enhanced transformer model for code understanding in open-source software ecosystems](https://dl.acm.org/doi/abs/10.1007/s10515-022-00341-1).

**I think, we can start simultaneously with 1) and 2)**, by collecting human feedback from copilot and informational solution, at the same time evaluating the quality and coverage of the working tests, coming from the iterative pipelines which more or less allow us to drop the worst garbage coming from LLMs by discarding it after N iterations.

While our solutions are weak, we can tailor them only to generate tests for the labeled cases (when bug report and/or real data exists), then slowly turning to exploration of code vulnerabilities.

Product 1) will also provide us with fine-level labeled data (proposed mini unit vs the one coded by software engineer) for improving both approaches.

When having the data, we can benefit from RL and fine tuning our ensemble of LLMs to work better with the types of data we provide (rubber duck tracebacks, excecution tracebacks, etc) instead of creating complicated pipelines for pretrained LLMs which were not designed for those specific tasks.

**During the development, I expect we need to seek the solutions to the following problems**:

1. Reducing the complexity the the problem for LLM by splitting the code into the smallest units possible, as well as generation of plans and pseudocode to guide LLM generation, debugging, and bug discovery. Exploring graph-based code representations might turn to be beneficial.
2. Develop better addressing of codebase (knowledge base). This might involve using code embeddings for RAG, code preprocessing like substitution of all irrelevant code blocks with their interfaces, extensive usage of summarizations and comments on different levels.
3. Improve LLM+middleware debugging capabilites.


## References <a name="references"></a>

1. [**CODAMOSA: Escaping Coverage Plateaus in Test Generation with Pre-trained Large Language Models:**](https://www.carolemieux.com/codamosa_icse23.pdf)
<a name="ref1"></a>
3. [**R. Feldt, S. Kang, J. Yoon, and S. Yoo. Towards autonomous testing agents via conversational large language models. CoRR, abs/2306.05152, 2023**](https://arxiv.org/pdf/2306.05152.pdf)** <a name="ref2"></a>
4. [**Large Language Models for Software Engineering: A Systematic Literature Review**](https://arxiv.org/pdf/2308.10620.pdf)  <a name="ref3"></a>
5. [**Large Language Models are Few-shot Testers: Exploring LLM-based General Bug Reproduction (LIBRO)**](https://arxiv.org/pdf/2209.11515.pdf), [Github](https://github.com/coinse/libro) <a name="ref4"></a>
6. [Teaching Large Language Models to Self-Debug](https://arxiv.org/pdf/2304.05128.pdf) <a name="ref5"></a>
7. [**Self-Edit: Fault-Aware Code Editor for Code Generation**](https://aclanthology.org/2023.acl-long.45v3.pdf)<a name="ref6"></a>
8. [DebugBench: Evaluating Debugging Capability of Large Language Models](https://arxiv.org/abs/2401.04621)<a name="ref7"></a>
9. [**The Program Testing Ability of Large Language Models for Code**](https://openreview.net/forum?id=PFdjJiZjPj)
<a name="ref8"></a>
10. [**Self-planning Code Generation with Large Language Models**](https://arxiv.org/pdf/2303.06689.pdf)
<a name="ref9"></a>
11. [**RLTF: Reinforcement Learning from Unit Test Feedback**](https://openreview.net/forum?id=hjYmsV6nXZ) [Code](https://github.com/Zyq-scut/RLTF)
<a name="ref10"></a>




