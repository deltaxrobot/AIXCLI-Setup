# AIXCLI-Setup
AIX software installation tool

## Introduciton
This is console application which uses to download and install environment for AIXCommandLineInterface

## Requirement
Windows 64-bit\
NVIDIA Driver Version >=452.39* (check it with command "nvidia-smi")\

If you use Windows Subsystem for linux environment (WSL):
- Windows version newer than 21h2 (OS build 10.19045.xxxx)
- Virtualization in BIOS is enable

## Installation

Download [last][] version of AIXCLI_SETUP.

Open a terminal as __administrator__ in __AIXCLI_SETUP__ folder, and run bellow command:
Default: 

```bash
AIXCLI_SETUP.exe install
```

For WSL:

```bash
AIXCLI_SETUP.exe install -s WSL
```
## Uninstallation

Run bellow command to uninstall AIXCLI:

```bash
AIXCLI_SETUP.exe uninstall
```


# AIX Cmd User manual
Instructions for using command for AIX.

## Activate AIX software with key
After the installation is finished. 
Open **New terminal** and type below command:

``` bash
AIX activate "your key license"
```

If you want use your key license on a new machine, run below commnad in old machine to release it:

``` bash
AIX unactivate
```

## Image extension

     Current version is support for: png, jpg, bmp

## Label Format

Image file name for Example: `foo.jpg`

- Segment:

     Label file name: `foo.txt`

     Label coordinate: `from 0 to 1`

     > id x1 y1 x2 y2 x3 y3 ...

     Example:

     ```txt
     0 0.31719558921416263 0.38029782655859684 0.15361778924980077 0.21635059496075829 .....
     3 0.31719558921416263 0.38029782655859684 0.15361778924980077 0.21635059496075829 .....
     ```
     
- Detect:

     Label file name: `foo.txt`

     Label coordinate: `from 0 to 1`

     > id x_center y_center width height

     Example:
     ```txt
     1 0.31719558921416263 0.38029782655859684 0.15361778924980077 0.21635059496075829
     0 0.31719558921416263 0.38029782655859684 0.15361778924980077 0.21635059496075829
     ```

- Orienty:

     Label file name: `foo.txt`

     Label coordinate: `from 0 to 1`

     > id x_center y_center width height degree

     Example:
     ```txt
     1 0.31719558921416263 0.38029782655859684 0.15361778924980077 0.21635059496075829 -180
     0 0.31719558921416263 0.38029782655859684 0.15361778924980077 0.21635059496075829 35.456
     ```

## Command Line Interface
- AIX Modes

     MODE (required) is one of: [init,train,detect]

     Run below command to see mode infomations:
     ``` bash
     AIX -h
     ```
     Output:
     ``` bash
     usage: AIX [-h] {init,train,detect} ...

     positional arguments:
     {init,train,detect}
     init               initial initialize process
     train              initial train process
     detect             initial detect process

     optional arguments:
     -h, --help           show this help message and exit
     ```
- Init Mode

     Help command for init mode:
     ``` bash
     AIX init -h
     ```
     Output:
     ``` bash
     usage: AIX init [-h] [-n OBJECTNAMES [OBJECTNAMES ...]] [-i IMAGEPATH] [-l LABELPATH]

     optional arguments:
     -h, --help            show this help message and exit
     -n OBJECTNAMES [OBJECTNAMES ...], --objectNames OBJECTNAMES [OBJECTNAMES ...]
                         name1 name2 name3 ...
     -i IMAGEPATH, --imagePath IMAGEPATH
                         full/path/to/images directory
     -l LABELPATH, --labelPath LABELPATH
                         full/path/to/labels directory
     ```
     Example:
     ``` bash
     AIX init -i "full/path/to/images directory" -l "full/path/to/labels directory" -n "mushroom" "wood" "shrimp"
     ```
- Train Mode

     Help command for train mode:
     ```bash
     AIX train -h
     ```
     Example:
     ```bash
     AIX train 
     #For auto-training: automatically read the training coefficients to determine whether to stop or continue training.
     ```
     ```bash
     AIX train --manual 
     #For manual training: you have to manually read the training coefficients to determine whether to stop or continue training.
     ```
     ```bash
     AIX train --manual batchSize=intValue imageSize=intValue learningRate=floatValue
     #Similar to the 'AIX train --manual' command, but you can pass additional arguments 
     ```
- Predict Mode

     Help command for predict mode:
     ```bash
     AIX predict -h
     ```
     Example:
     ```bash
     AIX predict -iou 0.5 -s "full/Path/to/image" #jpg,png,bmp
     ```
     ```bash
     AIX predict -iou 0.4 -s ip="127.0.0.1" port=5555
     #If 'ip' and 'port' are are pieces of information for a listening server, 'AIX' will connect to this server.
     #Else 'AIX' will open a new server with this 'ip' and 'port'.
     ```
## Prediction Result

coordinate: `pixcel`

- Segment:
     ```bash
     "#Object = object_name, x_center, y_center, x1, y1, x2, y2,... xn, yn, confidence;"
     ```
- Detect:
     ```bash
     "#Object = object_name, x_center, y_center, width, height, confidence;"
     ```
- Orienty:
     ```bash
     "#Object = object_name, x_center, y_center, width, height, degree, confidence;"
     ```

## The sending of an image via TCP socket

- C++

     ```cpp
     void RobotWindow::SendImageToExternalScript(cv::Mat input)
     {
     if (ui->cbDetectingAlgorithm->currentText() != "External Script")
          return;

     if (DeltaConnectionManager->ExternalScriptSocket == NULL || input.empty())
          return;

     int paras[3];
     paras[0] = input.cols;
     paras[1] = input.rows;
     paras[2] = input.channels();

     int len = 3 * sizeof(int);

     DeltaConnectionManager->ExternalScriptSocket->write((char*)paras, len);

     int colByte = input.cols*input.channels() * sizeof(uchar);
     for (int i = 0; i < input.rows; i++)
     {
          char* data = (char*)input.ptr<uchar>(i); //first address of the i-th line
          int sedNum = 0;
          char buf[1024] = { 0 };

          while (sedNum < colByte)
          {
               int sed = (1024 < colByte - sedNum) ? 1024 : (colByte - sedNum);
               memcpy(buf, &data[sedNum], sed);
               int SendSize = DeltaConnectionManager->ExternalScriptSocket->write(buf, sed);

               if (SendSize == -1)
                    return;
               sedNum += SendSize;
          }
     }
     }
     ```

- Python

     ```python
     import numpy as np
     import sys
     from typing import Union
     from PySide6 import QtCore, QtNetwork
     import cv2
     import functools


     class TcpServerError(Exception):...

     class ImageSendingAccepter:
          ACCEPT_KEY="ExternalScript\n"
          def __init__(self) -> None:
               self.accept=False
               self.latestUnusedData=None
          def isAccepted(self)->bool:
               return self.accept
          def updateAcceptanceStatusByData(self,data:QtCore.QByteArray):
               subData = data[:len(self.ACCEPT_KEY.encode())]
               if (subData.data().decode() == self.ACCEPT_KEY):
                    self.accept = True
               nextData =  data[len(self.ACCEPT_KEY.encode()):]
               self.__setLatestUnusedData(nextData)
          def getLatestUnusedData(self)->Union[QtCore.QByteArray,None]:
               """
               Return latestUnusedData if it not empty\n
               Else return None
               """
               latestUnusedData=self.latestUnusedData
               self.latestUnusedData=None
               return latestUnusedData
          def __setLatestUnusedData(self,newLatestUnusedData:QtCore.QByteArray):
               if newLatestUnusedData.isEmpty():
                    self.latestUnusedData=None
               else:
                    self.latestUnusedData=newLatestUnusedData


     class PredictingScriptsCommunicator(QtCore.QObject):
          """
          This class create TCP server to communicate with scripts Python
          """
          START_IMAGE_SENDING_KEY="deltax"
          imageSendingAccepted=QtCore.Signal()
          resultReceived=QtCore.Signal(str)
          def __init__(self, ip:str,port:int,parent: QtCore.QObject = None) -> None:
               super().__init__(parent)
               self._ip=ip
               self._port=port
               self.server = QtNetwork.QTcpServer()  
               self.socket = QtNetwork.QTcpSocket()
               self.imageSendingAccepter = ImageSendingAccepter()     
               self.server.newConnection.connect(self.__onNewConnection)
               self.__openServer()
          def __openServer(self):
               if not self.server.listen(QtNetwork.QHostAddress(self._ip),self._port):
                    raise TcpServerError(f"Script python could not start server IP: {str(QtNetwork.QHostAddress(self._ip))} ,port: {self._port}\n")
               sys.stdout.write(f"Script python server was open. IP:  {self.server.serverAddress()} ,port: {self.server.serverPort()}\n")
               sys.stdout.flush()
          def __onNewConnection(self):
               self.socket = self.server.nextPendingConnection()
               if self.socket.state() == QtNetwork.QTcpSocket.ConnectedState:
                    sys.stdout.write(f"Script python new connection established: {self.socket.peerAddress()}\n")
                    sys.stdout.flush()
               self.socket.readyRead.connect(self.__onReadyRead)
               self.socket.disconnected.connect(self.__onDisconnected)
          def __onReadyRead(self):
               data = self.socket.readAll()
               self.resultReceived.emit(data.data().decode())
               if self.imageSendingAccepter.isAccepted():
                    return
               self.imageSendingAccepter.updateAcceptanceStatusByData(data)
               if self.imageSendingAccepter.isAccepted():
                    self.socket.write(self.START_IMAGE_SENDING_KEY.encode())
                    self.imageSendingAccepted.emit()
          def __onDisconnected(self):
               sys.stdout.write(f"Script python client disconnected: from adress: {self.server.serverAddress()}, ip: {self.server.serverPort()}\n")
               sys.stdout.flush()
          def __getSed(self,colByte,sedNum):
               if 1024 < colByte - sedNum:
                    return 1024
               return colByte - sedNum          
          def writeImage(self,image:np.ndarray):
               if not self.socket.state() == QtNetwork.QTcpSocket.ConnectedState:
                    return
               if not self.imageSendingAccepter.isAccepted():
                    return
               height, width, channels = image.shape
               widthBytes = width.to_bytes(4, 'big')[::-1]
               heightBytes = height.to_bytes(4, 'big')[::-1]
               channelsBytes = channels.to_bytes(4, 'big')[::-1]
               shapeBytes = widthBytes + heightBytes + channelsBytes
               self.socket.write(shapeBytes)
               #send image data
               SIZE_OF_BYTE = 1
               colByte = width*channels * SIZE_OF_BYTE
               for i in range(0,height,1):
                    data = image[i].tobytes()
                    sedNum = 0
                    while (sedNum < colByte):
                         sed = self.__getSed(colByte,sedNum)
                         buf = data[sedNum:sedNum+sed]
                         sendSize = self.socket.write(buf)
                         if (sendSize == -1):
                              break
                         sedNum += sendSize  
          def __del__(self):
               self.server.close()
               self.server.deleteLater()

     if __name__ == "__main__":
          app = QtCore.QCoreApplication()
          image = cv2.imread(r"D:\TansFolder\python\testSomeThings\serversendImagetoAIXCLI\CamToolbox_20200121_154543.jpg")
          imageSendingTimer = QtCore.QTimer()
          imageSendingTimer.setInterval(500)
          predictingScriptsCommunicator = PredictingScriptsCommunicator("127.0.0.1",5555)
          predictingScriptsCommunicator.imageSendingAccepted.connect(imageSendingTimer.start)
          predictingScriptsCommunicator.resultReceived.connect(print)
          onTimeout = functools.partial(predictingScriptsCommunicator.writeImage,image)
          imageSendingTimer.timeout.connect(onTimeout)
          app.exec()
     ```
     

[last]: https://github.com/deltaxrobot/AIXCLI-Setup/archive/refs/tags/v1.0.0.zip
