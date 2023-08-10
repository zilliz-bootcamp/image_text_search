:exclamation::exclamation: **This repo will no longer be maintained, please visit https://github.com/milvus-io/bootcamp** :exclamation: :exclamation:

# Image and text retrieval system based on Milvus

This project is based on the paper **[Composing Text and Image for Image Retrieval - An Empirical Odyssey](https://arxiv.org/abs/1812.07119)**. In this project, we will build an image and text retrieval system which uses modified description text to query and retrieve images.

## Prerequisite

**[Milvus 0.10.4](https://www.milvus.io/cn/docs/v0.10.4/milvus_docker-gpu.md)**

**MySQL**

**[Tirg](https://github.com/google/tirg)**

## Data preparation

Download dataset from this [external website](https://drive.google.com/file/d/1wPqMw-HKmXUG2qTgYBiTNUnjz83hA2tY/view?usp=sharing).

Make sure the dataset includes following files: `<dataset_path>/css_toy_dataset_novel2_small.dup.npy` `<dataset_path>/images/*.png`

## Run TIRG model

First, clone the TIRG model:

```
cd tirg
git clone https://github.com/google/tirg.git
```

Then install the Python environment:

```
pip install -r requirement
```

To run our training & testing:

```
cd tirg
python main.py --dataset=css3d --dataset_path=./CSSDataset --num_iters=160000 \
  --model=tirg --loss=soft_triplet --comment=css3d_tirg

python main.py --dataset=css3d --dataset_path=./CSSDataset --num_iters=160000 \
  --model=tirg_lastconv --loss=soft_triplet --comment=css3d_tirgconv
```

If you don’t want to run the training and testing model separately, run the baseline model directly：

```
python main.py --dataset=css3d --dataset_path=./CSSDataset --num_iters=160000 \
  --model=concat --loss=soft_triplet --comment=css3d_concat
```

All log files will be saved at `./runs/<timestamp><comment>`. Use tensorboard to monitor the following benchmarks:training loss, retrieval performance of training, retrieval performance of testing:

```
tensorboard --logdir ./runs/ --port 8888
```

## Load data

Before running the script, please modify the parameters in **webserver/src/common/config.py**:

| Parameter    | Description               | Default setting |
| ------------ | ------------------------- | --------------- |
| MILVUS_HOST  | milvus service ip address | 127.0.0.1       |
| MILVUS_PORT  | milvus service port       | 19530           |
| MYSQL_HOST   | Mysql service ip          | 127.0.0.1       |
| MYSQL_PORT   | Mysql service port        | 3306            |
| MYSQL_USER   | Mysql user name           | root            |
| MYSQL_PWD    | Mysql password            | 123456          |
| MYSQL_DB     | Mysql datebase name       | mysql           |
| MILVUS_TABLE | default table name        | milvus_k        |

Please modify the parameters of Milvus and MySQL according to your environment.
Before executing this code, you need to move the target image vector **img.npy** file under the **tirg/css** path
```
$ cd ..
$ python insert_milvus.py ./tirg/css
```

## Run webserver

Start the image-text retrieval system.

```
$ python main.py
# You are expected to see the following output.
Using backend: pytorch
Using backend: pytorch
INFO:     Started server process [35272]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://192.168.1.58:7000 (Press CTRL+C to quit)
```

> Type http://127.0.0.1:7000/docs into your browser to get the APIs.
