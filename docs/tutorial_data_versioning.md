# Steps for data versioning

Data versioning is a essencial step in any Machine Learning projects. It enables developer's teams to create multiple datasets and easily change between them when training. It is useful when the team have a lot of data and is trying to use only the samples that increase model performance. In this project [dvc](https://dvc.org/doc/api-reference) combined with git is used to implement this task. All datasets versions are stored at a S3 bucket.

## Create a new data enviroment

Sometimes, it is necessary to start everything all again. The following steps show how can you do that:

1. Remove all tags already created (remote and local)

```Bash
git push origin --delete $(git tag -l)

git tag -d $(git tag -l)
```

- Ensure the tags were erased:

![tags_erased](./_static/imgs/tags_erased.png)

2. Run data.sh to create the file "data/data.zip" with your preprocessed data. Drop value is the ratio of the dowloaded dataset that will be erased.

```Bash
./scripts/data.sh <drop_value>
```

5. Run configure_dvc.sh and pass as argument the Bucket created for the dataset

```Bash
./scrips/configure_dvc.sh bucket-dataset-name
```

After that, you will have a tag v0.0.0 with the first version of the dataset!

## Create a new dataset version

Everytime you want to create a new dataset version, run the steps bellow:

1. Do changes in the function prepocess from [preprocess.py](./data/preprocess.py). Then, run [data.sh](./data.sh):

> [!WARNING]
> Check if you are at main:
> ```Bash
> git checkout main
> ```

```Bash
./scrips/data.sh <drop_value>
```

2. Run script that create new data version:

```Bash
./scripts/new_dataset_version.sh vA.B.C
```

3. To use a specific data version:

```Bash
git checkout vA.B.C
dvc checkout
```
