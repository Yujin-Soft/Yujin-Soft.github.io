---
layout: post
title: "PyCharm + Remote Docker + Tensorflow"
date: 2019-02-16
categories:
  - Study
description:
---

## PyCharm + Remote Docker + Tensorflow

잊어버릴까 봐 미리 정리

### Server Side
Ubuntu 16.04 또는 18.04 설치 후 nvidia 그래픽드라이버 설치한 뒤,
tensorflow.org 에서 install 가이드에 따라 설치.

docker 와 nvidia-docker 설치 완료하면

아래와 같이 default-runtime 부분을 추가해준다.
/etc/docker/daemon.json

```
{
  "default-runtime": "nvidia",
  "runtimes": {
    "nvidia":{
      "path": "/usr/bin/nvidia-container-runtime",
      "runtimeArgs": []
    }
  }
}
```

아래와 같이 ExecStart 끝부분에 IP 부분을 추가해준다.
/lib/systemd/system/docker.service

```
ExecStart=usrbindocker daemon -H fd:/ -H tcp://0.0.0.0:2375
```

위 변경이 끝나면

```
$ systemctl daemon-reload
$ sudo systemctl docker restart
```

위 작업이 끝나면 다음과 같이 실행해 볼 수 있다.

```
sudo docker run -it --rm -v $PWD:/tmp -w /tmp tensorflow/tensorflow:latest-gpu-py3
```


Ubuntu Server에 vsftpd를 설치하고 설정한다.

### Client Pycharm PRO
Pycharm을 열고 configure 에서 Preference -> Project Interpreter 클릭 후 우측에 톱니바퀴 눌러서 Add... 클릭

![Project Interpreter Add](https://github.com/Yujin-Soft/Yujin-Soft.github.io/raw/master/_screenshots/2019-02-16-003.png "Project Interpreter Add")

Add Python Interpreter 에서 좌측 Docker 탭 클릭 후 New... 클릭
아래와 같이 Name 설정, TCP socket에 IP 적어주고, Certificated folder는 뭔지 모르겠지만 프로젝트 폴더로 설정 그리고 OK
![Docker](https://github.com/Yujin-Soft/Yujin-Soft.github.io/raw/master/_screenshots/2019-02-16-004.png "Docker Setting")

Image name을 눌러보면 자동으로 다음과 같이 나타나고 tensorflow/tensorflow:latest-gpu-py3을 선택하고 OK를 눌러준다.
![Docker](https://github.com/Yujin-Soft/Yujin-Soft.github.io/raw/master/_screenshots/2019-02-16-005.png "Docker Setting")
![Docker](https://github.com/Yujin-Soft/Yujin-Soft.github.io/raw/master/_screenshots/2019-02-16-006.png "Docker Setting")

새 프로젝트로 Pure Python을 만들고 main.py 등의 작업 파일을 추가한다.
[Tools] -> [Deployment] -> Configuation... 을 클릭하여 현재 작업 중인 프로젝트 파일을 서버로 올릴 수 있는 FTP를 설정해 준다.
![FTP Setting](https://github.com/Yujin-Soft/Yujin-Soft.github.io/raw/master/_screenshots/2019-02-16-001.png "FTP Setting")
![FTP Setting](https://github.com/Yujin-Soft/Yujin-Soft.github.io/raw/master/_screenshots/2019-02-16-002.png "FTP Setting")

Run/Debug Configurations를 아래 그림과 같이 바꿔준다.
![Configurations](https://github.com/Yujin-Soft/Yujin-Soft.github.io/raw/master/_screenshots/2019-02-16-009.png "Run/Debug Configurations")

main.py 에 Python 예제 코드를 넣어본다.

```
import tensorflow as tf
mnist = tf.keras.datasets.mnist

(x_train, y_train),(x_test, y_test) = mnist.load_data()
x_train, x_test = x_train / 255.0, x_test / 255.0

model = tf.keras.models.Sequential([
  tf.keras.layers.Flatten(input_shape=(28, 28)),
  tf.keras.layers.Dense(512, activation=tf.nn.relu),
  tf.keras.layers.Dropout(0.2),
  tf.keras.layers.Dense(10, activation=tf.nn.softmax)
])
model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

model.fit(x_train, y_train, epochs=5)
model.evaluate(x_test, y_test)

```

그리고 수정된 main.py를 업로드

![FTP upload](https://github.com/Yujin-Soft/Yujin-Soft.github.io/raw/master/_screenshots/2019-02-16-007.png "FTP upload")

그리고 실행!

![Run](https://github.com/Yujin-Soft/Yujin-Soft.github.io/raw/master/_screenshots/2019-02-16-008.png "Run")
