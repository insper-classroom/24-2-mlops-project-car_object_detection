# 24-2-mlops-project-car_object_detection

Dataset from:

https://universe.roboflow.com/openglpro/stanford_car

I used the YOLOv8 dataset

This dataset can detect both cars and bikes. I merged both train and test dataset using the script data/merge_train_test.py

## Requirements
- python-dotenv
- dvc[s3]
- requests
- roboflow
- boto3
- ultralytics

> [!WARNING]
> If you would like to run with GPU, download CUDA Toolkit 12.6 https://developer.nvidia.com/cuda-downloads

## .env file
ROBOFLOW_API_KEY=""
AWS_ACCESS_KEY_ID=""
AWS_SECRET_ACCESS_KEY=""
AWS_REGION=""

## Steps for data versioning

### Create a new data enviroment

Sometimes, it is necessary to start everything all again. The following steps show how can you do that:

1. Remove all tags already created (remote and local)

```Bash
git push origin --delete $(git tag -l)

git tag -d $(git tag -l)
```

Ensure the tags were erased:

![tags_erased](./imgs/tags_erased.png)

2. Also, remove the folder ".dvc/" and the files "data/data.zip.dvc" and ".dvcignore"

```Bash
rm -rf .dvc/ data/data.zip.dvc .dvcignore
```

3. Create S3 bucket to save dataset versions, and remove any S3 bucket if necessary.

```Bash
# List S3 buckets in the AWS account if necessary
python3 data/list_S3_buckets.py

# Create S3 bucket
python3 data/create_S3_bucket.py --bucket_name $bucket_name

# Delete S3 bucket
python3 data/delete_S3_bucket.py --bucket_name $bucket_name
```

4. Run data.sh to create the file "data/data.zip" with your preprocessed data. Drop value is the ratio of the dowloaded dataset that will be erased.

```Bash
chmod +x data.sh

./data.sh <drop_value>
```

5. Run configure_dvc.sh and pass as argument the recently created Bucket

```Bash
chmod +x configure_dvc.sh

./configure_dvc.sh <BUCKET>
```

After that,  you will have a tag v0.0.0 with the first version of the dataset!

### Steps for creating new tag

Everytime you want to create a new dataset version, run the steps bellow:

1. Do changes in the function prepocess from [preprocess.py](./data/preprocess.py). Then, run [data.sh](./data.sh):

> [!WARNING]
> Checkout if you are at main:
> ```Bash
> git checkout main
> ```

```Bash
./data.sh
```

2. Run the following commands:

```Bash
dvc commit data/data.zip
dvc push

git add .
git commit -m "version vA.B.C"
git push

git tag -a vA.B.C -m "Release version A.B.C"
```

3. To save tag in remote repo:

```Bash
git push origin tag vA.B.C
```

3. To use a specific data version:

```Bash
git checkout vA.B.C
dvc checkout
```

## Steps for training

1. Unzip data using the command:

```Bash
unzip data/data.zip
```

2. Inside the Ultralytics folder, change it so runs are saved in the models folder of this repository. 

```Bash
cd /home/user/.config/Ultralytics

sudo vim settings.json
```

Do the following changes in settings.json:

```Json
"datasets_dir": "/home/user/your_path/24-2-mlops-project-car_object_detection",
"weights_dir": "/home/user/your_path/24-2-mlops-project-car_object_detection/models/weights",
"runs_dir": "/home/user/your_path/24-2-mlops-project-car_object_detection/models/runs",
```

3. Using the create_S3_bucket.py, create a S3 bucket for saving models and save the name in .env file:

```Bash
python3 data/create_S3_bucket.py $bucket_name --model_bucket
```

If you already have a bucket for models you can just set a variable in the .env file:

```Bash
BUCKET_MODEL="bucket name"
```

4. In the root folder of the repository, start Mlflow:

```Bash
mlflow ui --backend-store-uri ./runs/mlflow
```

![empty_mlfow](./imgs/empty_mlflow.png)

5. In another terminal, train model:

```Bash
cd src/

python3 train.py
```

6. Train again, changing hyperparameters if necessary.

![mlflow_working](./imgs/mlflow_working.png)

7. All runs will be saved in ["models/runs"](./models/runs/)

![mlflow_working_runs](./imgs/mlflow_working_runs.png)