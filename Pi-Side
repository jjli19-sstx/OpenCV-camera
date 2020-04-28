# Web streaming example
# Source code from the official PiCamera package
# http://picamera.readthedocs.io/en/latest/recipes2.html#web-streaming

#v components for camera
import io
import picamera
#v components for stream and tcp server(s)
import logging
import socketserver
from threading import Condition
from http import server
import socket
import threading
#v components for servo hat
from adafruit_servokit import ServoKit
import time



PAGE="""\
<html>
<head>
<title>Raspberry Pi - Surveillance Camera</title>
</head>
<body>
<center><h1>Raspberry Pi - Surveillance Camera</h1></center>
<center><img src="stream.mjpg" width="640" height="480"></center>
</body>
</html>
"""
#ipaddress=input('please enter your ip address')
strPort=input('please enter an available port')
Port=int(strPort)
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.bind(('',Port))
s.listen(1)
kit=ServoKit(channels=16)
angle=90
vangle=90
turner=410
kit.servo[8].angle= 90
kit.servo[9].angle= 90
    

#(conn, addr)=s.accept()

class StreamingOutput(object):
    def __init__(self):
        self.frame = None
        self.buffer = io.BytesIO()
        self.condition = Condition()

    def write(self, buf):
        if buf.startswith(b'\xff\xd8'):
            # New frame, copy the existing buffer's content and notify all
            # clients it's available
            self.buffer.truncate()
            with self.condition:
                self.frame = self.buffer.getvalue()
                self.condition.notify_all()
            self.buffer.seek(0)
        return self.buffer.write(buf)

class StreamingHandler(server.BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/':
            self.send_response(301)
            self.send_header('Location', '/index.html')
            self.end_headers()
        elif self.path == '/index.html':
            content = PAGE.encode('utf-8')
            self.send_response(200)
            self.send_header('Content-Type', 'text/html')
            self.send_header('Content-Length', len(content))
            self.end_headers()
            self.wfile.write(content)
        elif self.path == '/stream.mjpg':
            self.send_response(200)
            self.send_header('Age', 0)
            self.send_header('Cache-Control', 'no-cache, private')
            self.send_header('Pragma', 'no-cache')
            self.send_header('Content-Type', 'multipart/x-mixed-replace; boundary=FRAME')
            self.end_headers()
            try:
                while True:
                    with output.condition:
                        output.condition.wait()
                        frame = output.frame
                    self.wfile.write(b'--FRAME\r\n')
                    self.send_header('Content-Type', 'image/jpeg')
                    self.send_header('Content-Length', len(frame))
                    self.end_headers()
                    self.wfile.write(frame)
                    self.wfile.write(b'\r\n')
            except Exception as e:
                logging.warning(
                    'Removed streaming client %s: %s',
                    self.client_address, str(e))
        else:
            self.send_error(404)
            self.end_headers()

class StreamingServer(socketserver.ThreadingMixIn, server.HTTPServer):
    allow_reuse_address = True
    daemon_threads = True

def watcher(n, name):#recieving data from tcp server
    print("awaiting connection")
    (conn,addr)=s.accept()
    print("connected")
    while True:#optimal position x:420 y:380
        data = conn.recv(1024)
        print(data)
        breaker(data,conn)
        time.sleep(0.5)
        
def Aservo(channel, position, conn):
    print("servo fired")
    kit.servo[channel].angle= position
    confirmation="confirmed"
    conn.send(confirmation.encode())

def Cservo(channel, spd):
    kit.servo[channel].throttle=spd

def breaker(data,conn):
    print(type(data))
    if(data.decode().find('x') != -1 and data.decode().find('y')==-1):
            xdata=data.decode().replace('x','')
            print('got x!')
            int_xdata=int(xdata)
            h=threading.Thread(target=Aservo, args=(8,int_xdata, conn))
            h.start()
            #Aservo(8, int_xdata, conn)
            
                        
    elif(data.decode().find('y')!=-1 and data.decode().find('x')==-1):
            ydata=data.decode().replace('y','')
            print('got y!')
            int_ydata=int(ydata)
            v=threading.Thread(target=Aservo, args=(9,int_ydata, conn))
            v.start()
            #Aservo(9, int_ydata, conn)
    
with picamera.PiCamera(resolution='640x480', framerate=24) as camera:
    t=threading.Thread(target=watcher, name='thread1', args=(1,'threader'))
    output = StreamingOutput()
    #Uncomment the next line to change your Pi's Camera rotation (in degrees)
    #camera.rotation = 90
    camera.start_recording(output, format='mjpeg')
    t.start()# starts the threading programing
    try:
        address = ('', 8000)
        server = StreamingServer(address, StreamingHandler)
        server.serve_forever()
    finally:
        camera.stop_recording()
