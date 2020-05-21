This instructions is to setup coral USB dongle on raspberry pi(https://coral.ai/)  and run coral rest server  
How to create an image for raspberry is out of scope.  

Run the commands bellow  
---------------------------

echo "deb https://packages.cloud.google.com/apt coral-edgetpu-stable main" | sudo tee /etc/apt/sources.list.d/coral-edgetpu.list  
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -  
sudo apt-get update  

sudo apt-get install -y libopenjp2-7  
sudo apt-get install -y libtiff5  
sudo apt-get install -y libatlas-base-dev  
sudo apt-get install -y libedgetpu1-std  
sudo apt-get install -y python3-pip  
sudo apt-get install -y git  

sudo pip3 install https://dl.google.com/coral/python/tflite_runtime-2.1.0.post1-cp37-cp37m-linux_armv7l.whl  
sudo pip3 install https://dl.google.com/coral/edgetpu_api/edgetpu-2.14.0-py3-none-any.whl  
sudo pip3 install flask  
sudo pip3 install --upgrade --force-reinstall click  


git clone https://github.com/google-coral/edgetpu.git  

git clone https://github.com/google-coral/tflite.git  
cd tflite/python/examples/classification  
sudo bash install_requirements.sh  

** The basic installation is done **  
Now you can plug in the coral dongle to USB. If it was plugged before, please remove and plug again  
To check that installation was succeful please run the command bellow  
python3 classify_image.py --model models/mobilenet_v2_1.0_224_inat_bird_quant_edgetpu.tflite --labels models/inat_bird_labels.txt --input images/parrot.jpg  

coral-pi-rest-server installation  
----------------------------------
cd ~/coral/  
git clone https://github.com/robmarkcole/coral-pi-rest-server.git  
cd coral-pi-rest-server/  

** Installation done **  
To check the coral rest server please run  
python3 coral-app.py --models_directory ~/coral/edgetpu/test_data/  
In another terminal please run  
cd ~coral/coral-pi-rest-server   
curl -X POST -F image=@images/test-image3.jpg 'http://localhost:5000/v1/vision/detection'  
  
  
Run the coral reset server as service  
------------------------------------  
replace coral.service content with following  
[Unit]
Description=Flask app exposing tensorflow lite model on the Coral USB stick
After=network.target

[Service]
ExecStart=python3 /home/pi/coral/coral-pi-rest-server/coral-app.py --models_directory /home/pi/coral/edgetpu/test_data/
WorkingDirectory=/home/pi/coral/coral-pi-rest-server
StandardOutput=inherit
StandardError=inherit
Restart=always
User=pi

[Install]
WantedBy=multi-user.target

And run the coomand bellow
sudo cp coral.service /etc/systemd/system/coral.service  
sudo systemctl daemon-reload  
  
To start the service  
sudo systemctl start coral.service  
To check that everything is running well  
sudo systemctl status coral.service  
To enable the service after every startup
sudo systemctl enable  coral.service  



