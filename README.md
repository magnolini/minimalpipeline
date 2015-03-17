# Minimal Structural Reranking Pipeline

Thisi is a stripped down version of the reranking pipeline.  

With this software you will be able to apply structural reranking on a question answering dataset.

## Prerequisites

### UIMA

Go to https://uima.apache.org/downloads.cgi and download the latest binary distribution of UIMA.  

Decompress the archive and set the $UIMA_HOME environmental variable to the bin/ folder in the
UIMA distribution directory. You can use export="path/to/dir" if you use Linux/Mac and add this line
to ~/.bashrc in order to automatically set it in the future.

### Maven

You will need Maven for building this project. Install maven for your operating system (e.g. sudo
apt-get install maven).  

If you use Eclipse please install the M2E plugin. Go to Help > Install new software... and search
for m2e.

### UIMA tooling for Eclipse

UIMA tooling simplifies the development of typesystems and annotators in Eclipse. Install Eclipse EMF
following these instructions: http://tinyurl.com/UIMA4ECLIPSE. The useful UIMA visual
tools (UIMA tooling) can be found at this software update address:
http://www.apache.org/dist/uima/eclipse-update-site/.

## Installation

Clone the minimal pipeline from the repository with:

```
git clone https://github.com/mnicosia/minimalpipeline.git
```

Cd into the resources/ folder and execute the generate-types.sh script

Now go to the parent directory and type:

```
mvn compile
```

Then

```
mvn clean dependency:copy-dependencies package
```

## Running the pipeline

```
./run.sh
```

Look into the run.sh script and the arguments/trec-en-pipeline-arguments.txt file to understand what is happening.  

## Importing the project into Eclipse

Import the project into Eclipse by clicking on File -> Import -> Existing Maven Projects. Select the pipeline folder and click on Finish.

## Running an experiment

### Compiling SVMLightTK

Go to the tools/SVM-Light-1.5-rer/ folder and type:

```
  make clean  
  make
```

### The experiment
The experiment consists into reranking candidate answer passages related to questions from the TREC dataset.
This dataset contains 824 questions and their answer patterns. The candidate answer passages come from the AQUAINT corpus. The latter was indexed and queried using the question terms in order to retrieve a list of related passages.

`data/trec-en/questions.txt` contains the questions

`data/trec-en/terrier.BM25b0.75_0` contains the candidate answer passages

### The pipeline

The `TrecPipelineRunner` program is used to execute the pipeline which analyzes questions and passages and produces training and test data for the reranking task.

The program arguments are:

`-argumentsFilePath arguments/trec-en-pipeline-arguments.txt`

This file contains all the actual arguments of the program.

`-trainQuestionsPath data/trec-en/questions.txt`

`-trainCandidatesPath data/trec-en/terrier.BM25b0.75_0`

`-trainCasesDir CASes/trec-en/`

`-trainOutputDir data/trec-en/train/`

`-testQuestionsPath data/trec-en/questions.txt`

`-testCandidatesPath data/trec-en/terrier.BM25b0.75_0`

`-testCasesDir CASes/trec-en/`

`-testOutputDir data/trec-en/test/`

`-candidatesToKeepInTrain 10`

`-candidatesToKeepInTest 50`

`-lang en`

The same files are passed in both training and testing since we will generate the training and test data in a cross validation fashion.

The Java virtual machine arguments are the following:

`-Djava.util.logging.config.file="resources/logging.properties"`

This argument sets the logging properties.

After the execution of the pipeline, in `data/trec-en/` there will be the `train` and `test` folders with the data required for the experiment.

### Settings

The candidates kept for training are 10 while the candidates kept for testing are 50.

Out of 824 questions only 416 are used to generate training examples because the others do not have valid candidate passages in the top-50 candidates list.

### Producing the folds

The `python scripts/folds.py data/trec-en/ 5` command will use the data under the specified directory to output five folds. The scripts assumes that the train and test data are under the specified directory, and that they contains the examples and metadata in files having specific names.

### Learning, reranking and evaluation

The `python scripts/svm_run_cv.py --params="-t 5 -F 3 -C + -W R -V R -m 400" --ncpus 2 data/trec-en/folds/` will launch the learning, the reranking and eventually, the evaluation which will be carried out on single folds and then used to produce metrics averaged on all folds. The `--ncpus` parameter can be used to parallelize the jobs.
 
