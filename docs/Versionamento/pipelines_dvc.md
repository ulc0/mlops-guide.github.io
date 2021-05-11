# Working with Pipelines

After learning how to version data or models using DVC it's time to build your experiment pipeline.

Let's assume that we are using the last section dataset as a source of data for training a classification model. Let's also assume that we have three stages in this experiment:

- Preprocessing your data(extract features...)
- Train the model
- Evaluate the model

## Creating pipelines

DVC builds a pipeline based on three components: Inputs, Outputs, and Command. So for the preprocessing stage, this would look like this:

 - Inputs: dataset.csv and preprocess.py script
 - Outputs: dataset_ready.csv
 - Command: python preprocess.py dataset.csv

So to create this stage of preprocessing, we use ```dvc run```:

```
corrigir
$ dvc run -n preprocess \
         -d src/preprocess.py -d data/dataset.csv \
         -o data/dataset_ready.csv \
         python src/preprocess.py dataset.csv
```

We named this stage "preprocess" by using the flag ```-n```. We also defined this stage inputs with the flag ```-d``` and the outputs with the flag ```-o```. The command will always be the last piece of ```dvc run``` without any flag.

!!! tip
    Output files are added to DVC control when reproducing a DVC stage. When finalizing your experiment remember to use ```dvc push``` to version not only the data used but those outputs generated from the experiment.

The train stages would also be created using ```dvc run```:

```
corrigir
$ dvc run -n train \
         -d src/train.py -d data/dataset_ready.csv \
         -o model/model.joblib \
         python src/train.py dataset_ready.csv
```

At this point, you might have noticed that two new files were created: **dvc.yaml** and ***dvc.lock***.  The first one will be responsible for saving what was described in each ```dvc run``` command. So if you wanna created or change a specific stage, it's possible to just edit ***dvc.yaml*** . Our current file would look like this:

```
corrigir
stages:
  preprocess:
    cmd: python src/preprocess.py data/dataset.csv
    deps:
      - data/dataset.csv
      - src/preprocess.py
    outs:
      - data/dataset_ready.csv
  train:
    cmd: python src/train.py data/dataset_ready.csv
    deps:
      - data/dataset_ready.csv
      - src/train.py
    outs:
      - model/model.joblib
```

The second file created is ***dvc.lock***. This is also a YAML file and its function is similar to ***.dvc*** files. Inside, we can find the path and a hash code for each file of each stage so DVC can track changes. Tracking these changes is important because now DVC will know when a stage needs to be rerun or not.

Your currently pipeline looks likes this:

imagem


## Saving metrics

Finally, let's create our last stage so we can evaluate our model:

```
corrigir
$dvc run -n evaluate 
-d ./src/evaluate.py -d  ./data/dataset_ready.csv  -d ./models/model.joblib -o ./results/metrics.json \ -o ./results/precision_recall_curve.png -o ./results/roc_curve.png \ python3 ./src/evaluate.py ./data/weatherAUS_processed.csv ./src/model.py ./models/model.joblib
```

You might notice that we are using the -M flag instead of the -o flag. This is important because now we can keep the metrics generated by every experiment. If we run ```dvc metrics show``` we can see how good was the experiment:

ascii_cinema

Another import command is if we want to compare this experiment made in our branch to the model in production at the main branch we can do this by running ```dvc metrics diff```:

ascii_cinema


## Control the experiment

So now we have built a full machine learning experiment with three pipelines: 

imagem

DVC uses a Direct Acyclic Graph(DAG) to organize the relationships and dependencies between pipelines. This is very useful for visualizing the experiment process, especially when sharing it with your team. You can check the DAG just by running ```dvc dag```:

ascii_cinema

If you want to check any changes to the project's pipelines, just run:

```
$ dvc status
```

## Reproducibility

Either if you changed one stage and need to run it again or if you are reproducing someone's experiment for the first time, DVC helps you with that:

Run this to reproduce the whole experiment:

```
$ dvc repro
```

Or if you want just to reproduce one of the stages, just let DVC know that by:

```
$ dvc repro stage_name
```
!!! info 
    If you are using ```dvc repro``` for a second time, DVC will reproduce only those stages that changes have been made.

___