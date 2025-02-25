# A workflow for decoding, modifying and compiling a simple APK for demonstration purposes.
# this also assumes running Ubuntu on WSL (windows 11) which support GUI.
sudo apt update
sudo apt upgrade
sudo apt install -y openjdk-17-jdk
sudo apt install -y apktool
mkdir week6
cd week6
wget https://bitbucket.org/JesusFreke/smali/downloads/smali-2.5.2.jar -O smali.jar
wget https://bitbucket.org/JesusFreke/smali/downloads/baksmali-2.5.2.jar -O baksmali.jar
mkdir -p ~/tools && cd ~/tools
wget https://github.com/skylot/jadx/releases/download/v1.4.6/jadx-1.4.6.zip -O jadx.zip
unzip jadx.zip -d jadx
rm jadx.zip
chmod +x ~/tools/jadx/bin/jadx ~/tools/jadx/bin/jadx-gui
# add to path
echo 'export PATH=$HOME/tools/jadx/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
# Test JADX
jadx-gui # should launch a graphical interface!
jadx #command line version
# Get myapp.zip to the /home/week6 however you want.  Some options below.
# Option 1
# From Windows Explorer, \\wsl$ (hit enter)
# Browse to your linux install folder, home, <username>, week6
# copy the myapp.apk there.  If you want you can expand the it from the download zip first, or in the next step.
unzip myapp.zip
# Review code using jadx-gui.  
jadx-gui myapp.apk
# Source code > com.example.sabin.myapplication > MainActivity
# You should have some idea of what can be modified to give access to billy
unzip myapp.apk -d extracted_apk/
# review the contents, make sure you see a classes.dex.  The ls command should return the path before returning to prompt.  If not, unzip again
ls extracted_apk/classes.dex 
#convert to Smali
mkdir -p smali_code
java -jar ~/tools/baksmali.jar d extracted_apk/classes.dex -o smali_code/
# verify and find smali
find smali_code -name "*.smali"
nano smali_code/com/example/app/MainActivity.smali
# make changes, save and exit.
# recompile smali back into classes.dex
mkdir -p new_dex
java -jar ~/tools/smali.jar a smali_code -o new_dex/classes.dex
# Replace classes.dex in APK with the new one
zip -d myapp.apk classes.dex
zip -r myapp.apk new_dex/classes.dex
# Generate a signing key
keytool -genkey -v -keystore my-release-key.keystore -alias mykey \
        -keyalg RSA -keysize 2048 -validity 10000
# Sign apk
jarsigner -verbose -keystore my-release-key.keystore -sigalg SHA256withRSA -digestalg SHA-256 myapp.apk mykey
# verify
jarsigner -verify -verbose -certs myapp.apk

# copy the app to your desktop and run in emulator to verify it works.
