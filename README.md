# Training New Fonts with Tesseract 5
In this tutorial we will fine tune existing model to better read custom fonts, for this it is required Tesseract to be built from source as training Tesseract is not possible with the binary installer.
#### So what fonts have standard Tesseract been trained with?
See `okfonts.txt` in each language folder of training data repository [langdata_lstm](https://github.com/tesseract-ocr/langdata_lstm).
## 1. Build Tesseract from Source with Training Tools
### Install Linux on Windows with WSL (Windows Subsystem for Linux)
Open PowerShell in administrator mode by right-clicking and selecting "Run as administrator", enter the `wsl --install` command, then restart your machine. After the installation is complete, setup your new username/password. Please note that whilst entering the Password, nothing will appear on screen. For more info, please refer to [Microsoft](https://learn.microsoft.com/en-us/windows/wsl/setup/environment#set-up-your-linux-username-and-password).

### Install Tesseract
Enter WSL from Powershell terminal if you are not yet on Linux, then go to your root user directory, we are going to install Tesseract on the root folder.

```
    wsl
    cd
```
For error: [Windows Subsystem For Linux has no Installed Distributions] (https://appuals.com/how-to-fix-windows-subsystem-for-linux-has-no-installed-distributions-error/)
First updating sources, install all dependencies and training tools (copy/paste all and run in one go)

```
    sudo apt update && sudo apt upgrade
    sudo apt-get install automake
    sudo apt-get install ca-certificates 
    sudo apt-get install libtool
    sudo apt-get install make
    sudo apt-get install g++ # or clang++ (presumably)
    sudo apt-get install autoconf automake libtool
    sudo apt-get install pkg-config
    sudo apt-get install libpng-dev
    sudo apt-get install libjpeg8-dev
    sudo apt-get install libtiff5-dev
    sudo apt-get install zlib1g-dev
    sudo apt-get install libicu-dev
    sudo apt-get install libpango1.0-dev
    sudo apt-get install libcairo2-dev
    sudo apt-get install libleptonica-dev
    sudo apt install bc
```

Clone the Tesseract repository from github.

```
    git clone https://github.com/tesseract-ocr/tesseract.git
```

Select the latest version (5.3.0)

```
    cd tesseract
    git checkout 5.3.0
```

Build Tesseract with training tools by running the following one line at a time, see [Compiling–GitInstallation](https://tesseract-ocr.github.io/tessdoc/Compiling-%E2%80%93-GitInstallation.md) for details:

```
    ./autogen.sh
    ./configure --disable-debug 'CXXFLAGS=-g -O3'
    make
    sudo make install
    sudo ldconfig
    make training
    sudo make training-install    
```

Tesseract is now installed, run this command `tesseract -v` to check it out, the screen now should show something like:

```
    tesseract 5.3.0
    leptonica-1.82.0
     libgif 5.1.9 : libjpeg 8d (libjpeg-turbo 2.1.1) : libpng 1.6.37 : libtiff 4.3.0 : zlib 1.2.11 : libwebp 1.2.2 : libopenjp2 2.4.0
    Found AVX512BW
    Found AVX512F
    Found AVX512VNNI
    Found AVX2
    Found AVX
    Found FMA
    Found SSE4.1
    Found OpenMP 201511
```
### OCR with Thai language
Get the best (most accurated) trained model for the language at [tessdata_best](https://github.com/tesseract-ocr/tessdata_best) repository, save it to `tessdata` directory (i.e. `/usr/local/share/tessdata`), where Tesseract looks for the language file (.traineddata).

```
    sudo wget https://raw.githubusercontent.com/tesseract-ocr/tessdata_best/master/eng.traineddata -P /usr/local/share/tessdata
    sudo wget https://raw.githubusercontent.com/tesseract-ocr/tessdata_best/master/tha.traineddata -P /usr/local/share/tessdata
```
sidenote : Tesseract provides three types of models:- `tessdata_fast`, `tessdata_best` and `tessdata`. `tessdata_fast` is the default, balances speed and accuracy. For fine-tuning always use `tessdata_best`. `tessdata` is the lagacy models. See [Tesseract](https://tesseract-ocr.github.io/tessdoc/Data-Files.html) for more details.

List the support languages on screen with this command `tesseract --list-langs`.

```
   eng
   tha
```

#### Try OCR Thai text with this [image](https://drive.google.com/file/d/1l-lUKGSAjCIhrhqgu959EGynNbnxyWlY/view?usp=share_link)

```
    cd
    mkdir images
    sudo wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1l-lUKGSAjCIhrhqgu959EGynNbnxyWlY' -O images/023.jpg
    tesseract -l tha images/023.jpg stdout
```
    
## 2. Install `tesstrain`
[`tesstrain`](https://github.com/tesseract-ocr/tesstrain) is a set of Python tools that allow us to work with `make` files to train custom Tesseract models

```
    git clone https://github.com/tesseract-ocr/tesstrain.git
    cd tesstrain # wherever you saved this folder
    make tesseract-langdata
``` 

## 3. Create Training Data 

### 3.1 Get Language Data
[langdata_lstm](https://github.com/tesseract-ocr/langdata_lstm) repository provides source training data for Tesseract for lots of languages. We need at least English data to begin with, plus additional languages we do training (Thai, in this case).
Go to this [Google Drive folder](https://drive.google.com/drive/folders/1UZNBOdP7aAzLiTQrlFPoLIlojisw7jKi?usp=sharing) and download `eng` and `tha` to your local C drive
```
    mkdir langdata #on path where your heart desires
    cd langdata
    cp -r /mnt/c/.../eng .
    cp -r /mnt/c/.../tha .   
```
### 3.2 Get Fonts
See [Fonts for Tesseract training](https://tesseract-ocr.github.io/tessdoc/Fonts.html) for more detail.
#### List all available fonts on local computer

```
    text2image --fonts_dir /usr/share/fonts –list_available_fonts
```

#### List font for particular language to file

```
    text2image --find_fonts \
    --fonts_dir /usr/share/fonts \
    --text ./langdata/tha/tha.training_text \
    --min_coverage .9 \
    --outputbase ./langdata/tha/tha \
    |& grep raw \
    | sed -e 's/ :.*/@ \\/g' \
    | sed -e "s/^/  '/" \
    | sed -e "s/@/'/g" > ./langdata/tha/fontslist.txt
 ```

#### Installing new fonts on WSL
First copy font file (.otf) to the font folder. (e.g. [Font Chulalongkorn](https://www.chula.ac.th/download/8905/))

 ```
    cp /mnt/c/Users/<path to font file> /usr/local/share/fonts
    cd /usr/local/share/fonts
    fc-cache -f -v
    fc-list | grep "<name-of-font>" #This checks if fonts installed correctly
 ```

Or use existing Windows fonts by [following instruction here](https://x410.dev/cookbook/wsl/sharing-windows-fonts-with-wsl/):

### 3.3 Create ground truth and box files
Tesseract 5 requires images with single-line text for training, for this we can use @AstuteJoe's [Python script](https://github.com/astutejoe/tesseract_tutorial/blob/main/split_training_text.py) (See also his accompanied [Youtube tutorial](https://www.youtube.com/watch?v=KE4xEzFGSU8)) to create ground truth images and transcription from our `langdata` as many as we like. This script in turn run [`text2image`](https://github.com/tesseract-ocr/tesseract/blob/main/doc/text2image.1.asc) tool that installed with Tesseract. (please install Python3 if not yet done). Edit this script to reflect our project-

> Line 6 -- point to training text path (e.g. 'langdata/tha/tha.training_text')  
> Line 14 -- new model name (e.g. use font name for model name 'tesstrain/data/<font-name>-ground-truth')  
> Line 21 -- `count` is number of generated lines (files)  
> Line 36 -- the font name used to generated image files, and so to be trained  
> Line 43 -- width size of the generated image, `6000` is recommended for Thai language.  
> Line 48 -- path to unicharset file e.g. '--unicharset_file=langdata/tha/tha.unicharset'  

Running the python script generates sets of three files: tif image `.tif`, ground-truth text `.gt.txt` and coordinate of each letter's box `.box`, as many as `count` parameter, to the output directory. [See some file samples here](https://drive.google.com/drive/folders/198sJ5iAyqhtEMoMqy0oE0Td3SKK_gtyf?usp=sharing).
    
## 4. Training
We start training the model with small number of iterations (e.g. 200)
```
    cd tesstrain
    make training MODEL_NAME=<model_name> START_MODEL=tha TESSDATA=../tesseract/tessdata MAX_ITERATIONS=200
```

The result screen from the training :-

```
2 Percent improvement time=100, best error was 100 @ 0
At iteration 100/100/100, Mean rms=5.949000%, delta=59.381000%, BCER train=54.457000%, BWER train=93.801000%, skip ratio=0.000000%,  New best BCER = 54.457000 wrote best model:data/JasmineD/checkpoints/JasmineD_54.457000_100_100.checkpoint wrote checkpoint.

At iteration 200/200/200, Mean rms=6.002000%, delta=58.242000%, BCER train=70.099000%, BWER train=96.113000%, skip ratio=0.000000%,  New worst BCER = 70.099000 wrote checkpoint.

Finished! Selected model with minimal training error rate (BCER) = 54.457
lstmtraining \
--stop_training \
--continue_from data/JasmineD/checkpoints/JasmineD_checkpoint \
--traineddata data/JasmineD/JasmineD.traineddata \
--model_output data/JasmineD.traineddata
Loaded file data/JasmineD/checkpoints/JasmineD_checkpoint, unpacking...
```
The command first create training data set `.lstmf` files, one for each successful training of an image under the ground truth folder and `.traineddata` under `<model-name>` folder, to be used in training.
The iteration number are `learning`/`training`/`samples` iteration, as not all samples can be used for training, e.g. bad box file, and some training iteration does not yield any learning (delta=0).    
Here we try to optimize the BCER error rate by increase the number of iterations (default value is `10000`) or the number of training data (`count` in the python script) in the next run.
If something went wrong or maybe we want to increase `count` number, we can start over by cleaning up with command :
    
```
    make clean MODEL_NAME=<model_name>
```
    
## 5. Evaluate and deploy new model
To deploy new model, just copy `<model-name>.traineddata` to tesseract 's `tessdata` folder 
    
```
    cp tesstrain/data/<model-name>/<model-name>.traineddata /usr/local/share/tessdata
```    
### Training for handwritten text or additional characters
To train handwritten text, first create one-line images of handwriting, labeling can be done with this [R package tool](https://github.com/arcruz0/tesseractgt) to help speed things up. Also, see [its blog post](https://arcruz0.github.io/posts/finetuning-tess/index.html) for training with addtional characters not included in `.unicharset` (e.g. ๚, ๛, ๏)
    
    
