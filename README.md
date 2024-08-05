# Prerequisites and Notebooks for Oracle CloudWorld 2024 Hands-on-Lab Session 210

## Prerequisites

Note that the following prerequisites assume that you already have an Oracle Cloud Account, have access to OCI Generative AI, and are able to create Object Storage in your Oracle Cloud account.
If not, you can visit the following product pages for information to get them set up:
<ul><li> Oracle Cloud account — https://signup.cloud.oracle.com/ </li>
<li> Getting started with OCI Generative AI — https://docs.oracle.com/en-us/iaas/Content/generative-ai/getting-started.htm </li>
<li> OCI Object Storage - https://docs.oracle.com/en-us/iaas/Content/Object/home.htm </li>
</ul>

### 1. Set up

Follow links below to generate a config file and a key pair in your ~/.oci directory
<ul><li> https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdkconfig.htm </li>
<li> https://docs.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm </li>
<li> https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm#configfile </li>
</ul>
After completion, you should have following 2 things in your ~/.oci directory 
<ul><li> A config file(where key file point to private key:key_file=~/.oci/oci_api_key.pem) </li>
<li> A key pair named oci_api_key.pem and oci_api_key_public.pem </li>
</ul>    

Now make sure you change the reference of key file in config file (where key file point to private key:key_file=/YOUR_DIR_TO_KEY_FILE/oci_api_key.pem)

### 2. Upload Public Key

Upload your oci_api_key_public.pem to console: https://docs.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm#three


### 3. Make sure you have python installed on your machine
```
python --version
```
 
### 4. Install all dependencies:

We suggest you install dependencies in a virtual env to avoid conflicts on your system
```
python3 -m venv venv
. venv/bin/activate
pip install pandas
pip install -U oci
pip install word2number
pip install pyarrow
```

### 5. make sure you have updated all corresponding fields in notebooks 
For example, in “GenerateFinetuningData-MathProblems-GitHub.ipynb”, update Namespace = 'abcdefg' with your namespace where your bucket (object storage) is. These are all marked with preceding comments like # TODO: replace this with your **

### 6. Download orca-math-word-problems-200k data from huggingface and save the parquet to your local drive, e.g., in "./Data" folder
https://huggingface.co/datasets/microsoft/orca-math-word-problems-200k/tree/refs%2Fconvert%2Fparquet/default/train


## Notebooks

### 1. GenerateFinetuningData-MathProblems-GitHub.ipynb

This notebook prepares the fine-tuning dataset and uploads it to object storage. Specifically, it performs the following tasks:
<ul><li> Read in the math data set as a pandas dataframe </li>
<li> Insert content from the question column into a prompt template with instructions for the LLM to form the prompt column </li>
<li> Output the prompt and completion columns from the first 180K rows of the dataframe as jsonl into a local folder (this can be "./Data" folder). These are the training samples you will use for fine-tuning </li>
<li> Upload the saved jsonl file to an existing object storage bucket on Oracle Cloud </li>
</ul>

### 2. Inference&PerformanceEvaluation-MathProblems-GitHub.ipynb

After you have run the GenerateFinetuningData-MathProblems-GitHub.ipynb notebook to prepare and upload the fine-tuned dataset, you can configure your fine-tuning job on OCI Generative AI Service (https://docs.oracle.com/en-us/iaas/Content/generative-ai/fine-tune-models.htm) and start running fine-tuning. Upon completion of a fine-tuning job, you can create a hosting dedicated AI cluster to host the newly fine-tuned model as an endpoint. 

This notebook assumes that you have already created an endpoint for your fine-tuned model. It performs the following tasks:

<ul><li> Specify the entities such as endpoint OCID and parameters to be used for inferencing from the fine-tuned model. </li>
<li> Define two prompts: the first one matches with the one used for fine-tune dataset creation and is used for prompting the fine-tuned model for answering math questions; the second one is customized for extracting math answer keys from math answers. </li>
<li> Define an inferencing function that supports calling both the out-of-the-box Cohere model and the fine-tuned model. </li>
<li> Define an accuracy evaluation function that compares math answer keys extracted from the LLM response with the keys extracted from the expected answers .</li>
<li> Use only the last 1K rows from the math problem dataset for evaluation. Note that there are 200K records in the math problem dataset and we only use the first 180K for fine-tuning. The last 1K records are out-of-the-bag samples that have not been used in fine-tuning. </li>
<li> During the first round of inferencing, we use the OOTB Cohere model for these 1K records. We obtain the answers first as texts, then extract the answer keys from these texts. The second round we replace the OOTB Cohere model with the fine-tuned model and repeat the same steps. </li>
<li> Compare the two sets of answer keys with the expected answer keys (extracted from original data's answer column) using the accuracy evaluation function separately and note down the overall accuracy. </li>
<li> Optional: save all the answers and extracted keys for the 1K records in a local file so you don't have rerun the inferencing if you want to perform further analysis later. </li>
</ul>