# Code for MCQ-based evaluation, etc.

Here we describe Python programs for:
* Generating and evaluating MCQs
* Fine-tuning models based on supplied data
* Other useful things

Please email foster@anl.gov and stevens@anl.gov if you see things that are unclear or missing.

## Set up to access ALCF Inference Service 

**Before you start:** We recommend you follow the instructions for 
[ALCF Inference Service Prerequisites](https://github.com/argonne-lcf/inference-endpoints?tab=readme-ov-file#%EF%B8%8F-prerequisites)
to set up your ALCF auth token, required to access models via the inference service.
(You need to download and run `inference_auth_token.py`.)

## Code for generating and evaluating MCQs

### Programs to run PDFs &rarr; JSON &rarr; LLM-generated MCQs &rarr; LLM-generated answers &rarr; LLM-scored answers

1. Set up your working directory. From within your working directory, create directories for your PDF files and for JSON output.
The instructions below use the directory
names *myPDFcache* and *JSON-out* for these two directories. Substitute your directory names as appropriate.
Details on programs follow. Use `-h` to learn about other options.
+ Create your directories

```
mkdir myPDFdir myJSONdir
```

+ Populate your myPDFdir directory with your PDF files.

2. Set up your Conda environment.  If you already set up a conda env (such as via the
[prerequisites](https://github.com/argonne-lcf/inference-endpoints?tab=readme-ov-file#%EF%B8%8F-prerequisites)
page referenced above) then you can update your environment with the new dependencies you'll need here:  
*conda env update --name <name_of_your_conda_env> --file environment.yml*.  
Otherwise, create a new Conda environment as follows:
```
conda env create -f environment.yml
```
As specified in the first line (name:) of environment.yml, this will create a new Conda env named
*globus_env*. If you get an error *CondaValueError: prefix already exists* then you can change the name of the
Conda environment by editing the first line of the environment.yml file.

3. Extract text from PDFs with simple parser to create JSON files
```
python simple_parse.py -i myPDFdir -o myJSONdir
```

+ Or: Extract text from PDFs with higher-quality AdaParse 
See [https://github.com/7shoe/AdaParse/tree/main](https://github.com/7shoe/AdaParse/tree/main) 
(still testing this step)

4. Use specified LLM to generate MCQs for papers, after dividing paper text into chunks
and augmenting each chunk with extra info. In this example we will specify the
*allenai/Llama-3.1-Tulu-3-405B* model (see 
[alcf endpoints](https://github.com/argonne-lcf/inference-endpoints) 
for more options)
```
python generate_mcqs.py -i myJSONdir -o MCQ-JSON-file -m 'mistralai/Mistral-7B-Instruct-v0.3'
```

+ Next is useful if you run `generate_mcqs.py` multiple times and thus have multiple JSON files
```
python combine_json_files.py -i myJSONdir -o JSON-file
```

4. Select subset of MCQs from output of step 2, for subsequent use
```
python select_mcqs_at_random.py -i MCQ-JSON-file -o MCQ-JSON-file -n <N>
```

5. Use specified LLM to generate answers to MCQs generated in step 2
Read MCQs in <input-json>
Use LLM <model>, executed at <locn> (see below), to generate MCQs.
Place results in "<result-directory>/answers_<model>.json"

```
python generate_answers.py -i MCQ-JSON-file -o myRESULTSdir -m <locn>:<model>
```

6. Use specified LLM to score answers to MCQs generated in step 4
Look for file "answers_<model-A>.json" in <result-directory>
Produce file "scores_<locnA>:<model-A>:<locnA>:<model-B>.json", with any `/` replaced with `+`.
Where <model-A> and <model-B> are executed at <locn-A> and <locn-B>, respectively.

*python score_answers.py -o myRESULTSdir -a <locn-A>:<model-A> -b <locn-B>:<model-B>*

7. Run whatever LLMs are running on ALCF inference service to generate and/or score answers
(Based on:
+ Query to inference service to identify running models
+ Examining answers and scores files in <result-directory>)
```
python review_status.py -i MCQ-JSON-file -o myRESULTSdir
```

**CeC edits stop here**

**Note:**
* You need a file *openai_access_token.txt* that contains your OpenAI access token if you
are to use an OpenAI model like *gpt-4o*.

Examples of running *generate_answers.py*:
* `python generate_answers.py -o RESULTS -i MCQs.json -m openai:o1-mini.json`
  * Uses the OpenAI model `o1-mini` to generate answers for MCQs in `MCQs.json` and stores results in the `RESULTS` directory, in a file named `answers_openai:o1-mini.json`
* `python generate_answers.py -o RESULTS -i MCQs.json -m "pb:argonne-private/AuroraGPT-IT-v4-0125`
  * Uses the Huggingface model `argonne-private/AuroraGPT-IT-v4-0125`, running on a Polaris compute node started via PBS, to generate answers for the same MCQs. Results are placed in `RESULTS/answers_pb:argonne-private+AuroraGPT-IT-v4-0125.json`
 
Examples of running `score_answers.py`:
* `python score_answers.py -o RESULTS -i MCQs.json -a openai:o1-mini.json -b openai:gpt-4o`
  * Uses the OpenAI model `gpt-4o` to score answers for MCQs in `MCQs.json` and stores results in `RESULTS` directory, in a file named `answers_openai:o1-mini.json`
* `python score_answers.py -o RESULTS -a pb:argonne-private/AuroraGPT-IT-v4-0125 -b openai:gpt-4o`
  * Uses the OpenAI model gpt-4o to score answers previously generated for model `pb:argonne-private/AuroraGPT-IT-v4-0125`, and assumed to be located in a file `RESULTS/answers_pb:argonne-private+AuroraGPT-IT-v4-0125.json`, as above. Places results in file `RESULTS/scores_pb:argonne-private+AuroraGPT-IT-v4-0125:openai:gpt-4o.json`.
 

## Notes on different model execution locations
The class `Model` (in `model_access.py`) implements init and run methods that allow for use of different models. 
```
model = Model(modelname)
response = model.run(user_prompt='Tell me something interesting')
```
where `modelname` has a prefix indicating the model type/location:
* **alcf**: Model served by the ALCF Inference Service. You need an ALCF project to charge to.
* **hf**: Huggingface model downloaded and run on Polaris login node (not normally a good thing).
* **pb**: Huggingface model downloaded and run on a Polaris compute node. You need an ALCF project to charge to.
* **vllm**: Huggingface model downloaded and run via VLLM on Polaris compute node. Not sure that works at present.
* **openai**: An OpenAI model, like gpt-4o or o1-mini. You need an OpenAI account to charge to.


## Code for fine-tuning programs
```
# LORA fine-tuning
python lora_fine_tune.py -i <json-file> -o <model-directory>

# Full fine tune
python full_fine_tune.py -i <json-file> -o <model-directory>
```
Note:
* You need a file `hf_access_token.txt` if you want to publish models to HuggingFace.
* You need to edit the file to specify where to publish models in HuggingFace
* We are still debugging how to download and run published models

## Code for other useful things

Determine what models are currently running on ALCF inference service (see below for more info)
```
python check_alcf_service_status.py
```
Determine what answers have been generated and scored, and what additional runs could be performed, _given running models_, to generate and score additional answers. (You may want to submit runs to start models. Use `-m` flag to see what could be useful to submit.) 
```
python review_status.py -o <result-directory>
```
Perform runs of `generate_answers` and `grade_answers.py` to generate missing outputs. (See below for more info)
```
python run_missing_generates.py -o <result-directory>
```

### More on `check_alcf_service_status.py` 

The program `check_alcf_service_status.py` retrieves and processes status information from the [ALCF Inference service](https://github.com/argonne-lcf/inference-endpoints), and lists models currently running or queued to run. E.g., as follows, which shows six models running and one queued. Models that are not accessed for some period are shut down and queued models started. A request to a model that is not running adds it to the queue.
```
% python check_alcf_service_status.py
Running: ['meta-llama/Meta-Llama-3-70B-Instruct', 'meta-llama/Meta-Llama-3-8B-Instruct', 'mistralai/Mistral-7B-Instruct-v0.3']
Starting: ['N/A']
Queued : []
```
Note:
* You need a valid ALCF access token stored in a file `alcf_access_token.txt`.  See [how to generate an ALCF access token](https://github.com/argonne-lcf/inference-endpoints?tab=readme-ov-file#authentication).
* Here is a list of [models supported by the ALCF inference service](https://github.com/argonne-lcf/inference-endpoints?tab=readme-ov-file#-available-models).
* "N/A" is a test model used by ALCF, it can be ignored.

### More on `run_missing_generates.py`

The ALCF inference service hosts many models, as [listed here](https://github.com/argonne-lcf/inference-endpoints?tab=readme-ov-file#-available-models). At any one time, zero or more *running*, zero or more are *queued*, and the rest are neither running not queued. (See below for how to use `check_alcf_service_status.py` to determine which.)
You may want to run against all available models. To do so, you can specify `-a all`, which works out what commands are needed to process specified MCQs with all *running models*. Adding `-q` also considers *queued models*, and `-s` *non-running models*. For example, when I ran the following command I was informed of the commands to run three models for which results are not found:
```
% python run_missing_generates.py -i 100-papers-qa.json -o output_files -a all -m 100 -s
python generate_and_grade_answers.py -i 100-papers-qa.json -o outputs -a 'Qwen/Qwen2-VL-72B-Instruct' -b 'gpt-4o' -c -q -s 0 -e 100
python generate_and_grade_answers.py -i 100-papers-qa.json -o outputs -a 'deepseek-ai/DeepSeek-V3' -b 'gpt-4o' -c -q -s 0 -e 100
python generate_and_grade_answers.py -i 100-papers-qa.json -o outputs -a 'mgoin/Nemotron-4-340B-Instruct-hf' -b 'gpt-4o' -c -q -s 0 -e 100
```

`run_missing_generates.py` has options as follows:

```
  -h, --help            show this help message and exit
  -a MODELA, --modelA MODELA
                        modelA
  -o OUTPUTDIR, --outputdir OUTPUTDIR
                        Directory to look for run results
  -i INPUTFILE, --inputfile INPUTFILE
                        File to look for inputs
  -x, --execute         Run program
  -q, --queued          Process queued models
  -m MAX, --max MAX     Max to process
  -s, --start           Request to non-running models
```




