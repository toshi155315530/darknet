![Darknet Logo](http://pjreddie.com/media/files/darknet-black-small.png)

# 環境

ubuntu    


```
nvcc -v
nvidia-smi
```

# インストール

```
git clone https://github.com/miyamotok0105/darknet.git
cd darknet
```

Makefileの修正    

```
GPU=1に変更
```

```
make
```

# 実行

```
wget https://pjreddie.com/media/files/yolo.weights
./darknet detect cfg/yolo.cfg yolo.weights data/dog.jpg
```

# 閾値を変更

閾値を変更すると、認識される率が変わる。

```
./darknet detect cfg/yolo.cfg yolo.weights data/dog.jpg -thresh 0
```

# PASCAL Visual Object Classes形式で学習
PASCAL（Pattern Analysis, Statistical Modelling and Computational Learning）というヨーロッパの研究コミュニティが2005 年から 2012 年行なっていたコンペ
で使用していたデータセット。


```
wget https://pjreddie.com/media/files/VOCtrainval_11-May-2012.tar
wget https://pjreddie.com/media/files/VOCtrainval_06-Nov-2007.tar
wget https://pjreddie.com/media/files/VOCtest_06-Nov-2007.tar
tar xf VOCtrainval_11-May-2012.tar
tar xf VOCtrainval_06-Nov-2007.tar
tar xf VOCtest_06-Nov-2007.tar
python voc_label.py
cat 2007_train.txt 2007_val.txt 2012_*.txt > train.txt
mkdir backup
wget https://pjreddie.com/media/files/darknet19_448.conv.23
#学習
#cfg/voc.dataとcfg/yolo-voc.cfgを環境に合わせて修正。
./darknet detector train cfg/voc.data cfg/yolo-voc.cfg darknet19_448.conv.23
#テスト
./darknet detector test cfg/voc.data cfg/yolo-voc-test.cfg yolo.weights data/dog.jpg
```

# 学習データを作成する

toolを使用して学習データを作る。    
https://github.com/miyamotok0105/BBox-Label-Tool    
下記のフォーマットへである必要がある。    

```
#class_num area1 area2 area3 area4
11 10 10 100 100
14 20 20 120 130
```

# 独自学習データを使って学習

output_label_list.pyで学習するファイル一覧を作成。    
cfg/voc_hoge.dataとcfg/yolo-voc_hoge.cfgを環境に合わせて修正。
学習の場合はbatchを64に、テストの場合はbatchを1に変更。   
classes数を変更する。     
filters数は下記の計算式を使う。    

クラス数を変えた時の計算    
(classes + (coords + 4) + 1)*(NUM)    
6クラスの場合    
(6+4+1)5=55    
2クラスの場合    
(2+4+1)5=35    
1クラスの場合    
(1+4+1)5=30    

darknet/cfg/yolo-dog.cfgのファイル
```
[net]
# Testing
# batch=1
# subdivisions=1
# Training
batch=64
subdivisions=8

...


[convolutional]
size=1
stride=1
pad=1
filters=30 #最終層の値を変更
activation=linear


[region]
anchors =  1.3221, 1.73145, 3.19275, 4.00944, 5.05587, 8.09892, 9.47112, 4.84053, 11.2364, 10.0071
bias_match=1
classes=1 #class数を変える
coords=4
num=5

...

```

trainとvalidのパスを変更する。

darknet/cfg/dog.dataのファイル
```
classes= 1 #class数を変更
train  = /home/pjreddie/data/voc/train.txt #pathを変更
valid  = /home/pjreddie/data/voc/2007_test.txt #pathを変更
names = data/dog.names #labelとして使う名前を変えたい場合
backup = backup
```

学習データ作成が終わったら学習をする。

```
#学習
./darknet detector train cfg/voc_hoge.data cfg/yolo-voc_hoge.cfg darknet19_448.conv.23
```

学習が終わったらテストをする。    

```
#テスト
./darknet detector test cfg/voc_hoge.data cfg/yolo-voc_hoge.cfg yolo.weights data/dog.jpg
```


# 参照


https://pjreddie.com/darknet/


