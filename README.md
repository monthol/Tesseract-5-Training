# Training New Fonts with Tesseract 5
In this tutorial we will fine tune existing model to better read custom fonts, for this it is required Tesseract to be built from source as training Tesseract is not possible with the binary installer.
### So what fonts have standard Tesseract been trained with?
See `okfonts.txt` in each language folder of training data repository [langdata_lstm](https://github.com/tesseract-ocr/langdata_lstm). 
## Install Linux on Windows with WSL (Windows Subsystem for Linux)
Open PowerShell in administrator mode by right-clicking and selecting "Run as administrator", enter the `wsl --install` command, then restart your machine. After the installation is complete, setup your new username/password. Please note that whilst entering the Password, nothing will appear on screen. For more info, please refer to [Microsoft](https://learn.microsoft.com/en-us/windows/wsl/setup/environment#set-up-your-linux-username-and-password).

## Install Tesseract
Enter WSL from Powershell terminal if you are not yet on Linux, then go to your root user directory, we are going to install Tesseract on the root folder.

```
    wsl
    cd
```

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
## OCR with Thai language
Get the best (most accurated) trained model for the language at [tessdata_best](https://github.com/tesseract-ocr/tessdata_best) repository, save it to `tessdata` directory (i.e. `/usr/local/share/tessdata`), where Tesseract looks for the language file (.traineddata).

```
    wget https://raw.githubusercontent.com/tesseract-ocr/tessdata_best/master/eng.traineddata -P /usr/local/share/tessdata
    wget https://raw.githubusercontent.com/tesseract-ocr/tessdata_best/master/tha.traineddata -P /usr/local/share/tessdata
```

List the support languages on screen with this command `tesseract --list-langs`.

```
   eng
   tha
```

### Try OCR Thai text with this [image](https://drive.google.com/file/d/1l-lUKGSAjCIhrhqgu959EGynNbnxyWlY/view?usp=share_link)

```
    cd
    mkdir images
    wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1l-lUKGSAjCIhrhqgu959EGynNbnxyWlY' -O images/023.jpg
    tesseract -l tha images/023.jpg stdout
```
