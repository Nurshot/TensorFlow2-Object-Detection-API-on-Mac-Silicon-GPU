# TensorFlow 2.x Object Detection on M Chip With GPU

    
    Bu kılavuz, TensorFlow'un 2.13.0 kararlı sürümü için hazırlanmıştır. 
    Macbook M çiplerinde TensorFlow'u GPU ile kullanmanıza olanak sağlar.
    İsteğe bağlı olarak, aynı adımları (tensorflow-metal hariç) Windows veya Linux'ta takip ederek sorunsuz bir kurulum gerçekleştirebilirsiniz.


## Kurulan Cihaz
Macbook Pro M3 

## Conda Python 3.10 Yükleyin

Conda sayfasından isteğe bağlı olarak anaconda veya minicondayı [indirin](https://conda.io/projects/conda/en/latest/user-guide/install/index.html).

Ben bu rehberde minicondayı tercih edeceğim.

## Yeni bir Anaconda sanal ortamı oluşturun

Yeni bir Terminal penceresi açın

Aşağıdaki komutu yazın:

    conda create -n tensorflow pip python=3.10

Bu komut tensorflow adlı yeni bir ortam oluşturacaktır.

(NOT)Python 3.11 veya daha yeni sürümlerinde TensorFlow Object Detection API'yi kurduktan sonra hatalar aldım, ancak Python 3.10 sürümünde sorunsuz çalışmaktadır.




## Sanal ortam aktifleştirme

Sanal Ortamı Aktifleştirmek İçin Terminalimize Bu Kodu Yazıyoruz.

    conda activate tensorflow


Sanal ortamımızın etkinleştirildiğini anlamak için terminal penceresinin başında "(tensorflow)" yazması gerekir, aşağıdaki örnekte olduğu gibi.

    (tensorflow) ➜  Desktop



"which python" komutunu yazarak doğru Python ortamının eşleşip eşleşmediğini doğrulayabilirsiniz. Bu komut, terminalde kullanılan Python yürütülebilir dosyasının tam yolunu gösterecektir. Örneğin:

    /opt/miniconda3/envs/tensorflow/bin/python

## TensorFlow kurulumu

TensorFlow kurulumu ve testi için yapılması gerekenler;

#### TensorFlow yükleyin

Terminalimiz tensorflow sanal ortamında etkinken aşağıdaki komutu yazınız.

    pip install tensorflow==2.13.0

#### Kurulumunuzu doğrulayın

Aşağıdaki komutu Terminal penceresinde çalıştırın:

    python -c "import tensorflow as tf;print(tf.reduce_sum(tf.random.normal([1000, 1000])))"

Örnek Çıktı:

    tf.Tensor(-429.2135, shape=(), dtype=float32)

## GPU desteği

TensorFlow'u CPU ile de çalıştırabilirsiniz fakat GPU gücü , CPU ya göre daha iyi hesaplar. 

M çipleri ilk çıktığında GPU desteği verilmemişti fakat artık tek bir paket ile tensorflow'u GPU ile kullanabilirsiniz. [İlgili Kaynak](https://developer.apple.com/metal/tensorflow-plugin/)

### GPU Desteği Kurulumu
Terminalimizde sanal ortamımız aktifken:

    pip install tensorflow-metal



### GPU desteğini doğrulama

Gpu desteğini doğrulamak için kodu çalıştırın:

    python -c "import tensorflow as tf;print(tf.reduce_sum(tf.random.normal([1000, 1000])))"

Çıktı:
    
    2024-03-29 01:23:39.375077: I metal_plugin/src/device/metal_device.cc:1154] Metal device set to: Apple M3
    2024-03-29 01:23:39.375096: I metal_plugin/src/device/metal_device.cc:296] systemMemory: 16.00 GB
    2024-03-29 01:23:39.375100: I metal_plugin/src/device/metal_device.cc:313] maxCacheSize: 5.33 GB
    2024-03-29 01:23:39.375152: I tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc:303] Could not identify NUMA node of platform GPU ID 0, defaulting to 0. Your kernel may not have been built with NUMA support.
    2024-03-29 01:23:39.375271: I tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc:269] Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 0 MB memory) -> physical PluggableDevice (device: 0, name: METAL, pci bus id: <undefined>)
    tf.Tensor(-219.06421, shape=(), dtype=float32)

(NOT) Mac Silicon mimarisi UMA üzerine kurulmuştur bundan dolayı hata mesajını takmanıza gerek yok. Eğitirken, etkinlik monitöründen  GPU gücüyle çalıştığını görebilirsiniz.
# TensorFlow Object Detection API Kurulumu

TensorFlow'u kurduysanız, şimdi TensorFlow Object Detection API'sini kurabiliriz.

## TensorFlow Model Garden yükleme

Modelinizi eğiteceğiniz dizini belirleyin. Ben Desktop'ta Tensor adlı bir klasör oluşturdum. Terminalden onun içine girin.


Dizinimizin içindeyken, TensorFlow model reposunu klonluyoruz.

    git clone https://github.com/tensorflow/models.git




Bundan sonra Object Detection API için ön koşulları yüklemeliyiz. İlk önce protobuf'u indiriyoruz.

    (tensorflow) ➜  Tensor
    conda install -c anaconda protobuf

Tensor / models / research dizinine gidin 

    cd models/research
    protoc object_detection/protos/*.proto --python_out=.

(NOT) Bundan sonra terminalimizi kapatıp tekrar açıyoruz ve sanal ortamımızı aktif ediyoruz.

## COCO API ve PYYAML kurulumu
Object Detection API' sini kurmadan önce 2 adet yapmamız gereken işlem var. İlki kurulum yaparken hata almamak için PYYAML==5.4.1 versiyonunu kurmamız gerekmektedir. 

Bunun için:

    pip install wheel==0.40.0 
    
    pip install --no-build-isolation Cython==0.29.36 pyyaml==5.4.1


Ardından COCO API mizi kuruyoruz.

    pip install git+https://github.com/philferriere/cocoapi.git#subdirectory=PythonAPI



## Object Detection API Kurulumu

    cd /models/research

dizinine gidiyoruz ve

    cp object_detection/packages/tf2/setup.py .
    python -m pip install .

object detection api kurulumunu tamamlayın.


Kurulumu test etmek için research dizini içindeyken bu komutu çalıştırın

    python object_detection/builders/model_builder_tf2_test.py



Bu kısım sizi biraz bekletebilir.Fakat çalıştırmayı bitirdikten sonra yüksek ihtimalle hata mesajı alacaksınız. 

    File "/opt/miniconda3/envs/tensorflow2/lib/python3.10/site-packages/official/vision/image_classification/augment.py", line 31, in <module>
    from tensorflow.python.keras.layers.preprocessing import image_preprocessing as image_ops
    ModuleNotFoundError: No module named 'tensorflow.python.keras.layers.preprocessing'

Eğer bu üstteki hatayı aldıysanız panik yapmanıza gerek yok. Basit bir çözümü [var](https://github.com/tensorflow/models/issues/11129)


### Hatanın çözümü

Terminalimizde en sonda hatanın verdiği py dosyasının ismi yazacaktır. Baştan sona kopyalayıp bunu nano editörümüzde açıyoruz.

    sudo nano /opt/miniconda3/envs/tensorflow/lib/python3.10/site-packages/official/vision/image_classification/augment.py   

Açtıktan sonra:

    from tensorflow.python.keras.layers.preprocessing import image_preprocessing as image_ops

Yazan kodu

    from tensorflow.keras.preprocessing import image as image_ops

Şeklinde değiştirip CTRL+X e bastıktan sonra y tuşuna basıp ardından enterlayarak çıkıyoruz. 

 
Tekrardan kodumuzu yazıyoruz.

    python object_detection/builders/model_builder_tf2_test.py

Tadaa, gördüğünüz gibi!

    [  OK ] ModelBuilderTF2Test.test_invalid_faster_rcnn_batchnorm_update
    [ RUN] ModelBuilderTF2Test.test_invalid_first_stage_nms_iou_threshold
    INFO:tensorflow:time(_main_.ModelBuilderTF2Test.    test_invalid_first_stage_nms_iou_threshold): 0.0s
    I0329 01:57:27.708987 8012795904 test_util.py:2574] time(_main_.ModelBuilderTF2Test.test_invalid_first_stage_nms_iou_threshold): 0.0s
    [       OK ] ModelBuilderTF2Test.test_invalid_first_stage_nms_iou_threshold
    [ RUN      ] ModelBuilderTF2Test.test_invalid_model_config_proto
    INFO:tensorflow:time(_main_.ModelBuilderTF2Test.test_invalid_model_config_proto): 0.0s
    I0329 01:57:27.709128 8012795904 test_util.py:2574] time(_main_.ModelBuilderTF2Test.test_invalid_model_config_proto): 0.0s
    [       OK ] ModelBuilderTF2Test.test_invalid_model_config_proto
    [ RUN      ] ModelBuilderTF2Test.test_invalid_second_stage_batch_size
    INFO:tensorflow:time(_main_.ModelBuilderTF2Test.test_invalid_second_stage_batch_size): 0.0s
    I0329 01:57:27.709592 8012795904 test_util.py:2574] time(_main_.ModelBuilderTF2Test.test_invalid_second_stage_batch_size): 0.0s
    [       OK ] ModelBuilderTF2Test.test_invalid_second_stage_batch_size
    [ RUN      ] ModelBuilderTF2Test.test_session
    [  SKIPPED ] ModelBuilderTF2Test.test_session
    [ RUN      ] ModelBuilderTF2Test.test_unknown_faster_rcnn_feature_extractor
    INFO:tensorflow:time(_main_.ModelBuilderTF2Test.test_unknown_faster_rcnn_feature_extractor): 0.0s
    I0329 01:57:27.710033 8012795904 test_util.py:2574] time(_main_.ModelBuilderTF2Test.test_unknown_faster_rcnn_feature_extractor): 0.0s
    [       OK ] ModelBuilderTF2Test.test_unknown_faster_rcnn_feature_extractor
    [ RUN      ] ModelBuilderTF2Test.test_unknown_meta_architecture
    INFO:tensorflow:time(_main_.ModelBuilderTF2Test.test_unknown_meta_architecture): 0.0s
    I0329 01:57:27.710144 8012795904 test_util.py:2574] time(_main_.ModelBuilderTF2Test.test_unknown_meta_architecture): 0.0s
    [       OK ] ModelBuilderTF2Test.test_unknown_meta_architecture
    [ RUN      ] ModelBuilderTF2Test.test_unknown_ssd_feature_extractor
    INFO:tensorflow:time(_main_.ModelBuilderTF2Test.test_unknown_ssd_feature_extractor): 0.0s
    I0329 01:57:27.710497 8012795904 test_util.py:2574] time(_main_.ModelBuilderTF2Test.test_unknown_ssd_feature_extractor): 0.0s
    [       OK ] ModelBuilderTF2Test.test_unknown_ssd_feature_extractor
    ----------------------------------------------------------------------
    Ran 24 tests in 14.897s

    OK (skipped=1)


Bu mesaj, TensorFlow 2.13.0'u GPU desteğiyle ve Object Detection API'sini başarıyla kurduğumuz anlamına gelir. Artık veri setinizi ayarlayabilir ve işleyebilirsiniz.
