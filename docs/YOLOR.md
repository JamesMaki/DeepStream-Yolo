# YOLOR usage

**NOTE**: Select the correct branch of the YOLOR repo before the conversion.

**NOTE**: The cfg file is required for the main branch.

* [Convert model](#convert-model)
* [Compile the lib](#compile-the-lib)
* [Edit the config_infer_primary_yolor file](#edit-the-config_infer_primary_yolor-file)
* [Edit the deepstream_app_config file](#edit-the-deepstream_app_config-file)
* [Testing the model](#testing-the-model)

##

### Convert model

#### 1. Download the YOLOR repo and install the requirements

```
git clone https://github.com/WongKinYiu/yolor.git
cd yolor
pip3 install -r requirements.txt
pip3 install onnx onnxsim onnxruntime
```

**NOTE**: It is recommended to use Python virtualenv.

#### 2. Copy conversor

Copy the `export_yolor.py` file from `DeepStream-Yolo/utils` directory to the `yolor` folder.

#### 3. Download the model

Download the `pt` file from [YOLOR](https://github.com/WongKinYiu/yolor) repo.

**NOTE**: You can use your custom model.

#### 4. Convert model

Generate the ONNX model file

- Main branch

  Example for YOLOR-CSP

  ```
  python3 export_yolor.py -w yolor_csp.pt -c cfg/yolor_csp.cfg --dynamic
  ```

- Paper branch

  Example for YOLOR-P6

  ```
  python3 export_yolor.py -w yolor-p6.pt --dynamic
  ```

**NOTE**: To convert a P6 model

```
--p6
```

**NOTE**: To change the inference size (defaut: 640 / 1280 for `--p6` models)

```
-s SIZE
--size SIZE
-s HEIGHT WIDTH
--size HEIGHT WIDTH
```

Example for 1280

```
-s 1280
```

or

```
-s 1280 1280
```

**NOTE**: To simplify the ONNX model (DeepStream >= 6.0)

```
--simplify
```

**NOTE**: To use dynamic batch-size (DeepStream >= 6.1)

```
--dynamic
```

**NOTE**: To use implicit batch-size (example for batch-size = 4)

```
--batch 4
```

**NOTE**: If you are using the DeepStream 5.1, remove the `--dynamic` arg and use opset 12 or lower. The default opset is 12.

```
--opset 12
```

#### 5. Copy generated files

Copy the generated ONNX model file and labels.txt file (if generated) to the `DeepStream-Yolo` folder

##

### Compile the lib

Open the `DeepStream-Yolo` folder and compile the lib

* DeepStream 6.3 on x86 platform

  ```
  CUDA_VER=12.1 make -C nvdsinfer_custom_impl_Yolo
  ```

* DeepStream 6.2 on x86 platform

  ```
  CUDA_VER=11.8 make -C nvdsinfer_custom_impl_Yolo
  ```

* DeepStream 6.1.1 on x86 platform

  ```
  CUDA_VER=11.7 make -C nvdsinfer_custom_impl_Yolo
  ```

* DeepStream 6.1 on x86 platform

  ```
  CUDA_VER=11.6 make -C nvdsinfer_custom_impl_Yolo
  ```

* DeepStream 6.0.1 / 6.0 on x86 platform

  ```
  CUDA_VER=11.4 make -C nvdsinfer_custom_impl_Yolo
  ```

* DeepStream 5.1 on x86 platform

  ```
  CUDA_VER=11.1 make -C nvdsinfer_custom_impl_Yolo
  ```

* DeepStream 6.3 / 6.2 / 6.1.1 / 6.1 on Jetson platform

  ```
  CUDA_VER=11.4 make -C nvdsinfer_custom_impl_Yolo
  ```

* DeepStream 6.0.1 / 6.0 / 5.1 on Jetson platform

  ```
  CUDA_VER=10.2 make -C nvdsinfer_custom_impl_Yolo
  ```

##

### Edit the config_infer_primary_yolor file

Edit the `config_infer_primary_yolor.txt` file according to your model (example for YOLOR-CSP with 80 classes)

```
[property]
...
onnx-file=yolor_csp.onnx
...
num-detected-classes=80
...
parse-bbox-func-name=NvDsInferParseYolo
...
```

**NOTE**: The **YOLOR** resizes the input with center padding. To get better accuracy, use

```
...
maintain-aspect-ratio=1
symmetric-padding=1
...
```

**NOTE**: By default, the dynamic batch-size is set. To use implicit batch-size, uncomment the line

```
...
force-implicit-batch-dim=1
...
```

##

### Edit the deepstream_app_config file

```
...
[primary-gie]
...
config-file=config_infer_primary_yolor.txt
```

##

### Testing the model

```
deepstream-app -c deepstream_app_config.txt
```

**NOTE**: The TensorRT engine file may take a very long time to generate (sometimes more than 10 minutes).

**NOTE**: For more information about custom models configuration (`batch-size`, `network-mode`, etc), please check the [`docs/customModels.md`](customModels.md) file.
